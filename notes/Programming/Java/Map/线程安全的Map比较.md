由于HashMap在多线程下存在线程安全问题我们一般可以使用下面的三种方法来代替

- 使用Collections.synchronizedMap(Map)创建线程安全的map集合
- Hashtable
- ConcurrentHashMap

```java
private static class SynchronizedMap<K,V>
        implements Map<K,V>, Serializable {
    private static final long serialVersionUID = 1978198479659022715L;

    private final Map<K, V> m;     // Backing Map
    final Object mutex;        // Object on which to synchronize

    SynchronizedMap(Map<K, V> m) {
        this.m = Objects.requireNonNull(m);
        mutex = this;
    }

    SynchronizedMap(Map<K, V> m, Object mutex) {
        this.m = m;
        this.mutex = mutex;
    }

    public int size() {
        synchronized (mutex) {
            return m.size();
        }
    }

    public boolean isEmpty() {
        synchronized (mutex) {
            return m.isEmpty();
        }
    }

    public boolean containsKey(Object key) {
        synchronized (mutex) {
            return m.containsKey(key);
        }
    }

    public boolean containsValue(Object value) {
        synchronized (mutex) {
            return m.containsValue(value);
        }
    }

    public V get(Object key) {
        synchronized (mutex) {
            return m.get(key);
        }
    }

    public V put(K key, V value) {
        synchronized (mutex) {
            return m.put(key, value);
        }
    }

    public V remove(Object key) {
        synchronized (mutex) {
            return m.remove(key);
        }
    }

    public void putAll(Map<? extends K, ? extends V> map) {
        synchronized (mutex) {
            m.putAll(map);
        }
    }

    public void clear() {
        synchronized (mutex) {
            m.clear();
        }
    }
}
```

上面是Collections.SynchronizedMap的源码，根据源码我们可以知道SynchronizedMap内部维护了一个普通对象Map，还有排斥锁mutex。

我们在调用这个方法的时候就需要传入一个Map，可以看到有两个构造器，如果你传入了mutex参数，则将对象排斥锁赋值为传入的对象。如果没有，则将对象排斥锁赋值为this，即调用synchronizedMap的对象，就是上面的Map。创建出synchronizedMap之后，再操作map的时候，就会对方法上锁

Hashtable相对于HashMap而言是线程安全的，但是他的效率先对比较低，因为他的操作都是加锁的，所有的操作都是使用了synchronized进行修饰

Hashtable与HashMap的区别：
1. **实现方式不同**：Hashtable 继承了 Dictionary类，而 HashMap 继承的是 AbstractMap 类
2. **初始化容量不同**：HashMap 的初始容量为：16，Hashtable 初始容量为：11，两者的负载因子默认都是：0.75
3. **扩容机制不同**：当现有容量大于总容量 * 负载因子时，HashMap 扩容规则为当前容量翻倍，Hashtable 扩容规则为当前容量翻倍 + 1
4. **迭代器不同**：HashMap 中的 Iterator 迭代器是 fail-fast 的，而 Hashtable 有两种迭代器，其中的 Enumerator 不是 fail-fast 的

ConcurrentHashMap相对于HashMap线程安全，比上述的Hashtable性能更好

ConcurrentHashMap和Hashtable都是不允许键和值为null的，具体原因为：ConcurrentHashmap和Hashtable都是支持并发的，这样会有一个问题，当你通过get(k)获取对应的value时，如果获取到的是null时，你无法判断，它是put（k,v）的时候value为null，还是这个key从来没有做过映射。HashMap是非并发的，可以通过contains(key)来做这个判断。而支持并发的Map在调用m.contains（key）和m.get(key),m可能已经不同了
