## 2019年1月14日 HZ 多云  风暴-0114

>继续勇敢一点

今天在dubbo的subscribe mail list 上有一个bug（https://github.com/apache/incubator-dubbo/issues/3218）

大概如下

1.  In dubbo2.7 , we modify the ZookeeperTransport to make zookeeper using
 one singleton connection.
      But when the hook is invoked after closing server, it will get
 zookeeper connection for many times for remove some data from zookeeper
 server.  But the connection may be closed by the other hook.
      Removing the zookeeper data , it happened some error for the
 connection is closed.


简单了解了一下，在执行org.apache.dubbo.config.DubboShutdownHook.destroyProtocols之前，已经执行了AbstractRegistryFactory.destroyAll()， 这个方法里面执行了zk的close。

这个提交还没有被合到master上。

接下来会关注这一块如何修复的。

 个人理解，DubboShutdownHook 的unregister必须要执行，然后zk的close才可以。 destoryAll的调用应该在DubboShutdownHook之后。


 













