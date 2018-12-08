---
title: 使用javascript实现图片上下切换效果并且实现顺序循环播放の案例
subtitle: cutimg
categories: 案例
date: 2015-7-22
---
```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>图片切换</title>
    <style>
        .content{
            height: 500px;
            width: 500px;
            text-align: center;
            margin:0 auto;
        }
        .playBtn button,.showPages span,.upDownBtn button{
            margin: 20px;
        }
        .img{
            position: relative;
            border:1px solid #430d06;
            height: 350px;
            width: 500px;
        }
        .img img{
            width: 100%;
            height: 100%;
        }
        .img p{
            z-index: 1;
            margin: 0;
            position: absolute;
            bottom:0;
            height: 40px;
            width: 100%;
            font-size: 23px;
            line-height: 40px;
            color: #fff;
            background-color: #000;
            opacity: 0.2;
        }

    </style>
</head>
<body>
<div class="content">
    <div class="playBtn">
        <button id="order">顺序播放</button>
        <button id="loop">循环播放</button>
    </div>
    <div class="img">
        <img src="img/3.jpg" alt="" id="oImg">
        <p id="imgP"></p>
    </div>
    <div class="showPages">
        <span id="showPages"></span>
    </div>
    <div class="upDownBtn">
        <button id="up">上一张</button>
        <button id="down">下一张</button>
    </div>
</div>
<script>
    //首先一组图片放在数组里
    var imgAry=['img/1.jpg','img/2.jpg','img/3.jpg','img/4.jpg','img/5.jpg','img/6.jpg'];

    //用原生js获取元素(当然这些都是基础，看你自己喜欢用原生还是JQ了)
    var up=document.getElementById("up");
    var down=document.getElementById("down");
    var oImg=document.getElementById("oImg");
    var showPages=document.getElementById("showPages");
    var imgP=document.getElementById("imgP");
    var order=document.getElementById("order");
    var loop=document.getElementById("loop");

    //设置顺序还是循环播放的开关，这里默认是顺序播放(true),那循环播放就是false了
    var onOff=true;
    //点击进入顺序播放
    order.onclick=function () {
        onOff=true;
    };
    //点击进入循环播放
    loop.onclick=function () {
        onOff=false
    }

    //设置一个初始值作用相当于前边那个imgAry数组的index值一样
    var n=0;
    //点击跳转前一张图片
    up.onclick=function () {
        //找对应的索引值，向上所以就是索引-1，n--跟n-1一样的
        n--;

        //第一张临界点判断处理
        if(n<0){
           // 判断是顺序还是循环播放
            if(onOff){
                //这里边走的是顺序播放，顺序的时候在临界点时让其索引等于临界点的值就行了，顺便给个提示
                n=0;
                alert("已经是第一张了")
            }else {
                //这里是循环播放，将临界点的索引设置为最后一站的索引即可，即 倒叙
                n=imgAry.length-1;
            }
        }
        //再将公共组件执行赋值渲染即可
        commontComponent();
    };

    //向下和向上的逻辑是一样的，就颠倒一下就好，这里就不再详细解释了
    down.onclick=function () {
        n++;
        if(n>=imgAry.length){
            if(onOff){
                n=imgAry.length-1;
                alert("已经是最后一张了")
            }else {
                n=0;
            }
        }
        commontComponent();
    };

    //渲染的页面公共组件
    function commontComponent() {
        //这里是根据索引查找对应的图片并赋值
        oImg.src=imgAry[n];

        //这是图片上的提示文字
        /*强调一下为什么是n+1：因为n是从0开始的，直接显示0不符合人们阅读时的正常逻辑，所以+1好一点*/
        imgP.innerHTML='这是第'+(n+1)+'张图片';

        //这里是图片下边分页的显示，n+1同理
        showPages.innerHTML=n+1+'/'+imgAry.length;
    }
    commontComponent();
</script>
</body>
</html>
```
下边这个是一个简单效果展示，具体功能可以自己试验
<img src="https://github.com/Scar007/myBlogTest/blob/master/blog/source/images/dog.gif?raw=true"></img>


