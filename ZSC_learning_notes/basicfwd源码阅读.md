# basicfwd源码阅读

## 1 port_init()函数

```c
static inline int
port_init(uint8_t port, struct rte_mempool *mbuf_pool)
{
	struct rte_eth_conf port_conf = port_conf_default;// 这个rte_eth_conf结构体在配置网卡时要用到
	const uint16_t rx_rings = 1, tx_rings = 1;// 每个网口有多少rx和tx队列，这里都为1
	int retval;
	uint16_t q;

	if (port >= rte_eth_dev_count())  //检查端口号是否大于总的端口号，否则失败
		return -1;

	/* Configure the Ethernet device. */
	// rte_eth_dev_configure() 配置网卡
	/* 四个参数
		1. port id
		2. 要给该网卡配置多少个收包队列 这里是一个
		3. 要给该网卡配置多少个发包队列 也是一个
		4. 结构体指针类型 rte_eth_conf *
	*/
	retval = rte_eth_dev_configure(port, rx_rings, tx_rings, &port_conf);
	if (retval != 0)// 返回值0：成功，设备已配置。
		return retval;


	/*  rte_eth_rx_queue_setup() 顾名思义的函数名 配置rx队列
	配置rx队列需要六个参数
		1. port id
		2. 接收队列的索引。要在[0, rx_queue - 1] 的范围内（先前rte_eth_dev_configure中配置的）
		3. 为接收环分配的接收描述符数。（环的大小）
		4. socket id。 如果是 NUMA 架构 就使用 rte_eth_dev_socket_id(port)获取port所对应的以太网设备所连接上的socket的id；
		   若不是NUMA，该值可以是宏SOCKET_ID_ANY
		5. 指向rx queue的配置数据的指针。如果是NULL，则使用默认配置。
		6. 指向内存池mempool的指针，从中分配mbuf去操作队列。
	*/ 
	/* Allocate and set up 1 RX queue per Ethernet port. */
	for (q = 0; q < rx_rings; q++) {
		retval = rte_eth_rx_queue_setup(port, q, RX_RING_SIZE,
				rte_eth_dev_socket_id(port), NULL, mbuf_pool);
		if (retval < 0)
			return retval;
	}
	
	/* rte_eth_tx_queue_setup()
	配置tx队列需要五个参数（不需要mempool）
		1. port id
		2. 发送队列的索引。要在[0, tx_queue - 1] 的范围内（先前rte_eth_dev_configure中配置的）
		3. 为发送环分配的接收描述符数。（自定义环的大小）
		4. socket id
		5. 指向tx queue的配置数据的指针，结构体是rte_eth_txconf。
	*/
	/* Allocate and set up 1 TX queue per Ethernet port. */
	for (q = 0; q < tx_rings; q++) {
		retval = rte_eth_tx_queue_setup(port, q, TX_RING_SIZE,
				rte_eth_dev_socket_id(port), NULL);
		if (retval < 0)
			return retval;
	}
	// 启动设备
	// 设备启动步骤是最后一步，包括设置已配置的offload功能以及启动设备的发送和接收单元。成功时，可以调用以太网API导出的所有基本功能
	// （链接状态，接收/发送等）。
	/* Start the Ethernet port. */
	retval = rte_eth_dev_start(port);
	if (retval < 0)
		return retval;

	/* Display the port MAC address. */
	struct ether_addr addr;
	//rte_eth_macaddr_get()获得MAC地址
	rte_eth_macaddr_get(port, &addr);
	// #define PRIx8 "hhx"
	// 十六进制数形式输出整数 一个h表示short，即short int ，两个h表示short short，即 char。%hhx用于输出char
	printf("Port %u MAC: %02" PRIx8 " %02" PRIx8 " %02" PRIx8
			   " %02" PRIx8 " %02" PRIx8 " %02" PRIx8 "\n",
			(unsigned)port,
			addr.addr_bytes[0], addr.addr_bytes[1],
			addr.addr_bytes[2], addr.addr_bytes[3],
			addr.addr_bytes[4], addr.addr_bytes[5]);

	/* Enable RX in promiscuous mode for the Ethernet device. */
	rte_eth_promiscuous_enable(port); //设置网卡为混杂模式

	return 0;
}
```

## 2 lcore_main()函数

