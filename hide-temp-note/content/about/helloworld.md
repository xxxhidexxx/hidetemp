---
title: "这个网站的搭建方法"
date: 2024-01-25
description: "在网络连接的过程中遇到的一些问题"
tags: []
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "misc"
comment: false
draft: true
---

这是一个备忘录，不是一个教程，只记录一些解决方法，不赘述来龙去脉。

主要目标：快速搭建一个安全免费的静态网站，并且搭建的网站要支持 LaTeX 排版。

技术难点：github 各种各样的连接超时。

***

## 前置 

时间：2024 年 1 月；系统：windows 11。

依次安装 git，go，chocolatey，hugo，版本都选最新版，并注册一个 github 账号。

***

## 原理

流程：本地建站，网站文件存入本地仓库，本地仓库关联远程仓库，剩下交给 github 托管。

具体来说，我们将博客内容写到 markdown 格式的文件中，之后 hugo 会用 go 语言将 markdown 文件转换为 html 文件，就已经可以在本地浏览网页了。我们将 hugo 生成的 html 文件提交到 github 仓库，使用 github pages 生成静态网站，就搭建了由 github 托管的静态网站。

***

## 建立远程仓库

建立两个新的仓库，分别作为源仓库和 github pages 的仓库。

其中 github pages 的仓库必须命名为 id.github.io，id 是 github 账户名。建立选项中勾选 public 和 add readme 两项，意义是将 main 作为主分支。

源仓库也勾选 readme，其余随意。这个源仓库主要是备份用的。

建立之后将源仓库克隆到本地。

***

## 本地建站

进入刚刚克隆到本地的仓库文件夹，打开终端，这个目录将会作为网站的根目录。在终端输入 `~/>  hugo new site something` 就会看到在根目录下出现了 hugo 建立的特定名称的若干文件夹和文件。

***

## 配置样式

分别键入: 

    ~/> git submodule add https://github.com/AmazingRise/hugo-theme-diary.git themes/diary
    ~/> git submodule update --remote --merge
    ~/> hugo server --themesDir ../..
    
即可看到本地的样例网站搭建成功。现在借助这个样例对网站根目录进行改写。

先将 diary 文件夹中的全部内容直接 copy 到网站根目录，选择替换。

然后将 config.toml 的内容改为样例的 config 内容，自己改标题名称，将 baseURL 改为 id.github.io，将文件保存并重命名为 hugo.toml。

最后删除 exampleSite 文件夹。

***

## 建立本地仓库

首先 `~/> hugo new helloworld.md` 建立一个 hello world 文件，然后用 `~/> hugo` 更新 /public。

我们将 /public 初始化为 git 仓库并修改默认主分支名称为 main。

    ~/public> git init -b main

这样做的意义在于 git init 的默认主分支名称为 master，而 github pages 的默认主分支名称为 main。

***

## 重建网络连接

这是这篇文章的重点，因为其他的内容比较容易在网上查到。

首先我们假设一个前提条件，就是浏览器是可以正常访问 github 的。因为这个前提条件不是这篇文章的主题。

现在我们希望通过 ssh 完成远程关联。这里遇到了两个问题：1，默认的端口 22 连接超时；2，改成端口 443 依然连接超时。

首先，在根目录打开终端的时候，设置

    set http_proxy=127.0.0.1：10809
    set https_proxy=127.0.0.1：10809

端口自己改成转发端口。记得别打空格。这样就设置好了 power shell 的代理。

其次，在 `C:/Users/someone/.ssh` 打开 git bash 输入 `touch config` 建立一个 config 文件，并在 config 中写入 

    Host github.com
    Hostname ssh.github.com
    Port 443
    User git

这一步解决的是用 ssh 连接 github 走 http 代理。

最后再来设置 git 的代理。输入 

    git config --global http.proxy 127.0.0.1:10809

这样就可以解决目前遇到的全部连接超时的问题。

***

## 完成远程关联

    ~/public> git remote add origin git@github.com:xxxhidexxx/xxxhidexxx.github.io.git

***

## 更新内容

先将本地的修改更新到 github 的源仓库。

    cd ../
    git add .
    git commit -m "something"
    git push

再将网页文件的修改更新到 github pages 仓库。

    ~/> cd public
    ~/public> git add .
    ~/public> git commit -m "something"
    ~/public> git push origin main

在第一次更新到 github pages 时如果远程与本地仓库不一致可以使用 `git pull -rebase origin main` 进行合并。

***

## 补充注释

（0）建站过程中主要参考了 https://cuttontail.blog/blog/create-a-wesite-using-github-pages-and-hugo/ 这篇文章。他写的比较详细的东西我没重复写。

（1）解决网络连接的问题主要参考了 https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port 这篇官方文档。

（2）如果我们希望在 /content 建立子目录，则需要参照 hugo 文档，将 content/ 的子文件夹按照特定的规则进行命名，例如 /content/posts。

（3）记得将 /archetypes/default.md 中的 draft 值改为 false。