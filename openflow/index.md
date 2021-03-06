# 论文笔记——OpenFlow: Enabling Innovation in Campus Networks


# OpenFlow: Enabling Innovation in Campus Networks

OpenFlow：实现校园网络的创新

## 论文概况

[ACM SIGCOMM Computer Communication Review](https://dl.acm.org/toc/sigcomm-ccr/2008/38/2)[Volume 38](https://dl.acm.org/toc/sigcomm-ccr/2008/38/2)[Issue 2](https://dl.acm.org/toc/sigcomm-ccr/2008/38/2)

April 2008 pp 69–74

自翻：[https://github.com/leeshy-tech/PaperTranslate/blob/main/OpenFlow.md](https://github.com/leeshy-tech/PaperTranslate/blob/main/OpenFlow.md)

## 摘要

本白皮书提出了 OpenFlow：一种供研究人员在他们每天使用的网络中运行实验协议的方法。 OpenFlow 基于以太网交换机，具有内部流表和用于添加和删除流条目的标准化接口。我们的目标是鼓励网络供应商将 OpenFlow 添加到他们的交换机产品中，以部署在大学校园骨干网和配线间中。我们认为 OpenFlow 是一种务实的折衷方案：一方面，它允许研究人员以统一的方式以线速和高端口密度在异构交换机上运行实验；另一方面，供应商不需要公开其交换机的内部工作原理。除了允许研究人员在现实世界的流量设置中评估他们的想法外，OpenFlow 还可以作为一个有用的校园组件，用于提议的大规模测试平台（如 GENI）。斯坦福大学的两座建筑物将很快使用商用以太网交换机和路由器运行 OpenFlow 网络。我们将努力鼓励在其他学校部署；我们鼓励您也考虑在您的大学网络中部署 OpenFlow。

## 1. 对可编程网络的需求

 今天，几乎没有实用的方法可以在足够现实的环境（例如，大规模承载真实流量）中试验新的网络协议（例如，新的路由协议或 IP 的替代方案）以获得广泛部署所需的信心。 结果是来自网络研究社区的大多数新想法都未经尝试和测试。 因此，人们普遍认为网络基础设施已经“僵化”。

认识到这个问题后，网络社区正在努力开发可编程网络，例如 GENI [1]，这是一个提议的全国性研究机构，用于试验新的网络架构和分布式系统。 这些可编程网络需要可编程交换机和路由器（使用虚拟化）可以同时处理多个隔离实验网络的数据包。

虚拟化可编程网络可以降低新想法的进入门槛，提高网络基础设施的创新速度。 但全国性设施的计划雄心勃勃（而且成本高昂），部署起来需要数年时间。 本白皮书关注一个离家更近的短期问题：作为研究人员，我们如何在校园网络中进行实验？ 如果我们能弄清楚这个问题，我们可以很快开始并将该技术扩展到其他校园，以造福整个社区。

 我们的目标是提出一种新的交换机功能，可以帮助将可编程性扩展到大学校园的配线间。

- 一种我们不采用的方法是说服商业设备供应商在他们的交换机和路由器上提供一个开放的、可编程的、虚拟化平台，以便研究人员可以部署新协议，而网络管理员不用担心设备的支持问题。这种结果在短期内不太可能发生。
- 一些开放的软件平台已经存在，但没有我们需要的性能或端口密度。

- 具有用于线速处理的专用硬件的现有平台也不太适合大学布线室。

因此，商业解决方案过于封闭和不灵活，研究解决方案要么性能不足或扇出，要么过于昂贵。具有完全通用性的研究解决方案似乎不太可能克服其性能或成本限制。一种更有希望的方法是在通用性上妥协并寻求一定程度的开关灵活性，即：

- 适合高性能和低成本的实施。
- 能够支持广泛的研究。
- 确保将实验流量与生产流量隔离开来。
- 符合供应商对封闭平台的需求。 

本文描述了 OpenFlow 交换机 — 一种最初尝试满足这四个目标的规范。

## 2. OpenFlow交换机

基本思想很简单：我们利用大多数现代以太网交换机和路由器包含以线速运行的流表（通常由 TCAM 构建）来实现防火墙、NAT、QoS 和收集统计数据。虽然每个供应商的流表都不同，但我们已经确定了一组有趣的通用功能，可在许多交换机和路由器中运行。

OpenFlow 利用了这组通用功能。 OpenFlow 提供了一个开放协议来对不同交换机和路由器中的流表进行编程。网络管理员可以将流量划分为生产和研究流。研究人员可以通过选择他们的数据包遵循的路线和他们接收的处理来控制他们自己的流量。通过这种方式，研究人员可以尝试新的路由协议、安全模型、寻址方案，甚至是 IP 的替代方案。在同一个网络上，生产流量被隔离并以与今天相同的方式进行处理。

OpenFlow 交换机的数据路径由流表和与每个流条目关联的操作组成。 OpenFlow 交换机支持的操作集是可扩展的，但下面我们描述了所有交换机的最低要求。为了实现高性能和低成本，数据路径必须具有精心规定的灵活性。这意味着放弃指定任意处理每个数据包的能力，并寻求更有限但仍然有用的操作范围。因此，在本文后面，为所有 OpenFlow 交换机定义一组基本的必需操作。

一个 OpenFlow 交换机至少由三部分组成：（1）流表，与每个流条目相关联的动作，用于告诉交换机如何处理流，（2）将交换机连接到远程控制程序（称为控制器）的安全通道，它在控制器和交换机之间发送命令和数据包，使用 (3) OpenFlow 协议，该协议为控制器与交换机通信提供了一种开放和标准的方式。 通过指定一个标准接口（开放流协议），流表中的条目可以通过该接口在外部定义，开放流交换机避免了研究人员对交换机进行编程的需要。

**专用OpenFlow交换机。** 专用 OpenFlow 交换机是在端口之间转发数据包的哑数据路径元素，由远程控制程序定义。 图 1 显示了 OpenFlow 交换机的示例。

![image-20220628161538032](/image/Paper/image-20220628161538032.png)

在这种情况下，流被广泛定义，并且仅受流表的特定实现的能力的限制。 例如，一个流可以是一个 TCP 连接，或来自特定 MAC 地址或 IP 地址的所有数据包，或具有相同 VLAN 标记的所有数据包，或来自同一交换机端口的所有数据包。 对于涉及非 IPv4 数据包的实验，可以将流定义为与特定（但非标准）标头匹配的所有数据包。

每个流条目都有一个与之关联的简单动作； 三个基本的（所有专用 OpenFlow 交换机都必须支持）是：

1. 将此流的数据包转发到给定端口（或多个端口）。 这允许数据包通过网络进行路由。 在大多数交换机中，这预计将在线性速率下发生。
2. 封装此流的数据包并将其转发给控制器。 数据包被传递到安全通道，在那里它被封装并发送到控制器。 通常用于新流中的第一个数据包，因此控制器可以决定是否应将流添加到流表中。 或者在某些实验中，它可以用于将所有数据包转发到控制器进行处理。
3. 丢弃该流的数据包。 可用于安全、遏制拒绝服务攻击或减少来自终端主机的虚假广播发现流量。

Flow-Table 中的条目具有三个字段：（1）定义流的数据包头，（2）定义应如何处理数据包的操作，以及（3）统计数据，跟踪数据包的数量。 每个流的数据包和字节，以及自最后一个数据包与流匹配的时间（以帮助删除非活动流）。

在第一代“Type 0”交换机中，流标头是一个 10 元组，如表 1 所示。TCP 流可以由所有十个字段指定，而 IP 流可能不包括其定义中的传输端口。 每个标头字段可以是通配符，以允许聚合流，例如仅定义 VLAN ID 的流将应用于特定 VLAN 上的所有流量。

![image-20220628165616013](/image/Paper/image-20220628165616013.png)

**OpenFlow使能的交换机。** 一些商用交换机、路由器和接入点将通过添加流表、安全通道和 OpenFlow 协议来增强 OpenFlow 功能（我们在第 5 节中列出了一些示例）。 通常，流表将重用现有硬件，例如 TCAM； 安全通道和协议将被移植以在交换机的操作系统上运行。 图 2 显示了支持 OpenFlow 的商业交换机和接入点网络。 在这个例子中，所有的流表都由同一个控制器管理； OpenFlow 协议允许交换机由两个或多个控制器控制，以提高性能或鲁棒性。

![image-20220628165750397](/image/Paper/image-20220628165750397.png)

我们的目标是使实验能够在现有的生产网络中与常规流量和应用程序一起进行。 因此，为了赢得网络管理员的信任，启用 OpenFlow 的交换机必须将实验流量（由 Flow Table 处理）与将由交换机的正常第 2 层和第 3 层管道处理的生产流量隔离开来。 有两种方法可以实现这种分离。 一种是添加第四个动作：

4. 通过交换机的正常处理管道转发此流的数据包。

另一种是为实验和生产流量定义单独的 VLAN 集。 这两种方法都允许交换机以通常的方式处理不属于实验的正常生产流量。 所有启用 OpenFlow 的交换机都需要支持一种方法或另一种方法； 有些人会同时支持两者。

**额外的特性。** 如果交换机支持报头格式和上面提到的四个基本操作（在 OpenFlow 交换机规范中有详细说明），那么我们称其为“Type 0”交换机。 我们预计许多交换机将支持其他操作，例如重写部分数据包头（例如，用于 NAT，或混淆中间链路上的地址），并将数据包映射到优先级类别。 同样，一些流表将能够匹配数据包头中的任意字段，从而可以使用新的非 IP 协议进行实验。 随着一组特定功能的出现，我们将定义一个“Type 1”开关。

**控制器。** 控制器代表实验从流表中添加和删除流条目。 例如，静态控制器可能是在 PC 上运行的简单应用程序，用于在实验期间静态建立流以互连一组测试计算机。 在这种情况下，流类似于当前网络中的 VLAN——提供一种简单的机制来将实验流量与生产网络隔离。 从这个角度来看，OpenFlow 是 VLAN 的泛化。

人们还可以想象更复杂的控制器，它们会随着实验的进行动态地添加/删除流。 在一个使用模型中，研究人员可以控制整个 OpenFlow 交换机网络，并可以自由决定如何处理所有流。 更复杂的控制器可能支持多个研究人员，每个研究人员具有不同的帐户和权限，使他们能够在不同的流程集上运行多个独立的实验。 识别为在特定研究人员控制下的流（例如，通过在控制器中运行的策略表）可以传送到研究人员的用户级控制程序，然后由该程序决定是否应将新的流条目添加到交换机网络 .

## 3. 使用OpenFlow

例子略

关于随着实验的进行动态添加和删除流的控制器的性能、可靠性和可扩展性，有一些合理的问题要问：这样的集中式控制器能否足够快地处理新流和对流开关进行编程？当控制器发生故障时会发生什么？在某种程度上，这些问题在 Ethane 原型的背景下得到解决，该原型使用简单的流量交换机和中央控制器 [7]。初步结果表明，基于低成本台式 PC 的 Ethane 控制器每秒可以处理超过 10,000 个新流量，足以满足大型大学校园的需求。当然，新流的处理速度将取决于研究人员实验所需的处理复杂性。但这让我们相信可以进行有意义的实验。通过使控制器（和实验）无状态，可以实现可扩展性和冗余，允许在多个单独的设备上进行简单的负载平衡。

## 3.1 生产网络上的实践

很有可能，Amy正在许多其他人使用的网络中测试她的新协议。 因此，我们希望网络具有两个额外的属性： 

1. 属于 Amy 以外的用户的数据包应该使用在交换机或路由器中运行的标准且经过测试的路由协议进行路由，该协议运行在“知名品牌”供应商的交换机或路由器中。 
2. Amy 应该只能为她的流量或她的网络管理员允许她控制的任何流量添加流条目。

属性 1 是通过启用 OpenFlow 的交换机实现的。在 Amy 的实验中，所有不是来自 Amy 的 PC 的数据包的默认操作可能是通过正常的处理管道转发它们。 Amy 自己的数据包将直接转发到传出端口，而无需经过正常管道处理。

属性 2 取决于控制器。控制器应该被视为一个平台，使研究人员能够进行各种实验，而Property 2的限制可以通过适当使用权限或其他方式来限制单个研究人员控制流入口的权力来实现。这些类似权限的机制的确切性质将取决于控制器的实现方式。我们预计会出现各种各样的控制器。作为控制器具体实现的一个例子，一些作者正在开发一种称为 NOX 的控制器，作为 Ethane 工作的后续[8]。通过将 GENI 管理软件扩展到 OpenFlow 网络，可能会出现一个完全不同的控制器。

## 3.2 更多实例

与任何实验平台一样，这组实验将超过我们可以预先想到的那些——OpenFlow 网络中的大多数实验还有待考虑。 在这里，为了说明，我们提供了一些示例，说明如何使用支持 OpenFlow 的网络来试验新的网络应用程序和架构。

- 实验1：网络管理和接入控制。
- 实验2:VLAN。
- 实验3：移动无线VOIP客户端。
- 实验4：无IP网络。
- 实验5：处理包而不是流。

在支持 OpenFlow 的网络中处理数据包有两种基本方法。 首先，也是最简单的，是强制所有流的数据包通过控制器。 为此，控制器不会将新的流条目添加到流交换机中——它只是允许交换机默认将每个数据包转发到控制器。 这具有灵活性的优势，但以性能为代价。 它可能为测试新协议的功能提供了一种有用的方法，但对于在大型网络中的部署不太可能引起人们的兴趣。

处理数据包的第二种方法是将它们路由到进行数据包处理的可编程交换机——例如，基于 NetFPGA 的可编程路由器。 优点是可以以用户定义的方式以线速处理数据包； 图 3 显示了如何做到这一点的示例，其中支持 OpenFlow 的交换机本质上作为一个接线板运行，以允许数据包到达 NetFPGA。 在某些情况下，NetFPGA 板（插入 Linux PC 的 PCI 板）可能会放置在支持 OpenFlow 的交换机旁边的配线柜中，或者（更有可能）放在实验室中。

![image-20220628172117618](/image/Paper/image-20220628172117618.png)

## 4.  OpenFlow联盟

## 5. 部署OpenFlow交换机

## 6. 结论

我们认为 OpenFlow 是一种务实的折衷方案，它允许研究人员以统一的方式在异构交换机和路由器上运行实验，而无需供应商公开其产品的内部工作原理，也无需研究人员编写供应商特定的控制软件。

如果我们在校园中成功部署 OpenFlow 网络，我们希望 OpenFlow 将逐渐在其他大学中流行起来，增加支持实验的网络数量。 我们希望出现新一代的控制软件，让研究人员能够重复使用控制器和实验，并在他人工作的基础上再接再厉。 随着时间的推移，我们希望不同大学的 OpenFlow 网络孤岛将通过隧道和覆盖网络相互连接，也许还可以通过在连接大学彼此的骨干网络中运行的新 OpenFlow 网络相互连接。
