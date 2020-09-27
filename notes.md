# 1 NetStream数据流

## 1.1 设备配置数据上报

```
Fabric配置采集器IP
Campus配置南向浮动IP

# 查看是否建链
ss -anp | grep 30002

注意：设备配置数据上报后，要在应用面添加设备(设备添加进资源表白名单(Carbon或RDB))，否则南向Topic不能收到数据

其他：
Telemetry数据
	CE设备采用Telemetry机制进行指标数据的采样
		ss -anp | grep 30001
	S设备采用Netconf机制进行指标数据的采样
		ss -anp | grep 27371
```

### 1.1.1 CE设备

#### 原始流

##### 增强模式

```
system-view
# 使能增强模式
assign forward enp netstream enable slot 0
# 查看是否使用增强模式
display forward enp
display forward enp slot 0
commit
```

##### 采样

```
system-view
# 系统视图采样
netstream sampler random-packets 1 inbound 
netstream sampler random-packets 1 outbound
# 接口视图采样
interface 100GE 1/0/1
	netstream sampler random-packets 1 inbound 
	netstream sampler random-packets 1 outbound
```

##### 老化

```
system-view
# 活跃流老化(分钟)
netstream timeout ip active 1
netstream timeout ipv6 active 1
# 非活跃流老化(秒)
netstream timeout ip inactive 5
netstream timeout ipv6 inactive 5
# TCP连接的FIN和RST老化
netstream timeout ip tcp-session
netstream timeout ipv6 tcp-session
# 强制老化
reset netstream cache ip slot 0
reset netstream cache ipv6 slot 0
```

##### 输出

```
system-view
netstream export ip source 10.136.193.162
netstream export ip host 10.136.193.233 30002
netstream export ipv6 source 10.136.193.162
netstream export ipv6 host 10.136.193.233 30002
```

##### 接口索引

```
system-view
netstream export ip index-switch 32
netstream export ipv6 index-switch 32
```

##### 报文格式

```
system-view
netstream export ip version 9
netstream export ipv6 version 9
```

##### 使能接口

```
system-view
interface 100GE 1/0/1
	netstream inbound ip
	netstteam outbound ip
	netstream inbound ipv6
	netstteam outbound ipv6
```

##### 查看配置结果

```
# 查看原始流统计信息
display netstream cache ip origin slot 0
display netstream cache ipv6 origin slot 0
# 查看模板输出信息
display netstream export ipv6 template
# 查看流统计信息
display netstream statistics ipv6 slot 0
# 查看配置
display netstream all
```

#### 聚合流

##### 增强模式

```
system-view
# 使能增强模式
assign forward enp netstream enable slot 0
# 查看是否使用增强模式
display forward enp
display forward enp slot 0
commit
```

##### 采样

```
system-view
# 系统视图采样
netstream sampler random-packets 1 inbound 
netstream sampler random-packets 1 outbound
# 接口视图采样
interface 100GE 1/0/1
	netstream sampler random-packets 1 inbound 
	netstream sampler random-packets 1 outbound
```

##### 老化

```
system-view
# 活跃流老化(分钟)
netstream aggregation timeout ip active 1
netstream aggregation timeout ipv6 active 1
# 非活跃流老化(秒)
netstream aggregation timeout ip inactive 1
netstream aggregation timeout ipv6 inactive 1
# 强制老化
reset netstream cache ip slot 0
reset netstream cache ipv6 slot 0
```

##### 聚合流统计输出

```
system-view
# 会进入聚合视图
netstream aggregation ip [as|as-tos|bgp-nexthop-tos|destination-prefix|destination-prefix-tos|index-tos|mpls-label|prefix|prefix-tos|protocol-port|protocol-port-tos|source-prefix|source-prefix-tos|source-index-tos|vlan-id]

netstream aggregation ipv6 [as|as-tos|bgp-nexthop-tos|destination-prefix|destination-prefix-tos|index-tos|mpls-label|prefix|prefix-tos|protocol-port|protocol-port-tos|source-prefix|source-prefix-tos|source-index-tos|vlan-id]

netstream export ip source 10.136.193.162
netstream export ip host 10.136.193.233 30002
netstream export ipv6 source 10.136.193.162
netstream export ipv6 host 10.136.193.233 30002

# 使能聚合功能
enable
quit
```

##### 接口索引

```
system-view
netstream export ip index-switch 32
netstream export ipv6 index-switch 32
```

##### 报文格式

```
system-view
# 会进入聚合视图
netstream aggregation ip [as|as-tos|bgp-nexthop-tos|destination-prefix|destination-prefix-tos|index-tos|mpls-label|prefix|prefix-tos|protocol-port|protocol-port-tos|source-prefix|source-prefix-tos|source-index-tos|vlan-id]

netstream aggregation ipv6 [as|as-tos|bgp-nexthop-tos|destination-prefix|destination-prefix-tos|index-tos|mpls-label|prefix|prefix-tos|protocol-port|protocol-port-tos|source-prefix|source-prefix-tos|source-index-tos|vlan-id]

export version 9
commit
```

##### 使能接口

```
system-view
interface 100GE 1/0/1
	netstream inbound ip
	netstteam outbound ip
	netstream inbound ipv6
	netstteam outbound ipv6
```

##### 查看配置结果

```
# 查看聚合流信息
netstream aggregation ip [as|as-tos|bgp-nexthop-tos|destination-prefix|destination-prefix-tos|index-tos|mpls-label|prefix|prefix-tos|protocol-port|protocol-port-tos|source-prefix|source-prefix-tos|source-index-tos|vlan-id]

netstream aggregation ipv6 [as|as-tos|bgp-nexthop-tos|destination-prefix|destination-prefix-tos|index-tos|mpls-label|prefix|prefix-tos|protocol-port|protocol-port-tos|source-prefix|source-prefix-tos|source-index-tos|vlan-id]
# 查看模板输出信息
display netstream export ip template
display netstream export ipv6 template
# 查看流统计信息
display netstream statistics ip slot 0
display netstream statistics ipv6 slot 0
# 查看配置
display netstream all
```

#### 灵活流

##### 增强模式

```
system-view
# 使能增强模式
assign forward enp netstream enable slot 0
# 查看是否使用增强模式
display forward enp
display forward enp slot 0
commit
```

##### 配置灵活流模板

```
system-view
# 创建一个模板并进入模板视图
netstream record {record-name} ip
netstream record {record-name} ipv6
# 描述模板信息(可选)
description {description-information}
# 配置聚合关键字
match ip [destination-address|destination-port|tos|protoco|source-address|source-port]
match ipv6 [destination-address|destination-port|tos|protoco|source-address|source-port]
# 配置流中包含报文数和字节数(可选)
collect counter [bytes|packets]
# 配置流中包含出、入接口索引
collect interface [input|output]
commit
```

