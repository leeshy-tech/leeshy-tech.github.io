# ryu开发——集线器

## 什么是集线器
集线器是运作在OSI模型中的物理层，它将某个端口收到的包向所有端口广播（也叫洪泛，flood）。
## 实现代码
```py
# simple_hub.py
from ryu.base import app_manager
from ryu.controller import ofp_event
from ryu.controller.handler import MAIN_DISPATCHER
from ryu.controller.handler import set_ev_cls
 
class L2Switch(app_manager.RyuApp):
    def __init__(self, *args, **kwargs):
        super(L2Switch, self).__init__(*args, **kwargs)
 
    @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
    def packet_in_handler(self, ev):
        msg = ev.msg
        datapath = msg.datapath
        ofp = datapath.ofproto
        ofp_parser = datapath.ofproto_parser
 
        actions = [ofp_parser.OFPActionOutput(ofp.OFPP_FLOOD)]
        out = ofp_parser.OFPPacketOut(
            datapath=datapath, buffer_id=msg.buffer_id, in_port=msg.in_port,
            actions=actions)
        datapath.send_msg(out)
```
## 实现逻辑
当控制器收到PacketIn消息，下发PacketOut消息，指示交换机将该包洪泛（FLOOD）。
## 结束语
### 参考文献
[RYU入门教程](https://www.sdnlab.com/1785.html)
