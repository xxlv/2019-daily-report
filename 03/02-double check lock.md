## 2019年3月03日 HZ 天气阴  风暴-0303

有一个常见的Java中生成单例的范式:



``` java
public class Singleton {
    private volatile static Singleton uniqueSingleton;

    private Singleton() {
    }

    public Singleton getInstance() {
        if (null == uniqueSingleton) {
            synchronized (Singleton.class) {
                if (null == uniqueSingleton) {
                    uniqueSingleton = new Singleton();
                }
            }
        }
        return uniqueSingleton;
    }
}
```

在这个片段中，有趣的地方有两个，

- volidate 修饰的uniqueSignleton
- double check
volidate表明了uniqueSignleton的内存可见性，可以阻止虚拟机的指令重排。

指令重排是JVM并不保证执行的顺序是书写的顺序，但保证最终的结果一致。 

很多情况下JVM都会多指令进行重排和优化。