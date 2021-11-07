# GDB的基本使用

## 一、GDB的安装

```shell
[root@localhost gdb]# yum install gdb
[root@localhost gdb]# gdb -v
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-120.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
```

## 二、GDB的启动

本篇以demo.c文件为例，展示GDB的基本调试功能

demo.c

```c
#include <stdio.h>
int main(){
    int num,result=0,i=0;
    num = 100;
    while(i<=num){
        result += i;
        i++;
    }  
    printf("result=%d\n", result);
    return 0;
}

```

首先生成gdb可执行文件，命令为

`[root@localhost gdb]# gcc -g demo.c -o demo`

`[root@localhost gdb]# gdb demo`

## 三、GDB基本使用操作

### 3.1 列出行信息

list或者l 列出代码前10行，继续输入list或者回车则继续展示代码，也可以输入**list n（n代表行号） 或 list 函数名**

```shell
(gdb) list
1	#include <stdio.h>
2	int main(){
3	    int num,result=0,i=0;
4	    num = 100;
5	    while(i<=num){
6	        result += i;
7	        i++;
8	    }  
9	    printf("result=%d\n", result);
10	    return 0;
(gdb) 
11	}
(gdb) 

```

### 3.2 断点相关

| (gdb) break n      (gdb) break 函数名       (gdb) b n     (gdb) b 函数名        (gdb) b n(函数名）if 条件 | n为行号，添加if表示设置条件断点，只有条件为真时，才中断 |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| (gdb)  info breakpoints    (gdb) i breakpoints               | 查看断点                                                |
| (gdb) delete breakpoints n                                   | 删除断点                                                |
| (gdb) disable breakpoints n                                  | 使断点失效                                              |
| (gdb) enable breakpoints n                                   | 使断点生效                                              |



### 3.3 执行相关

| (gdb) run          (gdb) r                                   | 执行被调试的程序，其会自动在第一个断点处暂停执行。           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| (gdb) continue   (gdb) c                                     | 当程序在某一断点处停止运行后，使用该指令可以继续执行，直至遇到下一个断点或者程序结束。 |
| (gdb) next count                                          (gdb) n count | 令程序一行代码一行代码的执行。默认count = 1                  |
| (gdb) step count     (gdb) s count                           | 与next都是单行执行，不同的是step 命令所执行的代码行中包含函数时，会进入该函数内部，并在函数第一行代码处停止执行。 |
| (gdb) until     (gdb) until location                         | 不带参数的 until 命令，可以使 GDB 调试器快速运行完当前的循环体，并运行至循环体外停止。但是在循环体内部则是一步一步执行与next相同，带参数则是直接执行到location那行 |

### 3.4 监控变量

| (gdb) watch cond  | 可以监测某个变量被修改的情况               |
| ----------------- | ------------------------------------------ |
| (gdb) rwatch cond | 当要观察的变量cond被读时，程序暂停运行     |
| (gdb) awatch cond | 当要观察的变量cond被读或被写，程序暂停运行 |

### 3.5 显示变量值

| (gdb) print num  (gdb) p num               | 其中，参数 num 用来代指要查看或者修改的目标变量或者表达式。 单次输出 |
| ------------------------------------------ | ------------------------------------------------------------ |
| (gdb) display expr  (gdb) display/fmt expr | 其中，expr 表示要查看的目标变量或表达式；参数 fmt 用于指定输出变量或表达式的格式 |

| /fmt | 功 能                                |
| ---- | ------------------------------------ |
| /x   | 以十六进制的形式打印出整数。         |
| /d   | 以有符号、十进制的形式打印出整数。   |
| /u   | 以无符号、十进制的形式打印出整数。   |
| /o   | 以八进制的形式打印出整数。           |
| /t   | 以二进制的形式打印出整数。           |
| /f   | 以浮点数的形式打印变量或表达式的值。 |
| /c   | 以字符形式打印变量或表达式的值。     |

**注意 display与fmt没有空格 以 /x 为例，应写为 (gdb)display/x expr。**

## 四、 GDB调试正在运行的程序

book.c

```c
#include<string.h>
#include<stdio.h>

int bb()
{
        int ii=0;
        for (ii=0;ii<100000;ii++)
        {
                sleep(1);
                printf("ii=%d\n",ii);
        }
}


int main(){
        bb();
}

```

```shell
[root@iZ2zef1k78ot3rwvp4y0dhZ gdb]# gcc -g book.c -o book
[root@iZ2zef1k78ot3rwvp4y0dhZ gdb]# ./book 
ii=0
ii=1
ii=2
[root@iZ2zef1k78ot3rwvp4y0dhZ ~]# ps -aux|grep book
root     18726 18621  0 14:51 pts/0    00:00:00 ./book
root     18728 18679  0 14:51 pts/1    00:00:00 grep --color=auto book
[root@gdb ~]# gdb book -p 18726
(gdb) bt   #查看函数栈
#0  0x00007fbaf40c7f90 in __nanosleep_nocancel () from /lib64/libc.so.6
#1  0x00007fbaf40c7e44 in sleep () from /lib64/libc.so.6
#2  0x0000000000400584 in bb () at book.c:9
#3  0x00000000004005b5 in main () at book.c:16

(gdb) n #执行
Single stepping until exit from function __nanosleep_nocancel,
which has no line number information.
0x00007fbaf40c7e44 in sleep () from /lib64/libc.so.6
(gdb) bt
#0  0x00007fbaf40c7e44 in sleep () from /lib64/libc.so.6
#1  0x0000000000400584 in bb () at book.c:9
#2  0x00000000004005b5 in main () at book.c:16
(gdb) n
7		for (ii=0;ii<100000;ii++)
(gdb) b  #在line 7 打断点
Breakpoint 1 at 0x400598: file book.c, line 7.
(gdb) n
9			sleep(1);
(gdb) n
10			printf("ii=%d\n",ii);
(gdb) n

Breakpoint 1, bb () at book.c:7
7		for (ii=0;ii<100000;ii++)
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400598 in bb at book.c:7
	breakpoint already hit 1 time
(gdb) c  #利用continue执行 每到断点停一次
Continuing.

Breakpoint 1, bb () at book.c:7
7		for (ii=0;ii<100000;ii++)
(gdb) c
Continuing.

Breakpoint 1, bb () at book.c:7
7		for (ii=0;ii<100000;ii++)
(gdb) display ii  #展示ii的值
1: ii = 20
(gdb) c
Continuing.

Breakpoint 1, bb () at book.c:7
7		for (ii=0;ii<100000;ii++)
1: ii = 21
(gdb) c
Continuing.

Breakpoint 1, bb () at book.c:7
```



## 五、GDB调试多进程程序

book1.c

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

int main(){
        printf("begin\n");
        if(fork() !=0){
                printf("我是父进程：pid=%d,ppid=%d\n",getpid(),getppid());
                int ii;
                for(ii=0;ii<10;ii++){
                        printf("ii=%d\n",ii);
                        sleep(1);
                }
                exit(0);
}
else{
        printf("我是子进程：pid=%d,ppid=%d\n",getpid(),getppid());

        int jj;
        for(jj=0;jj<10;jj++){
                printf("jj=%d\n",jj);
                sleep(1);
        }
        exit(0);
}
}
```

```sehll
[root@iZ2zef1k78ot3rwvp4y0dhZ gdb]# gcc -g book1.c -o book1
```



- 调试多进程基本操作
  - 调试父进程：set follow-fork-mode parent    b 8
  - 调试子进程：set follow-fork-mode child   b 20
  - 设置调试模式：set detach-on-fork [on|off]  默认为on，表示调试当前进程时，其它进程继续运行，如果off则调试当前进程时，其它进程被挂起
  - 查看调试的进程：info inferiors  切换当前你调试的进程：inferior 进程id  需要设置调试模式off   b 8 b 20  

## 六、GDB调试多线程程序

book3.c

```c
#include<stdio.h>
#include<unistd.h>
#include<pthread.h>

int x=0,y=0;

pthread_t pthid1,pthid2;

void *pth1_main(void *arg);

void *pth2_main(void *arg);

int main(){
        if(pthread_create(&pthid1,NULL,pth1_main,(void*)0) !=0){
                printf("pthread_create pthid1 failed.\n");
                return -1;
        }

        if(pthread_create(&pthid2,NULL,pth2_main,(void*)0) !=0){
                printf("pthread_create pthid2 failed.\n");
                return -1;
        }

        printf("111\n");
        pthread_join(pthid1,NULL);


        printf("222\n");
        pthread_join(pthid2,NULL);

        printf("333\n");
        return 0;
}

void *pth1_main(void *arg){
        for(x=0;x<100;x++){
                printf("x=%d\n",x);
                sleep(1);
        }
        pthread_exit(NULL);
}

void *pth2_main(void *arg){
        for(y=0;y<100;y++){
                printf("y=%d\n",y);
                sleep(1);
        }
        pthread_exit(NULL);
}

```

```shell
[root@iZ2zef1k78ot3rwvp4y0dhZ gdb]# gcc -g book3.c -o book3 -lpthread

[root@iZ2zef1k78ot3rwvp4y0dhZ gdb]# ps aux | grep book  #只能看到一个进程
root     19364  0.0  0.0  22896   384 pts/0    Sl+  20:22   0:00 ./book3

[root@iZ2zef1k78ot3rwvp4y0dhZ gdb]# ps -aL | grep book #可以看到线程
19399 19399 pts/0    00:00:00 book3
19399 19400 pts/0    00:00:00 book3
19399 19401 pts/0    00:00:00 book3
[root@iZ2zef1k78ot3rwvp4y0dhZ gdb]# pstree -p 19399  #查看主线程与子线程关系树
book3(19399)─┬─{book3}(19400)
             └─{book3}(19401)

gdb b14 b 35 b 43
(gdb) info threads
  Id   Target Id         Frame 
* 3    Thread 0x7ffff6ff0700 (LWP 19416) "book3" pth2_main (arg=0x0) at book3.c:43
  2    Thread 0x7ffff77f1700 (LWP 19415) "book3" 0x00007ffff78b6fad in nanosleep () from /lib64/libc.so.6
  1    Thread 0x7ffff7ff1740 (LWP 19411) "book3" 0x00007ffff7bc7f47 in pthread_join () from /lib64/libpthread.so.0




gdb b14  然后执行3次n，查看线程id，然后切换，执行n，跑得快跑得慢，快是有时间就跑
```

- 调试多线程的基本操作
  - 查看线程：info threads
  - 切换线程：thread 线程id   
  - 只|全部运行当前线程：set scheduler-locking on|off
  - 指定某线程执行某gdb命令： thread apply 线程id cmd  有空就执行这个线程
  - 全部线程执行某gdb命令： thread apply all cmd

