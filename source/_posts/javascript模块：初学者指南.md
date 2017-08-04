---
title: javascript模块：初学者指南
date: 2017-08-04 14:10:00
categories: Model
---
 

>* 本文是一篇译文，原文是在 Preethi Kasireddy's[JavaScript Modules: A Beginner's Guide][1] (翻墙更快哟～)，感谢大神的分享！比心

----------
如果你是一个新的JavaScript，诸如“模块与模块的装载机，“Webpack vs. Browserify”和“AMD与CommonJS可以迅速成为压倒性的。

JavaScript模块系统可能是令人生畏的，但是理解它对于Web开发人员来说是至关重要的。

在这篇文章中，我会将这些流行语你在平原英语（和一些示例代码）。我希望你觉得有用！

注意：为了简单起见，这将分为两部分：第1部分将深入解释什么是模块以及为什么我们使用它们。第2部分（下星期发布）将介绍捆绑模块的含义和不同的方法。


----------

<!--more-->
### 第1部分：有人能解释一下什么是模块吗？
优秀的作者将他们的书分成章节和章节；优秀的程序员将他们的程序划分成模块。

就像书的章节一样，模块只是一堆单词（或者代码，可能是这样）。

然而，好的模块是高度独立的，具有不同的功能，允许它们在必要时进行清理、删除或添加，而不会破坏整个系统

### 为什么使用模块
有很多的好处，在一个庞大的主张使用模块，相互依存的基本代码。在我看来，最重要的是：

1）可维护性：根据定义，模块是自包含的。一个设计良好的模块旨在减少对代码库的部分尽可能的依赖关系，以便它可以成长和提高独立。当模块与其他代码分离时，更新单个模块要容易得多。

回到我们的书的例子，如果你想更新你书中的一章，如果一章的小改动需要你修改其他章节，那将是一场噩梦。相反，你想写的每一章，这样的改进可以不影响其他章节。

2）命名空间：在JavaScript中，在顶层的功能范围的变量都是全局的（意思是，每个人都可以访问它们）。因此，存在“命名空间污染”，即完全不相关的代码共享全局变量。
在不相关的代码之间共享全局变量是开发中的一大障碍。
正如我们在后面的文章中看到的，模块允许我们为变量创建私有空间来避免命名空间污染。

3）可重用性：让我们诚实地说：我们已经复制了我们以前在新项目中编写过的代码。例如，假设您复制了从以前的项目到当前项目编写的一些实用工具方法。

这一切都很好，但是如果你找到一个更好的方法来写代码的一部分，你就必须回去，记住在你写的地方更新它。

这显然是浪费时间。不容易得多，如果有 - 等待 - 模块，我们可以一遍又一遍地重复一遍吗？

### 你怎么组合模块？
有很多方法可以把模块集成到你的程序中。让我们看看几中方法：

#### 组件模式

模块化是用来模仿类的概念（因为JavaScript没有原生支持类），我们可以存储公共和私有方法和变量在一个单一的对象 - 相似类如何使用java、python等编程语言。这允许我们为我们希望公开的方法创建面向公共的API，同时还在闭包范围内封装私有变量和方法。

有几种方法可以完成模块模式。在第一个示例中，我将使用匿名闭包。这将帮助我们通过将所有代码放入匿名函数来实现我们的目标。（记住：在JavaScript中，函数是创建新范围的惟一方法）。

#### 示例1：匿名闭包
```
(function () {
  // We keep these variables private inside this closure scope
  
  var myGrades = [93, 95, 88, 0, 55, 91];
  
  var average = function() {
    var total = myGrades.reduce(function(accumulator, item) {
      return accumulator + item}, 0);
    
      return 'Your average grade is ' + total / myGrades.length + '.';
  }

  var failing = function(){
    var failingGrades = myGrades.filter(function(item) {
      return item < 70;});
      
    return 'You failed ' + failingGrades.length + ' times.';
  }

  console.log(failing());

}());

// ‘You failed 2 times.’

```
有了这个结构，我们的匿名函数有自己的作用域或“闭包”，然后我们立即评估它。这允许我们从父（全局）命名空间中隐藏变量。
是一个好方法，你可以使用局部变量在函数覆盖现有的全局变量，但仍然可以访问全局变量，像这样：
```
var global = 'Hello, I am a global variable :)';

(function () {
  // We keep these variables private inside this closure scope
  
  var myGrades = [93, 95, 88, 0, 55, 91];
  
  var average = function() {
    var total = myGrades.reduce(function(accumulator, item) {
      return accumulator + item}, 0);
    
    return 'Your average grade is ' + total / myGrades.length + '.';
  }

  var failing = function(){
    var failingGrades = myGrades.filter(function(item) {
      return item < 70;});
      
    return 'You failed ' + failingGrades.length + ' times.';
  }

  console.log(failing());
  console.log(global);
}());

// 'You failed 2 times.'
// 'Hello, I am a global variable :)'
```
值得注意的是，在匿名函数的圆括号是必需的，因为陈述与关键字功能开始一直被认为是函数声明（记住，你不能在JavaScript中，匿名函数声明）因此，周围的括号中创建一个函数表达式。

