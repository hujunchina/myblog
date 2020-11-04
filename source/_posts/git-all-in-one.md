---
title: Git操作和问题总结
date: 2020-10-23 14:07:25
categories:
	- 必备知识
tags:
	- 编程
	- GIT
---

# 1. 基本操作

Git 基本操作包括clone、init、fetch、checkout、branch、push、push等。

## 1.1 GitHub官网提供的基本操作

在GitHub上建好仓库后，需要拉到本地修改并提交：

```
git clone repourl			//	拉取远程仓库代码
git add -A						//  添加所有修改到本地
git commit -m "mark"	//	提交修改
git push							//	推到远程仓库
```

## 1.2 常规操作

```
git fetch remotename
git merge remotename/branchname
git pull remotename branchname
git init --bare my-project.git
git commit —amend
git diff --color-words
git diff master new_branch ./diff_test.txt
git remote add <remote_name> <remote_repo_url>
git push -u <remote_name> <local_branch_name>
```

## 1.3 配置文件设置

```
git config -l
git config --global user.email "your_email@example.com"
git config --global user.name <name>
git config --global color.ui false
git config --global alias.st status 
git config --global alias.co checkout 
git config --global alias.br branch 
git config --global alias.up rebase 
git config --global alias.ci commit
git config --global alias.amend 'ci —amend'
git config --global alias.unstage 'reset HEAD --'
```

## 1.4 分支操作

```
git branch <branch>        新增
git branch -d <branch>     删除
git branch -D <branch>    强制删除
git branch -m <branch>    重命名当前分支
git branch -a							列出所有分支，包括远程
```

## 1.5 合并操作

```
git checkout -b <new-branch>  从当前分支新建一个分支
git merge <branch> 把branch合并到当前所在的分支中
```

## 1.6 临时保存修改

```
保存现场
git stash (临时保存更改的文件)
恢复现场
git stash list (查看保存的更改记录)
git stash pop (像栈一样弹出第一个更改记录)
git stash pop stash@{2} (弹出第3个更改记录)
git stash drop stash@{2} (删除第3个更改记录)
```

其他操作

```
git stash    保存修改文件，新建文件不保存
git stash -u 保存所有改动文件
git stash pop弹出保存记录
git stash apply  弹出并应用所有分支的记录
git stash save "add style to our site"
git stash list
git stash show
```

## 1.7 如何打标签

```
git tag			查看标签
git tag -a "1.0" -m "myversion" 	打标签
git show v1.0 	查看标签内容
git tag -d v1.0 删除标签	
```



# 2. 基本原理

```
git blame README.md    查看所有更改记录
git log —oneline
git revert is the best tool for undoing shared public changes
git revert HEAD
git reset is best used for undoing local private changes
git reset HEAD
git clean -f  Removing untracked_file
```

Git操作

```
1. Git init->git status
2. 使用 Git stash 把不想上传远程的更改忽略掉
3. git pull 相当于git fetch + git merge （会自动合并更新）
4. 切换分支：git checkout \#some （特殊符号用转义字符）
5. 查看分支：git branch --list
```



# 3. 问题解决

## 3.1 Git 撤销上一次commit

场景：自己编写了代码并提交到远程端，但是编码的业务需求有更改，需要撤销修改。

方法：git 提供了 reset 和 revert 两种方法来处理撤销。 reset 把代码库设置为指定的 commit；revert 重新提交一次 commit。

步骤：

1. `git log -v` 	获得提交的版本号
2. `git reset --hard commit-snsjfie`  重新设置 commit
3. `git push origin HEAD --force ` 更新远程端代码

## 3.2 Git Hook 自动触发操作

自动化部署思路：在本地编写markdown文章，使用github上传修改到仓库，再自动下载到服务器。

git 有个功能叫做hook，也就是说在我们提交代码的时候会触发一些操作，这就是hook Git的挂钩（Hook）主要包含：

我们要用到的是post-update这个hook 进入到我们的git服务器的文件夹~/git/project/blog.git 进入到hook文件夹 使用ls命令可以看到许多hook脚本的sample

Hooks 文件夹是本地空白仓库才有的，需要使用`git init --bare myblog.git` 创建一个空白仓库。然后，根据提交前后的需求，改写脚本文件。

```
vim post-receive
#!/bin/sh
git --work-tree=/data/myblog --git-dir=/root/github/myblog.git checkout -f
```

修改 `post-receive` 文件，表示当push提交收到后执行下面的脚本。该脚本把代码直接放在`/data/myblog` 

