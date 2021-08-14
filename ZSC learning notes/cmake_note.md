# 1 cmake基础知识

## 1.1 编译流程

GCC编译器的编译流程是：预处理、汇编、编译和链接。预处理就是对程序中的宏定义等相关内容进行前期的处理。汇编是先将C文件转换为汇编文件。当C文件转换为汇编文件就是文件编译了，编译过程就是将C原文件编译成.o结尾的目标文件。编译生成的.o文件不能直接执行，而是需要最后的链接，将很多的.o文件链接成一个完整的可执行文件。

## 1.2四个步骤

```
 gcc --help
 -E                      仅作预处理，不进行编译、汇编和链接
 -S                      编译到汇编语言，不进行汇编和链接
 -c                      编译、汇编到目标代码，不进行链接
  -o <文件>               输出到 <文件>
```
预处理
```
gcc -E main.c -o main.i  生成main.i预处理文件
```
汇编器
```
gcc -S main.i  生成main.s 编译文件
```
编译器
```
gcc -c main.s 生成main.o 链接文件
```
链接器
```
gcc main.o -o main 生成可执行文件 main
```

## 1.3 makefile语法规则

```
目标···： 依赖文件集合···
	命令1
	命令2
	···
例如
main: main.o example1.o exampl2.o
	gcc -o main main.o example1.o exampl2.o
main.o: main.c
	gcc -c main.c
example1.o: example1.c
	gcc -c example1.c
example2.o: example2.c
	gcc -c example2.c

clean:
	rm *.o
	rm main
```

### 1.3.1 变量的使用

```
objects = main.o a.o b.o
main: $(objects)
	gcc -o main $(objects)
```

$(objects)为makefile中变量

- **赋值符“=” 变量利用“=”赋值，只有最后一次有效**，例如

```
name = zsc
curname = $(name)
name =zhou shu cheng 
print:
	@echo curname: $(name)
	
打印为zhou shu cheng 所以只有最后一次“=”有效
```

- **赋值符“:=”不会使用后面的变量，使用前面定义好的变量**

```
name = zsc
curname = $(name)
name :=zhou shu cheng 
print:
	@echo curname: $(name)
	
打印为zsc 
```

- **赋值符“？=”如果前面没赋值，那么就用后面的zhoushucheng，如果前面已经赋值，就用前面的zsc**

- **追加符"+="就是追加的意思，向变量后面追加**

### 1.3.2 模式规则
%.o:%.c    通配符的意思
```
objects = main.o a.o b.o
main: $(objects)
	gcc -o main $(objects)
%.o:%.c
	gcc -c $<     #自动匹配
```

### 1.3.3 自动化变量规则

| $@   | 表示规则中的目标文件集。在模式规则中，如果有多个目标，那么，"$@"就是匹配于目标中模式定义的集合。 |
| ---- | ------------------------------------------------------------ |
| $%   | 仅当目标是函数库文件中，表示规则中的目标成员名。例如，如果一个目标是"foo.a(bar.o)"，那么，"$%"就是"bar.o"，"$@"就是"foo.a"。如果目标不是函数库文件（Unix下是[.a]，Windows下是[.lib]），那么，其值为空。 |
| $<   | 依赖目标中的第一个目标名字。如果依赖目标是以模式（即"%"）定义的，那么"$<"将是符合模式的一系列的文件集。注意，其是一个一个取出来的。 |
| $?   | 所有比目标新的依赖目标的集合。以空格分隔。                   |
| $^   | 所有的依赖目标的集合。以空格分隔。如果在依赖目标中有多个重复的，那个这个变量会去除重复的依赖目标，只保留一份。 |
| $+   | 这个变量很像"$^"，也是所有依赖目标的集合。只是它不去除重复的依赖目标。 |
| $*   | 这个变量表示目标模式中"%"及其之前的部分。如果目标是"dir/a.foo.b"，并且目标的模式是"a.%.b"，那么，"$*"的值就是"dir/a.foo"。这个变量对于构造有关联的文件名是比较有较。如果目标中没有模式的定义，那么"$*"也就不能被推导出，但是，如果目标文件的后缀是make所识别的，那么"$*"就是除了后缀的那一部分。例如：如果目标是"foo.c"，因为".c"是make所能识别的后缀名，所以，"$*"的值就是"foo"。这个特性是GNU make的，很有可能不兼容于其它版本的make，所以，你应该尽量避免使用"$*"，除非是在隐含规则或是静态模式中。如果目标中的后缀是make所不能识别的，那么"$*"就是空值。 |

### 1.3.4 伪目标

如果make后目录下有重名的文件，例如make文件，这里使用.PHONY:clean处理，生成一个伪目标，避免重名，有规则区别。

