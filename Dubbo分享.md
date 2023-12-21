# `Dubbo`分享
>之前已经讲过传统巨型单体应用,到服务化的架构演进过程,现今服务化架构现在已非常成熟,服务化过程中更多的是面临合理业务拆分,确定业务边界,以达成服务编排复用的目的.

>众多的微服务架构就包括我们目前使用的`SpringCloud`,和今天主要讲的分布式RPC框架`Dubbo`

## `Dubbo`的发展历史
`Apache Dubbo`是微服务领域的先行者,是由`Alibaba`于2011年开源的,是一款高性能分布式`RPC`框架,开源伊始就被大量的公司广泛采用,如在2014年,当当网在`Dubbo`分支上扩展出的`Dubbox`,支持了`HTTP REST`通信,可以说`Dubbo`在国内服务化体系演进中,扮演了一个非常重要的角色

``Dubbo``暂停维护几年后,于2017年重启维护,并于2018年`Alibaba`将``Dubbo``捐献给`Apache`,`Dubbo`也由一个微服务领域的`RPC`框架,演进为一个完整的微服务生态

架构服务化演进最重要的就是微服务间的通信,回想`SpringCloud`使用`Feign`与`eureka/zk`作为注册中心,进行基于HTTP协议的通信,`Ribbon`为`Feign`做负载均衡,使用断路器`Hystrix`防止服务雪崩,通过多种组件,做到了对现有服务的简单,无侵入的服务化改造

`Dubbo`也是基于同样的思路,与`SpringCloud`一样,涵盖了服务交互全过程,支持同样简单,无侵入对服务调用过程中各个阶段做扩展和定制,如通信协议,序列化格式,路由策略,负载均衡,监控运维等

`Dubbo`通过微内核,`SPI`扩展的方式解决定制化问题,不管是`Dubbo`自身的特性,还是业务方的拓展,都统一通过`SPI`机制来实现,有定制化需求的公司可以基于`SPI`机制,定制或拓展`Dubbo`.保证了框架自身的可持续性发展和稳定性,这也是大多数框架,大多数软件设计中一个共有思路

## `Dubbo`简介
现在有个场景,由你来设计架构,根据业务需求,要求系统在分布式场景下实现高并发,高可拓展,自动容错加高可用,那我们首先要考虑的是分布式通信问题,分布式不能通信,便会降级为单体系统,`Dubbo`本身就是一款高性能分布式RPC框架,最初的目的就是为了解决分布式的通信问题

编写分布式下的高并发,高可拓展的系统对技能要求很高,其中涉及大量的序列化/反序列化,网络,多线程,设计模式,性能调优等众多专业知识,`Dubbo`将这些做了高层的抽象和封装,提供了各种开箱即用的特性,让用户可以简单的去使用

`Dubbo`也是目前是阿里`SOA`服务化的核心框架,每天为2000多个服务提供30亿次的访问量支持,并被广泛应用于阿里集团各成员站点,在分布式`RPC`框架中,`Dubbo`是Java类项目中非常好的一个框架,提供了注册中心机制,解耦了服务方和消费方动态发现的问题,提供高可用能力

### `Dubbo`RPC通信的过程

