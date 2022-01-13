# 论文笔记——A Survey of SDN

## 论文概况
[https://ieeexplore.ieee.org/abstract/document/6739370](https://ieeexplore.ieee.org/abstract/document/6739370)    
IEEE Communications Surveys & Tutorials     
Volume 16 Issue 3   
## 摘要
本文综述了可编程网络的最新进展，重点介绍了SDN。我们提供了可编程网络的历史视角，从早期的想法到最近的发展。然后介绍了SDN网络的体系结构和OpenFlow标准，讨论了当前基于SDN的协议和服务的实现和测试的替代方案，考察了当前和未来的SDN应用，并探讨了基于SDN模式的有前景的研究方向。
## 1 引言
传统网络出现的问题：网络管理和性能调优困难，网络僵化。
可编程网络具有革命性，比如软件定义网络，网络设备成为简单的包转发设备，可以通过开放的接口进行编程。
第二节：早期可编程网络    
第三节：SDN及其体系结构，以及OpenFlow协议。     
第四节：开发和测试SDN的平台和工具。   
第五节：在数据中心和无线网络的应用。    
第六节：面临的挑战和未来发展的方向。    
## 2 早期可编程网络
描述了一些SDN和OpenFlow概念的前身，在早期的一些项目中已经有了关于可编程网络和数控平面分离的思想。
## 3 SDN体系结构
路由器和交换机通常是封闭的系统，只有很少的提供给供应商的接口。适配新版本的协议（比如Ipv6）非常困难，更不用说部署全新的协议和服务。网络僵化效应主要是由于设备的数据和控制平面紧密耦合，新的app或功能的部署需要直接实现在物理设施中。解决网络僵化的一个手段是使用中间设备，比如CDN（内容交付网络）。  
软件定义网络将转发硬件和控制逻辑分离，可以更容易地部署新协议和应用程序。
### A 当前SDN架构
目前有两种SDN架构：ForCES和OpenFlow         
![SDN体系结构](/image/A_Survey_of_SDN_note/SDN体系结构.jpg)     
- ForCES将单个设备中的控制元素与转发元素分开，意图实现在单一网络设备中将转发硬件与第三方控制相结合。它定义了两个逻辑实体，转发元素FE和控制元素CE，它们通过ForCES协议通信，FE负责使用底层硬件来提供每个数据包的处理。CE执行控制和信令功能。    
- OpenFlow完全将控制平面从网络设备上剥离，转发设备基于流表进行转发，流表控制着转发规则。没有匹配流表时按照“table-miss”流表项执行相应的动作，比如丢弃、转发给控制器。控制平面与转发平面通过OpenFlow协议进行通信，远程控制器可以添加、删除或更新交换机的流表项。
## 语句摘录
As a result, network management and performance tuning is quite challenging and thus error-prone.   
因此，网络管理和性能调优非常具有挑战性，所以很容易出错。    
Because of its huge deployment base and the fact it is considered part of our society’s critical infrastructure (just like transportation and power grids), the Internet has become extremely difficult to evolve both in terms of its physical infrastructure as well as its protocols and performance.   
由于其庞大的部署基础，且被视为我们社会关键基础设施(就像交通和电网)的一部分，所以互联网在其物理基础设施、协议和性能方面的发展已经变得极其困难。
