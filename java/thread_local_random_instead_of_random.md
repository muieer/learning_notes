# 优先选择使用 ThreadLocalRandom 生成随机数

日常开发中，经常需要生成随机数。为了更好的性能，应该优先选择使用`java.util.concurrent.ThreadLocalRandom`生成随机数，而不是`java.util.Random`。

经我性能测试，单线程环境下，ThreadLocalRandom 性能优于 Random。多线程环境下，Random 对象被多个线程共用，ThreadLocalRandom 性能远胜于 Random。

## Random 线程安全但多线程性能差

> Instances of java.util.Random are threadsafe. However, the concurrent use of the same java.util.Random instance across threads may encounter contention and consequent poor performance. Consider instead using java.util.concurrent.ThreadLocalRandom in multithreaded designs.

上面的引用是 Java API 文档中对 Random 线程安全但多线程性能差地描述。

## Random 线程安全实现原理

```java
protected int next(int bits) {
    long oldseed, nextseed;
    AtomicLong seed = this.seed;
    do {
        oldseed = seed.get();
        nextseed = (oldseed * multiplier + addend) & mask;
    } while (!seed.compareAndSet(oldseed, nextseed));
    return (int)(nextseed >>> (48 - bits));
}
```

每次生成随机数，都会调用上面这段代码。每次生成随机数，都需要更新 seed，seed 是一个原子变量。使用 CAS 方式更新 seed，实现线程安全。

但是在多线程环境下，CAS 方式更新失败，就需要循环重试。线程越多，竞争越严重，性能越差。 

ThreadLocalRandom 是基于线程本地变量的原理实现，避免了多线程中的竞态问题，所以多线程环境中性能会更好。

## 性能测试

具体细节见性能测试项目 [java_util_random_benchmark](https://github.com/muieer/java_util_random_benchmark)。
