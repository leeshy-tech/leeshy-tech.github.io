# mininet-wifi——安装


# 安装mininet-wifi

mininet学了快半年了，还是安装七八次才能装上，人麻了，怒写一篇博客。平台：Ubuntu20.04。

## 安装git

`sudo apt install git`

## 安装python3

mininet在python2和python3环境下都是能正常运行的，但是，apt中一些python2的包已经升级到3了，比如python-matplotlib，你不升级python3，mininet-wifi就会安装它，但是apt里又没有，只会一直报错。你再怎么更新源，也找不到这个玩意，因为它已经变成python3-matplotlib了。所以还是要安装python3。

- 下载

  `sudo apt install python3`

- 查看python3的位置和版本号，用于第四步

  `whereis python3`

- 删除原来python2的软连接

  `sudo rm /usr/bin/python`

- 建立新的软连接

  `sudo ln -s /usr/bin/python3.9 /usr/bin/python`

- 查看版本

  `python --version`

## 安装mininet-wifi

### 有梯子，或者有git代理

- 克隆源代码

  `git clone https://github.com/intrig-unicamp/mininet-wifi`

- 查看安装可选项

  `sudo mininet-wifi/util/install.sh -h`

- 选择一些项安装（默认就按这个）

  `sudo mininet-wifi/util/install.sh -Wlnfv`

### 无代理

这里主要是指以https无法正常访问github的情况，需要把所有的git网址改为git://开头，因为install.sh里会有下载其他库的git命令，所以直接运行install.sh会报git超时，需要提前把相关库下好。

- 克隆

  `git clone git://github.com/intrig-unicamp/mininet-wifi`

  `git clone git://github.com/ramonfontes/mac80211_hwsim_mgmt`

  `git clone git://github.com/mininet/mininet`

  `git clone git://github.com/vchakour/wmediumd`

- 选择一些项安装（默认就按这个）

  `sudo mininet-wifi/util/install.sh -Wlnfv`

## 运行

- 运行，这句正常运行就说明下载成功。

  `sudo mn --wifi`

- 退出

  `exit`

- 清理

  `sudo mn -c`

## 参考文献

[https://github.com/intrig-unicamp/mininet-wifi](https://github.com/intrig-unicamp/mininet-wifi)

[Ubuntu将默认python版本改为python3](https://blog.csdn.net/sexyluna/article/details/105740519)

