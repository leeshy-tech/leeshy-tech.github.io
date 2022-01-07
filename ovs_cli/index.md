# OVS命令笔记

## 查看信息 
查看openvswitch的状态：`ovs-vsctl show`  
查看openvswitch中的所有网桥：`ovs-vsctl list-br`   
查看网桥的信息：`ovs-ofctl show br0`     
查看网桥中的所有端口：`ovs-vsctl list-ports br0`   
查看网桥各端口状态：`ovs-ofctl dump-ports br0`   
查看网桥上的流表：`ovs-ofctl dump-flows br0`    
查看网桥故障模式：`ovs-vsctl get-fail-mode br0`     
查看网桥是否开启stp协议：`ovs-vsctl get bridge br0 stp_enable`   
查看网桥所有配置信息：`ovs-vsctl list bridge br0`    
查看端口所有特性信息：`ovs-vsctl list port br0 port1 `
## 网桥操作
添加网桥：`ovs-vsctl add-br br0`    
删除网桥：`ovs-vsctl del-br br0`    
将网桥连接到控制器：`ovs-vsctl set-controller br0 tcp:<controller IP>:<port>`   
设置网桥的故障模式：`ovs-vsctl set-fail-mode br0 <secure or standalone> `   
给网桥开启stp协议：`ovs-vsctl set bridge br0 stp_enable=true`
## 端口操作
增加端口：`ovs-vsctl add-port br0 port1`  
删除端口：`ovs-vsctl del-port br0 port1`    
设置端口号：`ovs-vsctl set Interface port1 ofport_request=<number>`    
设置端口类型：`ovs-vsctl set Interface p4 type=<type>`    
| 端口类型      | 描述 |
| ----------- | ----------- |
|Normal|普通端口|
|Internal|这个端口相当于一个虚拟网卡|
|Patch|用于连接两个网桥的端口|
|Tunne|隧道端口是一种虚拟端口，支持使用gre或vxlan等隧道技术与位于网络上其他位置的远程端口通讯。|

## 网卡操作    
将物理网卡挂接到网桥：`ovs-vsctl add-port br0 eth0`  
列出所有挂接到网卡的网桥：`ovs-vsctl port-to-br eth0`   
删除网桥上已经挂接的网卡：`ovs-vsctl del-port br0 eth0` 
## 流表操作
添加流表：`ovs-ofctl add-flow br0 <match>,<action>`      
删除流表：`ovs-ofctl add-flow br0 <match>`
### 流表动作 action
从端口转发：`actions=output:<number>`     
丢弃数据包：`actions=drop`  
广播：`actions=NORMAL`
### 匹配项 match
如有多个匹配项，之间用逗号隔开。  
| 匹配项      | 描述 |
| ----------- | ----------- |
| in_port=port      | 网桥端口号       |
| dl_vlan=vlan   | 数据包的 VLAN Tag 值 |
|dl_vlan_pcp=priority|VLAN 优先级，改值取值区间为[0-7]。数字越大，优先级越高。|
|dl_src=<MAC>,dl_dst=<MAC>|匹配源或者目标的 MAC 地址|
|dl_type=ethertype | 匹配以太网协议类型 ,IPv4:0x0800,IPv6:0x086dd,ARP:0x0806|
|nw_src=<IP>,nw_dst=<IP>| 匹配源或目的IP，前提要设定dl_type |
|table=number| 指定流表编号 |
|reg<idx>=value[/mask]| 网桥中寄存器的值 |
|tp_src=number,tp_dst=number|TCP协议源或目的端口|

