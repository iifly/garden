# Java容器知识体系

## 容器

- 容器，就是可以容纳其他Java对象的对象。
- Java 容器里只能放对象，对于基本类型，需要将其包装成对象类型才能放到容器里。
- 容器主要包括 `Collection` 和 `Map` 两种，`Collection` 存储着对象的集合，而 `Map` 存储着键值对(两个对象)的映射表。

![Java容器知识体系](imgs/Java容器知识体系.png)
## Collection

### List

- `List` 是一个**有序**、**可重复**的集合，其中每个元素都有其对应的索引。
- `List` 默认按元素的添加顺序设置元素的索引，第一个添加到 `List` 集合中的元素的索引为 `0`，第二个为 `1`，依此类推。
- `List` 可以通过索引来访问指定位置的元素。

#### ArrayList

- `ArrayList` 是通过**数组**实现，且允许放入 `null` 元素。
- **支持随机访问**
- 对尾部成员的增加和删除支持较好，其他位置插入与删除元素的速度相对较慢。
- 向容器中添加元素时，如果容量不足，容器会**自动扩容**，增大底层数组的大小。
- 数组扩容会将老数组中的元素重新拷贝一份到新的数组中，新的数组容量大约是原来的 `1.5` 倍。
- 扩容操作的代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。
- `ArrayList` 也采用了快速失败的机制（Fail-Fast），通过记录 `modCount` 参数来实现。
- [ArrayList源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.1-ArrayList源码解析.md)

#### LinkedList

- `LinkedList` 基于**双向链表**实现，且允许放入 `null` 元素。
- 不支持随机访问，只能从头或尾开始顺序遍历访问。
- 可以快速地在链表中间插入和删除元素。
- `LinkedList` 还实现了 `Queue` 接口，可以用作**栈**、**队列**和**双向队列**。
- [LinkedList源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.2-LinkedList源码解析.md)

#### Vector

和 `ArrayList` 类似，但它是线程安全的。

#### Stack

栈，先进后出的数据结构，**已不推荐使用**！

### Set
- `Set` 是一个**无序**、**不可重复**的集合，最多只允许包含一个 `null` 元素。
- `Set` 集合**通常**不能记住元素的添加顺序，使用 `Iterator` 遍历 `HashSet` 得到的结果是不确定的。

#### HashSet

- `HashSet` 基于**哈希表**实现，具有很好的**存取和查找性能**。
- `HashSet` 不是同步的，如果多个线程同时访问或修改一个 `HashSet`，则必须通过代码来保证其同步。
- [HashMap与HashSet源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.3-HashMap与HashSet源码解析.md)

#### LinkedHashSet

- 在 `HashSet` 的基础上使用**双向链表**维护元素的插入顺序，使之既有 `HashSet` 的查找效率，又保证元素的有序性。
- [LinkedHashMap与LinkedHashSet源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.4-LinkedHashMap与LinkedHashSet源码解析.md)

#### TreeSet

- 基于**红黑树**实现，支持有序性操作。
- 但是查找效率不如 `HashSet`，`HashSet` 查找的时间复杂度为 O(1)，`TreeSet` 则为 O(logN)。 
- `TreeSet` 中的元素必须实现 `Comparable` 接口并重写 `compareTo()` 方法，`TreeSet` 判断元素是否重复 、以及确定元素的顺序 靠的都是这个方法；  
	- 对于Java 类库中定义的类，`TreeSet` 可以直接对其进行存储，如 `String`，`Integer` 等，因为这些类已经实现了`Comparable` 接口  
	- 对于自定义类，如果不做适当的处理，`TreeSet` 中只能存储一个该类型的对象实例，否则无法判断是否重复。
- [TreeMap与TreeSet源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.5-TreeMap与TreeSet源码解析.md)

### Queue

- 队列，先进先出的数据结构。

#### LinkedList

- 基于**双向链表**实现的双端队列。
- [LinkedList源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.2-LinkedList源码解析.md)

#### ArrayDeque

- 基于**数组**实现的双端队列，队列使用的首选，次选是 `LinkedList`，也可以当做栈来使用。
- [ArrayDeque 源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.6-ArrayDeque源码解析.md)

#### PriorityQueue

- 基于**堆**结构实现的优先队列。
- [PriorityQueue源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.7-PriorityQueue源码解析.md)

## Map

### HashMap

- 基于哈希表实现。
- [HashMap与HashSet源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.3-HashMap与HashSet源码解析.md)

### HashTable

- 和 `HashMap` 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 `HashTable` 并且不会导致数据不一致。
- 它是弃用类，不应该去使用它。 现在可以使用 `ConcurrentHashMap` 来支持线程安全，并且 `ConcurrentHashMap` 的效率会更高，因为 `ConcurrentHashMap` 引入了分段锁。

### LinkedHashMap

- 在 `HashMap` 的基础上使用**双向链表**来维护元素的顺序，顺序为插入顺序或者最近最少使用(LRU)顺序。
- [LinkedHashMap与LinkedHashSet源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.4-LinkedHashMap与LinkedHashSet源码解析.md)

### TreeMap

- 基于**红黑树**实现。
- [TreeMap与TreeSet源码解析](docs/Java/J2-Java进阶/Java容器/J2-1.5-TreeMap与TreeSet源码解析.md)
