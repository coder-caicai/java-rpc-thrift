# java-rpc-thrift
## 简单介绍
这是一个简单小巧的Java RPC框架，适用于Java平台内、为系统之间的交互提供了、高性能、低延迟的方案。适合在集群数量偏少的情况下使用（50台以下集群环境）。当然、它也可以在大型集群环境下使用，由于未引入Zookeeper支持，所以它在大型集群环境下不够成熟，例如服务发现以及监控都没有做，但是作为RPC框架来用已经足够，至少比使用rest、webservice等性能高得多，也比直接使用thrift、avro等方便的多。

为了让它保持小巧、简单，所以不打算引入Zookeeper支持。我认为50台server组成的集群，已经可以满足绝大部分需求，所以简单、小巧、高性能才是最重要的。如果你认为简单不重要，或者成熟度是最重要的，那么淘宝的Dubbo在等着你。http://dubbo.io/

## 背景以及需求
心血来潮，在公司很无聊，这才是主要原因。 其次是我们要对系统模块进行拆分，从原系统移出，那么就需要寻找一个远程调用工具。
1、基于HTTP（rest、webservice等） 主要性能很差，其次是难以支持高并发，而且组装HTTP请求也比较麻烦，难以形成规范，所以此项被pass。
2、基于thrift，性能虽好，但是使用起来非常麻烦，需要频繁的生成代码，而且对Client开发者要求较高，需要自己写连接池，长连接无法使用LVS还要写负载均衡和容错。而且thrift的服务端需要将业务逻辑全部放在一个接口（一个接口就需要发布一个服务，占用一个线程池），这将是个很恶心的事，所以也被pass。

正因为以上两点，所以我打算自己写一个框架。要求是：简单小巧、依赖少、高性能、高并发、支持集群、负载均衡、容错。无学习成本，源码简单可定制修改，我认为这些才是最主要的。

如果你也在寻找这样一个框架，那么很值得看一下。

## 框架详解
我做完了这个框架，没多久便发现了Dubbo，在看了Dubbo的设计以后，惊喜的发现此框架和Dubbo的核心功能几乎一样。

说起来很简单，就是框架会在Client端代理一个接口，调用这个接口的方法，将发送远程请求，参数序列化传递到远程Server端，Server端处理业务逻辑，完成后、将返回结果序列化给Client端，作为被调用方法的返回值。因此整个过程对用户是透明的。

项目底层使用thrift，这是为了使用thrift的各种Server模型，以此来支持高并发，低延迟。没有使用Netty，原因是Netty较重，延迟要比thrift稍高一些，Netty适合处理高吞吐的异步IO，对于RPC的同步调用没有好处。Netty并不适合。您不用担心thrift性能有问题，也不用担心thrift框架太重，我做过测试，性能和直接使用soket通信几乎不相上下，thrift框架的代码特别少，仅仅是对soket的简单封装，框架非常轻便。

序列化工具使用kryo，这也是性能的关键，您也可以自己去查一下kryo相关资料，这里就不说他了，序列化结果很小，速度很快就是了。

框架依赖 thrift、kryo、commons-pool、spring-beans（其中kryo可以自行替换为您喜欢的序列化工具）

集群支持随机负载均衡，轮询负载均衡（您也可以自己写负载均衡实现），优雅停机(kill pid不要加-9)，容错（集群某几台挂掉并不影响服务）

线程模型 以ThriftThreadPoolServer、ThriftTThreadedSelectorServer 两种为主，具体细节参考thrift（您也可以自己实现Server）

## 使用例子

### 服务端：
/appchina-rpc-test/src/main/java/main/Server.java  
/appchina-rpc-test/src/main/resources/application-server.xml

### 客户端：
/appchina-rpc-test/src/main/java/main/Client.java  
/appchina-rpc-test/src/main/resources/application-client.xml
