```bash
#-x language filename设定文件所使用的语言，使后缀无效
gcc -x c hello.xxx
#-x none filename关闭上一个选项，让gcc根据文件后缀
gcc -x c hello.xxx -x none hello2.c
#-c只激活预处理、编译和汇编，把程序做成obj文件，生成.o的obj文件
gcc -c hello.c
#-s只激活预处理和编译，把程序编译成汇编代码，生成.s的汇编代码，可以用文本编辑器查看
gcc -S hello.c
#-E只激活预处理，不生成文件，需要把它重定向到另一个输出文件中
gcc -E hello.c > pianoapan.txt 
gcc -E hello.c | more 
#-o执行目标名称，默认编译出来的文件是a.out
gcc -o hello.exe hello.c (哦,windows用习惯了) 
#-pipe使用管道代替编译中的临时文件，在使用非gnu汇编工具的时候，可能有些问题
gcc -pipe -o hello.exe hello.c 
#-ansi关闭 gnu c中与 ansi c 不兼容的特性, 激活 ansi c 的专有特性（包括禁止一些 asm inline typeof 关键字, 以及 UNIX,vax 等预处理宏）。
#-fno-asm此选项实现 ansi 选项的功能的一部分，它禁止将 asm, inline 和 typeof 用作关键字。
#-fno-strict-prototype只对 g++ 起作用, 使用这个选项, g++ 将对不带参数的函数,都认为是没有显式的对参数的个数和类型说明,而不是没有参数。而 gcc 无论是否使用这个参数, 都将对没有带参数的函数, 认为城没有显式说明的类型。
#-fthis-is-varialble允许条件表达式的第二和第三参数类型不匹配, 表达式的值将为 void 类型。
```

