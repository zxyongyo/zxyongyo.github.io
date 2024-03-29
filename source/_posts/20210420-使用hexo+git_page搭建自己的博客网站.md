---
title: 使用hexo+git page搭建自己的博客网站
categories: 其他
tags:
  - 其他
  - Hexo
keywords: Hexo,博客,git
description: 使用hexo+git page搭建自己的博客网站
cover: https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/20210426170934.jpg
aside: true
abbrlink: acb03bb7
date: 2021-04-20 14:47:02
---

👉 最近偶然在网上看到一个非常漂亮的个人博客网站，突然觉得自己作为一个前端攻城狮(小菜鸡)🙈，也要有一个这样的网站；<font color="red">❤</font>动不如行动，问了度娘后找到了最简便的方法，<font color="#70a1ff">GitHub+Hexo搭建静态博客网站</font>，下面介绍搭建过程，只需简单的几步。

## Hexo 👍

生成静态博客网页的一个脚手架，[hexo官网](https://hexo.io/zh-cn/) 的api教程，插件，主题什么的都非常丰富，有耐心的同学可以去自己摸索一下，我这里只写一下怎么搞一个能跑起来的博客网站。

## 安装hexo 🧰

安装前您的电脑上必须已经安装了 [Node.js](https://nodejs.org/zh-cn/) 和 [Git](https://git-scm.com/downloads)，如果已经安装这两个工具（不会安装的请自行百度吧），那么您就可以使用npm安装Hexo了，在任意目录打开终端，执行以下命令：

```bash
$ npm install hexo-cli -g
```

## 建站 ⚔️

### 初始化Hexo目录

依次执行以下命令：

```bash
$ hexo init <folder>	# 初始化hexo目录  folder-目录名
$ cd <folder>	# 进入hexo目录
$ npm install	# 安装依赖	
```

### 目录结构

如果以上步骤都正确，那么您现在的hexo目录结构应该是：

```
.
├── node_modules
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

- _config.yml：网站的 [配置](https://hexo.io/zh-cn/docs/configuration) 信息，您可以在此配置大部分的参数。

- package.json：应用的依赖信息。
- scaffolds：[模版](https://hexo.io/zh-cn/docs/writing) 文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。
- source：资源文件夹是存放用户资源的地方。除 `_posts` 文件夹之外，开头命名为 `_` (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 `public` 文件夹，而其他文件会被拷贝过去。
- themes：[主题](https://hexo.io/zh-cn/docs/themes) 文件夹。Hexo 会根据主题来生成静态页面。

## 运行&部署 🚀

### 本地预览

在终端执行`hexo server`，执行成功后，在浏览器输入`localhost:4000`，然后回车，如果成功打开页面，那么恭喜🎉，第一步已经成功!

> 如果执行遇到了报错，可能是因为hexo默认使用的4000端口被占用了，可以使用`hexo server -p 4000`指定端口运行。

### 部署到GitHub

现在网站只是能在本地打开，如果想要随时随地的到处炫耀😎，那么我们要将它部署到GitHub上，这样就可以在任何地方打开了。

1. 新建git仓库

   在自己的 [GitHub](https://github.com)上新建一个仓库*(没有的请自行去注册/登录)*，

   ![GitHub新建仓库](https://cdn.jsdelivr.net/gh/zxyongyo/pictures/blog/20210426170628.png "GitHub新建仓库")

2. 配置hexo部署到GitHub

   用您的编辑器打开hexo根目录的`_config.yml`文件，找到最下面的`deploy`配置，添加如下配置：

   ```yaml
   deploy:
     type: git
     repo: 
       github: git@github.com:zxyo/zxyong.git # 改为您自己刚才建的仓库地址
     branch: master
   ```

   > 上面github: 的仓库地址一定要改为您自己的仓库ssh克隆的地址。
   > 使用ssh地址而不用https是为了避免每次部署的时候都需要登录github账户。[[配置ssh]()]
   
3. 部署到gitee

   由于国内对访问GitHub的限制，我们也可以同时部署到gitee，配置方法与配置github相同，然后添加一行：

   ```yaml
   deploy:
     type: git
     repo: 
       github: git@github.com:zxyo/zxyong.git # 改为您自己的仓库地址
       gitee: git@gitee.com:zxyo/zxyong.git # 改为您自己的仓库地址
     branch: master
   ```

4. 确保一切无误后，就可以使用命令部署了

   ```bash
   $ hexo clean # 清除缓存文件
   $ hexo gnerate # 生成静态文件
   $ hexo deploy # 部署到_config.yml中配置的git仓库中
   ```

   > 在GitHub或Gitee的仓库设置里开启git page，就可以访问到您的博客了✌！
   > Gitee每次部署后都需要去手动更新git page。

## 更换主题 🎨

1. hexo官网的 [主题](https://hexo.io/themes/) 有很多，可以自己去找一个喜欢的，我用的是 [Butterfly](https://butterfly.js.org/posts/21cfbf15/) 这个主题。

2. 找到这个主题的 [GitHub仓库](https://github.com/jerryc127/hexo-theme-butterfly) 或 [Gitee仓库](https://gitee.com/iamjerryw/hexo-theme-butterfly) clone下来放到你的hexo项目的themes目录下。

3. 修改`_config.yml`文件，把主题改为butterfly：

   ```yaml
   theme: butterfly
   ```

4. 安装插件，如果你没有pug和stylus渲染器，执行命令安装：

   ```bash
   npm install hexo-renderer-pug hexo-renderer-stylus --save
   ```

5. 执行`hexo clean & hexo d -g`重新打开网页就能看见更换后的样子啦！

6. 主题配置

   在`/themes/butterfly`目录下还有一个`_config.yml`配置文件，这个是你主题的配置文件，里面配置项很多可以查看 [官方文档](https://butterfly.js.org/posts/4aa8abbe/) 进行自由配置。