##### 采样

```
system-view
# 系统视图采样
netstream sampler random-packets 1 inbound 
netstream sampler random-packets 1 outbound
# 接口视图采样
interface 100GE 1/0/1
	netstream sampler random-packets 1 inbound 
	netstream sampler random-packets 1 outbound
```

##### 老化

```
system-view
# 活跃流老化(分钟)
netstream timeout ip active 1
netstream timeout ipv6 active 1
# 非活跃流老化(秒)
netstream timeout ip inactive 1
netstream timeout ipv6 inactive 1
# 强制老化
reset netstream cache ip slot 0
reset netstream cache ipv6 slot 0
```

##### 灵活流统计输出

```
system-view
netstream export ip source 10.136.193.162
netstream export ip host 10.136.193.233 30002
netstream export ipv6 source 10.136.193.162
netstream export ipv6 host 10.136.193.233 30002
```

##### 接口索引

```
system-view
netstream export ip index-switch 32
netstream export ipv6 index-switch 32
```

##### 报文格式

```
system-view
netstream export ip version 9
netstream export ipv6 version 9 
commit
```

##### 使能接口的灵活流统计

```
system-view
interface 100GE 1/0/1
	# 将灵活流模板应用到接口上
	netstream record {record-name} ip
	netstream record {record-name} ipv6
	netstream inbound ip
	netstteam outbound ip
	netstream inbound ipv6
	netstteam outbound ipv6
```

##### 查看配置结果

```
# 查看灵活流信息
display netstream cache ip record {record-name}
# 查看模板输出信息
display netstream export ip template
display netstream export ipv6 template
# 查看流统计信息
display netstream statistics ip slot 0
display netstream statistics ipv6 slot 0
# 查看配置
display netstream all
```

#### VXLAN灵活流

##### 增强模式

```
system-view
# 使能增强模式
assign forward enp netstream enable slot 0
# 查看是否使用增强模式
display forward enp
display forward enp slot 0
commit
```

##### 配置灵活流模板

```
system-view
# 创建一个模板并进入模板视图
netstream record {record-name} vxlan inner-ip
# 描述模板信息(可选)
description {description-information}
# 配置聚合关键字
match inner-ip [destination-address|destination-port|tos|protocol|source-address|source-port|source-mac|destination-mac|cvlan]
# 配置流中包含报文数和字节数(可选)
collect counter [bytes|packets]
# 配置流中包含出、入接口索引
collect interface [input|output]
commit
```

##### 采样

```
system-view
# 系统视图采样
netstream sampler random-packets 1 inbound 
netstream sampler random-packets 1 outbound
# 接口视图采样
interface 100GE 1/0/1
	netstream sampler random-packets 1 inbound 
	netstream sampler random-packets 1 outbound
```

##### 老化

```
system-view
# 活跃流老化(分钟)
netstream timeout vxlan inner-ip active 1
# 非活跃流老化(秒)
netstream timeout vxlan inner-ip inactive 1
# 配置由TCP连接的FIN和RST报文触发老化
netstream timeout vxlan inner-ip tcp-session
# 强制老化
reset netstream cache vxlan inner-ip slot 0
```

##### VXLAN灵活流统计输出

```
system-view
netstream export vxlan inner-ip source 10.136.193.162
netstream export vxlan inner-ip host 10.136.193.233 30002
```

##### 接口索引

```
system-view
netstream export vxlan inner-ip index-switch 32
```

##### 报文格式

```
system-view
netstream export vxlan inner-ip version 9
commit
```

##### 使能接口的灵活流统计

```
system-view
interface 100GE 1/0/1
	# 将灵活流模板应用到接口上
	netstream record {record-name} vxlan inner-ip
	netstream inbound ip
	netstteam outbound ip
```

##### 查看配置结果

```
# 查看灵活流信息
display netstream cache vxlan inner-ip record {record-name}
# 查看模板输出信息
display netstream export vxlan inner-ip template
# 查看流统计信息
display netstream statistics vxlan inner-ip slot 0
# 查看配置
display netstream all
```

#### 二层流

##### 增强模式

```
system-view
# 使能增强模式
assign forward enp netstream enable slot 0
# 查看是否使用增强模式
display forward enp
display forward enp slot 0
commit
```

##### 采样

```
system-view
# 系统视图采样
netstream sampler random-packets 1 inbound 
netstream sampler random-packets 1 outbound
# 接口视图采样
interface 100GE 1/0/1
	netstream sampler random-packets 1 inbound 
	netstream sampler random-packets 1 outbound
```

##### 老化

```
system-view
# 活跃流老化(分钟)
netstream timeout ethernet active 1
# 非活跃流老化(秒)
netstream timeout ethernet inactive 5
# 强制老化
reset netstream cache ethernet slot 0
```

##### 输出

```
system-view
netstream export ethernet source 10.136.193.162
netstream export ethernet host 10.136.193.233 30002
```

##### 接口索引

```
system-view
netstream export ethernet index-switch 32
```

##### 报文格式

```
system-view
netstream export ethernet version 9
```

##### 使能接口

```
system-view
interface 100GE 1/0/1
	netstream inbound ethernet
	netstteam outbound ethernet
```

##### 查看配置结果

```
# 查看原始流统计信息
display netstream cache ethernet slot 0
# 查看模板输出信息
display netstream export ethernet template
# 查看流统计信息
display netstream statistics ethernet slot 0
# 查看配置
display netstream all
```

### 1.1.2 S设备

#### 原始流

```
# 采样
interface gigabitethernet 1/0/1
	ip netstream sampler fix-packets 1 inbound
	ip netstream sampler fix-packets 1 outbound
	ip netstream inbound
	ip netstream outbound
	
	ipv6 netstream sampler fix-packets 1 inbound
	ipv6 netstream sampler fix-packets 1 outbound
	ipv6 netstream inbound
 	ipv6 netstream outbound

# 老化
ip netstream timeout active 5
ip netstream timeout inactive 5 
ip netstream tcp-flag enable

# 输出
ip netstream export source 10.137.67.83
ip netstream export host 10.136.193.62 30002
ipv6 netstream export source 10.137.67.83
ipv6 netstream export host 10.136.193.62 30002

# 接口索引
ip netstream export index-switch 32
ipv6 netstream export index-switch 32

# 报文格式
ip netstream export version 9
ipv6 netstream export version 9

# 显示
display ip netstream statistics slot 1
display ipv6 netstream statistics slot 1
```

