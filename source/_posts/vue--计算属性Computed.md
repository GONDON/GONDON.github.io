---
title: vue--计算属性Computed
date: 2018-06-21 11:39:34
categories: vue
---

### 前言
计算属性在vue官方文档里简单的介绍了一下，可自定义一个函数去作用属性，说是提供的函数将作用属性的getter函数。其实，底层的实现原理并没有那么简单，用到了`闭包`,`Object.defineProperty`,`观察者模式`

### 1.首先是要了解Js的新特性。Object.defineProperty这个属性的getting和setting方法。
```
    var obj = {}
    Object.defineProperty(obj,'a',{
        get:function(){
            console.log("getting the a")
            return 1;
        }
    })
    
    console.log("this a is:",obj.a)
```
### 2.基于vue的观察者模式
```
    function defineReactive (obj,key,val){
        Object.defineProperty(obj,key,{
            get:function(){
                return val;
            },
            set:function(newValue){
                val = newValue
            }
        })
    }
//这里的做法是？就是把Object.defineProperty方法封装一遍？还是在做闭包
// create an object
    var person ={}
// 定义age和country的属性
defineReactive (person, 'age', 25);
defineReactive (person, 'country', 'Brazil');

// 可以拿到proson.age去做比较
if (person.age < 18) {
  return 'minor';
}
else {
  return 'adult';
}

// Set a value as well.
person.country = 'Russia';

```
有趣的是，当你在set去修改value的值的时候，你会发现事实上value`25`和`Brazil`是闭包变量val里面的值。person.country访问到的值，person.country并没有存在一个实际的value，反而是在getterfunction里闭包包含的那个value值。

### 我们去定一个计算属性的方法
```
defineComputed(
    person,//一个要计算的对象
    'status', //要计算的属性名称
    function(){//定义computed函数的逻辑
        consoleo.log("status getter called") 
        if (person.age < 18) {
          return 'minor';
        }
        else {
          return 'adult';
        }
    },
    function(){ //当computed执行后value发生更新时候执行
        console.log("status has change to ",newValue)
    }
)
// We can use the computed property like a regular property
console.log ("The person's status is: ", person.status);
```
现在我们对defineComputed做一些变形，他将支持计算函数，去调uodateCallBack。
```
    function defineComputed (obj, key, computeFunc, updateCallback) {
      Object.defineProperty (obj, key, {
        get: function () {
          // call the compute function and return the value
          return computeFunc ();
        },
        set: function () {
          // don't do anything. can't set computed funcs
        }
      })
    }
```
现在有一些问题：
1.每次属性被访问的时候这个计算属性都会执行。
2.我们不知道什么时候发生的更新
```
// 我们喜欢像下面一样去打印出日志
person.age = 17;
// console: status has changed to: minor

person.age = 22;
// console: status has changed to: adult

```

### 4.添加一个依赖追踪器
生命一个全局对象叫做`Dep`
```
    var Dep = {
        target:null
    }
    
```
这个依赖追踪器，将会去去边defineComputed函数的关键地方
```
    function defineComputed (obj,key,computeFunc,updateCallback){
        var onDependencyUpdate = function(){
            //todo
        }
        
        Object.defineProperty(obj,key,{
            get:function(){
                //set dependency target as this function
                Dep.target = onDependencyUpdated
                var value  = computedFunc()
                Dep.target = null
            },
            set:function(){
                //dont do anything.can`t set computed funcs
            }
        })
    }
```

然后我们返回去去看我们是怎么修改ractive peoperty的
```
    function defineReactive (obj,key,val){
        //所有计算的属性都依赖在这
        var deps = [];
        Object.defineProperty(obj,key,{
            get:function(){
                //检查如果有计算的属性被调用
                //这是一个getter，也回去检查这时候已经被依赖。
                if (Dep.target && ) {
                    // add the dependency 添加依赖
                    deps.push (target);
                 }
                return val;
            },
            set:function(newValue){
                val = newValue
                //通知所有依赖的计算属性
                deps.forEach((changeFunction)=>{
                    //请求重新计算和更新
                    changeFunction ();
                })
            }
        })
    }
```
我们还可以更新一个onDepdencyUpdated函数，然后去粗发UpdateCallBack。
```
var onDependencyUpdate = function(){
    //再一次去计算value
    var value = computeFunc();
    updateCallBack();
}
```

### 5.把所有的集合在一起。
我们将再一次去看看我们定义的自己算属性`person.status`
```
    person.age = 22;
    defineComputed(
        person,
        'status',
        function(){
            //计算函数
            if(person.age >18){
                return 'adult'
            }
        },
        function(newValue){
            console.log("status has changed to ",newValue)
        }
    )
    console.log ("Status is ", person.status);
```

步骤一：
get()在person.status的属性时被调用，然后粗发了sets里的dep.target去执行了回调![compprop1.png-26.4kB][1]


步骤二：
get()在person.status被调用了计算方法，然后调用了get()在person.age的属性上，因为这个回掉需要这个value。![compprop2.png-35.8kB][2]

步骤三：
person.age的get()与Dep一起去检查目标是否可用，然后将其存储为依赖。

![compprop3.png-31.8kB][3]

步骤四：
这个计算函数拿到了新的值并且返回，但是现在`person.age`知道去通知`person.status`当其`value`更新

  [1]: http://static.zybuluo.com/juanmao/avztxkttgr7wgkkpr6f0r2fr/compprop1.png
  [2]: http://static.zybuluo.com/juanmao/bu2l5tnzr39ca899gkjbd81y/compprop2.png
  [3]: http://static.zybuluo.com/juanmao/5ttuso87652a5r0tpda1orro/compprop3.png
  
  参考文献：

> https://skyronic.com/blog/vuejs-internals-computed-properties




