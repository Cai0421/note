## COM组件简介

COM标准即根据组件化的程序设计的思想的基础上建立，COM提出了组件间进行交互的规范，同时提供了实现交互的环境。同样由于COM对象是建立于二进制可执行代码级的基础上，所以COM对象是可以实现不同语言开发的组件进行交互。

COM是面向于对象的软件模型，对象是其的基本要素，其提供了接口成员函数为客户提供相关的服务。同时在COM组件中客户请求服务仅能通过接口进行，每个接口由128位的GUID进行表示，客户通过该全局唯一标识符获取接口指针，通过接口指针调用内部的成员函数。

同样客户也通过128位的GUID对COM对象进行标识，即**CLSID**，系统中若存在这类COM对象的信息，并同时包括COM对象所在的模块文件（DLL或者EXE）以及COM对象在代码中的入口点，用户即可以通过此CLSID来创建COM对象。

客户创建对象后，得到的是指向对象某个接口的指针，通过该指针调用接口的所有服务。同样的客户可以创建两个相同CLSID的对象，但是该两个对象可以具有不同的状态。（近似于对象-类之间的关系）

```sequence
COM->用户: CLISID对象标识
用户-->COM对象: 创建COM对象
COM对象-->用户: 接口指针
用户-> COM对象: 调用接口服务
```



COM本身通过COM库的相关调用，实现了对象和客户之间的二进制交互

1. 提供少量的API函数实现客户和服务端COM应用的创建过程，客户端：创建函数，服务器端：对象的访问支持
2. COM通过注册表查找本地服务器即EXE程序，以及程序名与CLSID的转换
3. 提供了一些标准的内存控制方法，使应用控制进程中内存的分配

进行COM组件开发是基于COM提供的API进行相关的编程，COM对象具有封装性、重用性以及多态性。



客户程序用一个指向接口的指针调用接口成员函数，接口指针实际指向第二个指针，其中第二个指针指向一组函数——接口函数表（虚函数表），接口函数表中每一项为4字节长的函数指针，每个函数指针同对象的具体实现连接起来。从而实现了客户获得接口指针，就可调用对象的实际功能。

```sequence
客户 -> 接口: 接口指针

接口 --> 接口函数表: 指针
Note right of 接口函数表: 存储函数指针
接口函数表 --> 客户: 函数指针
```

接口描述语言IDL

可用于定义COM接口，定义常用的数据类型，描述自定义的数据接口，对于接口成员函数，可以指定每个参数的类行，输入输出特性，支持可变长度数组的具体描述，IDL支持指针类型。

```c++
class IUnknown
{
Public:
virtual HRESULT _stdcall QueryInterface([in] REFIID iid, [out] void **ppv)=0;
virtual ULONG _stdcall AddRef(void)=0;
virtual ULONG _stdcall Release(void)=0;
}
```

```idl
interface IUnknown
{
HRESULT QueryInterface([in] REFIID iid, [out] void **ppv);
ULONG AddRef(void);
ULONG Release(void);
}
```

**进程内组件**

进程内组件和客户程序运行于同一进程地址空间中，对于客户程序与组件程序建立起通信关系后，客户程序能通过vtable调用成员函数。

1. 客户程序使用相关函数装入DLL，函数返回模块的实例句柄
2. 客户程序通过实例句柄以及相关函数，获取函数的地址
3. 最后卸载DLL出内存，释放资源

注意点

1. DLL程序可以引出全局变量，通过变量名进行引出
2. 客户程序同样可以为DLL程序，但需要先被装进进程空间

**进程外组件**

调用进程外组件需要解决的问题

1. 一个进程如何调用另外一个进程的函数
2. 参数如何从一个进程中传递至另外一个进程



COM库通过类厂创建COM对象，类厂本身也是一个COM对象，支持接口IClassFactory，其中的成员函数CreateInstance可用来创建对应的COM对象，类厂对象本身是通过DLLGetClassObject函数进行创建。

COM库接受对象创建的指令 -> 调用进程内组件的DLLGetClassObject函数，创建类厂对象，返回类厂对象的指针，通过类厂对象的指针接口中的成员函数创建相应的COM对象

```sequence
客户程序 -> COM库: 创建对象
COM库 --> 类厂对象: 调用进程内组件函数，创建
类厂对象 -> COM库: 指针
COM库 -> 客户程序: 创建COM对象
```

## ASP的COM组件开发

使用COM组件动态开发Web内容——即ASP pages。COM组件能关联数据库，处理消息，以及ASP页面可以发送给客户的独立输出。

