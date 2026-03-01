# 基本知识
![[Pasted image 20260227184419.png]]
![[Pasted image 20260227203108.png]]
# `GPIO`工作模式
![[Pasted image 20260227203137.png]]
![[Pasted image 20260227203147.png]]
![[Pasted image 20260227203203.png]]
![[Pasted image 20260227203218.png]]

![[Pasted image 20260227203423.png]]
![[Pasted image 20260227212911.png]]

# 开启时钟代码

```C
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC,ENABLE);
GPIO_InitTypeDef GPIO_InitStruct={0};
		
		GPIO_InitStruct.GPIO_Pin=GPIO_Pin_13;
		GPIO_InitStruct.GPIO_Mode=GPIO_Mode_Out_OD;
		GPIO_InitStruct.GPIO_Speed=GPIO_Speed_2MHz;
		GPIO_Init(GPIOC,&GPIO_InitStruct);
		GPIO_WriteBit(GPIOC,GPIO_Pin_13,Bit_SET);//写1 
		GPIO_WriteBit(GPIOC,GPIO_Pin_13,Bit_RESET);//写0
```
# `uint8_t GPIO_ReadInputDataBit`
```c
while(1)
{
	if(uint8_t GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_1)==Bit_RESET)
	{
		GPIO_WriteBit(GPIOA,GPIO_Pin_0,GPIO_Pin_SET);//亮灯
	}
	else
	{
		GPIO_WriteBit(GPIOA,GPIO_Pin_0,GPIO_Pin_RESET);//灭灯
	}
}
```
# 波特率
![[Pasted image 20260227205244.png]]
 在编程（尤其是串口通信、文本处理）中，\r是一个==**==转义字符==**==，代表 “回车”
# 获取当前时间
```c
uint32_t currentTick=GetTick();
```

# 串口
```c
Void USART_ITConfig(USART_TypeDef *USARTx,uint16 tUSART IT,FunctionalState NewState);
//串口的名称，USART1，USART2
//标志位的名称USART_IT_TXE, USART_IT_TC, USART_IT_RXNE1,USART_IT_PE, USART_IT_ERR
// 开关状态,ENABLE - 闭合 DISABLE -断开
```

```c
struct USART_InitTypeDef
{
	uint32_t USART_BaudRate;//波特率
	uint16_t USART_WordLength;// 数据位长度 USART_WordLength_8b-USART_WordLength_9b
	uint16_t USART_StopBits; //停止位长度 - USART StopBits_0_5						//USART StopBits_1 USART_StopBits_1_5 USART StopBits 2
	
	uint16_t USART_Parity;//校验方式 USART_Parity_No  USART_Parity_Even                 USART_Parity_Odd
	
	uint16_t USART_Mode;//数据收发方向
	//USART Mode Tx  USART Mode Rx  USART_Mode_Tx | USART_Mode_Rx
}
```
`USARTx_Tx`  全双工  半双工  推挽复用输出
`USARTx_Rx`  全双工  浮空输入或上拉输入
	      	半双工  未用