示例2：全局引入
像jQuery这样的库使用的另一种流行方法是全局导入。它类似于匿名关闭我们刚才看到的，除非我们通过全局变量作为参数：

```
(function (globalVariable) {

  // Keep this variables private inside this closure scope
  var privateFunction = function() {
    console.log('Shhhh, this is private!');
  }

  // Expose the below methods via the globalVariable interface while
  // hiding the implementation of the method within the 
  // function() block

  globalVariable.each = function(collection, iterator) {
    if (Array.isArray(collection)) {
      for (var i = 0; i < collection.length; i++) {
        iterator(collection[i], i, collection);
      }
    } else {
      for (var key in collection) {
        iterator(collection[key], key, collection);
      }
    }
  };

  globalVariable.filter = function(collection, test) {
    var filtered = [];
    globalVariable.each(collection, function(item) {
      if (test(item)) {
        filtered.push(item);
      }
    });
    return filtered;
  };

  globalVariable.map = function(collection, iterator) {
    var mapped = [];
    globalUtils.each(collection, function(value, key, collection) {
      mapped.push(iterator(value));
    });
    return mapped;
  };

  globalVariable.reduce = function(collection, iterator, accumulator) {
    var startingValueMissing = accumulator === undefined;

    globalVariable.each(collection, function(item) {
      if(startingValueMissing) {
        accumulator = item;
        startingValueMissing = false;
      } else {
        accumulator = iterator(accumulator, item);
      }
    });

    return accumulator;

  };

 }(globalVariable));
  
```
在这个例子中是唯一的变量是全局。这种方法在匿名闭包上的好处是，您可以预先声明全局变量，使读代码的人更加清楚。
示例3：对象接口
另一种方法是使用自包含的对象接口创建模块，如下所示：
```
var myGradesCalculate = (function () {
    
  // Keep this variable private inside this closure scope
  var myGrades = [93, 95, 88, 0, 55, 91];

  // Expose these functions via an interface while hiding
  // the implementation of the module within the function() block

  return {
    average: function() {
      var total = myGrades.reduce(function(accumulator, item) {
        return accumulator + item;
        }, 0);
        
      return'Your average grade is ' + total / myGrades.length + '.';
    },

    failing: function() {
      var failingGrades = myGrades.filter(function(item) {
          return item < 70;
        });

      return 'You failed ' + failingGrades.length + ' times.';
    }
  }
})();

myGradesCalculate.failing(); // 'You failed 2 times.' 
myGradesCalculate.average(); // 'Your average grade is 70.33333333333333.'
```

你可以看到，这种方法让我们决定什么变量/方法我们想保持私人（例如成绩）和哪些变量/方法我们要揭露被放在return语句（如平均与失败）。

例4：展示模块模式
这与上面的方法非常相似，只是它确保所有方法和变量都保持私有，直到显式暴露为止：
```
var myGradesCalculate = (function () {
    
  // Keep this variable private inside this closure scope
  var myGrades = [93, 95, 88, 0, 55, 91];
  
  var average = function() {
    var total = myGrades.reduce(function(accumulator, item) {
      return accumulator + item;
      }, 0);
      
    return'Your average grade is ' + total / myGrades.length + '.';
  };

  var failing = function() {
    var failingGrades = myGrades.filter(function(item) {
        return item < 70;
      });

    return 'You failed ' + failingGrades.length + ' times.';
  };

  // Explicitly reveal public pointers to the private functions 
  // that we want to reveal publicly

  return {
    average: average,
    failing: failing
  }
})();

myGradesCalculate.failing(); // 'You failed 2 times.' 
myGradesCalculate.average(); // 'Your average grade is 70.33333333333333.'
```
这似乎是一个很大的问题，但它只是模块模式的冰山一角。这里有一些我发现我自己探索有用的资源：


