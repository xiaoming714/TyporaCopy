# 安装

使用apt安装，但是这种方式安装的可能不是最新版本

```bash
sudo apt install cmake
```

# 单个文件案例

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo1)

# 指定生成目标
add_executable(Demo main.cc)
```

CMakeLists.txt的语法由命令、注释和空格组成，其中命令是不区分大小写的。符号 `#` 后面的内容被认为是注释。命令由命令名称、小括号和参数组成，参数之间使用空格进行间隔。

写好`CMakeList.txt`后，执行`cmake .`就会在当前文件生成一个makefile，然后执行make 加生成目标可执行文件的名字便可执行。

# 多文件案例

```cpp
./Demo2
    |
    +--- cmaketest.cc
    |
    +--- add.cc
    |
    +--- add.h
```

```cmake
# cmake最低版本号要求
cmake_minimum_required(VERSION 2.8)

# 项目信息
project(cmaketest)

# 指定生成mubiao
add_executable(demo cmaketest.cc add.cc)
```

唯一的改动就是`add_executable`中添加了一个add.cc源文件，这样没什么问题，但是随着源文件的增多，我们不能把所有源文件都通过硬编码的方式加进去，使用`aux_source_directory`命令可以查找指定目录下的所有源文件，将结果存进变量名

```cmake
aux_source_directory(<dir> <variable>)
```

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo2)

# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})
```

# 多个目录多个源文件

```cpp
./test
    |
    +--- main.cc
    |
    +--- math/
          |
          +--- add.cc
          |
          +--- add.h
```

对于这种情况，需要分别在项目根目录 test和 math 目录里各编写一个 CMakeLists.txt 文件。为了方便，我们可以先将 math 目录里的文件编译成静态库再由 main 函数调用。根目录中的 CMakeLists.txt ：

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo3)

# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 添加 math 子目录
add_subdirectory(math)

# 指定生成目标 
add_executable(Demo main.cc)

# 添加链接库
target_link_libraries(Demo MathFunctions)
```

子目录中的 CMakeLists.txt：

```cmake
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_LIB_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)

# 生成链接库
add_library (MathFunctions ${DIR_LIB_SRCS})
```