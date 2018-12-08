---
title: 关于JS面向对象继承问题
subtitle: faceinherit
date: 2015-05-3
categories : JS
tags:
    - 面向对象
    - 继承
---
## 1.原型继承（是JS中很常用的一种继承方式）
子类children想要继承父类father中所有的属性和方法（私有+公有），只需要让children.prototype=new father;即可。<br/>
<b>特点：</b> 它是把父类中私有的+公有的都继承在了子类原型上（子类公有的）
<b>核心：</b> 原型继承并不是把父类中的属性和方法克隆一份一模一样的给子类，而是让子类和父类之间增加了原型链的连接，以后子类的实例想要使用父类中的方法，需要一级一级的向上查找来使用
```javascript
    function father(){
        this.x = 100;
    }
    father.prototype.getX = function(){
        console.log(this.x)
    };

    function children(){
        this.x = 200;
    };
    children.prototype = new father;
    children.prototype.constructor = children;
```

## 2. call继承：把父类私有的属性和方法克隆一份一模一样的作为子类私有的属性
```javascript
    function father(){
        this.x = 100;
    }
    father.prototype.getX = function(){
        console.log(this.x);
    };

    function children(){
        //this->n
        father.call(this);  // father.call(n) 把father执行让father中的this变为了n
    }

    var n = new B;
    console.log(n.x);
```

## 3. 冒充对象继承：把子类私有的 + 公有的 克隆一份一模一样的给子类私有的
```javascript
    function father(){
        this.x = 100;
    };
    father.prototype.getX = function(){
        console.log(this.x);
    };

    function children(){
        //this->n
        var temp = new father;
        for(var key in temp){
            this[key] = temp[key];
        }
        temp = null;
    }

    var n = new children;
    n.getX();

```

## 4.混合模式的继承：原型继承+call继承
```javascript
    function father(){
        this.x = 100;
    };
    father.prototype.getX = function(){
        console.log(this.x);
    };

    function children(){
        father.call(this);  //n.x = 100  
    }
    children.prototype = new father; //B.prototype：x=100  getX= ...
    children.prototype.contructor = children;

    var n = new children;
    n.getX();
```

## 5.寄生组合式继承
```javascript
    function father(){
        this.x=100
    };
    father.prototype.getX = function(){
        console.log(this.x);
    };

    function children(){
        father.call(this);
    };
    children.prototype = objectCreate(father.prototype);
    children.prototype.contructor = children;

    var n = new children;
    console.dir(n);
```

## 6.中间类继承法（不兼容）
```javascript
    function avgFn(){
        arguments.__proto__ = Array.prototype;

        arguments.sort(function(a,b){
            return a-b;
        };
        arguments.pop();
        arguments.shift();
        return eval(arguments.join("+"))/arguments.length;
    };
    console.log(avgFn(10,20,30,20,10,30,40,50));
```

等等......









