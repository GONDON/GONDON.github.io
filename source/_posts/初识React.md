---
title: 初识React
date: 2017-07-23 17:38:56
tags: React
---

#### 1.don元素
在react中，到处都是可以复用的元素，这些元素并不是真实的实例，它只是让react告诉开发者想要在屏幕上显示什么。我们无法通过方法去调用这些元素，他们只是不可变的描述对象

#### 2.组建元素
这也是 React 的核心思想之一。因为有公共的表达方法，我们就可以让元素们彼此嵌套或混合。这些层层封装的组件元素，就是所谓的 React 组件，最终我们可以用递归渲染的方式构建出完全的 DOM 元素树。

JSX 将 HTML语法直接加入到JavaScript代码中，再通过翻译器转换到纯 JavaScript后由浏览器执行。在实际开发中，JSX在产品打包阶段都已经编译成纯JavaScript，不会带来任何副作用，反而会让代码更加直观并易于维护。尽管JSX是第三方标准，但这套标准适用于任何一套框架。

```
//JSON-dom
const DeleteAccount = () => 
({  type: 'div',  
    props: {    
        children: [{   
            type: 'p',
                props: {  children: 'Are you sure?',
                },  
            }, {
                type: DangerButton,
                props: {
                    children: 'Confirm',
                    },
                    }, {
                    type: Button,
                    props: {
                        color: 'blue',
                        children: 'Cancel',
                        },
                    }],
                }
            })
//react-dom
var DeleteAccount = function DeleteAccount() {
    return React.createElement(
        'div',    null,
        React.createElement(
            'p',
            null,
            'Are you sure?'
            ),
        React.createElement(
            DangerButton,
            null,
            'Confirm'
            ),
        React.createElement(
            Button,
            { color: 'blue' },
            'Cancel'
            )
        );
```
**其结构与一直在讲的 JSON 的结构是一致的**

JSX 并不是强制选项，我们可以像上述代码那样直接书写而无须编译，但这实在是极其槽糕的编程体验。JSX的出现为我们省去了这个烦琐过程，使用 JSX写法的代码更易于阅读与开发。事实上，JSX并不需要花精力学习。只要你熟悉HTML标签，大多数功能就都可以直接使用了。

### JSX 基本语法 

#### 1.比较xml
使用类XML语法的好处是标签可以任意嵌套，我们可以像 HTML一样清晰地看到 DOM 树状结构及其属性。

>* JSX
>* 定义标签时，只允许被一个标签包裹。
>* 标签一定要闭合。

#### 2.元素类型

>* dom元素，小写开头。
>* 组建元素，大写开头。
>* 可以通过命名空间的方式使用组件元素，以解决组件相同名称冲突的问题，或则对一组组件进行归类。

- 注释
 注意：在一个组件的子元素位置使用注释要用{}包起来。
```
const App = (
    <Nav>
        {/*节点注释*/}
        <Person
            /*
                多行注释
            */
            name={window.isLoggedIn ? window.name : ''}
        />
    </Nav>
)

```
- 条件注释
```
<!--[if IE]>   <p>Work in IE browser</p> <![endif]-->
```
可以通过使用javascript哦安段浏览器版本俩替换：
```
{  (!!window.ActiveXObject || 'ActiveXObject' in window) ?   <p>Work in IE browser</p> : '' } 
```
- DOCTYPE
一般会在使用React作为服务端渲染是用到。在html中，改元素是没有闭合的，也就是说我门无法直接渲染它。常见的做法是构造一个保存html的变量，将doctype与整个html标签渲染后的结果串起来。

#### 3. 元素属性
- class,for
>* class属性改完className;
>* for属性改为htmlFor;

值得注意的是，在写自定义属性的时候，都由标准写法改为小驼峰写法。

- Boolean属性
省略Boolean属性值会导致jsx认为bool值为了true。要传false时，必须使用属性表达式。这常用于表单元素，比如disabled、required、checked 和 readOnly 等。

