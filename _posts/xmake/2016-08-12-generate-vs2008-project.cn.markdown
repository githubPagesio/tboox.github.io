---
layout: post.cn
title: "xmake支持vs2008生成"
tags: xmake VisualStudio vs2008
categories: xmake
---

xmake master上最新版本已经支持vs2008工程文件的生成，通过`project`插件的方式提供，例如：

创建vs2008工程文件：

```bash
$ xmake project -k vs2008
```

默认输出目录是在当前工程的下面，会生成一个vs2008的工程文件夹，打开解决方案编译后，默认的输出文件路径跟xmake.lua描述的是完全一致的，一般都是在build目录下

除非你手动指定其他的构建目录，例如：`xmake f -o /tmp/build`

创建vs2008工程文件，并且创建工程文件到指定目录：

```bash
$ xmake project -k vs2008 f:\vsproject
```

目前这个插件也是刚刚跑通，身边暂时没有其他vs版本可供测试，理论上已经可以支持vs200x的所有版本了（vs2002, vs2003, vs2005, vs2008）

如果有兴趣的同学可以先行测试下其他版本。

另外目前vs2010以上版本，暂时还不支持，后续也会陆续实现掉，想要用vs2015来编译的话，理论上vs是可以向下兼容支持低版本vs工程的，可以尝试用vs2015加载vs2008的工程文件试试。。





后面的工作计划：

* 优化现有vs200x工程文件，目前的生成方式不利于vs编译优化，所以编译速度较慢
* 实现vs2010以上版本的生成插件
