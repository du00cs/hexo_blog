layout: post
title: "利用GitHub搭建个人博客-利其器(2)"
date: "2014-11-22 13:31"
tags:
- hexo
categories:
- 工具
---

　　工欲善其事，必先利其器。话虽然是这么说，但是在接触新事务的时候往往是相反的——先发现事情比较有意义，做着做着发现是不是做得太傻了，然后再发现其实运用某些工具可以让事情开展得更舒服。经历过这一过程的人来讲东西怎么用一般还是会把故事倒过来讲的。  
　　搭建博客的工具的调整主要还是在工具的微调上，具体说来有几点：
1. 掌握hexo的基本命令，这个一看就懂，无非就是创建文章、生成页面、预览和发布;
2. （可选，推荐）安装Atom并配置Markdown Writer
3. （可选，推荐）用github来跟踪blog的源文件

　　_Linux/Mac OSX中配置这些简直是太简单了，只有Windows才会有各种麻烦。_

## 1. hexo相关命令
　　以下均是在命令行中进行的
### 1.1 创建文章
```sh
hexo new [layout] title
```
　　layout不写默认就是post，会在`source/_posts`中创建以title命名的文件title.md，整个写博客的过程就是编辑符合markdown规范的文本。这也是hexo/jeklly号称能够让用户更专注的内容生产的原因。
　　即将发布的文章layout为post没有问题，但是也如果修改了一篇需要重新发布，而另外一篇才写了一半，这时就需要做些区分了。这时可以选layout为draft，文件就会被放到`source/_draft`中，在生成页面时会忽略这个目录，最后再文章完成之后再拖动文件到_posts中或者
```sh
hexo publish [layout] <filename>
```
就可以了。
### 1.2 生成页面
```sh
hexo clean #如果觉得页面比较诡异，这个命令将清除生成的页面
hexo generate #生成页面，生成的文件位于.deploy下
hexo g #g是generate的缩写，事实上以后基本上不会去用上一条
```
### 1.3 本地预览
```sh
hexo server # 启用本地预览，在localhost:4000可以访问到
hexo s #server的缩写
```
### 1.4 文章发布
```sh
hexo deploy #部署站点，实际上是将生成的页面git push到GitHub的Repository中
hexo d #缩写
```
　　掌握这部分已经可以完完全全开始了，但是总有一些其它的工具让你更写起来更爽、更舒服。这些工具往往是先行者们给予我们的便利，不必客气。君子生非异也，善假于物也。

## 2. Atom编辑器与Markdown Writer
　　首先选用一个好的支持Markdown编辑器还是非常有必要的，虽然说你完全可以用记事本/gedit来折磨自己，但是方便一点我想也是没有人反对的。在我眼中一个好的Markdown编辑器包括以下几个方面：
1. 文章支持实时/半实时预览（如果使用记事本，你就需要开启hexo s然后刷网页来看效果了）；
2. 支持语法至少与GitHub一致，比如\`\`\`的代码块要支持指定语言，否则那高亮完全就是来糊弄人的；
3. 支持Mathjax（如果不需要写公式则可以忽略）

　　[安装Atom](2014/11/22/利用GitHub搭建个人博客-工具准备/index.html)已经在前面介绍过了，是利用的Chocolatey。安装[Markdown Writer](https://github.com/zhuochun/md-writer)插件是通过`ctrl+,`来调出`settings`，在`pacakges`中搜索`markdown writer`，然后选择安装。这时Atom会提示一些错误（Mac上没有错误，估计Linux上也没有），如果你按照它的提示去安装VS Studio/Python等等东西的话，这日子真就没法过了，就那个VS Studio Express就有6G多。还好，没那那么麻烦，既然提示node-gyp不存在，那就用npm装一个好了。
```sh
npm install node-gyp
```
然后再点安装就可以搞定了。这个插件提供了很多markdown写作的快捷键，比如`ctrl+B`加粗、`ctrl+I`变斜体，插入图片则会给你弹一个窗口出来，确实是很方便。这里需要提一下的是Atom编辑器把所有的命令都放在了一个可以搜索的框内，直接输单词就可以调出某个功能，并不一定需要你去记某某某是什么什么快捷键。比如插入图片可以直接用`ctrl+shift+p`来调出一个窗口，然后输入`insert image`，回车或选择一个就可以调出插入图片的对话框。
![命令输入面板-输入insert im(age)](http://du00.qiniudn.com/2014/11/命令输入面板-insert_im.png)
　　上面是markdown方面的便捷功能，markdown writer还提供了与hexo/jeklly工作目录连接的功能，完成连接首先需要利用一个插件来生成它所需要的三个json文件：
```sh
#进入blog目录
e:
cd e:/blog
npm install --save hexo-generator-atom-markdown-writer-meta
hexo g #这时可以看到生成文件里面多了三个json文件了
hexo d #布署
```
　　配置与hexo的连接在**Settings->Filter Packages->输入Markdown Writer**在右侧的Settings中写入如下类似的东西即可：  
![插入图片-Markdown Writer连接hexo工作目录](http://du00.qiniudn.com/2014/11/markdown_writer的hexo设置.png)  
这里实际上是在修改`~/.atom/packages\markdown-writer\lib\config.coffee`文件，可以打开看一看。  
　　配置完成之后创建文章就可以不通过hexo命令了，在万能的`ctrl+shift+p`中输入`new post`或者`new draft`就可以生成新的文章了。不过这个功能目前还是有些问题，文件名生成有些障碍，需要手工修改一下文件名。

## 3. GitHub管理站点生成代码/源文件
　　简单地说就是把blog目录交给git管理，当然这里面要去掉一些没必要提交的。比如：.deploy目录，这里面的内容是可以通过`hexo g`生成的：node_modules目录，这里的东西是由`npm install`得到的。可以确认blog目录下是否有.gitignore文件并且内容包含以下部分：
>.DS_Store  
Thumbs.db  
db.json  
debug.log  
node_modules/  
public/  
.deploy/

有几点需要注意：
1. themes中如果包含了从git上拉下的主题，需要去主题目录下删掉.git文件夹，这样才能完成同步（如果不确定可以去GitHub的Repository中查看是否有主题）；
2. 从GitHub上拉下来的blog目录需要运行一次`npm install`安装需要的模块，否则是没法正常生成网页的；
3. 记得推送……git push，否则神也帮不了你  

　　为什么不用云盘/同步盘来做这件事？因为需要同步的有效内容非常少（比如修改了两行，那么就只需要同步这两行的修改），而且同一个目录下还有很多文件不需要同步，尤其是.deploy文件夹下的那些东西，`hexo g`一次你就会看见云盘的同步图标就开始转了，这其实没有必要。当然了，用GitHub显得更程序员、更任性。

下一步将是各种微调，是对感观上的增强，实际上来说也是最重要的部分，因为展示才是最重要的，上面这些只是为了更爽。
