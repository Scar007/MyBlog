---
title: 前端开发概述+JS基础细节知识点
subtitle: web
date: 2017-7-21
categories: JS
---
## 一 前端开发概述
+ html页面:html css javascript
+ 拿到UI设计图纸:切图-->html+css静态布局-->用JS写一写动态效果-->ajax和后台进行交互,把数据动态绑定到页面上-->用node.js服务平台做源代码管理-->用node.js做后台

## 二 js引入到页面的方式和细节

#### 知识点巩固:css引入方式
+ 行内式
+ 内嵌式(将css样式写在style标签快里面,放在head里面)
+ 外链式(只是将内嵌式中css样式放到外面一个单独的样式文件中了)
+ 导入式:@import"css/index.css";

#### 将JS引入到页面中的几种方式:
+ 行内式(不推荐,安全性能非常低)
+ 内嵌式(将js代码写在script 脚本块中间)
+ 外链式(将js代码写在外边的文件中,通过src找到引入)

+ 细节点:
    + 在外链式中script脚本块中,不可以写js代码,写了也不执行
    + 我们通常将js放在body最后面,原因:html页面是从上到下依次加载的,js通常是获取html标签给与动态操作效果的,所以需要先加载html标签再加载我们的js.

## 三 JS的四种输出方法
+ alert('内容') 在浏览器中弹出框显示我们的内容
+ document.write('内容') 在页面中输出显示我们的内容
+ console.log('内容') 最常用的一种方式,在控制台输出我们的内容    
    + console.error();向控制台抛出异常
    + console.dir();输出一个对象的全部属性(详细内容)
    + console.clear();清空控制台
        + 清空控制台的方法:
            + ctrl+l
            + 右击clear
            + 直接点击小图标
+ innerHTML/innerText向指定元素中动态添加内容

## 四 JS的组成和命名规范
#### 层级关系:浏览器(window浏览器对象)->文档(document文档对象)->html->head/body->...
#### js:javascript,是一门轻量级的脚本编程语言
#### js组成(三部分
+ ECMAScript:定义了js里边的命名规范,变量,数据类型,基本语法,操作语句等最核心的东西   
+ DOM:document object model 文档对象模型
+ BOM:browers object model 浏览器对象模型
#### 命名规范:    
+ js中严格区分大小写
+ 使用驼峰命名法
    + 首字母小写,其余的每个有意义的单词的首字母大写
    + 可以使用数字,字母,下划线,$, (数字不能作为首字母)
+ 不能使用关键字和保留字
    + 关键字:在js中有特殊意义的字
    + 保留字:未来可能成为关键字的    

## 五 JS中的变量和数据类型
#### js变量
+ 变量:可变的量
+ js中的变量是一个抽象的概念,变量是用来存储值和代表值的

在js中定义一个变量非常的简单:
+ var 变量名=变量值;例:var name='lucy';//定义一个变量,把字符串lucy赋值给这个变量 console.log(name);//把name代表的值输出在控制台
+ = 是赋值操作,左边是变量名,右边是存储的值
+ js中的变量是松散类型的:通过一个var变量名就可以存储任何的数据类型

#### js中的数据类型:
+ 1.基本数据类型
+ 2.引用数据类型

#### 基本数据类型:由简单的结构组成
+ 数字(number)
+ 字符串(string):双引号或者单引号包起来的都是字符串
+ 布尔(boolean)
+ null
+ undefined    

<hr>

