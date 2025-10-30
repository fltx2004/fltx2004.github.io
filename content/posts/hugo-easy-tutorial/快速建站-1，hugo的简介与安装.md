+++
date = '2025-09-20T15:05:57+08:00'
draft = false
title = '快速建站-1，hugo的简介与安装'
categories = ['快速建站教程']
tags = ['hugo', 'MemE', '建站', 'linux', 'chocolaty', 'git']
slug = 'Quick-website-building-1'
+++

今天我们来讲一下如何快速让你的网站上线，几乎一切都在电脑上搞定，你只专注于写作就可以了，对服务器来说不需要任何依赖，即使配置非常低的服务器也能轻松跑起来。

## 你需要

- 基础电脑操作能力

- 对命令行不恐惧

- 能举一反三

我们将使用**hugo**做一个纯静态的网站。

## 什么是hugo

- 世界上最快的网站构建框架

Hugo是最受欢迎的开源静态站点生成器之一。凭借其惊人的速度和灵活性，Hugo使构建网站再次变得有趣。

- 为速度而优化

采用 Go 编写，兼具高性能与灵活性。凭借先进的模板系统与高速资源管线，Hugo 可在数秒内渲染大型站点，往往更快。

- 灵活的框架

得益于多语言支持和强大的分类体系，Hugo 广泛用于构建文档站、落地页，以及企业、政府、非营利、教育、新闻、活动与项目类网站。

- 高速资源管线

支持图像处理（转换、调整尺寸、裁剪、旋转、调色、应用滤镜、叠加文字与图像、提取 EXIF 数据）、JavaScript 打包（树摇优化、代码分割）、Sass 处理，并对 TailwindCSS 提供出色支持。

- 内置 Web 服务器

在开发过程中使用 Hugo 的内置 Web 服务器，可即时查看内容、结构、行为和呈现方式的变更。

项目地址:[](https://github.com/gohugoio/hugo/)

## 为什么要使用hugo

- 清亮

hugo及其清亮，几乎只存储你的文章，你写几百上千篇文章也才几MB，生成的网页也超不过几十MB。

服务端不需要任何依赖，直接把生成的网页文件夹上传上去，你的网页就上线了，再也不用安装又大又笨重的数据库、php啦。

- 快速

即使文章很多，也可以在几百毫秒内直接生成。

- 写作方便

不需要登录博客后台去慢慢写，直接在电脑上写博客，告别各种网站魔板无障碍参差不齐的后台。

## 安装前的准备

### 安装git

[git官网](https://git-scm.com/)

可以通过它把将来使用的主题作为子模块添加进去，也方便文章的管理，回滚、增删、修改等。

这里建议学一些git的基础教程，不用多，会添加、提交、回退、查看提交历史记录、子模块的添加和更新就够用了，也许将来我也会开一个简单git的使用教程，嗯，也许吧。

你也可以听一下晴天老师的web编程入门，有基本的git操作，[点此收听git的安装](https://vbus.cc/src/2273)

实在不会也可以问AI，它会是你最好的老师。

#### windows

多数人用的都是windows，这里有几种方式。

##### 直接安装

可以从[此处下载git](https://git-scm.com/downloads/win)，其下载地址指向github，网络不好可以尝试用[这个地址](https://gh.yydjtc.top/https://github.com/git-for-windows/git/releases/download/v2.51.0.windows.1/Git-2.51.0-64-bit.exe)，在没速度就自行解决你的网络环境吧，你懂我的意思吧。

##### 第三方包管理器

这个就简单了，直接复制命令就行，我们一个一个说。

- chocolaty

powershell管理员输入

```
choco install git
```

搞定，多么简单优雅，这就是我喜欢包管理器的原因了。

- scoop

命令同上，就是把choco换成scoop。

```
scoop install git
```

- winget

最新版的系统肯定已经内置了这玩意，系统不是最新的或者是win10不一定有，命令行输入

```
winget install --id Git.Git -e --source winget
```

#### linux

这个大同小异了，根据你linux版本不同，包管理器的名字不同，需要的命令也不同，相信你都玩linux了，安装个git还不是小意思，就简单贴一个debian内核安装git的命令。

```
apt install git
```

#### macOS

有homebrew直接输入

```
brew install git
```

## 安装hugo

这里就不建议从github下载可执行文件安装了，因为麻烦，下载回来还要给它搞环境变量，直接放包管理器的了哈。

#### windows

方法也是多种多样，其实和git大同小异，我直接贴命令了

- chocolaty

```
choco install hugo-extended
```

- scoop

```
scoop install hugo-extended
```

- winget

```
winget install Hugo.Hugo.Extended
```

#### linux

##### 第三方包管理器

- snap

```
snap install hugo
```

- Homebrew

同时也适用于MacOS。

```
brew install hugo
```

##### linux官方存储库

- Alpine Linux

```
doas apk add --no-cache --repository=https://dl-cdn.alpinelinux.org/alpine/edge/community hugo
```

- Arch Linux

也同时包括Arch Linux的衍生版：EndeavourOS、Garuda Linux、Manjaro等。

```
pacman -S hugo
```

- Debian

也包括它的衍生版：elementary OS、KDE neon、Linux Lite、Linux Mint、Pop!_OS、Ubuntu、Zorin OS等

```
apt install hugo
```

- 其他系统

不再赘述，具体可查看[hugo文档](https://gohugo.io/installation/linux/)。

## 结语

本期文章就先说道这里吧，下篇我们讲主题的安装与配置。
