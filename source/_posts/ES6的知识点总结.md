---
title: ES6的基础知识点总结
subtitle: es6
categories: ES6
date: 2017-8-23
tags:
    - es6
---
## ES6
其实就是对ES5的一个完善
不是所有的浏览器都兼容,低版本浏览器就不兼容
目前项目中使用ES6非常多(React大部分开发者都是基于ES6写的),因为ES6非常的方便
我们需要把不兼容的ES6转换为兼容的ES3/ES5代码 =>"Babel"

### ES6中定义变量使用 let/const
var a=1;//在es5中,a是一个全局变量,相当于给window自定义了一个属性a,window.a,在es6中,它对全局对象window和全局作用域实现分离,但是为了保证兼容性,依然保留var声明的全局变量的特性,利用let声明块级变量来实现的只是相当于一个全局变量

#### let
let 声明只在当前块级作用域起作用的变量(利用块级作用域理解let)
+ 使用let定义的变量不能进行"变量提升"
```javascript
console.log(a);//Uncaught ReferenceError: a is not defined
let a = 1;
```
+ 同一个作用域中,let不能重复定义相同的变量名
```javascript
let b = 2;
let b = 3; // Uncaught SyntaxError: Identifier 'b' has already been declared
```
+ 使用var在全局作用域中定义的变量相当于给window增加了一个私有的属性,但是使用let定义的变量和window没有任何的关系

#### const
除了拥有let的那些特点之,const定义的变量是一个恒定的值(常量),存储的值是不能进行改变的
+ 如果常量的值是基本数据类型的值就不能变
+ 如果常量的值是引用数据类型它的引用指针就不能变
+ const定义的常量具有和let一样的特性
+ const定义的是一个恒定的值,是个常量,存储的值是不能更改的,必须"声明+定义"
+ 不能同时使用const和let声明同一个的变量

#### 块级作用域
从字面量理解 {}包含的就是一个块级作用域
+ ES5中的作用域只有两种
    + 全局作用域
    + 函数执行形成的私有作用域
+ ES6中的作用域多加了一种
    + 块级作用域:基本上凡是被"{ }"包起来的都是块级作用域,块级作用域和之前学习的私有作用域一样,保护里面的私有变量不外界的干扰;而且作用域链机制和之前相同
    ```javascript
        let a = 1;
    if(1){  //这是块级作用域
        let b = 2;
        console.log(a);//1
        console.log(b);//2
    }
    console.log(a);//1
    console.log(b);//Uncaught ReferenceError: b is not defined
for循环,每次循环都会形成一个块级作用域,而i是每一个私有作用域中的私有变量
    for (let i = 0; i < 5; i++) {
        setTimeout(function(){
            console.log(i);
        },0);
    }//依次输出0,1,2,3,4
对应在ES5中:做了以下工作
    var _loop = function _loop(i) {
        setTimeout(function () {
            console.log(i);
        }, 0);
    };
    for (var i = 0; i < 5; i++) {
        _loop(i);
    }
    ```
    + 每一次循环都会形成一个块级作用域,而i是每一个私有作用域中的私有变量

#### 变量的解构赋值
基本概念：
+ 本质上就是一种匹配模式，只要等号两边的模式相同，那么左边的变量就可以被赋予对应的值。
+ 结构赋值主要分为：
    + 数组的解构赋值
    + 对象的结构赋值
    + 基本类型的解构赋值

<b>【数组解构】</b>
```javascript
    const ary = ['鼠', '牛', '虎'];
    let [a]=ary;   =>var a = ary[0];
    let [,b]=ary;   =>var b = ary[1];
    let [c,d,e,f]=ary; =>var a = ary[0],b = ary[1],c = ary[2],d = ary[3]

    const ary = ['鼠', ['牛', '虎']];
    let [a,[b,c]]=ary;  //与上面一致 ,对应变量匹配对应位置的值
```

<b>【对象解构】</b>
```javascript
    let {a, b} = {b: 'bbb', a: 'aaa'};  => var a = 'aaa',b = 'bbb';
    let {a: b} = {a: 1};                => var b = 1;
```

<b>【基本类型的解构】</b>
```javascript
    let [a, b, c, d] = '1234';   => var a = 1,b = 2, c = 3, d = 4
    let {length: len} = 'qwqb';  => var len = 4;
```

#### 箭头函数
+ function () {};
+ () => {};
+ let fn(a=0,b=0) =>a+b
    + 可以给形参设置默认值,而不是undefined
+ 箭头函数中的this会继承父级作用域中的this,例如在定时器中的回调函数,或自执行函数,不再是window,而是继承
+ 真实项目中混用箭头函数和普通函数,视情况选用
+ 箭头函数没有arguments