`USARTx_CK`   同步模式  推挽复用输出
`USARTx_RTS`  硬件流量控制  推挽复用输出
`USARTx_CTS`  硬件流量控制  浮空输入或带上拉输入
```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);// 使AFIO模块的时钟GPIO_PinRemapConfig(GPI0_Remap_USART1, ENABLE) ; // USART1_REMAP=1
GPIO_InitTypeDef GPI0_InitStruct;
//Tx PB6
RCC_APB2PeriphClockCmd (RCC_APB2Periph_GPIOB, ENABLE);
GPIO_InitStruct.GPIO_Pin = GPIO_Pin_6;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;// 模式 输出推挽
GPI0_InitStruct.GPI0_Speed = GPIo_Speed_10MHz;// 最大速度
GPIO_Init(GPIOB, &GPIO_InitStruct);
//Rx PB7 输入浮空
RCC_APB2PeriphClockCmd (RCC_APB2Periph_GPIOB, ENABLE);
GPI0_InitStruct.GPI0_Pin = GPIO_Pin_7;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;// 模式 输入上拉
GPIO_Init(GPIOB, &GPIO_InitStruct);
```
![[Pasted image 20260227224848.png]]
`TxE:Transmit Data Register Empty - 发送数据寄存器空
当`TDR`(寄存器)空时，==`TxE=1`==;否则==`TxE=0`==
`TC:Transmit Complete-发送完成`
当`TDR`空且移位寄存器为空时   ==`TC=1`==;否则==`TC=0`==
`RxNE:Receive Data Register Not Empty `- 接收数据寄存器非空
当`RDR`非空时，==`RxNE=1`==;否则==`RxNE=0`==
PE:Parity Error - 奇偶校验错
如果接收到的数据有校验错误，则PE=1;否则PE=0
![[Pasted image 20260227230034.png]]
![[Pasted image 20260227230045.png]]
![[Pasted image 20260227230054.png]]
// 使能`USART1`
`USART_Cmd(USART1,ENABLE);`
![[Pasted image 20260227231102.png]]
`void USART_SendData(USART_TypeDef *USARTx,uint16_t Data);`
//串口名称    要发送的数据
`eg:  USART SendData(USART1,0x01)`
作用:把要发送的数据写入到发送数据寄存器里
```c
// @作用:使用串口一次性发送多个字节瓶组
//@参数:pData- 要发送的数据Size -字节的数量
void My_USART_SendBytes(USART_TypeDef *USARTx, uint8_t *pData, uint16_t Size)
{
	for(uint32_t i = 0; i < Size; i++)
	{
		//#1.等待发送数据寄存器空
		while(USART_GetFlagStatus (USART1, USART_FLAG_ TXE) == RESET) ;
		// #2.将要发送的数据写入到发送数据寄存器
		USART_SendData(USARTx, pData[i]);
	}
	// #3.等待数据发送完成
	while(USART_GetFlagStatus (USART1, USART_FLAG_TC) == RESET);
}
```
从接收数据寄存器读取数据
```c
//#1.等待接收数据寄存器非空
while(USART_GetFlagStatus (USARTx, USART_FLAG_RXNE)== RESET);
// #2.接收数据
uint8_t btyeRcvd =USART_ReceiveData(USARTx);//串口名称
//#3.处理数据
```

# `I2C`
![[Pasted image 20260227234607.png]]
`SCL` 单片机发送给从机（单向）   控制数据传输快慢
`SDA` 双向
![[Pasted image 20260227234725.png]]
 寻址 0-写操作       1-读操作
 ```c
struct I2C_InitTypeDef
{
	uint32_t I2C_ClockSpeed;
	// 波特率,<=100 Sm,<=400 Fm
	uint16_t I2C_Mode;
	// 模式 I2c_Mode_I2c - 标准I2c模式
	//I2c_Mode_SMBusDevice - 系统管理总线设备模式                  I2C_Mode_SMBusHost- 系统管理总线主机模式
	uint16_t I2c_DutyCycle;
	// 快速模式下时钟信号的占空比,
	//I2c_Dutycycle_16_9
	//I2c_Dutycycle_2
	uint16_t I2CAck;//与从机模式有关
	uint16_t I2C Own Address1;//与从机模式有关
	AcknowledgedAddress;
	//用于选择10位从机地址模式
 ```
==重点==
```c
RCC_APB1PeriphclockCmd(RCC_APB1Periph_I2C1, ENABLE);//开启I2c1的时钟
RCC_APB1PeriphResetCmd(RCC_APB1Periph_I2C1, ENABLE);//施加复位信号
RCC_APB1PeriphResetCmd(RCC_APB1Periph_I2C1, DISABLE);// 释放复位信号
I2C_InitTypeDef I2C_InitStruct;
I2C_InitStruct.I2C_ClockSpeed = 400000;// 波特率400k
I2C_InitStruct.I2c_Mode = I2c_Mode_I2C;// 标准的I2c
I2c_Initstruct.I2c_Dutycycle = I2C_DutyCycle_2; // 占空2:1
I2C_Init(I2C1, &I2C_InitStruct);
I2C_Cmd(I2C1,ENABLE);//闭合I2C1的总开关
```
![[Pasted image 20260228000811.png]]
==BUSY==    总线忙标志位  0-空闲   1- 忙
起始位写法：start写1 则拉低`SDA`
停止位写法：stop写1 则拉高`SDA`
```c
//作用:通过I2C向从机发送若干个字节
int My_I2c_SendBytes(I2C_TypeDef *I2Cx,uint8_t Addr,uint8_t *pData,uint16_t Size);
//I2c接口的名称
//从机地址，靠左,
//要发送的数据
//要发送的数据的数量(字节)
返回值:0-成功   -1-寻址失败   -2-发送的数据被拒绝
```
***等待总线空闲    发送起始位   发送地址   发送数据   发送停止位***
![[Pasted image 20260228001514.png]]
![[Pasted image 20260228114633.png]]
==寻址成功要进行后续操作需将`ADDR`清零==

```c
#1.等待总线空闲
while(I2C_GetFlagstatus(I2Cx,I2C_FLAG_BUSY)== SET);
```

```c
#2 发送起始位
I2C_GenerateStart(I2Cx, ENABLE);
while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_SB) == RESET);
```

```c
//清除ADDR(先读SR1，再读SR2)
I2C_ReadRegister(I2Cx, I2C_Register_SR1);
I2C_ReadRegister(I2Cx, I2C_Register_SR2);
```
```c
#3 发送地址
//清除AF
I2C_ClearFlag(I2Cx,I2C_FLAG_AF);
//发送地址+RW#
I2C_SendData(I2Cx,ADDR&0xfe);
```

```c
#5 发送停止位
I2C_GenerateStop(I2Cx,ENABLE);
return 0;//成功
```


# `OLED`        `0x78`
```c
uint8_t commands[]=
{
	0x00,//命令流
	0x8d,0x14,//使能电荷泵
	0xaf,//打开屏幕开关
	0xa5,//让屏幕全亮
}
```
作用:通过`I2C`从从机读取若干个字节返回:0-读取成功-1-寻址失败
```c
int My_I2C_ReceiveBytes(I2C_TypeDef *I2Cx,uint8_t Addr,uint8_t *pBuffer,uint16_t Size);
//I2c接口的名称   从机地址，靠左  接收缓冲区  要发送的数据的数量(字节)

