# 多功能计算器 设计介绍



## 功能概述

* 计算功能

>1. 实现 8 位<font color="red">整数</font>、<font color="red">浮点数</font>的 “+”、“-”、“*”、“/”计算，计算结果输出为 16 位。可以使用 （）运算符。
>2. 显示采用 DS1602上行显示输入信息，下行显示计算结果或错误输出。
>3. 按清除键可以清屏。

* 待机功能

>无按键操作超过2min自动进入待机状态，在显示屏上输出日期时间温度相关信息，其中时间精确到分钟，温度每10min刷新一次。
>
>其中时间采用 DS1302 芯片，温度采集采用 DS18B20 芯片。

* ~ 记忆数据功能（考虑）

> 将计算结果保存的功能。采用 AT24C02 芯片，I2C通信。



## 设计介绍

​		本部分介绍相关函数及其设计规范，开发板使用普中科技的89C52开发板，型号为HC6800-ES V2.0。

### 设计规范说明

#### I/O口功能规范

##### MCU I/O定义图

<img src="https://cdn.nlark.com/yuque/0/2021/png/22945577/1637126760921-fe6172fb-8eed-4c92-8f25-0989af76c053.png?date=1637126915043" alt="avatar" style="zoom:80%;" />

##### KEY （矩阵键盘）

<img src="https://cdn.nlark.com/yuque/0/2021/png/22945577/1637127323521-7f09ea8b-c06b-442c-b27c-24ba9036f4df.png" alt="avatar" style="zoom:50%;" />

> 从S1至S16按键功能依次为：
>
> * 1，2，3，+，4，5，6，—，7，8，9，*，.  （小数点） ，0，=  ，/（除号）

##### KEY（独立键盘）

<img src="https://cdn.nlark.com/yuque/0/2021/png/22945577/1637127336137-0258e18c-db81-40a1-b9dc-3a2b4bd3ba36.png" alt="avatar" style="zoom: 67%;" />

> 从K1至K4按键功能依次为：
>
> * （ （左括号）
> *   ）（右括号）
> *   AC（清屏）
> *   M（保存数据）

##### KEY（复位键）

![avatar](https://cdn.nlark.com/yuque/0/2021/png/22945577/1637127374410-ac8edd7d-b12c-442b-9fe0-e739174aa567.png)

##### LCD1602

<img src="https://cdn.nlark.com/yuque/0/2021/png/22945577/1637127354845-241a09eb-c60d-40c6-adef-8475327f06dd.png" alt="avatar" style="zoom:50%;" />

##### DS18B20

![avatar](https://cdn.nlark.com/yuque/0/2021/png/22945577/1637127385477-e1332dd7-1092-47ed-8821-cc7d200f6c86.png)

##### 晶振

![avatar](https://cdn.nlark.com/yuque/0/2021/png/22945577/1637127400036-88f45406-766a-4065-928e-ef284064e218.png)





#### 命名规范

* 函数采用二段式命名规则，其中第一段代表函数实现功能大类，即C文件名称，第二段代表函数实现功能，其中首字母必须小写。

> 示例：void core_count(void)

* C语言文件和对应头文件名称必须完全一致，并以函数对应功能命名，其中首字母必须小写。

> 示例：delay.c

* 功能描述必须清晰，做到见文知功能，采用驼峰命名法。

> 示例：void core_countFloat(void)



### 函数介绍及设计要求

​		该部分介绍各大类函数及其对应输入输出要求。

#### 函数介绍

​		大类函数分为以下几个部分：

  		1. 主函数 main.c
  		2. 调度核心 core.c
  		3. 输入函数 input.c
  		4. 显示函数 display.c
  		5. 计算函数 calc.c
  		6. 延时函数 delay.c
  		7. 时钟函数 time.c
  		8. 温度采集函数 temper.c
  		9. ~ 数据记忆函数 memory.c
  		10. ~ I2C函数 I2C.c



#### 设计要求

##### 全局变量

char maxLen[1024];

##### 主函数 main.c

> 该部分要求仅包含主函数、core函数和中断函数内容。
>
> 中断采用定时器中断，中断函数中包含输入检测、温度采集和显示模块的中断调用。

##### 调度核心 core.c

> * 输入数据和计算结果通过core函数显示到屏幕上
> * 接收按键信息并进行相应处理（对单个字符的拼接等）
> * 将拼接的字符串刷新至显示屏上
> * 接收其它函数的返回信息，做相应处理后显示在显示屏相应位置

##### 输入函数 input.c

> 按下相应按键后向core函数返回字符值，返回类型为 unsigned char
>
> 1. 数字，+，-，*，/ ，. ，(，) 返回相应字符
> 2. =  返回字符A（answer）
> 3. AC 返回字符C（clear）

##### 显示函数 display.c

> 通过core调用相应的显示函数显示字符串和错误信息

##### 计算函数 calc.c

> core接收到字符A后，调用该函数进行计算，并将计算结果返回给core，返回类型为 unsigned char*（也可以将需返回的字符串定义为长度为1024的全局变量 char answer[1024]）

##### 延时函数 delay.c

> 该函数要求延时任意毫秒数

##### 时钟函数 time.c

> 要求读取时钟芯片DS1302中的时、分、秒，并返回给core，返回格式为 HH-MM-SS，unsigned char* 型；
>
> 暂不要求设置时间，初始化默认一个时间

##### 温度采集函数 temper.c

> 要求获取温度芯片的温度，转换为摄氏度类型后，返回给core，返回格式为 unsigned char 型；

##### 显示器内容规范

<img src="https://cdn.nlark.com/yuque/0/2021/png/22945577/1638782337138-d5171fa8-8ad2-48e3-b35e-86a4e59711fb.png"></img>







