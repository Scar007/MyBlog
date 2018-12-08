---
title: GitHub上常用命令(工作中几乎每天用到的命令)
subtitle: gitHub
date: 2017-7-4
categories: GIT
tags: 
    - GIT
---
#### 查看自己当前分支
git branch

#### 查看远程和本地分支
git branch -a

#### 查看origin下的分支
git config -vv

git config --lis

#### 创建分支
git branch 分支名

#### 创建并切换分支
git checkout -b 分支名

#### 切换到某一个分支
git checkout 分知名

#### 删除本地分支
git branch -d 分知名

#### 删除远程分支和tag
+ 删除远程分支 git push origin :分知名(注意:origin后边有空格)
 git push origin --delete<branchName>
+ 删除tag git push origin --delete tag<tagName>
> 也可以推送一个空分支到远程分支上去，其实就相当于删除远程分支 git push origin :<branchName>
> 删除tag的方法，推送一个空的tag到远程tag git tag -d <tagName> git push origin :refs/tags/<tagName>

#### 连接远程仓库
git remote add origin 仓库的地址

#### 查看远程的仓库
git remove -v

#### 删除远程仓库
git remote rm origin

#### 合并分支
git merge scar
> 比如当前在master分支上，我要把scar分支合并在master就执行上边的命令

#### 查看git版本
git --version

#### 获取当前登录的用户
git config --global user.name

#### 获取当前登录的用户的邮箱
git config --global user.email

#### 设置git账号,username为你的git账号
git config --global user.name 'username' git config --global user.email 'email'

#### 初始化git仓库
git init
> 在nodejs文件夹下初始化一个仓库，此时文件里会有一个.git的隐藏文件夹

#### 增加到暂存区中
git add index.html

git add -A

> 全部添加到缓存区

#### 增加到版本库中
git commit -m '备注信息'

#### 查看版本
git log --oneline

#### 把所有在工作区新添加的或新修改的文件添加到暂存区
git add .

#### 从暂存区提交到历史区,并且添加注释给所有文件
git commit -am "注释内容"

#### 查看当前工作区下的状态
git status

#### 将网站上的代码克隆到本地(github和coding)
git clone 网站的路径

#### 配置用户名,不加""表示查询
git config --global user.name ""

#### 配置用户邮箱,不加""表示查询
git config --global user.email ""

#### 将本地的文档添加到网站上，在bash中输入账号,别使用弹窗
git push origin master

#### 查看当前文件夹的网站目录
git remote -v
