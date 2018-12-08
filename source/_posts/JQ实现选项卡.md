---
title: JQ实现选项卡(jQuery原型插件扩展)
subtitle: tab
date: 2016-12-23
categories: 案例
---

#### 下边分为两个版本，一种是点击切换选项(index.js)，一种是滑过切换选项(index1.js)
(jq使用jquery-1.11.3.js版本)

HTML文件：
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>jQuery选项卡</title>
    <link rel="stylesheet" href="css/reset.min.css"/>
    <link rel="stylesheet" href="css/index.css"/>
</head>
<body>
<div class="container">
    <!--<span class="select">选项一</span><span>选项二</span><span>选项三</span>-->
    <ul class="tip clear">
        <li class="cur">选项一</li>
        <li>选项二</li>
        <li>选项三</li>
    </ul>
    <div class="contain cur">页卡一</div>
    <div class="contain">页卡二</div>
    <div class="contain">页卡三</div>
</div>
<div class="container">
    <!--<span class="select">选项一</span><span>选项二</span><span>选项三</span>-->
    <ul class="tip clear">
        <li class="cur">选项一</li>
        <li>选项二</li>
        <li>选项三</li>
    </ul>
    <div class="contain cur">页卡一</div>
    <div class="contain">页卡二</div>
    <div class="contain">页卡三</div>
</div>
<script src="js/jquery-1.11.3.js"></script>
<!--jQuery原型上扩展的插件-->
<script src="js/index.js"></script>
<script type="text/javascript">
    /*$('.container').on('click', '.tip>li', function () {
        $(this).addClass('cur').siblings().removeClass('cur')
                .parent().siblings('.con').removeClass('cur')
                .eq($(this).index()).addClass('cur');
    });*/
 
 
    //事件委托：利用事件冒泡传播机制
    // 模拟选项卡的ajax请求
 
    /*$('.tab>.tip>li').mouseover(function () {
        var curIndex = $(this).index();
        // $(this).addClass('cur').siblings().removeClass('cur')
        //     .parent().siblings('.contain')
        //     .eq(curIndex).addClass('cur')
        //     .siblings().removeClass('cur');
 
        $(this).addClass('cur').siblings().removeClass('cur')
                .parent().siblings('.contain')
                .each(function (index, item) {
                    //->this:item
                    curIndex === index ? $(this).addClass('cur') : $(this).removeClass('cur');
                });
    });*/
</script>
</body>
</html>
```
index.css
```css
@charset "utf-8";
.container {
    width: 600px;
    height: 400px;
    margin: 20px auto;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}
 
.container .tip li {
    float: left;
    width: 200px;
    text-align: center;
    line-height: 40px;
    font-size: 18px;
    background-color: lightblue;
    cursor: pointer;
}
 
.container .tip li.cur {
    background-color: lightsteelblue;
}
 
.container .contain {
    display: none;
    width: 600px;
    height: 300px;
    line-height: 300px;
    font-size: 20px;
    text-align: center;
    background-color: lightsteelblue;
}
 
