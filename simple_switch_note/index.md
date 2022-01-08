# ryu源码解读————simple_switch.py

## 总览
simple_switch.py共110行  
1-18：注释  
21-29：引库  
32-110：继承类RyuApp  
## 32-37
```py
class SimpleSwitch(app_manager.RyuApp):
    OFP_VERSIONS = [ofproto_v1_0.OFP_VERSION]   # 声明支持的Open Flow版本
    # 继承，调用构造函数并添加一个新属性 mac_to_port
    def __init__(self, *args, **kwargs):
        super(SimpleSwitch, self).__init__(*args, **kwargs)
        self.mac_to_port = {}
```
## 39-51    
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
OFPFlowMod：修改流条目消息，控制器发送此消息来修改流表。  
OFPMatch：流匹配规则。  
flags：以下三个值之一
: | OFPFF_SEND_FLOW_REM   当流过期或删除时，发送删除流消息。  
: | OFPFF_CHECK_OVERLAP   首先检查重叠的条目。  
: | OFPFF_EMERG           标记为紧急情况。  

