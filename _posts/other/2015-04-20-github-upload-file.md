---
layout: post
title: GitHub Markdown 文件上传
category: Other
tags: [github, upload]
description: upload markdown file to github
---

# Jekyll Markdown 文件上传至Github

------

首次上传至github上，己经不需要再上传到gh-pages分支，直接上传至master分支即可。

基本步骤：
>* git init
>* git clone https://github.com/wangperry/wangperry.github.com.git blog
>* cd blog
>* git checkout master(如果不是在master分支)
>* git rm -rf . （如果是第一次上传，删除默认的所有文件）
>* git commit -m "frist delete"
>* 添加文件 
>* git add .
>* git commit -a -m "first blood"
>* git push origin master

## 修改文件后，上传
>* git add new_file
>* git commit -a -m "add new file"
>* git push origin master


