---
title: git常用命令及配置
date: 2018-08-17T22:54:44+08:00
author: Alvin
keywords: git,global ignore,ssh
---

## git 配置

* 基本

```sh
git config --global user.name  "your name"
git config --global user.email "your mail"
git config --global push.default simple
git config --global core.editor vim
git config --global core.autocrlf false
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ls "log --oneline"
git config --global alias.br branch
git config --global merge.tool vimdiff
```

* global ignore

```sh
git config --global core.excludesfile ~/.gitignore_global
```

文件 ~/.gitignore_global 内容像下面这样：

```sh
build
.*
*.o
*.dblite
*.pyc
*.idea
```

* ssh key

本机生成 ssh key: 

```sh
# -f 指定文件名   -P 指定密码
ssh-keygen -t rsa -f ~/.ssh/xx -P ''  # 生成xx与xx.pub
ssh-add ~/.ssh/xx
```

将公钥 xx.pub 中的内容复制到 github 上。

## git 常用命令

* 基本

```sh
git clone git@github.com:yourname/xx.git      # clone by ssh
git clone https://github.com/yourname/xx.git  # clone by https
git pull origin master                        # 从远端拉取master分支并merge到本地
git commit -am "xxx" files                    # 提交更改记录到本地
git push origin master                        # 本地更改推送到远端master分支
```

* 初始化本地仓库并提交到github

```sh
git init
git remote add origin git@github.com:yourname/xx.git
git add .
git commit -m 'xxx'
git push origin master
```
