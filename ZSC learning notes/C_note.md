# 1 C存储类

## 1.1 static存储类

**static** 存储类指示编译器在程序的生命周期内保持局部变量的存在，而不需要在每次它进入和离开作用域时进行创建和销毁。因此，使用 static 修饰局部变量可以在函数调用之间保持局部变量的值。（说白了就是只初始化一次，每次调用接着上次的值继续执行，不会被重置）

## 1.2 external存储类

**extern** 存储类用于提供一个全局变量的引用，全局变量对所有的程序文件都是可见的。*extern* 是用来在另一个文件中声明一个全局变量或函数。（extern就是为了声明一个所有文件都可以识别的变量，可以直接调用）例如

main.c

```c
#include <stdio.h>
 
int count ;
extern void write_extern();
 
int main()
{
   count = 5;
   write_extern();
}
```

support.c

```c
#include <stdio.h>
 
extern int count;
 
void write_extern(void)
{
   printf("count is %d\n", count);
}
```

在这里，第二个文件中的 *extern* 关键字用于声明已经在第一个文件 main.c 中定义的 *count*。编译后输出结果为 count is 5 

## 1.3 深拷贝与浅拷贝

- 浅拷贝：实际上就是将第二个变量的地址指向第一个变量，数据与指向的空间不会拷贝。下面是浅拷贝代码

  ```c
  # include<assert.h>
  # include<string.h>
  #include <stdlib.h>
  typedef struct Node//定义了一个结构体
  {
  	int size;
  	char *data;
  }S_Node;
  int main()
  {
  	S_Node node1;
  	node1.data = (char *)malloc(sizeof(char)*100);//指针所指向的地址
  	assert(node1.data != NULL);
  	strcpy(node1.data, "hello world");
  	node1.size = strlen(node1.data);
  	S_Node node2 = node1;
  	printf("%d, %s\n", node1.size, node1.data);
  	printf("%d,%s\n", node2.size, node2.data);
  	free(node1.data);
  	//free(node2.data);error重复释放
  	return 0;
   } 
  ```

  ![1](./c_note\1.png)

- 深拷贝：拷贝了所有数据与地址空间

  ```c
  # include<stdio.h>
  # include<assert.h>
  # include<string.h>
  #include <stdlib.h>
  typedef struct Node//结构体
  {
  	int size;
  	char *data;
  }S_Node;
  void CopyNode(S_Node *node3, S_Node node1)//CopyNode 函数实现结构体变量的深拷贝
  {
  	node3->size = node1.size;
  	node3->data = (char *)malloc(node3->size + 1);//申请空间
  	assert(node3->data != NULL);
  	strcpy(node3->data, node1.data);
  }
  int main()
  {
  	S_Node node1;
  	node1.data = (char *)malloc(sizeof(char)*100);
  	assert(node1.data != NULL);
  	S_Node node3;
  	CopyNode(&node3, node1);
  	printf("%d, %x\n", node1.size, node1.data);
  	printf("%d,%x\n", node3.size, node3.data);
  	free(node1.data);
  	free(node3.data);//释放申请的空间
  	return 0;
   } 
  ```

  ![2](./c_note\2.png)

# 2 位运算符

| p    | q    | p&q(与) | p\|q(或) | p^q(异或同0异1) |
| ---- | ---- | ------- | -------- | --------------- |
| 0    | 0    | 0       | 0        | 0               |
| 0    | 1    | 0       | 1        | 1               |
| 1    | 1    | 1       | 1        | 0               |
| 1    | 0    | 0       | 1        | 1               |

假设如果 A = 60，且 B = 13，现在以二进制格式表示，它们如下所示：

A = 0011 1100

B = 0000 1101

\-----------------

A&B = 0000 1100

A|B = 0011 1101

A^B = 0011 0001

~A = 1100 0011

# 3 enum(枚举)

## 3.1 定义方式

