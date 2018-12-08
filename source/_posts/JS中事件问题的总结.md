---
title: JS中事件问题的总结
subtitle: event
categories: JS
date: 2015-7-18
tags:
    - 事件
---
## 一.什么是事件？
事件就是DOM和浏览器之间的交互行为(只要触发了这个行为，也就相当于触发了事件)，我们可以通过事件监听来绑定事件，例如：box.onclick=function(){}，如果我们点击了这个盒子，就触发了盒子的click事件，同样会把事件绑定给这个方法，让其执行某一些特定的操作。(事件是浏览器默认给的，与生俱来的，绑定事件时不过是给某一特定元素绑定对应的事件名以及方法，让其执行特定的方法)
### DOM事件：
+ 点击: click
+ 滚轮: scroll
+ 滑动: move
+ 进入:enter
+ 加载:load
事件绑定：就是给某一个元素的相关事件行为绑定方法
    -  <b>DOM 0级事件：</b>
```javascript
    oDiv.onclick = function(){
        //当我们触发oDiv的click行为的时候，会把绑定的这个函数执行
    };
    // ->onclick 这个行为定义在当前元素的私有属性上
```
    - <b>DOM 2级事件：</b>
```javascript
    oDiv.addEventListener('click',function(){
        console.log('ok')
    },false);
    // -> addEventListener这个属性是定义在当前元素所属EventTarget这个类的原型上的
```

## 二.事件机制：
<b>事件的监听(事件的绑定)：</b>
+ 通过on+事件名这种类型绑定事件叫做DOM 0级事件
+ DOM０级事件，同一个元素的同一个事件监听，绑定的触发运行函数，不能重复绑定，有且只能绑定一回，下一次绑定的会将上一次的覆盖掉
+ 事件设为null，移除事件的监听
+ 监听事件不仅是浏览器的一种机制，也是浏览器赋予DOM文档元素的属性-事件的监听是时刻存在的，但是事件的触发时候运行的处理函数是我们自己定义的

<b>事件的触发：</b>
+ 触发事件的时候运行绑定处理的函数

## 三.DOM 0级与DOM 2级事件的区别：
+ 1.DOM 0级属于给当前元素的某一个私有的属性赋值：box.click=...
    - 1).我们通过事件的监听处理程序 onclick属性=function(){}这种形式绑定的事件，叫做DOM 0级事件
    - 2).所有的DOM 0级事件都是在冒泡，执行的事件处理函数
    - 3).只能给当前元素的某一个事件绑定一个方法，绑定多个的时候，后边绑定的会将前边的覆盖掉
    - 4).我们所说的事件是文档和浏览器的交互行为，而文档是DOM，DOM包含了html和document
    - 5).移除事件监听的处理程序就是直接访问DOM对象的事件监听属性，然后讲处理事件的函数设置为null，比如：
```javascript
    div.onclick = null;  //直接就移除了监听事件的处理函数
```
+ 2.DOM２级属于通过原型链的查找机制，找到EventTarget.prototype的addEventListen这个方法，执行这个方法，把需要绑定的函数放在当前元素的事件池中。
    - 1).可以给当前元素的某一事件类型绑定多个不同的方法，触发事件，这些绑定的方法会按照在事件池中的顺序(和绑定的的顺序相同)依次把方法执行(IE低版本有区别)
    - 2).DOM 2中有一些单独的事件，这些事件不是元素的私有属性，所以只有DOM2的绑定方式才会给这些事件绑定方法，例如：DOMContentLoaded...
    - 3).DOM 2级事件处理事件函数的执行时间是在捕获阶段执行还是在冒泡阶段执行
    - 4).DOM２级事件的事件监听，是通过DOM元素的原型链查找事件对象上的原型上的addEventListener实现的监听事件
    - 5).移除事件监听，是通过removeEventListener实现的移除事件监听(标准浏览器)
    - 6).如果第三个参数不传参的话，我们默认是false，也就是在冒泡阶段发生的
    - 7).事件处理程序中的this是当前触发事件的元素对象
> 注：1.在IE浏览器中是不同的，IE中的DOM 2级事件，是通过attachEvent()实现的事件监听和detachEvent()事件移除监听，只支持两个参数，将所有事件都定义在冒泡阶段执行，IE浏览器以及IE8以前，即使是DOM 2级事件，也只是支持事件的处理函数，在冒泡阶段执行。
>    2.IE中的事件处理程序的this是window

## 四.事件流：
事件的触发执行分为三个步骤：事件捕获阶段->捕获事件源->事件冒泡阶段
事件的默认传播机制:
　　->捕获阶段:从外向内依次查找元素
　　->目标阶段:当前事件源本身的操作
　　->冒泡阶段:从内到外依次触发相关的行为(我们最常用的就是冒泡阶段)
利用事件传播机制可以实现事件委托。
 