```
.PHONY : clean
clean:
	rm *.o
	rm main
```

### 1.3.5 条件判断

语句格式

```
ifxxx (arg1,arg2)
#do true
else
#do false
#endif
```

- 常用形式

  ifxxx (arg1,arg2）

- 其它合法形式
	ifxxx “arg1” “arg2”
	ifxxx ‘arg1’ ‘arg2’
	ifxxx “arg1” ‘arg2’
	ifxxx ‘arg1’ “arg2”
	

条件判断关键字
|关键字|功能|
|- |-|
|ifeq|判断参数是否相等，相等为true，否则为false|
|ifneq|判断参数是否不相等，不相等为true，否则为false|
|ifdef|判断参数是否有值，有值为true，否则为false|
|ifneq|判断参数是否没有值，没有值为true，否则为false|

### 1.3.6 函数

#### 1.3.6.1自定义函数

自定义函数的定义如下，类似python

```
define func1
    @echo "my name is $(0)"
endef
```

函数调用

- 自定义函数是一个多行变量，无法直接调用，需要使用**call**进行调用
- 自定义函数是一种过程调用，没有任何的返回值
- 自定义函数用于定义命令集合，并应用于规则中

```
define func1
    @echo "my name is $(0)"
endef

var1 := $(call func1)
```

#### 1.3.6.2 预定义函数

- make的函数提供了处理文件名，变量和命令的函数
- 可以在需要的地方**调用函数来处理指定的参数**
- 函数在**调用的地方被替换为处理结果**

```
调用方法   var := $(func_name arg1,arg2,...)
var表示返回值，func_name表示函数名，arg1，arg2表示函数实参
例如
var := $(abspath ./)
test :
   @echo "var => $(var)"
```

# 2 cmake安装

1. 打开https://cmake.org/download/，选择对应linux版本的压缩包，后缀为.tar.gz，选择对应位数

2. 然后拷贝到虚拟机上，选择一个位置进行解压，tar -zxvf cmake-3.10.0-rc4-Linux-x86_64.tar.gz

3. 将文件夹名字改为cmake   mv cmake-3.10.0-rc4-Linux-x86_64 cmake

4. 编辑环境变量，将其加入到环境变量中

5.  ```
    [root@xjfw3 ~]# vi .bash_profile 
    # .bash_profile
    # Get the aliases and functions
    if [ -f ~/.bashrc ]; then
            . ~/.bashrc
    fi
    # User specific environment and startup programs
    PATH=$PATH:$HOME/bin:/root/cmake/bin
    export PATH
    ```

6. 然后reboot 重启后开机 输入cmake --version 查看版本号，如果有则安装成功。

# 3 cmake使用

## 3.1 cmake的预定义变量

- PROJECT_SOURCE_DIR 工程的根目录
- PROJECT_BINARY_DIR 运行cmake命令的目录,通常是${PROJECT_SOURCE_DIR}/build
- CMAKE_INCLUDE_PATH 环境变量,非cmake变量
- CMAKE_LIBRARY_PATH 环境变量
- CMAKE_CURRENT_SOURCE_DIR 当前处理的CMakeLists.txt所在的路径
- CMAKE_CURRENT_BINARY_DIR target编译目录
  使用ADD_SURDIRECTORY(src bin)可以更改此变量的值
  SET(EXECUTABLE_OUTPUT_PATH <新路径>)并不会对此变量有影响,只是改变了最终目标文件的存储路径
- CMAKE_CURRENT_LIST_FILE 输出调用这个变量的CMakeLists.txt的完整路径
- CMAKE_CURRENT_LIST_LINE 输出这个变量所在的行
- CMAKE_MODULE_PATH 定义自己的cmake模块所在的路径
  SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake),然后可以用INCLUDE命令来调用自己的模块
- EXECUTABLE_OUTPUT_PATH 重新定义目标二进制可执行文件的存放位置
- LIBRARY_OUTPUT_PATH 重新定义目标链接库文件的存放位置
- PROJECT_NAME 返回通过PROJECT指令定义的项目名称
- CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS 用来控制IF ELSE语句的书写方式

系统信息

- CMAKE_MAJOR_VERSION cmake主版本号,如2.8.6中的2
- CMAKE_MINOR_VERSION cmake次版本号,如2.8.6中的8
- CMAKE_PATCH_VERSION cmake补丁等级,如2.8.6中的6
- CMAKE_SYSTEM 系统名称,例如Linux-2.6.22
- CAMKE_SYSTEM_NAME 不包含版本的系统名,如Linux
- CMAKE_SYSTEM_VERSION 系统版本,如2.6.22
- CMAKE_SYSTEM_PROCESSOR 处理器名称,如i686
- UNIX 在所有的类UNIX平台为TRUE,包括OS X和cygwin
- WIN32 在所有的win32平台为TRUE,包括cygwin