**1、先定义枚举类型，再定义枚举变量**

```c
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
enum DAY day;
```

**2、定义枚举类型的同时定义枚举变量**

```c
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;
```

**3、省略枚举名称，直接定义枚举变量**

```c
enum
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;
```

注意 第一个枚举变量的值为用户定义的1，后续成员自动在前面的值+1，如果用户未定义第一个的值，则默认为0，后续自动+1

## 3.2 枚举实例

```c
#include <stdio.h>
 
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
} day;
int main()
{
    // 遍历枚举元素
    for (day = MON; day <= SUN; day++) {
        printf("枚举元素：%d \n", day);
    }
}
```

输出结果为

```
枚举元素：1 
枚举元素：2 
枚举元素：3 
枚举元素：4 
枚举元素：5 
枚举元素：6 
枚举元素：7
```

# 4 指针

## 4.1 一级指针

```c
#include <stdio.h>
 
int main ()
{
   int  var = 20;   /* 实际变量的声明 */
   int  *ip;        /* 指针变量的声明 */
 
   ip = &var;  /* 在指针变量中存储 var 的地址 */
 
   printf("var 变量的地址: %p\n", &var  );
 
   /* 在指针变量中存储的地址 */
   printf("ip 变量存储的地址: %p\n", ip );
 
   /* 使用指针访问值 */
   printf("*ip 变量的值: %d\n", *ip );
 
   return 0;
}
```

```
var 变量的地址: 0x7ffeeef168d8
ip 变量存储的地址: 0x7ffeeef168d8
*ip 变量的值: 20
```

如上面代码，用*ip声明一个指针变量，然后将var的地址赋给指针，因为指针只能指向地址。

**赋值可以用**

**ip = &var**

**或定义时 int *ip=&var; **

**也可以赋给ip一个动态地址后，用\*ip直接赋值**

```c
int *ip=(int *)malloc(N*sizeof(int));
*ip=2;
```

## 4.2 指针数组

```c
double balance[50];
double *p;
p = balance;
取值的话可以 *(p+0),*(p+1)//因为p指向balance[0]
或者直接 double *p[]={1.0,2.0}
取值的话是*p[0],*p[1]
或者 int arr[] = { 99, 15, 100, 888, 252 };
    int *p = &arr[2];  //也可以写作 int *p = arr + 2;现在p指向的是arr[2]
     printf("%d, %d, %d, %d, %d\n", *(p-2), *(p-1), *p, *(p+1), *(p+2) );//输出为99, 15, 100, 888, 252
```

*p++ 等价于 *(p++)，表示先取得第 n 个元素的值，再将 p 指向下一个元素。

*++p 等价于 *(++p)，会先进行 ++p 运算，使得 p 的值增加，指向下一个元素，整体上相当于 *(p+1)，所以会获得第 n+1 个数组元素的值。

(*p)++ 就非常简单了，会先取得第 n 个元素的值，再对该元素的值加 1。假设 p 指向第 0  个元素，并且第 0 个元素的值为 99，执行完该语句后，第 0  个元素的值就会变为 100。

## 4.3 字符串指针

```c
#include <stdio.h>
#include <string.h>

int main(){
    char str[] = "http://www.baidu.com";
    char *pstr = str;
    int len = strlen(str), i;

    //使用*(pstr+i)
    for(i=0; i<len; i++){
        printf("%c", *(pstr+i));
    }
    printf("\n");
    //使用pstr[i]
    for(i=0; i<len; i++){
        printf("%c", pstr[i]);
    }
    printf("\n");
    //使用*(str+i)
    for(i=0; i<len; i++){
        printf("%c", *(str+i));
    }
    printf("\n");

    return 0;
}
```

字符数组归根结底还是一个数组，所以用法和指针数字类似。

## 4.4 多级指针

```c
#include <stdio.h>
int main(){
    int a =100;
    int *p1 = &a;
    int **p2 = &p1;
    int ***p3 = &p2;
}
```

