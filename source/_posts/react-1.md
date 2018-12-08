---
title: React
subtitle: React
date: 2018-9-18
categories: React
tags:
    - react
---
## React 基础篇
### 什么是React？
React是一个用于构建用户界面的JavaScript库，核心专注于视图，目的实现组件化开发。

### 组件化概念
我们可以很直观的将一个复杂的页面分割成若干个独立组件，每个组件包含自己的逻辑和样式，再将这些独立组件组合完成一个复杂的页面。这样既减少了逻辑复杂度，又实现了代码的重用

+ 可组合：一个组件可以和其他组件一起使用或者可以直接嵌套在另一个组件内部
+ 可重用：每个组件都具有独立功能，他可以被使用在多个场景中
+ 可维护：每个小的组件仅仅包含自身的逻辑，更容易被理解和维护

### 跑通React开发环境
```javascript
npm install create-react-app -g
create-react-app <项目名>
cd <项目名>
npm start
```
默认会自动安装React，React由两部分组成，分别是：
+ react.js 是React的核心库
+ react-dom.js 是提供与DOM相关的功能，会在window下增加ReactDOM属性，内部比较重要的方法是render，将react元素或者react组件插入到页面中。

### 简介JSX
是一种JS和HTML混合的语法，将组织的结构，数据甚至样式都聚合在一起定义组件，会编译成普通的JavaScript。需要注意的是JSX并不是HTML，在JSX中属性不能包含关键字，像class需要写成className，for需要写成htmlFor，并且属性名需要采用驼峰命名！

### createElement
JSX其实只是一种语法糖，最终会通过babel转译成createElement语法，以下代码等价
```javascript
ReactDOM.render(<div>高<span>佳睿</span></div>);
ReactDOM.render(React.createElement("div",null,"高",React.createElement("span",null,"佳睿")));
```
我们一般使用React.createElement来创建一个虚拟dom元素。

### react元素/JSX元素
```javascript
function ReactElement(type,props) {
    this.type = type;
    this.props = props;
};
let React = {
    createElement(type,props = {},...childrens){
        childrens.length === 1 ? childrens = childrens[0]: viod 0
        return new ReactElement(type,{...props,children:childrens})
    }
};
```
ReactElement就是虚拟dom的概念，具有一个type属性代表当前的节点类型，还有节点的属性props

### 模拟render实现
```javascript
let render = (eleObj,container)=>{
    // 先取出第一层 进行创建真实dom
    let {type,props} = eleObj;
    let elementNode = document.createElement(type); // 创建第一个元素
    for(let attr in props){ // 循环所有属性
        if(attr === 'children'){ // 如果是children表示有嵌套关系
            if(typeof props[attr] == 'object'){ // 看是否是只有一个文本节点
                props[attr].forEach(item=>{ // 多个的话循环判断 如果是对象再次调用render方法
                    if(typeof item === 'object'){
                        render(item,elementNode)
                    }else{ //是文本节点 直接创建即可
                        elementNode.appendChild(document.createTextNode(item));
                    }
                })
            }else{ // 只有一个文本节点直接创建即可
                elementNode.appendChild(document.createTextNode(props[attr]));
            }
        }else if(attr === 'className'){ // 是不是class属性 class 属性特殊处理
            elementNode.setAttribute('class',props[attr]);
        }else{
            elementNode.setAttribute(attr,props[attr]);
        }
    }
    container.appendChild(elementNode)
};
```