### number(数字)：
##### 强制类型转换
##### number:正数,负数,0,小数,NaN
+ NaN:not a number 不是一个有效数字,但他是属于number数据类型的
+ NaN==NaN 是不想等的
##### Number():强制将其他的数据类型转换为number类型,要求如果是字符串,字符串一定都需要是数字才可以转换;
+ number的返回值:不是数字就是NaN
+ 机制:属性名,从左向右依次查找,只有有一个不是数字,就会返回NaN
##### isNaN();是一个方法:检测一个值是否为有效数字,是有效数字返回false,不是有效数字是true;
+ 如果检测的值不是number类型的,浏览器会默认的把它转化成number类型的,然后判断是否为有效的数字
##### 非强制类型转换
+ parseInt/parseFloat
+ parseInt
    + 机制:从左向依次查找,只要碰到有一个字符不是数字就停止查找,将查找到的字符返回,但是它的数值为整数,自动忽略小数点
    + 返回值:为整数,不是数字就是NaN
    + 作用:可以解析字符串,返回一个整数
+ parseFloat
    + 机制:从左向右依次查找,只要碰到有一个字符不是数字就停止查找,将查找到的字符返回,返回的值是浮点数,带小数点
    + 作用:可以解析字符串,返回一个浮点数
    + ]返回值不是数字就是NaN
+ 一个=号:表示赋值
+ 二个=号:是判断左右两边的值是否相等


### boolean(布尔值)：
##### boolean:true/false
##### !:一个叹号是取反,首先将值转化为布尔类型的,然后再取反
##### !!:两个叹号,将其他的数据类型转换为boolean类型,相当于Boolean();->取反再取反
```javascript
console.log(Boolean("gaojiarui"));
console.log(!!"gaojiarui");  //以上两种表达的意思是一样的
```

<hr>

### 引用数据类型:结构相对复杂一些的
##### 对象数据类型(object)
##### 对象类(Object),数组类[Array],正则类|^$|,时间类Date,字符串类(String),布尔类(Boolean)...等对应类的实例:对象,数组,正则,时间...
##### 函数数据类型(function)

##  js数据类型中的对象数据类型
#### 特点:由多组[属性名和属性值]组成,多组键值对组成,由多个key:value
#### 属性名和属性值是用来描述这个对特征的
#### 创建方式
+ 字面量创建方式:var obj={name:'lucy'};
+ 实例创建方式:var obj=new Object();
#### 给一个对象增加一组属性名和属性值方法
+ obj.name="lucy";
+ obj["name"]="lucy"
#### 修改原有属性名的属性值
+ 规定:一个对象中的属性名不能重复,如果之前有就是修改,没有就是增加
+ obj.name="lucyjie"
+ obj["name"]="lucyjie"
#### 获取属性名和属性值
+ 如果属性名不存在,默认返回的结果是undefined console.log(obj.zz);->undefined
+ console.log(obj["name"]);
+ console.log(obj.name);
#### 删除属性名和属性值
+ 假删除:obj.age=null;
+ 真删除:delete.obj.age; console.log(obj);

## 六 数据类型区分和数据类型检测
#### 基本数据类型和引用数据类型的区别
```javascript
var num1 = 12;
var num2 = num1; //把num1代表的值赋值给了num2变量
num2++; //相当于num2=num1+1 在给自己原有的值的基础上加1，也可以写成num2+=1;
console.log(num1)  //12

/*
基本数据类型：把值直接给变量
接下来在操作的过程中，直接拿这个值来操作；
可能两个变量存贮一样的值，但是你的事你的，我的是我的，之间没有关系
其中一个改变，另外一个没有任何影响
*/

var obj1 = {name:"gjr"};
var obj2 = obj1;
obj2.name = "gaojiarui";
console.log(obj1.name)  //gaojiarui

/*
引用数据类型：
1.定义一个变量
2.开辟一个新的空间，然后把属性名和属性值保存在这个空间上，并且有个空间地址
3.把空间的地址给这个变量，变量并没有存贮这个值，而是存贮的是这个空间的引用地址
4.接下来我们把这个地址又告诉了另外一个变量，另外一个变量存贮的也是这个地址，此时如果发生改变的话，两个变量其实操作的是同一个空间
5.其中一个改变了空间的内容，另外一个其实也改变了
*/
```

#### 本质区别:
+ 基本数据类型操作的是值
+ 引用数据类型操作的是内存地址

