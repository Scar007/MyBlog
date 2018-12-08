---
title: Swiper+ ejs模板引擎+ iScroll插件知识总结
subtitle: swiper
date: 2017-2-11
categories: 移动端
tags:
    - Swiper
    - ejs
    - iScroll
---
## 一. Swiper
[Swiper中文网](http://www.swiper.com.cn/#)
> swiper是一个应用于移动端的动画插件,原理类似于轮播图

```html
    <div class="swiper-container">
        <div class="swiper-wrapper">
            <div class="swiper-slide">Slide 1</div>
            <div class="swiper-slide">Slide 2</div>
            <div class="swiper-slide">Slide 3</div>
        </div>
        
        <!-- 如果需要分页器 -->
        <div class="swiper-pagination"></div>
        
        <!-- 如果需要导航按钮 -->
        <div class="swiper-button-prev"></div>
        <div class="swiper-button-next"></div>
        
        <!-- 如果需要滚动条 -->
        <div class="swiper-scrollbar"></div>
    </div>  
```
  
导航等组件可以放在container之外

+ 引入css样式文件和js文件
+ 初始化
    + 创建Swiper的实例
    ```javascript
        var mySwiper = new Swiper ('.swiper-container', {
            direction: 'vertical',
            loop: true,
        
            // 如果需要分页器
            pagination: '.swiper-pagination',
        
            // 如果需要前进后退按钮
            nextButton: '.swiper-button-next',
            prevButton: '.swiper-button-prev',
        
            // 如果需要滚动条
            scrollbar: '.swiper-scrollbar',
        })
    ```
    + 参数对象中使用特定的属性名和属性值,即可调用对应效果
    + AJAX异步请求中,swiper的初始化要在页面内容成功加载完成后进行
    
## 二. ejs模板引擎
> ejs模板引擎与es6模板引擎一样,可以一次性的改变html结构,而不会引起多次的回流和重绘

+ 引入ejs.js文件
    + 原理上,通过解析模板字符串来创建一个有效的html结构字符串,再一次性的添加到指定的元素当中

+ 在html中编写对应的模板
    + script类型为text/template
    + 赋予特定的ID值
    + 所有的js代码必须使用<%%>包起来
        + 代码以行块的形式被解析,也就是一行中仅有js代码,那么整行都要用<%%>包起来
    + 变量引入时,使用<%=变量%>来调用变量
+ 在js中调用编写好的模板
    + 通过id值获取模板对应html内容
    + 使用ejs.render()方法来解析模板字符串            
        + 第一个参数为对应的模板字符串
        + 第二个参数为一个对象,通过属性名为模板中的变量名,属性值为js中的变量名,来实现变量的传递
        + 返回值是解析完毕的模板字符串
    + 将上述返回的模板字符串添加到指定的DOM元素中即可

## 三. iScroll插件
> iScroll是一款移动端使用的滚动插件,能实现上拉加载和下拉加载等特定的功能
    
[iScroll 4.2.5 中文API](http://www.360doc.com/content/14/0724/11/16276861_396699901.shtml)

+ 引入对应的iScroll.js文件
+ 初始化
    + 创建iScroll的实例
        + 第一个参数为指定的DOM元素或者ID值
        + 第二个参数为参数对象
            ```javascript
                  var myIScroll=new iScroll("wrapper",{
                      vScroll:true,
                      momentum:true
                  
                  })
            ```
    + 参数设置
        + hScroll/vScroll:是否允许水平/垂直滚动
        + bounce:是否超过实际位置反弹
        + bounceLock:当内容少于滚动是否可以反弹
        + momentum:是否开启拖动惯性
        + lockDirection:当水平或垂直拖动时是否锁定另一边的拖动
        + useTransform:是否使用CSS变形
        + useTransition:是否使用CSS变换
        + checkDOMChanges:是否自动检测内容变化
        + topOffset:已经滚动的基准值(一般用在拖动刷新)
        + x:滚动水平初始位置（负值）
        + y:滚动垂直初始位置（负值）
        + Scrollbar的相关参数    
            + hScrollbar/vScrollbar:是否显示水平/垂直滚动条
            + fixedScrollbar:在iOS系统上，当元素拖动超出了scroller的边界时，滚动条会收缩，设置为true可以禁止滚动条超出scroller的可见区域
            + hideScrollbar:是否隐藏滚动条
            + fadeScrollbar:滚动条是否渐隐渐显
        + Zoom放大相关的参数
            + zoom:是否放大
            + zoomMin:放大的最小倍数
            + zoomMax:放大的最大倍数
            + doubleTapZoom:双击放大倍数
        + Function 自定义函数
            + onRefresh:refresh的回调
            + onScrollStart:开始滚动的回调
            + onBeforeScrollMove:在内容移动前的回调
            + onScrollMove:内容移动的回调
            + onBeforeScrollEnd:在滚动结束前的回调
            + onScrollEnd:在滚动完成后的回调
            + onTouchEnd:手离开屏幕后的回调
            + onDestroy:销毁实例的回调
            + onZoomStart:放大开始时的回调
            + onZoom:放大的回调
            + onZoomEnd:放大结束后的回调
            
            
            
            