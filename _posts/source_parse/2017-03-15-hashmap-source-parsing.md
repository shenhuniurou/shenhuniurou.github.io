---
layout: post
title: HashMap实现原理及源码分析
category: 源码解析
tags: HashMap
---


HashMap 是基于哈希表的 Map 接口的非同步实现。此实现提供所有可选的映射操作，并允许使用 null 键和 null 值。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。之所以 HashMap 的应用场景这么多主要是因为它的性能相比其他的数据结构要好，这里就不得不提一下数组、链表以及二叉树对插入删除和查找操作的性能问题了。

- 数组，采用一段连续的存储单元来存储数据。对于指定下标的查找，时间复杂度为O(1)；通过给定值进行查找，需要遍历数组，逐一比对给定关键字和数组元素，时间复杂度为O(n)，当然，对于有序数组，则可采用二分查找，插值查找，斐波那契查找等方式，可将查找复杂度提高为O(logn)；对于一般的插入删除操作，涉及到数组元素的移动，其平均复杂度也为O(n)

- 链表，对于链表的新增，删除等操作（在找到指定操作位置后），仅需处理结点间的引用即可，时间复杂度为O(1)，而查找操作需要遍历链表逐一进行比对，复杂度为O(n)

- 二叉树，对一棵相对平衡的有序二叉树，对其进行插入，查找，删除等操作，平均复杂度均为O(logn)。

- 哈希表：相比上述几种数据结构，在哈希表中进行添加，删除，查找等操作，性能十分之高，不考虑哈希冲突的情况下，仅需一次定位即可完成，时间复杂度为O(1)。

我们知道，数据结构的物理存储结构只有两种：顺序存储结构和链式存储结构，而在上面我们提到过，在数组中根据下标查找某个元素，一次定位就可以达到，哈希表利用了这种特性，哈希表的主干就是数组。比如我们要新增或查找某个元素，我们通过把当前元素的关键字通过某个函数映射到数组中的某个位置，通过数组下标一次定位就可完成操作。


> 哈希冲突：如果两个不同的元素，通过哈希函数得出的实际存储地址相同怎么办？也就是说，当我们对某个元素进行哈希运算，得到一个存储地址，然后要进行插入的时候，发现已经被其他元素占用了，其实这就是所谓的哈希冲突，也叫哈希碰撞。前面我们提到过，哈希函数的设计至关重要，好的哈希函数会尽可能地保证计算简单和散列地址分布均匀，但是，我们需要清楚的是，数组是一块连续的固定长度的内存空间，再好的哈希函数也不能保证得到的存储地址绝对不发生冲突。那么哈希冲突如何解决呢？哈希冲突的解决方案有多种：开放定址法（发生冲突，继续寻找下一块未被占用的存储地址），再散列函数法，链地址法，而 HashMap 即是采用了链地址法，也就是数组+链表的方式。


**HashMap实现原理**


上面说到 HashMap 的主干其实是数组，没错，具体点其实是 HashMapEntry 类型的数组，初始值为空数组{}，主干数组的长度一定是2的次幂。看如下定义：


```
/**
 * An empty table instance to share when the table is not inflated.
 */
static final HashMapEntry<?,?>[] EMPTY_TABLE = {};

/**
 * The table, resized as necessary. Length MUST Always be a power of two.
 */
transient HashMapEntry<K,V>[] table = (HashMapEntry<K,V>[]) EMPTY_TABLE;
```

HashMapEntry 是 HashMap 的静态内部类，它实现了Map.Entry<K,V>接口，具体实现如下：

```
static class HashMapEntry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    HashMapEntry<K,V> next;// 存储指向下一个Entry的引用，单链表结构
    int hash;// 对key的hashcode值进行hash运算后得到的值，存储在Entry，避免重复计算

    /**
     * Creates new entry.
     */
    HashMapEntry(int h, K k, V v, HashMapEntry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }

    public final K getKey() {
        return key;
    }

    public final V getValue() {
        return value;
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }

    public final int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    public final String toString() {
        return getKey() + "=" + getValue();
    }

    /**
     * This method is invoked whenever the value in an entry is
     * overwritten by an invocation of put(k,v) for a key k that's already
     * in the HashMap.
     */
    void recordAccess(HashMap<K,V> m) {
    }

    /**
     * This method is invoked whenever the entry is
     * removed from the table.
     */
    void recordRemoval(HashMap<K,V> m) {
    }
}
```

我上面提到 HashMap 采用链地址法也就是由数组+链表组成的方式来解决哈希冲突，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前 HashMapEntry 的 next 指向 null ）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度依然为O(1)，因为最新的 HashMapEntry 会插入链表头部，只需要简单改变引用链即可，而对于查找操作来讲，此时就需要遍历链表，然后通过 key 对象的 equals 方法逐一比对查找。所以从性能方面考虑， HashMap 中的链表出现越少，它的性能才会越好。

