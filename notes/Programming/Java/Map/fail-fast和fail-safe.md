下面是Hashtable迭代器代码

```java
public class Map {
    public static void main(String[] args) {
        Hashtable<String,String> table = new Hashtable<>();
        table.put("1","1");
        table.put("2","2");
        table.put("3","3");
        table.put("4","4");

        Iterator<String> iterator = table.keySet().iterator();
        while (iterator.hasNext()) {
            //抛出java.util.ConcurrentModificationException
            table.put("666","666");
            iterator.next();
        }

        Enumeration<String> enumeration = table.keys();
        while(enumeration.hasMoreElements()){
            table.put("666","666");
            enumeration.nextElement();
        }
        System.out.println(table);
    }
}
```

根据上面来看Hashtable有两种迭代器，一种是和HashMap一样的iterator，这种迭代器是fail-fast，而他有另一种迭代器，那就是enumeration，他的用法和iterator是类似的，但是这种迭代器是fail-safe，在遍历的时候如果我们修改值是不会报错的。

快速失败（fail—fast）：java集合中的一种机制。fail-fast机制在遍历一个集合时如果集合结构被修改就会抛出java.util.ConcurrentModificationException。fail-fast会在下面两种情况下抛出异常：
1. 单线程环境中在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出java.util.ConcurrentModificationException
2. 多线程环境中当一个线程遍历这个集合，而另一个线程对这个集合的结构进行了修改

原理：迭代器在遍历时直接访问集合元素中的内容，并且在遍历过程中使用一个modCount变量。集合在被遍历期间如果内容发生了变化，就会修改modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。（这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出）

java.util下的集合类都是fail-fast，不能在多线程下发生并发修改，可以认为是一种安全机制

安全失败（fail—safe）：fail-safe任何对集合结构的修改都会在一个复制的集合上进行修改，因此不会抛出ConcurrentModificationException

fai-safe有两个问题：
1. 需要复制集合，产生大量的无效对象，开销很大
2. 无法保证读取的数据是目前原始结构中的数据

下面看下HashMap和ConcurrentHashMap的迭代器的代码

HashMap：

```java
public class FailFast {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        map.put("1", "1");
        map.put("2", "2");
        map.put("3", "3");
        map.put("4", "4");

        Iterator<String> iterator = map.keySet().iterator();

        while (iterator.hasNext()) {
        //抛出java.util.ConcurrentModificationException
            map.put("666", "666");
            iterator.next();
        }
        System.out.println(map);
    }
}
```

输出结果：
```text
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1442)
	at java.util.HashMap$KeyIterator.next(HashMap.java:1466)
	at com.wjx.osiris.lucian.collect.FailFast.main(FailFast.java:26)
```

ConcurrentHashMap
```java
public class FailSafe {
    public static void main(String[] args) {
        Map<String, String> map = new ConcurrentHashMap<>();
        map.put("1", "1");
        map.put("2", "2");
        map.put("3", "3");
        map.put("4", "4");

        Iterator<String> iterator = map.keySet().iterator();

        while (iterator.hasNext()) {
            map.put("666", "666");
            iterator.next();
        }
        System.out.println(map);
    }
}
```
输出结果
```text
{1=1, 2=2, 3=3, 4=4, 666=666}
```