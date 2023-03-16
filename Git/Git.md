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