#eg
uint8_t rcvd;
My_I2C_ReceiveBytes(I2C1,0x78,&rcvd,1);
```
![[Pasted image 20260228123335.png]]
`起始位 7位地址+RW# ACK 主机接收第1字节 ACK 主机接收第n字节 停止位`
`清除ADDR  ACK=1  等待RxNE=1 读取字节1  ACK=0,STOP=1  等待RxNE=1  读取字节2`
`RxNE 是STM32串口外设的一个状态标志位(全称是"Receive Data Register Not Empty”，即“接收数据寄存器非空")`
`当串口收到数据后，RxNE 会被硬件置为1;程序中写“等待RXNE=1，意思是让程序暂停在这里，直到串口接收到数据(RXNE标志置1)，再继续执行后续代码`
**读写**
`清除ADDR   ACK=0,STOP=1   等待RxNE  读取数据`
```c
if(Size == 1)
{
	//清除ADDR  
	I2C_ReadRegister(I2Cx, I2C_Register_SR1);
	I2C_ReadRegister(I2Cx, I2C_Register_SR2);
	//向AcK写0
	I2C_AcknowledgeConfig(I2Cx, DISABLE);
	// 发送停止位
	I2C_GenerateSTOP(I2Cx, ENABLE);
	//等待RxNE置位
	while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_RXNE) == RESET);
	// 读取数据
	pBuffer[0] = I2C_ReceiveData(I2Cx);
```
==`OLED`初始化==
```c
int OLED_Init(OLED_TypeDef *OLED,//所使用的OLED的名称
OLED_InitTypeDef *OLED_InitStruct);//OLED的初始化参数

struct OLED_InitTypeDef{
//i2c写数据回调函数
int (*i2c_write_cb) (uint8_t addr, const uint8_t *pdata, uint16_t size);
//按照回调函数的形式写一个函数
//把函数名填进去
```

```C
// 要显示的字符串
int OLED_DrawString(OLED_TypeDef *OLED,const char *Str);
//透明画刷
OLED_SetBrush(&oled,BRUSH_TRANSPARENT);

OLED_SetPen(&oled,PEN_COLOR_WHITE,1);//白色画笔

OLED_SetCursor(&oled, 24,50);//光标

OLED_DrawString(&oled, "Hello world");//打印

OLED_SetFont(&oled, &hyjk16);
uint16_t x = (OLED_GetScreenWidth(&oled) - OLED_GetStrWidth(&oled, "你好世界"))/2;
OLED_Setcursor(&oled, x,28);
OLED_DrawString(&oled,"你好世界");
```
设置文本区域
```c
//@开启文本区域
void OLED_StartTextRegion(OLED_TypeDef *OLED, intl6_t X, intl6_t Y, uintl6_t Width, uintl6_t Height);

OLED_StartTextRegion(&oled, 0,0,128,64);

OLED_DrawString(&oled, "There was a man named Jhon. \r\n");
OLED_DrawString(&oled, "He was 70 years old.\r\n");

//将缓冲区发送到屏幕上
OLED_SendBuffer(&oled);

//@停止文本区域
void OLED_StopTextRegion (OLED_TypeDef *OLED);
```