#### 聚合流

```
# 配置聚合流输出,会进入聚合视图
ip netstream aggregation [as|as-tos|destination-prefix|destination-prefix-tos|prefix|prefix-tos|protocol-port|protocol-port-tos| source-prefix|source-prefix-tos]
	ip netstream export source 10.1.2.1
	ip netstream export host 10.1.2.2 30002
	enable
	export version 9
	quit
interface gigabitethernet 1/0/1
	ip netstream inbound
	ip netstream outbound
	quik
display ip netstream statistics slot 1
```

#### 灵活流

```
ip netstream record {test}
	match ip destination-address
	match ip destination-port
	collect interface input
	collect interface output
	collect counter bytes
	collect counter packets
	quit
ip netstream export source 10.1.2.1
ip netstream export host 10.1.2.2 6000
interface gigabitEthernet 1/0/1 
	port ip netstream record test
	ip netstream inbound
	ip netstream outbound
	quit
display ip netstream statistics slot 1
```

## 1.2 数据上报到南向Topic（NTAFlowStreamRaw）

### 1.2.1 南向采集服务日志

```
# 南向采集服务日志，需开启Debug日志
FabricDriverService
/opt/oss/log/FabricInsight/FabricDriverService/fabricdriverservice-2-0/log/root.log
CampusDriverService
/opt/oss/log/CampusInsight/CampusDriverService/campusdriverservice-0-0/log/root.log
```

### 1.2.2 Kafka状态

```
方法一：通过FI面查看

方法二：后台查看
# 可通过如下方式查找Topic路径
find /srv/BigData -name *NTAFlowStreamRaw*

# 进入Topic所在路径
/srv/BigData/hadoopX/kafka/ODAEDATASET.NTAFlowStreamRaw.XXX

# log日志文件太大的解决方案
head -n 100 xxx.log > nta.log

# 按行分割
split -l 300 xxx.log newfile
# 按大小分割
split -b 500m xxx.log newfile

# 合并文件
cat newfile* > orifile
```

### 1.2.3 Spark任务状态

```
# pipeline未启动成功
ODAEPipelineMgrService日志
/opt/oss/log/FabricInsight/ODAEPipelineMgrService/odaepipelinemgrservice-2-0/log/root.log
ODAESparkDispService日志
/opt/oss/log/FabricInsight/ODAESparkDispService/odaesparkdispservice-2-0/log/root.log

# pipeline启动命令
/opt/odaetool-7.6.6-bin/bin
/opt/odaetool-7.6.6-bin/tmp/nta/
publisher status
publisher stop -id 84cca380-b894-360a-85c8-8faa71b9bd11
publisher start -folder /opt/odaetool-7.6.6-bin/tmp/nta/

网流的pipeline
fabricinsight_pipeline_nta_etl
	1、NtaFlowDeserializer解析NTAFlowStreamRaw中的Topic生成FlowDetailsPre
	2、FlowDetailsPre关联Device表和Port表，执行TQL语句，生成NTAFlowDetails
fabricinsight_pipeline_nta_splitstream
	

# pipeline启动成功，有目标数据输入但是无目标数据输出
/srv/BigData/hadoop/data1/nm/containerlogs/{Application_id}/
```

### 1.2.4 CE设备上报数据

#### IPv4（37个字段）

```json
{
	"rawData": {
		"dst_as": "0",
		"in_pkts": "400",
		"first_switched": "316473837",
		"ipv4_next_hop": "189.189.160.1",
		"l4_src_port": "9728",
		"templateId": "1507",
		"sampling_algorithm": "2",
		"src_vlan": "0",
		"in_bytes": "42400",
		"protocol": "17",
		"neMetricGroup": "10.136.193.163_1507",
		"tcp_flags": "0",
		"res_id": "54acc06d-b367-492f-b303-da3c94eca614",
		"dst_vlan": "0",
		"l4_dst_port": "4789",
		"src_as": "0",
		"direction": "1",
		"output_snmp": "13",
		"time_id": "1600642882205",
		"dst_mask": "32",
		"ipv4_dst_addr": "66.66.66.160",
		"src_tos": "0",
		"datasetName": "NTAFlowStreamRaw",
		"src_mask": "32",
		"index_length": "32",
		"ipv4_src_addr": "16.16.16.161",
		"sequence_number": "352460",
		"last_switched": "316533706",
		"input_snmp": "0",
		"unix_secs": "1600642837",
		"sys_up_time": "316534898",
		"dev_ip": "10.136.193.163",
		"source_id": "257",
		"metricGroupId": "NetStream",
		"bgp_ipv4_next_hop": "0.0.0.0",
		"sampling_interval": "1",
		"flowType": "Original"
	},
	"templateId": "1507"
}
```

#### IPv6（38个字段）

```json
{
	"rawData": {
		"ipv6_dst_addr": "ff02:0:0:0:0:1:ff00:1",
		"dst_as": "0",
		"in_pkts": "46",
		"first_switched": "304650735",
		"l4_src_port": "0",
		"templateId": "1599",
		"ipv6_src_addr": "1600:1:0:0:0:0:0:2",
		"sampling_algorithm": "2",
		"src_vlan": "2193",
		"in_bytes": "3312",
		"protocol": "58",
		"neMetricGroup": "10.136.193.161_1599",
		"bgp_ipv6_next_hop": "0:0:0:0:0:0:0:0",
		"tcp_flags": "0",
		"res_id": "bbf73867-0bd1-4e45-9ca3-5f7de0e8afa1",
		"dst_vlan": "0",
		"l4_dst_port": "34560",
		"src_as": "0",
		"direction": "0",
		"output_snmp": "0",
		"time_id": "1600642951002",
		"src_tos": "0",
		"datasetName": "NTAFlowStreamRaw",
		"index_length": "32",
		"sequence_number": "53894",
		"ipv6_src_mask": "0",
		"ipv6_flow_label": "0",
		"last_switched": "304710751",
		"input_snmp": "18",
		"unix_secs": "1600642980",
		"sys_up_time": "304711635",
		"ipv6_next_hop": "0:0:0:0:0:0:0:0",
		"dev_ip": "10.136.193.161",
		"ipv6_dst_mask": "0",
		"source_id": "50331905",
		"metricGroupId": "NetStream",
		"sampling_interval": "1",
		"flowType": "Original"
	},
	"templateId": "1599"
}
```

#### VXLAN（28个字段）

