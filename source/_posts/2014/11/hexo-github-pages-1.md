layout: post
title: "利用GitHub搭建个人博客-工具准备(1)"
date: "2014-11-22 03:34"
tags:
- hexo
categories:
- 工具
------

>　　最近两个星期对`Github Pages`这个东西非常着迷，这东西竟然可以DIY成自己的博客！ 两个星期的探索后终于摸索出一条比较好的安装/环境配置的方法，记录在此，就权当是备忘吧。

1. 什么是**Github Pages**?
=======================

　　Github Pages是Github交给用户自己定义的主页，自定义的程度非常高（仅限于静态页面）。至于怎么被人挖掘出来做个人博客，这个我就不得而知了，我只是觉得用起来很爽……先看看下面的效果图，有没有觉得跳出了新浪、百度等等烦人的框架后清新了许多？ ![博客首页截图](http://du00.qiniudn.com/2014/11/博客首页截图.png)

2. 博客搭建工具一览
================
　　其实吧，完整搭起这个博客除了文章是我自己写的（需要掌握Markdown，相信我，学会了之后你会爱上它的），其它都是借鉴（抄袭）的别人的……这个网页中我实际写的其实是这些，而看到的是下面这个页面。确实做到了让用户更关注于内容的产生，而不是各种要注意的特别格式。
![markdown原始文件](http://du00.qiniudn.com/2014/11/markdown原始文件.png)  
　　如果不是那些（姑且）称作“极客”的人开发了那么那么多的工具，你很难想象自己纯手工打造有多困难，尤其是针对像我这种对`html/js/css`一无所知的人。搭建工具包括：
1. node.js
2. hexo
3. git/github

<!-- more >

### 2.1 安装chocolatey
　　[chocolatey](https://chocolatey.org)是面向Windows的类apt/yum/brew这样的包管理工具，有了这么个好东西Windows下配置编程环境就不会再那么复杂了。在命令行（在Power Shell中执行引号中的内容在我的机器上会出错）中贴上以下语句
```sh
@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
```
即可完成安装。如果跟我一样有不把东西安装在C盘的习惯，可以按照如下操作：  
　　默认Chocolatey会安装到`C:\ProgramData\`下，可以将这个文件夹移动到别的地方，然后在**系统 -> 高级系统设置 -> 环境变量**中修改`ChocolateyInstall`和`PATH`为相应的路径就可以了。

### 2.2 安装git  
　　GitHub Pages的文章发布会使用到Git这个工具，对程序员来说Git肯定是相当熟悉的。
```sh
choco install git
```
　　坑爹，怎么把git安装到C盘去了？这个没办法，全自动的就是这么任性。但是这并不是说上面移动Chocolatey的位置就没有意义了，还是有其它更大的软件会装到那里的。

### 2.3 安装node.js  
　　搭建博客、生成页面的工具是用这个写的，各种插件及依赖也是node自动完成的。
```sh
choco install nodejs
choco install npm
```

### 2.4	安装hexo  
　　[hexo](http://hexo.io)是用来从markdown的纯文本自动生成网页的工具，比起同类产品`jeklly`来说，上手简单得不只一点半点（我在学jeklly的时候始终弄不明白，差点放弃GitHub Pages，幸运地是我找到了hexo）。
```sh
npm install -g hexo
```
　　这里需要注意的是直接在命令行中输入hexo是找不到命令的（工具还是不完善），需要手工去把hexo所在目录加入到PATH中。进入**环境变量**在`PATH`中加入类似于`E:\Programming\chocolatey\lib\nodejs.commandline.0.10.33\tools;`这样的路径。记得重新开一个命令行来保证环境变量生效。然后使用以下命令来完成一个hexo管理的博客的结构初始化：
```sh
mkdir blog
cd blog
hexo init
# 这时hexo会提醒你可能需要npm install来完成初始化
npm install
# 下面可选，完成后就可以从网页看到最原始的样子了
hexo g # 等同于hexo generate，生成静态页面
hexo s # 等同于hexo server，启动一个本地站点，可以在浏览器中输入 (localhost:4040) 来查看页面的样子，`ctrl + c`停掉。
```
![插图 - 初始网页](http://du00.qiniudn.com/2014/11/hexo本地截图.png)

### 2.5（可选）安装[Atom](https://atom.io/)
　　Atom是支持markdown的编辑器，类似于SublimeText，但是插件功能更为强大。有国人为Atom写了`Atom Markdown Writer`，用来编辑、管理hexo/jeklly生成的静态站点还是非常方便的。
```sh
choco install atom
```
　　Atom的配置以后再讲。

需要安装的基本工具就到此结束了。

2. 配置自己的GitHub Pages
======================
　　这一部分将会建立本地工具与远程站点之间的联系，并且能够把页面推送到远程站点。对于GitHub Pages来说则是将本地生成的静态站点推送到某个特定的Repository就可以完成站点的搭建/更新。
###2.1 创建GitHub Pages 　　
　　在GitHub上创建一个Repository，命名必须是类似于`du00cs.github.io`，创建之后在下一页选择`Settings`找到`Automatic page generator`，下一步下一步直接`Publish page`，然后按照提示等待10（多）分钟，再去访问`du00cs.github.io`就看到初始页面了。

###2.2 将hexo与GitHub远程库关联

1. 为了提交方便，需要把自己的`SSH-KEY`放到GitHub上，具体操作在[SSH Keys](https://github.com/settings/ssh)的帮助文档中有详细介绍。
2. 找到blog目录下的`_config.yml`，仿照类似修改`deploy`部分
```yaml
deploy:
  type: github
  repo: https://github.com/du00cs/du00cs.github.io.git
```

###2.3 发布博客

```sh
# hexo clean #如有必要可以清空
hexo g #生成站点内容
hexo s #你可能需要在本地预览
hexo d #发布，可以稍后在github上看到更新，一般没有什么延迟
```

　　这样就完成了所有工具的准备，剩下的就是需要一个内容的生产者和调整样式、主题的设计师。
