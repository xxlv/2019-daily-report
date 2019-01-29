## 2019年1月28日 HZ 晴朗  风暴-0128


### Exchanger 
>
 A synchronization point at which threads can pair and swap elements
  within pairs.  Each thread presents some object on entry to the
  {@link #exchange exchange} method, matches with a partner thread,
  and receives its partner's object on return.  An Exchanger may be
  viewed as a bidirectional form of a {@link SynchronousQueue}.
  Exchangers may be useful in applications such as genetic algorithms
  and pipeline designs.

Exchanger用来在线程间交换数据，简单的来说，线程A调用了exchange

Thread A 
```
exchange.exchange("PointA")
```

此时Thread A 会进入阻塞状态。

ThreadB  此时调用了

```
exchange.exchange("PointB")
```

此时，ThreadA获取PointB,ThreadB 获得PointA.


下面是一段demo 

``` java

    public static void main(String[] args) {
        ExecutorService executorService=Executors.newCachedThreadPool();
        final Exchanger exchanger=new Exchanger();
        executorService.execute(new Runnable() {
            String data1="1";
            @Override
            public void run() {
                doExchangeWork(data1,exchanger);
            }
        });

        executorService.execute(new Runnable() {
            String data2="b";
            @Override
            public void run() {
                doExchangeWork(data2,exchanger);
            }
        });
        executorService.shutdown();
    }

    public static void doExchangeWork(String data,Exchanger exchanger){
        try {
            System.out.println(Thread.currentThread().getName() + "正在把数据 " + data + " 交换出去");
            Thread.sleep((long) (Math.random() * 1000));
            String data2 = (String) exchanger.exchange(data);
            System.out.println(Thread.currentThread().getName()+"此时应该在等待，输");
            System.out.println(Thread.currentThread().getName() + "交换数据 到  " + data+"->"+data2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

```

场景分析：

考虑一种这样的场景，对象消费，A线程创建的对象需要被其他线程消费，对象O的创建成本非常高，我们希望在线程A创建的对象能够接力被B线程消费。 这样，在B线程不需要重建O对象。此时，就可以通过exchange来交换对象。

这个理解是在网上查询到的，根据JDK的doc来看，exchange被描述为一种线程间交换数据的工具，适用于管道设计和遗传算法。

对于管道设计场景，exchange其实很容易理解。  传统的Unix下的管道

``` shell

commandA  | commandB
```

commandA 的命令执行的结果将被输入到CommandB上。  线程A的对象在可以很容易的传递到线程B上，非常相似的思路。


### 实现原理

如何设计exchange 这个工具呢？ 

首先这个功能具有如下的能力
-  当调用了exchange 之后，检测当前的对象是否已经有了被其他线程exchange的标记，如果有，交换数据。notifyAll
-  如果没有检测到，则设置exchange标记，并wait 

TODO 






