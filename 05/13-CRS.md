## 2019年5月13日 HZ 天气阴雨  风暴-0513


今天的天气并不是很好，早上骑车中途伴随着淅淅沥沥的小雨。空气闷热，传递的衣服也有点热。

早上做完了一个一个CRS的问题，

下午继续在咖啡馆工作。晚上梳理了一下最近的计划。

## CRS
CRS（Current Real Status）。
> 场景是这样的，用户和专家私聊的时候,需要给与一定的特殊提醒。提醒依赖于当前的聊天状态。
比如如果当前的专家正在跟多个用户聊天，就会警告专家。如果当前的用户正在跟多个专家聊天，就会提醒专家注意提高服务质量。

- 当一方长时间未回复需要作出特殊的提醒。
- 需要提醒专家，用户同时在跟多个专家聊天

要满足这样的需求，就需要维护一张全局实时状态表，存储当前的状态。模型是这样设计的，
一个用户有多个relation，每一个relation是保存该用户和其他用户的连接。

``` java
## Relation
private String lastMessage;
private String relationId;
private String business;
private Timer timer;
```

Timer是一个维持当前通话Session的计时器。一旦离开太久，该relation就要失效。

为了简化工作，我将Timer设计的极可能的简单，

```java
### Timer
private Long startTime
private Long lastTouchTime
private Long alive;
```

在内存中维护一张CRS_TABLE用来保存全部用户的状态表（可持久到Redis）。为了将消息转换为对CRS的更新。
需要提高一个StatusMachine.

```java
### StatusMachine

Future<V> accept(Message message);

```

接受一个message后，将解析message中的用户和专家，并且更新或者创建对应的CRS，注意，这个更新动作需要被设置为异步执行。

```java
  executors.submit(()->updateCRS(message))
```

这样做的目的是，accept是在用户和专家进行消息发送的时候触发的，频率非常之高。我们要确保该接口并不会影响到私聊的核心用户体验。

还有一个非常重要的点在于，如果突然宕机，状态在内存中就会消失，需要重建状态。优先考虑的方案是Redis持久化，在初始化项目的时候恢复。每次状态的更新对写入到Redis.


核心思路如下：

Relation描述了一种简单的关联关系，绑定了Timer可以指示该Relation的时效性，而CRS持久多个relation。一个用户持久一个CRS来维护当前状态信息 Message 被StatusMachine接收后会更新CRS的状态。
