# Git

```bash
#找到一个合适的地方，创建目录
$ mkdir learngit
$ cd learngit
$ pwd

#通过git init命令把这个目录编程git可以管理的仓库
$ git init
Initialized empty Git repository in D:/Software/Git/learngit/.git/
#git只能跟踪文本文件的改动

#文件放到git仓库的两步
#1.用git add告诉git把文件添加到仓库
$ git add learngit.txt

#2.用git commit把文件提交到仓库，-m后面是本次提交的说明
$ git commit -m "coding xm's first commit"
[master (root-commit) 6d454d8] coding xm's first commit
 1 file changed, 2 insertions(+)#一个文件改动，添加两行
 create mode 100644 learngit.txt
#add添加到仓库是临时的，commit是真正的

#git log查看文件的历史记录
$ git log

#版本回退-git reset
$ git reset --hard HEAD^
HEAD is now at 6d454d8 coding xm’s first commit
#只要命令行还没有关闭就可以找到版本号回到被回滚掉的版本
$ git reset --hard b43cc22
HEAD is now at b43cc22 append GPL
#git reflog查看每一条命令
$ git reflog

#工作区就是电脑中的目录，.git文件夹为版本库，其中最重要的就是stage版本库
#还有git为我们自动创建的第一个分支master，以及只想master的指针HEAD
#git add实际上是把文件修改增加到暂存区，git commit是把暂存区的内容提交到当前分支
#创建git版本库时自动创建了master分支，所以commit就是往master分支上提交更改

#git之所以优秀是因为git管理的是修改而非文件，如果修改的内容没有提交到暂存区，那么commit是不会成功的

#撤销修改
#如果文件没有放到暂存区，那么撤销为和版本库一样的版本
#如果文件已经添加到暂存区，又做了修改，那么文件就撤销为添加到暂存库的状态
$ git checkout -- learngit.txt

#删除文件，此命令会将真实的文件也删除
$ rm asd.txt
#此时如果想要从版本库中删除掉该文件，那么就git rm然后git commit
$ git rm asd.txt
$ git commit -m "rm commit"
#如果此时想要找回则可以git checkout -- asd.txt，不过如果从来没有提交到版本库那就没办法找回了

#创建远程库，将本地库与远程库进行关联
$ git remote add origin https://github.com/xiaoming714/learngit.git
#将当前分支master推送到远程库，由于远程库是空的所以-u，将本地master推送到远程，同时关联
$ git push -u origin master
#从现在开始提交到远程库就直接
$ git push origin master

#删除远程库，其实是将远程库和本地库解绑了，如果想要真正地删除远程库需要在GitHub删除
$ git remote rm origin
#删除之前推荐先查看一下远程库的信息
$ git remote -v

#从GitHub下载代码
$ git clone 项目地址
```

# 问题

## clone失败

1. Git限制了推送数据的大小导致的错误。

   1. 解决：重新设置通信缓存大小，git config --global http.postBuffer 524288000

2. GitHub.com无法访问，连接超时

   1. 分析：[怀疑连接不到github.com](http://xn--github-vp7im6zhn6auvh6n5cq70d.com)，在cmd窗口中，尝试ping一下百度。命令窗口：ping [www.baidu.com](http://www.baidu.com)

   2. 然后再ping一下github.com

   3. 如果ping不通则怀疑是本地DNS无法解析导致的。

   4. 错误解决：打开C:WindowsSystem32driversetchosts，

   5. 确实没有github.com的解析 

   6.  在文件末尾添加如下内容，并保存：192.30.255.112  [github.com](http://github.com) git 185.31.16.184 [github.global.ssl.fastly.net](http://github.global.ssl.fastly.net)

      重启cmd窗口，[继续ping一下github.com](http://xn--pinggithub-tf2pee8281eoca.com)：

3. 上述办法都尝试了还是不行，于是我这样做成功了

   1. git clone https://github.com/xxx.git
   2. 提示下载失败,可以尝试把https://换成 git://git clone git://github.com/xxx.git

## penSSL SSL_read: Connection was reset, errno 10054

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20220906195526755.png" alt="image-20220906195526755" style="zoom: 50%;" />

