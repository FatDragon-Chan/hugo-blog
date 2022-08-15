---
title: "Tech"
date: 2022-05-05T00:17:58+08:00
lastmod: 2022-05-05T00:17:58+08:00
author: ["Sulv"]
keywords: 
- 
categories: 
- 
tags: 
- tech
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: true
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
    image: "" #图片路径例如：posts/tech/123/123.png
    caption: "" #图片底部描述
    alt: ""
    relative: false
---


# 手摸手使用Electron改造Vue项目（一）

## 前言

> ~~为什么要突然要看Electron，卷起来，不得什么都得看看~~🐴

### 现阶段面临的痛点

- 权限限制
    - 系统级通知
    - 文件加载权限
    - 与系统的交互
- 兼容性限制（在互联网上，没人知道你是一条狗，同样的，在互联网上，你永远不知道客户使用的版本会是多古早的版本）
    - 新的API
    - 部分解析库

### 那为什么不是`Flutter`或其他跨平台桌面应用构建

不可否认的是， `Fultter`的自绘性能要好于ELectron，但`Fultter`有一个学习的成本存在，且无法短时间或者较少的精力情况下修改已有项目~~（主要是不会，留着下次卷）~~



## 那开始把，从安装开始

### 0. 开发环境

- M1 MacOs Monterey **v12.5**
- Node **v16.16.0**
- Npm **v8.16.0**
- Yarn **v3.2.2**
- Electron **v20.0.2**
- python **v2.7.18**

### 1. 开始搭建Demo及环境

#### 1.1 寻找一个vue项目

> 此处使用Vue-typescript-admin-template代替常规的Vue项目方便理解，实际开发中不建议使用Web现成框架完成，不光是其中的部分兼容问题需要处理，同时客户端给与用户的使用体验也应该是不同的，比如Dock栏等的操作。如果可以，最好寻找下更贴合客户端UI/UX的库或者额外实现

此处项目改造示例使用的是 `vue-typescript-admin-template`[^1]

```bash
git clone git@github.com:Armour/vue-typescript-admin-template.git
cd vue-typescript-admin-template
npm i
```

#### 1.2 安装Electron库

> 此处可能会因为网络或者不可描述的问题会有失败的可能，我举例我处理过的部分，大部分Google都有大神给出了解决方案

```bash
# 全局安装
npm i  electron -g
```

安装过程中可能会出现问题，因为ELectron会去github拉取很多依赖库，大部分是网络的问题，此处我列举我遇到的部分问题，官方文档的安装指导[^2]里也有解答，更建议去查阅官方文档

#### 1.2.1 安装问题汇总

> 大部分这些网络问题可以通过挂机场等之类的方式解决，但很多时候你会发现终端没有走机场代理，因为代理常规是socket5的代理方式，后续使用软路由解决掉这一部分问题，如果有需求，我后续再分享软路由的搭建过程

##### 例子1：安装报错  socket hang up

##### 例子2：安装报错 connect timeout

##### 例子3：安装通过，运行时报错 Electron failed to install correctly

​		这些报错问题都是因为没有出国学习的问题导致的库拉取失败



##### 屏蔽掉之前配置的国内源

```bash
# 查看源
npm config get registry

# 设置官方源
npm config set registry https://registry.npmjs.org/
```

##### 查看全局代理：查看自己的科学上网端口[^3]

比如我的科学上网代理端口为http/https: 9999, Socks: 10000

因为在终端下不支持 socks5 代理，只支持 http 代理，wget、curl、git、brew、composer 等命令行工具都会变得很慢。

##### 究极解决办法[^4]

```bash
npx cross-env ELECTRON_GET_USE_PROXY=true GLOBAL_AGENT_HTTPS_PROXY=http://127.0.0.1:9999 npm install -D electron@latest -g 

# 安装完成后，检查版本
electron -v
# v20.0.2
```



#### 1.3 安装构建依赖

```bash
# 打开项目目录 ./vue-typescript-admin-template
npm install  electron-builder --save-dev
# 此处依赖于vue-cli脚手架的配置，所以请先全局安装好vue-cli
vue add electron-builder

```

​	安装完成后项目目录src中会新增一个background.js的文件，用途为electron的配置，可参考官网api文档调整。

​	且package.json的scripts也会增加两条命令

#### 1.4 运行

``` bash
npm run electron:serve
```

​	此时便会运行一个Demo应用啦

![image-20220811195137954](http://img.chenzian.com/uPic/image-20220811195137954_2022_08_11_19_51_38.png)



































































## 参考

[^1]: Armour.  [vue-typescript-admin-template](https://github.com/Armour/vue-typescript-admin-template)
[^2]: Electron. [文档-安装指导](https://www.electronjs.org/zh/docs/latest/tutorial/installation)
[^3]: 许公子丶.[macOS - 给Terminal终端命令行配置网络代理的方法](https://www.jianshu.com/p/0ad19c5e7def)
[^4]: [stackoverflow](https://stackoverflow.com/questions/60054531/how-can-i-solve-the-connection-problem-during-npm-install-behind-a-proxy)
[^5]: 




