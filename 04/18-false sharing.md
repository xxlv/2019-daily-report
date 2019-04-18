## 2019年4月18日 HZ 天气晴朗  风暴-0418

##### False sharing  

可以非常快速的遍历在连续的内存块中分配的任意数据结构。

cache line 可以预读8个long 也就是64字节的数据

多线程会cache line 竞争，所以采用padding让目标数据分散在不同的cache line上。


### Mechanical Sympathy



### Ring buffer

环形缓存没有锁开销，内存固定，不会产生内存碎片。

核心原理：
Tail(w)和head(R)

When both pointers are equal, the buffer is empty, and when the end pointer is one less than the start pointer, the buffer is full.


#### Reference
https://en.wikipedia.org/wiki/Circular_buffer#How_it_works
