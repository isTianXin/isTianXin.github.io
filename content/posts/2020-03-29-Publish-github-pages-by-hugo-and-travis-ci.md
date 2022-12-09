---
title: "使用 Travis CI 持续集成托管在 GitHub Pages 的 Hugo 博客"
date: 2020-03-29T22:25:58+08:00
draft: true
toc: true
tags: 
  - hugo
  - blog
  - github pages
  - travis ci
---

将 Hugo 博客托管到 GitHub Pages 时，官方文档提供了[使用 master 分支 /docs 文件夹发布](https://gohugo.io/hosting-and-deployment/hosting-on-github/#deployment-of-project-pages-from-docs-folder-on-master-branch)和[使用 gh-pages 分支发布](https://gohugo.io/hosting-and-deployment/hosting-on-github/#deployment-of-project-pages-from-your-gh-pages-branch)两种方式。然而当设置 GitHub Pages 中的 source 选项时，你会发现:

> User pages must be built from the master branch.

GitHub Pages 现在已经不支持指定文件夹和修改分支了。有些教程提供了[建立两个 repo]([https://medium.com/@chswei/%E5%9C%A8-github-%E9%83%A8%E7%BD%B2-hugo-%E9%9D%9C%E6%85%8B%E7%B6%B2%E7%AB%99-9c40682dfe40](https://medium.com/@chswei/在-github-部署-hugo-靜態網站-9c40682dfe40))或者[只发布 public 文件夹](https://brent-li.github.io/post/build-personal-site-with-hugo/)的方案，然而第一种管理太过繁琐，第二种有丢失本地文件的风险。因此这里我们使用 Travis CI，在同一个项目内维护两个分支，实现 push 之后博客的自动发布。

## 准备工作

### Hugo 本地建站

本文默认已经在本地搭建好了博客，并放在单独文件夹下，如 site-blog

### GitHub Pages

本文默认已经在 GitHub 建好了个人仓库。

## 使用 Travis CI 持续集成

### 创建 blog 分支

1. 进入 GitHub Pages 项目(如 `isTianxin.github.io`)，在根目录创建一个新的分支 blog

   > git checkout --orphan blog

2. 删除该分支内的所有文件(如果 GitHub Pages 项目之前生成了 CNAME, LISENCE等文件，这里需要保留)

   > git rm rf .

3. 复制本地 Hugo 博客文件夹(如 site-blog )下的全部内容到当前文件夹下，若上一步有 CNAME，LISENCE 等文件，也要一起复制过来。

4. 提交分支

   > git add .
   >
   > git commit -m 'init blog branch'

### 配置 Travis CI

1. 访问 [Travis CI](https://travis-ci.org/), 使用 GitHub 登陆并授权。

2. 生成 [GitHub token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line), 并在 Travis CI 里选择 GitHub Pages项目(如 `isTianXin.github.io`), [添加环境变量](https://docs.travis-ci.com/user/environment-variables#defining-variables-in-repository-settings)`GITHUB_TOKEN`, 这个变量会在 `.travis.yml` 中使用，添加后的效果如图。

   <a data-fancybox="gallery" href="https://tva1.sinaimg.cn/large/00831rSTgy1gdavq3atmgj322u0k241l.jpg"><img src="https://tva1.sinaimg.cn/large/00831rSTgy1gdavq3atmgj322u0k241l.jpg" alt="travis-env-var"></a>

3. 在GitHub Pages 项目 blog 分支根目录下，编写并保存 `.travis.yml`,

   ```yaml
   language: go
   
   #指定 Go 版本
   go:
       - "1.13.1"
   
   # 仅在 blog 分支提交时触发持续集成
   branches:
       only:
           - blog
   
   # 安装 hugo
   install:
       - wget https://github.com/gohugoio/hugo/releases/download/v0.68.3/hugo_0.68.3_Linux-64bit.deb
       - sudo dpkg -i hugo*.deb
   
   script:
       - hugo #编译
       - cp CNAME LICENSE ./public #将CNAME等文件拷贝到public
   
   deploy:
       provider: pages
       skip_cleanup: true
       github_token: $GITHUB_TOKEN
       keep_history: true
       target_branch: master
       on:
           branch: blog
       local_dir: public
   ```

   该配置的作用是在 push 到 blog 分支后，会自动编译并将 public 文件夹下的内容发布到 master 分支，从而实现博客的持续集成。

4. 推送 blog 分支到 GitHub

   >  git add .
   >
   >  git push origin HEAD

### 测试效果

在 GitHub Pages 项目 blog 分支下创建一篇博文并 push 到 GitHub，如:

>huho new posts/test.md
>
>git add .
>
>git commit -m '添加测试博文'
>
>git push origin blog

打开 Travis CI, 找到对应的项目，当看到 passed, 就代表集成成功，打开博客网站去查看效果吧。

<a data-fancybox="gallery" href="https://tva1.sinaimg.cn/large/00831rSTgy1gdavp6w0hjj31yo0o6tdj.jpg"><img src="https://tva1.sinaimg.cn/large/00831rSTgy1gdavp6w0hjj31yo0o6tdj.jpg" alt="travis-job"></a>

## 工作流

添加 Travis CI 之后，只需要将新添加的 Markdown 文件 push 到 blog 分支，即可自动发布到博客网站。

## 踩坑记录

- Travis CI 提供的 Ubuntu 包管理 apt 内的 hugo 版本过低，直接使用 `sudo apt install hugo` 安装，在编译时会得到如下报错:

  > Current theme does not support Hugo version 0.16. Minimum version required is 0.43

  因此这里直接下载 GitHub release 包进行安装。

- 如果你的 GitHub Pages 使用了自定义域名，请 <strong><i>务必</i></strong>将 CNAME 文件拷贝到 public 目录下，即:

  > cp CNAME LICENSE ./public

   否则使用自定义域名无法访问博客。
