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