```
例如，<Checkbox checked={true} /> 可以简写为 <Checkbox checked />，反之 <Checkbox checked={false} /> 就可以省略 checked 属性
```
- 展开属性
因为react不能帮你检查属性类型（propTypes），这样即使组件的属性类型有错误，也不能得到清晰的错误提示。
在这里就可以用es6里面的rest/spread的特性。
```
const data = {name:'foo',value:''bar};
const Component = <Component {...data}/>
```

- 自定义HTML属性
如果在jsx中直接往dom元素中传入自定义属性，React是不会渲染的
```
<div d="xxx">content</div>
```
如果要使用 HTML 自定义属性，要使用 data- 前缀，这与 HTML 标准也是一致的：
```
<div data-attr="xxx">content</div>
```
然而，在自定义标签中任意的属性都是被支持的：
```
<x-my-component custom-attr="foo" />
```
以 aria- 开头的网络无障碍属性同样可以正常使用： 
```
<div aria-hidden={true}></div>
```
不论组件是用什么方法来写，我们都需要知道，组件的最终目的是输出虚拟元素，也就是需要被渲染到界面的结构。其核心渲染方法，或称为组件输出方法，就是 render 方法。它是 React 组件生命周期的一部分，也是最核心的函数之一。


#### 4. JavaScript 属性表达式
属性值要使用表达式，只要用 {} 替换 "" 即可：
```
// 输入（JSX）：
const person = <Person name={window.isLoggedIn ? window.name : ''} />
```
子组件也可以作为表达式使用：
```
const content = <Container>{window.isLoggedIn ? <Nav /> : <Login />}</Container>;
```
#### 5. HTML 转义

此外，React 提供了 dangerouslySetInnerHTML 属性。正如其名，它的作用就是避免 React 转义字符，在确定必要的情况下可以使用它： 
```
<div dangerouslySetInnerHTML={{__html: 'cc &copy; 2015'}} />
```
### React 组件 
- 1用es6来写一个tab页
>*  基本的封装性。
>*  简单的生命周期呈现。
>*  明确的数据流动。

- web Compernents
包含：

![web Compernents](http://static.zybuluo.com/juanmao/4qgs52152u7x1nwj48ojz7ae/image.png)

----------


### React 组件的构建 
![React生命周期](http://static.zybuluo.com/juanmao/n5bp1syvqmvfbevzdgywp2f2/image.png)

- 1. React 与 Web Components
 >* React 自定义元素是库自己构建的，与 Web Components 规范并不通用
>* React 渲染过程包含了模板的概念，即 1.2 节所讲的 JSX
>*  React 组件的实现均在方法与类中，因此可以做到相互隔离，但不包括样式;
>* React 引用方式遵循 ES6 module 标准。

#### 2. React 组件的构建方法
-  React.createClass
最初始的一种构建方式，也是兼容性最好的写法。
```
const Button = React.creatClass({
    getDefaultProps(){
        return {
            color:'blue',
            text:'Confirm'
        };
    },
    render(){
        const {color,text} = this.props;
        return (
            <button className ={`btn btn-${color}`}>
                <em>{text}</em>
            </button>
        );
    }
});
```
从表象上看， React.createClass 方法就是构建一个组件对象。 

-  ES6 classes
```
import React,{Component} form 'react';

class Button extend Component{
    constructor(props){
        super(props);
    }
    static defaultProps = {
        color:'blue',
        text:'Comfirm'
    };
    render(){
        const {color,text} = this.props;
        
        return(
            <button className ={`btn btn-${color}`}>
                <em>{text}</em>
            </button>
        );
    }
}
```
- 无状态函数
```
function Button({ color = 'blue', text = 'Confirm' }) {      return (
    <button className={`btn btn-${color}`}>                    <em>{text}</em>
    </button>
 );
} 
```
无状态组件只传入props和context两个参数；也就是说，它不存在state，也没有生命周期方法，**组件本身即上面两种React组件构建方法中的 render 方法**。不过，像propTypes和defaultProps还是可以通过向方法设置静态属性来实现的。

在适合的情况下，我们都应该且必须使用无状态组件。无状态组件不像上述两种方法在调用时会创建新实例，它创建时始终保持了一个实例，避免了不必要的检查和内存分配，做到了内部优化。

#### React 数据流 






































