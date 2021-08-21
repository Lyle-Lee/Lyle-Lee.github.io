---
layout:     post
title:      Git Version Control
subtitle:   About Version Control
date:       2018-10-15
author:     Lyle
header-img: img/post-bg-debug.png
catalog: true
tags:
    - Git
---


>版本回退的正确姿势

# **git revert** 和 **git reset** 的区别

**sourceTree** 中 **revert** 译为**`提交回滚`**，作用为忽略你指定的版本，然后提交一个新的版本。新的版本中已近删除了你所指定的版本。

**reset** 为 **重置到这次提交**，将内容重置到指定的版本。`git reset` 命令后面是需要加2种参数的：`–-hard` 和 `–-soft`。这条命令默认情况下是 `-–soft`。

执行上述命令时，这该条commit号之 后（时间作为参考点）的所有commit的修改都会退回到git暂存区中。使用`git status` 命令可以在暂存区中看到这些修改。而如果加上`-–hard`参数，则暂存区中不会存储这些修改，git会直接丢弃这部分内容。可以使用 `git push origin HEAD --force` 强制将分区内容推送到远程服务器。


#### 代码回退 

实际工作中执行了如下两次提交：

	$ git add name.txt
	$ git commit -m "add txt"
	[master c7edbfe] add txt
	1 file changed, 1 insertion(+), 1 deletion(-)
	$ git add profile.jpg
	$ git commit -m "add img"
	[master b45959e] add img
	1 file changed, 1 insertion(+), 1 deletion(-)

现在，我们要把当前版本`add img`回退到上一个版本`add txt`，就可以使用`git reset`命令：

	$ git reset --hard HEAD^
	HEAD is now at c7edbfe add txt

默认参数 `-soft`，所有commit的修改都会退回到git暂存区
参数`--hard`，所有commit的修改直接丢弃
如果回退后想回去，只要命令行窗口还没有被关掉，可以直接找到回退之前某个版本的`commit_id`，回到这个指定版本：

	$ git reset --hard b45959e
	HEAD is now at b45959e add img

Git的版本回退速度非常快，因为Git在内部有个指向当前版本的`HEAD`指针，当你回退版本的时候，Git仅仅是把`HEAD`从指向`add img`：

<p>┌──┐</p>
<p>│HEAD│</p>
<p>└──┘</p>
<p>   │</p>
   └──> ○ add img
        │
        ○ add txt
        │
        ○ init

改为指向`add txt`：

┌──┐
│HEAD│
└──┘
   │
   │    ○ add img
   │    │
   └──> ○ add txt
        │
        ○ init

然后顺便把工作区的文件更新了。所以你让`HEAD`指向哪个版本号，你就把当前版本定位在哪。

推送到远程

	$ git push origin HEAD --force
	
	
#### 可以吃的后悔药->版本穿梭

当你回退之后关了电脑，又后悔了，想恢复到新的版本怎么办？找不到新版本的`commit_id`怎么办？

用`git reflog`打印你的每一次操作记录

	$ git reflog
	c7edbfe HEAD@{1}: reset: moving to HEAD^
	b45959e (HEAD -> master) HEAD@{2}: commit: add img
	c7edbfe HEAD@{3}: commit: add txt
	eaadf4e HEAD@{4}: commit (initial): init
	
找到你操作的id如：`b45959e`，就可以回退到这个版本
	
	$ git reset --hard b45959e
	HEAD is now at b45959e add img
	