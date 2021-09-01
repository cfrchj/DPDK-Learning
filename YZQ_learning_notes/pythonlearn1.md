#### 字符串

###### 1.capitalize 首字母大写

用法： newstr = string.capitalize()

###### 2.casefold/lower 字母全体小写

用法：newstr = string.casefold()

newstr = string.lower()

###### 3.upper 字母全体大写

用法：big_str = string.upper()

###### 4.swapcase 大小写字母翻转

用法：newstr = string.swapcase()

###### 5.zfill 填充0

用法：newstr = string.zfill(width) 

width：新字符串的长度

###### 6.count 返回当前字符串中某个成员（元素）的个数

用法：inttype = string.count(item)

###### 7.startswith endswith 判断字符串开始位/结尾是否是某元素，返回布尔值true/false

用法：string.startswith(item)

string.endswith(item) 

###### 8.find/ index 获取某个值的位置，返回整型

用法：string.find(item)

string.index(item)

###### 9.strip lstrip estrip 去掉字符串左右/开头/结尾的指定元素，默认为空格

用法：newstr = string.strip(item)

###### 10.replace 替换元素 可指定数量

用法：newstr = string.replace(old,new,max) max默认为全部

##### 格式化

###### 1.字符串使用操作符%来格式化（推荐使用）

格式化字符串与格式符变量用%连接，两边有空格

###### 2.format函数（适用格式较少）

string.format(data,data,data…)

###### 3.f-strings

定义一个变量——字符串前加f符号——需要格式化的位置使用{变量名}

##### 转义字符

###### 1.<img src="C:\Users\颜泽卿\AppData\Roaming\Typora\typora-user-images\image-20210814120439333.png" alt="image-20210814120439333" style="zoom:50%;" />

###### 2.转义无效字符

在字符串前加r将转义字符无效化（对%不起作用）



#### 列表

###### 1.append   

在列表中添加一个新的成员，到列表末尾

list.append(new_item)

###### 2.insert   

将新成员添加到列表指定位置

list.insert(index,new_item)

###### 3.count 

返回列表中某个成员的个数

inttype = list.count(item)

###### 4.remove 

删除列表中某成员，只删除第一个

list.remove(item)

del（内置函数） 把元素完全删除

###### 5.reverse  

对当前列表顺序反序

list.reverse()

###### 6.sort  

对当前列表按一定规律排序，元素类型需相同

list.sort(key = None,reverse = False) 

True降序 False升序

###### 7.clear  

清空列表所有成员

list.clear()

###### 8.copy 

复制一个新的列表，内存空间不同（与二次赋值不同），浅拷贝，新旧列表共享数据

list_new = list.copy(）

深拷贝 list_new02 = copy.deepcopy(list)

###### 9.extend  

将其他列表或元组中的元素一次性全部倒入当前列表中

list.extend(iterable)

###### 10.列表的索引与切片

索引：从最左边记录的位置就是索引，从0开始，最大索引是它的长度；索引生成的列表是新列表，地址不同。

切片：对一定范围的元素进行访问，用冒号把相隔的两个索引查找出来；规则“左含右不含” ；列表的反向获取加负号；步长获取 如 0：8：2（步长为2）

pop删除列表中第几个索引并返回 list.pop(index)

del list[index] 全部删除元素且无返回(元组只能删除整个元组不可删改单个元素)

###### 11.字符串的索引、切片

规则与列表相同

字符串无法通过修改与删除，字符串无法删除，只能获取

string.index(item) 获取不到报错

string.find(item)  获取不到返回-1

 

#### 字典

###### 1.updata

添加新的字典，如已有key则替换

dict.updata(new_dict)

###### 2.setdefault

获取某个key的value，若key不存在则添加key，设置value默认值

dict.setdefault(key,value)

###### 3.keys

获取当前函数中所有key值

dict.keys() 

返回key集合的伪列表，不具有列表功能，需传递给list才能当成列表使用

###### 4.values

获取所有key对应的值value

dict.values()

###### 5.指定key的value值获取方法  [] 与get 

dict.get(key,default  = None)  default可设置默认值

[]  key不存在时直接报错

get  key不存在返回默认值

###### 6.字典的删除 clear/pop/del/popitem

dict.clear()内容清空

dict.pop(key) 删除指定值key 并返回这个key对应的值

del dict['key']   或  del dict 整个删除

dict.popitem()  删除字典里末尾一组key值并返回

###### 7.copy

将当前字典复制一个新字典，内存地址不同

###### 8.判断成员存在 in/not in/get 

##### 数据类型与布尔值的关系

<img src="C:\Users\颜泽卿\AppData\Roaming\Typora\typora-user-images\image-20210814143547400.png" alt="image-20210814143547400" style="zoom:50%;" />



#### 集合set

集合是一个无序的不重复元素序列，对两个列表进行交并差的处理性

###### 1.与列表的区别

###### <img src="C:\Users\颜泽卿\AppData\Roaming\Typora\typora-user-images\image-20210814144454462.png" alt="image-20210814144454462" style="zoom:50%;" />

   

###### 2.通过set创建集合，不可直接{}创建  

new_set = set()

###### 3.add/updata/remove/clear/del

set.add(item)  item为成员

set.updata(iterable)

set.remove(item) 

set.clear()

del a_set

集合无法通过索引获取元素；无获取元素的任何方法；是用来处理列表或元组的临时类型，不适合存储传输

###### 4.difference 差集/intersection 交集/union 并集

a_set.difference(b_set)  返回存在于a_set而不存在于b_set的元素集合

a_set.intersection(b_set…) 返回同时存在与a_set与b_set…的元素集合

a_set.union(b_set…) 返回a_set与b_set所有元素集合

###### 5.isdisjoint 

判断是否有相同函数 有相同函数则返回False，没有相同元素返回True

a_set.isdisjoint(b_se)



#### 逻辑判断与逻辑语句

条件语句+业务语句

###### 1.if语句/else语句/elif语句（或者如果）

if bool_result:

   do

elif bool_result:

   elifdo

elif bool_result:

   elifdo

else：

   elsedo

有且只有一个if 可以有多个elif

###### 2.循环for语句

for item in iterable：

​    print(item)

嵌套for循环

###### 3.while循环

 在满足条件会无限循环 不满足时停止

while bool_result:

   do

###### 4.continue 和 break

 while bool:

   continue

for item in iterable:

   continue

   print(item)
