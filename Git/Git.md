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

```

