---
title: Luckyexcel搭建
author: Lilihx
tags:
  - Tool
categories:
  - code
date: 2021-02-22 14:25:00
---

## 前言
>需求：局域网下多人共享excel
>Luckyexcel是国内开源的在线excel工具，可以嵌入到网站中，也可以单独部署来满足我们的需求。

<!--more-->

## 1，依赖：node安装

Luckyexcel依赖于node。因此我们首先需要配置node。

windows下配置node链接：[点击跳转node安装教程](https://blog.csdn.net/zjh_746140129/article/details/80460965)

node安装里面有重要的几步：
1，下载node安装
2，配置环境变量
3，配置npm源

安装成功的验证方式：
windows打开cmd命令行，输入node -v，回车显示版本号

输入npm -v，回车显示版本号



## 2，LuckyExcel安装

LuckySheet的官方地址：[点击跳转](https://gitee.com/mengshukeji/Luckysheet)

LuckyExcel的官方地址:[点击跳转](https://gitee.com/mengshukeji/Luckyexcel#/mengshukeji/Luckyexcel/blob/master/src/index.html)

LuckySheet是一个开源的基于node.js的在线表格编辑器，LuckyExcel在luckySheet的基础至上增加了xlsx文件的导入导出。

LuckyExcel的安装方式：<br/>
1，使用cmd命令行，cd到某个文件夹，（路径自己定义，但**尽量不要出现中文路径**）

2，cmd命令行下执行命令

`git clone https://gitee.com/mengshukeji/Luckyexcel.git `

该命令会将源代码下载到当前路径，生成一个Luckyexcel开头的文件夹

3，cmd命令行下执行命令

```shell
# 进入到特定目录
cd Luckyexcel

# npm 下载, 需要时间等待
npm install luckyexcel

# npm下载，需要时间等待
npm install -g gulp-cli

# npm安装依赖，需要时间等待
npm install

```
4, 上述命令没有报错后，证明已经安装成功

运行 **npm run dev** 命令启动

![截屏2021-02-22 下午2.46.15](http://bj.bcebos.com/ibox-thumbnail98/b33071adb2562076cd47a46b0130a641?authorization=bce-auth-v1%252Ffbe74140929444858491fbf2b6bc0935%252F2021-02-22T06%253A46%253A19Z%252F1800%252F%252Ff8b10b7d0471819c0135f37fc2dd6e2ded46f60999d26e566d632efce94e0da4)

启动后如上述截图，使用图中的倒数第三行的external链接，在浏览器中访问。

## 3，共享

上述操作在Server的3000端口启动了一个进程，如果Server的网络设置完好，同一局域网下的其余用户也可以正常访问。

如果网络未设置好，请尝试配置windows下的防火墙或者网络设置，打开3000端口供外部访问即可。

## 4，重要说明

1，上述启动之后，该Server的生命周期 就和 该cmd窗口的生命周期绑定在一起，cmd窗口不能关闭，否则会杀死进程。

2，该工具暂不支持打印。可以使用 导入 导出功能，将文件导出到本地，使用Excel打印。

3，该工具不支持新建文件，可以使用子表格的方式。

4，... ... 其余不支持的功能，看是否能用导入导出 & excel的方式解决。