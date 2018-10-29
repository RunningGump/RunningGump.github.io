---
title: Hexo博客迁移到一台新电脑
date: 2018-10-29 20:39:42
categories: “Hexo”
tags: 
  - Hexo迁移
---

# 前言

由于10月份换了一台Macbook Pro，导致自己搭建的Hexo博客一直停滞了。我想人做事一定要善始善终，不要忘记当初搭建Hexo博客的初心。于是乎，就有了这一篇文章。

# 迁移思路

在已经推送到Github上的Hexo静态网页创建一个分支，利用这个分支来管理自己的Hexo环境文件。

# 迁移步骤

## 1.在旧机器上克隆Github上面生成的静态文件到本地

```bash
git clone https://github.com/username/username.github.io.git
```

## 2.针对克隆到本地的文件中，将除去`.git`文件的所有文件都删除

## 3.将旧机器中所有文件(`.gitignore`文件中包含的文件除外）拷贝到我们克隆下来的文件内

## 4. 创建并切换到一个叫hexo的分支

```bash
git checkout -b hexo
```

## 5. 提交复制过来的文件到暂存取

```bash
git add -A
```

## 6.提交

```bash
git commit -m "ceate a new branch file"
```

## 7.推送到Github

```bash
git push --set-upstream origin hexo
```

这个时候hexo分支已经创建完毕，接下来，我们在新电脑上搭建环境。

## 8.新电脑配置环境

安装node.js，根据自己电脑系统自行百度安装。

安装git，git相关教程推荐[廖雪峰老师的git教程](https://link.jianshu.com/?t=https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)。

安装hexo： `npm install -g hexo-cli`

## 9.clone远程仓库到本地

```bash
git clone -b hexo https://github.com/username/username.github.io.git
```

## 10.根据`package.json`安装依赖

```bash
npm install *** --save
```

将`***`替换为`package.json`文件内的依赖包

## 11.开始写文章

我们现在可以通过`hexo n "文章标题"` 创建一篇文章了！

## 12.  提交hexo环境文件

`git add .`

`git commit -m "some description"`

`git push origin hexo`

## 13.发布文章

```bash
hexo g -d
```

到这里，我们的Hexo博客就迁移完毕了！！以后再写文章时，只需要重复步骤11、12、13就ok啦！！