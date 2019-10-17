# lora的CAD模式

## SX1276：

> 概念 : 定时扫描信道，检测lora数据包的前导码，
降低功耗。随着扩频调制技术的应用，人们无法确认信道被占有，是由于低噪声影响，还是真的数据来到，所以引入了CAD检测。LoRa CAD检测方法：从机设置好频率和扩频因子，开启CAD模式，（注意：无论是否有信号到来，都会产生CADDone中断），当有匹配（相同的频率和扩频因子）的信号到来时，就会产生CADDetect中断，CADDone也会产生，并且，CADDetect和CADDone会同时产生。

[CAD模型的多节点检测介绍链接](http://www.pianshen.com/article/1026562256/;jsessionid=107825730CF41CCF2D201B4BFD374C09)
[无线节点的空中唤醒技术](https://blog.csdn.net/iotisan/article/details/55695465)，介绍CAD的原理，以及 空中唤醒优化的方法 及 噪音的应对，传输锁相的介绍

> 数据帧的结构形式:
![](https://raw.githubusercontent.com/leadercxn/SENSORO/a8d39bfa01b91ef11ab7d5d644835d63f6717bed/lora/lora数据帧的结构模型.png)
由于CAD检测数据包的前导码部分，因此要想实现空中唤醒，结合节点定期检测时间，需要设置合适的前导码发送时间，保证 前导码发送时间>节点定期检测时间 ，则需要设定一定的前导码长度，可通过配置RegPreambleMsb和RegPreambleLsb寄存器来实现。如下图所示，可将前导码寄存器长度设置在6-65536之间来改变发送前导码长度。

#### sx1276存在两种模式:
+ LORA模式
> LoRa调制解调器采用专利扩频调制和前向纠错技术，它融合了数字扩频、数字信号处理和前向纠错编码技术。
![](https://raw.githubusercontent.com/leadercxn/SENSORO/8c77111c4b7c5a531438f15620284b0787944790/lora/lora扩频调制.png)

+ FSK模式
> 用两个频率承载二进制1和0的双频FSK系统
![](https://raw.githubusercontent.com/leadercxn/SENSORO/8c77111c4b7c5a531438f15620284b0787944790/lora/FSK原理.png)

两者区别：
![](https://raw.githubusercontent.com/leadercxn/SENSORO/8c77111c4b7c5a531438f15620284b0787944790/lora/lora和FSK的区别.png)



> 相关科普文：
[lora跟lorawan的区别.pdf](https://wiki.ai-thinker.com/_media/lora/lorawan_faq问题.pdf)
[1276/77/78 datasheet](https://www.zlg.cn/data/upload/software/Wireless/ZM470SX-M_SX1278-data-cn.pdf)
[lora 技术pingpong系统](https://blog.csdn.net/weixin_39148042/article/details/81588897)

> [SX127x芯片数字IO引脚映射](https://blog.csdn.net/HowieXue/article/details/78052758),此文含有大量学习链接。
SX1276/7/8的6个DIO通用IO引脚在LoRa模式下均可用。它们的映射关系取决于RegDioMapping1和RegDioMapping2这两个寄存器的配置，如下表：
![](https://raw.githubusercontent.com/leadercxn/SENSORO/e0e061ea6324252dba57fffc49a4de02a8a4a31f/lora/LORA的DIO映射表.png)