HashMap 类内部还维护了其他几个重要的字段，来看看它们的含义：

```
// 默认的初始化容量，JDK源码中默认是16，而Android源码中它的默认值是4，它必须是2的次幂
static final int DEFAULT_INITIAL_CAPACITY = 4;

// 最大容量值，2^30
static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 主干数组的初始值
static final HashMapEntry<?,?>[] EMPTY_TABLE = {};

// 主干数组，长度必须为2的次幂
transient HashMapEntry<K,V>[] table = (HashMapEntry<K,V>[]) EMPTY_TABLE;

// 存储的键值对数
transient int size;

// 阈值，当 table == {}时，该值为初始容量（默认为4）；当 table 被填充了，也就是为 table 分配内存空间后，threshold 一般为 
// capacity * loadFactory。 HashMap 在进行扩容时需要参考 threshold。
int threshold;

//负载因子，代表了 table 的填充度有多少，默认是0.75
final float loadFactor = DEFAULT_LOAD_FACTOR;

// HashMap 被修改的次数，用于快速失败，由于 HashMap 非线程安全，在对HashMap进行迭代时，如果期间其他线程的参与导致HashMap的结构发生变化了（比如 put，remove 
// 等操作），需要抛出异常 ConcurrentModificationException
transient int modCount;
```

接着来看 HashMap的构造器，它有四个构造器：

```
public HashMap(int initialCapacity, float loadFactor) {

	// 长度最大不能超过2^30
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY) {
        initialCapacity = MAXIMUM_CAPACITY;
    } else if (initialCapacity < DEFAULT_INITIAL_CAPACITY) {
        initialCapacity = DEFAULT_INITIAL_CAPACITY;
    }

    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
  
    threshold = initialCapacity;
    init();
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}

public HashMap(Map<? extends K, ? extends V> m) {
    this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1, DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
    inflateTable(threshold);
    putAllForCreate(m);
}
```

init 方法没有实现，不过在子类 LinkedHashMap 中就有对应实现，在常规构造器中，没有为数组 table 分配内存空间（有一个入参为指定Map的构造器例外），而是在执行 put 操作的时候才真正构建 table 数组，看看 put 操作：

```
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = sun.misc.Hashing.singleWordWangJenkinsHash(key);
    int i = indexFor(hash, table.length);
    for (HashMapEntry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```

我们看到当判定 HashMap 是空表时，它会调用 inflateTable(threshold) 方法，这个方法的实现如下：

```
private void inflateTable(int toSize) {
    // Find a power of 2 >= toSize
    int capacity = roundUpToPowerOf2(toSize);

    // Android-changed: Replace usage of Math.min() here because this method is
    // called from the <clinit> of runtime, at which point the native libraries
    // needed by Float.* might not be loaded.
    float thresholdFloat = capacity * loadFactor;
    if (thresholdFloat > MAXIMUM_CAPACITY + 1) {
        thresholdFloat = MAXIMUM_CAPACITY + 1;
    }

    threshold = (int) thresholdFloat;
    table = new HashMapEntry[capacity];
}
```

我们看到 HashMap 的主干数组是这个时候才创建的，而它的长度 capacity 则是由 `roundUpToPowerOf2` 这个方法计算得来，参数 toSize 就是 初始化的 threshold 值。

```
private static int roundUpToPowerOf2(int number) {
    // assert number >= 0 : "number must be non-negative";
    int rounded = number >= MAXIMUM_CAPACITY
            ? MAXIMUM_CAPACITY
            : (rounded = Integer.highestOneBit(number)) != 0
                ? (Integer.bitCount(number) > 1) ? rounded << 1 : rounded
                : 1;

    return rounded;
}
```

如果 key 为 null，存储位置在 table[0] 或者 table[0] 的冲突链上，接着对 key 进行哈希运算，得到哈希值，然后根据哈希值和表长，得到该 key 在表中对应的位置索引，再遍历这个索引下的数据及其链表，如果该对应数据已存在，执行覆盖操作。用新 value 替换旧 value ，并返回旧 value ，直到该数据节点下所有的数据都往后移了一位。最后再添加这个 HashMapEntry，再来看看这个 addEntry 方法：

```
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? sun.misc.Hashing.singleWordWangJenkinsHash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}
```

当表中数据个数即将超过临界阈值 threshold 且将要发生哈希冲突时，就调用 resize 方法进行扩容，扩容时，需要新建一个长度为之前数组2倍的新的数组，然后将当前的 HashMapEntry 数组中的元素全部传输过去，扩容后的新数组长度为之前的2倍，所以扩容相对来说是个耗资源的操作。


我们之前提到了 HashMap 的长度必须是2的次幂，我们先来看看它扩容的方法 resize 的具体实现：