开关选项

- BUILD_SHARED_LIBS 控制默认的库编译方式。如果未进行设置,使用ADD_LIBRARY时又没有指定库类型,默认编译生成的库都是静态库 （可在t3中稍加修改进行验证）
- CMAKE_C_FLAGS 设置C编译选项
- CMAKE_CXX_FLAGS 设置C++编译选项

## 3.2 部分常用命令列表

- PROJECT
  PROJECT(projectname [CXX] [C] [Java])
  指定工程名称,并可指定工程支持的语言。支持语言列表可忽略,默认支持所有语言

- SET
  SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
  定义变量(可以定义多个VALUE,如SET(SRC_LIST main.c util.c reactor.c))

- MESSAGE
  MESSAGE([SEND_ERROR | STATUS | FATAL_ERROR] “message to display” …)
  向终端输出用户定义的信息或变量的值
  SEND_ERROR, 产生错误,生成过程被跳过
  STATUS, 输出前缀为—的信息
  FATAL_ERROR, 立即终止所有cmake过程

- ADD_EXECUTABLE
  ADD_EXECUTABLE(bin_file_name ${SRC_LIST})
  生成可执行文件

- ADD_LIBRARY
  ADD_LIBRARY(libname [SHARED | STATIC | MODULE] [EXCLUDE_FROM_ALL] SRC_LIST)
  生成动态库或静态库
  SHARED 动态库
  STATIC 静态库
  MODULE 在使用dyld的系统有效,若不支持dyld,等同于SHARED
  EXCLUDE_FROM_ALL 表示该库不会被默认构建

- SET_TARGET_PROPERTIES
  设置输出的名称,设置动态库的版本和API版本

- CMAKE_MINIMUM_REQUIRED
  CMAKE_MINIMUM_REQUIRED(VERSION version_number [FATAL_ERROR])
  声明CMake的版本要求

- ADD_SUBDIRECTORY
  ADD_SUBDIRECTORY(src_dir [binary_dir] [EXCLUDE_FROM_ALL])
  向当前工程添加存放源文件的子目录,并可以指定中间二进制和目标二进制的存放位置
  EXCLUDE_FROM_ALL含义：将这个目录从编译过程中排除

- SUBDIRS
  deprecated,不再推荐使用
  (hello sample)相当于分别写ADD_SUBDIRECTORY(hello),ADD_SUBDIRECTORY(sample)

- INCLUDE_DIRECTORIES

  INCLUDE_DIRECTORIES([AFTER | BEFORE] [SYSTEM] dir1 dir2 … )

  向工程添加多个特定的头文件搜索路径,路径之间用空格分隔,如果路径包含空格,可以使用双引号将它括起来,默认的行为为追加到当前头文件搜索路径的后面。有如下两种方式可以控制搜索路径添加的位置：

  - CMAKE_INCLUDE_DIRECTORIES_BEFORE,通过SET这个cmake变量为on,可以将添加的头文件搜索路径放在已有路径的前面
  - 通过AFTER或BEFORE参数,也可以控制是追加还是置前

- LINK_DIRECTORIES
  LINK_DIRECTORIES(dir1 dir2 …)
  添加非标准的共享库搜索路径

- TARGET_LINK_LIBRARIES
  TARGET_LINK_LIBRARIES(target lib1 lib2 …)
  为target添加需要链接的共享库

- ADD_DEFINITIONS
  向C/C++编译器添加-D定义
  ADD_DEFINITIONS(-DENABLE_DEBUG -DABC),参数之间用空格分隔

- ADD_DEPENDENCIES
  ADD_DEPENDENCIES(target-name depend-target1 depend-target2 …)
  定义target依赖的其他target,确保target在构建之前,其依赖的target已经构建完毕

- AUX_SOURCE_DIRECTORY
  AUX_SOURCE_DIRECTORY(dir VAR)
  发现一个目录下所有的源代码文件并将列表存储在一个变量中
  把当前目录下的所有源码文件名赋给变量DIR_HELLO_SRCS

- EXEC_PROGRAM
  EXEC_PROGRAM(Executable [dir where to run] [ARGS <args>][OUTPUT_VARIABLE ] [RETURN_VALUE <value>])
  用于在指定目录运行某个程序（默认为当前CMakeLists.txt所在目录）,通过ARGS添加参数,通过OUTPUT_VARIABLE和RETURN_VALUE获取输出和返回值,如下示例

