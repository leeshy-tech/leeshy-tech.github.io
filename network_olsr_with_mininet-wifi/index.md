# 网络实验——在linux平台安装OLSR协议


# 在linux平台安装OLSR协议

## 前言

因为要做一个OLSR和SDN自定义控制面的对比实验，所以要利用mininet-wifi平台自定义拓扑跑一下OLSR协议。

平台：ubuntu20.04

## 安装

- 官网：[www.olsr.org/](www.olsr.org/)

- github：[https://github.com/OLSR/olsrd](https://github.com/OLSR/olsrd)

### 通过mininet-wifi安装

- 进入mininet-wifi目录`sudo ./install.sh -o`

### 从git安装(没有亲自试，来自网络)

- 安装语法分析器：`sudo apt install bison flex`

- 下载源码：`git clone https://github.com/OLSR/olsrd`

- 编译：`cd olsrd;make`

- 安装：`sudo make install`

## mininet-wifi拓扑构建

> 构造一个网络拓扑来测试OLSR协议。

拓扑文件（参考example/adhoc.py）：

[https://github.com/leeshy-tech/mininet-wifi/blob/master/examples/OLSR/olsr_test.py](https://github.com/leeshy-tech/mininet-wifi/blob/master/examples/OLSR/olsr_test.py)

```python
#!/usr/bin/python

"""
This example use four motionless station to test the OLSR protocol in adhoc network.
It's almost the same as example/adhoc.py.
use "sudo python olsr_test.py olsrd" in terminal to run it.
"""

import sys

from mininet.log import setLogLevel, info
from mn_wifi.link import wmediumd, adhoc
from mn_wifi.manetRoutingProtocols import olsrd
from mn_wifi.cli import CLI
from mn_wifi.net import Mininet_wifi
from mn_wifi.wmediumdConnector import interference


def topology(args):
    "Create a network."
    net = Mininet_wifi(link=wmediumd, wmediumd_mode=interference)

    info("*** Creating nodes\n")
    kwargs = dict()
    if '-a' in args:
        kwargs['range'] = 100

    sta1 = net.addStation('sta1', ip6='fe80::1',position='25,50,0', **kwargs)
    sta2 = net.addStation('sta2', ip6='fe80::2',position='75,10,0', **kwargs)
    sta3 = net.addStation('sta3', ip6='fe80::3',position='75,90,0', **kwargs)
    sta4 = net.addStation('sta4', ip6='fe80::4',position='125,50,0', **kwargs)

    net.setPropagationModel(model="logDistance", exp=4)

    info("*** Configuring wifi nodes\n")
    net.configureWifiNodes()

    info("*** Creating links\n")
    # MANET routing protocols supported by proto:
    # babel, batman_adv, batmand and olsr
    # WARNING: we may need to stop Network Manager if you want
    # to work with babel
    protocols = ['babel', 'batman_adv', 'batmand', 'olsrd', 'olsrd2']
    kwargs = dict()
    for proto in args:
        if proto in protocols:
            kwargs['proto'] = proto

    net.addLink(sta1, cls=adhoc, intf='sta1-wlan0',
                ssid='adhocNet', mode='g', channel=5,
                ht_cap='HT40+',  **kwargs)
    net.addLink(sta2, cls=adhoc, intf='sta2-wlan0',
                ssid='adhocNet', mode='g', channel=5,
                ht_cap='HT40+', **kwargs)
    net.addLink(sta3, cls=adhoc, intf='sta3-wlan0',
                ssid='adhocNet', mode='g', channel=5,
                ht_cap='HT40+', **kwargs)
    net.addLink(sta4, cls=adhoc, intf='sta4-wlan0',
                ssid='adhocNet', mode='g', channel=5,
                ht_cap='HT40+', **kwargs)

    net.plotGraph(max_x=200, max_y=200)

    info("*** Starting network\n")
    net.build()

    info("\n*** Addressing...\n")
    if 'proto' not in kwargs:
        sta1.setIP6('2001::1/64', intf="sta1-wlan0")
        sta2.setIP6('2001::2/64', intf="sta2-wlan0")
        sta3.setIP6('2001::3/64', intf="sta3-wlan0")
        sta4.setIP6('2001::4/64', intf="sta4-wlan0")

    info("*** Running CLI\n")
    CLI(net)

    info("*** Stopping network\n")
    net.stop()


if __name__ == '__main__':
    setLogLevel('info')
    topology(sys.argv)
```

可视化：

![image-20220327224226493](/image/Linux/image-20220327224226493.png)

解读：

从图中可以看出，sta1只能与sta2和sta3进行单跳通信，如果要与sta4通信，就需要sta2或者sta3进行中继。如果没有OLSR协议，节点在收到目的IP不是本节点的包之后就会丢掉，无法完成中继。OLSR协议会在网络中的节点维护整个网络拓扑，就能完成中继。

## 实验测试

### 关闭NetworkManager

`sudo systemctl stop NetworkManager`

NetworkManager是linux的自动网络配置工具，我们希望自己配置网络，所以要把它关掉。

查看节点的网络状态`ip addr`，没有图中的`NO-CARRIER`说明NetworkManager已经被关闭。（这个命令可以在本机运行，也可以在mininet虚拟主机运行。）

![](/image/Linux/image-20220327230005057.png)

### 配置OLSR

编辑配置文件：`vim /etc/olsrd/olsrd.conf`

添加所有节点的网卡：

![image-20220327225209922](/image/Linux/image-20220327225209922.png)

退出、保存

### 运行拓扑文件

`sudo python olsr_test.py olsrd`

![image-20220327225449859](/image/Linux/image-20220327225449859.png)

### 网络测试

`sta1 ping sta4`

![image-20220327225429915](/image/Linux/image-20220327225429915.png)

能ping通，说明OLSR协议运行正常。

查看路由：`sta3 route`

![image-20220327225742818](/image/Linux/image-20220327225742818.png)

多了三条路由，这是OLSR协议运行的结果。

## 结束

### 恢复系统

开启NetworkManager：`sudo systemctl start NetworkManager`

退出mininet-wifi：`exit`

清理系统：`sudo mn -c`

### 参考文献

[centos7 开启 关闭 NetworkManager](https://blog.csdn.net/whatday/article/details/106096127)

[Linux虚拟机下OLSR协议的安装](https://blog.csdn.net/qq_35109869/article/details/79839357)

[Unable to create IPv6 multi hop mesh network in ad hoc mode #342](https://github.com/intrig-unicamp/mininet-wifi/issues/342)这个issue救了大命了

[Linux卸载olsrd,olsrd路由协议移植到嵌入式linux中使用](https://blog.csdn.net/weixin_29279047/article/details/116832580)

[www.olsr.org/](www.olsr.org/)官网写的说明太少了，根本看不懂，麻了。

[https://github.com/OLSR/olsrd](https://github.com/OLSR/olsrd)

