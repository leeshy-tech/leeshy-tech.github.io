# SDN自学习交换机工作原理分析

## 自学习交换机
交换机上电启动时，帧交换表为空，随着各主机间的通信，通过自学习算法自动逐渐建立帧交换表。帧交换表是mac地址和端口号的对应关系，交换机端口是固定的，连接的设备是可变的，所以只有建立起帧交换表之后才能明确某目的mac地址的数据包该向哪个端口转发。  

SDN的自学习交换机跟传统交换机不一样的点有： 
1. 帧交换表是由控制器来维护的，比如ryu里的数据结构：mac_to_port[dpid][mac] = port，控制器维护所有与之相连的交换机的帧交换表。   
2. 自学习的过程中可能会有流表的变化。   

## ping命令过程
ping命令使用ICMP传输协议，用于测试两主机之间的网络连通性。  
局域网ping命令的执行过程：  
网络模型为：h1  ----  s1  ----  h2，两台主机连接到同一个交换机。
假设h1 ping h2  
- 首先h1查询自己的mac地址表，若有h2对应的mac地址，就直接设为目的mac，否则发送一个ARP广播包，目的mac设为ff:ff:ff:ff。
- 交换机收到ARP后，如果交换机中有h2对应的mac地址，就返回给h1，否则向所有端口发送ARP广播。
- h2收到ARP报文后，返回ARP应答报文，告诉h1自己的mac地址，同时学习到h1的mac地址。
- h1收到应答后，学习到h2的mac地址，封装ICMP报文给h2。
- h2收到报文后应答，ping完成。
## 实验分析
### 实验内容
- 通过ryu控制器实现一个SDN自学习交换机simple_switch.py，分析博客：[ryu源码解读——simple_switch.py](http://localhost:1313/simple_switch_note/)。
- 使用OVS搭建网络模型：h1  ----  s1  ----  h2，将s1连接到控制器。
- h1 ping h2，观察PacketIn消息。
- 查看s1的流表变化。  
  
这里不用mininet平台搭建网络的原因是：mininet交换机会定时检查与控制器的连接，导致出现很多的冗余PacketIn消息，不利于观察。
### 实验结果
![PacketIn](/image/simple_switch_analyse/packetin.jpg)  
当h1 ping h2时，出现3条PacketIn消息，之后再ping，不再出现PacketIn消息。   
使用`ovs-ofctl dump-flows s1`命令观察流表，结果如下：     
![流表](/image/simple_switch_analyse/流表.jpg)  
执行完ping命令后多了两条流表。
### 流程分析
日志的输出格式是：packet in <交换机标号> <源mac> <目的mac> <输入端口>   
h1：mac地址mac1 = 3e:3b:50:01:23:e6，连接到s1的1号端口。    
h2：mac地址mac2 = d6:07:97:03:df:5b，连接到s1的2号端口。    
1. h1 ping h2，h1发送一个目的mac为ff:ff:ff:ff的ARP包，s1流表为空，发送PacketIn消息。
2. 控制器学习到`mac1--端口1`，查找不到h2连接的端口，发送PacketOut消息指示s1广播这个ARP包。
3. s1广播，h2收到并学习到h1的mac地址，发送`<src = mac2，dst = mac1>`的ARP应答报文，s1流表为空，发送PacketIn消息。
4. 控制器学习到`mac2--端口2`，查找到mac1的对应端口为1，此时下发一条流表`in_port=2,output:1`，发送PacketOut消息指示s1把这个应答报文转发到端口1。
5. h1收到应答报文，学习到h2的mac地址，发送`<src = mac1，dst = mac2>`的ICMP报文。
6. 此时s1中只有一条流表`in_port=2,output:1`，仍然发送PacketIn消息。
7. 控制器查找到mac2的对应端口为2，下发一条流表`in_port=1,output:2`，发送PacketOut消息指示s1把这个报文转发到端口2。
8. 之后略   

所以，整个过程中一共上传了三次PacketIn消息，分别是`ma1 ff:ff:ff:ff 1`，`mac2 mac1 2`，`mac1 mac2 1`。   
过程结束s1中有两条流表，分别是`in_port=2,src=mac2,dst=mac1,action=output:1`,`in_port=1,src=mac1,dst=mac2,action=output:2`。 
有了这两条流表之后，s1完全按照流表转发，不会产生PacketIn消息和新的流表。
## 结束语
### 总结

要始终用流表的思想看待整个过程，PacketIn消息产生的原因是没有匹配的流表。    
### 参考文献
[Ping过程 详解 ](https://blog.51cto.com/wanicy/335207)  
[ryu源码解读——simple_switch.py](http://localhost:1313/simple_switch_note/)