```c
OLED_SetPen(&oled, PEN_COLOR_WHITE, 3); // 画笔3
OLED_Setcursor(&oled, 29,32);//光标移动到第一个点
OLED_DrawDot(&oled);// 画点
//画剩下的7个点
for(uint32_t i=1;i<8;i++)
{
	OLED_MovecursorX(&oled，10);//光标左移10像素
	OLED_DrawDot(&oled);// 画点
}


//矩形
OLED_SetCursor(&oled, 20, 20);
OLED_SetPen(&oled, PEN_COLOR_WHITE, 1);
OLED_SetBrush(&oled, BRUSH_TRANSPARENT);
OLED_DrawRect (&oled, 40, 20);//画矩形  长  宽

//圆
OLED_SetCursor(&oled, 65, 30);
OLED_DrawCircle(&oled, 5);//半径

```

## 软件`I2C`
写
```c
int My_SI2c_SendBytes(uint8_t Addr,uint8_t*pData,uint16_t Size);
//从机地址，靠左  要发送的数据  数据的数量(字节)
```
读
```c
int My_SI2c_ReceiveBytes(uint8_t Addr,uint8_t*pBuffer,uint16_t Size);
//从机地址,靠左   接收缓冲区  数据的数量(字节)
```
代码
```c
#include“si2c.h”
SI2C_TypeDef si2c;//声明一个软i2c的变量
si2c.SCL_GPIOx=GPIOB;
si2c.SCt GPIO_Pin = GPIO_Pin_6;
si2c.SDA_GPIOx=GPIOB;
si2c.SDA_GPIO_Pin = GPIO_Pin_7;
SI2C_Init(&si2c);//初始化
```

# 按钮
![[Pasted image 20260228124312.png]]
代码
```c
int main(void)
{
	uint8_t previous = Bit SET, current = Bit SET;
	while(1)
	{
		previous =current;//保存上次按钮的状态
		current = GPIO ReadInputDataBit(...);
		if(previous != current)
		{
			if(current == Bit_SEI)// 按钮松开
			{
			// eg 改变LED的亮灭状态  用回调函数
			}
			else{}//按钮按下  用回调函数
		}
	}
}
```
初始化按钮
```c
void My_Button_Init(Button_TypeDef *Button, Button_InitTypeDef *Button_InistStruct);

struct GPIO_InitTypeDef
{
	GPIO_TypeDef *GPIOx;
	uint16_t GPIO_Pin;
	uint32_t LongPressTime;//长按的时间阈值，单位毫秒，0表示默认1000
	uint32_t LongPressInterval;
	//长按后持续触发的时间间隔，0表示默认(100)
	uint32_t ClickInterval;// 连击的最大时间间隔，0表示默认(200)
	//回调函数，没有的话填0
	void(*button_pressed_cb)(void);//回调函数 -按钮按下
	void(*button_released_cb)(void);//回调函数-按钮抬起
	void(*button_clicked_cb)(uint8_t clicks);//回调函数 - 按钮点击
	void(*button_long_pressed_cb)(uint8_t ticks);// 回调函数 - 按钮长按
}
Button_InitStruct.button_click_cb = button_click_cb;
My_Button_Init(&button, &Button_InitStruct);
//回调函数
void button_click_cb(uint8_t clicks)
{
	if(clicks == 1)
	{
	// 切换按钮的亮灭状态
	}
}
```

```c
#include "button.h"
Button_TypeDef button;//声明一个按钮
void App_Button_Init(void);
int main(void)
{
	App_Button_Init();//初始化
	while(1)
	{
	    My_Button_Proc(&button);//进程
	}
}
void App_Button_Init(void)
{
	Button_InitTypeDef Button_InitStruct = {0};
	Button_InitStruct.GPIOx = GPIOA;
	Button_InitStruct.GPIO_Pin = GPIO_Pin_0;
	My_Button_Init(&button, &Button_InitStruct);
}
```

# `spi`
![[Pasted image 20260228150844.png]]
`DI(Slave DataInput MOSI) - 接主机MOSI`
`DO(Slave Data Output MISO) - 接主机MISO`
`CLK(Serial Clock SCK) - 接主机的SCK` 
![[Pasted image 20260228140924.png]]
![[Pasted image 20260228140944.png]]
 

`PB3 -> SCK       复用输出推挽  2MHz`
`PB4->  MISO    输入上拉`
`PB5 -> MOSI    复用输出推挽  2MHz`
`PA15->普通IO  通用输出推挽   2MHz`


![[Pasted image 20260228140541.png]]
==`spi`初始化==
```c
void SPI_Init(SPI_TypeDef *SPIx,SPI_InitTypeDef *SPI_Initstruct); 
```

