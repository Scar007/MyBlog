---
title: ejs模板引擎的使用
subtitle: ejs
date: 2015-8-11
categories: ejs
---
## 1.引入ejs.min.js
## 2.创建模板，以<%=jsCode%>包裹起来其余的html和html结构一样  
focusTemplateData是模板使用的数据,使用$.each()方法遍历绑定数据
```javascript
    <%$.each(focusTemplateData,function(index,item){)%>
        <div class="swiper-slide">
            <a href="<%=item.link.weburl%>">
                <img src="<%=item.thumbnail%>" alt="<%=item.title%>"/>
                <span class="bg"></span>
                <span class="txt"><%=item.title%><span>
            </a>
        </div>
    <%}%>
```

## 3.获得模板绑定数据 通过id获取模板
```javascript
    var focusTemplate = $("#focusTemplate");
    使用固定的ejs.render(模板结构,{模板使用数据:真实数据});方法绑定数据
    focusTemplate.html()是获取模板字符串
    var focusResult = ejs.render(focusTemplate.html(),{focusTemplateData:focusData});
```

## 4.渲染到页面
```javascript
    $('.swiper-wrapper').html(focusResult);
```


