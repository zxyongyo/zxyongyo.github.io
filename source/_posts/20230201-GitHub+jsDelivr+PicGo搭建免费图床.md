---
title: GitHub+jsDelivr+PicGo搭建免费图床
categories: 其他
tags: 其他
keywords: '图床,picgo'
description: 使用GitHub+jsDelivr+PicGo搭建免费图床
cover: https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/home_cover_1670914123147_9aaccc.jpeg
abbrlink: d97654d1
date: 2023-02-01 14:58:25
---

### 前言

因为嫌麻烦且不想花钱🥱，所以选择使用 ***GitHub + jsDelivr + PicGo*** 搭建图床......

下面是步骤，很简单🎈

### 新建 Github 仓库

打开 [https://github.com](https://github.com/)，登录后新建仓库用于存放图片

<img src="https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/202302011554079.png" alt="新建github仓库" />

仓库创建好之后，打开 [https://github.com/settings/tokens](https://github.com/settings/tokens)，生成一个token用于PicGo操作你的仓库

![生成token](https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/202302011554081.png)

![生成token](https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/202302011554082.png)

然后点击页面最底部的绿色 <span style="display: inline-block; padding: 0 16px; background: #2da44e; border-radius: 6px; color: white;">Generate token</span> 按钮，就能看到生成的token了。

**<span style="color: red;">注意：这个token只显示一次！把这个token复制一下，等下要用。</span>**

### 配置 PicGo

PicGo下载地址: [https://github.com/Molunerfinn/PicGo/releases](https://github.com/Molunerfinn/PicGo/releases)

下载安装后打开进行配置; 

- 点击`图床设置 -> Github`

- 仓库名格式为 `github用户名/仓库名`

- 分支一般选择主分支即可

- 粘贴刚刚生成的 token

- 设定自定义域名, 因为GitHub国内访问受限, 所以需要使用 jsDelivr 加速访问, 

  自定义域名设置为: `https://cdn.jsdelivr.net/gh/github用户名/仓库名`

我的GitHub用户名是 zxyongyo, 存放图片的仓库名是 pictures, 图片存放在仓库的 blog 文件夹下, 配置就是下面这样, 可以参考一下:

![配置PicGo](https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/202302011631119.png)


> [PicGo 官网 - *https://picgo.github.io/PicGo-Doc*](https://picgo.github.io/PicGo-Doc/)
> [jsDelivr 官网 - *https://www.jsdelivr.com*](https://www.jsdelivr.com/)
