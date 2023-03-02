
# 定义变量

```go
package main

import "fmt"

func main() {
	//定义变量：var
	//定义常量：const
	//1.先定义变量在赋值，var 变量名 数据类型
	var name string
	name = "wangming"
	fmt.Println(name)
	//2.定义时直接赋值
	var name1 = "wangming"
	fmt.Println(name1)
	//3.定义直接赋值使用自动推导（最常用）
	name2 := "wangming"
	fmt.Println(name2)
    //4.平行赋值
	i, j := 10, 20
	fmt.Println(i, j)
	//交换两个变量的值
	i, j = j, i
	fmt.Println(i, j)
}
```

数据类型

```go
int, int8, int16, int32, int64
float32, float64
true/false
```

# 自增语法

Go语言中没有++i和--i，自增语法必须单独一行

# 指针

```go
package main

import "fmt"

func main() {
	//go语言在使用指针时，会使用内部的gc，开发人员不需要手动释放
	//C语言不允许返回栈上的指针，go语言可以返回栈上的指针，程序会在编译的时候就确定了变量的分配位置
	//编译的时候，如果发现有必要的话，就将变量分配到堆上
	name := "name"
	ptr := &name
	fmt.Println(*ptr)
	fmt.Println(ptr)
	//使用new关键字定义
	name2 := new(string)
	*name2 = "wangming"
	fmt.Println(name2)
	fmt.Println(*name2)
	//指针被分配在了堆上，所以可以返回
	res := testPtr()
	fmt.Println(*res)
}

//定义一个函数，返回一个string类型的指针
func testPtr() *string {
	name := "wangming"
	name2 := &name
	*name2 = "王铭"
	return name2
}
```

# go不支持的语法

```go
//go中的空指针是nil
//go的判断语句即使只有一行代码，也必须使用{}
if res == nill {
    fmt.Println("res是空指针")
}
else {
    fmt.Println("res不是空指针")
}
```

1. ++i --i 都不支持
2. 不支持地址加减
3. 不支持三目运算
4. go只有false才能代表逻辑假

# 字符串

```go
package main

import "fmt"

func main() {
	//1.定义
	name  := "wangming"
	//需要换行的时候使用反引号
	usage := `wangming
		wangming
		wangming`
	fmt.Println(name)
	fmt.Println(usage)
	//2.长度，访问
	//string没有.length，可以使用自由函数len()来计算长度
	l1 := len(name)
	fmt.Println(l1)
	//遍历字符串
	for i := 0; i < len(name) ; i++ {
		fmt.Printf("index:%d, value:%c\n", i, name[i])
	}
	//字符串拼接
	str1, str2 := "wangming", "wangming"
	fmt.Println(str1 + " " + str2)
	//常量字符串
	const name1 = "wangming"
}
```

# 定长数组

```go
package main

import "fmt"

func main() {
	//1.定义
	nums := [10]int{1,2,3,4}//常用
	//var nums = [10]int{1,2,3,4
	//var nums [10]int = [10]int{1,2,3,4}
	//2.遍历1
	for i := 0; i < len(nums); i++ {
		fmt.Println(nums[i])
	}
	//2.遍历2，修改val不会修改nums[i]，key和val实质是副本
	//val是一个临时变量，不断地被重新赋值
	for key, val := range nums {
		fmt.Println(key, val)
	}
	//只想用key或者val其中一个，可以使用_代替
	//如果两个都想忽略就不能使用推导，应该直接=
	for _, val := range nums {
		fmt.Println(val)
	}
}
```

# 不定长数组（切片、slice）

```go
package main

import "fmt"

func main() {
	//1.定义
	name := []string{"wangming", "wangming", "wangming"}
	//2.对于一个切片，不仅有长度的概念len()，还有容量的概念cap()
	//如果append超出原本容量，那么新的容量就翻倍，但是大空间可能就不是2倍而是1.几倍
	fmt.Println(cap(name))
	//3.追加，左侧必须接收
	name = append(name, "wangming")
	fmt.Println(cap(name))
	for _, val := range name {
		fmt.Println(val)
	}
	//4.想要基于name创建一个新的数组
	//本质是引用，浅拷贝，如果修改新数组的元素，旧数组的元素也会被修改
	//1.如果从0元素开始，冒号左侧的数字可以省略
	//2.同样的道理，如果截取到最后一个元素，那么冒号右边的数字可以省略
	//3.如果截取全部元素左右数组都可以省略
	name1 := name[0:2]//左闭右开
	for _, val := range name1 {
		fmt.Println(val)
	}
	//5.创建空切片的时候，明确指定切片的容量，减小运行时分配内存的开销
	//创建一个容量是20，当前长度是0的string类型切片
	name2 := make([]string, 0, 20)
	fmt.Println(len(name2))
	fmt.Println(cap(name2))
	//6.copy，深拷贝
	name3 := make([]string, len(name))
	copy(name3, name[:])
	for _, val := range name3 {
		fmt.Println(val)
	}
}
```

# map

```go
package main

import (
	"fmt"
)

func main() {
	//1.定义字典，使用之前要对map分配空间
	//学生id->名字name
	//var name map[int]string

	//2.使用make分配空间
	//name = make(map[int]string, 10)//10为分配空间大小，建议分配空间

	//3.定义时直接分配空间，最常用的方法
	name := make(map[int]string)
	name[0] = "duke"
	name[1] = "lily"
	//4.遍历map
	for key, value := range name {
		fmt.Println(key, value)
	}
	//5.判断key是否存在
	//map不存在访问越界的问题，所以访问一个不存在的key不会崩溃，会返回一个这个类型的零值
	value, ok := name[1]
	if ok {
		fmt.Println(value)
	}
	//6.删除，使用自由函数delete删除指定的key
	delete(name,1)
}
```

# 函数

```go
package main

import (
	"fmt"
)

//1.函数返回值在参数列表之后
//2.多个返回值用小括号，使用逗号分割
func test(a int, b int) (int, bool) {
	return a + b, a == b
}

//直接使用返回值的变量名字参与运算
func test1(a, b int, c string) (res int, str string) {
	res = a + b
	str = c
	//当返回值有名字的时候，可以直接写return
	return
}

func main() {
	a := 1
	b := 2
	fmt.Println(test(a, b))
	s := "string"
	fmt.Println(s)
	fmt.Println(10, 20, "china")
}
```

# 内存逃逸

```go
package main

import "fmt"

func testPtr1() *string {
	city := "深圳"
	ptr := &city
	return ptr//ptr需要从栈区逃逸到堆区
}

func main() {
	p1 := testPtr1()
	fmt.Println(*p1)
}
```

# import

