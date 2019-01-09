---
layout: post
title: tcp 有限状态机(译)
date: 2015-12-25
categories: network
tags: tcp
---





*   [状态、事件、流转](#set)


#### 几个术语

*   `State`(状态): The particular “circumstance” or “status” that describes the protocol software on a machine at a given time.
*   `Transition`(转换、流转): The act of moving from one state to another.
*   `Event`(事件): Something that causes a transition to occur between states.
*   `Action`(动作): Something a device does in response to an event before it transitions to another state.

#### 状态state、事件event、流转transition

Table 151 briefly describes each of the TCP states in a TCP connection, and also describes the main events that occur in each state,and what actions
and transitions occur as a result. For brevity, three abbreviations are used for three types of message that controltransitions between states,
which correspond to the TCP header flags that are set to indicate a message is serving that function.　　

下表简要的描述了一个tcp连接中的每一个状态，也描述了每个状态会发生的主要的事件，以及相应的动作和状态流转. 有三种类型的消息来控制状态之间的流转，在tcp的header中

*   `SYN`: A synchronize message, used to initiate and establish a connection. It is so named since one of its functions is to synchronizes sequence numbers between devices.
    同步包，用来初始化和建立连接的，之所以叫做同步包，是因为它的作用之一是来同步不同设备之间的序列号的
*   `FIN`: A finish message, which is a TCP segment with the FIN bit set, indicating that a device wants to terminate the connection.
    结束包，tcp报文段有一个FIN bit位，表明一个设备想要终止连接
*   `ACK`: An acknowledgment, indicating receipt of a message such as a SYN or a FIN.
    响应包，一个消息（如SYN或FIN）的回执相应

下表并不是所有可能的状态流转

|状态(state)|状态描述|事件(event) | transition (事件发生后的动作及其状态流转)|
|---|---|---|
|CLOSED|这是每个连接建立之前默认的状态，状态还没建立或者刚刚被关闭|**被动打开**|一个server开始连接进程通过被动的设置一个tcp port,同时它设置一个叫做transmission control block TCB的数据结构来管理连接，然后过度到LISTEN状态|
|　|　|**主动打开，发送SYN**|一个client开始一个连接通过发送一个SYN包,同设置一个TCB，然后过度到SYN-SENT状态|
|LISTEN|一个设备(通常是一个server)等待接收一个来自client的SYN,此时它还没有发送自己的SYN呢|**接收到client的SYN，发送SYN+SCK**|server设备接收到一个client的SYN包，然后它回传一个包含自己SYN和对client的ACK，进入SYN-RECEIVED|
|SYN-SENT|设备(通常是一个client)已经发送了自己的SYN，它在等待来自其他设备(通常是一个server)的匹配的SYN|**接收到SYN,发送ACK**|如果一个已经发送了SYN包的设备接收到了来自别的设备的SYN但是没有自己SYN的ACK(响应)，它会响应接受到的SYN，发送ACK,然后过渡到SYN-RECEIVED等到它的SYN的响应|
|　|　|**接收到SYN+ACK,发送ACK**|如果一个已经发送了SYN包的设备同时收到了它的SYN的ACK(响应)和其他设备的SYN,它对接受到的SYN进行响应ACK,过渡到ESTABLISHED状态|
|SYN-RECEIVED|设备已经发送了自己的SYN,并且接收到来自其他设备的SYN。它在等待它的SYN的ACK响应来完成连接|**接收到ACK**|当设备接收到它发送的SYN的ACK响应，它过渡到ESTABLISHED状态|
|ESTABLISHED|一个TCP连接的稳定状态，数据可以在设备间自由的交换直到连接被关闭|**关闭，发送FIN**|一个设备可以关闭通过发送一个FIN来关闭连接，然后过渡到FIN-WAIT-1状态|
|　|　|**接收到FIN**|一个设备可能会接收到来自连接方的关闭的请求FIN，然后它会对其进行响应，并过渡到CLOST-WAIT状态|
|CLOSE-WAIT|设备接收到一个来自另一个设备的关闭请求(FIN)。它必须等待本地设备来对这个请求进行响应，并产生一个对应的请求|**关闭，发送FIN**|被通知要关闭的使用TCP的应用，会向它运行的机器的TCP层发送一个关闭的请求，然后TCP会向远端的设备发送一个终止连接的请求，设备过渡到LAST_ACK|
|LAST-ACK|设备接收到一个关闭请求并对其进行了响应，也发送了自己的FIN，等待着自己FIN的ACK|**接收到FIN的ACK**|设备发送了自己的FIN包并接收到了响应，同时也接收到了其他设备的FIN并进行了响应，然后过渡到CLOSED状态|
|FIN-WAIT-1|这个状态的设备等待着它发送的FIN的ACK响应，或者别的设备的关闭请求|**接收到FIN的ACK响应**|设备接收到自己发送FIN的ACK,然后过渡到FIN-WAIT-2|
|　|　|**接收到FIN,发送ACK**|设备没有接收到自己发送的FIN的ACK,但是收到了别的设备的FIN。它对其进行响应之后，过渡到CLOSING状态|
|FIN-WAIT-2|处于这个状态的设备接收到自己FIN的ACK,并在等待对应的别的设备的FIN|**接收到FIN，发送ACK**|设备接收到来自别的设备的FIN包，它对其进行响应，并过渡到TIME_WAIT状态|
|CLOSING|设备已经接收到别的设备的FIN,并且也发送了其ACK响应，但是还没收到自己FIN的ACK|**接收到自己FIN的ACK**|设备接收到自己关闭请求的响应，过渡到TIME_WAIT状态|
|TIME-WAIT|连接双方都已经发送了FIN,并且收到了对方的ACK|**Timer Expiration**|在一段特定等待时间后，设备过渡到CLOSED状态|

#### 有限状态机

![有限状态机](http://lcj1992.github.io/images/network/tcpFsm.png)

[1]<http://www.tcpipguide.com/free/t_TCPOperationalOverviewandtheTCPFiniteStateMachineF.htm>

[2]<https://en.wikipedia.org/wiki/Transmission_Control_Protocol>

[3]<http://www.cnblogs.com/sunxucool/p/3449068.html>