想要获取指针指向的数据时，一级指针加一个`*`，二级指针加两个`*`，三级指针加三个`*`，以此类推，

以三级指针 p3 为例来分析上面的代码。\*\*\*p3`等价于`\*(\*(\*p3))。\*p3 得到的是 p2 的值，也即 p1 的地址；\*(\*p3) 得到的是 p1 的值，也即 a 的地址；经过三次“取值”操作后，\*(\*(*p3)) 得到的才是 a 的值。

# 5 结构体与共用体

## 5.1 结构体作为函数参数

```c
#include <stdio.h>
#include <string.h>
 
struct Books
{
   char  title[50];
   char  author[50];
   char  subject[100];
   int   book_id;
};
 
/* 函数声明 */
void printBook( struct Books book );
int main( )
{
   struct Books Book1;        /* 声明 Book1，类型为 Books */
   struct Books Book2;        /* 声明 Book2，类型为 Books */
 
   /* Book1 详述 */
   strcpy( Book1.title, "C Programming");
   strcpy( Book1.author, "Nuha Ali"); 
   strcpy( Book1.subject, "C Programming Tutorial");
   Book1.book_id = 6495407;
 
   /* Book2 详述 */
   strcpy( Book2.title, "Telecom Billing");
   strcpy( Book2.author, "Zara Ali");
   strcpy( Book2.subject, "Telecom Billing Tutorial");
   Book2.book_id = 6495700;
 
   /* 输出 Book1 信息 */
   printBook( Book1 );
 
   /* 输出 Book2 信息 */
   printBook( Book2 );
 
   return 0;
}
void printBook( struct Books book )
{
   printf( "Book title : %s\n", book.title);
   printf( "Book author : %s\n", book.author);
   printf( "Book subject : %s\n", book.subject);
   printf( "Book book_id : %d\n", book.book_id);
}
```

## 5.2 指向结构体的指针

和指针同样用法，声明的话可以

```c
struct Books *struct_pointer;
```

然后指针存储地址，这里存的是结构体的地址

```c
struct_pointer = &Book1;//指针名=结构体的地址
```

这里访问变量与结构体访问不同，用的是->

```c
struct_pointer->title;
```

下面重写上面的例子

```c
#include <stdio.h>
#include <string.h>
 
struct Books
{
   char  title[50];
   char  author[50];
   char  subject[100];
   int   book_id;
};
 
/* 函数声明 */
void printBook( struct Books *book );
int main( )
{
   struct Books Book1;        /* 声明 Book1，类型为 Books */
   struct Books Book2;        /* 声明 Book2，类型为 Books */
 
   /* Book1 详述 */
   strcpy( Book1.title, "C Programming");
   strcpy( Book1.author, "Nuha Ali"); 
   strcpy( Book1.subject, "C Programming Tutorial");
   Book1.book_id = 6495407;
 
   /* Book2 详述 */
   strcpy( Book2.title, "Telecom Billing");
   strcpy( Book2.author, "Zara Ali");
   strcpy( Book2.subject, "Telecom Billing Tutorial");
   Book2.book_id = 6495700;
 
   /* 通过传 Book1 的地址来输出 Book1 信息 */
   printBook( &Book1 );
 
   /* 通过传 Book2 的地址来输出 Book2 信息 */
   printBook( &Book2 );
 
   return 0;
}
void printBook( struct Books *book )
{
   printf( "Book title : %s\n", book->title);
   printf( "Book author : %s\n", book->author);
   printf( "Book subject : %s\n", book->subject);
   printf( "Book book_id : %d\n", book->book_id);
}
```

## 5.3 上述两种区别如下

1 传的参数不同，5.1是结构作为函数参数 ，5.2是结构的指针作为参数

2 5.2在函数调用时，参数需要的是地址（指针的用法） printBook( &Book1 );

3 访问结构成员时，5.1是book.title 5.2则为book->title

