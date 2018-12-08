---
title: 关于JS中的数据类型
subtitle: numType
date: 2015-05-6
categories : JS
tags:
    - 数据类型
---
JS中的数据类型，大范围包括两种：
+ 基本数据类型（简单数据类型）
    包括:
        Number、String、Boolean、null、undefined
+ 引用数据类型（复杂数据类型）
    包括：
        Object：object（对象）、Array（数组）、RegExp（正则）、Date（日期）、
        Function：Math（数学）

```javascript
    var num = 12;
    var obj = {
        name:"高佳睿",
        age:18
    };
    function fn(){
        console.log("勿忘初心，方得始终！");
    };
    console.log(fn); //把整个函数的定义部分（函数本身）在控制台输出

    console.log(fn()); //把当前函数执行的返回结果（return后边写的是什么，返回值就是什么，如果没有return，默认返回值就是undefined）
```


