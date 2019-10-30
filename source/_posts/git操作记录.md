---
title: git操作记录
date: 2019-10-01 07:24:55
tags: 
    - git
---

### git配置别名
``` bash
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```
<!-- more -->
### git checkout 远程分支
```bash
1.git branch -a   查看远程分支

2.git checkout -b xxxx（本地分支名称） yyyy(上条命令查找到的远程分支的名称)

3.git branch 检查下 本地分支是否创建成功
```
### git tag 操作
```
git tag -a ［name］ -m ［msg］  添加标签
git ls-remote --tags        列出远程标签
git push origin [tagname]   推送标签
```