```c
struct SPI_InitTypeDef
{
	uint16_t SPI_Direction;//用来选择SPI通信的方向
	//-  SPI_Direction_2Lines_FullDuplex 2线全双工
	//-  SPI_Direction_2Lines_Readonly   2线只读
	//-  SPI_Direction_1Line_Rx          单线接收
	//-  SPI_Direction_1Line_Tx          单线发送

	uint16_t SPI_Mode;//用来选择sPI的模式，
	//SPI_Master-主机，SPI_Slave -从机
	uint16_t SPI Datasize; // 数据宽度
    // SPI Datasize 8b - 8bit, SPI Datasize 16b - 16bit
	uint16_t SPI_CPOL;//时钟的极性
	uint16_t SPI_CPHA;//时钟的相位
	uint16_t SPI_NSS;// 软件NSS/硬件NSS
	uint16_t SPI_BaudRatePrescaler;//用来选择波特率分频器的分频系数
	uint16_t SPI_FirstBit;//比特位的传输顺序,SPI_FirstBit_MSB
}
```

```c
void App_SPI_MasterTransmitReceive
(SPITypeDef *SPIx,//SPI的名称
const uint8_t *pDataTx,//要发送的数据
uint8_t *pDataRx, //接收到的数据
uint16_t Size); // 收发数据的数量
```
==`TXE`:发送数据寄存器为空  `RXNE`:读取数据寄存器为空==
使用`SPI`总线收发数据
```c
void App_SPI_MasterTransmitReceive(SPI_TypeDef *SPIx,const uint8_t *pDataTx, uint8_t *pDataRx, uint16_t Size)
{
	SPI_Cmd(SPIx，ENABLE);//#1.闭合总开关
	SPI_I25_SendData(SPIx，pDataTx[e]);//#2.发送第一个字节
	for(uint16_t i=0; i<Size-1; i++)
	{
		//发送一个字节
		while(SPI_I2S_GetFlagStatus(SPIx, SPI_I2S_FLAG_TXE) == RESET);
		SPI_I2s_SendData(SPIx, pDataTx[i+1]);
		// 接收一个字节
		while(SPI_I2S_GetFlagStatus(SPIx,SPI_I2S_Flag_RXNE==RESET);
		pDataRx[i] = SPI_I2S_ReceiveData(SPIx);
	}
	//#4 读出最后一个字节
	while(SPI_I2S_GetFlagStatus(SPIx, SPI_I2S_FLAG_RXNE) == RESET);
	pDataRx[Size-1] = SPI_I2S_ReceiveData(SPIx);
	//#5 断开总开关
	SPI_Cmd(SPIx, DISABLE);
}
```

==`W25Q64`==
```c
void App_W25Q64_SaveByte(uint8_t Byte); //使用W25Q64保存一个字节
uint8_t App_W25Q64_LoadByte(void);  //把保存的字节读出来
```
写使能  扇区擦除  等待空闲 |  写使能  页编程  等待空闲
==#1.写使能==     主机发`0x06`
```c
uint8_t buffer[10];//声明一个数组
buffer[0] = 0x06;
GPIO_WriteBit(GPIOA,GPIO_Pin_15,Bit_RESET); //NSS=0
App_SPI_MasterTransmitReceive(SPI1, buffer, buffer,1);
GPIO_WriteBit(GPIOA, GPIO_Pin_15,Bit_SET); //NSS=1
```
==#2.扇区擦除==   主机发`0x20+24位地址`
```c
buffer[0] = 0x20;
buffer[1] = 0x00;
buffer[2] = 0x00;
buffer[3] = 0x00;
GPIO_WriteBit(GPIOA, GPIO_Pin_15, Bit_RESET); //NSS=0 
App_SPI_MasterTransmitReceive(SPI1, buffer, buffer,4);
GPIO_WriteBit(GPIOA, GPIO_Pin_15, Bit_SET); // NSS=1
```
==#3.等待空闲==    主机发`0x05  然后再收一个字节`
```c
while(1)
{
	GPIO_WriteBit(GPIOA, GPI0_Pin_15, Bit_RESET); // NSS=0
	//写0x05
	buffer[0] = ex05;
	App_SPI_ MasterTransmitReceive(SPI1, buffer, buffer, 1);
	//读一个字节
	buffer[e] = 0xff;
	App_SPI_MasterTransmitReceive(SPI1, buffer, buffer, 1);
	GPIO_WriteBit(GPI0A, GPI0_Pin_15, Bit_SET); // NSS=1
	//如果BUSY=0则退出等待
	if((buffer[0] & 0x01)==0)
	break;
}
```
==#5 页编程==  `主机发0x02 +24位地址 +要写的数据`
```c
buffer[0]=0x02;
buffer[1]=0x00;
buffer[2]=0x00;
buffer[3] =0x00;
buffer[4] = Byte;
GPIO_WriteBit(GPIOA, GPIO_Pin_15, Bit_RESET); //NSS=0 
App_SPI_MasterTransmitReceive(SPI1, buffer, buffer, 5);
GPIO_WriteBit(GPIOA, GPIO_Pin_15, Bit_SET); // NSS=1
```
==读取==   `主机发0x03+24位地址，然后读取数据`
```c
//读取一个字节
buffer[0] = 0x02; 
buffer[1] = 0x00;
buffer[2] = 0x00; 
buffer[3] = 0x00;
GPIO_WriteBit(GPIOA, GPIO_Pin_15, Bit_RESET);// NSS=0
App_SPI_MasterTransmitReceive(SPI1, buffer, buffer, 4);// 发0x03+24位地址
App_SPI_MasterTransmitReceive(SPI1, buffer, buffer, 1);// 收-个字节
GPIO_WriteBit(GPIOA, GPIO_Pin_15, Bit_SET); // NSS=1
return buffer[0];
```

