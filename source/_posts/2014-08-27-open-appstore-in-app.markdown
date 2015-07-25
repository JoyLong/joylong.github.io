---
layout: post
title: "在自己应用中直接打开app store上的应用页"
date: 2014-08-27 16:56:52 +0800
comments: true
categories: iOS
---


iOS 6中新增了Store Kit

在其中有一个SKStoreProductViewController

![Mou icon](http://github-octopress.qiniudn.com/howtouseSKStoreProductViewController1.png)

实现唯一的代理方法

![Mou icon](http://github-octopress.qiniudn.com/howtouseSKStoreProductViewController2.png)


Over

**注1：SKStoreProductParameterITunesItemIdentifier:后面填自己应用的ID,类型是NSNumber**

**注2：一直没用过内购功能的我，才知道可以这么用，我自己真是蠢蠢哒T_T**