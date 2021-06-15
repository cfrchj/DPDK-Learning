# **QNSM配置指南**	

运行QNSM主要需要对qnsm_inspect.cfg文件进行修改配置，需要配置的内容主要包括DPDK运行环境，如网卡、内存等，以及QNSM各组件的数量，占用资源和在流水线中的位置。通过修改该配置文件，可以控制QNSM运行流水线的结构和组件之间的关系。

QNSM组件包括以下三类：

- 转发面组件：SESSM，VIP_AGG，SIP_AGG，DUMP， EDGE，DETECT

- 控制面组件：MASTER
- IDPS管理服务：FM、FR、CS

CPU开启超线程，两个NUMA node共计40个核心，我们需要给各个组件分配相应CPU资源。

- DDOS检测和IDPS原本应该各占用一个NUMA node，每个node 20个逻辑核，我们暂时不需要IDPS模块，因此两个node上的cpu核都可以分配给DDOS检测组件使用（同node上的逻辑核之间数据传输速度较快）。
- DDOS检测包括SESSM，VIP_AGG，SIP_AGG，DUMP， EDGE，MASTER组件
- IDPS包括DETECT，FM，FR，CS（暂时用不到）

配置前需要先明确CPU结构，一共有多少个核，哪些核可用：

```
$RTE_SDK/tools/cpu_layout.py
============================================================
Core and Socket Information (as reported by '/proc/cpuinfo')
============================================================

cores =  [0, 1, 2, 3, 4, 8, 9, 10, 11, 12]
sockets =  [0, 1]

        Socket 0        Socket 1
        --------        --------
Core 0  [0, 20]         [10, 30]
Core 1  [1, 21]         [11, 31]
Core 2  [2, 22]         [12, 32]
Core 3  [3, 23]         [13, 33]
Core 4  [4, 24]         [14, 34]
Core 8  [5, 25]         [15, 35]
Core 9  [6, 26]         [16, 36]
Core 10 [7, 27]         [17, 37]
Core 11 [8, 28]         [18, 38]
Core 12 [9, 29]         [19, 39]
```

上面列出的是目前实验室服务器上的cpu结构，Socket 0/1代表有2个NUMA node，每个NUMA node上各有10个cpu核，每个cpu核上又各有2个逻辑核（也可以叫超线程），所以一共有2 * 2 * 10 = 40个逻辑核可用。需要注意的是，在配置时，每一个逻辑核都有一个配置文件中的表示方法与其对应，举例来说上面的23号逻辑核，在配置文件中表示为s0c2h，表示该核是node 0 core 2 hyperthread 1，6号逻辑核表示为s0c6，即node 0 core 6 hyperthread 0。有两点需要注意，一是上面显示的cores =  [0, 1, 2, 3, 4, 8, 9, 10, 11, 12]中的core编号与配置文件中的编号不同，配置文件中的编号为从0开始递增（不会空掉5,6,7），二是带h表示选择超线程1，不带h表示选择超线程0。

除了配置组件与逻辑核之间的绑定关系，还需要配置各个组件用到的队列，即组件输入从哪个队列来，输出从哪个队列出（有输入必须配置输出，否则会dump core），配置举例：

