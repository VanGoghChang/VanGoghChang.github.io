---
layout: post
title: Electron 打包流程及优化方法
---

## 打包准备
由于 Electron 官方提供的打包流程比较繁杂，所以这里打算采用 electron-builder 来打包

首先 npm install electron-builder --save-dev 来安装插件

根据 electron-builder 文档所述，在项目的 package.json 下添加两条打包指令

![](/images/19_10_16/pack_0.png)

pack 只是创建包的文件目录，并不是实际打包，主要用于测试

dist 打包成可分发的格式，如 mac 下的 .dmg 和 windows 下的 .msi 等

在运行 electron 打包指令之前，首先我们先要运行 build 指令将开发的 react 项目打包到 ./build 文件夹下，这里推荐
使用 npm script 钩子添加两个指令，其意思就是在执行 pack 和 dist 指令之前先执行 react build 的工作

![](/images/19_10_16/pack_1.png)

同时将项目入口文件 main.js 中的 `win.loadURL` 方法的参数改成 build 后的文件夹路
径 `file://${path.join(__dirname, "./build/index.html")}`

## 添加打包配置

