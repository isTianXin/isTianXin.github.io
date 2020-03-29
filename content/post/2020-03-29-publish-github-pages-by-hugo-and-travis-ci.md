---
title: "使用 Travis CI 持续集成托管在 GitHub Pages 的 Hugo 博客 "
date: 2020-03-29T14:02:54+08:00
categories:
    - hugo
    - ci
tags:
    - hugo
    - blog
    - github pages
    - travis ci
draft: false
---

将 Hugo 博客托管到 GitHub Pages 时，官方文档提供了[使用 master 分支 /docs 文件夹发布](https://gohugo.io/hosting-and-deployment/hosting-on-github/#deployment-of-project-pages-from-docs-folder-on-master-branch)和[使用 gh-pages 分支发布](https://gohugo.io/hosting-and-deployment/hosting-on-github/#deployment-of-project-pages-from-your-gh-pages-branch)两种方式。然而当设置 GitHub Pages 中的 source 选项时，你会发现:

> User pages must be built from the master branch.

GitHub Pages 现在已经不支持指定文件夹和修改分支了。有些教程提供了[建立两个 repo]([https://medium.com/@chswei/%E5%9C%A8-github-%E9%83%A8%E7%BD%B2-hugo-%E9%9D%9C%E6%85%8B%E7%B6%B2%E7%AB%99-9c40682dfe40](https://medium.com/@chswei/在-github-部署-hugo-靜態網站-9c40682dfe40))或者[只发布 public 文件夹](https://brent-li.github.io/post/build-personal-site-with-hugo/)的方案，然而第一种管理太过繁琐，第二种有丢失本地文件的风险。因此这里我们使用 Travis CI，在同一个项目内维护两个分支，实现 push 之后博客的自动发布。

## 准备工作

### Hugo 本地建站

本文默认已经在本地搭建好了博客，并放在 blog  文件夹内。

### GitHub Pages

本文默认已经在 GitHub 建好了个人仓库。

## 使用 Travis CI 持续集成

### 创建 blog 分支

1. 进入 GitHub Pages 项目(如 `isTianxin.github.io`)，在根目录创建一个新的分支 blog

   > git checkout --orphan blog

2. 删除该分支内的所有文件(如果 GitHub Pages 项目之前生成了 CNAME, LISENCE, README.md 等文件，这里需要保留)

   > git rm rf .

3. 复制 Hugo 博客内的全部内容到当前文件夹下

4. 将之前保留的 CNAME 等文件复制到 public 文件夹(若没有该文件夹，使用 `hugo` 命令生成一个)

5. 提交分支

   > git add .
   >
   > git commit -m 'init blog branch'

### 配置 Travis CI

1. 访问 [Travis CI](https://travis-ci.org/), 使用 GitHub 登陆并授权。

2. 生成 [GitHub token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line), 并在 Travis CI 里选择 GitHub Pages项目(如 `isTianXin.github.io`), [添加环境变量](https://docs.travis-ci.com/user/environment-variables#defining-variables-in-repository-settings)`GITHUB_TOKEN`, 这个变量会在 `.travis.yml` 中使用，添加后的效果如图。

   ![travis-enviroment-var](/img/20200329/travis-enviroment-var.png)

3. 在GitHub Pages 项目 blog 分支根目录下，编写并保存 `.travis.yml`,

   ```yaml
   language: generic
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
   
   该配置的作用是在 push 到 blog 分支后，会自动将 public 文件夹下的文件复制到 master 分支，从而实现自动发布。
   
4. 推送 blog 分支到 GitHub

   >  git push origin HEAD

### 测试效果

在 GitHub Pages 项目 blog 分支下创建一遍博文，并执行 `hugo` 命令生成静态内容到 public 文件夹，并将改动 push 到 GitHub，如:

>huho new post/test.md
>
>hugo
>
>git add .
>
>git commit -m '添加测试博文'
>
>git push origin blog

打开 Travis CI, 找到对应的项目，当看到 passed, 就代表集成成功，打开你的博客网站去查看效果吧。

![travis-job](/img/20200329/travis-job.png)

## 工作流

添加 Travis CI 之后，只需要在写完博客并编译，将生成的所有文件推送到 blog 分支就可以了。