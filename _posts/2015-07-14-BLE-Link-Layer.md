---
layout: post
category: "蓝牙"
title: "蓝牙BLE  LL"
tags: ["蓝牙,BLE"]
---

**以下文中所有数据格式为小端模式，即最左侧为LSB，最右侧为MSB**
### 设备地址DEVICE　ADDRESS

每个设备都有地址，地址长6字节，分为公共地址（ public device address）和随机地址（random device address）两类。设备可以同时拥有这两种类型的地址。

## 空中接口

### 空中数据包格式

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |


|Preamble|Access Address|PDU|CRC|
|--|--|--|--|
|1字节|4字节|2-39字节|3字节|



#### 前导码（Preamble）

LL层所有数据包都有一个8bit前导码，接收机用此码做同步、符号定时估计、自动增益控制。
**广播信道**包前导码为10101010b 。
**数据信道**包前导码有两种，如果接入地址（Access
Address）的LSB为1，则前导码为01010101b。否则前导码为10101010b

#### 接入地址（Access Address）

广播信道接入地址是规定好的，地址为10001110100010011011111011010110b (0x8E89BED6) 。
对数据信道而言，设备间的每次连接都就使用不同的接入地址。在 Initiating State状态时，设备就产生一个32bit数做为接入地址。此地址包含在CONNECT_REQ命令的数据中。

#### 数据单元（PDU）
数据单元分两种，广播信道PDU和数据信道PDU。

### 广播信道PDU

其格式如下：

|Header|Payload|
|-|-|
|2字节|Header中说明的长度|

广播信道PDU的Header格式如下：

|PDU Type|RFU|TxAdd|RxAdd|Length|RFU|
|-|-|-|-|
|4 bits|2 bits|1 bit|1 bit|6 bits|2 bits|

**PDU Type** 分以下几种类型：
|PDU Type<br> b<sub> 3 </sub>b<sub> 2</sub> b <sub>1</sub> b <sub>0</sub>|Packet Name|
|-||||||
|0000|ADV_IND|
|0001|ADV_DIRECT_IND|
|0010|ADV_NONCONN_IND|
|0011|SCAN_REQ|
|0100|SCAN_RSP|
|0101|CONNECT_REQ|
|0110|ADV_SCAN_IND|
|0111-1111|Reserved|

#### 广播信道PDU说明
The following advertising channel PDU Types are called advertising PDUs and
are used in the specified events:
• ADV_IND: connectable undirected advertising event
• ADV_DIRECT_IND: connectable directed advertising event
• ADV_NONCONN_IND: non-connectable undirected advertising event
• ADV_SCAN_IND: scannable undirected advertising event
These PDUs are sent by the Link Layer in the Advertising State and received
by a Link Layer in the Scanning State or Initiating State.

### 数据信道PDU

其格式如下：

|Header|Payload|MIC|
|-|-|-|
|2字节|Header中说明的长度|4字节|

数据信道Header格式如下：
|LLID|NESN|SN|MD|RFU|Length|RFU|
|-|-|-|-|-|
|2 bits|1 bits|1 bit|1 bit|3 bits|5 bits|3 bits|