```json
{
	"rawData": {
		"in_pkts": "1",
		"first_switched": "304719295",
		"l4_src_port": "55165",
		"templateId": "8007",
		"sampling_algorithm": "2",
		"in_bytes": "56",
		"protocol": "6",
		"neMetricGroup": "10.136.193.161_8007",
		"res_id": "bbf73867-0bd1-4e45-9ca3-5f7de0e8afa1",
		"l4_dst_port": "33333",
		"direction": "1",
		"output_snmp": "11",
		"time_id": "1600642964238",
		"ipv4_dst_addr": "111.111.111.5",
		"datasetName": "NTAFlowStreamRaw",
		"layer2_segment_id": "72057594037933159",
		"index_length": "32",
		"ipv4_src_addr": "222.222.222.215",
		"sequence_number": "15562968",
		"last_switched": "304719295",
		"input_snmp": "0",
		"unix_secs": "1600642993",
		"sys_up_time": "304724861",
		"dev_ip": "10.136.193.161",
		"source_id": "67109121",
		"metricGroupId": "NetStream",
		"sampling_interval": "1",
		"flowType": "Flexible"
	},
	"templateId": "8007"
}
```

### 1.2.3 S设备上报数据

#### IPv4（38个字段）

```json
{
	"rawData": {
		"dst_as": "0",
		"forwarding_status": "0",
		"in_pkts": "2",
		"first_switched": "3308495065",
		"ipv4_next_hop": "0.0.0.0",
		"l4_src_port": "5247",
		"templateId": "1315",
		"src_vlan": "0",
		"in_bytes": "160",
		"protocol": "17",
		"neMetricGroup": "10.137.67.82_1315",
		"tcp_flags": "0",
		"res_id": "37a2cc6c-93e9-494c-99f5-95b57906c098",
		"dst_vlan": "0",
		"l4_dst_port": "59133",
		"src_as": "0",
		"responder_octets": "0",
		"direction": "0",
		"output_snmp": "105",
		"field_210": "0",
		"time_id": "1600085867083",
		"dst_mask": "0",
		"ipv4_dst_addr": "10.20.30.244",
		"src_tos": "224",
		"datasetName": "NTAFlowStreamRaw",
		"src_mask": "0",
		"index_length": "32",
		"ipv4_src_addr": "10.20.30.1",
		"sequence_number": "393215",
		"last_switched": "3308496064",
		"input_snmp": "58",
		"unix_secs": "1600085942",
		"sys_up_time": "3308496821",
		"dev_ip": "10.137.67.82",
		"source_id": "1",
		"metricGroupId": "NetStream",
		"bgp_ipv4_next_hop": "0.0.0.0",
		"flowType": "Original"
	},
	"templateId": "1315"
}
```

## 1.3 NtaFlowDeserializer解析NTAFlowStreamRaw

```
NtaFlowDeserializer类路径
pipeline/src/main/java/com.huawei.fabricinsight.ntaservice.pipeline.parser.NtaFlowDeserializer.java

V9报文头：20字节

# 通过 nta_flow_details_etl.json 配置清洗入口为NtaFlowDeserializer
deployment/src/main/release/pub/pm/fabric/ETLConfig/nta_flow_details_etl.json

输入Topic：NTAFlowStreamRaw
输出Topic：NTAFlowDetails
```

#### TQL

```
1、device 表和 port 表先通过 ne_dn 字段关联生成临时表d
2、临时表d 和 NTAFlowStreamRaw 通过 IP 和接口关联生成 NTAFlowDetails Topic中的数据
```

```sql
SELECT 
a.timestamp AS ts_s,d.ne_dn AS device_dn,d.ne_ip AS device_ip,d.ne_name AS device_name,d.fabric_id AS fabric_id,concat(d.ne_ip,'|',d.if_name) AS if_uniq_id,a.if_index AS if_index,d.if_name AS if_name, d.if_alias AS if_alias,a.direction AS direction,a.src_addr AS src_addr,a.dst_addr AS dst_addr,a.l4_src_port AS src_port,a.l4_dst_port AS dst_port,a.vni AS vni,a.conv_id AS conv_id,a.protocol AS protocol_id,a.flow_source AS flow_source,a.in_bytes*a.sampling_interval AS flow_bytes,a.in_bytes*a.sampling_interval/1024/1024 AS flow_mb_bytes,a.in_pkts*a.sampling_interval AS flow_packet 
FROM NTAFlowStreamRaw_output a  
LEFT JOIN 
( SELECT b.ne_dn AS ne_dn, c.ne_ip AS ne_ip, b.ne_name AS ne_name,c.fabric_id AS fabric_id, b.if_index AS if_index, b.if_name AS if_name, b.if_alias AS if_alias 
 FROM 
 uniresource_port_output b 
 LEFT JOIN 
 uniresource_device_output c 
 ON 
 b.ne_dn = c.ne_dn ) d 
ON 
a.dev_ip=d.ne_ip 
AND 
a.if_index=d.if_index 
where 
d.ne_ip is not null and d.if_index is not null
```

```
Fabric
设备表(6个字段)
接口表(16个字段)

Campus
设备表(25个字段)
接口表(16个字段)
```

```sql

```



## 1.4 NTAFlowDetails 

```
find /srv/BigData -name *NTAFlowDetails*
```

#### Topic数据（27个字段）

```json
{
	"tenant_id": null,
	"fabric_id": "94dee411-4eb1-46d5-9dea-1914bf2ddde4",
	"ts_s": 1600631362000,
	"normal_sess_cnt": null,
	"if_name": "10GE1/0/1",
	"device_name": "pod13_Serverleaf",
	"if_alias": "jiekou1",
	"device_dn": "bbf73867-0bd1-4e45-9ca3-5f7de0e8afa1",
	"max_conn_latency": null,
	"dst_addr": "111.111.111.5",
	"if_index": "11",
	"flow_bytes": 56.0,
	"direction": "1",
	"abn_sess_cnt": null,
	"avg_conn_latency": null,
	"flow_mb_bytes": 5.340576171875E-5,
	"src_addr": "222.222.222.249",
	"if_uniq_id": "10.136.193.161|10GE1/0/1",
	"protocol_id": "6",
	"device_ip": "10.136.193.161",
	"src_port": "40199",
	"vni": "5223",
	"dst_port": "33333",
	"conv_id": "222.222.222.249|111.111.111.5",
	"flow_packet": 1.0,
	"protocol_name": null,
	"flow_source": "0"
}
```

```
输出Topic NTAFlowDetails再按照不同的Dataset(detail、if、conv、prot、vni)进行聚合

之后通过自定义分流插件分流出主机维度
```