----------


## CommonJS and AMD

上述方法有一个共同点：使用单个全局变量将其代码封装到函数中，从而使用闭包范围为自己创建一个私有命名空间。
虽然每一种方法都以自己的方式是有效的，他们都有自己的缺点
。
首先，作为开发人员，您需要知道正确的依赖项顺序来加载文件。

例如，假设您在项目中使用了Backbone，因此在文件中包含了Backbone源代码的脚本标记。

然而，由于Backbone对Underscore.js硬的依赖，为Backbone的文件的脚本标签不能被放置在Underscore.js文件。

作为开发人员，管理依赖性和正确地处理这些事情有时会让人头痛。

另一个缺点是它们仍然可以导致命名空间冲突。例如，如果你的两个模块具有相同的名字吗？或者如果你有一个模块的两个版本，你需要吗？

所以您可能会疑惑：我们是否可以设计一种方法，在不涉及全局范围的情况下请求模块的接口？

幸运的是，答案是肯定的。

有两种实现方法：CommonJS和AMD。

### CommonJS
CommonJS是志愿者工作组，设计并实现了JavaScript API声明模块。

CommonJS的模块基本上是一个可重用块JavaScript出口特定物体，使它们可用于其他模块需要在他们的程序。如果你的程序在Node.js，你会被这种格式很熟悉。

随着CommonJS，每个JavaScript文件存储模块，其独特的模块上下文（就像它包裹在一个封闭的）。在这个范围内，我们使用module.exports对象暴露的模块，并要求进口。
当你定义一个CommonJS模块，它可能看起来像这样：
```
function myModule() {
  this.hello = function() {
    return 'hello!';
  }

  this.goodbye = function() {
    return 'goodbye!';
  }
}

module.exports = myModule;
```
我们使用特殊的对象模块，把我们的功能引用到module.exports。这让CommonJS模块系统知道我们想暴露，其他文件可以使用它。
当有人想用他们可以要求mymodule，在他们的文件，就像这样：
```
var myModule = require('myModule');

var myModuleInstance = new myModule();
myModuleInstance.hello(); // 'hello!'
myModuleInstance.goodbye(); // 'goodbye!'
```
这种方法对我们以前讨论过的模块模式有两个明显的好处：

1. 避免全局命名空间污染
2. 让我们的关系明确

此外，语法是非常紧凑的，我的爱。

另一个要注意的是，以第一种方法和CommonJS的服务器同步加载模块。这很重要，因为如果我们需要其他三个模块，它会一个一个地加载它们。

现在，这在服务器上很好，但不幸的是，在为浏览器编写JavaScript时更难使用它。
可以说，从Web上读取模块要比从磁盘读取数据要长得多。只要加载一个模块的脚本运行，它就阻止浏览器运行任何其他东西，直到它完成加载。

它的行为是这样的，因为JavaScript线程一直停止，直到代码被加载为止。（我将在第2部分讨论模块捆绑时如何处理这个问题。现在，这就是我们需要知道的）。

### AMD
CommonJS是好的，但如果我们要加载的模块异步？答案是称为异步模块定义，或AMD的短。
加载模块采用AMD看起来像这样：
```
define(['myModule', 'myOtherModule'], function(myModule, myOtherModule) {
  console.log(myModule.hello());
});
```
这里所发生的是，定义函数作为第一个参数作为每个模块依赖关系的数组。

这些依赖项在后台加载（以非阻塞方式），并且一旦加载定义调用，就会调用回调函数。

其次，回调函数，作为参数，被加载 - 在我们的情况下，依赖，和myothermodule mymodule - 允许函数使用这些依赖关系。

最后，依赖项本身也必须使用定义关键字来定义。
例如，mymodule可能看起来像这样：
```
define([], function() {

  return {
    hello: function() {
      console.log('hello');
    },
    goodbye: function() {
      console.log('goodbye');
    }
  };
});
```
所以，不像CommonJS，AMD以浏览器第一法与异步行为的完成工作。（注意，有很多人强烈认为在开始运行代码时动态地加载文件是不好的，我们将在下一节的模块构建中进行更多的探索）。

