# Make工具

多个文件联合编译时

1. 第一种编译方式 :

```bash
//先写好 main.c calcu.c input.c calcu.h input.h文件,然后在main.c中引入头文件#include "calcu.h" #include "input.h"
//在终端输入命令 gcc main.c calcu.c input.c -o main 编译成可执行文件
//最后 ./main 输出结果
```

用这种方式如果修改了某个.c文件, 就要重新编译所有文件, 如果文件较多, 编译过程会很耗费时间



2. 第二种编译方式

先将 .c 文件全编译成 .o 文件, 最后将这些 .o 文件链接成可执行文件

```bash
//分别将.c编译成.o
gcc -c calcu.c
gcc -c input.c
gcc -c main.c
//再将这些 .o 文件链接
gcc calcu.o input.o main.o -o main
```

以后假如修改了calcu.c的内容, 只需

```bash
gcc -c calcu.c
gcc calcu.o input.o main.o -o main
```

就不用编译其他 .c 文件, 但依然有大量重复工作



3. 引入make工具

利用make工具可以自动完成编译工具, 这些工作包括 :

* 如果修改了某几个源文件 (.c文件) , 则重新编译这几个源文件
* 如果某个头文件 (.h文件) 被修改了, 则重新编译所有包含该头文件的源文件 (.c文件) , 并且链接成可执行文件

​        利用这种自动编译可以简化开发工作, 避免不必要的重新编译. make工具通过一个称为**Makefile的文件**来完成并自动维护编译工作. Makefile文件描述了整个工程的编译, 链接规则

==在工程目录下创建名为 "Makefile" 的文件, 文件名 "Makefile" (区分大小写)== 

```bash
//目录下包含
calcu.c calcu.h input.c input.h main.c Makefile 
```

Makefile和C文件**处于同一目录下**, 在Makefile文件中输入代码 :

```
目标(main): 依赖文件集合(.o文件集合)
	命令1(gcc -*)
	命令2
	...
```

```bash
/* 目标main依赖于main.o input.o calcu.o 但是这些.o文件还没生成,这些.o需要对应的.c文件编译 */

main: main.o input.o calcu.o
	gcc -o main main.o input.o calcu.o

/* 此时目标为这些.o文件,他们需要对应的命令 */
main.o: main.c
	gcc -c main.c
input.o: input.c
	gcc -c input.c
calcu.o: calcu.c
	gcc -c calcu.c
	
/* 执行清理工程命令时, 删掉所有.o文件和main */
clean:
	rm *.o
	rm main
```

当Makefile文件编写完成后, 在终端输入make命令

```bash
hjx@hjx-virtual-machine~/桌面/code$ make
gcc -c main.c
gcc -c input.c
gcc -c calcu.c
gcc -o main main.o input.o calcu.o
hjx@hjx-virtual-machine~/桌面/code$ ls
calcu.c calcu.h calcu.o input.c input.h input.o main main.c main.o Makefile
hjx@hjx-virtual-machine~/桌面/code$ ./main (即可执行main)
```

清除工程 : make clean

```bash
hjx@hjx-virtual-machine~/桌面/code$ make clean
rm *.o
rm main
hjx@hjx-virtual-machine~/桌面/code$ ls
calcu.c calcu.h input.c input.h main.c Makefile (.o和main都没了)
```

**假如修改了input.c**

```bash
hjx@hjx-virtual-machine~/桌面/code$ make
gcc -c input.c
gcc -o main main.o input.o calcu.o
```



## Makefile基本语法

规则:

```bash
目标(main 或 .o文件): 依赖文件集合(.o文件集合 或 .c文件)
	命令1(gcc -o:将.o文件链接成可执行文件 或 gcc -c:将.c文件编译成.o文件)
	命令2
	...
```

```bash
1 main: main.o input.o calcu.o
2 	 gcc -o main main.o input.o calcu.o
3
4 main.o: main.c
5    gcc -c main.c
6 input.o: input.c
7    gcc -c input.c
8 calcu.o: calcu.c
9	 gcc -c calcu.c
10 
11 clean:
12    rm *.o
13	  rm main
```

上述代码有5条规则 : 

1~2行, 4~5行, 6~7行, 8~9行, ==11~13行是自己设定的clean规则(功能)==: 输入 ==make clean==就执行下属命令

**第一条规则中的目标是默认目标**, 只要**默认目标更新**, 就视为Makefile**完成**

