---
layout: post
title:  "什么是字节序"
date:   2018-06-24 22:14:54
categories: byte
tags: Byte-Order
---

* content
{:toc}

在不同的计算机体系结构中，对于数据（比特、字节、字）等的存储和传输机制有所不同，因而引发了计算机领域中一个潜在但是又很重要的问题，即通信双方交流的信息单元应该以什么样的顺序进行传送。如果达不成一致的规则，计算机的通信与存储将会无法进行。
目前在各种体系的计算机中通常采用的字节存储机制主要有两种：大端（Bin-endian）和小端（Little-endian）。






### 例子
假设用户申请了一个2个字节的内存，要存入一个数字15（该数字类型为int16，二进制表示为：00000000 00001111）

![stack_order]({{site.url}}/images/stack_order.png)

### 定义
* 大端（Big Endian）: 高位地址存放在低位的内存地址
* 小端（Little Endian）: 高位址存放在高位的内存地址

### 网络字节序
TCP/IP协议规定使用“大端”字节序为网络字节序，某些不使用大端字节序的计算器就要在发送数据之前把字节序转成大端的字节序，接收的时候再把大端的字节序转小小端的字节序

* 将2个字节的数字15转成大端序

```go
buf := make([]byte, 2)

buf[0] = byte(15 >> 8)
buf[1] = byte(15 >> 0)
// 结果：buf = {0, 15}
```
* 将2个字节的数字15转成小端序

```go
buf := make([]byte, 2)

buf[0] = byte(15 >> 0)
buf[1] = byte(15 >> 8)
// 结果： buf = {15, 0}
```
* 将大端序的2字节转换成值

```go
buf := []byte{0, 15}

val := uint16(buf[0] << 8) | uint16(buf[1] << 0)
// 结果： val = 15
```
* 将小端序的2字节转换成值

```go
buf := []byte{15, 0}

val := uint16(buf[0] << 0) | uint16(buf[1] << 8)
// 结果： val = 15
```

### 判断机器的字节序

```c
#include <stdio.h>
	   
int main() {
   unsigned short x = 0x000F;
   char *c = (char*)&x;

   if (*c == 0x0F) {
       printf("Little Endian\n");
  } else {
      printf("Big Endian\n");
  }

  return 0;
}
```