### IIS COM组件概述

通过COM组件的组合来进行Web应用程序的开发，其中ASP即IIS的应用程序

#### 关于组件

详见上，对于ASP的组件自定义开发，理论上能通过支持COM组件模型开发的语言（java,C,C++）等实现

#### 组件的使用

可开发用于各种Internet和Intranet方案的组件，以实现业务逻辑。此处主要介绍可用于在支持Active Server Pages（ASP）的Web服务器上运行的组件使用的事件方法和接口。在构建应用程序之前，应将应用程序的一般功能组织为不同的功能单元。但应将服务器组件实现为DLL（进程内）服务器，DLL同Web应用程序位于同一进程内，能快速创建组件对象。但对于在进程外创建的组件如上面所写易遇到相关的错误。

对于组件对象模型的学习内容，详见：

https://docs.microsoft.com/zh-cn/windows/win32/cossdk/com--programming-reference?redirectedfrom=MSDN

https://docs.microsoft.com/zh-cn/windows/win32/cossdk/component-services-portal?redirectedfrom=MSDN

#### 事务处理

事务处理可用于COM组件和ASP页。在COM组件中的实现涉及调用正确的COM接口，在ASP页面中的实现使用。

#### 排队组件

排队组件使您可以创建可以在客户端和服务器连接后立即执行的组件。它们提供了一种简便的方法来异步调用和执行组件。如果客户端和服务器未连接，则组件可以保持执行直到建立连接。

### 组件类型

#### 表示组件(Presentation Components)

表示组件从用户那里获取信息或返回已格式化的数据。 ASP Request和Response对象为内置对象，可用于建立表示组件。表示组件构成了多层应用程序中的表示层，使业务组件与特定用户界面充分分离，并能保证业务组件在添加新客户端时保持不变。

表示组件的主要作用，是对于将要传入界面的数据进行格式化，方面不同的界面能统一显示数据。

#### 业务组件(Business Components)

业务组件主要是对于目标事件的内在处理，主要是通过组件实现业务规则。

需要考虑的实际内容为

输入：实际输入的相关数据

处理：对于输出数据的实际处理

更新：相关的更新规则，何时触发组件

安全：用户的权限

#### 数据组件(Data Components)

将数据存储在数据组件中为其他组件访问数据提供了一种标准方法。

### IIS中的Windows 脚本组件

## IIS结构

主要介绍了IIS不同版本的较狗，以及IIS是如何处理来自客户的请求，IIS应用是如何独立的，IIS 进程和线程的安全标识

### IIS 在Client/Server关系中的定位

#### 静态文件以及动态文件的传输

**静态文件**：文件在客户端以及服务器端是相同的，该文件内容即为静态的，静态内容不允许发生任何改变。

下图为静态文件的传输方式

![Static Request and Response in IIS](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/images/ms525077.iisstaticrequestresponse(en-us,vs.90).gif)

下图为动态文件的传输方式

![Dynamic Request and Response in IIS](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/images/ms525077.iisdynrequestresponse(en-us,vs.90).gif)

#### IIS的角色

IIS 履行 Web 服务器的角色，响应来自 Web 客户端（如 IE）的文件请求和日志记录活动。 IIS 维护有关内容文件的位置、哪些安全标识有权访问这些文件、内容文件如何分离到应用程序中以及哪些URL映射到这些应用程序的信息。

### IIS 架构解析

1. Windows Process Activation Service 进程激活服务，主要用来支持HTTP/HTTPS之外的控制协议
2. 自定义模块的引擎
3. 集成了IIS和ASP.Net的请求处理管道

#### IIS的组成组件

##### 协议监听器

协议侦听器是一个可以侦听预定义通信通道（端口），传递数据（请求的数据）和参与服务和客户端通信的程序。

当客户端浏览器通过Internet请求一个page页面，这个时候首先被HTTP协议监听器HTTP.sys监听到，经过检查之后将请求发送给IIS来处理请求。一旦IIS处理完请求之后，HTTP.sys就会将请求的响应结果输送到客户端。

默认的IIS提供了HTTP.sys作为协议侦听器来侦听HTTP/HTTPS的请求，因为我们web大部分通过HTTP/HTTPS协议作为请求协议。

###### HTTP.sys

1. 内核模式缓存。如果某个资源被频繁请求，HTTP.sys会把响应的内容进行缓存，缓存内同可以直接响应后续请求，由于这是基于内核模式缓存，不存在内核模式和用户模式的切换，响应速度得到了极大的改进。
2. 内核模式请求队列。请求的上下文切换，因为内核将直接向正确的辅助进程的请求转发导致开销也更少。如果没有工作进程可用来接受请求，内核模式请求队列保存请求，直到工作进程拾了起来。
3. 请求前处理和安全筛选。

