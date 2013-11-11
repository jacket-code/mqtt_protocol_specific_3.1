MQTT 3.1 protocol specific
===================================
MQ(Message Queue) Telemetry Transport(MQTT)是一个轻量级的基于经纪的发布/订阅消息协议，  
设计原则是开放、简单，并且容易实现。这些特点使MQTT协议成为有限的环境下的比较完美的解决方案，  
有限的环境可能有：
>1.在网络费用比较高的地方，只有较低的带宽或者网络不稳定  
>2.在嵌入式设备中，性能有限的处理器和内存资源  

##协议特点包括：
>1.发布/订阅的消息模式提供了一种消息一对多的分发和与应用程序解耦的机制  
>2.对传递的内容是不可知的一种消息传输方式  
>3.使用TCP/IP协议去提供基础的网络连接  
>4.三个层次的消息分发服务：

>>最多一次，消息发布完全依赖底层 TCP/IP 网络。会发生消息丢失或重复。这一级别可用于如下情况，环境传感器数据，丢失一次读记录无所谓，因为不久后还会有第二次发送。  
>>"至少一次"，确保消息到达，但消息重复可能会发生。  
>>"只有一次"，确保消息到达一次。这一级别可用于如下情况，在计费系统中，消息重复或丢失会导致不正确的结果。

>5.小型传输，开销很小（固定长度的头部是 2 字节），协议交换最小化，以降低网络流量。  
>6.使用 Last Will 和 Testament 特性通知有关各方客户端异常中断的机制。  

#目录
##1.引言
包括三个小节：
>1.对所有类型的数据包通用的消息格式  
>2.每种数据包的消息内容的具体信息  
>3.数据包是如何在客户端和服务器之间流转  
在附录中提供了主题是如何使用通配符的介绍。

###1.1 与V3版本的改变
下面是MQTT V3 和MQTT V3.1之间的变化：
>1.用户名和密码现在可以用一个连接数据包来发送  
>2.新的返回代码在"CONNACK"数据包，提高安全性  
>3.客户端将不会收到没有经过授权的发布/订阅命令，并且MQTT流程必须完成不管命令有没有被执行  
>4.消息中的字符串新增UTF-8的支持，不仅仅支持US-ASCII  

协议的版本号通过“CONNECT”数据包，仍然停留在3.x版本。MQTT V3的服务器可以接受V3.1的客户端的请求，只要请求中正确处理"剩余长度"字段，并且因此忽略了额外的安全问题。

##2.消息格式
每条MQTT命令消息都包含一个固定的头。有些消息也需要可变的头和有效载荷。下面的章节描述了消息头的每个部分：

###2.1固定头
每条MQTT命令消息都包含一个固定的头。下面表格展示了固定头的格式：
| 位            | 7       | 6       | 5       |   4     |          3           | 2        |1       | 0          |
| --------      | -----:  | :----:  | -----:  | :----:  | -----:               | :----:  | :----:  | :----:      |
| 字节1      |              消息类型          ||||   DUP flag     |Qos level    || RETAIN|
| 字节2      |                                            剩余内容                                                      |

###第一个字节
    包括消息类型和标识（DUP，QoS level， RETAIN）字段
###第二个字节
    （至少一个字节）包括剩余的信息字段

所有的字段在下面的章节描述。所有的数据排序规则是顺序值越大越优先。1个16bit的字母出现在最重要的字节，后面是最不重要的字节。

###1.消息类型
位置：第一个字节，7-4bits
是1个4bit的无符号值。
Mnemonic|Enumeration|Description
---------------| ----------------: |:--------------|
Reserved  |         0            |Reserved   |
CONNECT |         1            |Client request to connect to Server|
CONNACK |        2            |Connect Acknowledgment|
PUBLISH    |         3            |   Publish message |
PUBACK     |         4           |  Publish Acknowledgment|
PUBREC      |        5           |   Publish Received (assured delivery part 1)|
PUBREL      |        6            |  Publish Release (assured delivery part 2)|
PUBCOMP |        7            |  Publish Complete (assured delivery part 3)|
SUBSCRIBE|        8            | Client Subscribe request |
SUBACK     |        9            | Subscribe Acknowledment |
UNSUBSCRIBE|10            | Client Unsubscribe request|
UNSUBACK |     11           | Unsub Acknowledgment|
PINGREQ    |     12            | PING Request|
PINGRESP  |      13            | PING Response|
DISCONNECT|  14            | Client is Disconnecting |
Reserved    |      15           |  Reserved|

###2.标识
第一个字节剩下的位数包括DUP，QoS 和 保留字。这几个位数的位置设置：
Bit position | Name |Description|
----------------| -------- :| :--------------:|
3                  | DUP    | Duplicate delivery|
2-1               | QoS    | Quality of Service |
0                  | RETAIN | RETAIN flag |

####DUP
位置：字节1的第三位
当客户端或服务器尝试重新分发一个PUBLISH,PUBREL,SUBSCRIBE或者  
UNSUBSCRIBE消息的时候设置这个标识。这可以让消息的QoS>0，并且需要认同。  
如果设置了DUP位，变量头就包括一个消息的ID。

消息的接收者可以把这个标识作为前一个消息是否已经接收到的标记。但是不应该依赖于检测重复。
