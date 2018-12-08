---
title: JS实现 穿墙效果
subtitle: wall
date: 2016-8-28
categories: 案例
---
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>判断鼠标移入移出方向</title>
    <style type="text/css">
        * {
            margin: 0;
            padding: 0;
        }

        .outer {
            width: 400px;
            height: 400px;
            border: 2px solid orange;
            margin: 100px auto;
            overflow: hidden;

        }

        .outer div {
            width: 100%;
            height: 100%;
            background-color: yellow;
            opacity: 0.5;
            display: none;
        }
    </style>
</head>
<body>
<div id="box" class="outer">
    <div id="mask" style="left: 0;top:0;"></div>
</div>

<script type="text/javascript">

    var oDiv = document.getElementById('box');
    var oMask = document.getElementById('mask');

    oDiv.disL = oDiv.offsetLeft;
    oDiv.disT = oDiv.offsetTop;
    oDiv.disR = oDiv.disL + oDiv.offsetWidth;
    oDiv.disB = oDiv.disT + oDiv.offsetHeight;

    oDiv.addEventListener('mouseenter', moveIn, false);
    oDiv.addEventListener('mouseleave', moveOut, false);


    function moveIn(e) {
        var resL = Math.abs(e.clientX - oDiv.disL),//距离左边
                resT = Math.abs(e.clientY - oDiv.disT),//距离上边
                resR = Math.abs(e.clientX - oDiv.disR),//距离右边
                resB = Math.abs(e.clientY - oDiv.disB);//距离下边
        var min = Math.min(resL, resB, resR, resT);

        //console.log(resL, resB, resR, resT, min);
        if (min === resL) {
            console.log('左边移入');
            maskIn('left');
        } else if (min === resT) {
            console.log('上边移入');
            maskIn('top');
        } else if (min === resR) {
            console.log('右边移入');
            maskIn('right');
        } else {
            console.log('下边移入');
            maskIn('bottom');
        }
    }
    function moveOut(e) {
        var resL = Math.abs(e.clientX - oDiv.disL),//距离左边
                resT = Math.abs(e.clientY - oDiv.disT),//距离上边
                resR = Math.abs(e.clientX - oDiv.disR),//距离右边
                resB = Math.abs(e.clientY - oDiv.disB);//距离下边
        var min = Math.min(resL, resB, resR, resT);

        //console.log(resL, resB, resR, resT, min);
        if (min === resL) {
            //console.log('左边移出');
            maskOut('left');
        } else if (min === resT) {
            //console.log('上边移出');
            maskOut('top');
        } else if (min === resR) {
            //console.log('右边移出');
            maskOut('right');
        } else {
            //console.log('下边移出');
            maskOut('bottom');
        }

    }
    function maskIn(direction) {
        var attr = 'marginTop', otherAttr = 'marginLeft', step = -15, maxDis = 'offsetHeight';
        if (direction === 'left' || direction === 'top') {
            step = 15;
        }
        if (direction === 'left' || direction === 'right') {
            attr = 'marginLeft';
            maxDis = 'offsetWidth';
            otherAttr = 'marginTop';
        }
        clearInterval(oMask.timer);
        step < 0 ? oMask.style[attr] = oDiv[maxDis] + 'px' : oMask.style[attr] = -oDiv[maxDis] + 'px';
        oMask.style[otherAttr] = 0;
        oMask.style.display = 'block';
        oMask.timer = setInterval(function () {
            var disL = parseFloat(oMask.style[attr]) + step;
            if (step > 0) {
                if (disL >= 0) {
                    oMask.style.marginLeft = 0;
                    oMask.style.marginTop = 0;
                    clearInterval(oMask.timer);
                } else {
                    oMask.style[attr] = disL + 'px';
                }
            } else {
                if (disL <= 0) {
                    oMask.style.marginLeft = 0;
                    oMask.style.marginTop = 0;
                    clearInterval(oMask.timer);
                } else {
                    oMask.style[attr] = disL + 'px';
                }
            }

        }, 10);
    }

    function maskOut(direction) {
        clearInterval(oMask.timer);
        oMask.style.marginLeft = 0;
        oMask.style.marginTop = 0;
        var attr = 'marginTop', step = 15, maxDis = 'offsetHeight';
        if (direction === 'left' || direction === 'top') {
            step = -15;
        }
        if (direction === 'left' || direction === 'right') {
            attr = 'marginLeft';
            maxDis = 'offsetWidth';
        }
        oMask.timer = setInterval(function () {
            var disL = parseFloat(oMask.style[attr]) + step;
            if (step < 0) {
                if (disL <= -oDiv[maxDis]) {
                    oMask.style.display = 'none';
                    clearInterval(oMask.timer);
                } else {
                    oMask.style[attr] = disL + 'px';
                }
            } else {
                if (disL >= oDiv[maxDis]) {
                    oMask.style.display = 'none';
                    clearInterval(oMask.timer);
                } else {
                    oMask.style[attr] = disL + 'px';
                }
            }
        }, 10);
    }
</script>
</body>
</html>
```

