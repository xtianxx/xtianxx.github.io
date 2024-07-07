[教程来源](https://www.bilibili.com/video/BV175411Y7TN/?spm_id_from=333.999.0.0&vd_source=d14c75864413529edfb05b0050105c4e)
# 环境
1，Proteus仿真[连接图](https://xtianxx.github.io/48/1.png)，vps虚拟串口，vscode编译（EIDE）。

# 串口中断方式

## 代码

> src/
┣ delay.c
┣ delay.h
┗ main.c

**main.c**
```C

#include <REG52.H>
#include "delay.h"

void UartInit(void);
void main(void)
{
    UartInit();

    while (1) {
        SBUF = 0x88;

        DelayXms(2000);
    }
}

void UartInit(void) // 9600bps@11.0592MHz
{
    PCON &= 0x7F; // 波特率不倍速
    SCON = 0x50;  // 8位数据,可变波特率
    TMOD &= 0x0F; // 设置定时器模式
    TMOD |= 0x20; // 设置定时器模式
    TL1 = 0xFD;   // 设置定时初始值
    TH1 = 0xFD;   // 设置定时重载值
    ET1 = 0;      // 禁止定时器中断
    TR1 = 1;      // 定时器1开始计时

    ES = 1; // 打开串口中断
    EA = 1; // 打开总中断
}

void uart_ISR(void) interrupt 4
{
    if (TI == 1) {
        TI = 0;
    }
}

```
**delay.c**

```C

#include"delay.h"

void DelayXms(unsigned int xms) {
    unsigned int i, j;
    for (i = 0; i < xms; i++) {
        for (j = 0; j < 124; j++);
    }
}

```
**delay.h**

```C

#ifndef __DELAY_H__
#define __DELAY_H__

void DelayXms(unsigned int xms);

#endif

```
#  查询方式(常用)
## 代码

> src/
> ┣ delay.c
> ┣ delay.h
> ┣ main.c
> ┣ uart.c
> ┗ uart.h

main.c
```C
#include <REG52.H>
#include "delay.h"
#include "uart.h"

void main(void)
{
    UartInit(); // 串口初始化

    while (1) {
        sendByte('8');
        DelayXms(2000);
    }
}

```
uart.c
```C
#include "uart.h"
#include <REG52.H>
void UartInit(void) // 9600bps@11.0592MHz
{
    PCON &= 0x7F; // 波特率不倍速
    SCON = 0x50;  // 8位数据,可变波特率
    TMOD &= 0x0F; // 设置定时器模式
    TMOD |= 0x20; // 设置定时器模式
    TL1 = 0xFD;   // 设置定时初始值
    TH1 = 0xFD;   // 设置定时重载值
    ET1 = 0;      // 禁止定时器中断
    TR1 = 1;      // 定时器1开始计时
}

void sendByte(unsigned char dat) // 串口发送一个字节    这里数据不能用data ，因为data是保留字。(慎用ai帮写)
{
    SBUF = dat;
    while (TI == 0);
    TI = 0;
}

```
delay.c
```C

#include"delay.h"

void DelayXms(unsigned int xms) {
    unsigned int i, j;
    for (i = 0; i < xms; i++) {
        for (j = 0; j < 124; j++);
    }
}

```
uart.h

```C
#ifndef __UART_H__
#define __UART_H__

void UartInit(void);

void sendByte(unsigned char dat); // 串口发送一个字节

#endif

```
delay.h
```C

#ifndef __DELAY_H__
#define __DELAY_H__

void DelayXms(unsigned int xms);

#endif

```
###发送一串字符
**在 uart.c 源文件中添加：**

```C

void sendString(unsigned char *str) // 串口发送字符串 windows中 \r\n 是换行，linux 中是 \n 。
{
    while (*str) {
        sendByte(*str);
        str++;
    }
}
```
**最后记得在uart.h中声明**

# printf串口输出重定向

## 用途

由于 mcu 没有屏幕所以需要重写 putchar 函数。因为 printf 的底层函数是它。

##代码
重写后的 putchar 函数。
同样需要在 uart.c 中定义和 uart.h 中声明
```C

char putchar(char c) // 串口发送一个字符
{
    sendByte(c);
    return c;
}

```

### 最终效果

src/
┣ delay.c
┣ delay.h
┣ main.c
┣ uart.c
┗ uart.h

## 代码

main.c
```C
#include <REG52.H>
#include "delay.h"
#include "uart.h"
#include <STDIO.H>
void main(void)
{
    unsigned char dat = 88;

    UartInit(); // 串口初始化
    while (1) {
        printf("dat1 = %bd\r\n", dat);
        //        sendString("Hello World!\r\n");
        DelayXms(2000);
    }
}

```

uart.c
```C

#include "uart.h"
#include <REG52.H>
#include <STDIO.H>
void UartInit(void) // 9600bps@11.0592MHz
{
    PCON &= 0x7F; // 波特率不倍速
    SCON = 0x50;  // 8位数据,可变波特率
    TMOD &= 0x0F; // 设置定时器模式
    TMOD |= 0x20; // 设置定时器模式
    TL1 = 0xFD;   // 设置定时初始值
    TH1 = 0xFD;   // 设置定时重载值
    ET1 = 0;      // 禁止定时器中断
    TR1 = 1;      // 定时器1开始计时
}

void sendByte(unsigned char dat) // 串口发送一个字节    这里数据不能用data ，因为data是保留字。(慎用ai帮写)
{
    SBUF = dat;
    while (TI == 0);
    TI = 0;
}

void sendString(unsigned char *str) // 串口发送字符串
{
    while (*str) {
        sendByte(*str);
        str++;
    }
}

char putchar(char c) // 串口发送一个字符
{
    sendByte(c);
    return c;

}

```

delay.c，delay.h  同上已省略

uart.h
```C
#ifndef __UART_H__
#define __UART_H__

void UartInit(void); // 串口初始化

void sendByte(unsigned char dat); // 串口发送一个字节

void sendString(unsigned char *str); // 串口发送字符串

char putchar(char c); // 串口发送一个字符

#endif

```

# 注意事项
printf 格式化输出时格式符与标准 C [不同如：](https://xtianxx.github.io/48/48-2.png) 具体需要到 c51 帮助文档中查看。