## 1.5 SplitStreamPlugin对NTAFlowDetail进行分流

```
find /srv/BigData -name *NTAFlowHost*
```

### Topic数据(28个字段)

```json
{
	"tenant_id": null,
	"fabric_id": "13862f70-d3e2-4010-bbae-e848791d995e",
	"ts_s": 1600738871000,
	"host_port": "0",
	"normal_sess_cnt": null,
	"if_name": "25GE1/0/40",
	"host_long": "176734506",
	"host_type": "1",
	"device_name": "Serverleaf_193.162",
	"if_alias": null,
	"device_dn": "08090a60-9eab-49e9-bd1c-f427d86ff8b3",
	"max_conn_latency": null,
	"app_id": "1",
	"host_addr": "10.136.193.42",
	"flow_bytes": 84.0,
	"direction": "1",
	"abn_sess_cnt": null,
	"avg_conn_latency": null,
	"if_uniq_id": "10.136.193.162|25GE1/0/40",
	"protocol_id": "1",
	"device_ip": "10.136.193.162",
	"vni": null,
	"app_name": null,
	"conv_id": "10.136.193.162|10.136.193.42",
	"flow_packet": 1.0,
	"flow_source": "0",
	"host_is_ipv4": "1",
	"protocol_name": null
}
```

## 1.6 Redis集群名字

```
Fabric：FABRIC_DC_SERVICE
Campus：campus_issues_redis
```



# 2 资源表（Carbon/RDB）

## 2.1 下载并安装Spark客户端

```
1、登录FI面下载Spark客户端
/tmp/FusionInsight-Client/FusionInsight_Cluster_1_Spark2x_Client.tar

2、解压客户端
cd /tmp/FusionInsight-Client/
tar -xvf FusionInsight_Cluster_1_Spark2x_Client.tar

3、校验安装包
sha256sum -c FusionInsight_Cluster_1_Spark2x_ClientConfig.tar.sha256

4、解压获取安装文件
tar -xvf FusionInsight_Cluster_1_Spark2x_ClientConfig.tar

5、安装客户端
cd FusionInsight_Cluster_1_Spark2x_ClientConfig
mkdir /opt/sparkclient
./install.sh /opt/sparkclient
```

## 2.2 下载odaeuser用户凭证

```
1、FI面 -> 系统 -> odaeuser(机机) -> 更多 -> 下载认证凭证

2、解压下载的文件，将user.keytab上传到spark客户端安装目录

3、执行下面联调命令
source bigdata_env
kinit -kt user.keytab odaeuser
```

## 2.3 查看Carbon中数据

参考链接：http://3ms.huawei.com/km/blogs/details/5858133

```
1、执行启动脚本
cd /opt/sparkclient/
spark-shell --master yarn --deploy-mode client --executor-memory 1G --driver-cores 1 --executor-cores 1 --driver-memory 1G --num-executors 1

2、创建上下文
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.CarbonSession._
val carbon = SparkSession.builder().config(sc.getConf).getOrCreateCarbonSession("hdfs://hacluster/srv/bigdata/hive/warehouse/carbon.store")

3、查询当前carbon中的表
carbon.sql(s"SHOW TABLES IN _DEFAULT").collect().foreach(println)

4、查看表结构
carbon.sql("desc _DEFAULT.uniresource_port_e3d7b62181610bdf3ae0efe897b1f9be_21").collect().foreach(println)

4、统计表中的数据条目
carbon.sql(s"select count(*) from _DEFAULT.iresource_device_a0e8765c0b4dbf64aa51747c0980a1d5_44").collect().foreach(println)
carbon.sql(s"select count(*) from _DEFAULT.uniresource_port_e3d7b62181610bdf3ae0efe897b1f9be_98").collect().foreach(println)

5、查看表中的条目
carbon.sql(s"select * from _DEFAULT.uniresource_port_e3d7b62181610bdf3ae0efe897b1f9be_27").collect().foreach(println)
carbon.sql(s"select * from _DEFAULT.uniresource_port_e3d7b62181610bdf3ae0efe897b1f9be_98").collect().foreach(println)
```

### FabricInsight

#### uniresource_device(6个字段)

```
[res_id,string,null]
[ne_dn,string,null]
[name,string,null]
[ne_ip,string,null]
[fabric_id,string,null]
[mac,string,null]
```

#### uniresource_port(16个字段)

```
[res_id,string,null]
[ne_dn,string,null]
[if_index,string,null]
[if_name,string,null]
[slot_id,string,null]
[slot_name,string,null]
[ne_ip,string,null]
[fabric_id,string,null]
[if_admin_status,string,null]
[if_oper_status,string,null]
[is_has_link,string,null]
[ne_name,string,null]
[if_type,string,null]
[if_speed,string,null]
[if_high_speed,string,null]
[if_alias,string,null]
```

### CampusInsight

#### uniresource_device(25个字段)

```
[resid,string,null]
[name,string,null]
[alias,string,null]
[mac,string,null]
[ipaddress,string,null]
[category,string,null]
[tenantid,string,null]
[typeid,string,null]
[version,string,null]
[directregion,string,null]
[positiontype,string,null]
[siteid,string,null]
[regionlevelone,string,null]
[regionleveltwo,string,null]
[regionlevelthree,string,null]
[regionlevelfour,string,null]
[regionlevelfive,string,null]
[regionlevelsix,string,null]
[regionlevelseven,string,null]
[regionleveleight,string,null]
[parentresid,string,null]
[acname,string,null]
[publicarea,bigint,null]
[supportbusiness,bigint,null]
[sitename,string,null]
```

#### uniresource_port(16个字段)

```
[res_id,string,null]
[ne_dn,string,null]
[if_index,string,null]
[if_name,string,null]
[slot_id,string,null]
[slot_name,string,null]
[ne_ip,string,null]
[fabric_id,string,null]
[if_admin_status,string,null]
[if_oper_status,string,null]
[is_has_link,string,null]
[ne_name,string,null]
[if_type,string,null]
[if_speed,string,null]
[if_high_speed,string,null]
[if_alias,string,null]
```



### resource_device表示例

```
[red_id, ne_dn, name, ne_ip, fabric_id, mac]

[ed6a2d53-d2ab-4134-9708-9bfa265e547c,ed6a2d53-d2ab-4134-9708-9bfa265e547c,spine189,10.136.194.189,94dee411-4eb1-46d5-9dea-1914bf2ddde4,6C-EB-B6-51-37-C1]
[ff5dfe6f-8a67-451f-b5fd-24992f632bcb,ff5dfe6f-8a67-451f-b5fd-24992f632bcb,serverleaf190,10.136.194.190,94dee411-4eb1-46d5-9dea-1914bf2ddde4,6C-EB-B6-51-36-01]
```

