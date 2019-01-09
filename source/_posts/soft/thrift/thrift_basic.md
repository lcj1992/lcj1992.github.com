---
layout: post
title: thrift
date: 2016-07-26
categories: soft
tags: thrift rpc
---


  
  
  
  
  * [thrift VS protobuf](#thrift_vs_protobuf)

  
  
    * [编写thrift接口定义（idl）](#thrift_file)
    
    
    
    
  

### thrift基本概念 

![thrift_architecture](/images/soft/thrift_architecture.png)

Thrift通过一个中间语言(IDL, 接口定义语言)来定义RPC的接口和数据类型，然后通过一个编译器生成不同语言的代码（目前支持C++,Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk等）,并由生成的代码负责RPC协议层和传输层的实现。我们只需要去实现具体的接口实现就可以了。
* 服务层（service）：RPC接口定义与实现
* 协议层（protocol）：RPC报文格式和数据编码格式
* 传输层（transport）：实现底层的通信（如 socket）以及系统相关的功能（如事件循环、多线程）

#### 数据类型 

参见org.apache.thrift.protocol.TType

* Base Types：基本类型
* Struct：结构体类型
* Container：容器类型，即List、Set、Map
* Exception：异常类型
* Service： 定义对象的接口，和一系列方法
详见[数据类型](https://diwakergupta.github.io/thrift-missing-guide/#_types)

#### 协议层类型 

参见org.apache.thrift.protocol.TProtocol
Thrift可以让你选择客户端与服务端之间传输通信协议的类别，在传输协议上总体上划分为文本(text)和二进制(binary)传输协议, 为节约带宽，提供传输效率，一般情况下
使用二进制类型的传输协议为多数，但有时会还是会使用基于文本类型的协议，这需要根据项目/产品中的实际需求：

* TBinaryProtocol – 二进制编码格式进行数据传输。
* TCompactProtocol – 这种协议非常有效的，使用Variable-Length Quantity (VLQ) 编码对数据进行压缩。
* TJSONProtocol – 使用JSON的数据编码协议进行数据传输。
* TSimpleJSONProtocol – 这种节约只提供JSON只写的协议，适用于通过脚本语言解析
* TDebugProtocol – 在开发的过程中帮助开发人员调试用的，以文本的形式展现方便阅读。

#### 传输层类型 

参见org.apache.thrift.transport.TTransport

* TSocket – 使用堵塞式I/O进行传输，也是最常见的模式。
* THttpTransport – 采用Http传输协议进行数据传输
* TFileTransport – 顾名思义按照文件的方式进程传输，虽然这种方式不提供Java的实现，但是实现起来非常简单。
* TZlibTransport – 使用执行zlib压缩，不提供Java的实现。

    下面几个类主要是对上面几个类地装饰（采用了装饰模式），以提高传输效率。
* TBufferedTransport – 对某个Transport对象操作的数据进行buffer，即从buffer中读取数据进行传输，或者将数据直接写入buffer
* TFramedTransport – 以frame为单位进行传输，非阻塞式服务中使用。同TBufferedTransport类似，也会对相关数据进行buffer，同时，它支持定长数据发送和接收。
* TMemoryBuffer – 从一个缓冲区中读写数据，使用内存I/O，就好比Java中的ByteArrayOutputStream实现。

#### 服务端类型 

参见org.apache.thrift.server.TServer

* TSimpleServer– 简单的单线程服务模型，常用于测试
* TThreadedServer – 多线程服务模型，使用阻塞式IO，每个请求创建一个线程。
* TThreadPoolServer – 线程池服务模型，使用标准的阻塞式IO，预先创建一组线程处理请求。
* TNonblockingServer – 多线程服务模型，使用非阻塞式IO（需使用TFramedTransport数据传输方式）

#### thrift VS protobuf 

Thrift has integrated RPC implementation, while for Protobuf RPC solutions are separated, but available (like Zeroc ICE ).
参照[thrift_protobuf](http://stackoverflow.com/questions/69316/biggest-differences-of-thrift-vs-protocol-buffers)

### thrift使用 

#### 安装 

方式一

    brew install Boost
    brew install libevent
    brew install openssl
    ./configure
    sudo make
    sudo make install

方式二

    brew install thrift

#### 使用 

##### 编写a.thrift 

    namespace java tutorial
    namespace py tutorial
    typedef i32 int // We can use typedef to get pretty names for the types we are using
    service MultiplicationService{
        int multiply(1:int n1, 2:int n2),
    }

##### 生成代码 

    vi hello.thrifht
    thrift --gen java a.thrift

##### 接口实现类 

    import org.apache.thrift.TException;

    public class MultiplicationHandler implements MultiplicationService.Iface {

        @Override
        public int multiply(int n1, int n2) throws TException {
            System.out.println("Multiply(" + n1 + "," + n2 + ")");
            return n1 * n2;
        }
    }

##### server

    import org.apache.thrift.server.TServer;   
    import org.apache.thrift.server.TServer.Args;
    import org.apache.thrift.server.TSimpleServer;
    import org.apache.thrift.transport.TServerSocket;
    import org.apache.thrift.transport.TServerTransport;

    public class MultiplicationServer {

        public static MultiplicationHandler handler;

        public static MultiplicationService.Processor processor;

        public static void main(String [] args) {
            try {
              handler = new MultiplicationHandler();
              processor = new MultiplicationService.Processor(handler);

              Runnable simple = new Runnable() {
                public void run() {
                  simple(processor);
                }
              };     

              new Thread(simple).start();
            } catch (Exception x) {
              x.printStackTrace();
            }
          }

       public static void simple(MultiplicationService.Processor processor) {
           try {
              TServerTransport serverTransport = new TServerSocket(9090);
              TServer server = new TSimpleServer(new Args(serverTransport).processor(processor));

              System.out.println("Starting the simple server...");
              server.serve();
            } catch (Exception e) {
              e.printStackTrace();
            }
       }
    }

##### client

    import org.apache.thrift.TException;
    import org.apache.thrift.protocol.TBinaryProtocol;
    import org.apache.thrift.protocol.TProtocol;
    import org.apache.thrift.transport.TSocket;
    import org.apache.thrift.transport.TTransport;

    public class MultiplicationClient {
        public static void main(String [] args) {
            try {
              TTransport transport;

              transport = new TSocket("localhost", 9090);
              transport.open();

              TProtocol protocol = new  TBinaryProtocol(transport);
              MultiplicationService.Client client = new MultiplicationService.Client(protocol);

              perform(client);

              transport.close();
            } catch (TException x) {
              x.printStackTrace();
            }
        }

        private static void perform(MultiplicationService.Client client) throws TException{
            int product = client.multiply(3,5);
            System.out.println("3*5=" + product);
        }
    }

#### 抓包验证 

下边的报文就是上边的例子的，server提供multiply的服务实现，client进行调用。主要看CALL multiply和REPLY multiply

![thrift_wireshark](/images/soft/thrift_wireshark.png)

看代码可以看出：我们使用的是传输层我们使用的是TSocket，协议层使用了TBinaryProtocol，服务端我们使用的是TSimpleServer。

似乎还没有官方的协议说明，不过可以从代码里找(各个语言都有实现)

wireshark输出字段与thrift对应表可参照[字段说明](https://www.wireshark.org/docs/dfref/t/thrift.html)：

thrift中相关类：

org.apache.thrift.protocal.TMessage

org.apache.thrift.protocol.TMessageType:  CALL,REPLY,EXCEPTION,ONEWAY

org.apache.thrift.protocol.TField: 域类型，域的id（第几个），域名字

org.apache.thrift.protocol.TType: 参见数据类型

![thrift_request](/images/soft/thrift_request.png)

![thrift_response](/images/soft/thrift_response.png)

注：不同的协议，报文格式是不同的。

比如将协议由TBinaryProtocol变为TJSONProtocol，发送报文就变成这样了：

![thrift_json](/images/soft/thrift_json.png)