## 五.事件对象：
```javascript
    oDiv.onclick = function(e){
        //当我们触发oDiv的click行为的时候，会把绑定的这个函数执行
        console.dir(e)
    };
    //onclick这个行为定义在当前元素的私有属性上
```
我们是把匿名函数定义的部分当做一个值赋值给oDiv的点击行为(函数表达式)
当我们触发#div的点击行为的时候，会执行对应绑定的方法。
<b>重点：</b>不仅仅要把绑定的方法执行了，而且浏览器还默认的给这个方法传递了一个参数(e)，->mouseEvent:鼠标事件对象，
+ 1.它是一个对象数据类型值，里边包含了很多的属性名和属性值，这些都是用来记录鼠标的相关信息的
+ 2.MouseEvent-->UIEvent-->Event-->Object
+ 3.MouseEvent记录的是页面中唯一一个鼠标每一次触发时候的相关信息,和到底是在哪个元素上触发的没有关系
事件对象本身的获取存在兼容问题：标准浏览器中是浏览器给方法传递的参数，我们只需要定义形参e就可以获取到;在IE6~8中浏览器不会给方法传递参数，我们如果需要的话，需要到window.event中获取查找;
e = e || window.event;
e.type：存储的是当前鼠标触发的行为类型 "click"
e.clientX/ e.clientY :当前鼠标触发点距离当前屏幕左上角的x/y轴的坐标值
e.currentTarget 保存的是当前绑定事件的对象
e.target:事件源,当前鼠标触发的是哪个元素,那么它存储的就是哪个元素，但是在IE6~8中不存在这个属性(e.target的值是undefined),我们使用e.srcElement来获取事件源
e.pageX/e.pageY：当前鼠标触发点距离body左上角(页面第一屏幕最做上端)的x/y轴的坐标，但是在IE6~8中没有这个属性,我们通过使用clientY+滚动条卷去的高度来获取也可以
e.preventDefault:阻止浏览器的默认行为,在IE6~8中没有这个方法,我们需要使用e.returnValue=false;来代替
e.stopPropagation:阻止事件的冒泡传播,在IE6~8中不兼容,使用e.cancelBubble=true来代替

<b>总结兼容写法：</b>　　　
e = e || window.event; 
eTarget = e.target || e.srcElement;
e.stopPropagation ? e.stopPropagation():e.cancelBubble = true;
e.preventDefault ? e.preventDefault() : e.returnValue = false;

## 六.mouseenter和mouseover的区别?
mouseover默认有事件的默认传播机制，而mouseenter默认阻止了事件的冒泡传播。
=>事件传播机制分为：捕获和冒泡
=>冒泡机制可以让我们采用事件委托来处理程序
->mouseenter属于进入，mouseover属于滑过，进入到容器中的子元素中，会触发父元素的mouseout和子元素的mouseover(传播到父元素上，父元素的mouseover也被触发了)；
但是mouseenter则不会这样，进入到子元素，不会触发父元素的mouseleave，子元素的mouseenter触发，但是不传播到父级容器上。
+ 1.onmouseenter 当鼠标进入, 鼠标进入,是刹那间的行为,进入只可能有一次,只要在这个绑定事件的文档内容中,都算进入.无论是子集还是自己.浏览器默认切断了整个onmouseenter的冒泡传播
+ 2.onmouseleave 当鼠标离开, 鼠标离开,也00是刹那间的行为,只要在这个绑定事件的文档内容中,无论子集还是自己,都算处于进入状态,只有彻底离开了这个盒子.才会触发mouseleave事件.浏览器默认切断了整个onmouseleave的冒泡传播

## 七.事件的相关兼容处理：
ele.addEventListener(type,fn,[bool]);  DOM 2级 绑定事件
ele.removeEventListener(type,fn,[bool]);   DOM 2级 移除事件
ele.attachEvent('on'+type,fn);   DOM 2级 IE中绑定事件
ele.detachEvent('on'+type,fn); DOM 2级 IE中移除事件

e=e||window.event;  获取事件对象
e.target=e.target||e.srcElement;  获取事件源
e.stopPropagation?e.stopPropagation():e.cancelBubble=true;  阻止事件传播
e.preventDefault?e.preventDefault():e.returnValue=false;  阻止事件冒泡
e.pageX=e.pageX||(document.documentElement.scrollLeft||document.body.scrollLeft)+e.clientX;
e.pageY=e.pageY||(document.documentElement.scrollTop||document.body.scrollTop)+e.clientY;

<b>IE6~8中DOM 2级事件绑定存在的问题：</b>
1).this指向问题，默认的this指向window；
2).重复问题，绑定事件的时候，能够重复给同一元素的同一行为，绑定同一事件处理程序；
3).顺序混乱，给元素绑定事件处理程序后，执行的先后顺序与绑定的先后顺序不一致；