### resource_port表示例

```
[res_id, ne_dn, if_index, if_name, slot_id, slot_name, ne_ip, fabric_id, if_admin_status, if_oper_status, is_has_link, ne_name,  if_Type, if_Speed, if_High_Speed, if_alias]

[d12a3884-f9c5-11ea-8af9-8ce5ef807d21,27b428d4-179c-4db0-9ace-f2e2daf43de8,2,Sip5/0/0,17104897,CE-MPUA 5,10.136.242.163,94dee411-4eb1-46d5-9dea-1914bf2ddde4,active,inactive,0,POD6-Spine4,1,1000000000,1000,null]
[d12a3885-f9c5-11ea-8af9-8ce5ef807d21,27b428d4-179c-4db0-9ace-f2e2daf43de8,3,Sip5/0/1,17104897,CE-MPUA 5,10.136.242.163,94dee411-4eb1-46d5-9dea-1914bf2ddde4,active,inactive,0,POD6-Spine4,1,1000000000,1000,null]
```

## 2.4 查看HDFS数据

```
1、创建hdfsclient人机用户
FI面 -> 系统 -> 权限 -> 用户 -> 添加用户
用户组：hadoop、hadoopmanager、supergroup
主组：supergroup
角色：HDFSOper、System_administrator

2、用hdfsclient用户登陆FI，进入hdfs文件系统页面
Utilities -> Browse the file system

3、查询Carbon相关元数据信息
/srv/bigdata/datasets/_DEFAULT/uniresource_device/number
uniresource_device			carbon对应的设备、AP的元数据
uniresource_device_oper		设备、AP对应的中间Dataset
uniresource_optical			carbon光模块对应的元数据
uniresource_optical_oper	光模块对应的中间dataset
uniresource_port			carbon端口对应的元数据
uniresource_port_oper		端口对应的中间dataset
```

# 3 ODAE

```
NSCFRTCatalogService：元数据模型包管理服务，为了方便其他服务自管理模型包，提供pub机制下模型包上传以及升级的功能。
包放置规则为：/pub/nscf_rtcatalog/*/*.zip
如网流的分流插件：fabricinsight_pipeline_nta_splitstream.zip

应用抽象 -> Pipeline
	处理模式
		batch  	输入：file、stream、olap、rdb
			input -> Spark Engine -> output
		stream	实时数据切分成多个batch(spark stream) 输入：kafka、hdfs、rdb
			input -> spark stream -> spark engine -> output
	执行引擎
		OLAPLoader：OLAPLoader是将批量数据流或者实时数据流加载到OLAP仓库的任务中，插件所用的执行引擎。负责将三方数据接入ODAE。
			异构数据 -> AP -> ODAE
		SparkBatch：SparkBatch是在计算服务（ODAESparkDispService）基于批量数据处理任务中，插件所用的执行引擎。
			input -> Spark Engine -> output
		SparkStream：SparkStream是在计算服务（ODAESparkDispService）基于实时数据流的数据处理任务中，插件所用的执行引擎。
			input -> Spark Stream -> output
		AP：AP是在数据接入服务（ODAEAccessPointService）的数据接入任务中，插件所用的执行引擎。负责将三方数据接入ODAE。
			异构数据 -> AP -> ODAE
	执行步骤
		Sink Stage：Sink定义了将数据从执行引擎保存到存储系统的插件（Plugin），目前支持向Kafka，HDFS，RDB及OLAP（Druid）写入数据，其中RDB包括高斯，			MYSQL两种数据库。
		Source Stage：Source插件主要功能是将数据从存储系统加载到执行引擎，目前支持从HDFS，Kafka，RDB，Druid四种存储系统中读取数据，其中RDB包括高斯，  		   MYSQL两种数据库。
		Function Stage：Function也是实现计算的执行步骤，其实际是给用户提供了可扩展的计算插件。Rows->Rows 或 Row->Row
		Analytics Stage：Analytics实际真正负责计算步骤，其定义的是执行引擎中的多行数据计算的逻辑（Rows->Rows）。
```



# 4 网流Stage与Connections

```
Stages
    NTAFlowDetails_input_stage(source)
    	plugin
    		dataset:NTAFlowDetails
    	inputs:[]
    	outputs；xxx
    KPI_SPLIT_STREAM(analytics)
    	inputs
    		name: NTAFlowDetails
    	outputs:
    		port:NTAFlowHost
    NTAFlowHost_output_stage(sink)
    	plugin
    		dataset:NTAFlowHost
    	inputs:
    		name:input
    
    OlapLoaderStreamSource_NTAFlowHost_min(source)
    OlapLoaderSink_NTAFlowHost_min(sink)
    
    OlapLoaderStreamSource_NTAFlowHost_hour(source)
    OlapLoaderSink_NTAFlowHost_hour(sink)

Connections
	NTAFlowDetails_input_stage(source) -> KPI_SPLIT_STREAM(analytics) -> NTAFlowHost_output_stage(sink)
	
	Kafka -> Druid
	OlapLoaderStreamSource_NTAFlowHost_min(source) -> OlapLoaderSink_NTAFlowHost_min(sink)
	OlapLoaderStreamSource_NTAFlowHost_hour(source) -> OlapLoaderSink_NTAFlowHost_hour(sink)
```

# 5 Druid

## 5.1 Druid存储

