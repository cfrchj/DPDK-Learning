## suricata笔记

### 一、概述

#### 1.suricata简介

- Suricata是一个免费，开源，成熟，高性能，稳定的网络威胁检测引擎
- suricata相对于传统snort引擎，它是由很多单独的线程模块所组成，使用者可以自行配置组合所需要的线程模块，进行优化、自定义需求。
- suricata支持多种抓包引擎，如：pcap、netmap、pfring、socket等。

- 系统功能包括：
  - 实时入侵检测(IDS)：依照一定的安全策略，通过软、硬件，对网络、系统的运行状况进行监视，尽可能发现各种攻击企图、攻击行为或者攻击结果，以保证网络系统资源的机密性、完整性和可用性。
  - 内联入侵预防(IPS)：IPS技术可以深度感知并检测流经的数据流量，对恶意报文进行丢弃以阻断攻击，对滥用报文进行限流以保护网络带宽资源。
  - 网络安全监控(NSM)
  - 离线pcap处理

- Suricata依靠强大的可扩展性的规则和特征语言过滤网络流量，并支持LUA脚本语言

- 输出文件格式为YAML或JSON，方便与其他数据库或安全数据分析平台集成

#### 2.suricata总体架构

- 报文检测系统通常四大部分，报文获取、报文解码、报文检测、日志记录；

- suricata不同的功能安装模块划分，一个模块的输出是另一个模块的输入，suricata通过线程将模块串联起来。

![image-20211030140414628](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211030140414628.png)

​        **Receive**：收集网络数据包，并将其封装在Packet结构中，然后放入下一个缓冲区。
　　**Decode**：对数据包进行解码，主要是对数据包头部信息进行分析并保存在Packet结构中。
　　**StreamTCP**：对数据包进行TCP流重组。
　　**Detect**：检测数据包是否包含入侵行为。
　　**Verdict**：对检测后的数据包进行判定，并将判定结果告诉内核（通过ipq_set_verdict函数），方便内核对数据包进行接收、丢弃等处理。
　　**RespondReject**：通过libnet对那些要执行Reject操作的数据包进行相应的回应，需要向双端发送reset包

​         **logs模块：**将处理结果记录在日志中。

