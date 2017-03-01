---
layout: post
title: 关于Github Pages 搭建blog
categories: [技术]
tags: [技术]
description: Github Pages免费blog搭建，从这里重新开始.
---

  程序猿交友网站Github是众所周知的，他可以提供我们代码托管，和各地程序猿协作开发，也是许多开源项目的聚集地。为了交友（技术知识分享），Github提供了一种功能叫做Github Pages，它可以让我们基于自己项目建立自己的网站，
这里就日常流水般的记录一下自己搭建的blogs。

  建立我们自己的网站的时候我们可以选取一些静态网站生成框架，这样可以自由的定制blog风格，而且很多工具可以在网上找到一堆的主题，选个中意的可以省去很多事情。看好的有两个框架[Jekyll](http://jekyll.com.cn)、[Hexo](https://hexo.io),
两个的区别在于Jekyll基于Ruby环境，而Hexo基于NodeJs环境（其实Jekyll也是可以在本地直接预览的）。而我选择的是Jekyll，因为是用的Mac，mac自带Ruby环境，平时也用的多一点，安装Jekyll起来也比较方便（其实也是个巨坑），其实想想，我也早就装NodeJs环境的。当然也有
比较牛的，不用本地环境，直接将copy下来的主题一次改成自己喜欢的样式提交到Github上去，直接预览。

  [Github Pages](https://pages.github.com)提供了一些简单的指引，你可以根据指引来做一个很简单的Hello World 页面。

  这个博客搭建可以有几种选择：
    <br />1、完全定制自己的博客
    <br />2、找一会你喜欢的框架，然后修改后使用
    <br />3、在Github上Fork别人的Blog，然后添加文章

  这里选择的是第二种 博客主题选择的是[dbyll](https://github.com/dbtek/dbyll)

  首先需要在Github首页上New 一个repository，名字尽可能取和Github userName一样的，如果不是，之后生成的链接会超长。创建完成之后Clone到本地文件就可以。接下来就需要我们配置Jekyll 环境了。


  打开终端，具体的命令如下 

{% highlight ruby %}
##查看ruby 版本，必须要在2.0.0之上才能安装Jekyll
~ruby -v                                                                    
~ruby 2.2.0p0 (2014-12-25 revision 49005) [x86_64-darwin14]
##如果ruby不够高，可以更新下
~sudo gem update --system
##安装jekyll
$ sudo gem install jekyll
##安装成功之后，查看版本号
$ jekyll -v
{% endhighlight %}

注意: 
    <br />1、Operation not permitted（EI Capitan） 如果Mac系统是EI Capitan 并且ruby环境装在了root目录下，
    那需要关闭SIP保护，关闭它必须进入recover 模式（重启之后按住command+r），在工具中找到terminal执行csrutil disable命令，回车，然后重启，SIP保护就被关闭了。 
    <br />2、ruby版本必须高于2.0.0。   如果ruby版本比较低 可以使用rvm 更新ruby版本[链接](https://segmentfault.com/a/1190000003784636)
    
如果Jekyll安装成功了，那现在就可以将本地的serve运行起来了
<br />1、将dbyll这个工程down下来，拷贝到Github repoeitory的本地目录下 
<br />2、安装bundle用来管理gem，gem install bundle
<br />3、cd 到Clone的repository目录下，bundle install
<br />4、启动服务 bundle exec jekyll serve

{% highlight ruby %}
##安装成功之后，查看版本号
~gem install bundler
##到博客目录下
~cd MyBlogs
~bundle install
##运行serve
~bundle exec jekyll serve
{% endhighlight %}

然后就可以以本地服务运行搭载dbyll这个静态网站了，地址一般为http://127.0.0.1:4000/ ,在网站里面很多内容都是定制好了，接下来感觉哪里不合适自己就改哪里了。文章路径是在_posts目录下，格式为md的，如果想布局好点可以稍微去看看markdown的语法，
很容易的。

github+jekyll的工作机制
<br />1、简单来说，当你访问你github上面http:user.github.io的时候，GitHub会使用jekyll解析username下面的username.github.io项目中，分支为master的源码，构建一个静态网站，并将生成的index.html展示出来。每一个账户名下都会有一个
GitHub分配的一个域名，同时每个项目下面也会有一个域名，这里就可以选择不同的分支来决定我们需要展现什么样子的网站。

<br />2、Jekyll 提供了插件功能，在网站源代码目录下，有一个名为 _plugins的目录， 你可以将一些插件放进去，这样，Jekyll在解析网站源代码时，就会运行你的插件， 这样插件是Ruby写成的。可以为Jekyll添加功能，例如，Jekyll默认是不提供分类 
页面的，你可以写一个插件，根据文章内容生成分类页面。如果没有插件，你只能每次写文章，添加分类时，为每个分类手动写HTML页面。在本地运行 Jekyll 时，这些插件会自动被调用，但是GitHub在解析网站源代码时， 出于安全考虑，会开启安全模式，禁用
这些插件，所以我们我们需要在本地用jekyll解析最终生成静态网站，提交到GitHub上。

好吧 GitHub博客已经建立好了，接下来就是把自己以前记录在印象毕竟的东西整理挪过来了。