![[Pasted image 20260228142250.png]]
![[Pasted image 20260228142306.png]]
![[Pasted image 20260228142423.png]]

# 中断

![[Pasted image 20260228170407.png]]
![[Pasted image 20260228154017.png]]
![[Pasted image 20260228154025.png]]
![[Pasted image 20260228165844.png]]

![[Pasted image 20260228171340.png]]抢占优先级数越小 中断优先级越高
中断排队（优先级相仿 不打断原来优先级）
```c
uint32_t blinkInterval = 1000;// 闪灯间隔
int main(void)
{
	while(1)
	{
		GPIO_WriteBit(..., Bit_RESET);//亮灯
		Delay(blinkInterval); // 延迟
		GPIO_WriteBit(...,Bit_SET);//灭灯
		Delay(blinkInterval); // 延迟
	}
}	
	void USART1_IRQHandler(void)
	{
		//中断响应函数
		uint8_t byte = USART_ReceiveData(...);
		if(byte == ‘0’) blinkInterval = 1000;//慢
		if(byte == ‘1’) blinkInterval = 200;//中
		if(byte == '2') blinkInterval = 50;//快
	}
```

```c
void NVIC_Init(NVIC_InitTypeDef *NVIC_InitStruct);//初始化

void NVIc_Init(NVIC_InitTypeDef *NVIC_InitStr
struct I2C_InitTypeDef
{
	//中断的名称，见stm32f10x.h IRQn
	uint8_t NVIC_IRQchannel;
	// 抢占优先级
	uint8_t NVIC_IRQChannelPreemptionPriority;
	//子优先级
	uint8_t NVIC_IRQChannelSubPriority;
	//开关状态
	FunctionalState NVIC_IRQChannelCmd;
}
```
==*EXIT*==
作用:使用AFIO模块为EXTI的线选择IO引脚
```C
void GPIO_EXTILineconfig(uint8_t GPI0_PortSource,uint8_t GPI0_PinSource); // 端口号 - GPIO_PortSourceGPIOA, GPIO_PortSourceGPIOB GPIO_PortSourceGPIOC, GPIO_PortSourceGPIOD 
//线编号 -  GPI0_PinSource0 ~ GPI0_PinSource15
```

```C
开启AFI0的时钟
RCC_APB2PeriphCLockCmd(RCC_APB2Periph_AFIO, ENABLE);
//线5选择端口A
GPIo_EXTILineConfig(GPIO_PortSourceGPIOA, GPIO_PinSource5);
//线6选择端口A
GPIO_EXTILineConfig(GPIO_PortSourceGPIOA, GPIO_PinSource6);
```
==用来初始化`EXTI`的一条线==
```C
void EXTI_Init(EXTI_InitTypeDef *EXTI_InitStruct);

struct EXTI_InitTypeDef
{
	uint32_t EXTI_Line;//线编号 EXTI_Line0~EXTI_Line19
	EXTIMode_TypeDef EXIT_Mode;
	//选择模式 -EXTI_Mode_Interrupt-中断
	//-EXTI_Mode_Event-事件
	EXTITrigger_TypeDef EXTI_Trigger; 
	// 边沿 EXTI_Trigger_Rising - 上升沿
	//EXTI_Trigger_Falling - 下降沿
	//EXTI_Trigger_Rising_Falling - 双边沿
	FunctionalState EXTI_LineCmd;//开关ENABLE-闭合，DISABLE-断开
}
```
==获取`EXTI`线的标志位的值软件触发==
```c
FlagStatus ExTI_GetFlagStatus(uint32_t EXTI_Line);


void EXTI9_5_IRQHandler(void)
{
	if(EXTI_GetFLagStatus(EXTI_Line5) == SET)// 线5触发的中断
	GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET); // 亮灯
	
	if(EXTI_GetFlagStatus(EXTI_Line6)== SET)//线6触发的中断
	GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET); // 灭
}
```
==清除`EXTI`线标志位==
```c
void EXTI_ClearFlag(uint32_t EXTI_Line);
```

