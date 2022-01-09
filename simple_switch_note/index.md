# ryu源码解读——simple_switch.py

## 总览
文件位置：ryu/ryu/app/simple_switch.py
simple_switch.py共110行  
1-18：注释  
21-29：引库  
32-110：继承类RyuApp  
## 32-37
> 继承类RyuApp，并调用构造函数
```py
class SimpleSwitch(app_manager.RyuApp):
    OFP_VERSIONS = [ofproto_v1_0.OFP_VERSION]   # 声明支持的Open Flow版本
    # 继承，调用构造函数并添加一个新属性 mac_to_port
    def __init__(self, *args, **kwargs):
        super(SimpleSwitch, self).__init__(*args, **kwargs)
        self.mac_to_port = {}
```
## 39-51    
> 实现一个添加流表的函数

```py
# 函数：添加流表
def add_flow(self, datapath, in_port, dst, src, actions):
        ofproto = datapath.ofproto

        match = datapath.ofproto_parser.OFPMatch(
            in_port=in_port,
            dl_dst=haddr_to_bin(dst), dl_src=haddr_to_bin(src)# 源和目的mac地址
            )

        mod = datapath.ofproto_parser.OFPFlowMod(
            datapath=datapath, match=match, cookie=0,
            command=ofproto.OFPFC_ADD, # 命令：添加新流表
            idle_timeout=0, hard_timeout=0,
            priority=ofproto.OFP_DEFAULT_PRIORITY, # 优先级：默认
            flags=ofproto.OFPFF_SEND_FLOW_REM, 
            actions=actions)
        datapath.send_msg(mod)      #控制器下发消息
```
### datapath
控制器与交换机之间是一条Open Flow数据通路，所以控制器通过datapath来区分不同的交换机，datapath具有一个ofproto参数指示OpenFlow协议内容。ofproto的ofproto_parser定义了协议相关的数据结构。
### 协议细节
OFPFlowMod：修改流表消息，控制器发送此消息来修改流表。  
OFPMatch：流匹配规则。   
flags：以下三个值之一
: | OFPFF_SEND_FLOW_REM   当流过期或删除时，发送删除流消息。  
: | OFPFF_CHECK_OVERLAP   首先检查重叠的条目。  
: | OFPFF_EMERG           标记为紧急情况。  
## 53-94
> PacketIn消息的处理逻辑：  

```py
@set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
def _packet_in_handler(self, ev):
    # 解析数据包
    msg = ev.msg
    datapath = msg.datapath
    ofproto = datapath.ofproto

    pkt = packet.Packet(msg.data)
    eth = pkt.get_protocol(ethernet.ethernet)

    if eth.ethertype == ether_types.ETH_TYPE_LLDP:
        # 忽略LLDP类型的包
        return
    dst = eth.dst
    src = eth.src

    # 初始化mac_port对应规则
    dpid = datapath.id
    self.mac_to_port.setdefault(dpid, {})  

    # 打印消息
    self.logger.info("packet in %s %s %s %s", dpid, src, dst, msg.in_port) 

    # 记录此包对应的mac_port对应规则
    self.mac_to_port[dpid][src] = msg.in_port

    # 如果目的IP的对应端口已经知道，就直接设置为输出端口，否则就洪泛
    if dst in self.mac_to_port[dpid]:
        out_port = self.mac_to_port[dpid][dst]
    else:
        out_port = ofproto.OFPP_FLOOD

    # 封装一个OFPActionOutput类型动作：从out_port端口输出
    actions = [datapath.ofproto_parser.OFPActionOutput(out_port)]

    # 如果已经明确了目的IP的输出端口，那么就下发一条流表
    if out_port != ofproto.OFPP_FLOOD:
        self.add_flow(datapath, msg.in_port, dst, src, actions)

    # 如果交换机没有缓存该包，那么就把该包还回去
    data = None
    if msg.buffer_id == ofproto.OFP_NO_BUFFER:
        data = msg.data

    # 发送PacketOut消息
    out = datapath.ofproto_parser.OFPPacketOut(
        datapath=datapath, buffer_id=msg.buffer_id, in_port=msg.in_port,
        actions=actions, data=data)
    datapath.send_msg(out)
```
细节分析：
### LLDP
### PacketIn与PacketOut
当交换机收到某个包之后，没有对应的流表，就向控制器发送PacketIn消息，控制器收到之后，进行一些处理，然后发送PacketOut消息给交换机，指示交换机该如何处理这个包。   
所以PacketIn消息应当包含这个包，在控制器处理逻辑里面首先就是解析出这个包。  
PacketOut消息应当包含一个action，当交换机收到PacketOut时执行这个action。    
### 处理逻辑
- 解析出数据包，根据数据包的IP和输入端口，绑定这个IP和交换机端口。
- 如果目的IP对应的交换机端口已知，那么就把输出端口赋这个值。如果未知，就指示交换机洪泛这个包。
- 如果输出不是洪泛，那么就可以下发流表，绑定目的IP和源IP的转发关系。
- 封装PacketOut消息，下发。
  
### buffer_id与data
交换机具有缓存，不知道如何处理某个包时，它可以选择是否暂存这个包。  
- 如果没有暂存，那么就应当由控制器通过PacketOut消息把该包传回来，通过OFPPacketOut类的data参数。
- 如果暂存了，data参数就是None，PacketOut消息指示这个包暂存的位置，也就是buffer_id。
## 96-110
> 如果端口发生了一些变化，比如端口增加或者删除，那就在命令行打印相关的消息
```py
@set_ev_cls(ofp_event.EventOFPPortStatus, MAIN_DISPATCHER)
def _port_status_handler(self, ev):
    # 解析消息
    msg = ev.msg
    reason = msg.reason
    port_no = msg.desc.port_no

    # 打印
    ofproto = msg.datapath.ofproto
    if reason == ofproto.OFPPR_ADD:
        self.logger.info("port added %s", port_no)
    elif reason == ofproto.OFPPR_DELETE:
        self.logger.info("port deleted %s", port_no)
    elif reason == ofproto.OFPPR_MODIFY:
        self.logger.info("port modified %s", port_no)
    else:
        self.logger.info("Illeagal port state %s %s", port_no, reason)
```
## 结束语
### 参考文献
[RYU入门教程](https://www.sdnlab.com/1785.html)
### 备注
ryu官方：[https://github.com/faucetsdn/ryu](https://github.com/faucetsdn/ryu)   
本人注释版：[https://github.com/leeshy-tech/ryu](https://github.com/leeshy-tech/ryu)