![](https://user-gold-cdn.xitu.io/2020/6/20/172cf3b56da9c183?w=1060&h=824&f=png&s=48717)
- 首先这里涉及到几个角色,分别是 
1. `Provider` RPC服务提供者
2. `Consumer` RPC服务消费者
3. `Register` RPC服务注册中心
4. `Moniter` 监控中心
- 通信流程
1. Provider启动时,会向注册中心把自己的元数据注册(register),如IP和端口
2. Consumer启动时,从注册中心订阅(subscribe)服务方提供的元数据(第一次订阅会拉取全量数据)
3. 注册中心Zookeeper某个节点发生事务操作(操作zk的客户端任何新增,删除,修改,会话创建和失效的操作),该节点的版本号就会发生变化,触发watcher事件(事件通知),客户端接到通知后,会把对应节点下的全量数据都拉取
4. 在获得服务元数据后,Consumer可以对Provider发起RPC调用,在RPC调用的前后会向监控中心上报统计信息(如并发数和调用的接口)

> 补充知识一:同步/异步,阻塞/非阻塞
>- 这些概念都是针对调用者来说的

> 同步(sync),异步(async)的关注点在于消息通信机制
> - 同步调用:调用者发起一个同步调用时,在没有得到结果之前,调用不返回,一旦调用返回,就得到返回值,就是说,调用者主动等待这个调用的结果,却并不关心调用者线程是否阻塞
如顺序代码,如果说`func()`结果没有返回,`next()`就不会执行
```js
    int n = func();
    next();
```

> 异步调用:在发出一个异步调用时,调用者不会立即得到结果,该调用就返回了,所以此时没有返回结果,而是在调用发出后,被调用者通过状态,通知来通知调用者,或者通过回调函数来处理这个调用

> 这段中的func()执行后,还没得出结果就立即返回,然后执行next();
等到结果出来,`func()`调用`callback()`通知调用者结果

```js
    func(callback);
    next();
    ...

    void callback(int n)     // func 结果回调
    {
    int k = n;
    }
```


> 阻塞和非阻塞关注的是程序在等待调用结果(消息,返回值时的状态)

>- 阻塞:调用者发起一个阻塞调用,调用者会进入阻塞状态,只有得到结果才会返回
如socket server中的accept(),直到connection建立都是blocked的,程序就停在recv()这里阻塞,后面的代码都不会执行了,如果线程始终阻塞着，永远得不到资源，于是就发生了死锁
```java
//The method blocks until a connection is made.
 Socket socket = serverSocket.accept();
```
> 非阻塞调用则是不能立即得到结果之前,该函数不会阻塞当前线程,而会立即返回
 比如非阻塞socket的send()，调用这个函数，它只是把待发送的数据复制到TCP输出缓冲区中
 就立刻返回了，线程并不会阻塞，数据有没有发出去send()是不知道的,不会等待它发出去才返回

>同步的定义看起来跟阻塞很像，但是同步跟阻塞是两个概念，同步调用的时候，线程不一定阻塞，
调用虽然没返回，但它还是在运行状态中的，CPU很可能还在执行这段代码，而阻塞的话，
它就肯定不在CPU中跑这个代码了。这就是同步和阻塞的区别。同步是可以在CPU，阻塞是肯定不在CPU。

>异步和非阻塞的定义比较像，两者的区别是异步是说调用的时候结果不会马上返回，
线程可能被阻塞起来，也可能不阻塞，两者没关系。非阻塞是说调用的时候，线程肯定不会进入阻塞状态。


> 补充知识二
线程的状态轮转
进入阻塞状态的线程将失去CPU时间片
![](https://user-gold-cdn.xitu.io/2020/6/20/172cf3644fbd02f6?w=721&h=443&f=png&s=49988)



![](https://user-gold-cdn.xitu.io/2020/6/20/172cf3c911017d56?w=1060&h=824&f=png&s=48717)
看完补充知识我们再次回到`Dubbo`的流程图
1. 首先`Dubbo`的`provider`启动后,进行初始化的注册(1.register),这一步`Dubbo`使用默认的`curator`(一个框架风格的`zookeeper client`,它封装了对`zookeeper`的各种操作)作为操作zk的客户端,这一步操作比较简单,仅仅是通过`curator`在注册中心上创建了一个目录,对应的,取消发布则是把zk注册中心上对应的路径删除

![](https://user-gold-cdn.xitu.io/2020/6/20/172cf507fa1a56bf?w=806&h=120&f=png&s=12580)


2. subscribe和notify(订阅和通知的实现)

订阅通常有`pull`和`push`两种方式,一种是客户端定时轮询注册中心拉取配置,另一种是注册中心主动推送数据给客户端,`Dubbo`采用的是第一次启动拉取全量,后续进行事件通知+消费者拉取的方式,消费者在第一次连接上注册中心时,会获取对应目录下的全量数据,并在订阅的节点上注册一个watcher,消费者与注册中心之间保持TCP长连接,后续每个节点有任何数据变化的时候,注册中心会根据watcher的回调主动通知消费者(事件通知),这个操作就是异步的,客户端接到通知后,会把对应节点下的全量数据拉取过来
3. invoke
最后Consumer基于客户端的负载均衡策略对provider进行RPC同步调用




## `Dubbo`的特性以及SPI机制-微内核,富插件的设计原则
### `Dubbo`的关键特性

![](https://user-gold-cdn.xitu.io/2020/6/20/172cfafa1178cf93?w=1419&h=524&f=png&s=77989)
我们看到Dubbo是面向接口代理的高性能RPC调用,那么什么是面向接口代理?Dubbo实际调用流程是怎样的?这里我们再补充一个代理模式的知识


> 补充知识三:代理模式?什么是静态代理,动态代理?
>首先来讲讲设计模式之代理模式
![](https://user-gold-cdn.xitu.io/2020/6/16/172bbcc119675afc?w=380&h=97&f=png&s=30105)
简单来说就是为真正要访问的对象提供一种代理,控制对真正对象的访问
>- 用处一:远程代理,也就是为一个对象在不同的地址空间提供局部代表,这样可以隐藏一个对象存在于不同地址空间的事实
>- 用处二:安全代理,使用代理类控制对真实对象的访问权限
>- 用处三:对象增强,使用代理类,可以为原始对象的功能进行增强,而不改变原始对象,例如加一些方法前验证对象,方法后关闭连接的工作,`Spring`中`AOP`就是依靠`JDK动态代理`或`Cglib`字节码增强实现的,下面讲一下各个代理模式
>**静态代理**
>这里涉及到三个角色 
>被代理类,代理类,共同接口

>静态代理要求代理类与被代理类实现同一个接口,即拥有共同的行为
这里我们定义了一个`IBlogService`,定义了写博客,发博客的行为
在`BlogServiceImpl`中实现了`IBlogService`,并重写了接口方法
在没有代理的情况下,我们就可以直接使用这个类了,这时我们看`BlogStaticProxy`静态代理类,同样的实现了`IBlogService`接口,
且通过构造方法注入了一个引用类型为`IBlogService`的对象
在测试类中,使用代理类,可以看见我们实现了原始对象的代理,这里我们使用了用法三,即对原始对象的增强.

>**动态代理**

>我们可以很容易看到静态代理的缺点,在需要对原始对象方法增强时,我们需要手动在静态代理类中对每个方法增强,有一劳永逸的解决方式吗?**JDK动态代理**

>JDK动态代理是AOP的一种实现,包括`Dubbo`也是使用动态代理,实现对远程对象的代理,也是众多框架的基础,今天我们来仔细看一下

> 首先我们弄清楚一点,JDK动态代理只能为实现接口的类进行代理,而cglib字节码增加没有这个限制,cglib是继承代理类后进行字节码增强,限制是不能为final类代理,这是java语言方面的限制

> 首先我们创建处理类`MyInvocationHandler` 实现`InvocationHandler`接口,看接口的javadoc我们可以明白 
`InvocationHandler`是`package java.lang.reflect`反射包中的接口
每个动态代理类都要实现它,每个动态代理的实例都有一个关联的`invocation handler`
当一个方法被动态代理实例调用,将会调用其中的invoke方法,这个类是我们动态生成的动态代理类调用的处理类

> 同样的,用构造方法将被代理实例注入,之后重写`invoke()`方法同静态代理一样进行对象增强,这里我们看到invoke的参数,
三个参数
> 1. proxy 动态代理实例
> 2. method 与代理实例上调用的接口方法相对应的method实例
> 3. args 一个对象数组，其中包含在代理实例的方法调用中传递的参数的值
```java
 @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("DynamicStart...");
        //反射调用原方法,方法的调用对象为被代理对象,参数为args,返回值为方法返回的结果
        Object result = method.invoke(blogService, args);
        System.out.println("DynamicEnd...");
        return result;
    }
```
> 接着我们来使用刚刚创建的InvocationHandler实例

```java
public class TestJdkProxy {
    public static void main(String[] args) {

        //创建处理类实例
        MyInvocationHandler myInvocationHandler = new MyInvocationHandler(new BlogServiceImpl());


        //获取代理类实例dynamicProxy
        //获取实例的三个参数
        //loader：定义了代理类的ClassLoader;
        //interfaces：代理类实现的接口列表
        // h：调用处理器，也就是我们上面定义的实现了InvocationHandler接口的类实例
        IBlogService proxyInstance = (IBlogService)(Proxy.newProxyInstance(IBlogService.class.getClassLoader(), new Class[] {IBlogService.class}, myInvocationHandler));

        System.out.println(proxyInstance.getClass().getName());

        //通过代理类对象调用代理类方法，实际上会转到invoke方法调用
        proxyInstance.writeBlog();
        proxyInstance.releaseBlog();

        System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
    }


}
```

>这里看到得到动态代理实例我们传入了三个参数,类加载器,接口数组,InvocationHandler实例
我们看到在JDK动态代理中涉及如下角色：

>- 业务接口Interface、业务实现类target、业务处理类Handler、JVM在内存中生成的动态代理类`$Proxy0.class`
>1. `Proxy`通过传递给它的参数 `interfaces/invocationHandler`生成代理类`$Proxy0.class`；
>2. `Proxy`通过传递给它的参数`ClassLoader`来加载生成的代理类`$Proxy0.class`的字节码文件；
`$Proxy0.class`在构造函数中调用父类构造函数对`handler`进行初始化后,各方法回调handler的invoke方法完成方法调用


看了JDK动态代理使用和原理后,我们回到`Dubbo`,看到`提供高性能的基于代理的远程调用能力,服务以接口为粒度,为开发者屏蔽远程调用的细节`这时对这句话就有了一定的理解,`Dubbo` 默认使用 `Javassist` 框架为服务接口生成动态代理类,在动态代理类中的`InvocationHandler`中进行`RPC`调用,实现了远程方法的本地调用

Dubbo的调用过程

![](https://user-gold-cdn.xitu.io/2020/6/20/172d0c15b0040a89?w=1368&h=332&f=png&s=179233)
首先服务消费者通过代理对象 Proxy 发起远程调用，接着通过网络客户端 Client 将编码后的请求发送给服务提供方的网络层上，也就是 Server。Server 在收到请求后，首先要做的事情是对数据包进行解码。然后将解码后的请求发送至分发器 Dispatcher，再由分发器将请求派发到指定的线程池上，最后由线程池调用具体的服务。这就是一个远程调用请求的发送与接收过程。至于响应的发送与接收过程，这张图中没有表现出来。对于这两个过程，我们也会进行详细分析。

**服务调用方式**
Dubbo 支持同步和异步两种调用方式，其中异步调用还可细分为“有返回值”的异步调用和“无返回值”的异步调用。所谓“无返回值”异步调用是指服务消费方只管调用，但不关心调用结果，此时 Dubbo 会直接返回一个空的 RpcResult。若要使用异步特性，需要服务消费方手动进行配置。默认情况下，Dubbo 使用同步调用方式。


### 微内核+富插件的思想,平等的对待第三方

`Dubbo`良好的扩展性涉及到两个方面,一个是在框架中针对不同场景,使用了合适的设计模式,第二个就是现在要讲的,`Dubbo SPI`加载机制,让整个框架的接口和具体实现完全解耦,从而奠定了整个框架良好可拓展性的基础

>策略模式,面向接口编程是现在各框架的基础思路,比如之前讲的SpringCloud Feign,和SpringCloud结合后,将注解和代码翻译为HTTP请求,具体是由哪个HTTP框架发送的可以自定义,Feign采用面向接口编程,可以方便的使用HttpClient,OkHttp等http客户端进行传输,再如Mysql的存储引擎,都是实现了mysql Server提供的接口,不限于软件,甚至于网络协议,OSI七层同样是各上层协议提供接口,下层协议提供实现

回到`Dubbo`,它默认提供了很多可以直接使用的拓展点,`Dubbo`中几乎所有的组件都是基于扩展机制`SPI`,`Dubbo`没有使用`JavaSPI`而是在`JavaSPI`的基础上做了自己的实现,并且兼容`Java SPI`,服务在启动的时候,首先了解下Java SPI

#### Java SPI
首先了解下Java SPI的使用,JavaSPI全程是`Service Provider Interface`,起初是提供给厂商做插件开发的,在`Mysql Driver`等一些jar中可以看到它的应用,JavaSPI采用策略模式,一个接口多种实现,我们只声明接口,具体的实现并不在程序中直接确定,而是由程序之外的配置掌控,使用的具体步骤如下
1. 定义一个接口和几个实现类
2. 在META-INF/services/目录下,创建一个以接口全路径命名的文件,其内容为具体实现类的全路径名,多个则使用分行分隔
3. 在代码中使用`java.util.ServiceLoader`加载具体的实现类
经过这些配置,PrintService的具体实现就可以由文件配置的实现类来确定了

> 补充知识四 策略模式
![](https://user-gold-cdn.xitu.io/2020/6/20/172d01090908e810?w=1044&h=197&f=png&s=160769)
策略模式是一种定义一系列算法的方法,从概念上看,所有这些算法完成的都是相同的工作,只是实现不同,他可以以相同的方式调用所有的算法,减少了各种算法类与使用算法类之间的耦合
并且策略模式简化了单元测试,因为每个算法都有自己的类,可以通过自己的接口单独测试,每个算法可保证它没有错误,修改其中一个也不会影响到其他的算法


### `Dubbo`对于扩展点加载机制的改进

与JavaSPI相比,`Dubbo`SPI做了一定的改进和优化,如Java标准的SPI会一次性实例化所有拓展点,如果有拓展实现则初始化比较耗时,如果没用上也在家,则浪费资源,`Dubbo`增加了对扩展IOC和AOP的支持,一个扩展可以直接setter注入其他的扩展,我们看一下Dubbo中使用Dubbo SPI的例子
1. 同样的定义一个接口和几个实现类
2. 在META-INF/dubbo/internal目录下,创建一个以接口全路径命名的文件,其内容区别与Java SPI,采用key-value的形式,key为扩展类别名,可以使用`@SPI注解`,在接口上标注默认的拓展类,多个则使用分行分隔
3. 在代码中使用`ExtensionLoader`加载具体的实现类

有了这一点认识,结合`Dubbo`源码,我们可以初步的理解`Dubbo`使用`SPI`机制实现的微内核,富插件的思想,平等对待第三方,我们看到动态代理默认使用的实现类为`javassist`,传输默认使用了`netty`,我们同样可以实现接口,由SPI机制,使用我们自己的实现,以满足特殊的业务需求
![](https://user-gold-cdn.xitu.io/2020/6/20/172d0b75c3d2ce8f?w=481&h=170&f=png&s=8896)


![](https://user-gold-cdn.xitu.io/2020/6/20/172d0b7b608734fd?w=588&h=226&f=png&s=11620)


## Dubbo-Demo与Dubbo与SpringCloud结合实现RPC调用
Dubbo可以使用多种配置方式,这里依据官网实现了`XML配置方式`,`注解配置`,`Dubbo API`配置方式,详见代码

另外Dubbo可以与SpringBoot完美整合,实现基于ZK注册中心的RPC调用,是我们实现微服务调用的选型之一,我们上次已经讲微服务时已经演示过.