# 时钟树
![[Pasted image 20260228172422.png]]
  带颜色则可开关 灰色表示默认状态下是关闭的 绿色表示默认状态下是开启的
开启enable 关闭disable
分频器-对频率做除法
锁相环-对频率做乘法
复用器-对频率做选择

![[Pasted image 20260228172216.png]]
![[Pasted image 20260228174707.png]]


`RCC`   复位和时钟控制器
```
void RCC_HSEConfig(uint32_t RCC_HSE); // HSE开关
void RCC_HSICmd(FunctionalState NewState); // HSI开关
void RCC_PLLConfig(uint32_t RCC_PLLSource, uint32_t RCC_PLLMul);
void RCC_PLLCmd(FucntionalState NewState); // PLL开关
void RCC_SYSCLKconfig(uint32_t RCC_SYSCLKSource); // SYSCLK未源
void RCC_HCLKconfig(uint32_t RCC_SYSCLK);// 配置HCLK
void RCC_PCLKIConfig(uint32_t RCC_HCLK); // 配置PCLK1
void RCC_PCLK2config(uint32_t RCC_HCLK); // 配置PCLK2
FlagStatus RCC_GetFlagStatus(uint8_t RCC_FLAG); // 获取RCC状态
uint8_t RCC_GetsYscLKSource(void);// 获取SYSCLK的来源
```
```c
//HSE开关
void RCC_HSEConfig(uint32_t RCC_HSE); //RCC_HSE_ON - 开 RCC_HSE_OFF 关
 //获取Rcc的状态
FlagStatus RCC_GetFlagStatus(uint8_t RCC_FLAG); //RCC_FLAG_HSERDY - HSE就绪

//#1.开启HSE
RCC_HSEConfig(RCC_HSE_ON);//开启HSE
while(RCC_GetFlagStatus(RCC_FLAG_HSERDY) == RESET);// 等待HSE就绪
```
配置锁相环的参数 
```c
// @参数 RCC_PLLSource 选择锁相环的输入 
//   RCC_PLLSource_HSE_Div1    HSE 
//   RCC_PLLSource_HSE_Div2    HSE/2 
//   RCC_PLLSource_HSI_Div2    HSI/2 
void RCC_PLLConfig(uint32_t RCC_PLLSource, uint32_t RCC_PLLMul);
// 控制锁相环的开关
void RCC_PLLCmd(FunctionalState NewState);
// ENABLE - 开 DISABLE - 关 
// 获取RCC的状态 
FlagStatus RCC_GetFlagStatus(uint8_t RCC_FLAG); 
// RCC_FLAG_PLLRDY - PLL就绪 

// #2. 配置锁相环 
RCC_PLLConfig(RCC_PLLSource_HSE_Div1, RCC_PLLMul_9); // HSE * 9
RCC_PLLCmd(ENABLE); // 开启PLL 
while(RCC_GetFlagStatus(RCC_FLAG_PLLRDY) == RESET);// 等待PLL就绪
```

```c
// 设置AHB分频器的分频系数 
void RCC_HCLKConfig(uint32_t RCC_SYSCLK); // RCC_SYSCLK_Div1 .. 512 
// 设置APB1分频器的分频系数 
void RCC_PCLK1Config(uint32_t RCC_HCLK); // RCC_HCLK_Div1 .. 16 
// 设置APB2分频器的分频系数 
void RCC_PCLK2Config(uint32_t RCC_HCLK); // RCC_HCLK_Div1 .. 16
 
// #3. 配置AHB分频器、APB1分频器、APB2分频器
RCC_HCLKConfig(RCC_SYSCLK_Div1); // HCLK = SYSCLK / 1
RCC_PCLK1Config(RCC_HCLK_Div2); // PCLK1 = HCLK / 2
RCC_PCLK2Config(RCC_HCLK_Div1); // PCLK2 = HCLK / 1
```
设置`SYSCLK`的来源
```c
// @参数 RCC_SYSCLKSource 选择SYSCLK的来源 
//RCC_SYSCLKSource_HSI 
//RCC_SYSCLKSource_HSE 
//RCC_SYSCLKSource_PLLCLK 
void RCC_SYSCLKConfig(uint32_t RCC_SYSCLKSource); 
// 获取SYSCLK的来源 
// @返回值 0x00 - HSI
// 0x04 - HSE
//0x08 - 锁相环 
uint8_t RCC_GetSYSCLKSource(void); 
// #4. 选择SYSCLK的来源
RCC_SYSCLKConfig(RCC_SYSCLKSource_PLLCLK); // SYSCLK来自锁相环
while(RCC_GetSYSCLKSource() != 0x08); // 等待切换完成
```


