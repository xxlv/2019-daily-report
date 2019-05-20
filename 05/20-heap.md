## 2019年5月20日 HZ 天气阴郁转晴  风暴-2019年5月20日

### priorityQueue

priorityQueue 是一个优先级队列。queue是有序的，按照compator所定义的规则进行排序。
在JDK1.8中，priorityQueue是采用Heap作为底层的数据结果。定义一个priorityQueue的过程是一个heapify的过程。

### heapify

插入数据：```bash  [1,23,32,3,41] ```到queue中，将构建如下的Heap结构(图为使用dot生成 https://graphviz.gitlab.io/_pages/doc/info/lang.html)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/01.png)  

![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/02.png)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/03.png)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/04.png)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/05.png)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/06.png)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/07.png)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/08.png)


![Image text](https://raw.githubusercontent.com/xxlv/2019-daily-report/master/resources/images/05/09.png)



这个offer的过程，是一个 **siftUp** 的过程