## 5.4 共用体

共用体和结构体用法类似，只是为了共用存储空间，其中共用体在内存占用的大小则是这个共用体中的各个变量的最大值，例如下面这个共用体的内存则为char类型的20个字节。

```c
union Data
{
   int i;
   float f;
   char  str[20];
};
```

其中共用体的主要的目的就是为了节省空间，所以这就导致了无法同时输出每个成员的值，因为有损坏，例如

```c
#include <stdio.h>
#include <string.h>
 
union Data
{
   int i;
   float f;
   char  str[20];
};
 
int main( )
{
   union Data data;        
 
   data.i = 10;
   data.f = 220.5;
   strcpy( data.str, "C Programming");
 
   printf( "data.i : %d\n", data.i); //data.i : 1917853763
   printf( "data.f : %f\n", data.f);//data.f : 4122360580327794860452759994368.000000
   printf( "data.str : %s\n", data.str);//data.str : C Programming
 
   return 0;
}
```

但是同一时间只使用一个变量的话就可以正常输出

```c
 data.i = 10;
   printf( "data.i : %d\n", data.i);//data.i : 10
   
   data.f = 220.5;
   printf( "data.f : %f\n", data.f);//data.f : 220.500000
   
   strcpy( data.str, "C Programming");
   printf( "data.str : %s\n", data.str);//data.str : C Programming
```

## 5.5 typedef vs #define

**#define** 是 C 指令，用于为各种数据类型定义别名，与 **typedef** 类似，但是它们有以下几点不同：

- **typedef** 仅限于为类型定义符号名称，**#define** 不仅可以为类型定义别名，也能为数值定义别名，比如定义 1 为 ONE。
- **typedef** 是由编译器执行解释的，**#define** 语句是由预编译器进行处理的。



# 6 位域

## 6.1 概念

有些信息在存储时，并不需要占用一个完整的字节，而只需占几个或一个二进制位。例如在存放一个开关量时，只有 0 和 1 两种状态，用 1 位二进位即可。为了节省存储空间，并使处理简便，C 语言又提供了一种数据结构，称为"位域"或"位段"。（说白了就是自己定义一个变量后，设置二进制多少位，10进制转换为2进制时不可以超，从而进行与、或、非操作）

## 6.2 定义形式

```c
type [member_name] : width ;
```

|元素 |描述|
|---|---|
|type|只能为 int(整型)，unsigned int(无符号整型)，signed int(有符号整型) 三种类型，决定了如何解释位域的值。|
|member_name|位域的名称。|
|width|位域中位的数量。宽度必须小于或等于指定类型的位宽度。|

## 6.3 主要用法

```c
#include <stdio.h>
#include <string.h>
 
struct
{
  unsigned int age : 3;
} Age;
 
int main( )
{
   Age.age = 4;
   printf( "Sizeof( Age ) : %d\n", sizeof(Age) );//Sizeof( Age ) : 4
   printf( "Age.age : %d\n", Age.age );//Age.age : 4
 
   Age.age = 7;
   printf( "Age.age : %d\n", Age.age );//Age.age : 7
 
   Age.age = 8; // 二进制表示为 1000 有四位，超出
   printf( "Age.age : %d\n", Age.age );//Age.age : 0
 
   return 0;
}
```

