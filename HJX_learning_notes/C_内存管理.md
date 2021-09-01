# C 内存管理

本章将讲解 C 中的动态内存管理。C 语言为内存的分配和管理提供了几个函数。这些函数可以在 **<stdlib.h>** 头文件中找到。

| 序号 | 函数和描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | **void \*calloc(int num, int size);** 在内存中动态地分配 num 个长度为 size 的连续空间，并将每一个字节都初始化为 0。所以它的结果是分配了 num*size 个字节长度的内存空间，并且每个字节的值都是0。 |
| 2    | **void free(void \*address);** 该函数释放 address 所指向的内存块,释放的是动态分配的内存空间。 |
| 3    | **void \*malloc(int num);** 在堆区分配一块指定大小的内存空间，用来存放数据。这块内存空间在函数执行完成后不会被初始化，它们的值是未知的。 |
| 4    | **void \*realloc(void \*address, int newsize);** 该函数重新分配内存，把内存扩展到 **newsize**。 |

**注意：**void * 类型表示未确定类型的指针。C、C++ 规定 void * 类型可以通过类型转换强制转换为任何其它类型的指针。



## 动态分配内存

编程时，如果预先知道数组的大小，那么定义数组时就比较容易。例如，一个存储人名的数组，它最多容纳 100 个字符，所以可以定义数组，如下所示：

```c
char name[100];
```

但是，如果预先不知道需要存储的文本长度，例如想存储有关一个主题的详细描述。在这里，我们需要定义一个指针，该指针指向未定义所需内存大小的字符，后续再根据需求来分配内存，如下所示：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
 