```
void resize(int newCapacity) {
    HashMapEntry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    HashMapEntry[] newTable = new HashMapEntry[newCapacity];
    transfer(newTable);
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

扩容之后数组中的索引也会改变，需要重新计算 index，而方法 transfer 就是将数据重新计算 index 并从旧表中迁移到新表。

```
/**
 * Transfers all entries from current table to newTable.
 */
void transfer(HashMapEntry[] newTable) {
    int newCapacity = newTable.length;
    for (HashMapEntry<K,V> e : table) {
        while(null != e) {
            HashMapEntry<K,V> next = e.next;
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```

再回到 HashMap 的长度必须是2的整数次幂这个问题上来，因为它在计算索引时，是根据 `hash & (length - 1)` 这个公式来的，当表长为2的次幂时，length - 1换算成二进制，其低位部分全部是1，再&上 hash 时，就可以保证最终的结果只和hash的低位部分有关，而计算 hash 时就做了散列优化，所以最后得到的索引在数组上就可以散列分布，有效避免了哈希冲突。



get 操作实现：

```
public V get(Object key) {
	// 如果 key 为 null ，则直接去 table[0] 处去检索即可。
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}
```

```
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : sun.misc.Hashing.singleWordWangJenkinsHash(key);
    for (HashMapEntry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

这没什么可说的，和 put 的操作相反。不过要注意的是在遍历链表节点时，有一个判断 e.hash == hash，这个判断是必须要的，如果传入的 key 对象重写了 equals 方法却没有重写 hashCode，而恰巧此对象定位到这个数组位置，如果仅仅用 equals 判断可能是相等的，但其 hashCode 和当前对象不一致，这种情况，根据 Object 的 hashCode 的约定(对象的 hashCode 返回值跟其存储地址有关)，不能返回当前对象，而应该返回 null。我们在进行 get 和 put 操作的时候，使用的 key 通过 equals 比较是相等的，但由于没有重写 hashCode 方法，所以 put 操作时，key(hashcode1)-->hash-->indexFor-->最终索引位置 ，而通过 key 取出 value 的时候 key(hashcode2)-->hash-->indexFor-->最终索引位置，由于 hashcode1 不等于 hashcode2，导致没有定位到一个数组位置而返回逻辑上错误的值 null（也有可能碰巧定位到一个数组位置，但是也会判断其 entry 的 hash 值是否相等。）所以，在重写equals的方法的时候，必须注意重写hashCode方法，同时还要保证通过equals判断相等的两个对象，调用hashCode方法要返回同样的整数值。



**HashMap非线程安全的原因**

我们知道 HashMap 是非线程安全的，在并发环境下，可能会形成环状链表，导致 get 操作时，cpu 空转，所以，在并发环境中使用 HashMap 是非常危险的。下面结合源码来分析下 HashMap 费线程安全的原因以及环状链表形成的原因。

1、resize死循环

HashMap 的初始容量是16，一般来说，当有数据要插入时，都会检查容量有没有超过设定的 thredhold ，如果超过，需要增大Hash表的尺寸，但是这样一来，整个 Hash 表里的元素都需要被重算一遍。这叫 rehash，这个成本相当的大。也就是 `addEntry` 这个方法，当超过 thredhold 时，会调用 resize 方法，这个方法做两件事，一是计算新的长度并生成一个新的 HashMapEntry 数组，二是把旧数组中的元素转移到新的数组来，这个转移过程在 transfer 方法中实现，代码这里就不贴了，上面有，我们看看这个方法中的步骤：

```
while(null != e) {
    HashMapEntry<K,V> next = e.next;
    int i = indexFor(e.hash, newCapacity);
    e.next = newTable[i];
    newTable[i] = e;
    e = next;
}
```

1、对索引数组中的元素遍历
2、对链表上的每一个节点遍历：用 next 取得要转移那个元素的下一个，将 e 转移到新 Hash 表的头部，使用头插法插入节点
3、循环2，直到链表节点全部转移
4、循环1，直到所有索引数组全部转移


因为是单链表，转移头指针时必须记住next节点，不然转移后链表就丢了，然后 e 要插入到链表的头部，所以要先用 e.next 指向新的 Hash 表第一个元素，接着将新 Hash 表的头指针指向 e，然后转移 e 的下一个结点，这也就是 rehash 的过程，在单线程中是没有问题的，如果在多线程中， 假设有两个线程同时在进行 put 操作，并且进入了 transfer 阶段，再假设线程1在执行完 `HashMapEntry<K,V> next = e.next;` 后被挂起了，因为访问的是同一份数据，此时线程2 rehash 后链表的顺序其实是和原来相反的，比如以前是3->5->7，扩容后如果还在同一链表的话，链表顺序就会变成7->5->3，在线程1中 next 指向5，但是线程2继续操作时取5的 next，又指向了3，即 a.next = b，而 b.next = a，这样就导致了环状链表出现。



