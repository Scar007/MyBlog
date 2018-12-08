---
title: 关于js盒子模型的知识梳理
subtitle: boxModel
date: 2015-9-6
categories: JS
tags:
    - 盒子模型
---

## JS盒子模型中的13个常用属性：
+ clientWidth/clientHeight：可视区域的宽高，宽高+PADDING组成
+ clientTop/clientLeft：上边框和左边框的宽度
+ offsetWidth/offsetHeight：clientWidth/clientHeight+border
+ offsetParent：父级参照物，和父级元素没有必然关系，默认元素的父级参照物都是BODY，如果设置定位后，可以改变默认的父级参照物
+ offsetLeft/offsetTop：当前元素的外边框距离父级参照物内边框的偏移(左/上) =>IE8下是当前元素的外边框距离父级参照物的外边框算作偏移
+ scrollWidth/scrollHeight：当前元素真实内容的宽度和高度，在没有内容溢出的情况下，和clientWidth/clientHeight的值一模一样；但是在内容有溢出的情况下，scrollHeight获取的值是上padding+真实内容的高度(由于浏览器对于字体等渲染不太一样，所以我们获取的结果都是约等于的值，并且是否这是overflow:hidden也会对结果产生影响)
+ scrollTop/scrollLeft：当前区域如果有内容溢出，并且我们也让其出现了滚动条，那么这两属性获取的即使滚动条卷去的宽度和高度
    - 1、scrollTop/scrollLeft是13个属性中唯一可以进行读写的，其余的都是只读的
    - 2、通过JS盒子模型的属性获取到的结果都是整数(它默认会对小数值进行四舍五入，而且获取的值是不带单位的)
    - 3、如果操作的是整个页面(不管是获取还是设置)，想要兼容的话，需要写两套： document.documentElement.xxx=xxx; document.body.xxx=xxx;
    - 4、JS中的盒子模型，获取的样式值都是复合值，如果想获取单独某一个具体的样式值(例如:我就想获取width或者paddingLeft)使用这13个属性就不行了

获取当前元素的具体样式属性值
```javascript
/*
     * getCss：Gets the value of the specific style property for the current element
     * @parameter：
     *   curEle[object]：current element
     *   attr[string]：style properties of elements
     * @return：
     *   Style attribute values for elements
     * By Team on 2017-04-23 12:29
     */
    function getCss(curEle, attr) {
        var val = null,
            reg = null;
        if ('getComputedStyle' in window) {
            val = window.getComputedStyle(curEle, null)[attr];
        } else {
            //->IE6~8
            switch (attr) {
                case 'filter':
                case 'opacity':
                    val = curEle.currentStyle['filter'];
                    reg = /alpha\(opacity=(.+)\)/i;
                    val = reg.test(val) ? RegExp.$1 / 100 : 1;
                    break;
                default:
                    val = curEle.currentStyle[attr];
            }
        }
        reg = /^-?\d+(\.\d+)?(px|rem|em|pt)$/i;
        val = reg.test(val) ? parseFloat(val) : val;
        return val;
    }
```
## JS中浏览器兼容检测的三大方式
+ 使用TRY、CATCH
```javascript
    try{
        //执行的JS代码
    }catch(e){
        //如果上述的代码执行出现了问题，则进入catch中执行
    }
    前提条件：
        - 需要保证不兼容的代码，在浏览器中执行的时候，抛出异常信息，这样才会进行catch捕获
        - 性能不太好，不管是否兼容，都需要把try中的代码执行一下才可以
```

+ 当前浏览器信息监测来判断是否兼容
```javascript
    var curInfo = window.navigator.userAgent;  //获取浏览器的基本信息（包括类型和版本号）
    if (curInfo.indexOf('MSIE 8') === -1) {
        //非IE8浏览器
    }

    使用这种方式检测兼容，首先我们需要知道的是有哪些浏览器不兼容，这样才可以书写条件信息
``` 

+ 验证当前即将使用的属性和方法是否在当前对象中
```javascript
//方案1
if (window.getComputedStyle) {
    //兼容：首先获取属性值，如果兼容，是一个具体值，不兼容的结果是undefined，undefined代表False，条件不成立
}

//方案2
if (typeof window.getComputedStyle !== "undefined") {
    //兼容：在上一个基础上稍作修改
}

//方案3
if ("getComputedStyle" in window) {
    //兼容
}
```



