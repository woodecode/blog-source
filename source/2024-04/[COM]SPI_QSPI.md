```json
{
    "front-matter": {
        "title": "SPI/QSPI",
        "date": "2023-04-02",
        "author": "我是谁",
        "tags": ["Example1", "Example1"],
        "categories": ["通信协议"],
        "description": "文章描述",
        "cover": "封面图片链接",
        "featured": false, 
        "draft": true 
	}
}
```

# SPI/QSPI



## SPI(Serial Peripheral Interface) 

> https://www.analog.com/en/resources/analog-dialogue/articles/introduction-to-spi-interface.html

### 简介

SPI（串行外设接口）是一种广泛使用的同步串行数据通信协议，主要用于短距离通信，特别是在微控制器和其外围设备之间。这种协议因其简单性、高速和效率而在嵌入式系统中非常受欢迎。

### 设备连接框图

![](https://www.analog.com/en/_/media/images/analog-dialogue/en/volume-52/number-3/articles/introduction-to-spi-interface/205973_fig_06.svg)

### 四线连接引脚说明

- **SCLK (Serial Clock)** - 串行时钟。这个引脚由**主设备控制**，用于同步数据传输。所有的数据传输都是在时钟信号的边缘发生的，具体是在上升沿还是下降沿取决于具体的设备配置。

- **MOSI (Master Output, Slave Input)** - 主输出，从输入。这个引脚用于主设备向从设备发送数据。数据从主设备流向从设备。

- **MISO (Master Input, Slave Output)** - 主输入，从输出。这个引脚用于从设备向主设备发送数据。数据从从设备流向主设备。

- **SS (Slave Select) 或 CS (Chip Select)** - 从设备选择或芯片选择。这个引脚由主设备控制，用于激活特定的从设备。在多从设备的系统中，每个从设备通常会有自己的选择信号线。当这个引脚为低电平时（通常配置），相应的从设备被选中并可以开始通信。

### 极性和相位 Clock Polarity and Clock Phase


CPOL（Clock Polarity，时钟极性）和CPHA（Clock Phase，时钟相位）是SPI（Serial Peripheral Interface，串行外设接口）协议中用于定义时钟信号特性的两个参数，它们决定了数据应当在时钟信号的哪个边缘进行采样和传输。不同的CPOL和CPHA的组合定义了四种SPI通信模式，使得SPI能够兼容不同外设的时钟和数据采样要求。

#### CPOL（时钟极性）

CPOL决定了SPI总线**空闲时SCK的电平状态**：

- **CPOL = 0**：空闲时SCK保持低电平。
- **CPOL = 1**：空闲时SCK保持高电平。

这意味着数据传输活动开始之前和结束之后，时钟线将保持在CPOL定义的电平上。

#### CPHA（时钟相位）

CPHA决定了**数据在时钟的哪一边缘被采样**（主设备读取MISO或从设备读取MOSI）：

- **CPHA = 0**：数据在时钟的第一个边缘（即SCK线的上升沿或下降沿，取决于CPOL）被采样。
- **CPHA = 1**：数据在时钟的第二个边缘（与CPHA = 0相反的边缘）被采样。

结合CPOL和CPHA，我们可以得到四种基本的SPI通信模式：

| SPI模式 | CPOL (时钟极性)     | CPHA (时钟相位) | 数据触发边缘  | 数据采样边缘 |
| ------- | ------------------- | --------------- | ------------- | ------------ |
| 模式 0  | 0 (空闲时SCK低电平) | 0 (第一个边缘)  | SCK上升沿之前 | SCK上升沿    |
| 模式 1  | 0 (空闲时SCK低电平) | 1 (第二个边缘)  | SCK上升沿之后 | SCK下降沿    |
| 模式 2  | 1 (空闲时SCK高电平) | 0 (第一个边缘)  | SCK下降沿之前 | SCK下降沿    |
| 模式 3  | 1 (空闲时SCK高电平) | 1 (第二个边缘)  | SCK下降沿之后 | SCK上升沿    |

### 通信时序

下面的四幅图显示了四种SPI模式下的通信示例。

传输的**开始和结束用绿色虚线表示**，**采样沿用橙子表示**，而**移位沿用蓝色表示**。

🟡实际使用较多的是**模式0**和**模式3**。

#### 模式0

![SPI MODE 0](https://www.analog.com/en/_/media/images/analog-dialogue/en/volume-52/number-3/articles/introduction-to-spi-interface/205973_fig_02.png)

…



### GPIO模拟





## QSPI





