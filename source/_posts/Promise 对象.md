---
title: Promise 对象
date: 2017-08-02 14:10:34
categories: ES6
---

Promise对象有以下两个特点

>* Promise对象代表一个异步操作
 - 有三种状态：Pending（进行中）、Resolved（已完成，又称 Fulfilled）和Rejected（已失败）
>* 一旦状态改变，就不会再变，任何时候都可以得到这个结果
 - Promise对象的状态改变，只有两种可能：从Pending变为Resolved和从Pending变为Rejected。
 - 事件的特点是，如果你错过了它，再去监听，是得不到结果的.
 - 有了Promise对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。
 
缺点
>* 无法取消Promise，一旦新建它就会立即执行，无法中途取消。
>* 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。
>* 当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

注意
如果某些事件不断地反复发生，一般来说，使用 Stream 模式是比部署Promise更好的选择。
<!--more-->
### 基本用法
下面代码创造了一个Promise实例:
```
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```
Promise实例生成以后，可以用then方法分别指定Resolved状态和Rejected状态的回调函数。
```
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```
then方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为Resolved时调用，第二个回调函数是Promise对象的状态变为Rejected时调用。其中，第二个函数是可选的，不一定要提供。
```
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('Resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// Resolved
```
上面代码中**，Promise 新建后立即执行**，所以首先输出的是Promise。然后，then方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，所以Resolved最后输出.

```
var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

    function handler() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```
getJSON是对 XMLHttpRequest 对象的封装，用于发出一个针对 JSON 数据的 HTTP 请求，并且返回一个Promise对象。需要注意的是，**在getJSON内部，resolve函数和reject函数调用时，都带有参数。**

Promise嵌套
```
var p1 = new Promise(function (resolve, reject) {
  // ...
});

var p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```
**注意**，这时p1的状态就会传递给p2，也就是说，**p1的状态决定了p2的状态**。如果p1的状态是Pending，那么p2的回调函数就会等待p1的状态改变；如果p1的状态已经是Resolved或者Rejected，那么p2的回调函数将会立刻执行。
```
var p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

var p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
```
上面代码中，p1是一个Promise，3秒之后变为rejected。p2的状态在1秒之后改变，resolve方法返回的是p1。由于p2返回的是另一个 Promise，导致p2自己的状态无效了，由p1的状态决定p2的状态。所以，后面的then语句都变成针对后者（p1）。又过了2秒，p1变为rejected，导致触发catch方法指定的回调函数.

**注意，调用resolve或reject并不会终结 Promise 的参数函数的执行。**

```
new Promise((resolve, reject) => {
  resolve(1);
  console.log(2);
}).then(r => {
  console.log(r);
});
// 2
// 1
```
调用resolve(1)以后，后面的console.log(2)还是会执行，并且会首先打印出来。这是因为立即 resolved 的 Promise 是在本轮事件循环的末尾执行，总是**晚于本轮循环的同步任务**.
调用resolve或reject以后，Promise 的使命就完成了，后继操作应该放到then方法里面，而不应该直接写在resolve或reject的后面。所以，最好在它们前面加上return语句，这样就不会有意外.
```
new Promise((resolve, reject) => {
  return resolve(1);
  // 后面的语句不会执行
  console.log(2);
})
```
### Promise.prototype.then()
then方法是定义在原型对象Promise.prototype上的。它的作用是为 Promise 实例添加状态改变时的回调函数。then方法的第一个参数是Resolved状态的回调函数，第二个参数（可选）是Rejected状态的回调函数。

then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。

采用链式的then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。
```
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("Resolved: ", comments);
}, function funcB(err){
  console.log("Rejected: ", err);
});
```
第一个then方法指定的回调函数，返回的是另一个Promise对象。这时，第二个then方法指定的回调函数，就会等待这个新的Promise对象状态发生变化。如果变为Resolved，就调用funcA，如果状态变为Rejected，就调用funcB。

采用箭头函数，上面的代码:
```
getJSON("/post/1.json").then(
  post => getJSON(post.commentURL)
).then(
  comments => console.log("Resolved: ", comments),
  err => console.log("Rejected: ", err)
);
```
### Promise.prototype.catch() 
Promise.prototype.catch方法是.then(null,rejection)的别名，用于指定发生错误时的回调函数。
```
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```
getJSON方法返回一个**Promise对象**，如果该对象状态变为Resolved，则会调用then方法指定的回调函数；如果**异步操作抛出错误**，状态就会变为Rejected，就会**调用catch方法指定的回调函数**，处理这个错误。另外，**then方法指定的回调函数，如果运行中抛出错误，也会被catch方法捕获。**

如果Promise状态已经变成Resolved，再抛出错误是无效的。
```
var promise = new Promise(function(resolve, reject) {
  resolve('ok');
  throw new Error('test');
});
promise
  .then(function(value) { console.log(value) })
  .catch(function(error) { console.log(error) });
// ok
```
Promise 在resolve语句后面，再抛出错误，不会被捕获，等于没有抛出。因为 **Promise 的状态一旦改变，就永久保持该状态，不会再变了**。

Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。
```
getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
```
一共有三个Promise对象：一个由getJSON产生，两个由then产生。它们之中任何一个抛出的错误，都会被最后一个catch捕获。