.container .contain.cur {
    display: block;
}
```

reset.min.css文件
```css
body,h1,h2,h3,h4,h5,h6,hr,p,blockquote,dl,dt,dd,ul,ol,li,button,input,textarea,th,td{margin:0;padding:0}body{font-size:12px;font-style:normal;font-family:"\5FAE\8F6F\96C5\9ED1",Helvetica,sans-serif}small{font-size:12px}h1{font-size:18px}h2{font-size:16px}h3{font-size:14px}h4,h5,h6{font-size:100%}ul,ol{list-style:none}a{text-decoration:none;background-color:transparent}a:hover,a:active{outline-width:0;text-decoration:none}table{border-collapse:collapse;border-spacing:0}hr{border:0;height:1px}img{border-style:none}img:not([src]){display:none}svg:not(:root){overflow:hidden}html{-webkit-touch-callout:none;-webkit-text-size-adjust:100%}input,textarea,button,a{-webkit-tap-highlight-color:rgba(0,0,0,0)}article,aside,details,figcaption,figure,footer,header,main,menu,nav,section,summary{display:block}audio,canvas,progress,video{display:inline-block}audio:not([controls]),video:not([controls]){display:none;height:0}progress{vertical-align:baseline}mark{background-color:#ff0;color:#000}sub,sup{position:relative;font-size:75%;line-height:0;vertical-align:baseline}sub{bottom:-0.25em}sup{top:-0.5em}button,input,select,textarea{font-size:100%;outline:0}button,input{overflow:visible}button,select{text-transform:none}textarea{overflow:auto}button,html [type="button"],[type="reset"],[type="submit"]{-webkit-appearance:button}button::-moz-focus-inner,[type="button"]::-moz-focus-inner,[type="reset"]::-moz-focus-inner,[type="submit"]::-moz-focus-inner{border-style:none;padding:0}button:-moz-focusring,[type="button"]:-moz-focusring,[type="reset"]:-moz-focusring,[type="submit"]:-moz-focusring{outline:1px dotted ButtonText}[type="checkbox"],[type="radio"]{box-sizing:border-box;padding:0}[type="number"]::-webkit-inner-spin-button,[type="number"]::-webkit-outer-spin-button{height:auto}[type="search"]{-webkit-appearance:textfield;outline-offset:-2px}[type="search"]::-webkit-search-cancel-button,[type="search"]::-webkit-search-decoration{-webkit-appearance:none}::-webkit-input-placeholder{color:inherit;opacity:.54}::-webkit-file-upload-button{-webkit-appearance:button;font:inherit}.clear:after{display:block;height:0;content:"";clear:both}
```

index.js文件
```javascript
$.fn.extend({
    changeTab: function (index, type) {
        //->this:current tab
        index = index || 0;
        type = type || 'click';

        var $list = $(this).find('.tip>li'),
            $contain = $(this).find('.contain');

        //->select default
        $list.removeClass('cur').eq(index).addClass('cur');
        $contain.removeClass('cur').eq(index).addClass('cur');

        //->click
        var preIndex = index;
        $(this).on(type, function (e) {
            var tar = e.target,
                $tar = $(tar),
                tarTag = tar.tagName.toUpperCase();

            //->TARGET IS TAB
            if (tarTag === 'LI' && $tar.parent().hasClass('tip')) {
                //->REMOVE
                $list.eq(preIndex).removeClass('cur');
                $contain.eq(preIndex).removeClass('cur');

                //->ADD
                var curIndex = $tar.index();
                $tar.addClass('cur');
                $contain.eq(curIndex).addClass('cur');
                preIndex = curIndex;
            }
        });
    }
});

// $('#tab1').changeTab(0, 'mouseover');
// $('#tab2').changeTab(2);
$('.container').each(function () {
    $(this).changeTab();
});
```

index1.js文件
```javascript
$.fn.extend({
    changeTab: function (index, type) {
        $.each(this, function () {
            //->this:current tab
            index = index || 0;
            type = type || 'click';

            var $list = $(this).find('.tip>li'),
                $contain = $(this).find('.contain');

            //->select default
            $list.removeClass('cur').eq(index).addClass('cur');
            $contain.removeClass('cur').eq(index).addClass('cur');

            //->click
            var preIndex = index;
            $(this).on(type, function (e) {
                var tar = e.target,
                    $tar = $(tar),
                    tarTag = tar.tagName.toUpperCase();

                //->TARGET IS TAB
                if (tarTag === 'LI' && $tar.parent().hasClass('tip')) {
                    //->REMOVE
                    $list.eq(preIndex).removeClass('cur');
                    $contain.eq(preIndex).removeClass('cur');

                    //->ADD
                    var curIndex = $tar.index();
                    $tar.addClass('cur');
                    $contain.eq(curIndex).addClass('cur');
                    preIndex = curIndex;
                }
            });
        });
    }
});

// $('#tab1').changeTab(0, 'mouseover');
// $('#tab2').changeTab(2);
$('.container').changeTab(0, 'mouseover');
```


