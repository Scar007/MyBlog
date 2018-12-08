---
title: JS实现弹性势能效果(弹力球效果[实现插件封装])
subtitle: ball
date: 2017-8-6
categories: 案例
---
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>弹性势能动画(弹力球效果)</title>
    <style type="text/css">
        * {
            margin: 0;
            padding: 0;
        }

        #box {
            position: absolute;
            top: 0;
            left: 0;
            width: 100px;   
            height: 100px;
            font-size: 30px;
            text-align: center;
            line-height: 100px;
            background-color: greenyellow;
            border-radius: 50%;
            cursor: move;
            user-select: none;
        }
    </style>
</head>
<body>
<div id="box">弹</div>

<script type="text/javascript">
    var oDiv = document.getElementById('box');
    oDiv.addEventListener('mousedown', down, false);
    function down(e) {
        this.startX = e.clientX;
        this.startY = e.clientY;
        this.startL = this.offsetLeft;
        this.startT = this.offsetTop;

        var _this = this;
        this.MOVE = function (e) {
            move.call(_this, e);
        };
        this.UP = function (e) {
            up.call(_this);
        };
        document.addEventListener('mousemove', this.MOVE, false);
        document.addEventListener('mouseup', this.UP, false);

        window.clearInterval(this.flyTimer);
        window.clearInterval(this.dropTimer);
    }
    function move(e) {
        var curL = e.clientX - this.startX + this.startL,
                curT = e.clientY - this.startY + this.startT;

        var minL = 0, minT = 0, maxL = (document.documentElement.clientWidth || document.body.clientWidth) - this.offsetWidth, maxT = (document.documentElement.clientHeight || document.body.clientHeight) - this.offsetHeight;

        curL = curL < minL ? minL : (curL > maxL ? maxL : curL);
        curT = curT < minT ? minT : (curT > maxT ? maxT : curT);
        this.style.left = curL + 'px';
        this.style.top = curT + 'px';

        if (!this.pre) {
            this.pre = this.offsetLeft;
        } else {
            this.speedFly = this.offsetLeft - this.pre;
            this.pre = this.offsetLeft;
        }
    }
    function up() {
        document.removeEventListener('mousemove', this.MOVE, false);
        document.removeEventListener('mouseup', this.UP, false);

        fly.call(this);

        drop.call(this);
    }

    function fly() {
        var _this = this;
        _this.flyTimer = setInterval(function () {
            if (Math.abs(_this.speedFly) < 0.5) {
                clearInterval(_this.flyTimer);
                return;
            }
            _this.speedFly *= 0.98;
            var curL = _this.offsetLeft + _this.speedFly;
            var minL = 0, maxL = (document.documentElement.clientWidth || document.body.clientWidth) - _this.offsetWidth;
            if (curL <= minL) {
                _this.style.left = 0;
                _this.speedFly *= -1;
            } else if (curL >= maxL) {
                _this.style.left = maxL + 'px';
                _this.speedFly *= -1;
            }
            else {
                _this.style.left = curL + 'px';
            }
        }, 10);
    }

    function drop() {
        var _this = this;
        _this.dropFlag = 0;
        _this.dropTimer = setInterval(function () {
            if (_this.dropFlag > 1) {
                clearInterval(_this.dropTimer);
                return;
            }
            !_this.dropSpeed ? _this.dropSpeed = 10 : _this.dropSpeed += 10;
            _this.dropSpeed *= 0.98;
            var curT = _this.offsetTop + _this.dropSpeed;
            var maxT = (document.documentElement.clientHeight || document.body.clientHeight) - _this.offsetHeight;
            if (curT >= maxT) {
                _this.style.top = maxT + 'px';
                _this.dropSpeed *= -1;
                _this.dropFlag++;
            } else {
                _this.style.top = curT + 'px';
                _this.dropFlag = 0;
            }
        }, 10);
    }
</script>
</body>
</html>
```







