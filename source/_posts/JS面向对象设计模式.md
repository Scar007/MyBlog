---
title: JS面向对象程序设计模式
subtitle: OOP
date: 2018-1-3
categories: JS
tags:
    - 面向对象
---
## 1.对象:JS中万物皆对象,它是一个泛指

#### 类:对象的具体的细分 (物以类聚，人与群分。具有相同属性和方法的实例的一个集合总称)
#### 实例:某一个类别中具体的一个事物
> 对象是一个抽象的概念,类似于我们的自然界；我们自然界中分为了 人类、动物类、植物类...,而我们每一个人都是人类中的一个实例

## 2.单例模式

#### 对象数据类型的作用:
把描述同一个事物(同一个对象)的属性和方法放在一个内存空间下,起到了分组的作用,这样不同事物之间的属性即使属性名相同,相互也不会发生冲突
+ ->我们把这种分组编写代码的模式叫做"单例模式"
+ ->在单例模式中我们把person1或者person2也叫做"命名空间"
```javascript
    var person1 = {
        name:"吴亦凡",
        age:18
    }
    var person2 = {
        name:"李嘉恒",
        age:20
    }
```
#### 单例模式是一种项目开发中经常使用的模式,因为项目中我们可以使用单例模式来进行我们的"模块化开"
#### "模块化开":对于一个相对来说比较大的项目,需要多人协作的开发的,我们一般情况下会根据当前项目的需求划分成几个功能版块,每个人负责一部分,同时开发,最后把每个人的代码进行合并

## 3.工厂模式
+ 单例模式虽然解决了分组的作用,但是不能实现批量的生产,属于手工作业模式->"工厂模式"
+ 把实现同一件事情的相同的代码放到一个函数中，以后如果在想实现这个功能，不需要从新的编写这些代码来了，只需要执行当前的函数即可-->"函数的封装" -->"低耦合高内聚":减少页面中的冗余代码,提高代码的重复利用率
```javascript
    function createJsPerson(name,age) {
        var obj = {};
        obj.name = name;
        obj.age = age;
        obj.writeJs = function(){
            console.log("my name is"+this.name+"my age is"+ this.age +",i can write Js la~")
        }
        return obj
    }
    var p1 = createJsPerson("吴亦凡",18);
    p1.writeJs();
    var p2 = createJsPerson("kris",20);
    p2.writeJs();
```
JS是一门轻量级的脚本"编程语言"(HTML+CSS不属于编程语言,属于标记语言)
> .net C# php Java c c++ vb vf object-c .... 所有的编程语言都是面向对象开发的->类的继承、封装、多态
+ 继承:子类继承父类中的属性和方法
+ 多态:当前方法的多种形态->后台语言中:多态包含重载和重写
+ “JS中不存在重载”,方法名一样的话,后面的会把前面+ 的覆盖掉,最后只保留一个
+ "JS中有一个操作类似重载但是不是重载":我们可以根据传递参数的不一样的,实现不同的功能

## 4.构造函数
构造函数模式的目的就是为了创建一个自定义类,并且创建这个类的实例
```javascript
    function createJsPerson (name,age){
        //浏览器默认创建的对象就是我们的实例p1 -> this
        this.name = name;  //p1.name = n
        this.age = age;
        this.writeJs = function(){
            console.log("my name is"+this.name+"my age is"+ this.age +",i can write Js la~")
        }
        //浏览器在把创建的实例默认的进行返回
    }
    var p1 = new createJsPerson("吴亦凡",18);  //createJsPerson -> this p1
    p1.writeJs();  //writeJs -> this p1
```
```javascript
    var res = createJsPerson("高佳睿",18);
    /*
        这样写不是构造函数模式执行而是普通函数执行
        由于没有写return所以res返回时undefined
        并且createJsPerson这个方法中的this是window
    */
    console.log(res);
```

## 5.原型模式
构造函数模式中拥有了类和实例的概念,并且实例和实例之间是相互独立开的->实例识别
基于构造函数模式的原型模式解决了 方法或者属性公有的问题->把实例之间相同的属性和方法提取成公有的属性和方法 ->想让谁公有就把它放在CreateJsPerson.prototype上即可
+ 1、每一个函数数据类型(普通函数、类)都有一个天生自带的属性:prototype(原型),并且这个属性是一个对象数据类型的值
+ 2、并且在prototype上浏览器天生给它加了一个属性constructor(构造函数),属性值是当前函数(类)本身
+ 3、每一个对象数据类型(普通的对象、实例、prototype...)也天生自带一个属性：proto，属性值是当前实例所属类的原型(prototype)
+ 4、Object是JS中所有对象数据类型的基类(最顶层的类)
    + 1)、f1 instanceof Object ->true 因为f1通过__proto__可以向上级查找,不管有多少级,最后总能找到Object
    + 2)、在Object.prototype上没有__proto__这个属性

## 6.原型链模式
f1.hasOwnProperty("x");//->hasOwnProperty是f1的一个属性
但是我们发现在f1的私有属性上并没有这个方法,那如何处理的呢?
1)通过 对象名.属性名 的方式获取属性值的时候，首先在对象的私有的属性上进行查找,如果私有中存在这个属性,则获取的是私有的属性值;
如果私有的没有,则通过__proto__找到所属类的原型(类的原型上定义的属性和方法都是当前实例公有的属性和方法)，原型上存在的话，获取的是公有的属性值; 如果原型上也没有，则继续通过原型上的__proto__继续向上查找,一直找到Object.prototype为止...
-->这种查找的机制就是我们的"原型链模式"
在IE浏览器中，我们原型模式也是同样的原理，但是IE浏览器怕你通过__proto__把公有的修改，禁止我们使用__proto__

+ 1、只有浏览器天生给Fn.prototype开辟的堆内存里面才有constructor,而我们自己开辟的这个堆内存没有这个属性,这样constructor指向就不在是Fn而是Object了
```javascript
    console.log(f.constructor);  //没做任何处理之前 Object
```
为了和原来的保持一致,我们需要手动的增加constructor的指向    
+ 2、用这种方式给内置类增加公有的属性 给内置类Array增加数组去重的方法
```javascript
Array.prototype = {
    constructor:Arrar,
    unique:function(){

    }
};
console.log(Array.prototype);
```
我们这种方式会把之前已经存在于原型上的属性和方法给替换掉，所以我们中这种方式修改内置类的话,浏览器是给屏蔽掉的
但是我们可以一个个的修改内置的方法,当我们通过下述方式在数组的原型上增加方法,如果方法名和原来内置的重复了，会把人家内置的修改掉-->我们以后再内置类的原型上增加方法，命名都需要加特殊的前缀

## 可枚举和不可枚举
```javascript   
    Object.prototype.aaa = function(){
        var obj = {name:"高佳睿",age:18};
        for (var key in obj) {
            /*
                for in 循环在遍历的时候
                默认的话可以把自己私有的和它所属类的原型上扩展的属性和方法都可以遍历到，
                但是一般情况下，我们遍历一个对象只需要遍历私有的即可，
                我们可以使用一下的判断处理
            */
            if (obj.prototypeIsEnumerable(key)) {
                console.log(key);
            }
            if (obj.hasOwnPrototype(key)) {
                console.log(key);
            }
        }
    }
```
