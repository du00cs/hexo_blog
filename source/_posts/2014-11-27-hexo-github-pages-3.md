layout: post
title: "2014-11-27-利用GitHub搭建个人博客-美化(3)"
date: "2014-11-27 20:39"
tags:
- hexo
categories:
- 工具
---

　　美化调整其实是重头戏，但是对于像我这种不懂Html/JS的人来说美化就只有两步：
1. 使用别人的主题
2. 调参——对各项参数进行微调
这部分的内容比较繁杂，大多数都是在耗在反复对比上了。

## 应用主题
　　Hexo的爱好者们DIY了很多各式各样的[主题](https://github.com/hexojs/hexo/wiki/Themes)，并且还分享出来了。有很多非常的酷，可以尝试尝试。安装方法很简单，比如要安装`metro-light`，在博客的主目录下执行
```sh
git clone https://github.com/halfer53/metro-light.git themes/metro-light
```
然后在主目录中的`_config.yml`中设置theme为`metro-light`即可。别人共享的站点中都有如何安装的说明，以及有哪些配置项，记得扫一眼！
> 注意：如果用GitHub来同步整个博客，记得把**themes/metro-light**下的**.git**文件夹删掉。

## 配置参数
　　参数配置很简单，作者都留好了入口，挨个填就可以 了。首先要注意区分根目录和主题目录下的各有一个`_config.yml`文件，参数需要分别在两个地方进行配置。
- 配置主目录_config.xml
　　主目录的_config.xml的配置不能够马上反映的本地站点(`hexo s`)上，需要`hexo g`一次，主题内的配置文件修改完就可以在网页上刷新看到。下面列出了至少需要配置的内容，按自己的情况一一修改，不放心就本地预览。

```yaml
# Site
title: Oh Captain, My Captain - Du00
subtitle: Qzone-天涯-163-百度-新浪，削足适履，不如亲手打造
description: Du00的博客
author: Du00
email: du00cs@gmail.com
language: zh-CN

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://du00cs.github.com

# Extensions
theme: metro-light

# Deployment
deploy:
  type: github
  repo: https://github.com/du00cs/du00cs.github.io.git
```
- 配置主题中的_config.yml
　　每个主题中可配置的项各不一样，下面还是以`metro-light`为例说明一些配置，主要是评论系统和分享按钮。

```yaml
#duoshuo_short_name是需要去“多说”申请的，填错无效……
comment:
 duoshuo: true
 duoshuo_short_name: du00cs
## to enable disqus, you need to fill in the disqus_shortname in config.yml
## to enable duoshuo, you need duoshuo id and set duosuo to true

#share plugins at the bottom of the article
share:
  enable: true
  jiathis: true ## Jiathis是一个面向国内的分享插件，你不会想分享到google/twitter的……
  twitter: false
  google: false

bottom_link:
  github: du00cs ## 填写用户名即可
  weibo: du00cs ## 填写微博数字ID或者用户名（不是昵称）
  renren: ##e.g. 333333333 for http://www.renren.com/333333333

#google analytics id, 这个可以用来对网站进行统计，同样需要申请
google_analytics: UA-56718947-1
```

## 点点滴滴，需要耐心
- hexo是一个台湾学生写的，不得不佩服
- 首行缩进：英文首行有没有缩进无所谓，中文不写就很难看了——输入两个全角空格即可（一般可用`shift+space`切换到全角输入)
- 添加公式支持：网上有加语句的，事实上加个插件就好了（尤其是对我这种小白）
```sh
hexo install hexo-renderer-mathjax --save
```
并在_config.xml标明使用了该插件(**注意空格**)
```yaml
plugins:
  - hexo-renderer-mathjax
```
- Atom公式预览：安装markdown-preview-plus，注意Display的公式需要写成
```
$$
  ax^2+bx+c=0
$$
```
- 调整markdown的样式：别人设置的样式可能有你不喜欢的，如果你看见“引用”部分居中了想修改，去`metro-light/source/css/_partial/article.styl`中修改即可。或者如果你有喜欢的样式，比如Mou中的GitHub2的表格是有颜色间隔的，这时可以找一个css转stylus的工具(npm install stylus)，在生成的文件中把table部分代码贴过来替换掉即可。同理，如果文本是两边对齐的想替换成左对齐，可以先用浏览器的“审查元素”的功能，定位到相应的文本域，查看它的CSS就可以进行相应定位了。
- 文章预览只显示部分内容：原始模板中首页的预览把所有文章都显示了，如果主题没有只显示部分的功能，可以手工在文章中加上`<!-- more -->`，这一句之后的部分就不会在首页中显示了。


　　主题、插件这两个东西需要好好借助它们来为自己服务，相关的文章其实还是不少的，但是大同小异。如果不懂JS之类的，可以调整的范围也非常有限，但是试试总是没有坏处吧～最后多看看别人的主题，偷偷代码改一改，说不定会有惊喜的。最后，用Hexo搭博客就是为了好好写东西，内容，还是最重要的。
