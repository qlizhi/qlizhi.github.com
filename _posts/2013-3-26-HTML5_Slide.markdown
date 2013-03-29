---
layout: post
styles: [syntax]
title: 基于HTML5的幻灯片
---

今天从网上看到关于HTML5幻灯片，一见钟情，决定毕业论文答辩采用HTML5来做。其实这个技术出来几年了，唉，落后了。
先发几个资源：
   
1. 先来个比较炫的：[slides.html5rocks.com](http://slides.html5rocks.com/#landing-slide "HTML5");  
1. 在来yihui的：		[yihui](http://slides.html5rocks.com/#landing-slide "HTML5");   
1. 再来Google的模版：[google template](http://html5slides.googlecode.com/svn/trunk/template/index.html#1 "HTML5");  
1. 还有几个基于CSS的框架:
    - [Slides Presentation with HTML5 ](https://github.com/briancavalier/slides "HTML5");
    - [csss](https://github.com/LeaVerou/CSSS "HTML5");
    - [Impress.js](http://www.oschina.net/p/impress-js "HTML5");
    - [Reveal.js](http://www.oschina.net/p/reveal-js "HTML5");
		
相关介绍可以在[这里](http://en.wikipedia.org/wiki/Web-based_slideshow)看看。  
还是重点介绍html5slides这个例子吧，网上资料什么的比较多。   
1. [html5slides 源码](https://code.google.com/p/html5slides/);  
1. [html5slides源码分析](http://firerails.diandian.com/post/2012-04-03/google-html5slides-source )   
1. [基于python的landslide开发html5slides](https://github.com/adamzap/landslide)

另外还有基于ruby [keydown开发html5](http://infews.github.com/keydown/)的工具，用的是markdown + deck.js, 网上找到另外一个有意思的东东[controldeck](http://controldeck.aws.af.cm/)，能在另外一个网页上远程操纵slides播放。最终觉得[html5slides的第二版本](http://io-2012-slides.googlecode.com/git/template.html#1)最省事，所以决定用这个了。

* * * * *
## Landslide

Landslide generates a slideshow using from markdown, ReST, or textile.  
安装Landslide有多种方法，[不过用python包管理器比较方便](http://www.cnblogs.com/jiekk/archive/2012/03/29/2423602.html)：     
> 先安装setuptools-0.6c11.win32-py2.7.exe   
> 在安装pip或者easy_install  
> 接下来就交给pip包管理器，输入`pip install landslide`搞定    

## 关于Markdown的一些格式说明：

1. 以后缀名`.md`, `.markdn`, `.mdwn`, `.mdown`, `.markdown`   
1. 第一张slide用一个标题  
1. 多张slides以`---`分隔   
1. 代码高亮加, 在代码块第一行以`!lang`方式，如`!python`为python的语法高亮。

## 编译

运行`landslide slides.md`得到一个presentation.html文件。

一些命令参见[landslide的说明文档](https://github.com/adamzap/landslide)