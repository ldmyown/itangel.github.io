---
layout: post
title: git 使用总结(持续更新中)
category: other
tags: [other]
---

# git 使用总结

## 说明
git是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。
git是作为一个coder经常使用的工具，作为一个代码托管工具，广泛应用在公司的代码管理上，可以非常方便的管理coder编写的代码，方便协同合作开发。
本文主要介绍git在使用过程中的一些常用命令。

## git 常用命令

**git init**
 - 在本地新建一个repo,进入一个项目目录,执行git init,会初始化一个repo,并在当前文件夹下创建一个.git文件夹.

**git clone XXX**
 - 获取一个url对应的远程Git repo, 创建一个local clone.
 - clone下来的repo会以url最后一个斜线后面的名称命名,创建一个文件夹,如果想要指定特定的名称,可以git clone [url] newname指定.

**git status**
 - 查询repo的状态.
 - git status -s: -s表示short, -s的输出标记会有两列,第一列是对staging区域而言,第二列是对working目录而言.
 
**git log**
 - 显示code的提交历史。
 - git log --oneline --number: 每条log只显示一行,显示number条.
 - git log --oneline --graph:可以图形化地表示出分支合并历史.
 - git log branchname可以显示特定分支的log.
 - git log --oneline branch1 ^branch2,可以查看在分支1,却不在分支2中的提交.^表示排除这个分支(Window下可能要给^branch2加上引号).
 - git log --decorate会显示出tag信息.
 - git log --author=[author name] 可以指定作者的提交历史.
 - git log --since --before --until --after 根据提交时间筛选log.
      --no-merges可以将merge的commits排除在外.
 - git log --grep 根据commit信息过滤log: git log --grep=keywords
      默认情况下, git log --grep --author是OR的关系,即满足一条即被返回,如果你想让它们是AND的关系,可以加上--all-match的option.
 - git log -S: filter by introduced diff.
      比如: git log -SmethodName (注意S和后面的词之间没有等号分隔).
 - git log -p: show patch introduced at each commit.
      每一个提交都是一个快照(snapshot),Git会把每次提交的diff计算出来,作为一个patch显示给你看.
      另一种方法是git show [SHA].
 - git log --stat: show diffstat of changes introduced at each commit.
      同样是用来看改动的相对信息的,--stat比-p的输出更简单一些.
      
**git add**
 - 在提交之前,Git有一个暂存区(staging area),可以放入新添加的文件或者加入新的改动. commit时提交的改动是上一次加入到staging area中的改动,而不是我们disk上的改动.
        git add .
        会递归地添加当前工作目录中的所有文件.
        
**git diff**
 - 此命令比较的是工作目录中当前文件和暂存区域快照之间的差异,也就是修改之后还没有暂存起来的变化内容.
 
**git commit**
 - 提交已经被add进来的改动.
 - git commit -m “the commit message"
 - git commit -a 会先把所有已经track的文件的改动add进来,然后提交(有点像svn的一次提交,不用先暂存). 对于没有track的文件,还是需要git add一下.
 - git commit --amend 增补提交. 会使用与当前提交节点相同的父节点进行一次新的提交,旧的提交将会被取消.
 
**git reset**
 - undo changes and commits.
 - 这里的HEAD关键字指的是当前分支最末梢最新的一个提交.也就是版本库中该分支上的最新版本.
 - git reset HEAD: unstage files from index and reset pointer to HEAD
        这个命令用来把不小心add进去的文件从staged状态取出来,可以单独针对某一个文件操作: git reset HEAD - - filename, 这个- - 也可以不加.
 - git reset --soft
        move HEAD to specific commit reference, index and staging are untouched.
 - git reset --hard
        unstage files AND undo any changes in the working directory since last commit.
        使用git reset —hard HEAD进行reset,即上次提交之后,所有staged的改动和工作目录的改动都会消失,还原到上次提交的状态.
        这里的HEAD可以被写成任何一次提交的SHA-1.
        不带soft和hard参数的git reset,实际上带的是默认参数mixed.
    
 > 总结:
        git reset --mixed id,是将git的HEAD变了(也就是提交记录变了),但文件并没有改变，(也就是working tree并没有改变). 取消了commit和add的内容.
        git reset --soft id. 实际上，是git reset –mixed id 后,又做了一次git add.即取消了commit的内容.
        git reset --hard id.是将git的HEAD变了,文件也变了.
        按改动范围排序如下:
        soft (commit) < mixed (commit + add) < hard (commit + add + local working)
        
