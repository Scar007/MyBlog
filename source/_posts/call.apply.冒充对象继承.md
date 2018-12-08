---
title: call.apply
subtitle: call
date: 2017-10-2
categories: JS
tags:
    - call
    - apply
---
## call方法：让调用对象执行，然后第一参数是谁。调用对象的this就改变,指向谁，后边跟参数，依次对应传入

## apply方法：让调用对象执行，然后第一参数是谁。调用对象的this就改变指向是谁，后边跟参数，以数组的形式传入
```javascript
    function a(n,age){
        this.per = n;
        this.age = age;
        this.num = "100";
        //console.log(this.age)
    }
    var b = {
        per:"h"   //这里的"h"只是做比较用的
    }
    console.log(b);
    a.call(b,"sss",666,"hhh");  //第一个是调用，后边的参数是改变里边的值，a先执行才能使用call改变b里边的东西
    console.log(b);
    a.apply(b,["cc",888]);  //跟call一样，只是形式变了一下
    console.log(b)
```

```javascript
Array.prototype.sum = function () {
    var num = null;
    for (var i=0;i<this.length;i++) {
        var pre = parseFloat(this[i]);
        if (!isNaN(pre)) {
            num+=pre;
        }
    }
    return num
};
var ary = [1,2,3,4,5,"h","6","7px","8.1"];
console.log(ary.sum())
```

## call继承 --- 改变调用对象this的指向

#### call 继承  把父类（A）设置私有的属性，克隆一份作为子类(B)私有的
```javascript
    function A(){
        this.name = "aaa";
        this.age = 333;
    }
    function B(){
        this.age = 888
    }
    B.prototype.showtime = "good good study,day day up";
    A.prototype = new B();
    A.prototype.constructor = A;
    var a = new A();
    console.log(a.showtime);
    console.log(a.age);
    console.log(a.__proto__.age);
```
```javascript
    function A (){  //构造函数中的this指的是当前函数的实例
        this.name = "kris";  //这里的this指的是a
        this.age = [];
    };
    function B (){
        this.name = "gjr";
        A.call(this);  //这里的this指的是B，但是改变A中的this指向，将A中的this指向改变成了B中的实例(b)，即：将A中的私有属性定义给B
    };
    var a = new A();
    var b = new B();
    console.log(b);  //kris Array(0) 指的就是上边那个age定义的那个空数组，0代表的是数组长度
    console.log(a);  //kris Array(0)
```


