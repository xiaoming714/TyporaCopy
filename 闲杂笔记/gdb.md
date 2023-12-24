# 启动调试

GDB（GNU Debugger）是UNIX及UNIX-like下的强大调试工具，可以调试Ada, c, c++, asm, minimal, d, FORTRAN, objective-c, go, Java,pascal等语言。本文以C程序为例，介绍GDB启动调试的多种方式。

对于C程序来说，需要在编译时加上-g参数，保留调试信息，否则不能使用GDB进行调试。
但如果不是自己编译的程序，并不知道是否带有-g参数，可以gdb 可执行文件查看是否有调试标记


```bash
#调试启动无参程序
gdb helloWorld
$ gdb gdbStep    #启动调试
(gdb)#输入run命令即可运行程序

#有参程序启动调试
gdb hello
(gdb)run 参数#程序启动
#或者
gdb hello
(gdb) set args 编程珠玑
(gdb) run

#查看是否会产生core文件
ulimit -c#如果是0则表示不产生core文件
ulimit -c unlimied  #表示不限制core文件大小
ulimit -c 10        #设置最大大小，单位为块，一块默认为512字节

#调试已经运行的程序
ps -ef|grep 进程名#得到进程id
gdb
(gdb) attach 进程id
#已运行的程序没有调试信息
gdb
(gdb) file hello
Reading symbols from hello...done.
(gdb)attach 进程id
```

# 断点

```bash
#查看已经设置的断点
info breakpoints
#根据行号设置断点
b 9  #break 可简写为b
b test.c:9 #使用以上两个指令程序运行到第九行就会停下来
#根据函数名设置断点
b printNum
#根据条件设置断点
break test.c:23 if b==0#当b=0的时候程序就会在23行停下来
condition 1 b==0#如果b=0会产生断点1
#根据规则设置断点
rbreak printNum*#对所有调用printNum函数都设置断点
#对所有函数设置断点
#用法：rbreak file:regex
rbreak . 
rbreak test.c:. #对test.c中的所有函数设置断点
rbreak test.c:^print #对以print开头的函数设置断点
#设置临时断点
tbreak test.c:l0  #在第10行设置临时断点
#跳过多次设置断点
ignore 1 30#跳过1断点30次  通过info breakpoints可以看到断点
#根据表达式值变化产生断点
watch a#当a的值发生变化时会打印相关内容
#禁用或启用断点
disable  #禁用所有断点
disable bnum #禁用标号为bnum的断点
enable  #启用所有断点
enable bnum #启用标号为bnum的断点
enable delete bnum  #启动标号为bnum的断点，并且在此之后删除该断点
#断点清除
clear   #删除当前行所有breakpoints
clear function  #删除函数名为function处的断点
clear filename:function #删除文件filename中函数function处的断点
clear lineNum #删除行号为lineNum处的断点
clear f:lename：lineNum #删除文件filename中行号为lineNum处的断点
delete  #删除所有breakpoints,watchpoints和catchpoints
delete bnum #删除断点号为bnum的断点
```

# 变量查看

再查看变量之前，需要启动调试并设置断点

```bash
#基本变量查看
(gdb) p a#打印变量a的值（print）
(gdb) p 'testGdb.h'::a#通过函数名或文件名来区分多个函数或多个文件中的同一个变量名的变量
 #打印指针执行的内容
 (gdb) p *d#这样只能打印一个值，如果需要打印数组则需要用@加上打印的长度或变量
 (gdb) p *d@a#打印数组中的a个元素
 #打印数组的更好实践
 (gdb) set $index=0
(gdb) p b[$index++]
$11 = 1
(gdb) p b[$index++]
$12 = 2
(gdb) p b[$index++]
$13 = 3
#自动显示变量内容
(gdb) display e
#查看寄存器内容
(gdb)info registers
```



# 单步调试

```bash
#列出源代码
(gdb) list(l)
#单步执行
(gdb) n     #单步执行
(gdb) n 2   #执行两次
#单步进入-step
(gdb) s     #单步进入
(gdb) finish    #继续完成该函数调用
#继续执行到下一个断点-continue
(gdb) c      #继续运行，直到下一次断住
#继续运行到指定位置-until
(gdb) u 29#运行到29行停住
#跳过执行—skip
(gdb) skip function add    #step时跳过add函数
(gdb)skip file gdbStep.c#也可以跟文件，这样文件中的函数都不会进入
```

# 源码查看

```bash
#直接查看源码
(gdb) l#直接输入l可从第一行开始显示源码，继续输入l，可列出后面的源码。后面也可以跟上+或者-，分别表示要列出上一次列出源码的后面部分或者前面部分。
#列出指定行附近源码
(gdb) l 9#列出9行附近的源码
#列出指定函数附近的源码
(gdb) l printNum#后接函数名
#设置源码一次列出行数
(gdb) set listsize 20
(gdb) show listsize
#列出指定行之间的源码
(gdb) l 3,15#列出3到15行之间的源码，省略结束行的时候，它列出从开始行开始，到指定大小行结束，而省略开始行的时候，到结束行结束，列出设置的大小行
#列出指定文件的源码l location，其中location可以是文件名加行号或函数名
(gdb) l test.c:1

```