rest 参数形式为（“...变量名”），用于获取函数的多余参数，这样就不需要使用arguments对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。
箭头函数体内没有自己的this对象，所以在使用的时候，其内部的this就是定义时所在环境的对象，而不是使用时所在环境的对象。
不能给箭头函数使用 call apply bind 去改变其内部的this指向
箭头函数体内没有arguments对象，如果要用，可以用Rest参数代替。
不可以当作构造函数，不可以使用new命令，否则会抛出一个错误。
```javascript
const fn1=()=>{};    => var fn1 = function(){};
    const fn2=a=>a;      => var fn2 = function(a){return a};
    const fn3=(a,b)=>a+b;=> var fn3 = function(a,b){return a+b;};
    const fn4=(a,b)=>{       //这种是需要做特殊处理或者多条处理逻辑使用
        return a*100+b*100;
    }
const fn5 = (a = 10,b = 20)=>a+b;  初始化参数
    var fn5 = function(a,b){
        if(typeof a === 'undefined'){
            a = 10;
        }
        if(typeof b === 'undefined'){
            b = 20;
        }
        return a+b;
    }
```

#### 类（类的封装和继承）
+ 类的创建
```javascript
class Fn{
    //类,其中可以定义实例的私有属性
    constructor(){
        
    }
    //公有的方法
    eat(){
        
    }
    //作为类的私有属性
    static say(){
        
    }
}
```
+ 类的继承(寄生组合式继承)
```javascript
class fn2 extends Fn{
    constructor(){
        //指的是fn2
        super();//继承了父类实例的私有属性
    }
    //继承父类的公共方法
    //不能继承父类的私有属性
}
```
class 的继承等相关知识

extends、 static、 super
```javascript
class Person {
        constructor(name, age) {
            this.name = name;
            this.age = age;
        }
        say() {}
        fn() {}
        static eat() {}
    }
    console.dir(Person);//-> 其中name/age是当前类实例的私有属性；say/fn写在了Person.prototype上；eat写在了Person上；

    class ManPerson extends Person {
        constructor(name, age) {
            super(name, age);//->supper就是继承的类,这里相当于Person.prototype.constructor.call(this, name,age) 或者 Person.call(this,name,age);
            this.sex = '男';
        }
        moneyFn() {
            console.log('男人要赚钱养家!');
        }
    }
    console.dir(ManPerson);
    //->ManPerson 继承了 Person
```

#### 对象中增加函数属性
+ 直接函数名(){},不需要用键值对的方式
+ 对象的解构赋值需要保证变量名和属性名一致才可以获取对应位置的值(默认情况下)
+ let {a:x,b:u,x;z}=obj 相当于在给原有的属性名设置别名

#### 扩展运算符"..."
在使用拓展运算符的很时候,...keys只能放在最后,如数组解构,函数参数解构的时候
```javascript
let ary=["ss",12,21];
let [a,...b]=ary
console.log(a)//->"ss"
let fn=(info,...scoreAry){
    //可以弥补arguments的缺失,接受实参
}


    const arr = [1, 2, 3, 4];
    let [a,...key]=arr;     =>var a = 1,key = [2,3,4];

    const fn = (parm1, ...keys)=> {
        console.log(keys);      =>输出 keys = [ 2, 3, 4, 5, 6];
    };
    fn(1, 2, 3, 4, 5, 6);
```

#### 新增的数据结构Set和Map
数据结构 Set数据结构集合的基本概念：集合是由一组无序且唯一（即不能重复）的项组成的。这个数据结构使用了与有限集合相同的数学概念，应用在计算机的数据结构中。
特点：key 和 value 相同，没有重复的 value。

ES6 提供了数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。
+ 如何创建一个 Set const s = new Set([1,2,3]);
+ Set 类的属性 console.log(s.size); =>3
+ Set 类的方法
    + a) set.add(value) 添加一个数据，返回Set结构本身。
    + b) set.delete(value) 删除指定数据，返回一个布尔值，表示删除是否成功。
    + c) set.has(value) 判断该值是否为Set的成员，反回一个布尔值。
    + d) set.clear() 清除所有数据，没有返回值。
    + e) keys() 返回键名的遍历器
    + f) values() 返回键值的遍历器
    + g) entries() 返回键值对的遍历器
    + h) forEach() 使用回调函数遍历每个成员

数据结构 Map 字典：是用来存储不重复key的Hash结构。不同于集合(Set)的是，字典使用的是[键，值]的形式来储存数据的。

JavaScript 的对象（Object:{}）只能用字符串当作键。这给它的使用带来了很大的限制。

