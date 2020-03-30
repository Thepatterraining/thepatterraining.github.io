---
title: node-xlsx 使用node.js导出excel
date: 2018-04-04
tags: ['node','node-xlsx']
category: node
article: node-xlsx
---

# 导出excel

关于excel，相信这玩意大家都用过，我们这边，主要运营那边，对这些数据导出很有需要，之前拿php做过，因为现在改用node了，所以这边要用node来搞。

## export-xlsx

`excel-export` 这个库你们也可以用用，简单实用，我一开始用的这个，只不过后来不满足我的需求了，所以改用了`node-xlsx`

>https://github.com/functionscope/Node-Excel-Export

## node-xlsx

来说说`node-xlsx`这个东西吧，非常简单上手，我这种node新人也可以用明白，不容易啊，哈哈。

<!--more-->


### 下载安装

```node
    npm install node-xlsx
```
或者
>https://github.com/mgcrea/node-xlsx

### 使用
直接安装该模块，然后进入代码阶段，其实这个也很简单，使用起来简单粗暴

```node
    import nodeExcel from 'node-xlsx'
    import fs from 'fs'
    import path from 'path'

    const EXPORT_PATH = '../../static'

    //创建一个buffer name是你的sheet的名称，data是你的内容，一个二维数组，我是把数据库查询出来的转成了二维数组
    //data = [[name,age][章三,11]]
	let buffer = nodeExcel.build([{ name: 'mySheetName', data: arr }])
	//写入一个文件
    //filePath 是你的文件路径，自己定义一下就好了
    //fileName 是你的文件名，带后缀xlsx的哦
    let filePath = path.resolve(__dirname, EXPORT_PATH, fileName)
	fs.writeFileSync(filePath, buffer, 'binary')
```

这样就完成转换成这个文件了，接下来你可以把文件路径返回给前端，或者把文件流返回给前端