**git revert**
 - 反转撤销提交.只要把出错的提交(commit)的名字(reference)作为参数传给命令就可以了.
 - git revert HEAD: 撤销最近的一个提交.
 - git revert会创建一个反向的新提交,可以通过参数-n来告诉Git先不要提交.
 
**git rm**
 - git rm file: 从staging区移除文件,同时也移除出工作目录.
 - git rm --cached: 从staging区移除文件,但留在工作目录中.
 - git rm --cached从功能上等同于git reset HEAD,清除了缓存区,但不动工作目录树.
 
**git clean**
 - git clean是从工作目录中移除没有track的文件.
     通常的参数是git clean -df:
     -d表示同时移除目录,-f表示force,因为在git的配置文件中, clean.requireForce=true,如果不加-f,clean将会拒绝执行.
     
**git branch**
- git branch可以用来列出分支,创建分支和删除分支.
  - git branch -v可以看见每一个分支的最后一次提交.
  - git branch: 列出本地所有分支,当前分支会被星号标示出.
  - git branch (branchname): 创建一个新的分支(当你用这种方式创建分支的时候,分支是基于你的上一次提交建立的). 
  - git branch -d (branchname): 删除一个分支.
- 删除remote的分支:
   git push (remote-name) :(branch-name): delete a remote branch.
   这个是因为完整的命令形式是:
   git push remote-name local-branch:remote-branch
   而这里local-branch的部分为空,就意味着删除了remote-branch
   
**git checkout**
 - git checkout (branchname)
  - 切换到一个分支.
    git checkout -b (branchname): 创建并切换到新的分支.
    这个命令是将git branch newbranch和git checkout newbranch合在一起的结果.
    checkout还有另一个作用:替换本地改动:
    git checkout --<filename>
    此命令会使用HEAD中的最新内容替换掉你的工作目录中的文件.已添加到暂存区的改动以及新文件都不会受到影响.
    注意:git checkout filename会删除该文件中所有没有暂存和提交的改动,这个操作是不可逆的.
    
**git merge**
- 把一个分支merge进当前的分支.
     git merge [alias]/[branch]
     把远程分支merge到当前分支.
     如果出现冲突,需要手动修改,可以用git mergetool.
     解决冲突的时候可以用到git diff,解决完之后用git add添加,即表示冲突已经被resolved.
    
**git fetch**
 - 可以git fetch [alias]取某一个远程repo,也可以git fetch --all取到全部repo
     fetch将会取到所有你本地没有的数据,所有取下来的分支可以被叫做remote branches,它们和本地分支一样(可以看diff,log等,也可以merge到其他分支),但是Git不允许你checkout到它们. 
 
**git pull**
 -  pull == fetch + merge FETCH_HEAD
    git pull会首先执行git fetch,然后执行git merge,把取来的分支的head merge到当前分支.这个merge操作会产生一个新的commit.    
    如果使用--rebase参数,它会执行git rebase来取代原来的git merge.
    
**git push**
     git push [alias] [branch]
     将会把当前分支merge到alias上的[branch]分支.如果分支已经存在,将会更新,如果不存在,将会添加这个分支.
     如果有多个人向同一个remote repo push代码, Git会首先在你试图push的分支上运行git log,检查它的历史中是否能看到server上的branch现在的tip,如果本地历史中不能看到server的tip,说明本地的代码不是最新的,Git会拒绝你的push,让你先fetch,merge,之后再push,这样就保证了所有人的改动都会被考虑进来.