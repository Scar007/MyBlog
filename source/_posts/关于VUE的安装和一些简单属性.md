---
title: 关于VUE的安装和一些简单属性
subtitle: vue
date: 2018-3-7
categories: VUE
tags: 
    - VUE
---
## 安装vue
+ 安装前初始化package.json 主要用来描述自己的项目,记录安装过得文件有哪些,在当前文件夹下生产json
+ 安装vue
    + --save(-S)代表项目依赖
    + --save-dev(-D)代表开发依赖
```
npm info vue
npm install vue --save
npm install //跑环境,将package中的依赖全部安装
```
+ 安装后默认会生产\node_modules文件夹
    + 上传到git上\node_modules是忽略掉的,拉下代码后,需要重新\npm install安装依赖


## Vue属性

#### el
+ 指定的元素不能是html和body
+ 使用querySelector

#### data
+ 可以使用vm来代理data属性,vm是Vue的实例
+ 在html中使用的属性必须先声明,不能新增不存在的属性,使用Object.defineProperty,vue不兼容IE8以及下包IE8

#### 小胡子语法（双层大括号）
+ 这是v-text的缩写,直接再标签上使用v-text可以避免出现大括号闪烁
+ 括号内可以使用三元表达式
+ 最终的结果需要有返回值,可以赋值,运算
+ 等待数据加载好后,将内容插入到div中

#### v-model
+ 只能绑定定变量,先将对应的数据放到输入框的value值上
+ 会监听输入框的输入事件 oninput,将值放回到数据中

#### v-once
+ 在标签中使用,变量在内容中调取
+ 被这个属性包住的属性,就不会再发生变化,只绑定一次

#### v-html
+ 包住的内容会被解析成htlm结果

#### method
+ 定义函数的一个对象,放所有函数
+ 函数中的this为vm,必须使用普通函数,箭头函数存在this指向问题

#### v-on
+ 事件绑定函数传参会丢失event事件源,可以手动传递属性$event
+ v-on可以简写成@
+ 修饰符,可以连续嵌套
    + ".stop",可以阻止默认事件的冒泡
    + ".capture",捕获阶段
    + ".self",自己,只在自己身上触发
    + ".once",只触发一次
    + ".prevent",阻止默认行为
    + ".13",".enter",点击回车时触发      
        + 可以使用键盘修饰符,支持keyCode和名称,只能用于@keyup和@keydown

#### v-for
+ 写在要循环的子标签中
+ 数组中a in xxx,那么a的值是数组里的value
+ 对象中f in xxx,那么f的值是对象里的value
+ (value,key) in xxx,可以取到属性值,属性名

#### v-if/v-else-if/v-else
+ 在标签中使用
+ 判断结果为true的时候才增加,false删除操作的是DOM,会影响性能
```html
<div v-if=""></div>
<div v-else-if=""></div>
<div v-else=""></div>
```

#### v-show
+ 同样在标签中使用
+ 但不操作DOM,只操作样式,判断结果为true时,display:block,false时,display:none
```html
<div v-show=""></div>
<div v-show=""></div>
<div v-show=""></div>
```

+ 与v-if的使用情景需要判断
    + v-if有阻断作用,当外面的条件不满足是,内部代码不在执行    
    + 频发显示隐藏,则选择v-show
+ 如果想要将内容绑定在标签上,绑定的是动态属性,需要使用":",不能使用小胡子语法（双层大括号）
    + style和class可以使用绑定对象,或者数组的方式
    + 其他的样式直接使用":"+样式名="属性名"
    + class和原生的不冲突,冒号会覆盖普通的class
```html
<div :style="[s1,s2]"></div>  <!--data中的属性名,支持字符串和对象属性-->
<div :style="s1"></div>  <!--data中的属性名,支持字符串和对象属性-->
<div :class="['color','background']"></div> <!--类名-->
<div :class="{'color':true,'background:true'}"></div>  <!--类名-->