除了异步，AMD的另一个好处是，你的模块可以是对象，函数，构造函数的字符串，JSON和许多其他类型的对象，而CommonJS模块只支持对象。

也就是说，AMD不兼容的IO，文件系统，和其他面向服务器的功能可通过CommonJS，和功能语法有点冗长的包装相比，一个简单的要求的声明

### UMD

项目需要你支持AMD和CommonJS的功能，还有另外一种格式：通用模块定义（UMD）。
UMD本质上创建了一种使用两个，同时还支持全局变量定义。作为一个结果，UMD模块可以工作在客户端和服务器。
这里是一个快速的味道如何UMD我行我素：
```
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
      // AMD
    define(['myModule', 'myOtherModule'], factory);
  } else if (typeof exports === 'object') {
      // CommonJS
    module.exports = factory(require('myModule'), require('myOtherModule'));
  } else {
    // Browser globals (Note: root is window)
    root.returnExports = factory(root.myModule, root.myOtherModule);
  }
}(this, function (myModule, myOtherModule) {
  // Methods
  function notHelloOrGoodbye(){}; // A private method
  function hello(){}; // A public method because it's returned (see below)
  function goodbye(){}; // A public method because it's returned (see below)

  // Exposed public methods
  return {
      hello: hello,
      goodbye: goodbye
  }
}));
```
更多的UMD格式的例子，看看这个启蒙回购在GitHub上。

### Native JS

唷！你还在吗？我没有失去你在这里的森林？好的！因为我们有一个* *型模块定义之前，我们做了。
你也许注意到，以上的模块没有原生JavaScript。相反，我们创造了通过使用模块化仿真模块的系统方式，CommonJS或AMD。
幸运的是，聪明的人在tc39（标准体定义ECMAScript的语法和语义）已经推出了内置模块，ECMAScript 6（6）。
6提供了各种可能性，导入和导出模块，有人做了大量的工作，解释 - 这里有一些资源：

有什么了不起的ES6模块相对于CommonJS或AMD是如何管理提供了两全其美：紧凑和声明性语法和异步加载，加如循环依赖更好的保障效益。

可能是我最喜欢的特点是进口的活ES6模块的出口的只读视图。（相比之下，进口份CommonJS，出口因此不活）。

这里的一个例子是如何工作的：
```
// lib/counter.js

var counter = 1;

function increment() {
  counter++;
}

function decrement() {
  counter--;
}

module.exports = {
  counter: counter,
  increment: increment,
  decrement: decrement
};


// src/main.js

var counter = require('../../lib/counter');

counter.increment();
console.log(counter.counter); // 1
```
在这个例子中，我们基本上复制两个模块：一个是导出的模块，一个是我们需要的模块。
此外，在main.js副本现在已经从原来的模块断开。这就是为什么即使我们增加我们的计数仍然返回1 - 因为计数器变量，我们进口的是断开连接的副本的计数器变量的模块。
所以，递增计数器将在模块的增加，但不会增加你的复制版。修改计数器变量的复制版本的唯一方法是手动执行：
```
counter.counter++;
console.log(counter.counter); // 2
```
另一方面，6创建活的只读视图我们导入模块：
```
// lib/counter.js
export let counter = 1;

export function increment() {
  counter++;
}

export function decrement() {
  counter--;
}


// src/main.js
import * as counter from '../../counter';

console.log(counter.counter); // 1
counter.increment();
console.log(counter.counter); // 2
```
很酷的东西，嗯？我发现活只读视图真正吸引人的是它们如何让你的模块分解成更小的部分而不会失去功能。
然后你可以转身和合并起来，没问题。它只是“工作”。

## 展望未来：集成模块

真的!时间到哪里去了？这是一个疯狂的旅程，但我真心希望它能让您更好地理解JavaScript中的模块。
在下一节中，我将介绍模块捆绑，包括核心主题，包括：

- 为什么我们捆绑模块
- 捆绑销售的不同方法
- ECMAScript的模块加载器的API
- …更多。：）

注：为了简便起见，我跳过了一些细节（比如：循环依赖）在后。如果我遗漏了一些重要的和/或令人着迷的东西，请在评论中告诉我！


  [1]: https://www.infoq.com/articles/state-of-javascript-2016?utm_source=articles_about_reactive-programming&utm_medium=link&utm_campaign=reactive-programming#