---
title: jQuery的实现原理和核心
subtitle: axiom
date: 2015-04-27
categories : jQuery
---
jquery-1.xxx ：专门为PC端诞生的类库，兼容所有的浏览器 
jquery-2.xxx：当初是为了移动端而准备的，所以IE低版本浏览器一般不兼容，但是这个版本针对移动端的事件等操作也不是特别的完善，被Zepto这个类库取代了 
jquery-3.xxx：可以自己扩展一下，看看新增加或者修改了哪些方法
[JQ的API中文速查:](http://jquery.cuishifeng.cn/) 

## jQuery充分利用了JS中函数的三种特性：普通函数、类、普通对象；jQuery就是这个类；
```javascript
    (function(window,undefined) {
        /*
            selector:jQuery选择器内容
            context：获取元素上下文
           
        */
        var version = "1.11.3",
        jQuery = function(selector,context) {
            return new jQuery.fn.init(selector,context);
        }
        
        jQuery.fn = jQuery.prototype = {
            ...
            addClass:...,
            init:function (selector,context) {
                
            }
        }
        jQuery.ajax = function...
        window.jQuery = window.$ = jQuery;
    })();
    
    jQuery();  <==> $()
    $().addClass()
    $.ajax()
```

## 1.jQuery的实现原理
JQ本身就是一个类，在外面使用的$和jQuery是同一个东西，JQ中提供的方法分为两部分：
+ 写在jQuery原型上的方法，专门给JQ的实例使用
+ 写在jQuery私有属性上的，通过$.xxx可以获取进行操作等 jQuery() / $() =>创建JQ的实例，需要传递两个参数，第一个参数一般是选择器内容；第二个参数是获取的上下文，如果不传递默认是document；==>“此操作通俗的叫法：通过JQ选择器获取元素”
+ 返回结果是一个类数组(它也是JQ的实例)，这个类数组是JQ自己去创建的，里面有一些自己特定的属性：length/context/selector/prevObject…
+ 获取到的结果我们叫做JQ对象(JQ实例)，可以调取JQ原型上提供的方法，但是它不是原生的JS对象，不能调取浏览器提供的默认属性方法，当然原生JS也不能调用JQ上提供的属性方法；

extend：在JQ的私有属性上和它的原型上都有这个方法，但是指向的都是同一个方法：jQuery.extend = jQuery.fn.extend = function(){} 
$.extend() 
$.fn.extend() 
虽然执行的是同一个方法，但是方法中的this是不一样的，extend是向现有的方法库中扩展方法的意思，不同的执行方式扩展的位置不一样
```javascript
var jQuery = function () {
    return new jQuery.fn.init(selector,context); 
}
```
+ 1）jQuery 采用的是构造函数模式进行开发，jQuery是一个类
+ 2）上边说的常用的方法（CSS、属性、筛选、事件、动画、文档处理）都是定义在jQuery.prototype上的，只有jQuery的实例才能使用这些方法

## 2.选择器/筛选
+ 1） 我们的选择器其实就是创造jQuery类的一个实例 -> 获取页面中元素用的jQuery(); ->$()
    $()就是jQuery的选择器，就是创建jQuery这个类的一个实例
+ 2） 执行的时候需要传递两个参数
        ```javascript
            select -> 选择器的类型 一般都是“string”类型
            context -> 获取的上下文 第二个参数一般不传，不传默认为document
            $("#div1");
            $(".box");
            $("div1 span") ->$("span",div1)
            console.log($("#div1 span;first"));
        ```
+ 3) 通过选择器获取的是一个jQuery类的实例 -> jQuery 对象
 ```javascript
 console.log($("div1"));
 ```
 [jQuery对象的私有属性]
 $("#div1")[0] -> div1 这个元素对象
 $("div1").selector -> "#div1"
 $("div1").context -> document
 $("div1").length -> 1 获取元素的个数

 [jQuery对象的公有属性]
 jQuery.prototype
 
+ 4)我们获取的是jQuery对象(他是jQuery的实例)不是我们的原生js对象
```javascript
    jQuery:$("#div1")
    JS:document.getElementById("div1")  //原生JS的对象不能直接的使用jQuery的方法，同理，jQuery的对象也不能使用原生js的方法
    $("#div1").className = "box";  //no
    document.getElementById("div1").addClass();
```
        
+ 5)互相转化
```javascript   
 var $oDiv = $("#div1");
 var oDiv = document.getElementById("div1");
 JS -> jQuery : $(oDiv).addClass()
  jQuery -> JS:$oDiv[0] / $oDiv.get(0)
```
 
## 核心
```javascript
    $(document).ready(function(){
        //HTML结构加载完成就执行这里的代码
    });
    $(function(){

    });
```
 ```javascript
    each
    $("selector").each(function(){})   //遍历获取的元素 jQuery.prototype
    $.each(ary)  //遍历数组中的每一项 jQuery.each 
```

我们的jQuery不仅仅是一个类（在它的原型上定义了很多方法，每一个jQuery的实例都可以使用方法），它还是一个普通的对象，在jQuery/本身的属性中还增加了一系列方法：Ajax、each、工具
$.unique(ary)
$.ajax()

```javascript
    $.extend() //->把jQuery当做一个对象，给它扩展属性和方法 -> 完善类库
    $.fn.extend() //-> 在jQuery的原型上扩展属性和方法 -> 编写jQuery插件
    $.extend({
        a:function(){

        }
    });
    $.a();

    $.fn.extend({
        b:function(){

        }
    });
    $().b();
```

```javascript
    var jQuery = function(selector,context) {
        return new jQuery.fn.init(selector,context);
    }
    
    jQuery.fn = jQuery.prototype = {
        jquery:version,
        constructor:jQuery
        ...
    }
    
    init = jQuery.fn.init = function(selector,context) {}
    init.prototype = jQuery.fn;
```
定义原型扩展和工具包扩展的方法
```javascript
    jQuery.extend = jQuery.fn.extend = function() {}
    window.jQuery = window.$ = jQuery;
```




