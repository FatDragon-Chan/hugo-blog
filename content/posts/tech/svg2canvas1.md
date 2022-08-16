---
title: " 从零实现Svg2Canvas（一）"
date: 2022-08-16T15:13:16+08:00
lastmod: 2022-08-16T15:13:16+08:00
author: ["AHone"]
keywords: 
- 
categories: 
- 
tags: 
- web
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: true
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: false # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: false # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    caption: "" #图片底部描述
    alt: ""
    relative: false
---



## 0. 前言

> 一切的起源都来自于一次迭代需求

需求描述：色区域可点击，点击后弹窗选择方案。确认了选择后，该区域变成蓝色并显示该区域id

我们可以简单的看下最终实现的效果

Tips：弹窗无法遮盖[^1]是微信开发者工具的bug，真机情况下并不会显示出canvas

![iShot_2022-08-16_17.52.54](http://img.chenzian.com/uPic/iShot_2022-08-16_17.52.54_2022_08_16_17_53_15_2022_08_16_17_53_19.gif)

也可以扫码体验![image-20220816175433469](http://img.chenzian.com/uPic/image-20220816175433469_2022_08_16_17_54_33.png)

完整的项目代码也会在文末给出，欢迎讨论



## 1. 预研方案

> 那么此时拿到需求的我，~~在努力构思可以实现需求的方法和方案~~，努力找轮子

- 方案一：万能图片切换大法

  - 胎死腹中，不规则点击区域加上有相关的指引线，实现起来比较复杂且相对不那么好用
- 方案二：SVG一把梭

  - 胎死腹中+1，小程序不支持原生的svg实现
  - 小程序支持svg`Cax`[^2]渲染，但本质上仍然是通过canvas的转绘，且已经有段时间没有维护了，所以也暂时未考虑使用
  - 此时转念一想，既然svg可以通过canvas 实现，那我用geoJson描绘地图的方式岂不是也可行。此时我想到了第三种方法

- 方案三：通过GeoJson用canvas实现（类似于地图点击效果）

  - [Echarts中对于Map的实现](https://echarts.apache.org/examples/zh/editor.html?c=geo-beef-cuts)
  - [svg2geoJson](https://github.com/yuhonyon/svg2geojson)但需要利用echarts等第三库实现对geoJson的解析。



在努力寻找了这么久之后，~~开始寻求其他方向上的突破~~，请产品喝奶茶

但最终确定仍然是需要实现这么个需求，此时我注意到了方案二，既然小程序官方都是用canvas转绘svg，那我是不是可以找找有没有大佬实现过svg2canvas 的转绘轮子呢？需求确定下来了之后，便开始了又一轮的寻找



## 2. SVGToCanvas

## 参考文档

[^1]: 微信开放社区 [开发者工具 - 遮罩无法覆盖canvas](https://developers.weixin.qq.com/community/develop/doc/000624965c4cd8aae71d0f7d55bc00)
[^2]: 微信开放社区 [今天，小程序正式支持SVG](https://developers.weixin.qq.com/community/develop/article/doc/000ca493bc09c0d03a8827b9b5b013)
[^3]: 许公子丶.[macOS - 给Terminal终端命令行配置网络代理
