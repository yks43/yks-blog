# 集合概述
## 概览图
![](../images/Java-Collections.jpeg)
## List,Set,Map三者的区别
* list：存储的元素是有序的、可重复的
* set：存储的元素是无序的、不可重复的
* Map：使用键值对（kye-value）存储，Key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值。
## 各个集合的底层数据结构
1. List
* ArrayList: Object[]数组
* Vector: Object[]数组
* LinkedList: 双向链表（JDK1.6之前为循环链表，JDK1.7取消了循环）
2. Set
* HashSet(无序，唯一): 基于HashMap实现的，底层采用HashMap来保存元素
* LinkedHashSet: 是HashSet的子类，内部是通过LinkedHashMap来实现的
* TreeSet(有序，唯一): 红黑树（平衡的排序二叉树）
3. Map
* HashMap： DK1.8 之前 HashMap 由数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间
* LinkedHashMap： LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
* Hashtable： 数组+链表组成的，数组是 HashTable 的主体，链表则是主要为了解决哈希冲突而存在的
* TreeMap： 红黑树（自平衡的排序二叉树）
## 如何选用集合
主要根据集合的特点来选用，比如我们需要根据键值获取到元素值时就选用 Map 接口下的集合，需要排序时选择 TreeMap,不需要排序时就选择 HashMap,需要保证线程安全就选用 ConcurrentHashMap。

当我们只需要存放元素值时，就选择实现Collection 接口的集合，需要保证元素唯一时选择实现 Set 接口的集合比如 TreeSet 或 HashSet，不需要就选择实现 List 接口的比如 ArrayList 或 LinkedList，然后再根据实现这些接口的集合的特点来选用。

## 为什么使用集合？
当我们需要保存一组类型相同的数据的时候，我们应该是用一个容器来保存，这个容器就是数组，但是，使用数组存储对象具有一定的弊端， 因为我们在实际开发中，存储的数据的类型是多种多样的，于是，就出现了“集合”，集合同样也是用来存储多个数据的。

数组的缺点是一旦声明之后，长度就不可变了；同时，声明数组时的数据类型也决定了该数组存储的数据的类型；而且，数组存储的数据是有序的、可重复的，特点单一。 但是集合提高了数据存储的灵活性，Java 集合不仅可以用来存储不同类型不同数量的对象，还可以保存具有映射关系的数据

## Iterator迭代器
### Iterator是什么？
``` java
public interface Iterator<E> {
    //集合中是否还有元素
    boolean hasNext();
    //获得集合中的下一个元素
    E next();
    ......
}
```

Iterator 对象称为迭代器（设计模式的一种），迭代器可以对集合进行遍历，但每一个集合内部的数据结构可能是不尽相同的，所以每一个集合存和取都很可能是不一样的，虽然我们可以人为地在每一个类中定义 hasNext() 和 next() 方法，但这样做会让整个集合体系过于臃肿。于是就有了迭代器。

迭代器是将这样的方法抽取出接口，然后在每个类的内部，定义自己迭代方式，这样做就规定了整个集合体系的遍历方式都是 hasNext()和next()方法，使用者不用管怎么实现的，会用即可。迭代器的定义为：提供一种方法访问一个容器对象中各个元素，而又不需要暴露该对象的内部细节。

### Iterator有啥用？
Iterator 主要是用来遍历集合用的，它的特点是更加安全，因为它可以确保，在当前遍历的集合元素被更改的时候，就会抛出 ConcurrentModificationException 异常。

### 如何使用？
通过使用迭代器来遍历 HashMap，演示一下 迭代器 Iterator 的使用。
``` java
Map<Integer, String> map = new HashMap();
map.put(1, "Java");
map.put(2, "C++");
map.put(3, "PHP");
Iterator<Map.Entry<Integer, String>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
  Map.Entry<Integer, String> entry = iterator.next();
  System.out.println(entry.getKey() + entry.getValue());
}
```
## 哪些集合线程不安全？怎么解决？
常用的 Arraylist,LinkedList,Hashmap,HashSet,TreeSet,TreeMap，PriorityQueue都不是线程安全的，可以使用线程安全的集合来替代
如果要使用线程安全的集合的话， java.util.concurrent 包中提供了很多并发容器供使用：
* ConcurrentHashMap: 可以看作是线程安全的 HashMap
* CopyOnWriteArrayList:可以看作是线程安全的 ArrayList，在读多写少的场合性能非常好，远远好于 Vector.
* ConcurrentLinkedQueue:高效的并发队列，使用链表实现。可以看做一个线程安全的 LinkedList，这是一个非阻塞队列。
* BlockingQueue: 这是一个接口，JDK 内部通过链表、数组等方式实现了这个接口。表示阻塞队列，非常适合用于作为数据共享的通道。
* ConcurrentSkipListMap :跳表的实现。这是一个Map，使用跳表的数据结构进行快速查找。

# Collection子接口之List