```c
static __attribute__((noreturn)) void
lcore_main(void)
{
	const uint8_t nb_ports = rte_eth_dev_count();// 获取当前可用以太网设备的总数
	uint8_t port;

	/*
	 * Check that the port is on the same NUMA node as the polling thread
	 * for best performance.
	 */
	// 当有NUMA结构时，检查网口是否在同一个NUMA node节点上，只有在一个NUMA node上时线程轮询的效率最好
	for (port = 0; port < nb_ports; port++)//检查所有设备，如果设备socket_id不等于正在运行的lcore所对应的物理socket_id，或者小于0，报错
		if (rte_eth_dev_socket_id(port) > 0 &&
				rte_eth_dev_socket_id(port) !=
						(int)rte_socket_id())
			printf("WARNING, port %u is on remote NUMA node to "
					"polling thread.\n\tPerformance will "
					"not be optimal.\n", port);

	printf("\nCore %u forwarding packets. [Ctrl+C to quit]\n",
			rte_lcore_id());//退出转发包

	/* Run until the application is quit or killed. */
	for (;;) {
		/*
		 * Receive packets on a port and forward them on the paired
		 * port. The mapping is 0 -> 1, 1 -> 0, 2 -> 3, 3 -> 2, etc.
		 */
		/*
			一个端口收到包，就立刻转发到另一个端口
			0 和 1
			2 和 3
			……
		*/
		for (port = 0; port < nb_ports; port++) {

			/* Get burst of RX packets, from first port of pair. */
			struct rte_mbuf *bufs[BURST_SIZE];// mbuf的结构体 收到的包存在这里，也是要发出去的包
			
			/* 收包函数：rte_eth_rx_burst 从以太网设备的接收队列中检索一连串（burst收发包机制）输入数据包。检索到的数据包存储在rte_mbuf结构中。
			参数四个
				1. port id （收到哪个网口）
				2. 队列索引 （的哪一条队列），范围要在[0, rx_queue - 1] 的范围内（rte_eth_dev_configure中的） 这里就1个队列
				3. 指向 rte_mbuf 结构的 指针数组 的地址。要够容纳第四个参数所表示的数目的指针。（把收到的包存在哪里？）
				4. 要检索的最大数据包数
			rte_eth_rx_burst()是一个循环函数，从RX队列中收包达到设定的最大数量为止。
			*/
			const uint16_t nb_rx = rte_eth_rx_burst(port, 0,
					bufs, BURST_SIZE);

			if (unlikely(nb_rx == 0))
				continue;

			/* Send burst of TX packets, to second port of pair. */
			/* 发包函数：rte_eth_tx_burst 在由port id指示的以太网设备的传输队列（由索引指示）发送一连串输出数据包。
			参数四个：
				1. port id（从哪个网口）
				2. 队列索引（的哪条队列发出），范围要在[0, tx_queue - 1] 的范围内（rte_eth_dev_configure中的）
				3. 指向包含要发送的数据包的 rte_mbuf 结构的 指针数组 的地址。（要发送的包的内容在哪里）
				4. 要发送的数据包的最大数量。
			返回值是发送的包的数量。
			*/
			const uint16_t nb_tx = rte_eth_tx_burst(port ^ 1, 0,
					bufs, nb_rx);// port 异或 1 --> 0就和1是一对，2就和3是一对。  
								// 0 收到包就从 1 转发， 3 收到包 就从 2 口转发。	 如果是0  就异或 则1发		
			/* Free any unsent packets. */
			//unlikely代表可能不太出现 出现则释放
			if (unlikely(nb_tx < nb_rx)) {
				uint16_t buf;
				for (buf = nb_tx; buf < nb_rx; buf++)
					rte_pktmbuf_free(bufs[buf]);
			}
		}
	}
}
```

## 3 main()函数

```c
int
main(int argc, char *argv[])
{
	struct rte_mempool *mbuf_pool;// 指向内存池结构的指针
	unsigned nb_ports;// 网口个数
	uint8_t portid;// 网口号

	/* Initialize the Environment Abstraction Layer (EAL). */
	int ret = rte_eal_init(argc, argv);
	if (ret < 0)
		rte_exit(EXIT_FAILURE, "Error with EAL initialization\n");

	argc -= ret;
	argv += ret;

	/* Check that there is an even number of ports to send/receive on. */
	nb_ports = rte_eth_dev_count();// 获取当前可用以太网设备的总数
	if (nb_ports < 2 || (nb_ports & 1))// 检查端口个数是否小于两个或者是奇数，则出错。
		rte_exit(EXIT_FAILURE, "Error: number of ports must be even\n");


	// dpdk用mbuf保存packet，mempool用于操作mbuf
	/* rte_pktmbuf_pool_create() 创建并初始化mbuf池，是 rte_mempool_create 这个函数的封装。
		五个参数：
		1. mbuf的名字 "MBUF_POOL"
		2. mbuf中的元素个数。每个端口给了8191个
		3. 每个核心的缓存大小，如果该参数为0 则可以禁用缓存。本程序中是250
		4. 每个mbuf中的数据缓冲区大小
		5. 应分配内存的套接字标识符。
		返回值：分配成功时返回指向新分配的mempool的指针。
		mempool的指针会传给 port_init 函数，用于 setup rx queue
	*/

	/* Creates a new mempool in memory to hold the mbufs. */
	mbuf_pool = rte_pktmbuf_pool_create("MBUF_POOL", NUM_MBUFS * nb_ports,
		MBUF_CACHE_SIZE, 0, RTE_MBUF_DEFAULT_BUF_SIZE, rte_socket_id());//rte_socket_id()返回正在运行的lcore所对应的物理socket。

	if (mbuf_pool == NULL)
		rte_exit(EXIT_FAILURE, "Cannot create mbuf pool\n");

	/* Initialize all ports. */
	for (portid = 0; portid < nb_ports; portid++)
		if (port_init(portid, mbuf_pool) != 0)
			rte_exit(EXIT_FAILURE, "Cannot init port %"PRIu8 "\n",
					portid);

	if (rte_lcore_count() > 1)// basicfwd只需要使用一个逻辑核
		printf("\nWARNING: Too many lcores enabled. Only 1 used.\n");

	/* Call lcore_main on the master core only. */
	lcore_main();

	return 0;
}
```

## 4 宏定义

```C
#define RX_RING_SIZE 128 // 接收环大小
#define TX_RING_SIZE 512 // 发送环大小

#define NUM_MBUFS 8191 // mbuf中的元素个数，推荐数量是2的幂次-1
#define MBUF_CACHE_SIZE 250//cache大小
#define BURST_SIZE 32// Burst收发包模式的一次完成多个数据包的收发

static const struct rte_eth_conf port_conf_default = {
	.rxmode = { .max_rx_pkt_len = ETHER_MAX_LEN }
};
```