```c
int main(){
    struct bs{
        unsigned a:1;
        unsigned b:3;
        unsigned c:4;
    } bit,*pbit;
    bit.a=1;    /* 给位域赋值（应注意赋值不能超过该位域的允许范围） */
    bit.b=7;    /* 给位域赋值（应注意赋值不能超过该位域的允许范围） */
    bit.c=15;    /* 给位域赋值（应注意赋值不能超过该位域的允许范围） */
    printf("%d,%d,%d\n",bit.a,bit.b,bit.c);    /* 以整型量格式输出三个域的内容 */
    pbit=&bit;    /* 把位域变量 bit 的地址送给指针变量 pbit */
    pbit->a=0;    /* 用指针方式给位域 a 重新赋值，赋为 0 */
    pbit->b&=3;    /* 使用了复合的位运算符 "&="，相当于：pbit->b=pbit->b&3，位域 b 中原有值为 7，与 3 作按位与运算的结果为 3（111&011=011，十进制值为 3） */
    pbit->c|=1;    /* 使用了复合位运算符"|="，相当于：pbit->c=pbit->c|1，其结果为 15 */
    printf("%d,%d,%d\n",pbit->a,pbit->b,pbit->c);    /* 用指针方式输出了这三个域的值 */
}
```

## 6.4 注意事项

```c
struct bs{
    unsigned a:4;
    unsigned  :4;    /* 空域 */
    unsigned b:4;    /* 从下一单元开始存放 */
    unsigned c:4
}
```

其中在这个位域定义中，a 占第一字节的 4 位，后 4 位填 0 表示不使用，b 从第二字节开始，占用 4 位，c 占用 4 位。所以位域在本质上就是一种结构类型，不过其成员是按二进位分配的。



# 7函数

## 7.1 函数指针和指针函数

### 7.1.1 函数指针

**函数指针**是指向函数的指针变量，

函数指针变量的声明：

```c
typedef int (*fun_ptr)(int,int); // 声明一个指向同样参数、返回值的函数指针类型
```

例如下面代码，

```c
#include <stdio.h>
 
int max(int x, int y)
{
    return x > y ? x : y;
}
 
int main(void)
{
    /* p 是函数指针 */
    int (* p)(int, int) = & max; // &可以省略
    int a, b, c, d;
 
    printf("请输入三个数字:");
    scanf("%d %d %d", & a, & b, & c);
 
    /* 与直接调用函数等价，d = max(max(a, b), c) */
    d = p(p(a, b), c); 
 
    printf("最大的数字是: %d\n", d);
 
    return 0;
}
```

### 7.1.2 指针函数

指针函数，简单的来说，就是一个返回指针的函数，其本质是一个函数，而该函数的返回值是一个指针。如下面实例，需要用一个指针的结构体接住返回值。

```c
typedef struct _Data{
    int a;
    int b;
}Data;

//指针函数
Data* f(int a,int b){
    Data * data = new Data;
    data->a = a;
    data->b = b;
    return data;
}

int main()
{
    //调用指针函数
    Data * myData = f(4,5);
    printf("f(4,5) = %d %d",myData->a,myData->b); //输出为f(4,5) = 4 5
}
```

### 7.1.3 两者区别

- 定义不同

  指针函数本质是一个函数，其返回值为指针。
  函数指针本质是一个指针，其指向一个函数。

- 写法不同

  指针函数：int* fun(int x,int y);
  函数指针：int (\*fun)(int x,int y);
  可以简单粗暴的理解为，指针函数的*是属于数据类型的，而函数指针的星号是属于函数名的。
  再简单一点，可以这样辨别两者：函数名带括号的就是函数指针，否则就是指针函数。

## 7.2 回调函数

函数指针作为某个函数的参数

函数指针变量可以作为某个函数的参数来使用的，回调函数就是一个通过函数指针调用的函数。

简单讲：回调函数是由别人的函数执行时调用你实现的函数。

实例

实例中 populate_array 函数定义了三个参数，其中第三个参数是函数的指针，通过该函数来设置数组的值。

实例中我们定义了回调函数 getNextRandomValue，它返回一个随机值，它作为一个函数指针传递给 populate_array 函数。

populate_array 将调用 10 次回调函数，并将回调函数的返回值赋值给数组。

