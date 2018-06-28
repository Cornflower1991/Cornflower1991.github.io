---
title: git使用
date: 2016-11-01 
tags: [git]
---

## 一、git配置
>git config 
>git config  --global
>git config  --system

Git的三个配置文件分别是**版本库级别**的配置文件（/.git/config）、**全局配置文件**（用户主目录下）和**系统级配置文件**（/etc目录下）。这个命令的作用是打开相应的配置文件，并且进行编辑。其中**版本库级别的配置文件的优先级最高**，全局配置文件次之，系统级别配置文件最低。

在全局空间中添加新的用户
>git config --global user.name "xxx"
>git config --global user.email xxx@github.com

删除git全局配置文件中的用户名
<!--more-->
>git config --unset --global user.name
>git config --unset --global user.email

### 创建ssh 

- **查看是否已经有了ssh密钥** ：$ cd ~/.ssh，如果没有密钥则不会有此文件夹，有则备份删除
- **生成密钥**：$ ssh-keygen -t rsa -C “youremail@yourcompany.com”，按3个回车，密码为空。会在 ~/.ssh/ 目录下生成 id_rsa 和 id_rsa.pub 两个文件
- **如果想要生成多个账户**：执行生成秘钥命令，**注意不要一路回车**，要给这个文件起一个名字，输入命令后第一步 输入文件的名称， 比如叫 id_rsa_github, 所以相应的也会生成一个 id_rsa_github.pub 文件。后续密码可以继续回车。
![@生成的文件 | center](http://ogzf36bsb.bkt.clouddn.com/blog/20161121/155701141.png)
- **添加私钥**：如果只有一个只需要添加一个即可
	 ssh-add ~/.ssh/id_rsa 
	 ssh-add ~/.ssh/id_rsa_github  
	>**注意**：如果执行ssh-add时提示"Could not open a connection to your authentication  agent"，可以现执行命令：$ ssh-agent bash

	然后再运行ssh-add命令,可以通过 ssh-add -l 来确私钥列表 可以通过 ssh-add -D 来清空私钥列表
- **修改配置文件**, 在 ~/.ssh 目录下新建一个config文件
>touch config
>添加内容：已添加github和gitlib为例
``` perl
# gitlabHost gitlab.com
   HostName gitlab.com
   PreferredAuthentications publickey
   IdentityFile ~/.ssh/id_rsa
# githubHost github.com
   HostName github.com
   PreferredAuthentications publickey
   IdentityFile ~/.ssh/id_rsa_github
```
- **测试**
>ssh -T git@github.com
>ssh -T git@gitlib.com
输出
Hi user! You've successfully authenticated, but GitHub does not provide shell access. 就表示成功的连上了。

## 二、新建仓库


>在当前目录新建一个Git代码库
>$ git init 

>新建一个目录，将其初始化为Git代码库
> $ git init [project-name]

 >下载一个项目和它的整个代码历史提交
> $ git clone [url]


## 三、添加、删除文件

>添加指定文件到暂存区
>$ git add ...

 >添加当前目录的所有文件到暂存区
>$ git add .

如果有多次修改只想暂存一个文件的部分改动，比如你修改了2个bug，但是只想缓存第一个修改的地方，可以使用该命令git add -p
你会有如下选择
>输入y来缓存该块
>输入n不缓存该块
>输入e来人工编辑该块
>输入d来退出或进入下一个文件
>输入s来分割这个块

>选择你需要的操作
>$ git add -p

 >删除工作区文件，并且将这次删除放入暂存区
>$ git rm [file1] [file2] ...

 >停止追踪指定文件，但该文件会保留在工作区
>$ git rm --cached [file]

> 改名文件，并且将这个改名放入暂存区
>$ git mv [file-original] [file-renamed]

## 四、代码提交

>提交暂存区到仓库区
> $ git commit -m [message]

>提交暂存区的指定文件到仓库区
>$ git commit [file1] [file2] ... -m [message]

>提交工作区自上次commit之后的变化，直接到仓库区
>$ git commit -a

>提交时显示所有diff信息
>$ git commit -v

## 五、远程同步

>下载远程仓库的所有变动
>$ git fetch [remote]

>显示所有远程仓库
>$ git remote -v

>显示某个远程仓库的信息
>$ git remote show [remote]

>增加一个新的远程仓库，并命名
>$ git remote add [shortname] [url]

>取回远程仓库的变化，并与本地分支合并
>$ git pull [remote] [branch]

>上传本地指定分支到远程仓库
>$ git push [remote] [branch]

>强行推送当前分支到远程仓库，即使有冲突
>$ git push [remote] --force

>推送所有分支到远程仓库
>$ git push [remote] --all

## 六、撤销

>恢复暂存区的指定文件到工作区
>$ git checkout [file]

 >恢复某个commit的指定文件到暂存区和工作区
>$ git checkout [commit] [file]

>恢复暂存区的所有文件到工作区
>$ git checkout .

>重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
>$ git reset [file]

>重置暂存区与工作区，与上一次commit保持一致
>$ git reset --hard

>重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
>$ git reset [commit]
 
 >重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
>$ git reset --hard [commit]

>重置当前HEAD为指定commit，但保持暂存区和工作区不变
>$ git reset --keep [commit]

>新建一个commit，用来撤销指定commit后者的所有变化都将被前者抵消，并且应用到当前分支
>$ git revert [commit]

>暂时将未提交的变化移除，稍后再移入
> git stash 
> git stash pop
> 
## 七、查看信息

>显示有变更的文件
>$ git status

>显示当前分支的版本历史
>$ git log

>显示commit历史，以及每次commit发生变更的文件
>$ git log --stat

 >搜索提交历史，根据关键词
>$ git log -S [keyword]

 >显示某个文件的版本历史，包括文件改名
> git log --follow [file]
> git whatchanged [file]

>显示指定文件相关的每一次diff
>$ git log -p [file]

>显示过去5次提交
>$ git log -5 --pretty --oneline

> 显示所有提交过的用户，按提交次数排序
>$ git shortlog -sn

>显示指定文件是什么人在什么时间修改过
>$ git blame [file]

>显示暂存区和工作区的差异
>$ git diff

 >显示暂存区和上一个commit的差异
>$ git diff --cached [file]

> 显示工作区与当前分支最新commit之间的差异
>$ git diff HEAD

 >显示两次提交之间的差异
>$ git diff [first-branch]...[second-branch]

>显示某次提交的元数据和内容变化
>$ git show [commit]

>显示某次提交发生变化的文件
>$ git show --name-only [commit]

>显示当前分支的最近几次提交
>$ git reflog
>
## 八、分支

>列出所有本地分支
>$ git branch

> 列出所有远程分支
>$ git branch -r

>列出所有本地分支和远程分支
>$ git branch -a

>新建一个分支，但依然停留在当前分支
>$ git branch [branch-name]

> 新建一个分支，并切换到该分支
>$ git checkout -b [branch]

>新建一个分支，指向指定commit
>$ git branch [branch] [commit]

>新建一个分支，与指定的远程分支建立追踪关系
>$ git branch --track [branch] [remote-branch]

> 切换到指定分支，并更新工作区
>$ git checkout [branch-name]

>切换到上一个分支
>$ git checkout -

>合并指定分支到当前分支
>$ git merge [branch]

>选择一个commit，合并进当前分支
>$ git cherry-pick [commit]

>删除分支
>$ git branch -d [branch-name]

> 删除远程分支
>  git push origin --delete [branch-name]
>  git branch -dr [remote/branch]
