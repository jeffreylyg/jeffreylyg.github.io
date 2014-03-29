---
layout: post
title: "My First Octopress Blog"
date: 2014-03-12 14:37:20 +0800
comments: true
categories: 
---

很久之前就想搭一个个人**Blog**了，但一直苦于找不到一个合适的**Blog**平台。之前搞ACM的时候在CSDN上写过一些题解，主要做模板和记录的作用，写过几篇后就不了了之了。然后体验过国外很火的轻博客[**Tumblr**](https://www.tumblr.com/)以及国内的[**点点**](http://www.diandian.com/)等，这些平台都不能完全的私人定制，感觉甚是不爽。说到这儿可能会有人问为什么不去用[**WordPress**](http://wordpress.com/)，其实之前是有试着用[**WordPress**](http://wordpress.com/)去搭的，但感觉很是麻烦，先不说部署时的学习曲线，还要自己去找服务器，国内的免费服务器支持的不是很好，**新浪云**有支持，但免费的还是不能完全的私人定制，花钱租服务器觉得不划算，觉得还没有到花钱的地步，不是自己舍不得这笔钱，主要是认为自己才刚开始，博客一时半会儿不会到拥有大量读者的地步而且主要是自娱自乐。还有一个原因当时真得没有时间去折腾这个。所以这件事也就搁置了好久，一直到现在。
<!--more-->
最近偶然在Google问题浏览他人Blog的时候发现了[**Octopress**](http://octopress.org/)的存在，于是马上去它的官网查了查，瞬间被它的一句话所吸引：
>A blogging framework for hackers.

于是大概看了一下它是如何部署的，原来是用`ruby`脚本帮你封装了生成静态页面以及git的一系列操作，虽然从来没用过`ruby`，但对git还是蛮熟悉的，而且它可以部署在[**Github Page**](http://pages.github.com/)上，所以服务器这一项完全免费还支持custom domain，于是立马开始了对`Octopress`的部署（网上有大量的Octopress的部署教程，这里不赘述了，请大家自行Google）。博文支持当下很流行的`Markdown`语法，`Markdown`确实强大而且`Markdown`还是蛮容易上手的，这里推荐一个Mac OS上很不错的Markdown编辑器[**Mou**](http://mouapp.com/):

![Mou preview](http://mouapp.com/images/Mou_Screenshot_1.png)

随后发现了一款Octopress上非常不错的主题，第一眼看到它就喜欢上它的高大上了，这款主题叫[**Greyshade**](http://shashankmehta.in/archive/2012/greyshade.html), 也就是本人现在正在用的这款主题。引入这款主题的时候会出现一个问题，那就是如果你引入了第三方评论系统[**Disqus**](http://disqus.com/)，那么引入这款主题后[**Disqus**](http://disqus.com/)后就看不见了，在网上Google了好久终于在这篇文章<http://bryanone.com/blog/2014/03/01/problem-with-greyshade/>里找到了原因并有解决方案，很感谢这位博主。

最后，还是好基友说的那句话，搭一个Blog平台容易，但坚持写Blog不是一件容易的事。所以，总之，加油吧，在这里记录点什么。