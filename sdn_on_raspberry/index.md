# 在树莓派上构建SDWN网络教程

## 导入
在SDN领域的学习，几乎所有的入门实践都是以mininet平台为基础，搭配开源控制器进行实验，但这种实验本质上也只是在一台linux设备上进行SDN网络的仿真罢了，实际应用还是要构建一个实际的物理网络。本文以OVS、ryu控制器和树莓派构建一个SDWN物理网络，SDWN是将SDN对于无线场景的应用，实际区别在于底层网络。    
OVS：OpenvSwitch的简称，他是一种支持OpenFlow协议的软交换机。    
ryu：基于Python开发的SDN开源控制器。
## 准备工作
- 树莓派搭载linux操作系统，有无线网卡，利用`ifconfig`命令查得无线网卡名称。 
- 树莓派之间先组成adhoc网络，参考文章：[在树莓派上搭建ad-hoc网络教程](https://blog.csdn.net/lby0910/article/details/53420459)   
- 在两台树莓派上安装OVS，参考文章：[Open vSwitch系列之二 安装指定版本ovs](https://www.cnblogs.com/goldsunshine/p/10331606.html) 
- 在其中一台树莓派上安装ryu控制器。
## 组网步骤
假设两台树莓派的IP分别为10.0.0.1和10.0.0.2，两台树莓派的无线网卡名均为wlan0，在10.0.0.1上搭载控制器，则网络结构如下图： 
![组网结果结构](/image/SDN_on_RaspberryPi/树莓派组网图示.jpg)   
### 控制节点 IP = 10.0.0.1 
开启控制器（进入ryu/ryu/app/文件夹执行）：
```
ryu-manager simple_switch.py
```
OVS的相关操作需要进入管理员模式：
```
sudo su
```  
开启OVS
```
export PATH=$PATH:/usr/local/share/openvswitch/scripts
ovs-ctl start 
```

创建交换机：
```
ovs-vsctl add-br s1
```
将无线网卡挂接到交换机的一个端口：
```
ovs-vsctl add-port s1 wlan0
```
网卡设置，此部分详解见下方解释：
```
ifconfig wlan0 0
ifconfig s1 10.0.0.1
ifconfig s1 up
```
交换机连接控制器：
```
ovs-vsctl set-controller s1 tcp:10.0.0.1:6653
```
### 普通节点 IP = 10.0.0.2
OVS的相关操作需要进入管理员模式：
```
sudo su
```  
开启OVS
```
export PATH=$PATH:/usr/local/share/openvswitch/scripts
ovs-ctl start 
```
创建交换机：
```
ovs-vsctl add-br s1
```
将无线网卡挂接到交换机的一个端口：
```
ovs-vsctl add-port s1 wlan0
```
网卡设置，此部分详解见下方解释：
```
ifconfig wlan0 0
ifconfig s1 10.0.0.2
ifconfig s1 up
```
交换机连接控制器：
```
ovs-vsctl set-controller s1 tcp:10.0.0.1:6653
```
### 关于网卡设置的解释
对于一般的adhoc网络，主机产生的数据包是由无线网卡进行发送的，当我们使用OVS接管了无线网卡wlan0后，主机产生的数据包已经无法从无线网卡发出了，并且，无线网卡连接到OVS后成为了交换机的一个普通端口，交换机端口只有mac地址没有IP地址，也就是原主机IP失效，所以此时ping该主机时是ping不通的。而OVS的交换机在创建之后带有一个同名的Internal类型端口，它类似于一个虚拟网卡，将原主机的IP转移到该网卡上就能解决这个问题。所以执行了以下三条命令：    
取消wlan0设置的IP地址：`ifconfig wlan0 0`   
设置网卡s1的IP地址为原主机地址：`ifconfig s1 10.0.0.1`  
开启网卡s1：`ifconfig s1 up`    
## 测试
> 如何证明我们成功搭载了SDN网络，而不是之前的adhoc网络呢？       
 
注意我们开启的控制器是simple_switch.py，这个文件是一个实现自学习功能的控制器应用。      
查询两个交换机的流表信息：      
`ovs-ofctl dump-flows s1`       
输出为空，此时交换机中没有流表。    
在控制节点ping普通节点：    
`ping 10.0.0.2`  
发现可以ping通，同时可以在控制器窗口看到PacketIn消息。  
再次查询流表：`ovs-ofctl dump-flows s1`     
发现多了两条流表，说明此时交换机已经明确了两台主机的IP地址所对应的端口号。    
## 应用
模拟控制节点向普通节点分发命令，在两台树莓派上分别运行socket通信程序：  
控制节点：`python3 SDN_on_Raspberry_pi/client.py`   
普通节点：`python3 SDN_on_Raspberry_pi/sever.py`    
从程序中我们可以看出，这与adhoc网络或者有线网络的socket编程是一模一样的，因为应用层编程不需要考虑网络层架构，只要能ping通即可。
## 结束语
因财力有限，本文只用了两台树莓派进行组网，多台组网操作也是一样的。
### 参考文献
[在树莓派上搭建ad-hoc网络教程](https://blog.csdn.net/lby0910/article/details/53420459)   
[OVS初级教程：使用Open vSwitch构建虚拟网络](https://www.sdnlab.com/sdn-guide/14747.html)  
[Open vSwitch系列之二 安装指定版本ovs](https://www.cnblogs.com/goldsunshine/p/10331606.html)  
[ovs之组网实验](https://www.cnblogs.com/mrwuzs/p/10242737.html)  
[SDN系列学习课程-OpenFlow-Ryu-Mininet](https://www.bilibili.com/video/BV1ft4y1a7ip?spm_id_from=333.1007.top_right_bar_window_default_collection.content.click)  
[TCP/IP网络通信之Socket编程入门](https://www.bilibili.com/video/BV1eg411G7pW?spm_id_from=333.1007.top_right_bar_window_default_collection.content.click)  
