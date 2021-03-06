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
总的来说，SDN中的故障恢复过程并不比传统网络中的更差。因此，存在类似的可伸缩性问题，并且在传统网络中用于最小化停机时间的相同技术也适用于SDN。

## 不同的网络设置中的可伸缩性

采用一种不同的方法，解释不同类型网络中的可伸缩性模式和陷阱。

### 数据中心

一个典型的数据中心网络有数万个交换元件，并且可以快速增长。在该规模的任何网络中生成的控制事件的绝对数量足以使任何集中式控制器过载。解决这个问题的一种方法是主动在交换机上安装规则，在它们进入控制平面之前有效地消除大多数控制请求。当然，这里的成本是控制器精度和反应性的损失。

当应用程序需要精确的流量统计和/或响应性时，可以将应用程序部署在交换机附近。例如，只要最小化对全局状态的访问，就可以将频繁的事件委托给运行在终端主机上的进程。考虑到整个数据中心网络的处理资源可用性，可以使用像Kandoo[9]这样的解决方案达到任意可伸缩性级别。分布式控制器(如HyperFlow或Onix)也可以是数据中心网络的合理解决方案。考虑到这种网络中的低延迟，状态和流设置的同步延迟将是最小的，对于大多数应用程序来说是可以接受的。

### 服务提供商网络

通常，服务提供商网络没有数据中心网络那么多的交换机/路由器；然而，这种网络中的节点通常是在地理上分布的。这些网络的大直径加剧了控制器的可伸缩性问题、流设置和状态收敛延迟以及一致性要求。我们可以利用网络的物理分布将其划分为单独的区域；每个分区都可以由一个独立的控制器控制，这些控制器只能交换所需的状态变化事件，有效地向外部控制器隐藏了大部分事件。考虑到这种网络中固有的延迟，所有的控制应用程序都应该是容忍延迟的，并且有较弱的一致性要求。

除了高延迟外，服务提供商网络通常比其他网络有更多的流量。因此，这里也关注数据路径资源限制。流的聚合是一个简单的解决方案，它以控制中的粒度为代价。我们注意到，这些问题也存在于传统的网络中，而这并不是SDN所独有的。

## 机遇和挑战

传统上，网络的可伸缩性是基于性能指标来研究的，也就是说，当我们沿着给定的维度扩展网络时，特定的性能指标是如何变化的。在实践中，还有其他正交的方面深刻地影响系统如何适应增长。例如,可管理性(如何方便地管理网络,在大尺度状态下添加,删除,或修改网络元素)和功能可伸缩性(是否方便将新功能添加到网络)与网络可伸缩性、性能一样重要，不应被忽视。关于行为和编程抽象、测试和验证以及SDN可扩展性的初步结果表明，我们认为SDN在这一领域提供了一个重要的机遇。显然，在我们能够充分发挥SDN的潜力之前，我们在这些方面都面临着重大挑战。

### 行为和编程抽象

### 测试和验证

### 可延展性

## 结论

自SDN引入以来，可伸缩性一直是一个主要问题。目前关于SDN可伸缩性的研究表明：

- 这些担忧既不是由SDN引起的，也不是由SDN所独有的。
- 这些问题大多可以得到解决，而不会失去SDN的好处。

在这个领域，通常被忽视的是SDN对网络增长的其他限制因素的影响，如网络规划和管理复杂性。软件定义的网络增加了一种灵活性，可以大规模地适应网络编程和管理。传统的网络在这一领域历来都失败过。最近在这个方向上的尝试非常有希望，尽管未来仍面临许多挑战。
