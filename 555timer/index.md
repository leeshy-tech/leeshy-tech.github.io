# 555定时器


## 什么是555定时器
555定时器是一种集成电路芯片，常被用于定时器、脉冲产生器和振荡电路。555可被作为电路中的延时器件、触发器或起振元件。  
![555timer](/image/555timer/555timer_pin.jpg)  

内部组成如下：由比较器C1C2、RS触发器、缓冲非门G、放电二极管T组成

![555timer](/image/555timer/555timer_inner.jpg)
## 芯片分析
电压比较器：一种模拟输入、数字输出的接口电路，输出特性如下：
| in          | out         |
| :---        |    :----:   |
| Vp > Vn     | 1           |
| Vp < Vn     | 0           | 

在555定时器内部，左侧是纯电阻分压电路，将比较器阈值设置为   

$$ 
V_{P1}=\frac{2}{3} V_{cc}, V_{N2}=\frac{1}{3} V_{cc} 
$$

C1C2的输出特性如下：    
| C1          |             |C2           |             | 
| :---        |    :----:   | :---        |    :----:   |
| in          | out         | in          | out         |
| Vp > 2/3Vcc     | 0           | Vn > 1/3Vcc     | 1           |
| Vp > 2/3Vcc     | 1           | Vn < 1/3Vcc     | 0           |

结合RS触发器的真值表可得555定时器的真值表如下： 
| Ui1          | Ui2            |复位Rd           | Uo            |放电管T             |
| :---        |    :----:   | :---        |    :----:   |    :----:   |
|x|x|0|0|导通|
|<2/3Vcc|<1/3Vcc|1|1|截止|
|>2/3Vcc|>1/3Vcc|1|0|导通|
|<2/3Vcc|>1/3Vcc|1|不变|不变|
## 典型应用
- 单稳态电路
- 多谐振荡器
- 施密特触发器
## 555定时器为核心的单稳态电路
### 理论分析
应用电路如下：  
![单稳态电路](/image/555timer/application.jpg)    
- 稳定状态时，管脚2输入信号为高电平，管脚6跨电容与地相连，相当于输入低电平，复位引脚4接高电平，故输出保持低电平不变，放电二极管保持导通不变。
- 当输入信号的下降沿到来，输出变为高电平，放电二极管截止，管脚7向外放电，电容C开始充电，直到$
u_C=\frac{2}{3} V_{cc}$
若忽略晶体管的饱和压降，则充电时间为
$t_w=RCln \frac{\mathrm{Vcc}}{Vcc-\frac{2}{3} Vcc} =RCln 3 \approx 1.1 RC$
- 当输入信号回到高电平且电容充电完毕后，输出回到低电平，放电二极管截止。  
### 仿真验证
使用Mutisim仿真，电路如下： 

![单稳态电路仿真](/image/555timer/mutisim.jpg)

选用回弹式开关使输入信号在电容充电完毕之前回到高电平，以便于从示波器图形上测得充电时间。    
仿真结果：  

![仿真结果](/image/555timer/result.jpg)  
电路行为与理论分析一致，根据理论分析公式
$\mathrm{t}_{\mathrm{w}}=1.1 \times 10 \mathrm{k} \times 4.7 \mathrm{u}=51.7 \mathrm{~ms} $,仿真测得充电时间为50.94ms，且充满后电压为2.985V，与理论分析相符的很好。
### 总结
该单稳态触发器的功能总结为：将低电平脉冲信号（持续时间小于1.1RC）转化为持续时间为1.1RC的高电平信号，持续时间大于1.1RC的话，就只是由低电平转化为高电平。
## 参考文献及备注
### 参考文献
[555定时器及应用](https://blog.csdn.net/acslsr/article/details/105730908)  
[电压比较器_百度百科](https://baike.baidu.com/item/%E7%94%B5%E5%8E%8B%E6%AF%94%E8%BE%83%E5%99%A8/775444)    
[555定时器_百度百科](https://baike.baidu.com/item/555%E5%AE%9A%E6%97%B6%E5%99%A8/6740686)  
[NE555P_立创商城](https://item.szlcsc.com/47753.html) 
### 备注
仿真文件及数据手册：[https://github.com/leeshy-tech/Electronic-Design/tree/main/555timer](https://github.com/leeshy-tech/Electronic-Design/tree/main/555timer)  