跟传统的try/catch代码块不同的是，如果没有使用catch方法指定错误处理的回调函数，Promise对象抛出的错误不会传递到外层代码，即不会有任何反应。
```
var someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
  });
};

someAsyncThing().then(function() {
  console.log('everything is great');
});
```
someAsyncThing函数产生的 Promise 对象会报错，但是由于没有指定catch方法，这个错误不会被捕获，也不会传递到外层代码。正常情况下，运行后不会有任何输出，但是浏览器此时会打印出错误“ReferenceError: x is not defined”，不过不会终止脚本执行，如果这个脚本放在服务器执行，退出码就是0（即表示执行成功）。


不要在then方法里面定义Reject状态的回调函数（即then的第二个参数），总是使用catch方法。
```
// bad
promise
  .then(function(data) {
    // success
  }, function(err) {
    // error
  });

// good
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```
第二种写法要好于第一种写法，理由是第二种写法可以捕获前面then方法执行中的错误，也更接近同步的写法（try/catch）。因此，建议总是使用catch方法，而不使用then方法的第二个参数。

**Node** 有一个unhandledRejection事件，专门监听未捕获的reject错误。
```
process.on('unhandledRejection', function (err, p) {
  console.error(err.stack);
  // p catch方法返回的还是一个Promise对象，因此后面还可以接着调用then方法。
});
```
catch方法返回的还是一个 Promise 对象，因此后面还可以接着调用then方法。
```
var someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
  });
};

someAsyncThing()
.catch(function(error) {
  console.log('oh no', error);
})
.then(function() {
  console.log('carry on');
});
// oh no [ReferenceError: x is not defined]
// carry on
```
运行完catch方法指定的回调函数，会接着运行后面那个then方法指定的回调函数。如果没有报错，则会跳过catch方法。
```
Promise.resolve()
.catch(function(error) {
  console.log('oh no', error);
})
.then(function() {
  console.log('carry on');
});
// carry on
```
上面的代码因为没有报错，跳过了catch方法，直接执行后面的then方法。此时，要是then方法里面报错，就与前面的catch无关了。

catch方法之中，还能再抛出错误。
```
someAsyncThing().then(function() {
  return someOtherAsyncThing();
}).catch(function(error) {
  console.log('oh no', error);
  // 下面一行会报错，因为y没有声明
  y + 2;
}).catch(function(error) {
  console.log('carry on', error);
});
// oh no [ReferenceError: x is not defined]
// carry on [ReferenceError: y is not defined]
```
### Promise.all()
Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例
```
var p = Promise.all([p1, p2, p3]);
```
Promise.all方法接受一个数组作为参数，p1、p2、p3都是 Promise 实例，如果不是，就会先调用下面讲到的Promise.resolve方法，将参数转为 Promise 实例，再进一步处理。（Promise.all方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。

**p的状态由p1、p2、p3决定，分成两种情况。**

 - 只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1 、p2、p3的返回值组成一个数组，传递给p的回调函数。
 - 只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。
 
```
// 生成一个Promise对象的数组
var promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```
promises是包含6个 Promise实例的数组，只有这6个实例的状态都变成fulfilled，或者其中有一个变为rejected，才会调用Promise.all方法后面的回调函数。

如果作为参数的 Promise实例，自己定义了catch方法，那么它一旦被rejected，并不会触发Promise.all()的catch方法。
```
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result)
.catch(e => e);

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
})
.then(result => result)
.catch(e => e);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// ["hello", Error: 报错了]
```
p1会resolved，p2首先会rejected，但是p2有自己的catch方法，该方法返回的是一个新的 Promise 实例，p2指向的实际上是这个实例。该实例执行完catch方法后，也会变成resolved，导致Promise.all()方法参数里面的两个实例都会resolved，因此会调用then方法指定的回调函数，而不会调用catch方法指定的回调函数。

```
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result);

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
})
.then(result => result);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// Error: 报错了
```
如果p2没有自己的catch方法，就会调用Promise.all()的catch方法。

### Promise.race()
Promise.race方法同样是将多个Promise实例，包装成一个新的Promise实例.

```
var p = Promise.race([p1, p2, p3]);
```

### Promise.resolve()
有时需要将现有对象转为Promise对象，Promise.resolve方法就起到这个作用。
```
var jsPromise = Promise.resolve($.ajax('/whatever.json'));
```
### Promise.reject()
Promise.reject(reason)方法也会返回一个新的Promise实例，该实例的状态为rejected。

### 两个有用的附加方法
- done()
- finally()

### 应用
- 加载图片
- Generator函数与Promise的结合

### Promise.try() 
Promise.try就是模拟try代码块，就像promise.catch模拟的是catch代码块。


### 总结：
一开始没搞懂promise什么，中文意思是“承诺”，黑人？？？看了很多它的api，心中一郁，为什么我要把我的异步变成一个promise对象。看到了generator章节，豁然开朗。
>* Promise 对象就是为了解决这个问题而提出的。它不是新的语法功能，而是一种新的写法，允许将回调函数的嵌套，改成链式调用。

其实只是一种新的写法，告别原来的嵌套回调，写法简化像是一个‘同步’操作。而promise也有缺点

>* Promise 的最大问题是代码冗余，原来的任务被 Promise 包装了一下，不管什么操作，一眼看去都是一堆then，原来的语义变得很不清楚。

接下来就可以转站**Generator**章节了～