---
title: "自动调整 import 代码顺序"
date: 2022-08-16T15:13:16+08:00
lastmod: 2022-08-16T15:13:16+08:00
author: ["AHone"]
keywords:
- 
categories: 
- 
tags: 
- 
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



因为项目持续迭代的问题，`import`的文件越来越多，之前主要通过`webstorm`的导入功能加上简单的格式化并加上代码注释，类似这样

```ts
// hooks
import Taro, { useDidShow, useRouter } from '@tarojs/taro'
import React, { useEffect, useMemo, useRef, useState } from 'react'
import useGlobalVar from 'src/common/hooks/useGlobalVar'
import { customThrottle, formatTime, getTelphone } from 'src/common/util'

// 全局组件
import { Button, Image, Navigator, Text, View } from '@tarojs/components'
import ImgToast from 'src/component/ImgToast/index'
import Navigate from 'src/component/Navigate/index'

import { traceEventsReport } from 'src/common/report'
// 数据上报配置
import pagePathCode from 'src/dataReport/page.config'
```



但每次调整过于费劲，想着寻找可以格式化的工具处理，在闲逛时发现有人接受`Eslint`[^1]自带的import 配置

但尝试了下配置规则发现并不那么好用。比如多行导入的处理。

此时看到一个插件， `eslint-plugin-simple-import-sort`[^2]，配置简单又好用。

```bash
yarn add -D eslint-plugin-simple-import-sort
```



在 `.eslintrc`中配置规则及插件

```js
module.exports = {
  ...,
  rules: {
  	...,
    'simple-import-sort/imports': 'warn',
    'simple-import-sort/exports': 'warn'
  },
  plugins: [
    ...,
    'simple-import-sort'
  ]
}
```

此刻尝试 `eslint --fix` 即可

[^1]:<a href="https://eslint.org/docs/latest/rules/sort-imports" target="_blank">Eslint - sort-imports</a>

[^2]: <a href="https://www.npmjs.com/package/eslint-plugin-simple-import-sort" target="_blank">eslint-plugin-simple-import-sort</a>
