# KCool-RSF 分布式服务中间件软件
##	术语定义
Router：提供路由功能的应用服务器
Client：客户端应用服务器
Service：经过路由对外提供服务的应用服务器
Server：不经过路由直接对外提供服务的应用服务器
Event：事件，各服务器之间用来交换数据的对象
Dataset：数据对象，用于在各应用服务器之间交互的统一对象
Handler：事件处理器，mina接收到消息时调用
IoSession：mina的会话session，用于发送事件、接收事件
KCool-RSF：分布式服务中间件软件的英文名称

##	需求概述
1.	提供Router组件。
2.	提供Client和Service基础支持。
3.	服务调用时支持远程调用。
4.	远程调用支持按功能进行路由。
5.	系统初始化时支持功能自动注册。

##	框架设计
分布式服务中间件软件KCool-RSF以Spring3.2.0.RELEASE为基础，用Mina2.0.9处理网络数据传输，传输的数据采用kryo2.24.0进行序列化。整个中间件按如下几个大的模块进行处理：
###	基础模块
基础模块负责Spring的加载，并在配置文件中用通配符的方式加载其他模块。在其他模块中，只需要把Spring的配置文件按照通配符的规则进行配置，引入后就可以自动加载。通配符的规则为：classpath*:spring/applicationContext-*.xml。系统初始化由基础模块进行处理。
要求所有模块的加载都使用Spring进行处理。
###	Router路由模块
路由模块是分布式服务中间件软件的核心模块，该模块负责Client和Service的注册、服务按需转发等功能，并提供完善的断线重连机制，并能对服务做一定的负载均衡处理。
###	Client客户端模块
Client客户端模块的主要功能是根据需要远程调用的服务接口自动生成代理类，在通过接口调用服务时，实际处理由代理类进行代理，通过这样的方式，屏蔽在调用服务时本地调用和远程调用的差异，极大地降低编码的工作量。
###	Service服务端模块
Service服务端对外提供服务，所有可提供出来的服务都在Router上进行注册，在Client调用服务时，由Router根据具体服务转发到对应的Service上进行处理。
###	Server服务模块
Server服务模块是为了适应不使用路由直接对外提供服务的情况，在这种场景中，Client客户端直接连接Server服务，Server服务处理完之后返回应答给Client。
###	模块依赖方式
在应用分布式服务中间件软件时，根据实际的需求，按如下方式进行依赖处理：
1.	不管是那种应用服务，都需要依赖基础模块，因为基础模块是所有其他模块的基础。
2.	不同应用服务器的依赖方式：
```
1. Router应用服务器：依赖Router路由模块。一般地，路由模块可以直接应用，只需要修改部分配置文件即可。
2. Service应用服务器：依赖Service服务端模块，此时，会自动依赖Client客户端模块。
3. Client应用服务器：依赖Client客户端模块。
4. Server应用服务器：依赖Server服务模块。
```

