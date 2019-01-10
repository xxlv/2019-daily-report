## 2019年1月10日 HZ 多云  风暴-0110

> 任何系统都存在Bug，被发现只是时间问题

根据这条著名的bug存在定理，今天发现的一切bug都将有一个合理的解释，至少看上去是这样的。

诡异的情况发生在部署一个content service微服务的时候，build failed.

于是在服务器执行

```
java -jar -Xms128m -Xmx512m -Djava.security.egd=file:/dev/./urandom -Dspring.profiles.active=dev /opt/jar/x-content-service.jar
```

发现最后exit -1 

定位到发生错误的位置发生在

``` 
2019-01-10 16:05:48,702 WARN  [localhost-startStop-2] org.apache.catalina.loader.WebappClassLoaderBase : The web application [ROOT] appears to have started a thread named [AsyncReporter{OkHttpSender{http://x.x.x.x:9411/api/v2/spans}}] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 sun.misc.Unsafe.park(Native Method)
 java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
 java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2078)
 zipkin2.reporter.ByteBoundedQueue.drainTo(ByteBoundedQueue.java:81)

```

猜测最可能的错误发生在


```
This is very likely to create a memory leak
```

内存不足？

执行
``` shell
free -h
```

得到的结果是

```
              total        used        free      shared  buff/cache   available
Mem:            15G         13G        651M        1.7M        1.3G        1.5G
Swap:            0B          0B          0B

```

What ? 竟然只剩下651M free space.. 真可恶。

执行```jps ``` 发现

```
9920 xxx-api.jar
11393 xxx-user-service.jar
4289 xxx-order-service.jar
19523 xxx-marketing-service.jar
7236 xxx-auth-service.jar
6544 Jps
5521 xxx-login-service.jar
5713 xxx-finance-service.jar
13972 xxx-data-api.jar
13592 xxx-data-service.jar
6107 xxx-content-service.jar
31324 xxx-risk-service.jar
12637 xxx-user-api.jar
20638 xxx-work-order-service.jar
19742 xxx-marketing-api.jar
20834 xxx-work-order-api.jar
31715 xxx-risk-api.jar
24361 xxx-delivery-api.jar
8748 xxx-message-service.jar
14701 xxx-delivery-service.jar
9071 xxx-message-api.jar
12207 xxx-user-chat.jar
5872 xxx-login-api.jar
3253 xxx-test-service.jar
11319 xxx-product-service.jar

```


Wow  這麽多業務，竟然用15G內存。 拜拜了您~

接着，我让我们的运维小哥加内存，但是告知太贵，拜托今天刚发工资好吧，咱们能不能有点职业素养自己买点内存争取让项目上线早日让老板开上玛莎拉蒂？


运维的解决方案是使用了另一台备用服务来解决这个问题，于是content service被移动到了服务器B上。 

然后不幸总是接二连三的，马上新的问题出现了。

Oops 

新部署的机器跟zk的地址ping不同，服务启动了但是注册不到注册中心....

想着不在麻烦运维,快速键入

``` shell
iptables -L -n -v
```
结果提示

```
iptables v1.4.21: can't initialize iptables table `filter': Permission denied (you must be root)
Perhaps iptables or your kernel needs to be upgraded.
```

OKOK,我放弃。

NO ROOT NO CODE 

最后再次找了运维小哥。解决了改问题。


### Java技术

在php中，有一个函数叫 array_intersect .这个函数返回两个数组的交集。

在java中，如何获取两个list的交集呢？

``` java
  copyList.retainAll(targetList);
```


### 编程思考

在系统设计最初的时候，需要尽可能的考虑到很多的东西，在精简核心功能的同时，一些扩展必要信息是要提前保存的。即便是设置一个hook。 比如在登陆后调用loginSuccessHook，总是有益的。



