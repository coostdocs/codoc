---
title: git 简记
date: 2018-08-17T22:54:44+08:00
author: Alvin
keywords: git,ssh
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
*.opendb
*.exe
*.o
*.dblite
*.db
*.pyc
.vscode
.xmake
.idea
```

* ssh key

本机生成 ssh key: 

```sh
# -f 指定文件名   -P 指定密码
ssh-keygen -t rsa -f ~/.ssh/xx -P ''  # 生成xx与xx.pub
ssh-keygen -t rsa -P ''   # 遇提示直接回车，默认在 ~/.ssh 下生成 id_rsa 与 id_rsa.pub
ssh-add ~/.ssh/xx
```

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

* 查看提交历史

```sh
git log        # 列出全部提交历史
git log -3     # 列出最近的 3 次更新
git log -p -2  # -p 显示每次提交的内容差异
```

* 撤销本地 commit 信息

```sh
# --soft 仅撤销 commit 信息，不会撤销文件的修改
git reset --soft 83518e53f8366f0126cf7a3a6fb7460d4e9d2b7f
```

* 回退到指定版本

```sh
git reset --hard 83518e53f8366f0126cf7a3a6fb7460d4e9d2b7f
git push -f
```

* 分支操作

```sh
git branch                         # 列出本地分支
git branch -r                      # 列出远程分支
git branch -a                      # 列出全部分支

git branch -d name                 # 删除本地分支
git branch -r -d origin/name       # 删除本地的远程分支
git push origin -d name            # 删除远程分支

git checkout -b new origin/xxx     # 新建并切换到本地分支
git checkout new                   # 切换到本地分支 new

git branch -m oldname newname      # 重命名本地分支
git push origin newname            # 重命名后的分支推送到远程
git push --delete origin oldname   # 删除远程旧分支
```

* tag

```sh
git tag                          # 列出所有tag
git tag -l "v1.*"                # 列出 v1. 开头 的 tag

git tag v1.0                     # 新建 tag
git tag -a v1.0 -m "notes"       # 带备注的 tag

git push origin --tags           # 推送本地所有 tag
git push origin v1.0             # 推送本地 tag

git tag -d v1.0                  # 删除本地 tag
git push origin :refs/tags/v1.0  # 删除远程 tag
```
