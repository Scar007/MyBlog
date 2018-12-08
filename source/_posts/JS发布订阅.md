---
title: JS中的发布订阅模式
subtitle: pubsub
date: 2016-4-6
categories: JS
tags:
    - 发布订阅
---
## 一. 你是如何理解发布订阅模式的
+ JS中的设计模式:
    + 单例模式:处理业务逻辑   
    + 构造原型模式:封装类库,组件,框架,插件等
        - 类库:jQuery
            只是提供了一些常用的方法,可以应用到任何的项目中,不具备业务性
        - 组件:bootstrap
            提供了很多通用的组件(HTML/CSS/JS都是别人规定好的),我们只需要按照要求使用,就可以直接的达到效果
        - 插件: swiper/iscroll
            针对于一个具体业务的封装,例如选项卡插件或者轮播图插件
        - 框架:React/Vue
            具备一定编程思想的(MVC/MVVM)叫做框架
    + 发布订阅模式:处理一些具体需求的
    + promise模式:处理一些具体需求的
    
+ 发布订阅模式
    + 发布一个计划表(发布)
    + 往计划表中追加一些需要处理的事情(订阅)
## 二. 发布订阅模式
发布订阅模式不是一个死的机制,他是一种思想,一种写代码的形式;主要为了处理一对多的场景,应用于不同情况下的不同函数的调用,this很重要
+ 手动进行模拟浏览器事件机制
    + 订阅
    ```javascript
        function on(ele,type,fn) {
          //  一个动作可以绑定多个处理程序，我们要把多个处理程序放在一个盒子里边
        ele.boxAry = [];
        if (!ele["boxAry"+type]) {
            ele["boxAry"+type]
        }
         ele["boxAry"+type].push(fn);
      }
    ```
    + 发布
        + 执行
+ 改变this指向
    + 创建一个小工具盒
    + 把我们的方法放到了小工具盒里边
    + 当我们在想用这个方法的时候,直接就到这个小工具盒变量就可以
+ 处理顺序问题的,其实根本就是处理IE事件池乱序问题,那么我们就不能用ie的事件池进行,我们模拟一个事件池(利用发布订阅的思想来模拟我们的事件池)
    + 监听的时候添加处理程序
    + 在发布的时候将监听的事件的处理程序都执行    
    
<i>这里结合ES6的语法，简单写一个发布订阅模式的小案例：</i>
```javascript
// 发布 emit 订阅 on {}
function Girl() {
    this._events = {}
}
Girl.prototype.on = function (eventName,callback) {

    //这里判断他是不是第一次添加(订阅)
    if(this._events[eventName]){ 
        this._events[eventName].push(callback); 
    }else{
        this._events[eventName] = [callback]
    }
};
Girl.prototype.emit = function (eventName,...args) { 
    if(this._events[eventName]){
        this._events[eventName].forEach(cb=>cb(...args));
    }
};

let cry = (who) =>{console.log(who+'哭');};
let shopping = (who) =>{console.log(who+'购物');};
let eat = (who) =>{console.log(who+'吃');};
let smile=(who)=>{console.log(who+'笑')};

let girl1 = new Girl();
girl1.on('失恋',cry); 
girl1.on('失恋',eat);
girl1.on('失恋',shopping);
girl1.emit('失恋','小明');  

let girl2 = new Girl();
girl2.on('恋爱',shopping);
girl2.on('恋爱',smile);
girl2.emit('恋爱','小华');
```
    