```
/opt/oss/share/FabricInsight/ODAEOLAPAgentService/druid/
/opt/oss/rtsp/ODAEDruidClient-21.30.200/druid
# 一个Datasouce包含多个Segment，一个Segment包含多个分区
# 某一个Segment的分区会随机存储在不同节点的druid1~druid12中
# Druid入库日志
HDFS上的 /srv/bigdata/druid/indexing-logs/

/srv/druid/var/
		# node189
			druid
				pids
					broker.pid
					coordinator.pid
					historical.pid
					middleManager.pid
					overlord.pid
				task
			druid1
				{dataSource}/{intervalStart}-{intervalEnd}/{segmentCreatedTime}/{partitionNum}
				例:fi_nta_flow_host_1h/2020-09-15T10:00:00.000Z_2020-09-15T11:00:00.000Z/2020-09-15T09:58:00.118Z/1
					00000.smoosh
					factory.json
					meta.smoosh	# 记录Segment的元数据信息
					version.bin	# Segment内部版本号
			druid2
            druid3
            druid4
            druid5
            druid6
            druid7
            druid8
            druid9
			druid10
			druid11
			druid12
			tmp # 本地历史的持久化存储
		# node190
			druid
				pids
					broker.pid
					coordinator.pid
					historical.pid
					middleManager.pid
					overlord.pid
				task
			druid1
			druid2
            druid3
            druid4
            druid5
            druid6
            druid7
            druid8
            druid9
			druid10
			druid11
			druid12
			tmp # 本地历史的持久化存储
		# node191
			druid
				pids
					broker.pid
					coordinator.pid
					historical.pid
					middleManager.pid
					overlord.pid
				task
			druid1
			druid2
            druid3
            druid4
            druid5
            druid6
            	{dataSource}/{intervalStart}-{intervalEnd}/{segmentCreatedTime}/{partitionNum}
            	例：fi_nta_flow_host_1h/2020-09-15T10:00:00.000Z_2020-09-15T11:00:00.000Z/2020-09-15T09:58:00.118Z/0
            druid7
            druid8
            druid9
			druid10
			druid11
			druid12
			tmp # 本地历史的持久化存储
```

## 5.2 Druid查询

### 查询SQL

```json
curl -X POST 'http://10.136.193.78:26201/druid/v2/?pretty' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @query.json
```

### 网流表

```json
fi_nta_flow_host_1h_top10000
fi_nta_flow_host_1m
ODAEDATASET__DEFAULT_fi_nta_flow_conv_1h__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_conv_1h_top10000__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_conv_1m__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_details_1h__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_details_1m__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_if_1h__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_if_1m__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_prot_1h__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_prot_1m__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_vni_1h__DEFAULT
ODAEDATASET__DEFAULT_fi_nta_flow_vni_1m__DEFAULT
```

### Timeseries

```json
{
  "queryType": "timeseries",
  "dataSource": "ODAEDATASET__DEFAULT_fi_nta_flow_if_1h__DEFAULT",
  "granularity": "hour",
  "filter": {
      "type": "selector", "dimension": "device_ip", "value": "10.136.193.163" 
  },
  "aggregations": [
    { "type" : "count", "name" : "total" }
  ],
  "intervals": [ "2020-09-25T00:00:00.000/2020-09-27T00:00:00.000" ]
}
```

```json
{
  "queryType": "timeseries",
  "dataSource": "ODAEDATASET__DEFAULT_fi_nta_flow_if_1h__DEFAULT",
  "granularity": "hour",
  "filter": {
      "type": "selector", "dimension": "device_ip", "value": "10.136.193.163" 
  },
  "aggregations": [
    { "type" : "longSum", "name" : "total", "fieldName" : "flow_bytes" }
  ],
  "intervals": [ "2020-09-25T00:00:00.000/2020-09-27T00:00:00.000" ]
}
```

### TopN

```json
{
  "queryType": "topN",
  "dataSource": "ODAEDATASET__DEFAULT_fi_nta_flow_if_1h__DEFAULT",
  "dimension": "if_name",
  "threshold": 5,
  "metric": "total",
  "granularity": "all",
  "aggregations": [
    { "type" : "longSum", "name" : "total", "fieldName" : "flow_bytes" }
  ],
  "intervals": [
    "2020-09-25T00:00:00.000/2020-09-27T00:00:00.000"
  ]
}
```

# 6 集群免密登录

```
ssh-keygen -t rsa
ssh-copy-id root@192.168.11.130
scp * root@192.168.11.131:/opt/bigdata/
jps
/opt/bigdata/apache-zookeeper-3.6.2-bin/
/opt/bigdata/kafka_2.12-2.6.0

./zkServer.sh [start|stop|status]cd 
./kafka-server-start.sh -daemon ../config/server.properties
./kafka-server-stop.sh

# 创建Topic
./kafka-topics.sh --create --zookeeper 192.168.11.129:2181 --replication-factor 2 --partitions 1 --topic abc

# 查看创建是否成功
./kafka-topics.sh --list --zookeeper 192.168.11.129:2181

# 创建发布者
./kafka-console-producer.sh --broker-list 192.168.11.129:9092 --topic abc

# 创建订阅者
./kafka-console-consumer.sh --bootstrap-server 192.168.11.129:9092 --topic abc --from-beginning
```

```
表名	关系表	fi_nta_flow_details_1m、fi_nta_flow_details_1h			
序号	列	名称	类型	说明	
1	__time	时间	TIMESTAMP	时间戳	
2	count	计数	BIGINT	汇聚计数	
3	device_ip	设备IP	VARCHAR	来自关联RDB	
4	device_name	设备名称	VARCHAR	来自关联RDB	
5	fabric_id	fabric ID	VARCHAR	来自关联RDB	
6	tenant_id	租户ID	VARCHAR	空，预留	
7	if_name	接口名称	VARCHAR	来自关联RDB	
8	if_alias	接口别名	VARCHAR	来自关联RDB	
9	if_uniq_id	接口唯一ID	VARCHAR	dev_ip+ "|" + if_name	
10	direction	流方向	VARCHAR	0：入方向、1：出方向	
11	src_addr	源地址	VARCHAR	来自网流报文	
12	dst_addr	目的地址	VARCHAR	来自网流报文	
13	dst_port	目的端口	VARCHAR	来自网流报文	
14	vni	vni	VARCHAR	来自网流报文layer2_segment_id	
15	conv_id	会话ID	VARCHAR	来自网流报文 src_addr + "|" + dst_addr	
16	protocol_id	协议ID	VARCHAR	来自网流报文	
17	flow_source	P5流标记	VARCHAR	P5流标记  0：原始流、1：P5流	
18	flow_bytes	流量	DOUBLE	来自网流报文in_bytes * .sampling_interval，单位B	
19	flow_packet	数据包	DOUBLE	来自网流报文in_pkts * sampling_interval	
20	normal_sess_cnt	P5流指标	DOUBLE	P5流指标	
21	abn_sess_cnt	P5流指标	DOUBLE	P5流指标	
22	avg_conn_latency	P5流指标	DOUBLE	P5流指标	
23	max_conn_latency	P5流指标	DOUBLE	P5流指标				
```

