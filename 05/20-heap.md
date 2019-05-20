## 2019年5月20日 HZ 天气阴郁转晴  风暴-2019年5月20日

### priorityQueue

priorityQueue 是一个优先级队列。queue是有序的，按照compator所定义的规则进行排序。
在JDK1.8中，priorityQueue是采用Heap作为底层的数据结果。定义一个priorityQueue的过程是一个heapify的过程。

## heapify

### siftUp

**插入（OFFER）** 数据：```bash  [1,23,32,3,41] ```到queue中，将构建如下的Heap结构(图为使用dot生成 https://graphviz.gitlab.io/_pages/doc/info/lang.html)

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/01.png)

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/02.png)  

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/03.png)

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/04.png)

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/05.png)

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/06.png)

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/07.png)

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/08.png)

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/09.png)



这个offer的过程，是一个 **siftUp** 的过程。

### siftDown

从queue 中移除掉一个element是一个siftDown的过程。在上面的例子中，queue.remove()将移除掉element#41。
具体的原理如下图所示。

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/10.png)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/11.png)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/12.png)



queue 中，remove将移除掉root，并且将最后一个元素放置在root的位置，从上而下，进行siftDown，调整使得二叉树满足heap的特性。如图表示了在一个大顶堆（父元素总不小于它的孩子节点）上进行siftDown的过程。


JDK的源码中，对于这个过程的代码如下，

```java
private void siftDownUsingComparator(int k, E x) {
     int half = size >>> 1;
     while (k < half) {
         int child = (k << 1) + 1;
         Object c = queue[child];
         int right = child + 1;
         if (right < size &&
             comparator.compare((E) c, (E) queue[right]) > 0)
             c = queue[child = right];
         if (comparator.compare(x, (E) c) <= 0)
             break;
         queue[k] = c;
         k = child;
     }
     queue[k] = x;
 }
```
