---
title: JS面向对象程序设计
subtitle: OOP
date: 2017-12-21
categories: JS
tags:
    - 面向对象
---
## 你是如何理解编程语言中的面向对象的?
我们研究JS和使用JS编程本身就是基于面向对象的思想来开发的，JS中的一切内容都可以统称为要研究的“对象”，我们按照功能特点把所有内容划分成“几个大类，还可以基于大类划分小类”，我们开发研究的时候拿出类中的一个具体事物“类的实例”来操作，当前实例具备的一些特点，同属于当前类的其他实例也具备这些特点；我们还要研究关于类的“封装、继承、多态”，这样有助于我们的编程开发。

### JS中的类：内置类、自定义类
+ Function：所有的函数数据类型都是它的一个实例，普通函数、类(自定义的类、内置的一些类)这些都是函数数据类型的
+ Object：对象类，所有的对象数据类型(普通对象{}、数组[]、正则/^$/、Math、日期对象、类的实例、类.prototype、函数本身也具备普通对象的特点)都是它的一个实例；Object是一个大类，也是基类，下面可以划分跟多的小类：  
    + Array
    + RegExp
    + Date
    + String
    + Number
    + Null
    + Undefined
    + Boolean
    + EventTarget
        + Node
            + Element
                + HTMLElement
                    + HTMLDivElement
                    + HTMLParagraphElement
                    + HTMLAnchorElement
                    + HTMLImageElement
                    + ...
                + ...
            + Text
        + Comment
        + Document
        + ...
    + ...
    + HTMLCollection：通过getElementsByTagName/getElementsByClassName等获取的元素集合就是它的一个实例
    + NodeList：通过getElementsByName和childNodes等获取的节点集合都是它的一个实例
    + ...
+ 面向对象中的实例和类的关系：实例除了可以调取自己的私有属性方法使用之外，还可以调取自己率属类原型上的公共属性方法；一个类可以创造很多很多的实例，不同实例之间既具备独立性也具备共同性；
+ 构造函数模式中的原型链

<img src="https://images2017.cnblogs.com/blog/1140938/201710/1140938-20171011165415637-497302518.png"/>

<img src="https://images2017.cnblogs.com/blog/1140938/201710/1140938-20171011165453668-1274696718.png">

```javascript
    //在内置类的原型上扩展一个方法：myHasPubPrototype,检测某一个属性是否属于当前这个对象的公有属性
    Object.prototype.myHasPubPrototype = function myHasPubPrototype(){
        //this：我们当前操作的实例
        'use strict';
        var attr = arguments[0];
        if (typeof attr === "undefined") {
            // throw new Error('parameters error!'); Error:ReferenceError/TypeError/RangeError/SyntaxError...
            return false;
        }
        return (attr in this) && this.hasOwnPrototype(attr) === false;
    }
    var obj = {};
    obj.hasOwnPrototype('name');
    obj.myHasPubPrototype('name');
```

