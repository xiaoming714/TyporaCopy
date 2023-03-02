# bash特性

```bash
history [-c] [-r] #显示历史命令 -c清空历史 -r恢复历史
echo $HISTSIZE #打印history的大小
echo $HISTFILE #history保存的位置
!历史id #执行历史id的命令
!！ #执行上次的命令
```

# 变量

```bash
#设置变量
name="wangming"
#删除变量
unset variable_name#变量被删除以后不能重复使用，unset命令不能删除只读变量

```



- 变量临时存储在内存中
- 变量和值之间不能有空格
- bash变量是弱类型
- 变量存在作用域
- sh脚本文件会开启子shell，source或.的方式则会在当前shell中加载变量，不会开启子shell
- 反引号中的命令，会被执行并且保存下来

- 单引号，所见即所得，强引用
- 双引号，输出引号中所有内容，识别特殊符号，弱引用
- 无引号，练习的符号可以不加引号，空格存在歧义，最好使用双引号
- 反引号，引用命令执行结果，等于$()用法

## 特殊变量

参数变量

```bash
$0 #获取shell脚本文件名，及脚本路径
$n #获取shell脚本的第n和参数，n在1-9之间，大于9则需要写${10}，参数空格隔开
$# #获取正在执行的shell脚本后面的参数总个数
$* #获取shell脚本所有参数，不加引号等同于$@的作用，加上引号"$*"作用是接收所有参数为单个字符串
$@ #不加引号，效果同上，加引号，是接收所有参数为独立字符串
#*和@都表示所有参数，加上""后，*会将所有参数当作一个整体来处理，@会把每个参数当作一个个体来处理
```

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20221128110623815.png" alt="image-20221128110623815" style="zoom: 50%;" />

特殊状态变量

``` bash
$? #上一次命令执行状态返回值，0正确，非0失败
$$ #当前shell脚本的进程号
$! #上一次后台进程的PID
#nohup xxx & 1>/dev/null
$_ #在此之前执行的命令的最后一个参数
```



# 环境变量

环境变量用于定义shell的运行环境、保证shell命令的正确执行

shell通过环境变量确定登录用户的用户名、PATH路径、文件系统等各种应用

环境变量可以在命令行中临时创建， 但是用户退出shell终端，环境变量就丢失了，如果需要永久保存则需要修改配置文件

- 用户个人配置文件在`~/.bash_profile`、`~/.bashrc`远程登录用户特有文件，以个人配置优先加载
- 全局配置文件`/etc/profile`、`/etc/bashrc`，且系统建议最好创建在`/etc/profile.d`否则会影响所有用户

*检查系统环境变量*

- set，输出所有变量，包括全局变量、局部变量
- env，只显示全局变量
- declare，输出所有的变量，如同set
- export，显示和设置环境变量值

bash一行中可以用分号分隔执行多条命令

# Shell子串

## bash一些基础的内置命令

```bash
echo [-n不换行输出] [-e解析字符串中的特殊符号] #输出参数内容
eval #执行多个命令
exec #不创建子进程，执行后续命令，且执行完毕后，自动exit
```

```bash
${变量} #返回变量值
${#变量} #返回变量长度
${变量：start} #返回start数值之后的字符，包含start的数字
${变量：start：length} #返回start后length长度的字符
${变量#word} #从变量开头删除最短匹配的word子串
${变量##word} #从变量开头删除最长匹配的word子串，word可以使用通配符
${变量%word} #从变量结尾删除最短的word
${变量%%word} #从变量结尾开始删除最长匹配的的word
${变量/pattern/string} #用string替代第一个匹配的pattern
${变量//pattern/string} #用string替代所有pattern
```

# 统计变量子串的长度及时间

```bash
cat test.txt | wc -l #统计当前文件有多少行
cat test.txt | wc -L #统计当前文件最长行有多少字符
expr length "{name}" #同上
awk #TODO
echo ${#name} #最快
#如何得到谁比较快呢
使用time命令循环执行
for 变量 in 序列
do
做事
done
seq [-s "分隔符"]数字#生成1到该数字的序列-s可以指定分隔符，默认是空格
```

利用#和%可以对字符串进行截取，/可以替换（第一次）//（多次）













































# 一些语法

- ！ 非运算
- -o 或运算
- -a 与运算

test命令可以用来检测某个条件是否为真

if else中的判断语句

- -gt 大于
- -ls 小于
- == 等于

set -euo pipefail

- -e 脚本发生错误就终止执行
- -u 执行脚本时遇到不存在的变量就会报错，并停止执行
- -o pipefail 多个命令用管道连接会返回最后一个命令的值并用来判断，-e就失效了，此选项用来针对管道，只要一个子命令失败，整个管道命令就失败，脚本就会终止执行

shift将参数整体向左移

[if中的选项](linux 下shell中if的“-e，-d，-f”是什么意思)

[bash中的各种括号](https://www.runoob.com/w3cnote/linux-shell-brackets-features.html)

使用>重定向如果没有则创建有则覆盖、>>则为追加模式

使用>

1. `while read name surname films;\`
2. ` do`
3. ` echo $films $name $surname >> filmsfirst;\`
4. ` done < CBActors`

- [while …; do … done](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-7.html) 是一个循环结构。当 `while` 后面的条件成立时，`do` 和 `done` 之间的部分会一直重复执行；
- [read](https://linux.die.net/man/2/read) 语句会按行读入内容。`read` 会从标准输入中持续读入，直到没有内容可读入；
- `CBActors` 文件的内容会通过 `<` 从标准输入中读入，因此 `while` 循环会将 `CBActors` 文件逐行完整读入；
- `read` 命令可以按照空格将每一行内容划分为三个字段，然后分别将这三个字段赋值给 `name`、`surname` 和 `films` 三个变量，这样就可以很方便地通过 `echo $films $name $surname >> filmsfirst;\` 来重新排列几个字段的放置顺序并存放到 `filmfirst` 文件里面了。
