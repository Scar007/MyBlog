---
title: Hexo + gitHub免费搭建一个属于自己的专属博客
subtitle: hexo
date: 2015-03-14 08:37:59
categories : Blog
---
之前有在[我的博客园](http://www.cnblogs.com/Scar007/)写自己的博客，但是作为一名程序员，一直都想在gitHub上开博的，毕竟这里是程序员的圣城。研究了一下午，其实还蛮简单的，这里分享一下自己的研究经验就当是这里的处女作了。

hexo是一款基于Node.js的静态博客框架,官网地址:https://hexo.io/zh-cn/.

## 安装环境
+ 安装node.js，去官网下载安装即可，[node官网](https://nodejs.org/zh-cn/)
+ 安装Hexo 
    全局安装hexo
    > npm install hexo-cli -g

创建hexo目录并初始化
```
 hexo clean
 hexo generate 或 hexo g
 hexo server 或 hexo s
```
然后就可以生成网站，启动服务了：
```
 hexo clean
 hexo generate
 hexo server
```
hexo文件夹<br/>
    先来看一下hexo文件夹下的内容：
    ```
            hexo/
        |- node_modules/  # hexo需要的模块，不需要上传GitHub
        |- themes/        # 主题文件，需要上传GitHub的dev分支
        |- sources/       # 博文md文件，需要上传GitHub的dev分支
        |- public/        # 生成的静态页面，由hexo deploy自动上传到gh-page分支
        |- package.json   # 记录hexo需要的包信息，不需要上传GitHub
        |- _config.yml    # 全局配置文件，需要上传GitHub的dev分支
        |- .gitignore     # hexo生成默认的.gitignore，它已经配置好了不需要上传的hexo文件
    ```
## 关联到gitHub
在github上创建项目仓库,仓库名为yourBlogName/github.io【注意后边的github.io是固定写法，前边的用户名你可以随便起】

+ 在本地打开hexo文件，修改_config.yml配置文件
```
title: 高佳睿的博客   # 标题
subtitle:
description:
author: 高佳睿
language: zh-CN         # 语言设置
url: http://palanceli.github.io/blog
root: /blog/
```
```
deploy: 
    type: git 
    repository: https://github.com/Scar007/Scar007.github.io.git
    branch: master 
```
    >  1.你需要将repository后的Scar007换成你自己的用户名。hexo 3.1.1版本后type:值为git。
        2.注意在配置所有的_config.yml文件时（包括theme中的），在所有冒号后都要加一个空格，否则执行hexo命令会报错
+ 在hexo文件目录下执行生成静态命令页面命令：
```
$ hexo generate 或者：hexo g
```
若此时出现如下报错：
```
ERROR Local hexo not found in ~/blog
ERROR Try runing: 'npm install hexo --save'
则执行命令：npm install hexo --save
若无报错，自行忽略此步骤。
```
再执行配置命令：
```
hexo deploy 或者：hexo d
```
若执行命令hexo deploy
仍然报错：无法连接git或找不到git，则执行如下命令来安装hexo-deployer-git：
```
 npm install hexo-deployer-git --save
```
再次执行hexo generate和hexo deploy命令。
若你未关联Github，则执行hexo deploy
命令时终端会提示你输入Github的用户名和密码，即
```
Username for 'https://github.com':
Password for 'https://github.com':
```
hexo deploy
命令执行成功后，浏览器中打开网址http://yourBlogName.github.io（将yourBlogName
换成你的用户名）能看到和打开http://localhost:4000
时一样的页面。

## 更换主题
这里有大量的主题：https://github.com/hexojs/hexo/wiki/Themes 或 https://hexo.io/themes/
找到自己喜欢的主题，然后将它下载到themes文件夹下，同时更改_config.yml配置文件中的theme属性为你所下载主题的文件名，ok，接下来重新生成静态文件，可以本地预览，你就会看到一个自己比较满意的博客了，大功告成，尽情的发表你的所言所想吧！

### next
个人目前用的就是next主题，简洁大方，而且各终端的支持也不错。<br/>
下边是关于next的一些配置，具体的一些东西还请阅读[hexo主题](https://hexo.io/zh-cn/docs/index.html)文档
```
 cd your-hexo-site
 git clone https://github.com/iissnan/hexo-theme-next themes/next
```
克隆最新版本的主题，将博客下的_config.yml得theme配置修改为（上边已经说过）：
```
theme: next
```
#### 标签、分类
想要添加标签、分类等，可以直接命令新建文件。这里用标签举例，其他的标签同理。<br/>
进入到你的博客的根目录下，命令执行：
```
 cd yourHexoBlog
 hexo new page tags
```
它会自动生成标签页面，找到tags文件，将文件内容修改为(可以按照自己的要求来自行设定，我这里只是个小demo)：
```
title: 标签
date: 2014-12-22 12:39:04
type: "tags"
```
然后在自己的文章中加上(同上，只是举例),一个可以直接写，多个就像这样用-分开就好：
```
tags:
    - 测试
    - blog
```
记得修改主题下的配置哦，如下：
```
menu:
  home: / || home
  tags: /tags/ || tags
```
还有像关于我这一类的文章，就更简答了，只需要新建文件后在文件中直接写自己想要表达的内容就好了！

#### 网站链接
找到主题下的social，设置你想要添加的网站链接，如下：
```
social:
  GitHub: https://github.com/Scar007 || github
  博客园: http://www.cnblogs.com/Scar007/ || cnblogs
  微博: https://weibo.com/5604488894/profile?rightmod=1&wvr=6&mod=personnumber || weibo
```

ok,这篇文章就先写到这，后边有问题再继续补充，谢谢支持！





