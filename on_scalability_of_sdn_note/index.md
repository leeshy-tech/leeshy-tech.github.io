# 论文笔记——On Scalability of SDN

## 论文概况
[https://ieeexplore.ieee.org/abstract/document/6461198](https://ieeexplore.ieee.org/abstract/document/6461198)  
IEEE Communications Magazine     
Volume 51 Issue 2 
## 摘要
在本文中，我们解构了软件定义网络中的可扩展性问题，并认为它们不是SDN所独有的。我们探讨了在不同环境中经常出现的问题，讨论了SDN设计空间中可扩展性的权衡，并介绍了一些关于SDN可扩展性的最新研究。此外，我们还列举了在可扩展性方面的重要机遇和挑战。
## 引言
普遍认为SDN的控制是集中的，这导致了对SDN规模的能力和弹性的担忧。毕竟，无论控制器的能力如何，中央控制器都不会随着网络的增长而扩展(增加交换机、流量、带宽等的数量)，并且在提供相同的服务保证的同时也无法处理所有传入的请求。此外，由于大多数早期的SDN提议是基于流的，额外的流初始化延迟成为一个问题。
