# Git基础

## 使用git需要进行的最小配置

```bash
git config --global user.name 'name'
git config --global user.email 'email'
```

## 创建第一个仓库并配置用户信息

```bash
git config --local #只对某个仓库有效
git config --global #对当前用户所有仓库有效
git config --system # 对系统所有登录的用户有效
#显示config配置
git config --list --local
#清楚设置
git config --unset --local user.name
```

## 认识工作区和暂存区

```bash
git add -u #将文件的修改、删除，添加到暂存区
git add . #将文件的修改、新建，添加到暂存区
git add -A #将问价你的修改、删除、新建，添加到暂存区，等同于git add -all
```

## 给文件重命名

```bash
git mv readme readme.md #使用git进行重命名可以直接commit
```

## git log

```bash
git log #基础日志
--oneline #只显示一行基础信息
--n4 #显示四条log信息
--all #显示所有分支的log
--graph #以图的方式显示

#查看不同
git diff #可以加两个文件或者两个commit，也可以用HEAD来指代当前HEAD指向的commit
```

# 场景

## 删除不需要的分支

```bash
git branch -d 分支名 #如果分支没有和其他分支合并过，git不会删除分支，而是报出讲稿
git branch -D 分支名 #强制删除一个分支
```

## 修改commit的信息

```bash
git commit --amend #修改最新commit的massage，进入修改界面直接添加即可
git rebase -i commit号 #修改历史commit的massage，commit号为我们要修改的commit的父commit，-i表示交互式界面，将我们要修改的commit前面的单词修改为r（reword）保存，git会给出修改界面
git rebase -i commit号 #合并连续的commit，同上，将要合并的commit前的单词修改为s（squash）保存，在修改界面修改commit的massage
git rebase -i commit号 #合并间断commit，同上，将间隔的commit放到一起，被合并的commit前面单词修改为s，后git rebase --continue就可以进入修改massage界面
```

## 文件差异

```bash
git diff --cached #比较暂存区和HEAD文件的差异
git resert HEAD [-- 文件名]#让暂存区恢复成和HEAD一样
git diff [-- 文件名] #比较工作区和暂存区文件的差异
git checkout [-- 文件名] #让工作区的文件恢复为何暂存区一样
git resert --hard commit号 #清除提交
git diff [两分支、两commit号] [-- 文件] #查看不同提交的文件差异
git stash #将当前未提交修改加入暂存区
git stash apply #恢复暂存区内容，但不会删除暂存区中对应内容
git stash pop #恢复缓存内容并删除

```



# 正常的git工作流

```bash
# 首先需要对本地仓库进行初始化
git init

# 如果有远程仓库，去远程仓库找到将本地仓库和远程仓库关联起来的方法

# 如果已经和远端仓库建立关联，使用以下指令切换到最新版本
git checkout master
git pull --rebase origin master

# 版本最新之后通常需要创建自己的分支，在自己的分支上对代码进行修改
git branch -b test/feature_xxx

# coding......

# 查看当前仓库的状态
git status 

# 查看仓库的本地和远端日志
git log

# 查看文件的差异
git diff

# 将修改的文件添加到暂存区，.的意思是将当前目录所有修改添加到暂存区，也可以指定文件进行提交
git add .

# 提交到本地git仓库，-m的意思是添加commit信息，如果不写信息可能会报错
git commit -m "my first commit message"

# 将本地仓库的最新版本推送到远端，通常也是推送到自己的分支
# 这时git会返回一个链接，进入这个链接即可查看自己的代码检查情况，如果检查通过，在链接中对代码进行合并
git push origin test/feature_xxx
```



# 一些问题

```bash
# 删除已经提交的文件
# 1.直接在本地删除文件再提交
git rm <file> # 使用git rm命令，在删除文件的同时还会将这个删除操作记录下来
git commit -m""

# 2.只在暂存区中删除文件，本地工作区不做改变
git rm  <file>

# 3.如果误删了某个文件，使用以下指令恢复
git checkout -- <file> # 如果没有-- 改命令就变成了切换到另一个分支

# 回滚已经commit的代码
git reset <commit号> # 回滚到某次提交
git reset --sort # 这次提交之后的修改会退回暂存区
git reset --hard # 这次提交之后的修改不做任何保留，使用git status也是看不到记录的
git reset <file> # 回退文件，暂存区的也可以回退

# 合并commit，合并多个相似功能的commit，保持历史的整洁
git rebase -i HEAD~3 # 从HEAD版本开始数过去的三个版本
git rebase -i [commitid] # 合并commitid到现在的多个版本，不包括commitid的版本

# 清除还没有提交的修改
git checkout .
```
交互式界面的指令

![image-20230731135046226](photo/image-20230731135046226.png)

```bash
# 修改最近提交的commit信息
git commit --amend --message="message" --author="author"

# 修改历史提交的commit信息
git rebase -i commitid或者对HEAD的偏移 # 列出commit列表

# git grep 查找提交文件中的关键字
```

