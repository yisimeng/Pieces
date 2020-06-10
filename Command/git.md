# Git 

开源的分布式版本控制系统。

## Git 与 SVN 区别

GIT不仅仅是个版本控制系统，它也是个内容管理系统(CMS),工作管理系统等。

如果你是一个具有使用SVN背景的人，你需要做一定的思想转换，来适应GIT提供的一些概念和特征。

1. GIT是分布式的，SVN不是：这是GIT和其它非分布式的版本控制系统，例如SVN，CVS等，最核心的区别。

2. GIT把内容按元数据方式存储，而SVN是按文件：所有的资源控制系统都是把文件的元信息隐藏在一个类似.svn,.cvs等的文件夹里。

3. GIT分支和SVN的分支不同：分支在SVN中一点不特别，就是版本库中的另外的一个目录。

4. GIT没有一个全局的版本号，而SVN有：目前为止这是跟SVN相比GIT缺少的最大的一个特征。

5. GIT的内容完整性要优于SVN：GIT的内容存储使用的是SHA-1哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏。

## 基础概念

**工作区：**当前看到操作文件目录。

**版本区：**`.git`的隐藏目录，历史版本信息。

**暂存区：**`.git/index`，当前修改部分暂存。

## 基础命令

1. **git clone remoteURL localPath** 从远程地址克隆到本地目录。
2. **git add** 参数可选。
	* 文件路径：修改提交至暂存区；
	* "."：将所有修改提交至暂存区。
3. **git commit -m '内容描述'** 从暂存区提交到版本区。
4. **git pull origin master** 从远程指定分支拉取最新代码。
5. **git push origin master** 将本地版本库代码推送到远程指定分支。
6. **git status** 查看当前工作区和暂存区修改文件。
7. **git diff file** 查看文件修改内容。
8. **git branch name** 创建name分支。
9. **git checkout** 参数可选。
	* 分支名：切换分支；
	* 文件路径：撤销该文件修改；
	* "."：撤销所有文件修改。
10. **git log** 输出所有的版本信息。
11. **git tag** 打标签。

## 进阶命令

### git cherry-pick

场景：一个仓库中的多个分支，在其中一个分支中修改了部分通用的代码，其他分支也需要修改，直接merge会包含所有修改，如果只想合并当前某次的修改，该怎么做？

有两种方案：

* **git rebase** 变基。但是可能需要所有分支先进行回退。（未测试，待验证）
* **git cherry-pick** 挑选合并，通过指定 commit id 进行合并。

**git cherry-pick 用法：** 

单个commit： git cherry-pick commit_id
多个commit： git cherry-pick commit_id1..commit_id10 或 git cherry-pick commit_id1^..commit_id10

多个commit时，两个的差异在于`^`符号，表示为半开区间，即不包含commit1，包含commit10。

拉取多个commit时，中间有可能会有冲突，遇到冲突会停止，即后面的不会继续同步。解决冲突并add之后可以选择是否继续同步：

```
--quit                end revert or cherry-pick sequence 停止
--continue            resume revert or cherry-pick sequence 继续
--abort               cancel revert or cherry-pick sequence 取消
```

多个commit时建议使用rebase（待验证）。

### git reflog

输出修改 HEAD 的记录。 git log 是输出commit的记录。

背景：在一次 reset 到错误的 commit id 后，没有记录最新的 commit id，差点重写后面所有修改。

### git commit --amend

修改最后一次 commit 信息。

### git stash

存储当前的修改。项目已经进行了一部分，此时要切换到去做其他的紧急修改，但又不想放弃已经做完的工作，可以用此命令，将修改部分进行存储，并且可以进行多次存储。

```
git stash list 查看所有的存储列表。
git stash pop 将存储栈顶的修改覆盖到当前工作区。
git stash apply stash@{num} 使用指定存储覆盖到工作区，apply后不跟参数等同于pop。
git stash apply --index 待补充。
git stash drop stash@{num} 移除指定存储。
```

### git push origin master --force

在后面添加 '--force',强制推送到远程，覆盖远程仓库。

### git show commit_id

显示某个 commit_id 中的所有修改，后面可以跟具体的 filename。

## 扩展 git-extras

> GIT utilities -- repo summary, repl, changelog population, author commit percentages and more

一个 git 的扩展库，扩展了更多的git功能。git-changelog、git rename-branch、git rename-tag等。

安装：brew install git-extras

[扩展库命令详解](https://github.com/tj/git-extras/blob/master/Commands.md)

#### 单行输出git log

使用以下代码，为git log 定义别名：git lg

```bash
git config --global alias.lg "log --no-merges --color --graph --date=format:'%Y-%m-%d %H:%M:%S' --pretty=format:'%C(yellow)%h%Creset -%C(green)%d %s %C(cyan)(%cd) %C(dim green)<%an>%Creset' --abbrev-commit"
```

### git rebase 流程

#### 将某些修改添加到之前的某次commit中。

1. git rebase <指定commit的前一个commit id> --interactive
2. 将需要改动的commit 前的 pick改为 edit，保存退出
3. 现在已经切换到指定的commit中了，进行修改。
4. git add <修改的文件>
5. git commit --amend
6. git rebase --continue



## 参考

http://www.runoob.com/git/git-tutorial.html