```
[EAL]
log_level = 3                 ; 日志等级
n = 4                         ; 内存通道数目
socket_mem = 20480            ; node0, 1上分配的大页内存大小，单位M
master_lcore = 24             ; DPDK eal初始化逻辑核

;mbuf mempool cfg
;add mbuf priavte size para
[MEMPOOL0]                    ; 收包使用mbuf内存池
buffer_size = 2304            ; mbuf大小, 字节为单位
pool_size = 131072            ; DDOS收包内存池大小
cache_size = 256              ; mempool per-lcore cache size
cpu = 0	;socket_id             ; 表示numa node 0
private_size = 64             ;sizeof(QNSM_PACKET_INFO)

;for dump
[MEMPOOL1]                    ; 数据包dump复制使用的内存池
buffer_size = 2304
pool_size = 131072
cache_size = 256
cpu = 0	                      ; 表示numa node 0
private_size = 64            ; sizeof(QNSM_PACKET_INFO)

;link cfg
[LINK0]                      ; 网卡LINK0，支持多个网卡
rss_qs = 0 1 2 3             ; 0-7队列开启RSS
rss_proto_ipv4 = TCP UDP     ; Ipv4的RSS方式
rss_proto_ipv6 = TCP TCP_EX UDP UDP_EX ; Ipv6的RSS方式
symmetrical_rss = yes        ; 开启对称hash

[LINK1]
rss_qs = 0 1 2 3
rss_proto_ipv4 = TCP UDP
rss_proto_ipv6 = TCP TCP_EX UDP UDP_EX
symmetrical_rss = yes

;rx queue cfg
[RXQ0.0]                     ; 网卡0的收包队列0
size = 2048                  ; 队列长度
burst = 32                   ; burst收包的大小

[RXQ0.1]
size = 2048
burst = 32

[RXQ0.2]
size = 2048
burst = 32

[RXQ0.3]
size = 2048
burst = 32

[RXQ1.0]
size = 2048
burst = 32

[RXQ1.1]
size = 2048
burst = 32

[RXQ1.2]
size = 2048
burst = 32

[RXQ1.3]
size = 2048
burst = 32

[SWQ16]                      ; 软件队列16
cpu = 0                      ; Numa node 0
mempool = MEMPOOL1           ; 数据包复制时从MEMPOOL1分配mbuf
dump = yes                   ; dump组件收包使用的队列

[SWQ17]
cpu = 0
mempool = MEMPOOL1
dump = yes

[SWQ18]
cpu = 0
mempool = MEMPOOL1
dump = yes

[SWQ19]
cpu = 0
mempool = MEMPOOL1
dump = yes

[SWQ20]
cpu = 0
mempool = MEMPOOL1
dump = yes

[SWQ21]
cpu = 0
mempool = MEMPOOL1
dump = yes

[SWQ22]
cpu = 0
mempool = MEMPOOL1
dump = yes

[SWQ23]
cpu = 0
mempool = MEMPOOL1
dump = yes

;app cfg
[PIPELINE0]        ; pipeline 组件配置项，编号不要重复即可
type = MASTER      ; 组件类型，这里是QNSM控制面MASTER组件
core = s0c0h       ; 部署在node 0 core 0 hyperthread 1

[PIPELINE1]        
type = SESSM       ; SESSM组件
core = s0c1        ; 部署在node 0 core 1 hyperthread 0
pktq_in = RXQ0.0   ; 从网卡0的队列0收包
pktq_out = SWQ0 SWQ8 SWQ16 SWQ32 SWQ40 ; 发出数据包所用队列
timer_period = 10  ; 定时器调度最小间隔，ms为单位，实时性要求高，可以调小间隔，占用更多CPU资源；反之，调大间隔，释放CPU资源。

[PIPELINE2]
type = SESSM
core = s0c2
pktq_in = RXQ0.1
pktq_out = SWQ1 SWQ9 SWQ17 SWQ33 SWQ41
timer_period = 10

[PIPELINE3]
type = SESSM
core = s0c3
pktq_in = RXQ0.2
pktq_out = SWQ2 SWQ10 SWQ18 SWQ34 SWQ42
timer_period = 10

[PIPELINE4]
type = SESSM
core = s0c4
pktq_in = RXQ0.3
pktq_out = SWQ3 SWQ11 SWQ19 SWQ35 SWQ43
timer_period = 10

[PIPELINE8]
type = SESSM
core = s0c6
pktq_in = RXQ1.0
pktq_out = SWQ4 SWQ12 SWQ20 SWQ36 SWQ44
timer_period = 10

[PIPELINE9]
type = SESSM
core = s0c9
pktq_in = RXQ1.1
pktq_out = SWQ5 SWQ13 SWQ21 SWQ37 SWQ45
timer_period = 10

[PIPELINE10]
type = SESSM
core = s0c8
pktq_in = RXQ1.2
pktq_out = SWQ6 SWQ14 SWQ22 SWQ38 SWQ46
timer_period = 10

[PIPELINE11]
type = SESSM
core = s0c6h
pktq_in = RXQ1.3
pktq_out = SWQ7 SWQ15 SWQ23 SWQ39 SWQ47
timer_period = 10

[PIPELINE12]
type = DUMP        ; DUMP组件，dump DDOS攻击数据包并保存为pcap文件
core = s0c5        ; Node 0 core 5 hyperthread 0
pktq_in = SWQ16 SWQ17 SWQ18 SWQ19 SWQ20 SWQ21 SWQ22 SWQ23
timer_period = 10

[PIPELINE17]
type = SIP_IN_AGG  ; DDOS攻击源ip数据聚合并输出
core = s0c7h       ; Node 0 core 7 hyperthread 1

[PIPELINE18]
type = VIP_AGG
core = s0c9h
pktq_in = SWQ0 SWQ1 SWQ2 SWQ3 SWQ4 SWQ5 SWQ6 SWQ7
timer_period = 10

[PIPELINE19]
type = VIP_AGG     ; IDC公网IP自学习，数据聚合和输出
core = s0c4h
pktq_in = SWQ8 SWQ9 SWQ10 SWQ11 SWQ12 SWQ13 SWQ14 SWQ15
timer_period = 10

[PIPELINE20]
type = VIP_AGG
core = s0c5h
pktq_in = SWQ32 SWQ33 SWQ34 SWQ35 SWQ36 SWQ37 SWQ38 SWQ39
timer_period = 10

[PIPELINE21]
type = VIP_AGG
core = s0c8h
pktq_in = SWQ40 SWQ41 SWQ42 SWQ43 SWQ44 SWQ45 SWQ46 SWQ47
timer_period = 10

[PIPELINE22]
type = EDGE        ; 数据输出中心，以kafka接口输出
core = s0c7
```