---
title: JS实现动画的四条优化方法
subtitle: animadesign
date: 2016-2-8
categories: JS
tags:
    - 动画
    - 优化
---

####  1)如果使用的是setTimeout实现的轮询动画,在每一次执行方法之前需要把前面的设置的定时器清除掉
```javascript
    var timer=null;
    function move(){
        window.clearTimeout(timer);
          //<js code>
        timer=window.setTimeout(move,15);
    }
    move();
```

#### 2)为了防止全局变量的污染,我们把定时器的返回值赋值给当前操作元素的自定义属性;这样做还有一个好处,就是如果当前动画没有完成,执行了下一个动画,由于我们每一次都是给自己的自定义属性,那么下一个动画开始的时候默认的把当前的动画的结束了;
```javascript
    function move(){
        window.clearTimeout(curEle.timer);
        //<js code>
        curEle.timer=window.setTimeout(move,15);
    }
    move();
```
 
####  3)关于作用域累积的问题->在move中编写一个_move来执行我们的动画操作,_move里面不需要传递参数,每一次都用move中存储下来的值即可
```javascript
    function move(target){
            ~function _move(){
               window.clearTimeout(curEle.timer);
               // <js code>
               curEle.timer=window.setTimeout(_move,15);
               //_move这里不建议使用arguments.callee因为在严格模式下不兼容这个属性,而写在项目中一般都是需要使用严格模式编写代码的
            }();
        }
        move();
```

#### 4)为了防止走一步超了,不走还到不了边界,我们在做边界判断的时候需要加上步长来做
```javascript
function move(target){
        var step=10;

        ~function _move(){
           window.clearTimeout(curEle.timer);

           var curL=utils.getCss(curEle,"left");
           if(curL+step>=target){
              utils.setCss(curEle,"left",target);
              return;
           }
           utils.setCss(curEle,"left",curL+step);

           curEle.timer=window.setTimeout(_move,15);
        }();
    }
    move();
```



