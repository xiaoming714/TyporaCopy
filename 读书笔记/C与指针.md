# 数据

将char类型变量的值限定在signed和unsigned之间，可以获得最大程度的可以执行，同时又不牺牲效率。

枚举类型可以归类到整型家族，枚举类型可以和整型无差别混合在一起使用。

指针的理解：没有人会把房子的门牌号和房子里面的东西弄混

unsigned short int a;和unsigned short a;是等价的

```c
func(int i) {//默认返回int
    return i;
}
```

链接属性一共有3种—external(外部)、internal(内部)和none(无)。没有链接属性的标识符(none)总是被当作单独的个体，也就是说该标识符的多个声明被当作独立不同的实体。属于internal链接属性的标识符在同一个源文件内的所有声明中都指同--个实体，但位于不同源文件的多个声明则分属不同的实体。最后，属于external链接属性的标识符不论声明多少次、位于几个源文件都表示同一个实体。static和extern可以用来在声明中修改标识符的链接属性：static为标识符改为internal链接属性，extern为标识符改为external链接属性

# 操作符和表达式

在C语言中避免混用整型值和布尔值。若一个变量用于表示布尔值，不要通过把它与任何特定的值进行比较来测试这个变量是否为真值，哪怕是与`TRUE`或`FALSE`。

两个指针做减法（两个指针指向同一个数组中的元素），结果是一个`ptrdiff_t`类型的有符号整型，结果是两个指针指向内存的距离。大小于对于同一个数组的两个指针可能会得到前后关系，具体看编译器实现，标准并未定义。

**指针部分收获较大**

# 函数

```c
//可变参数
#include <stdarg.h>
float average(int nums, ...) {
    va_list var_arg;//va_list类型的变量用来处理可变参
    int count = 0;
    float sum = 0;
    va_start(var_arg, nums);//将var_arg设置为第一个可变参，第一个参数是va_list类型变量，第二个变量是最后一个有名参数，
    for (;count < nums; nums++) {
        sum += va_arg(var_arg, int);//将var_list作为int类型使用
    }
    va_end(var_arg);//完成处理可变参数
    return sum / nums;
}

```

# 数组

使用for循环遍历数组的时候，将初始值赋值给循环变量，通过*而不是下标的方式访问元素效率更高，因为下表运算需要做乘法运算，每一次循环都需要进行运算。

将某些变量声明为register会提高效率。但现在的编译器可能比程序员更懂如何分配寄存器

```c
arr[3,4];//并不是三行四列的意思，而是对逗号运算符求解	
int (*p) [10];//指向数组的指针，如果想在指针上进行运算，数组的长度不能忽略
//多维数组的长度只有第一维可以忽略，因为编译器需要计算长度
```

# 字符串

库函数有时是用汇编实现的，故效率较高，轻易不要想着自己实现

程序员通常需要保证传给字符串处理函数的参数不会溢出（还有需要传长度的字符串处理函数）

# 高级指针话题

函数指针在使用的时候并不需要解引用是因为编译器会将函数名隐式转换为函数指针，如果将函数指针解引用为函数名那么编译器还是会隐式转换回去

# 预处理器

```c
//宏定义不需要使用分号结尾，分号出现再调用这个宏的语句上
#define MALLOC(n, type) \
((type *) malloc((n) * sizeof(type)))//宏的其中一个优势在于，有些参数无法作为函数参数进行传递，如本例第二个

#define MAX( a, b ) ((a) > (b)? ( a) : (b) )//x++作为这样宏的参数将会被调用两次


```

# 输入输出函数

```c
/*以简单、统一的方式报告错误
将errno(再errno.h中定义)中保存错误代码后传递给用户程序
如果message不是NULL并指向一个非空的字符串，perror将打印出这个字符串
并打印出一条用于解释errno当前错误代码的信息*/
#include <stdio.h>
void perror ( char const *message ) ;
/*终止程序的运行
status返回给操作系统，用于告知操作系统程序是否正常完成*/
#include <stdlib.h>
void exit(int status);
```

**标准IO函数库引入了缓冲IO的概念，尽可能减少使用系统调用read和write函数的调用次数，一次提高效率**

1. 全缓冲: 其特点是需要填满缓冲区后才进行实际的 IO 操作(当然, 也可以使用 flush 对缓冲区进行冲洗), 一般对于磁盘上的文件实施的是全缓冲. 默认全缓冲的大小为 4096;
2.  行缓冲: 当输入输出遇到换行符时, 才执行 IO 操作. 当流涉及一个终端时, 通常使用行缓冲. 行缓冲的长度是固定的, 稍微小一些, 默认是 1024;
3. 不带缓冲: 标准 IO 库不对字符进行缓冲. 比如标准错误输出stderr通常是不带缓冲的, 这样即使没有换行符, 出错信息也能尽快显示. 此时缓冲的大小为 1.

如果通过printf调试程序但是没有得到想要的结果时，可以使用fflush()函数强制刷新缓存区

# 标准函数库

```c
#include<stdlib.h>//算数
//返回参数的绝对值
int abs (int value);
//长整型版本的abs
long int labs ( long int value ) ;
//把denominator除以numerator，产生商和玉树，用一个div_t结构返回
//div_t包含两个字段int quot(商)和int rem(余数)
div_t div( int numerator, int denominator );
//div的长整型版本
ldiv_t ldiv( long int numer, long int denom ) ;

#include<stdlib.h>//随机数
//返回一个0-32767之间的伪随机数，为了得到更小范围的伪随机数，就需要对结果进行取模在加上或减去偏移量
int rand(void);
//seed为种子，通常为(unsigned int)time(0)
void srand(unsigned int seed);
```