### [js中检测数据类型的方法](https://scar007.github.io/2016/11/25/testtype/)

#### typeof 运算符
+ 用来检测数据类型的:typeof 要检测的值
+ 返回值:是一个字符串;
+ 包含了数据类型字符"number","string","boolean","undefined","object","function"
+ //特殊记忆:typeof null;->"object"
+ 局限性:不能具体的检测object下细分的类型,检测这些返回的都是"object"
```javascript
var num1 = 12;
console.log(typeof num1); //把num1代表的值进行检验，检测是什么数据类型的

//面试题
console.log(typeof (typeof (typeof [])))  //"string"
/*
    分析：
    typeof [] -> "object"
    typeof "object" -> "string"
    typeof "string" -> "string"
    //规律：出现两个或者两个以上的typeof的时候，最终的结果都是字符串
*/
```

#### instanceof 运算符
#### constructor
#### Object.prototype.toString.call()

## 七 js中三个判断的语法

三个判断:if else,三元运算符,switch case
#### if ,else if ,else 是最常用的判断,可以解决js中所有的判断需求
#### 三元运算符应用于简单的if else情况
#### switch case应用于不同值情况下的不同操作
```javascript
// if else
//语法
if (条件1){
    执行条件1成立后js代码
}else if(条件2){
    执行条件2成立后js代码
}else if(条件3){
    执行条件3成立后js代码
}...
else{
    执行以上所有条件都不成立的js代码
}

/*
 if 中的条件可以是 大于 小于 等于
 还可以是一个值（判断当前值代表的是真还是假）
 if中的条件可以是多个小的条件的组合，中间用||（只要有一个为真，整体就为真）和&&（所有小条件全都为真后，整体才为真）隔开
*/

```
```javascript
// 三元运算符
//语法
//条件?条件成立执行的代码:条件不成立执行的代码

num=>0?console.log("整数或零"):console.log("负数");
// 可以用void来补位，例如：
num=>0?console.log("整数或0"):void 0;
```
```javascript
// switch case
/*
每一种case情况下都要加break
如果不加break，不管后边的代码是否成立都被执行了
每一个case情况都相当于===比较，一定要注意数据类型是否一致；
*/
//语法：
var num = 10;
switch (num) {
    case 0:
    console.log("0");
    break;
    case 5:
    console.log("5");
    break;
    case 10:
    console.log("10");
    break;
    default:
    console.log("NaN");
}
```

## 八 js中三个循环
#### for 循环:
+ 四部曲
    + 第一步:设置初始值 var i=0;
    + 第二步:设置循环执行的条件 i<5;
    + 第三步:执行循环体中的内容{[循环体]}包起来的部分
    + 第四步:执行每一轮循环完成后都执行我们的i++累加操作
+ break/continue:(关键字)
+ 在循环体中,只要遇到break和continue,循环体中后面的代码就不在执行了;
    + break:在循环题中,出现break,整个循环就直接的结束了,i++最后的这个累加的操作也不再执行了
    + continue:在循环体中,出现continue,当前这一轮的循环就结束了,继续下一轮的循环,后面的累加操作继续执行
    ```javascript
        for (var i=0;i<5;i++) {
            console.log(i); // 0,1,2,3,4,5
        }
        //例
        for(var i=0;i<5;i++){}
        console.log(i); //10  循环体执行完后才输出最后的
    ```
#### for in循环
+ 用来循环一个对象中的属性名和属性值的
+ 对象中有多少组键值对,我们就循环多少次
+ 顺序问题:首先循环数字的属性名(按照从小到大),再把剩下的属性名按照我们写的顺序循环    
```javascript
//语法：对象中有多少组键值对，我们就循环多少次
for (var key in obj) {
    console.log(key)  //每一次循环获取的属性名
    console.log(obj[key]) //获取属性值，在for in中只能通过 对象名[key]来获取，不能写obj.key
}
```