为了解决这个问题，ES6提供了Map数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object结构提供了“字符串—值”的对应，Map结构提供了“值—值”的对应，是一种更完善的Hash结构实现。如果你需要“键值对”的数据结构，Map比Object更合适。
+ 如何创建一个 Map => const map = new Map([['a', 1],['b', 2]]);
+ Map 类的属性 console.log(map.size);
+ Map 类的方法
    + 1 map.set(key, value) 设置键名key对应的键值为value，然后返回整个 Map 结构。如果key已经有值，则键值会被更新，否则就新生成该键。
    + 2 map.get(key) get方法读取key对应的键值，如果找不到 key，返回undefined。
    + 3 map.delete(key) 删除某个键，返回true。如果删除失败，返回false。
    + 4 map.has(key) 方法返回一个布尔值，表示某个键是否在当前Map对象之中。
    + 5 map.clear() 清除所有数据，没有返回值。
    + 6 map.keys() 返回键名的遍历器
    + 7 map.values() 返回键值的遍历器
    + 8 map.entries() 返回键值对的遍历器
    + 9 map.forEach() 使用回调函数遍历每个成员

#### 什么是 Symbol ?
Symbol，表示独一无二的值。它是 JS 中的第七种数据类型。
```javascript
// 基本的数据类型： Null Undefined Number Boolean String Symbol
   // 引用数据类型：Object

   let s1 = Symbol();
   let s2 = Symbol();
   // console.log(typeof s1); // 'symbol'
   //
   // console.log(s1 === s2); // false
```
Symbol 函数前不能使用 new 否则会报错，原因在于 Symbol 是一个原始类型的值，不是对象。
Symbol 函数接收一个字符串作为参数，表示对Symbol的描述，主要是为了在控制台显示，或者转为字符串的时候，比较容易区分 let s4 = Symbol('leo');
Symbol 数据类型的转换
作为对象的属性名

不能被for...in循环遍，历虽然不能被遍历，但是也不是私有的属性，可以通过Object.getOwnPropertySymbols方法获得一个对象的所有的Symbol属性

#### 字符串扩展
+ 模板字符串
+ repeat
```javascript
// let str1 = 'a';
// let str2 = str1.repeat(3);
// console.log(str2);   => 'aaa'
```
+ includes() startsWith() endsWith() //开始位置,结束位置或任意位置是否包含某字符串,包含返回true,不包含返回false

#### 对象扩展
+ 1)简洁表示
```javascript
let a = 1;
const obj = {a};  => obj = {a:a};
```
+ 2)Object.is() 判断值是否"一样"
```javascript
// console.log(Object.is(NaN, NaN));  => true
// console.log(Object.is(+0, -0));    => false
```
+ 3)Object.assign()
 用于对象的合并，将源对象的所有可枚举属性，复制到目标对象。

#### 数组的扩展
+ Array.from([12,34,5]) 将类数组转化成数组
+ Array.of() //创建新数组
    + Array.of(12,34,56)
    + [12, 34, 56]
+ find() findIndex() 返回复合条件项的索引
+ fill() 替换
```javascript
const arr = [1, 2, 3, 4];
arr.fill('abc', 1, 3); => arr = [1, 'abc', 'abc', 4];
```

#### 其它
在ES6中新增了Set和Map两种数据结构，再加上JS之前原有的数组和对象，这样就有了四种数据集合，平时还可以组合使用它们，定义自己的数据结构，比如数组的成员是Map，Map的成员是对象等。这样就需要一种统一的接口机制，来处理所有不同的数据结构。
+ 1.Iterator就是这样一种机制。它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署Iterator接口，就可以完成遍历操作，而且这种遍历操作依次处理该数据结构的所有成员。
    + Iterator遍历器的做用：
        + 为各种数据结构，提供一个统一的、简便的访问接口。
        + 使得数据结构的成员能够按某种次序排列。
        + ES6新增了遍历命令for...of循环，Iterator接口主要供for...of消费。
    + Iterator的遍历过程：
        + 创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象。
        + 第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员。
        + 第二次调用指针对象的next方法，指针就指向数据结构的第二个成员。
        + 不断调用指针对象的next方法，直到它指向数据结构的结束位置。

每一次调用next方法，都会返回数据结构的当前成员的信息。具体来说，就是返回一个包含value和done两个属性的对象。其中，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。
+ 2 凡是具有 Symbol.iterator 属性的数据结构都具有 Iterator 接口
+ 3 具备iterator接口的数据结构都可以进行如下操作
    + 解构赋值
    + 扩展运算符
+ 4 for...of循环