```
表名	接口表	fi_nta_flow_if_1m、fi_nta_flow_if_1h			
序号	列	名称	类型	说明	
1	__time	时间	TIMESTAMP	时间戳	
2	count	计数	BIGINT	汇聚计数	
3	device_ip	设备IP	VARCHAR	来自关联RDB	
4	device_name	设备名称	VARCHAR	来自关联RDB	
5	fabric_id	fabric ID	VARCHAR	来自关联RDB	
6	tenant_id	租户ID	VARCHAR	空，预留	
7	if_name	接口名称	VARCHAR	来自关联RDB	
8	if_alias	接口别名	VARCHAR	来自关联RDB	
9	if_uniq_id	接口唯一ID	VARCHAR	dev_ip+ "|" + if_name	
10	direction	流方向	VARCHAR	0：入方向、1：出方向	
11	flow_source	P5流标记	VARCHAR	P5流标记  0：原始流、1：P5流	
12	flow_bytes	流量	DOUBLE	来自网流报文in_bytes * .sampling_interval，单位B	
13	flow_packet	数据包	DOUBLE	来自网流报文in_pkts * sampling_interval	
14	normal_sess_cnt	P5流指标	DOUBLE	P5流指标	
15	abn_sess_cnt	P5流指标	DOUBLE	P5流指标	
16	avg_conn_latency	P5流指标	DOUBLE	P5流指标	
17	max_conn_latency	P5流指标	DOUBLE	P5流指标	
18	flow_mb_bytes	流量MB	DOUBLE	flow_bytes 单位MB	
```

```
表名	会话表	fi_nta_flow_conv_1m、fi_nta_flow_conv_1h			
序号	列	名称	类型	说明	
1	__time	时间	TIMESTAMP	时间戳	
2	count	计数	BIGINT	汇聚计数	
3	tenant_id	租户ID	VARCHAR	空，预留	
4	src_addr	源地址	VARCHAR	来自网流报文	
5	dst_addr	目的地址	VARCHAR	来自网流报文	
6	dst_port	目的端口	VARCHAR	来自网流报文	
7	conv_id	会话ID	VARCHAR	来自网流报文 src_addr + "|" + dst_addr	
8	flow_source	P5流标记	VARCHAR	P5流标记  0：原始流、1：P5流	
9	flow_bytes	流量	DOUBLE	来自网流报文in_bytes * .sampling_interval，单位B	
10	flow_packet	数据包	DOUBLE	来自网流报文in_pkts * sampling_interval	
11	normal_sess_cnt	P5流指标	DOUBLE	P5流指标	
12	abn_sess_cnt	P5流指标	DOUBLE	P5流指标	
13	avg_conn_latency	P5流指标	DOUBLE	P5流指标	
14	max_conn_latency	P5流指标	DOUBLE	P5流指标	
```

```
表名	协议表	fi_nta_flow_prot_1m、fi_nta_flow_prot_1h			
序号	列	名称	类型	说明	
1	__time	时间	TIMESTAMP	时间戳	
2	count	计数	BIGINT	汇聚计数	
3	tenant_id	租户ID	VARCHAR	空，预留	
4	protocol_id	协议ID	VARCHAR	来自网流报文	
5	flow_source	P5流标记	VARCHAR	P5流标记  0：原始流、1：P5流	
6	flow_bytes	流量	DOUBLE	来自网流报文in_bytes * .sampling_interval，单位B	
7	flow_packet	数据包	DOUBLE	来自网流报文in_pkts * sampling_interval	
8	normal_sess_cnt	P5流指标	DOUBLE	P5流指标	
9	abn_sess_cnt	P5流指标	DOUBLE	P5流指标	
10	avg_conn_latency	P5流指标	DOUBLE	P5流指标	
11	max_conn_latency	P5流指标	DOUBLE	P5流指标	
```

```
表名	VNI表	fi_nta_flow_vni_1m、fi_nta_flow_vni_1h			
序号	列	名称	类型	说明	
1	__time	时间	TIMESTAMP	时间戳	
2	count	计数	BIGINT	汇聚计数	
3	tenant_id	租户ID	VARCHAR	空，预留	
4	vni	vni	VARCHAR	来自网流报文layer2_segment_id	
5	flow_source	P5流标记	VARCHAR	P5流标记  0：原始流、1：P5流	
6	flow_bytes	流量	DOUBLE	来自网流报文in_bytes * .sampling_interval，单位B	
7	flow_packet	数据包	DOUBLE	来自网流报文in_pkts * sampling_interval	
8	normal_sess_cnt	P5流指标	DOUBLE	P5流指标	
9	abn_sess_cnt	P5流指标	DOUBLE	P5流指标	
10	avg_conn_latency	P5流指标	DOUBLE	P5流指标	
11	max_conn_latency	P5流指标	DOUBLE	P5流指标	
```

```
表名+B114:G140	主机表	fi_nta_flow_host_1m、fi_nta_flow_host_1h			
序号	列	名称	类型	说明	
1	__time	时间	TIMESTAMP	时间戳	
2	count	计数	BIGINT	汇聚计数	
3	device_ip	设备IP	VARCHAR	来自关联RDB	
4	device_name	设备名称	VARCHAR	来自关联RDB	
5	fabric_id	fabric ID	VARCHAR	来自关联RDB	
6	tenant_id	租户ID	VARCHAR	空，预留	
7	if_name	接口名称	VARCHAR	来自关联RDB	
8	if_alias	接口别名	VARCHAR	来自关联RDB	
9	if_uniq_id	接口唯一ID	VARCHAR	dev_ip+ "|" + if_name	
10	direction	流方向	VARCHAR	0：入方向、1：出方向	
11	host_addr	主机地址	VARCHAR	主机类型 来自网流报文src_addr和dst_addr	
12	host_long	主机地址	VARCHAR	ipToLong	
13	host_is_ipv4	主机端口	VARCHAR	1：IPV4、0：IPV6	
14	host_type	类型	VARCHAR	0: src, 1: dst	
15	vni	vni	VARCHAR	来自网流报文layer2_segment_id	
16	conv_id	会话ID	VARCHAR	来自网流报文 src_addr + "|" + dst_addr	
17	protocol_id	协议ID	VARCHAR	来自网流报文	
18	app_id	应用ID	VARCHAR	NTA消费时补齐的字段	
19	flow_source	P5流标记	VARCHAR	P5流标记  0：原始流、1：P5流	
20	flow_bytes	流量	DOUBLE	来自网流报文in_bytes * .sampling_interval，单位B	
21	flow_packet	数据包	DOUBLE	来自网流报文in_pkts * sampling_interval	
22	normal_sess_cnt	P5流指标	DOUBLE	P5流指标	
23	abn_sess_cnt	P5流指标	DOUBLE	P5流指标	
24	avg_conn_latency	P5流指标	DOUBLE	P5流指标	
25	max_conn_latency	P5流指标	DOUBLE	P5流指标	
```

