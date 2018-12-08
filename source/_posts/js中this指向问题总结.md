---
title: JS中this指向问题总结
subtitle: this
date: 2015-04-2
categories : JS
tags:
    - this
---
## ES5中this指向问题
this的指向在函数定义的时候是确定不了的，只有函数执行的时候才能确定this到底指向谁，实际上this的最终指向的是那个调用它的对象
+ 1.函数执行的时候，首先看函数名前边是否有点 ‘·’，有的话点’·‘前边是谁是this就是谁，没有的话就是window
```javascript
function(){
    console.log(this)
}
var obj={fn:fn};
fn(); //this指的是window
obj.fn(); //this指的是obj
sum(); //this指的是window
```
+ 2.自执行函数中this永远指的是window，严格模式下是undefined
+ 3.给元素的某一个事件绑定方法，当事件触发的时候执行对应的方法，方法中的this是当前的DOM对象
```javascript
document.getElementById.onclick=fn; //fn中的this指的是box
document.getElementById.onclick=function(){
    fn();  //fn()中的this是window
}
```
+ 4.在构造函数模式中、类中(函数体中)出现的this.xxx=xxx中的this是当前类的实例
```javascript
function perso(name,age){
    this.name=name;
    this.age=age;
    this.write=function(){
        console.log('my name is' + this.name +',我'+ this.age +'岁了！')
    }
}
var p1=new person('高佳睿','18') //这里的person是一个类
```
+ 5.在原型模式中，this常用2种方法：
    + 在类中this.xxx=xxx;this是当前类的实例
    + 某一方法中的this,看点'.'前边是谁this就是谁
        + 先确定this指向
        + 把this替换成对应的代码  
        + 按照原型链查找机制，一步一步查找

+ 6.call和apply强制改变this的指向->以上所有的this情况在遇到call/apply的时候都不好使,都已强制改变的为主obj.fn.call(1);//this->1

一般情况下,我们执行call方法第一个传递的参数值是谁,那么fn中的this就是谁
[在非严格模式下]
第一个参数没有传递值、传递的是null、传递的是undefined fn中的this都是window
[严格模式下]
第一个参数传递的是谁this就是谁,传递null/undefined,fn中的this都是对应的null/undefined,不传递值默认也是undefined

> 一定要切记的一句话：你以为 你以为的 就是你以为的

## ES6中this：
箭头函数中的this因为绑定了词法作用域，所以始终指向自身外的第一个this（由于自身没有声明this，所以会去作用域链上找this），也就是始终等于调用它的函数的this（以为这个this离它最近）。
