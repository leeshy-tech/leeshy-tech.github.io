# 论文笔记——On Scalability of SDN

## 论文概况
[https://ieeexplore.ieee.org/abstract/document/6461198](https://ieeexplore.ieee.org/abstract/document/6461198)  
IEEE Communications Magazine     
Volume 51 Issue 2 
## 摘要
在本文中，我们解构了软件定义网络中的可扩展性问题，并认为它们不是SDN所独有的。我们探讨了在不同环境中经常出现的问题，讨论了SDN设计空间中可扩展性的权衡，并介绍了一些关于SDN可扩展性的最新研究。此外，我们还列举了在可扩展性方面的重要机遇和挑战。
## 引言
普遍认为SDN中的控制是集中的，这导致了对SDN可伸缩性和弹性的关注。毕竟，无论控制器的能力如何，中央控制器都不会随着网络的增长而扩展(增加交换机、流量、带宽等的数量)，并且在提供相同的服务保证的同时也无法处理所有传入的请求。此外，由于大多数早期的SDN提议都是基于流的，额外的流启动延迟成为一个问题。     
我们认为SDN的可扩展性没有固有的瓶颈;我们认为这些可扩展性限制并不局限于SDN;传统的控制协议设计也面临着同样的挑战。虽然这并没有解决这些问题，但这表明我们在SDN中不需要比在传统网络中更担心可扩展性。
## SDN中可扩展性的根源
SDN与传统数据网络的根本区别在于控制与转发平面的分离。这种解耦导致了一些有趣的特性。     
然而，这种解耦也有它自己的陷阱。在这两个平面之间定义一个标准的API是绝对重要的。从技术上讲，这个API应该能够处理各种体系结构的需求，并且应该能够促进这两个平面的独立演化。此外，所有或大多数交换机供应商应该采用相同的标准API，以使其有用；否则，网络将与特定的供应商绑定，会阻碍网络的快速变化和创新。将传统的本地控制功能移动到远程控制器可能会导致新的瓶颈。它还可能导致信令开销。     
在接下来的内容中，我们首先讨论SDN控制器的可扩展性，概述为什么它一直受到关注，以及最近在这个领域的工作。然后，我们回顾一些其他经常提到的SDN可扩展性问题，包括流程设置开销和故障恢复能力。我们认为，尽管这些问题不是SDN特有的，但它们可以通过另一种设计来缓解(其中一些设计现在很常见)。
## 控制器可扩展性
一种可能的SDN设计是将所有的控制功能推到一个集中式控制器上。控制器有可能成为网络操作中的瓶颈，随着网络规模的增长，更多的事件和请求会被发送到控制器，并且在某个时刻，控制器无法处理所有传入的请求。缓解这种担忧的一种方法是在多核系统中提高并行性，并提高IO性能。第二种方法是减少转发到控制器的请求数量，比如DevoFlow通过底层网络约束，牺牲细粒度流级可见性（ fine-grained flow-level visibility）换来了可扩展性。   
或者，还可以将控制功能的状态和/或计算分配到多个控制器上。提供一个严格一致的集中视图可能会阻碍响应时间和吞吐量。在保持可用性和分区容差的同时实现强一致性并不总是可行的。因此，选择一个合适的一致性水平是SDN中一个重要的设计权衡。    
有一些解决方案，我们可以物理地分布控制平面元素，同时保持网络范围的视图。例如，Onix就是一个分布式控制平台，它促进了分布式控制平面的实现。它为控制应用程序提供了一组通用api，以方便访问分布在Onix实例上的网络状态(NIB)。另一方面，HyperFlow在多个控制器实例之间同步网络状态，使控制应用程序(在每个控制器实例上运行)产生控制整个网络的错觉。这保持了在中央控制器上开发控制平面的简单性，同时减轻了与中央控制器相关的可伸缩性问题，尽管这是针对满足某些特性的一组更受限制的控制应用程序。   
Kandoo[9]采用了一种不同的方法来分配控制平面。它定义了一个操作范围，使具有不同要求的应用程序能够共存：本地作用域的应用程序（即，可以使用交换机的本地状态进行操作的应用程序）部署在数据路径附近，以处理频繁的请求并保护控制平面的其他部分免受负载。另一方面，根控制器负责需要网络范围状态的应用程序，同时也充当本地控制器之间需要的任何协调的中介。   
一个有趣的观察结果是，SDN中的控制平面可扩展性挑战（例如，收敛性和一致性要求）与传统网络设计中所面临的挑战并没有本质上的不同。SDN本身既不太可能消除控制平面设计的复杂性，也不太可能使其或多或少具有可扩展性。    
与传统网络不同，在SDN中，我们不需要反复解决基本但具有挑战性的问题，如拓扑发现、状态分布和恢复力。
## 其他SDN可扩展性问题
### 流启动开销
让我们回顾一下流设置过程来解释瓶颈，并展示一个好的设计如何可以避免它们：    
- 包到达但是没有匹配到流规则
- 交换机产生一个流请求给控制器
- 控制器返回一个新的流转发规则
- 交换机更新流表
前三个步骤和最后一个步骤的性能部分取决于交换机能力和资源(管理CPU、内存等)【？这里为什么要说前三个和最后一个】。以及其软件堆栈的性能。第三步中的延迟是由控制器的资源以及控制程序的性能来决定。最后，交换机的FIB更新时间导致了完成流程设置过程的延迟。    
目前支持OpenFlow的软交换机性能远好于硬件交换机，原因是交换机上缺乏资源（管理CPU），对交换机芯片组和管理CPU之间的高频通信的支持不足，以及不佳的软件实现。可以预见，FIB更新时间将成为开关侧流设置延迟的主要因素。
While we argue that controllers and, in the near future, switches would be able to sustain sufficient throughput with negligible latency for reactive flow setup, in the end the control logic determines the scalability of a reactive design. A control program installing an end-to-end path on a per-flow basis does not scale, because the per switch memory is fixed but the number of forwarding entries in the data path grows with the number of active flows in the network. However,  the control program may install aggregate rules matching a large number of micro-flows (thereby facing the same scalability challenges as a proactive design), or proactively install rules in the network core to provide end-to-end connectivity and identify quality of service (QoS) classes, while classifying and reactively labeling flows at the edge.  A viable solution to the scalability challenges of the proactive designs in the former class due to data path memory scarcity is proposed in DIFANE [5]; while the scalability  the latter class follows from the observation that the fanout of an edge switch and thus the number of flows initiated there is bounded (just add edge controllers as the network grows in size).  【没看懂】
### 故障恢复
早期的系统为单中心控制的设计树立了榜样，因此对故障的恢复能力一直是一个主要问题。一个状态同步的从控制器将足以从控制器故障中恢复，但一个网络分区将留下一半的网络无脑。在多控制器网络中，如果有适当的控制器发现机制，交换机总是可以发现一个存在于其分区中的控制器。因此，在一个可伸缩的发现机制下，控制器故障不会对SDN的可伸缩性构成挑战。 
响应于链路失效的收敛有五个步骤。交换机检测到变化。然后交换机通知控制器。在收到通知后，控制程序计算修复操作，并将更新推到受影响的数据路径元素，这些元素反过来更新它们的转发表。在传统网络中，链路故障通知会在整个网络中泛滥，而在SDN中，该信息直接发送到控制器。因此，SDN网络中的信息传播时延并不比传统网络差。另外，SDN的一个优点是，计算是在更有能力的控制机器上进行的，而不是在所有交换机的弱管理cpu上进行的，无论它们是否受到故障的影响。    
请注意，上述参数建立在隐含的假设之上，即故障的交换机或链路不影响交换机-控制器通信信道。如果有一个出现故障的链路或交换机是控制网络本身的一部分，则需要首先修复控制网络本身。因此，在这种角落的情况下，收敛速度可能比传统网络要慢。   
总的来说，SDN中的故障恢复过程并不比传统网络中的更差。因此，存在类似的可伸缩性问题，并且在传统网络中用于最小化停机时间的相同技术也适用于SDN