##### Word Wide Web Publishing Service(W3SVC)

WWW service内容别拆分为W3SVC以及WAS服务两部分内容

W3SVC是对于HTTP进行监听，主要承载以下三部分功能

- HTTP请求的接受和配置管理
- 进程管理：创建、回收、监听工作进程
- 性能检测

WAS为IIS提供了非HTTP协议的支持，如TCP监听器等等，同时进行进程的管理和应用进程池的管理。

WAS读取ApplicationHost.config信息 -> 传递给监听适配器 -> 建立WAS和协议监听器之间通信

**配置信息**：全局配置信息，HTTP和非HTTP协议的配置信息，应用进程池的配置，网站的配置信息，应用程序的配置信息

##### IIS模块化

IIS 提供一个 Web 服务器引擎，用来根据你的需求来增加/删除组件和模块的调用。模块是服务器用来处理请求的单独的功能组件。

###### 本地化模块

- HTTP 模块：IIS7之后部门模块主要用来处理HTTP请求。HTTP模块能够将响应信息输出到客户端，返回HTTP错误，重定向请求等。
- 安全模块：在 IIS 中的几个模块执行请求处理管线中的安全相关的任务。除此之外，每个验证机制都有独立的模块，可供你自己选择自己需要的验证模块。还有一些模块用来过滤请求和URL授权。
- 内容模块：在 IIS 中的几个模块执行请求处理管道中的内容相关的任务。内容模块包括模块来处理请求的静态文件，当客户端没有指定具体页面时，以返回一个默认的页面，列出一个目录，和更多的内容中指定的资源。
- 压缩模块：在 IIS 中的两个模块在请求处理管道中执行压缩。
- 缓存模块：网站或应用程序中通过存储网页之类的处理信息来提高性能和可重用性。
- 日志记录和诊断模块：日志记录模块支持加载自定义模块，并将信息传递给HTTP.SYS.诊断模块用来在处理请求过程中关注和报告事件。

| 模块名称                    | 描述                                                         | 资源                |
| --------------------------- | ------------------------------------------------------------ | ------------------- |
| CustomLoggingModule         | 装载用户定义日志模块                                         | Inetsrv\Logcust.dll |
| FailedRequestsTracingModule | 支持请求失败跟踪                                             | Inetsrv\Iisfreb.dll |
| HttpLoggingModule           | 传递信息和请求处理状态到HTTP.SYS作为日志                     | Inetsrv\Loghttp.dll |
| RequestMonitorModule        | 跟踪请求在工作进程中的执行信息，报告运行时的状态和控制应用程序接口 | Inetsrv\Iisreqs.dll |
| TracingModule               | 事件报告给 Microsoft 跟踪 Windows 事件 (ETW)。               | Inetsrv\Iisetw.dll  |

##### IIS中的应用进程池

IIS的工作进程隔离模式：即不同进程之间是独立运行的，每个进程中是存在应用进程池，这就决定了即时其中某个进程出现问题不会影响其余的进程中的应用进程池。

集成应用进程池模式：如果托管应用程序在采用集成模式的应用程序池中运行，服务器将使用 IIS 和 ASP.NET 的集成请求处理管道来处理请求。

经典模式：用经典模式的应用程序池中运行，服务器会继续通过 Aspnet_isapi.dll 路由托管代码请求。

![image-20210119214112041](C:\Users\CRUN\AppData\Roaming\Typora\typora-user-images\image-20210119214112041.png)

##### IIS请求的流程部分

1. 当客户端发起你个面向服务器的http请求后，HTTP.sys截获该请求。
2. HTTP.sys通知WAS从配置文件中获取必要的信息。
3. WAS从applicationHost.config文件中请求配置信息。
4. W3SVC接收到相应的配置信息：应用程序池，网站配置等信息。
5. W3SVC使用配置信息来配置HTTP.sys.
6. WAS为请求隔离模式相匹配的应用程序池开启一个工作进程。
7. 工作进程处理请求并且返回响应给HTTP.sys.
8. 客户端接收响应。

