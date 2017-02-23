# Audio Codec

>codec指在`硬件电路`上`数字信号`到`模拟信号`的转换

## 基本功能

|	功能		|	说明		|
|	---		|	---		|
|	PGA		|	Programmable Gain Amplifier即可通过程序控制增益的放大器 |
| 	BIAS	|	偏置电压，提供一个电压，比如给 Mic供电，让其能够产生信号	|
|	VMID	|	Middle voltage,用于给放大器输出提供一个参考电压，使负载静态电压稳定。VMID一般也是一个放大器的输出提供	|
|	MIXER	|	用于将不路的信号合成一路信号输出，也可起信号选择作用，即多路开关	|
|	ADC		|	模拟信号转数字信号，即将模拟信号抽样、量化、编码，成为可为计算机处理的数字信号，如录音	|
|	I2S/PCM	|	为 codec与 CPU之间的接口，用于传输数字信号	|
|	PCM编解码电路	|	pulse code modulation，脉冲编码调制，其实就是 ADC的另一种说法		|


## 数据的时序




## 硬件设备

### wm8594

#### 格式:

其所支持的常见音频格式:

1. I2S
2. Left Justified(LJ)
3. Right Justified(RJ)
4. DSP mode A
5. DSP mode B

#### 工作流程

##### 上电

1. 使用合适的电压


##### 断电



