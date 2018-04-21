---
title: git操作积累
permalink: gitcao-zuo-ji-lei
id: 20
updated: '2016-09-22 16:03:19'
tags:
---

git操作积累

add 后还没有commit   如何删除一些文件    git rm -r --cached fstack/static/avms/PDF.tar    也可以用于还没有add的

删除远程分支：    git push origin --delete <branchName>

回滚    git reset --hard 3628164


本地获取远程的一个分支   git checkout -b serverfix origin/serverfix

查看分支的详细信息  git branch -vv

拉取远程分支的主分支并与当前分支合并	
git pull origin master

http://blog.csdn.net/agul_/article/details/7835786


git branch -a   显示所有分支


git merge dev 现在，我们把dev分支的工作成果合并到master分支上： 

http://josh-persistence.iteye.com/blog/2215214

git工作区删除修改：
	 git checkout --  文件名  


