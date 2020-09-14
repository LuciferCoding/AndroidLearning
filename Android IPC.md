# 1. Android IPC  
---
## [Android Binder设计与实现](https://blog.csdn.net/universus/article/details/6211589)

>传输性能： 

IPC | 数据拷贝次数
---|---
共享内存 | 0
Binder | 1
Socket/管道/消息队列 | 2  

>安全性：
传统IPC没有任何安全措施，完全依赖上层协议来确保。1.传统IPC方式接收方无法获得对方进程可靠的UID/PID(用户ID/进程IO),从而无法鉴别对方身份。Android为每个安装好的应用分配UID，进程的UID是鉴别进程身份的重要标志。  

==Binder优势总结：Binder基于C/S(Client-Server)通信模式，传输过程只需一次拷贝，为发送方添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。==

Binder是面向对象的：Client通过Binder的引用来访问Server(句柄)。Binder模糊了进程边界，淡化了进程间通信过程，使整个系统仿佛运行于同一个面向对象的程序中。  

* Binder的通信模型  
Binder框架四个角色：==Client，Server，ServiceManager和Binder驱动==。Client，Server，ServiceManager运行于用户空间，Binder驱动运行于内核空间。类似于互联网模型：Server是服务器，Client是客户端，ServerManager是DNS，Binder驱动是路由器。  
    + Binder驱动负责各进程Binder通信的建立，传递，引用计数管理等底层支持。
    
    + ServiceManager负责将字符形式的Binder名字转化成Client中对该Binder的引用，使Client能够通过Binder名字获取对Server中Binder实体的引用。ServiceManager的实现：ServiceManager与其他进程通信同样需要跨进程通信，对于其他进程，ServiceManager为Server端，有自己的Binder，其他进程都是Client，需要通过这个Binder的引用来实现Binder的注册，查询和获取。==ServiceManager提供的Binder没有名字也不需要注册，当一个进程将自己注册成ServiceManager时Binder驱动会自动为它创建Binder实体，而这个Binder的引用在所有Client中都固定为0==。之后每个Server向ServiceManager注册自己的Binder就必须通过0这个引用号和ServiceManager的Binder通信。0号引用就好比域名服务器的地址，必须预先配置好。
    
    + Server创建Binder实体，生成一个字符形式的名字，将这个Binder和名字一起以数据包的形式通过Binder驱动发送给ServiceManager，ServiceManager会将这个Binder进行注册。==Binder驱动则为这个Binder创建位于内核中的实体节点以及ServiceManager对实体的引用，将名字和新建的引用打包传递给ServiceManager==。ServiceManager收到数据包之后，将名字和引用填入一张查找表中。
    
    + Client通过名字向ServiceManager获得Server的Binder的引用。Client通过保留的0号引用向ServiceManager请求访问名为xxx的某个Binder，ServiceManager就会从表里去查询对应名字的Binder的引用，然后将该引用回传给Client。多个Client向Server中请求则会产生多个引用，且这些引用都是==强类型==，以确保只要存在应用，Binder就不会被释放掉。

---

