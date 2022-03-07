# Ryu——安装


# Ryu安装

## 安装git

`sudo apt install git`

## 安装python3

> 时代变了，不装不行。

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

## 安装Ryu

- 更新pip

  `sudo pip install --upgrade pip`

- 克隆源代码

  `git clone git://github.com/faucetsdn/ryu.git`

- 进入ryu目录

  `cd ryu`

- 安装依赖

  `sudo pip install -r tools/pip-requires`

- 安装ryu

  `sudo python setup.py install`

## 测试

- 进入ryu/ryu/app目录

  `cd ryu/ryu/app`

- 运行simple_switch.py

  `ryu-manager app/simple_switch.py`

  出现以下输出为正常：

  ```
  loading app app/simple_switch.py
  loading app ryu.controller.ofp_handler
  instantiating app app/simple_switch.py of SimpleSwitch
  instantiating app ryu.controller.ofp_handler of OFPHandler
  ```

## 参考文献

[Ubuntu将默认python版本改为python3](https://blog.csdn.net/sexyluna/article/details/105740519)

[https://github.com/leeshy-tech/ryu](https://github.com/leeshy-tech/ryu)