# 定时器
![[Pasted image 20260228174841.png]]

![[Pasted image 20260228174906.png]]
![[Pasted image 20260228174911.png]]
![[Pasted image 20260228175200.png]]
![[Pasted image 20260228174918.png]]

![[Pasted image 20260228175452.png]]


// 记录当前的时间，单位ms 
```c
volatile uint32_t currentTick = 0; 
// @简介: 延迟一段时间 
// @参数 ms: 要延迟的时间 (单位ms)  
void App_Delay(uint32_t ms) 
{ 
	uint32_t expireTime = currentTick + ms; // 延迟结束的时间
	while(currentTick < expireTime); // 等待延迟结束 
}
```

==使能/禁止定时器的中断==
```c
// @参数:TIM_IT 中断的名字 TIM_IT_Update, TIM_IT_Trigger, TIM_IT_CC1,...
// @参数:NewState 使能/禁止   ENABLE-使能，DISABLE-禁止
void TIM_ITConfig(TIM_TypeDef *TIMx, uint16_t TIM_IT, FunctionalState NewState);
```
# 输出比较
==`CNT（计数寄存器）`计数时   持续和`CCR`中预设的 “目标值” 做比较==
**ARR（自动重装载寄存器）只是用来限定 `CNT` 的计数范围**


# ADC
连续变化的模拟信号变为离散数字信号（0、1 组成的二进制数）
![[cdadcbb963938adbdb36f30dc6a66259.jpg]]
![[11d98efe4f06b2a9babe1fa1d8a787a8.jpg]]

![[Pasted image 20260301154635.png|504]]
![[2787ceaad9ed0cca31abfb29573e00df.jpg]]

```c
ADC_InitTypeDef ADC_InitStruct = {0};

// 连续模式
ADC_InitStruct.ADC_ContinuousConvMode = ;
//对齐方式(左/右)
ADC_InitStruct.ADC_DataAlign = ;
// 选择常规序列的外部触发信号
ADC_InitStruct.ADC_ExternalTrigConv = ;
//双ADC模式
ADC_InitStruct.ADC_Mode = ;
// 常规序列的通道数
ADC_InitStruct.ADC_NbrOfChannel = ;
// 扫描模式
ADC_InitStruct.ADC_ScanConvMode = ;

ADC_Init(ADC1, &ADC_InitStruct);

```

```c
//设置分频器的分频系数(6分频)
RCC_ADCCLKConfig(RCC_PCLK2_Div6);
//使能ADC1的时钟
RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1,ENABLE);
```
![[90d899bf9577800f7408c22cf530328c.jpg]]

`CCR`  捕获 / 比较寄存器
定时器的计数寄存器`（CNT）`会从 0 开始递增计数，当` CNT` 的值等于` CCR` 中预先设定的值时，单片机就会触发一个动作（比如翻转 IO 口电平、产生中断、启动 `DMA` 等
通过修改` CCR `的值，改变 `PWM `的占空比；


```c
//读取CCR1的值
自动重装寄存器ARR
uint16_t ccr1 = TIM_GetCapture1(TIM1);
//读取CCR2的值
uint16_t ccr2 = TIM_GetCapture2(TIM1);
// 计算距离=(ccr2-ccrl)*分辨率*声速÷2
float distance = (ccr2 - ccrl) * 1.0e-6f * 340.0f / 2;
//打印结果
My_USART_Printf(USART1, "%.3f\n", distance * 100) ;
//延迟
Delay(100);
```

```c
//初始化输入捕获
void TIM_ICInit(TIM_TypeDef* TIMx, TIM_ICInitTypeDef* TIM_ ICInitStruct);
//设置CCR1的值
void TIM_SetCompare1(TIM_TypeDef* TIMx, uint16_t Compare1);
```
![[013d9e789b1e36c7df6a481ae1935a3b.jpg]]
![[7b2d3e275b3895f06008826a5825f198.jpg]]

![[Pasted image 20260301181125.png]]
```c
// @简介:初始化输出比较通道1的参数
//@参数:TIM_OCInitStruct-初始化参数列表
void TIM_OC1Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);

//@简介:闭合/断开MOE开关
//@参数:NewState ENABLE- 闭合MOE,       DISABEL - 断开MOE
void TIM_CtrlPWMoutputs(TIM_TypeDef* TIMx, FunctionalState NewState);
```