```c
#include <stdlib.h>  
#include <stdio.h>
 
// 回调函数
void populate_array(int *array, size_t arraySize, int (*getNextValue)(void))
{
    for (size_t i=0; i<arraySize; i++)
        array[i] = getNextValue();
}
 
// 获取随机值
int getNextRandomValue(void)
{
    return rand();
}
 
int main(void)
{
    int myarray[10];
    /* getNextRandomValue 不能加括号，否则无法编译，因为加上括号之后相当于传入此参数时传入了 int , 而不是函数指针*/
    populate_array(myarray, 10, getNextRandomValue);
    for(int i = 0; i < 10; i++) {
        printf("%d ", myarray[i]);
    }
    printf("\n");
    return 0;
}
```

## 7.3 函数参数传递方式

### 7.3.1值传递

```c
void Exchg1(int x, int y)
{
int tmp;
tmp = x;
x = y;
y = tmp;
printf("x = %d, y = %d\n", x, y);
}
main()
{
int a = 4,b = 6;
Exchg1(a, b);
printf("a = %d, b = %d\n", a, b);
return(0);
}

/*
输出结果为
x = 6, y = 4.
a = 4, b = 6.
*/
```

这里的exchg1()函数调用的是将主函数中a与b的分别赋给了exchg1（）中的x与y，只是在函数内部做交换，只是x与y的交换，并不是a与b的交换，所以a与b不变。

### 7.3.2 地址传递

```c
void Exchg2(int *px, int *py)
{
int tmp = *px;
*px = *py;
*py = tmp;
printf("*px = %d, *py = %d.\n", *px, *py);
}
main()
{
int a = 4;
int b = 6;
Exchg2(&a, &b);
printf("a = %d, b = %d.\n", a, b);
return(0);
}
/*
结果为
*px = 6, *py = 4.
a = 6, b = 4.
*/
```



这里exchg2（）传的参数为a和b的地址，exchg2（）函数里修改的是px，py的值，他们的地址是不变的，所以最终结果不会改变

### 7.3.3 引用传递

```c
void Exchg3(int &x, int &y) /* 注意定义处的形式参数的格式与值传递不同 */
{
int tmp = x;
27
x = y;
y = tmp;
printf("x = %d, y = %d.\n", x, y);
}
main()
{
int a = 4;
int b = 6;
Exchg3(a, b); /*注意：这里调用方式与值传递一样*/
printf("a = %d, b = %d.\n”, a, b);
}
       /*
       x = 6, y = 4.
	   a = 6, b = 4
       */
```

这里在exchg3（）函数中，参数接收的是引用，所以修改的是实际参数，即a与b的值，因为引用就是将变量重新定义一个别名而已。

## 7.4 形式参数与实际参数的区别

- **形参:**在函数定义中出现的参数可以看做是一个占位符，它没有数据，只能等到函数被调用时接收传递进来的数据，所以称为**形式参数**，简称**形参**。
- **实参:**函数被调用时给出的参数包含了实实在在的数据，会被函数内部的代码使用，所以称为**实际参数**，简称**实参**。

1) 形参变量只有在函数被调用时才会分配内存，调用结束后，立刻释放内存，所以形参变量只有在函数内部有效，不能在函数外部使用。

2) 实参可以是常量、变量、表达式、函数等，无论实参是何种类型的数据，在进行函数调用时，它们都必须有确定的值，以便把这些值传送给形参，所以应该提前用赋值、输入等办法使实参获得确定值。

3) 实参和形参在数量上、类型上、顺序上必须严格一致，否则会发生“类型不匹配”的错误。当然，如果能够进行自动类型转换，或者进行了强制类型转换，那么实参类型也可以不同于形参类型。

4) 函数调用中发生的数据传递是单向的，只能把实参的值传递给形参，而不能把形参的值反向地传递给实参；换句话说，一旦完成数据的传递，实参和形参就再也没有瓜葛了，所以，在函数调用过程中，形参的值发生改变并不会影响实参。

**一句话，形参是函数内部做计算，函数内部中的局部变量，主函数中的变量不会跟着变化。实参则是主函数中的变量一起做计算。**
