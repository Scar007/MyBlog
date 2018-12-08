---
title: jQuery中$(document).ready()和window.onload的区别
subtitle: jq
date: 2015-5-27
categories: jQuery
tags:

---
document.ready 和 window.onload的区别？（JQ中$(document).ready()和window.onload的区别？）
<b>window.onload</b>，是采用DOM0级事件绑定监听的load事件
+ 1） load事件本身就是当所有资源加载完成才会触发，window.onload指的是浏览器中的资源文件(HTML结构、图片、文字、音视频...)加载完成就会被触发
+ 2） DOM0绑定方式决定了他只能绑定一个方法，绑定多个，后面把前面会覆盖掉
<b>$(document).ready()</b>是JQ拿原生JS封装好的方法，等同于$(function(){}),JQ在封装这个方法的时候，采用DOM2级事件绑定，而且监听的是DOMContentLoaded(在IE6~8下监听的是onreadystatechange事件)；
+ 1） DOMContentLoaded事件本身就是DOM结构渲染完成就会被触发，所以只要页面中的HTML结构加载完成，就会触发对应的事件，把绑定的方法执行
+ 2） 采用DOM2绑定，所以可以绑定多个方法








