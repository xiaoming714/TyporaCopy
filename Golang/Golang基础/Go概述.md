# go语言和C语言的区别

https://hyperpolyglot.org/c

# 一、Go语言概述

## 1.工程管理

**GOPATH**

gopath是所有项目的根路径，go语言的项目，需要有特定的目录结构进行管理，一个标准的go工程需要有三个目录：使用一个名为GOPATH的环境变量来指定：

- **src**
  - 存放源代码
- bin
  - 编译过后产生的程序，使用标准命令go install之后存放的位置
- pkg
  - 缓存包

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20220830090836778.png" alt="image-20220830090836778" style="zoom: 80%;" />

**GOROOT**

存放go语言的标准库sdk：sofrware development kit

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20220830090959225.png" alt="image-20220830090959225" style="zoom:80%;" />

在goland中打开GOPATH中的src，配置完成。

<img src="C:\Users\wm\AppData\Roaming\Typora\typora-user-images\image-20220830091603938.png" alt="image-20220830091603938"  />

## 2.Hello world概览

```go
//每个go程序都要有一个package main
//一个package相当于命名空间
package main
//导入标准包fmt：format，一般用于格式化输出
import "fmt"
//函数以func开头
//一个函数的返回值不是放在func前，而是放在func后
//函数的左花括号必须与函数名同行，不能写到下一行
func main() {
    //go语言语句不需要分号结尾
    fmt.Println("Hello world")
}
```

## 3.Go语言特点

- 没有头文件概念，只有.go文件
- 强类型的语言，解释型语言（python是弱类型解释型）
- 一个Go语言的应用程序，在运行的时候是不需要依赖外部库的
  - 把执行时需要的所有库都打包到程序中
  - Go程序会比较大
  - 如果import的包在程序中没有使用那么程序不能通过编译
- Go语法是不区分平台的，在Windows编译在Linux中可以运行，需要两个环境变量来控制，所以跨平台性能比较好
  - GOOS：设定运行的平台
    - mac：GOOS = darwin
    - Linux：GOOS = linux
    - Windows：GOOS = windows
  - GOARCH：目标机器的体系架构
    - 386：GOARCH = 386

## 4.Go命令

1. go build -o test.exe main.go
   1. 编译.go文件，-o指定生成文件的命令
2. go run *.go
   1. 直接运行程序，不会编译成exe文件
3. 安装程序
   1. 拿到源码，想自己编译出exe
      1. ./configure
      2. make
      3. make install 
   2. 需配置GOBIN环境变量为项目路径下的bin目录，使用go install 将应用程序安装到GOBIN下（配置）
