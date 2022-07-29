# Tomcat底层架构及原理



# 底层原理



# Tomcat顶层架构

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea72489533aa4ee79b5dc9ffdc0ed614~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

Tomcat中最顶层的容器是Server，代表着整个服务器，从上图中可以看出，一个Server可以包含至少一个Service，用于具体提供服务。

Service主要包含两个部分：Connector和Container。从上图中可以看出 Tomcat 的心脏就是这两个组件，他们的作用如下：

1、Connector用于处理连接相关的事情，并提供Socket与Request和Response相关的转化; 2、Container用于封装和管理Servlet，以及具体处理Request请求；

一个Tomcat中只有一个Server，一个Server可以包含多个Service，一个Service只有一个Container，但是可以有多个Connectors，这是因为一个服务可以有多个连接，如同时提供Http和Https链接，也可以提供向相同协议不同端口的连接,示意图如下（Engine、Host、Context下边会说到）：

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/793a566aff194d30b426a52c59a0b8a9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

多个 Connector 和一个 Container 就形成了一个 Service，有了 Service 就可以对外提供服务了，整个 Tomcat 的生命周期由 Server 控制。

另外，上述的包含关系或者说是父子关系，都可以在tomcat的conf目录下的`server.xml`配置文件中看出，下图是删除了注释内容之后的一个完整的`server.xml`配置文件（Tomcat版本为8.0）

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c1edde724734da9b9dffe04cceab010~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

