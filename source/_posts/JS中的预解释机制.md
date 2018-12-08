---
title: JS中的预解释机制
subtitle: explain
categories: JS
date: 2015-5-16
tags:
    - 预解释
---
预解释是一种毫无节操的机制（自从学了预解释，从此节操是路人）
in：‘num’ in window 是判断num是否是window这个对象的一个属性，是的话就返回true，不是返回false
```javascript
    var obj = {
        name:"高佳睿",
        age:18
    };
    console.log("name" in obj);  //true
    console.log("eat" in obj);  //false
```
## 1.预解释的时候不管条件是否成立，都要把带var的进行提前声明
window的预解释：var num； -> window.num;
```javascript
    if(!("num" in window)){   //"num" in window -> true
        var num = 12;
    }
    console.log(num);  //undefined
```
## 2.预解释的时候只先预解释“=”左边的，右边的是值，不参与预解释
匿名函数之函数表达式：把函数定义的部分当做一个值赋给我们的变量/元素的某一事件
```javascript
    //window下的预解释：var fn；
    fn(); //-> undefined() Uncaught TypeError: fn is not a function
```

```javascript
    var fn = function(){
        console.log("OK");
    };

    fn();  //->"OK"
    function(){
        console.log("ok")
    };
    fn();  //->"ok"

```
## 3.函数执行定义的那个function在全局作用域下不进行预解释；当代码执行到这个位置的时候定义和执行一起完成了
    自执行函数：定义和执行一起完成了
```javascript
//自执行函数的几张方式
    (function(){})(100)
    !function(){}(100)
    +function(){}(100)
    -function(){}(100)
    ~function(){}(100)
```

## 4.函数体中的return下边的代码虽然不执行了，但是需要进行预解释；return后边跟着的都是我们返回的值；所以不进行预解释；
```javascript
    function fn(){
        //预解释：var num
        console.log(num); //->undefined
        return function(){

        };
        var num = 100;
    };
    fn();
```

## 5.在预解释的时候如果名字已经声明过了，不需要再重新声明，但是需要重新赋值在JS中如果变量的名字和函数的名字重复了，也算冲突
预解释：var fn; window.fn; fn=xxxfff000 window.fn=xxxfff000
```javascript
    var fn = 13;
    function fn(){
        console.log("OK")
    }
```
window预解释：
    声明+定义 fn=xxxfff111
    声明var fn;(不需要重新声明)
    声明（不重复进行）+定义 fn=xxxfff222
        -> fn = xxxfff222
```javascript
    fn();  //22
    function fn(){console.log(1)};
    fn();  //2
    var fn = 10; // fn=10
    fn(); // 10(); Error:fn is not a function
    function fn(){console.log(2)};
    fn();
```