![img](https://pic4.zhimg.com/80/v2-cfe85f34814564a59d471cc447d16403_1440w.jpg)



### 二、suricata的安装

参考资料：

[3. Installation — Suricata 6.0.2 documentation](https://suricata.readthedocs.io/en/suricata-6.0.2/install.html#rhel-centos-8-and-7)

[Network/suricata/安装踩坑.md · 韩若明溪/my_notes - Gitee.com](https://gitee.com/han_gx/my_notes/blob/master/Network/suricata/安装踩坑.md)

[(11条消息) CentOS7 和 Ubantu 18.04 安装suricata 6.0.0_九阳道人的博客-CSDN博客](https://blog.csdn.net/qq_31507523/article/details/109877224)

环境：VMware centos7/8 

安装suricata 6.0.3

#### 1.使用yum安装

```
yum install -y gcc gcc-c++ #安装前确保有这些环境

yum install epel-release yum-plugin-copr
yum copr enable @oisf/suricata-6.0
yum install suricata

suricata -V    #验证, eg: This is Suricata version 6.0.3 RELEASE
```

#### 2.使用源码安装

```
#先安装工具包（不全）
yum install bash-completion -y
yum install -y perl
yum install -y net-tools
yum install -y gcc
yum install -y wget
yum install git
yum install -y rustc cargo

#从官网下载安装包
wget https://www.openinfosecfoundation.org/download/suricata-6.0.3.tar.gz
#解压
tar xzvf suricata-6.0.3.tar.gz
#进入文件夹
cd suricata-6.0.3

#检查依赖，这一步会出现很多ERROR，会提示缺少的依赖，缺什么就安装什么，有些会提示安装命令，有些需要从官网下载再make install 百度基本可以找到解决办法
./configure

#提示出现可以make了就开始make
make
make install-full

#不报错后，输入suricata 如果还是查询不到suricata，可以输入如下命令进行全局配置
ln -s /usr/local/bin/suricata /usr/sbin/suricata

```

报错实例：

1.报错 `configure: error: pcre.h not found ...`

​    缺少[pcre](https://baike.baidu.com/item/PCRE/7401536?fr=aladdin), 安装pcre

```
wget https://netix.dl.sourceforge.net/project/pcre/pcre/8.40/pcre-8.40.tar.gz
tar -zxvf pcre-8.40.tar.gz
cd pcre-8.40
./configure
make && make install
pcre-config --version
```

2.报错 `ERROR! libyaml library not found, go get it.` 这个会给出很详细的提示信息，从官网下载。

```
wget http://pyyaml.org/download/libyaml/yaml-0.2.5.tar.gz
./configure
make && make install
```

安装成功后输入suricata，可以查看版本号，以及提示suricata的命令行![image-20211030144548239](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211030144548239.png)

#### 3.查看文件存放位置

- 默认配置文件suricata.yaml位置为：(/usr/local)`/etc/suricata/suricata.yaml` 


![image-20211027204124526](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211027204124526.png)

- 默认规则rules位于：（/usr/local）`/var/lib/suricata/rules/suricata.rules`


![image-20211027204141264](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211027204141264.png)

- 自定义规则集存放位置： (/usr/local) `/etc/suricata/rules`（自己新建的）


![image-20211027204254520](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211027204254520.png)

- 更新后规则文件存放位置：`/usr/share/suricata/rules`![image-20211030145403818](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211030145403818.png)

- 默认日志位置 ：`/var/log/suricata`


![image-20211027143424958](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211027143424958.png)

如果想要修改位置，就新建文件夹然后把相关的文件移动过去，也可以用命令行参数更改目录，或直接在.yaml文件中更改目录。

### 三、输入输出

#### 1.输入

##### 1.1命令

```
-c  #配置文件路径，例如:suricata.yaml存放路径。

-i  #输入想用来抓包的网卡名称，以pcap live模式运行。

-r  #输入记录数据的抓包文件的路径和文件名，查看包数据。

-s  #设置说明名文件，这个文件将与yaml中设置的rules文件集一起载入使用。

-l  #设置默认的日志文件夹。如果已在yaml文件中设置了默认的日志文件夹(default-log-dir)，只有-l后的选项参数起做用。如果不使用-l选项，则使用yaml中设置的默认值。

-D  #使用suricata成为一个后台运行的进程。

--list-runmodes  #使用该选项将列出所有可能的运行模式。

--runmode  #该选项与-i或-r选项联合使用。用这个选项你可以设置suricate工作于你所选择的运行模式。该选项的值可以覆盖yaml文件中设置的运行模式。

-u  #该选项用于运行单元测试，测试suricata代码。

-U  #该选项用于选择能运行哪一个单元测试。这个先项可以使用正则表达式。例如：Suricata –u –U http。

--list-unittests  #该选项用于例出可用的单元测试。

--fatal-unittests  #使用该选项，当一个单元测试失败后能立即停止，以便于立即查看错误信息。
```

##### 1.2常用命令行：

1.`suricata -c /etc/suricata/suricata.yaml -i ens33`

![image-20211030152445349](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211030152445349.png)

2.suricata -c /etc/suricata/suricata.yaml -r test.pcap   #read the pcap file
3.suricata -c /etc/suricata/suricata.yaml -T # test the rules and configuration

##### 1.3程序配置

###### 1.3.1Suricata.yaml配置

- Suricata网络配置

  ```
  配置Suricata监控的网络地址，支持变量配置，变量名为大写。
  ```

-  配置规则开启或关闭

  ```
  设置rules、classification.config、reference.config及rhreshold.config文件路径，调整规则的应用和关闭。
  ```

- 设置输出

  ```
  输出设置有三个部分：
  输出状态：配置全局输出。
  输出日志：配置具体输出日志的开启、格式和路径等。
  输出方式：配置日志输出的具体方式,显示器、文件或系统日志。
  ```

- 基本监听设置

  ```
  基本监听接口及数据包捕获配置。
  ```

- 应用层协议配置

  ```
  配置应用层协议解析，启用部分有‘yes’,‘no’‘detection-only’三个选项，分别对应：检测并解析、不开启、只检测不解析
  ```

- 高级配置

  ```
  run-as：是指可以运行suricata的用户及用户组
  coredump：配置suricata核心转储配置
  host-mode：设置suricata设置是监听设备还是路由器。
  unix-command：开启通过unix命令使用外部工具连接suricata获取信息或修改配置。
  magic-file：magic文件路径
  engine-analysis：开启配置后，各模块会打印模块配置到默认日志内。
  pcre：为pcre支持递归和匹配
  host-os-plicy：配置检测设备系统及ip地址，增加suricata数据检测、处理、还原的能力。
  defrag：数据包碎片整理
  flow：设置流最大可用内存及紧急模式等。
  flow-timeouts：通过配置各协议时间限制流在内存中的时间。
  stream：stream模块可以跟踪tcp连接。该模块包含stream-tarcking和reassembly-engine
  detect：检测模块配置，检测模块为内部组生成签名，并可以通过修改配置文件去使用签名和管理内存、cpu的使用。
  profiling：性能分析设置。
  nfq：nfqueue设置
  nflog：设置nflog。
  netmap：netmap支持设置
  pfring：使用pf_ring库增强libcap的抓包性能，执行抓包。配置网卡，支持负载均衡
  ipfw：FreeBSD上防火墙 
  napatech：napatech网卡支持设置
  ```

###### 1.3.2Rules配置

​        Rules内的足则配置支持三种方式：proofpoint规则、snort规则和自定义规则。同时支持使用Oinkmaster规则管理系统。一条suricata 规则由三个部分组成：操作(Action)；头部(Header)；规则(Rule options)

- ##### 操作


四种可选操作分别为：Pass，Drop，Reject 和 Alert。

```
Pass
如果匹配到了规则，则 Suricata 停止扫描数据包并跳过当前所有规则（指的是当前规则包内的所有规则）。

Drop
这个值只能在 IPS/inline 模式下才可使用。
如果匹配到了规则，则 Suricata 也会立马停止扫描数据包并将当前数据包丢弃。

Reject
不同于 Drop 直接丢弃数据包，Reject 在匹配到规则时会主动进行拒绝数据包。当数据包为 TCP 时，会返回一个 RST 数据包重置连接；为其他数据包时，则会返回一个 ICMP-error 数据包。
在拒绝了连接后 Suricata 也会产生相应的警报。也是在IPS下才有。

Alert
当匹配到规则时，Suricata 不会对数据包进行任何操作，会像对正常数据包一样进行放行，除了会记录一条只有管理员能够看到的警报。
```

以上四种操作也是有优先级的，默认的优先级为：Pass > Drop > Reject > Alert。也就是规则在匹配时会优先考虑包含 Pass 的规则，其次才是 Drop，再然后是 Reject，最后在考虑包含 Alert 的规则。

- ##### 头部


头部中包含如下几项：

协议(Protocol)；源/目的地址(Source and destination)；端口号(Ports)；流向(Direction)

```
协议：这个字段用来告诉 Suricata 当前规则所包含的协议。其取值可以为：tcp，udp，icmp，ip，http，ftp，tls(包含ssl)，smb，dns等等。

源/目的地址：可以设置为IP 地址或者在配置文件(Suricata.yaml)里定义的变量。仅支持ip及掩码设置，不支持域名解析，多个ip地址是可以使用‘[  ]’,各ip或ip段之间使用‘，’分隔。否定运算符用‘！’表示。‘any’表示匹配任意ip。

端口号：不同的协议使用不同的端口号，例如 HTTP 使用 80 端口，而 HTTPS 则使用 443。通常情况下端口号会设置为 any，这样会适应所有的协议。

流向：告诉规则匹配哪些流量数据，是匹配从外部网络进来的，还是匹配从内部网络出去的，亦或者两种同时匹配。其中，每条规则都必须有一个向右的箭头。
```

- 规则

元信息(meta-information)；头部(headers)；有效载荷(payloads)；流(flows)

1.Meta关键字：

```
msg：msg就是当本条规则匹配中时，显示在日志中的提示内容，必须为每条规则的第一个关键字；
sid(signature ID)：用于唯一规则标识，sid不能重复，0-10000000 VRT保留，20000000-29999999 Emerging保留，30000000+：公用
rev(revision)：表示规则的版本，每次修改规则rev递增1
reference：连接外部信息来源，补充描述。直接指向可以找到有关规则和规则试图解决的问题的信息的位置字段表明这条规则相关信息所在url，在reference.config配置文件中引用。
classtype	根据规则检测到的攻击行为类型为规则分类
metadata	允许在规则中添加其他非功能性信息，相当于备注，suricata会自动忽略其后内容
```

2.Payload关键字：

```
content：大部分规则都要使用这个关键字来匹配数据包中的内容。
nocase:内容修饰，表示不区分大小写
offset:内容修饰，表示从数据包开始位置0往后偏移多少字节后进行匹配
depth：内容修饰，表示匹配数据包结束的位置
distance：内容修饰，表示本次匹配必须在上一次匹配结束位置到distance设置的偏移区间之外
within：内容修饰，表示本次匹配必须在上一次匹配结束位置到within设置的偏移区间之内
dsize:内容修饰，表示有效载荷大小
startswith:内容修饰，表示匹配内容以content开头
endswith:内容修饰，表示匹配内容以content结尾
isdataat:内容修饰，表示查看负载的特定部分是否仍有数据
pcre:匹配特定正则表达式，优先级低于content
```

3.Prefiltering关键字:	

```
fast_pattern:快速匹配模式，即对这一项进行优先匹配，若没有命中则跳过本条规则。
```

4.flow关键字:

```
flow:用于匹配流的方向，例如to/from客户端或to/from服务器。还可以匹配是否建立了流。
```


#### 2.输出

有几种类型的输出。总体结构为：

```
outputs:
  - fast:
    enabled: yes
    filename: fast.log
    append: yes/no
```

启用所有日志，将导致性能降低，并使用更多的磁盘空间，因此只启用所需的输出。

##### 1.基于行的警报日志（fast.log）

此日志包含由一行组成的警报。单个fast.log文件行的外观示例：

```
10/05/10-10:08:59.667372  [**] [1:2009187:4] ET WEB_CLIENT ACTIVEX iDefense
  COMRaider ActiveX Control Arbitrary File Deletion [**] [Classification: Web
  Application Attack] [Priority: 3] {TCP} xx.xx.232.144:80 -> 192.168.1.4:56068
-fast:                    #The log-name.
   enabled:yes            #This log is enabled. Set to 'no' to disable.
   filename: fast.log     #The name of the file in the default logging directory.
   append: yes/no         #If this option is set to yes, the last filled fast.log-file will not be
                          #overwritten while restarting Suricata.
```

##### 2.EVE（可扩展事件格式）

这是警报和事件的JSON输出。它允许与第三方工具（如logstash）轻松集成。

##### 3. 基于行的HTTP请求日志（http.log）

此日志跟踪所有HTTP流量事件。它包含HTTP请求、主机名、URI和用户代理。此信息将存储在http.log中（默认名称，在suricata日志目录中）。也可以通过使用 [Eve-log capability](https://www.osgeo.cn/suricata/output/eve/eve-json-format.html#eve-json-format) .

具有非扩展日志记录的HTTP日志行示例：

```
07/01/2014-04:20:14.338309 vg.no [**] / [**] Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_2)
AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.114 Safari/537.36 [**]
192.168.1.6:64685 -> 195.88.54.16:80
```

##### 4.数据包日志（PCAP日志）

使用PCAP日志选项，您可以将通过Suricata注册的所有数据包保存在名为 _log.pcap_. 这样，您可以随时查看所有数据包。在正常模式下，PCAP文件在默认日志目录中创建。如果在yaml文件中设置了绝对路径，也可以在其他地方创建它。

保存在示例-log dir/var/log/suricata中的文件可以用支持PCAP文件格式的每个程序打开。这可以是wireshark、tcpdump、suricata、snort和其他。

##### 5.详细警报日志（alert debug.log）

这是一种日志类型，提供有关警报的补充信息。这对于那些调查假阳性和写签名的人来说特别方便。但是，由于必须存储的信息量太大，它会降低性能。

##### 6. 警报输出到前奏（警报前奏）

要使用这种类型，您必须首先与前奏曲管理器连接。

前奏警报包含许多信息和字段，包括触发警报的包中的IPFields。此信息可分为三部分：

- 警报描述（传感器名称、日期、规则的ID（SID）等）。这总是包括在内的
- 数据包头（几乎所有IP字段、TCP-UDP等，如果相关）
- 整个包的二进制形式。

##### 7.统计

在stats中，可以设置stats.log的选项。当启用stats.log时，您可以设置将输出数据写入日志文件的时间（以秒为单位）。

##### 8.系统日志

使用此选项，可以将所有警报和事件输出发送到syslog。

##### 9. 文件存储（文件提取）

这个 file-store 输出允许将提取的文件存储到磁盘，并配置存储文件的位置。

### 四、自定义规则

#### 1.规则关键字

（上面讲了）

#### 2.自定义rules

1.访问百度www.baidu.com

在/etc/suricata/rules路径下新建test-baidu.rules文件

`alert http any any -> any any (msg:"hit baidu.com...";content:"baidu"; reference:url, www.baidu.com;)`

![image-20211027155509723](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211027155509723.png)



在`/usr/local/etc/suricata/suricata.yaml` 中添加test-baidu.rules的路径和文件（拉到最底部）

![image-20211027204717811](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211027204717811.png)

运行 suricata ：`suricata -c /etc/suricata/suricata.yaml -i ens33 -s /etc/suricata/rules/test-baidu.rules`（报错）

![image-20211029212448619](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211029212448619.png)

 解决：运行命令行`suricata -c /etc/suricata/suricata.yaml -i ens33` 即可









### 几个问题

#### 1.alert http any any -> any any (msg:"hit baidu.com...";content:"baidu"; reference:url, www.baidu.com;)

一条suricata 规则由三个部分组成：

操作(Action)；头部(Header)；规则(Rule options)

##### 1.1操作

四种可选操作分别为：Pass，Drop，Reject 和 Alert。

```
Pass
如果匹配到了规则，则 Suricata 停止扫描数据包并跳过当前所有规则（指的是当前规则包内的所有规则）。

Drop
这个值只能在 IPS/inline 模式下才可使用。
如果匹配到了规则，则 Suricata 也会立马停止扫描数据包并将当前数据包丢弃。

Reject
不同于 Drop 直接丢弃数据包，Reject 在匹配到规则时会主动进行拒绝数据包。当数据包为 TCP 时，会返回一个 RST 数据包重置连接；为其他数据包时，则会返回一个 ICMP-error 数据包。
在拒绝了连接后 Suricata 也会产生相应的警报。也是在IPS下才有。

Alert
当匹配到规则时，Suricata 不会对数据包进行任何操作，会像对正常数据包一样进行放行，除了会记录一条只有管理员能够看到的警报。
```

以上四种操作也是有优先级的，默认的优先级为：Pass > Drop > Reject > Alert。也就是规则在匹配时会优先考虑包含 Pass 的规则，其次才是 Drop，再然后是 Reject，最后在考虑包含 Alert 的规则。

这条规则中使用的是Alert，即记录警报。

##### 1.2头部

头部中包含如下几项：

协议(Protocol)；源/目的地址(Source and destination)；端口号(Ports)；流向(Direction)

```
协议：这个字段用来告诉 Suricata 当前规则所包含的协议。其取值可以为：tcp，udp，icmp，ip，http，ftp，tls(包含ssl)，smb，dns等等

源/目的地址：可以设置为IP 地址或者在配置文件(Suricata.yaml)里定义的变量，这里为地址为any

端口号：不同的协议使用不同的端口号，例如 HTTP 使用 80 端口，而 HTTPS 则使用 443。通常情况下端口号会设置为 any，这样会适应所有的协议。

流向：告诉规则匹配哪些流量数据，是匹配从外部网络进来的，还是匹配从内部网络出去的，亦或者两种同时匹配。其中，每条规则都必须有一个向右的箭头。
```

alert（操作） http（协议） **any**（源地址） any（端口号） -> （流向）**any**（目的地址） any（端口号） (msg:"hit baidu.com...";content:"baidu"; reference:url, www.baidu.com;)

##### 1.3规则 name: settings;

元信息(meta-information)；头部(headers)；有效载荷(payloads)；流(flows)

```
msg：msg就是当本条规则匹配中时，显示在日志中的提示内容，必须为每条规则的第一个关键字；

content：大部分规则都要使用这个关键字来匹配数据包中的内容。这里为baidu，即在数据包中匹配“baidu”字符

reference字段表明这条规则相关信息所在url，在reference.config配置文件中引用。

```

##### 2.百度是http还是https

https 

2015年开始使用https，可能是为了更安全吧

![image-20211029162609174](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211029162609174.png)

##### 3.为什么给百度配规则时报错（已解决）

命令行输入重复

3.1改为https 协议  报错![image-20211029162750825](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211029162750825.png)

打开http.log记录日志



![image-20211030191529431](C:\Users\anna\AppData\Roaming\Typora\typora-user-images\image-20211030191529431.png)