int main()
{
   char name[100];
   char *description;
 
   strcpy(name, "Potato Hu");
 
   /* 动态分配内存 */
   description = (char *)malloc( 200 * sizeof(char) );
   if( description == NULL )
   {
      fprintf(stderr, "Error - unable to allocate required memory\n");
   }
   else
   {
      strcpy( description, "Potato Hu a DPS student in class 10th");
   }
   printf("Name = %s\n", name );
   printf("Description: %s\n", description );
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
Name = Potato Hu
Description: Potato Hu a DPS student in class 10th
```

例如要建立一个**double 类型**的动态数组，先要说明一个指向 double 对象的指针： double * pd; 然后由下面语句建立所需大小（例如 100 ）的内存空间，并让指针指向这个空间：

```c
double *pd;
pd = (double *)malloc( 100*sizeof(double) );
```

上面的程序也可以使用 **calloc()** 来编写，只需要把 malloc 替换为 calloc 即可，如下所示：

```
calloc(200, sizeof(char));
```

当动态分配内存时，用户有完全控制权，可以传递任何大小的值。而那些预先定义了大小的数组，一旦定义则无法改变大小。

| 函数名         | 成功返回               | 失败返回 | errno  |
| -------------- | ---------------------- | -------- | ------ |
| malloc         | 指向被分配内存的指针   | NULL     | ENOMEM |
| aligned一alloc | 指向被分配内存的指针   | NULL     | ENOMEM |
| calloc         | 指向被分配内存的指针   | NULL     | ENOMEM |
| realloc        | 指向重新分配内存的指针 | NULL     | ENOMEM |

在调用上表中的这些内存分配函数时，必须进行返回值检查，以便能够及时得到内存分配是否成功与失败（如果分配失败则返回 NULL 指针），这样也可以避免因为内存分配错误而导致的不可预知和意外程序行为发生，如下面的示例代码所示：

```c
char *p = (char *)malloc( 100 * sizeof(char) );
if (p == NULL)
{
    /*处理内存分配错误，并返回错误状态*/
    return -1;
}
```

除通过使用“if（p == NULL）”或者“if（p != NULL）”语句进行简单防错处理之外，**如果指针 p 是函数的参数，那么还可以在函数的入口处用 assert (p !=NULL) 进行检查，**从而避免发生内存分配未成功却使用了它的情况。



## 重新调整内存的大小和释放内存

当程序退出时，操作系统会自动释放所有分配给程序的内存，但是，建议用户在不需要内存时，都应该调用函数 **free()** 来释放内存。

或者，您可以通过调用函数 **realloc()** 来增加或减少已分配的内存块的大小。让我们使用 realloc() 和 free() 函数，再次查看上面的实例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
 
int main()
{
   char name[100];
   char *description;
 
   strcpy(name, "Potato Hu");
 
   /* 动态分配内存 */
   description = (char *)malloc( 30 * sizeof(char) );
   if( description == NULL )
   {
      fprintf(stderr, "Error - unable to allocate required memory\n");
   }
   else
   {
      strcpy( description, "Potato Hu a DPS student.");
   }
   /* 假设您想要存储更大的描述信息 */
   description = (char *) realloc( description, 100 * sizeof(char) );
   if( description == NULL )
   {
      fprintf(stderr, "Error - unable to allocate required memory\n");
   }
   else
   {
      strcat( description, "he is in class 10th");
   }
   
   printf("Name = %s\n", name );
   printf("Description: %s\n", description );
 
   /* 使用 free() 函数释放内存 */
   free(description);
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```
Name = Potato Hu
Description: Potato Hu a DPS student.She is in class 10th
```

如果不重新分配额外的内存，strcat() 函数会生成一个错误，因为存储 description 时可用的内存不足。

在 C 语言中，如果内存的分配和释放在不同的模块或抽象层内，不仅会加大程序员追踪内存块生命周期的负担，而且可能会导致内存泄漏、内存双重释放（double-free）、非法访问已经释放的内存、写入已释放或未分配的内存区域等问题。

看下面一段示例代码：

```c
#define MIN_MEM_SIZE 10
int CompareMemorySize(char *p, size_t size)
{
    if (size < MIN_MEM_SIZE)
    {
        free(p);
        p = NULL;
        return -1;
    }
    return 0;
}
void AllocMemory(size_t size)
{
    char *p = (char *)malloc(size);
    if (p == NULL)
    {
        /*...*/
    }
    if (CompareMemorySize(p, size) == -1)
    {
        free(p);
        p = NULL;
        return;
    }
    /*...*/
    free(p);
    p = NULL;
}
```

在上面的示例代码中，p 的内存是在 AllocMemory 函数中进行分配的，然后再将它通过语句“CompareMemorySize(p,size)”传给 CompareMemorySize 函数。在 CompareMemorySize 函数中，首先通过语句“if(size<MIN_MEM_SIZE)”检查 p 所分配的内存长度，如果内存长度小于最小值(MIN_MEM_SIZE)，则释放 p。

然后，再将 CompareMemorySize 函数的返回值“-1”返回给调用者 AllocMemory 函数。在 AllocMemory 函数中执行语句“if(CompareMemorySize(p,size)==-1)”条件成立，再次释放 p。

很显然，这样不仅违背了“内存资源的分配与释放应该限定在同一模块或者同一抽象层内进行”的原则，同时导致了内存的双重释放。因此，需要对代码做如下修改：

```c
#define MIN_MEM_SIZE 10
int CompareMemorySize(size_t size)
{
    if (size < MIN_MEM_SIZE)
    {
        return -1;
    }
    return 0;
}
void AllocMemory(size_t size)
{
    char *p = (char *)malloc(size);
    if (p == NULL)
    {    
        /*...*/
    }
    if (CompareMemorySize(size) == -1)
    {
        free(p);
        p = NULL;
        return;
    }
    /*...*/
    free(p);
    p = NULL;
}
```

现在，函数 CompareMemorySize 与 AllocMemory 的职责很清楚了。其中，Compare-MemorySize 函数只负责检查内存分配的长度，而内存的分配与释放都放在 AllocMemory 函数内进行。这样不仅不会导致内存的双重释放，而且完全遵从“内存资源的分配与释放应该限定在同一模块或者同一抽象层内进行”原则



## 方便的内存分配方法

为了能够简单调用，也可以将 malloc 函数使用 define 定义成如下形式：

```c
#define MALLOC(type) ((type *)malloc(sizeof(type)))
/*或者*/
#define MALLOC(number,type) ((type *)malloc((number) * sizeof(type)))
```

现在，调用就简单多了，如下面的代码所示：

```c
char *p = MALLOC(char);
/*或者*/
char *p = MALLOC(10, char);
```



## 确保指针指向一块合法的内存

在 C 语言中，只要是指针变量，那么在使用它之前必须确保该指针变量的值是一个有效的值，它能够指向一块合法的内存，并从根本上避免未分配内存或者内存分配不足的情况发生。

```c
struct phonelist
{
    int number;
    char *name;
    char *tel;
}list, *plist;

int main(void)
{
    list.number = 1;
    strcpy(list.name, "Abby");
    strcpy(list.tel, "13511111111");
    /*...*/
    return 0;
}
```

对于上面的代码片段，在定义结构体变量 list 时，并**未给结构体 phonelist 内部的指针变量成员 “char* name” 与 “char* tel” 分配内存**。这时候的指针变量成员 “char* name” 与 “char* tel” 并**没有指向一个合法的地址**，从而导致其内部存储的将是一些未知的乱码。

因此，在调用 strcpy 函数时，如 strcpy(list.name,"Abby") 语句会将字符串"Abby"向未知的乱码所指的内存上拷贝，而这块内存 name 指针根本就无权访问，从而导致程序出错。

解决的办法 : **为指针变量成员分配内存，使其指向一个合法的地址**

```c
list.name = (char*)malloc(20*sizeof(char));
strcpy(list.name, "Abby");

list.tel = (char*)malloc(20*sizeof(char));
strcpy(list.tel, "13511111111");
```

除此之外, 还有一种常见错误

```c
struct phonelist
{
    int number;
    char *name;
    char *tel;
}list, *plist;

int main(void)
{
    plist = (struct phonelist*)malloc(sizeof(struct phonelist));
    if (plist ！= NULL)
    {
        plist->number = 1;
        strcpy(plist->name, "Abby");
        strcpy(plist->tel, "13511111111");
            /*...*/
    }
    /*...*/
    free(plist);
    plist = NULL;
    return 0;
}
```

上面的代码片段虽然为结构体指针变量 plist 分配了内存，但是仍旧没有给结构体指针变量成员“char* name”与“char* tel”分配内存，从而导致结构体指针变量成员 “char* name” 与 “char* tel” 并没有指向一个合法的地址。因此，应该做如下修改：

```c
plist->name = (char*)malloc( 20*sizeof(char) );
strcpy(plist->name, "Abby");

plist->tel = (char*)malloc( 20*sizeof(char) );
strcpy(plist->tel, "13511111111");
```



## 确保为对象分配足够的内存空间

对于上面的结构体指针变量 plist 的内存分配语句

```c
plist = (struct phonelist*)malloc(sizeof(struct phonelist));
```



## 避免内存分配成功，但并未初始化

其实，内存的默认初值究竟是什么并没有统一的标准。如 malloc 函数分配得到的内存空间就是未初始化的，而它所分配的内存空间里可能包含出乎意料的值。因此，一般在使用该内存空间时，就需要调用函数 memset 来将其初始化为全 0。如下面的示例代码所示：

```c
int * p = NULL;
p = (int*)malloc(sizeof(int));
if (p == NULL)
{
    /*...*/
}
/*初始化为0*/
memset(p, 0, sizeof(int));
```