### Makefile变量

​	跟c语言一样可以支持变量, 比如main.o input.o calcu.o这些依赖文件重复输入浪费时间也容易出错. 所以Makefile加入了变量支持. **Makefile中的变量都是 字符串**, 类似c语言中的**宏** (要在一开始就声明)

```bash
objects = main.o input.o calcu.o

main: $(objects)
	gcc -o main $(objects)
//对比
main: main.o input.o calcu.o
    gcc -o main main.o input.o calcu.o

```

### 输出命令(echo)

```bash
//定义变量name
name = hjx
uname = $(name)
name = potatohu

//自定义功能print:
print:
	@echo username: $(name)
	
输出: username: potatohu
(如果不加'@', 输出: echo username: potatohu)
```

### 赋值命令

```bash
=		普通赋值, 变量以最后一次赋的值为准

:=		不会使用后面定义的变量,只能使用 := 前面已经定义好的
		name = hjx
		uname := $(name)
		name = potatohu
		最后输出username:hjx
		
?=		如果变量在之前没有被赋值, 则赋值=后的值, 如果被赋值了就是用之前的值

+=		给变量中加内容
		objects = main.o input.o
		objects += calcu.o
		等价于 objects = main.o input.o calcu.o
```



### 批量编译 ( 比如.c -> .o ) 模式规则和自动化变量

 1. **模式规则**

    模式规则中, 在规则的目标定义中药包含 '%' , '%' 表示对文件名的匹配, =='%' 表示任意长度的非空字符串== 

    比如 =='%.c'== 就是所有的*以 .c结尾的文件*, =='a%.c'== 就是所有*以a开头 以.c结尾的文件*

* 当 '%' 出现在**目标**中, **目标中 '%' 代表的字符串决定了依赖中 '%' 的字符串(一一对应)**

```bash
objects = main.o input.o calcu.o

main: $(objects)
	gcc -o main $(objects)

%.o : %.c
	#命令 不能使用%代替, 需引入自动化变量
	
clean: 
	rm *.o
	rm main
```

2. **自动化变量**

   目标和依赖都是一系列文件, 每次对模式规则进行解析的时候都会是不同的目标和依赖文件, 而命令只有一行, **自动化命令可以通过一行命令从不同的一来文件中生成对应目标**, 常用的有三种 : 

| 自动化变量 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| $@         | 规则中的目标集合, 在模式规则中, 如果有多个目标的话, $@表示匹配模式中定义的目标集合 |
| $<         | 依赖文件集合中的第一个文件, 如果依赖文件是以模式 ( % ) 定义的, 那么$<就是符合模式的一系列的文件集合 |
| $^         | 所有依赖文件的集合, 使用空格分开, 如果在依赖文件中有多个重复文件, $^会去除重复的依赖文件只保留一份 |

```bash
objects = main.o input.o calcu.o

main: $(objects)
	gcc -o main $(objects)

%.o : %.c
	gcc -c $<
	
clean: 
	rm *.o
	rm main
```

```bash
对比:
main: main.o input.o calcu.o
	gcc -o main main.o input.o calcu.o

main.o: main.c
	gcc -c main.c
input.o: input.c
	gcc -c input.c
calcu.o: calcu.c
	gcc -c calcu.c
	
clean:
	rm *.o
	rm main
```



### 伪目标

有时需要编写一个规则来执行一些命令, 但是这个规则不是用来创建文件, 比如

```bash
clean:
	rm *.o
	rm main
```

这个规则中并没有创建文件clean的命令, 因此工作目录下不会有clean, 当输入 

```bash
make clean
```

后, 后面的

```bash
rm *.o
rm main
```

命令会被执行. 但如果在工作目录下创建一个名为clean的文件, 执行make clean就不会执行rm命令, 清理功能也无法实现. 

* 为了避免这个问题, 可以将clean**声明为伪命令**

```bash
.PHONY: clean
```

```bash
objects = main.o input.o calcu.o

main: $(objects)
	gcc -o main $(objects)

%.o : %.c
	gcc -c $<
	
.PHONY: clean
clean: 
	rm *.o
	rm main
```



## Makefile函数使用

Makefile中的函数时已经定义好的, 直接使用, 不支持自定义

用法:

```bash
$(函数名 参数1, 参数2, 参数3)
// $(函数名 参数集合)
```

或

```bash
${函数名 参数集合}
```