![img](https://images2015.cnblogs.com/blog/352688/201611/352688-20161106205641455-1219191566.png)

![img](https://images2015.cnblogs.com/blog/352688/201611/352688-20161106210425752-349181825.png)

## IIS Web开发技术

### 设计IISWeb应用

#### 决定使用的IIS开发技术

详见下列三个主题部分内容（需要了解

- [Comparison of IIS Administration Features](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/ms525885(v=vs.90))
- [Comparison of IIS Development Technologies](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/ms525913(v=vs.90))
- [Other Products and Technologies that Interoperate with IIS](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/ms524970(v=vs.90))

##### 设计新的IIS应用

https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/ms525076(v=vs.90)

详见上述，具体不想概括了。主要是对于设计IIS应用中的问题以及需要考虑的点的提出

#### 设计多层IIS应用

![Multi-Tier Web Application Design](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/images/ms524900.multitierwebappdesign(en-us,vs.90).gif)

1. 用户界面和导航

   该层不仅提供图形用户界面（GUI），以便用户可以与应用程序进行交互，输入数据并查看请求的结果，而且一旦客户端从服务器接收回数据，它还管理数据的操作和格式化。 

2. 业务逻辑

   业务逻辑涉及管理应用程序处理的规则，将第1层中的用户与第3层中的数据连接起来。这些规则所管理的功能紧密模仿日常业务任务，可以是单个任务或一系列任务。

3. 数据服务

   管理数据的存储，提供数据的接口

#### IIS的应用边界

在开发Web应用程序时，面临的最重要的任务之一是确定如何将Web内容文件组合到单个Web应用程序中。 

一般会有如下的四种处理方式

1. 将所有ASP文件和组件与IIS放在同一进程中；
2. 将所有ASP文件和组件放在IIS进程以外的其他进程中
3. 将所有ASP文件和组件放入IIS进程以外的合并进程中，并使用组件服务管理器隔离COM +组件。
4. 将所有ASP文件放入IIS进程中，并将组件放入另一个进程中。

跨进程调用的机制称为封送处理。封送处理的呼叫比单个进程中的呼叫要慢。因此，池化和隔离的应用程序的性能不及共享IIS进程的应用程序。

#### 控制IIS应用的流

ASP提供了六种不同的方式来影响一般的执行流程。下图中描述了这六个方法。图中的箭头表示执行流程。

**重定向**

![Redirection in Classic ASP](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/images/ms525295.iisredirectclassicasp(en-us,vs.90).gif)

需要通过服务器对于请求进行重定向

**转移**

![Transferring Control in Classic ASP](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/images/ms525295.iistransfercntrlclassicasp(en-us,vs.90).gif)

同样是进行重定向的过程，但是不需要通过服务器，能自己通过ASP内部进行转移调用，对于外部的ASP能通过引用外部ASP的方式进行重定向。

**执行**

![Executing Script in Classic ASP](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/images/ms525295.iisscriptexecuteclassicasp(en-us,vs.90).gif)

即ASP执行后返回结果的过程，适用于未将ASP内置至组件中的结构

**组件调用**

![Invoking COM Components in Classic ASP](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/images/ms525295.iisinvokecomcompclassicasp(en-us,vs.90).gif)

组件调用控制应用程序流的最常用方法。，即通过ASP对于组件相关函数的调用过程

**退出**

![Exiting a Script in Classic ASP](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/images/ms525295.iisexitscriptclassicasp(en-us,vs.90).gif)

退出A*SP脚本执行

**程序处理**

![Executing Procedures in Classic ASP](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/images/ms525295.iisexecuteprocclassicasp(en-us,vs.90).gif)

循环中断的执行流程

#### 最小化IIS应用程序中客户同服务器之间的进程

1. 最小化HTTP连接之间的通信。
2. 向客户端公开对完成处理任务绝对必要的那些服务器资源。来自客户端的每个请求都应完全合格，以便服务器不必响应客户端以获取更多信息，从而增加HTTP连接上的往返次数。



### ETW事件

![+](https://images2015.cnblogs.com/blog/624797/201510/624797-20151018151912616-644281100.jpg)（1）Event Session（事件会话）。事件提供者需要向控制器提交事件数据的元数据，控制器会将事件数据的元数据提供给事件消费者。它还可以控制是否接受事件提供者的事件提交。它对应Microsoft.Diagnostics.Tracing.TraceEventSession类。

（2）Event Provider（事件提供者），作用是引发事件，提供事件数据。它对应Microsoft.Diagnostics.Tracing.EventSource 类。

（3）Event Consumer（事件消费者），作用是消费事件。它对应Microsoft.Diagnostics.Tracing.TraceEventSource类。







日志部分可见下列

https://www.cnblogs.com/dehai/p/4889900.html