- INCLUDE
  INCLUDE(file [OPTIONAL]) 用来载入CMakeLists.txt文件
  INCLUDE(module [OPTIONAL])用来载入预定义的cmake模块
  OPTIONAL参数的左右是文件不存在也不会产生错误
  可以载入一个文件,也可以载入预定义模块（模块会在CMAKE_MODULE_PATH指定的路径进行搜索）
  载入的内容将在处理到INCLUDE语句时直接执行

- FIND_

  - FIND_FILE(<VAR> name path1 path2 …)
    VAR变量代表找到的文件全路径,包含文件名
  - FIND_LIBRARY(<VAR> name path1 path2 …)
    VAR变量代表找到的库全路径,包含库文件名
  - FIND_PATH(<VAR> name path1 path2 …)
  VAR变量代表包含这个文件的路径
  - FIND_PROGRAM(<VAR> name path1 path2 …)
  VAR变量代表包含这个程序的全路径
  - FIND_PACKAGE(<name> [major.minor] [QUIET] [NO_MODULE] [[REQUIRED | COMPONENTS] [componets …]])
  用来调用预定义在CMAKE_MODULE_PATH下的Find<name>.cmake模块,你也可以自己定义Find<name>
  模块,通过SET(CMAKE_MODULE_PATH dir)将其放入工程的某个目录供工程使用

## 3.3 demo例子

### 3.3.1 单目录单文件

```shell
AKE_MINIMUM_REQUIRED(VERSION 3.10) #声明版本

PROJECT(demo1) #指定工程名称,并可指定工程支持的语言。

ADD_EXECUTABLE(demo1  main.cpp)   #生成可执行文件

```

### 3.3.2 单目录多文件

```shell
CMAKE_MINIMUM_REQUIRED(VERSION 3.10) #版本

PROJECT(demo2)

AUX_SOURCE_DIRECTORY(./ DIR_SRCS)  #发现一个目录下所有的源代码文件并将列表存储在一个变量中把当前目录下的所有源码文件名赋给变量DIR_HELLO_SRCS   变量名随便写

ADD_EXECUTABLE(demo2  ${DIR_SRCS})  #makefile是$() cmake是${}

```

### 3.3.3 多目录多文件

文件目录如下，需要每个目录写出CMakeLists.txt

── CMakeLists.txt
├── demo.cpp
└── mylib
    ├── CMakeLists.txt
    ├── mymath.cpp
    └── mymath.hpp

 mylib中的CMakeLists.txt

```shell
aux_source_directory(. DIR_LIB_SRCS) #当前目录下的所有文件打包成变量

add_library(Mylib STATIC ${DIR_LIB_SRCS}) #添加静态库，名字为Mylib  SHARED 动态库  实时更新
	#STATIC 静态库  不会更新
```

写顶层CMakeLists.txt

```shell
CMAKE_MINIMUM_REQUIRED(VERSION 3.10)

PROJECT(demo3)

ADD_SUBDIRECTORY(./mylib)  #向当前工程添加存放源文件的子目录,并可以指定中间二进制和目标二进制的存放位置

AUX_SOURCE_DIRECTORY(./ DIR_SRCS)

ADD_EXECUTABLE(demo3  ${DIR_SRCS})

TARGET_LINK_LIBRARIES(demo3 Mylib)  #为target添加需要链接的共享库

```

### 3.3.4 多目录多文件之标准工程格式

├── CMakeLists.txt
├── mylib
│   ├── CMakeLists.txt
│   ├── mymath.cpp
│   └── mymath.hpp
└── src
    ├── CMakeLists.txt
    └── demo.cpp

src下的CMakeLists.txt

```shell
INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/mylib) #向工程添加多个特定的头文件搜索路径,路径之间用空格分隔,如果路径包含空格,可以使用双引号将它括起来,默认的行为为追加到当前头文件搜索路径的后面。 ${PROJECT_SOURCE_DIR}表示根目录 也可以用../

SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)#设置输出路径为根目录下的bin

AUX_SOURCE_DIRECTORY(./ DIR_SRCS) #设置变量

ADD_EXECUTABLE(demo4 ${DIR_SRCS}) #生成可执行文件

TARGET_LINK_LIBRARIES(demo4 Mylib) #为target添加需要链接的共享库

```

 mylib下的CMakeLists.txt

```shell
AUX_SOURCE_DIRECTORY(. DIR_LIB_SRCS)

SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)#设置输出路径为根目录下的lib

ADD_LIBRARY(Mylib STATIC ${DIR_LIB_SRCS})

```

最顶层的CMakeLists.txt

```shell
CMAKE_MINIMUM_REQUIRED(VERSION 3.10)

PROJECT(demo3)

ADD_SUBDIRECTORY(./mylib)

ADD_SUBDIRECTORY(./src)

```

### 

