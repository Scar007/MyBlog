---
title: DOM相关知识总结
subtitle: dom
date: 2015-10-14
categories: JS
tags:
    - DOM
---
## 1.获取DOM元素
+ document.getElementById
+ document.getElementsByName
+ document.getElementsByTagName
+ document.getElementsByClassName
+ document.documentElement
+ document.body
+ document.querySelector
+ document.querySelectorAll
## 2.DOM节点

/  | 	nodetype  |	nodeName  |	nodeValue
---- | ------- | ------- | ------- | ---
元素节点  |	1  |	大写的标签名	|  null
文本节点  |	3  |	#text  |	文本内容
注释节点  |	8  |	#comment  |	注释内容
document  |	9  |	#document  |	null

#### nodeType:
如果节点是元素节点，则 nodeType 属性将返回 1。
如果节点是属性节点，则 nodeType 属性将返回 2。
document.body.nodeType;

## 3.DOM节点属性
+ parentNode // document.getElementById("item1").parentNode;
+ childNodes // ele.childNodes; 获得 ele(当前) 元素的子节点集合,它会把空的文本节点当成节点
+ children //ele.children; 只获得元素节点
+ firstChild (firstElementChild) // ele.firstChild 返回首个子节点
+ lastChild (lastElementChild)
+ previousSibling (previousElementSibling) // ele.previousSibling 返回上一个元素节点
+ nextSibling (nextElementSibling) // ele.nextSibling 返回下一个元素节点

## 4.DOM操作
+ document.createElement('p');//创建节点
+ box.appendChild(oDiv);//添加节点
+ box.removeChild(oDiv);// 删除节点
+ box.parentNode.insertBefore(oDiv,box); //把新创建的元素添加到指定元素前面
+ oDiv.replaceChild(oP,oDiv);// oP替换oDiv
+ console.log(oDiv.cloneNode(true)); //深度克隆 不传参数默认是false只克隆oDiv这个标签 //如果参数是 true 会把里面的标签也克隆一份
+ box.setAttribute('name','zhufeng'); // 添加属性
+ console.log(box.getAttribute('name')); // 获取属性
+ box.removeAttribute('name') // removeAttribute 移除属性