配置文件图解：Server标签设置的端口号为8005，shutdown=”SHUTDOWN” ，表示在8005端口监听“SHUTDOWN”命令，如果接收到了就会关闭Tomcat。一个Server有一个Service，当然还可以进行配置，一个Service有多个，Service左边的内容都属于Container的，Service下边是Connector。 ![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d07dc0814d6478da57ceff2b918bb70~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

Tomcat顶层架构小结

（1）Tomcat中只有一个Server，一个Server可以有多个Service，一个Service可以有多个Connector和一个Container； （2） Server掌管着整个Tomcat的生死大权； （4）Service 是对外提供服务的； （5）Connector用于接受请求并将请求封装成Request和Response来具体处理； （6）Container用于封装和管理Servlet，以及具体处理request请求；

知道了整个Tomcat顶层的分层架构和各个组件之间的关系以及作用，对于绝大多数的开发人员来说Server和Service对我们来说确实很远，而我们开发中绝大部分进行配置的内容是属于Connector和Container的，所以接下来介绍一下Connector和Container。

## 三、Connector和Container的微妙关系

由上述内容我们大致可以知道一个请求发送到Tomcat之后，首先经过Service然后会交给我们的Connector，Connector用于接收请求并将接收的请求封装为Request和Response来具体处理，Request和Response封装完之后再交由Container进行处理，Container处理完请求之后再返回给Connector，最后在由Connector通过Socket将处理的结果返回给客户端，这样整个请求的就处理完了！

Connector最底层使用的是Socket来进行连接的，Request和Response是按照HTTP协议来封装的，所以Connector同时需要实现TCP/IP协议和HTTP协议！

Tomcat既然处理请求，那么肯定需要先接收到这个请求，接收请求这个东西我们首先就需要看一下Connector！

## 四、Connector架构分析

Connector用于接受请求并将请求封装成Request和Response，然后交给Container进行处理，Container处理完之后在交给Connector返回给客户端。

因此，我们可以把Connector分为四个方面进行理解：

（1）Connector如何接受请求的？ （2）如何将请求封装成Request和Response的？ （3）封装完之后的Request和Response如何交给Container进行处理的？ （4）Container处理完之后如何交给Connector并返回给客户端的？

首先看一下Connector的结构图（图B），如下所示：

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4082c51389d471a8c2fc1a150e7f566~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

Connector就是使用ProtocolHandler来处理请求的，不同的ProtocolHandler代表不同的连接类型，比如：Http11Protocol使用的是普通Socket来连接的，Http11NioProtocol使用的是NioSocket来连接的。

其中ProtocolHandler由包含了三个部件：Endpoint、Processor、Adapter。

（1）Endpoint用来处理底层Socket的网络连接，Processor用于将Endpoint接收到的Socket封装成Request，Adapter用于将Request交给Container进行具体的处理。

（2）Endpoint由于是处理底层的Socket网络连接，因此Endpoint是用来实现TCP/IP协议的，而Processor用来实现HTTP协议的，Adapter将请求适配到Servlet容器进行具体的处理。

（3）Endpoint的抽象实现AbstractEndpoint里面定义的Acceptor和AsyncTimeout两个内部类和一个Handler接口。Acceptor用于监听请求，AsyncTimeout用于检查异步Request的超时，Handler用于处理接收到的Socket，在内部调用Processor进行处理。

至此，我们应该很轻松的回答（1）（2）（3）的问题了，但是（4）还是不知道，那么我们就来看一下Container是如何进行处理的以及处理完之后是如何将处理完的结果返回给Connector的？

## 五、Container架构分析

Container用于封装和管理Servlet，以及具体处理Request请求，在Connector内部包含了4个子容器，结构图如下（图C）：

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/407fa0c57fa24b7497254bd276fdd565~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

4个子容器的作用分别是：

（1）Engine：引擎，用来管理多个站点，一个Service最多只能有一个Engine； （2）Host：代表一个站点，也可以叫虚拟主机，通过配置Host就可以添加站点； （3）Context：代表一个应用程序，对应着平时开发的一套程序，或者一个WEB-INF目录以及下面的web.xml文件； （4）Wrapper：每一Wrapper封装着一个Servlet；

下面找一个Tomcat的文件目录对照一下，如下图所示：

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4db32c4a65e9479da071b308a74e8fd9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

Context和Host的区别是Context表示一个应用，我们的Tomcat中默认的配置下webapps下的每一个文件夹目录都是一个Context，其中ROOT目录中存放着主应用，其他目录存放着子应用，而整个webapps就是一个Host站点。

我们访问应用Context的时候，如果是ROOT下的则直接使用域名就可以访问，例如：[www.ledouit.com,如果是Host（webapps）下的其他应用，则可以使用](https://link.juejin.cn/?target=http%3A%2F%2Fwww.ledouit.com%2C%25E5%25A6%2582%25E6%259E%259C%25E6%2598%25AFHost%25EF%25BC%2588webapps%25EF%25BC%2589%25E4%25B8%258B%25E7%259A%2584%25E5%2585%25B6%25E4%25BB%2596%25E5%25BA%2594%25E7%2594%25A8%25EF%25BC%258C%25E5%2588%2599%25E5%258F%25AF%25E4%25BB%25A5%25E4%25BD%25BF%25E7%2594%25A8)[www.ledouit.com/docs](https://link.juejin.cn/?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttp%253A%2F%2Fwww.ledouit.com%2Fdocs)进行访问，当然默认指定的根应用（ROOT）是可以进行设定的，只不过Host站点下默认的主营用是ROOT目录下的。

看到这里我们知道Container是什么，但是还是不知道Container是如何进行处理的以及处理完之后是如何将处理完的结果返回给Connector的？别急！下边就开始探讨一下Container是如何进行处理的！

## 六、Container如何处理请求的

Container处理请求是使用Pipeline-Valve管道来处理的！（Valve是阀门之意）

Pipeline-Valve是责任链模式，责任链模式是指在一个请求处理的过程中有很多处理者依次对请求进行处理，每个处理者负责做自己相应的处理，处理完之后将处理后的请求返回，再让下一个处理着继续处理。

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94782e305bb248ef963dfee928b8ecaa~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

但是！Pipeline-Valve使用的责任链模式和普通的责任链模式有些不同！区别主要有以下两点：

（1）每个Pipeline都有特定的Valve，而且是在管道的最后一个执行，这个Valve叫做BaseValve，BaseValve是不可删除的；

（2）在上层容器的管道的BaseValve中会调用下层容器的管道。

我们知道Container包含四个子容器，而这四个子容器对应的BaseValve分别在：StandardEngineValve、StandardHostValve、StandardContextValve、StandardWrapperValve。

Pipeline的处理流程图如下（图D）：

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3582e0aab7343ae9abeecebee77e3c2~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

（1）Connector在接收到请求后会首先调用最顶层容器的Pipeline来处理，这里的最顶层容器的Pipeline就是EnginePipeline（Engine的管道）；

（2）在Engine的管道中依次会执行EngineValve1、EngineValve2等等，最后会执行StandardEngineValve，在StandardEngineValve中会调用Host管道，然后再依次执行Host的HostValve1、HostValve2等，最后在执行StandardHostValve，然后再依次调用Context的管道和Wrapper的管道，最后执行到StandardWrapperValve。

（3）当执行到StandardWrapperValve的时候，会在StandardWrapperValve中创建FilterChain，并调用其doFilter方法来处理请求，这个FilterChain包含着我们配置的与请求相匹配的Filter和Servlet，其doFilter方法会依次调用所有的Filter的doFilter方法和Servlet的service方法，这样请求就得到了处理！

（4）当所有的Pipeline-Valve都执行完之后，并且处理完了具体的请求，这个时候就可以将返回的结果交给Connector了，Connector在通过Socket的方式将结果返回给客户端。