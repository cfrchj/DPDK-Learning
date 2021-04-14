QNSM应用层开发简介。

### 代码结构

以dns.c为例

- dns_init

  初始化，调用dns_reg

  - dns_reg
    - qnsm_dpi_proto_enable(EN_QNSM_DPI_DNS)
    
      启用DNS解析

    - qnsm_dpi_service_classify_reg
  
    注册相应端口。以DNS为例，下层会将UDP，端口53的流量就会交给dns.c进行解析。其中还调用了dns_udp_classify。
  
      - dns_udp_classify
  
        dns_info是用于储存解析的字段的结构。初始化指针DNS_INFO \*dns_info = NULL; 并指示该结构对应dns解析: dns_info = qnsm_dpi_proto_data(EN_QNSM_DPI_DNS);  数据包类型为DNS: pkt_info->dpi_app_prot = EN_QNSM_DPI_CoAP; 最后将结构传给参数*arg = dns_info;
  
    - qnsm_dpi_proto_init_reg
    
      - dns_udp_info_init
    
        分配对应的DPDK内存空间用于解析字段。调用DPDK函数rte_zmalloc_socket为dns_info分配内存空间，并将部分内存置0.
    
    - qnsm_dpi_prot_reg
    
      - dns_parse
    
        解析dns packet。其中的dns_info来自于(DNS_INFO \*)arg, 即之前分配的DPDK内存。从\*pkt_info中读取数据包字段并解析，将解析的字段存至dns_info。
      
    - qnsm_dpi_prot_final_reg
    
      - dns_free
    
        将dns_info中的字段置0/NULL，以便解析下一个数据包使用。
  
- 发送至kafka

  要将解析的数据发至kafka，还需要调用dns_send, dns_encap_info, dns_msg_proc. dns.c中已经实现了这几个函数，但没有调用。调用时，在dns_reg中添加

  (void)qnsm_dpi_prot_reg(EN_QNSM_DPI_DNS, dns_send, 5);

  (void)qnsm_dpi_msg_reg(EN_QNSM_DPI_DNS, dns_encap_info, dns_msg_proc);

  - dns_send

    调用后续与kafka相关的组件。

  - dns_encap_info

    将存在dns_parse解析时存在dns_info中的字段读取至\*buf。

  - dns_msg_proc

    从\*buf中依次读取字段并调用cJSON封装，最后调用qnsm_kafka_send_msg发送至kafka。

### 开发注意事项

新增应用层解析时按照上述代码结构修改即可。

除了解析字段之外，涉及到额外的几处代码改动。

需要在qnsm_dpi_ex.h中增加新的协议，例如，增加了MQTT协议，那么需要添加XX(21, QNSM_DPI_MQTT,   MQTT)。注意需要在MAX之前。例如MAX原本是21，将MQTT增加至21，将MAX修改为XX(22, QNSM_DPI_PROTO_MAX, other). 修改了qnsm_dpi_ex.h需要重新编译基础库。

需要在qnsm_session.c添加new_procotol_init()，例如增加mqtt_init()。

发送至kafka时，在kafka中新建topic，还需要在qnsm_sessm.xml增加相应的protocol。

  