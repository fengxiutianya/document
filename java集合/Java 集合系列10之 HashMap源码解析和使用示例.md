# Java 集合系列10 之HashMap详细介绍和使用示例

### 概要

>1. HashMap介绍
>2. HashMap源码解析
>3. HashMap遍历方式
>4. HashMap使用示例

### 1. HashMap介绍

>HashMap简介
>
>HashMap是一个散列表，他存储的内容是键值对(key-value)，其中key允许为null。这也是他与HashTable的一个重要区别。
>
>HashMap继承于AbstractMap，实现了Map，Cloneable，java.io.Serualizable接口
>
>HashMap的实现不是同步的，这意味着他不是线程安全的，在oracle给出的官方文档中，如果要并发使用，推介使用下面这种方式，
>
>```
>Map m = Collections.synchronizedMap(new HashMap(...));
>```
>
>HashMap 的实例有两个参数影响其性能：“**初始容量**” 和 “**加载因子**”。容量是哈希表中数组的长度，初始容量 只是哈希表在创建时的容量。加载因子 是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有大约两倍的数组长度。
>通常，**默认加载因子是 0.75**, 这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。
>
>**HashMap的构造函数和API**
>
>```java
>//构造函数
>HashMap()
>HashMap(int initialCapacity)
>HashMap(int initialCapacity,float loadFactor)
>HashMap(Map<? extends k,? extends V> m)
>  
>//函数
>void clear()   //清空集合
>Object clone() //返回本HashMap的一个浅拷贝，key和value本身不拷贝
>boolean containsKey(Object key) //判断是否包含key
>boolean containsValue(Object value) //判断是否包含value
>Set<Map.Entry<K,V>> entrySet();  //返回本集合key-value的Set集合
>V get(Object key)  //返回和指定key匹配的value值
>  //返回和指定key匹配的值，如果没有key，则返回defaultValue值
>V getOrDefault(Object key,V defaultValue)
>boolean isEmpty()   //返回Map是否为空
>Set<K> keySet() // 返回此集合包含key的一个浅拷贝
>V put(K key,V value) //添加 key-value到此Map中
>void putAll(Map<? extends k,? extends V> m) //添加集合到此集合中
>  // 如果指定的key不存在，则添加key-value到集合中，如有，则返回value值
>v putIfAbsent(K key,V value)
>V remove(Object key)              //删除指定key对应的key-value集合
>boolean remove(Object key,Object v)//删除和key-value匹配的entry
>V replace(K key,V value) // 取代key对应的value
>    
>boolean replace(K key,V old,V new)   //当key-old在集合中匹配时，用new 替换value值
>int size() //返回集合的大小
>Collextion<V> values()  //返回Map集合中value值组成的collection
>```
>
>

### 2. HashMap 源码解析

>下面是基于java 8的源码分析，还有一部分没有理解好，所以没怎么进行注解，另外就是java 8新加的关于函数式编程的函数也没有进行解析。
>
>```java
>package java.util;
>
>import java.io.IOException;
>import java.io.InvalidObjectException;
>import java.io.Serializable;
>import java.lang.reflect.ParameterizedType;
>import java.lang.reflect.Type;
>import java.util.function.BiConsumer;
>import java.util.function.BiFunction;
>import java.util.function.Consumer;
>import java.util.function.Function;
>
>/**
> * HashMap 实现了map的基本功能。大体上HashMap和HashTable是相同的，但是HashTable
> * 不允许key-value为控制，同时HashTable是synchronized。
> * 
> * 影响HashMap查找速度快慢的因素有俩个，
> * 1. initial capacity 初始容量
> * 2. load factor 加载因子，决定元素数量超过多少时，自动增加容量 默认值0.75
> *  HashMap 不是线程安全的，如果希望线程安全，可以使用下面的方式
> *   Map m = Collections.synchronizedMap(new HashMap(...));
> * 
> * 
> * HashMap 遍历的方式：keySet entrySet values
> * 三种的遍历速率大概相同，因为三种的实现方式一样。
> *      
> *
> */
>public class HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable {
>
>    private static final long serialVersionUID = 362498820763181265L;
>
>    /**
>     * 实现笔记
>     *  
>     *Map 存储节点
>     * 1. 创建一个数组，对key的hashCode进行散列（利用除余法），找到对应的索引位置，
>     *      然后以链表的方式进行存储多个相同索引位置的节点
>     * 2. 如果数组对应索引的链表长度大于一个值，则将链表转换成红黑树来存储
>     * 3. 如果总的节点数（key-value）个数大于一个指定的值，会影响查找速率，
>     *    所以需要重新进行hash
>     * 
>     *有几点需要注意的：
>     * 1. 如何比较elements，如果俩个elements具有以下格式
>     *      class C implements Comparable<C>
>     *      将使用compareTo来进行比较
>     * 2. 由于TreeNodes的空间大小是node的俩倍大小，所以当值较小时，
>     *      不采用TreeNode 
>     * 3. LoadFactor：加载因子，这个值除非你有确定的替换原因，尽量不要改
>     */
>
>    /**
>     *  初始容量 必须是 2 的幂
>     */
>    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
>
>    /**
>     * 最大容量
>     */
>    static final int MAXIMUM_CAPACITY = 1 << 30;
>
>    /**
>     * 默认加载因子
>     */
>    static final float DEFAULT_LOAD_FACTOR = 0.75f;
>
>    /**
>     * threshold : 节点转换成树的临界值，当小于这个值时就用链表
>     * 
>     */
>    static final int TREEIFY_THRESHOLD = 8;
>
>    /**
>     * 当节点已经是树形时，小于这个临界值时将节点的
>     *  组织方式转换成链表形式
>     */
>    static final int UNTREEIFY_THRESHOLD = 6;
>
>    /**
>     * 节点下面的链表被转换成红黑树的最小设置值
>     */
>    static final int MIN_TREEIFY_CAPACITY = 64;
>
>    /**
>     * Basic hash bin node, used for most entries.  (See below for
>     * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
>     */
>    static class Node<K, V> implements Map.Entry<K, V> {
>        final int hash;
>        final K key;
>        V value;
>        Node<K, V> next;
>
>        Node(int hash, K key, V value, Node<K, V> next) {
>            this.hash = hash;
>            this.key = key;
>            this.value = value;
>            this.next = next;
>        }
>
>        public final K getKey() {
>            return key;
>        }
>
>        public final V getValue() {
>            return value;
>        }
>
>        public final String toString() {
>            return key + "=" + value;
>        }
>
>        public final int hashCode() {
>            return Objects.hashCode(key) ^ Objects.hashCode(value);
>        }
>
>        public final V setValue(V newValue) {
>            V oldValue = value;
>            value = newValue;
>            return oldValue;
>        }
>
>        public final boolean equals(Object o) {
>            if (o == this)
>                return true;
>            if (o instanceof Map.Entry) {
>                Map.Entry<?, ?> e = (Map.Entry<?, ?>) o;
>                if (Objects.equals(key, e.getKey()) && Objects.equals(value, e.getValue()))
>                    return true;
>            }
>            return false;
>        }
>    }
>
>    /* ---------------- Static utilities -------------- */
>
>    /**
>     * Computes key.hashCode() and spreads (XORs) higher bits of hash
>     * to lower.  Because the table uses power-of-two masking, sets of
>     * hashes that vary only in bits above the current mask will
>     * always collide. (Among known examples are sets of Float keys
>     * holding consecutive whole numbers in small tables.)  So we
>     * apply a transform that spreads the impact of higher bits
>     * downward. There is a tradeoff between speed, utility, and
>     * quality of bit-spreading. Because many common sets of hashes
>     * are already reasonably distributed (so don't benefit from
>     * spreading), and because we use trees to handle large sets of
>     * collisions in bins, we just XOR some shifted bits in the
>     * cheapest possible way to reduce systematic lossage, as well as
>     * to incorporate impact of the highest bits that would otherwise
>     * never be used in index calculations because of table bounds.
>     */
>    static final int hash(Object key) {
>        int h;
>        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
>    }
>
>    /**
>     * 返回 x's Class 如果x 满足这种形式"
>     *  class C implements Comparable<C
>    
>     */
>    static Class<?> comparableClassFor(Object x) {
>        if (x instanceof Comparable) {
>            Class<?> c;
>            Type[] ts, as;
>            Type t;
>            ParameterizedType p;
>            if ((c = x.getClass()) == String.class) // bypass checks
>                return c;
>            if ((ts = c.getGenericInterfaces()) != null) {
>                for (int i = 0; i < ts.length; ++i) {
>                    if (((t = ts[i]) instanceof ParameterizedType)
>                            && ((p = (ParameterizedType) t).getRawType() == Comparable.class)
>                            && (as = p.getActualTypeArguments()) != null && as.length == 1 && as[0] == c) // type arg is c
>                        return c;
>                }
>            }
>        }
>        return null;
>    }
>
>    /**
>     * 比较 k 和 x，
>     * k是已经实现了Comparable接口的函数
>     * 如果x 实现了 Comparable接口，则进行比较
>     * 否则返回0，
>    */
>    static int compareComparables(Class<?> kc, Object k, Object x) {
>        return (x == null || x.getClass() != kc ? 0 : ((Comparable) k).compareTo(x));
>    }
>
>    static final int tableSizeFor(int cap) {
>        int n = cap - 1;
>        n |= n >>> 1;
>        n |= n >>> 2;
>        n |= n >>> 4;
>        n |= n >>> 8;
>        n |= n >>> 16;
>        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
>    }
>
>    /* ---------------- Fields -------------- */
>
>    /**
>     * 在第一个使用的时候初始化，在需要的时候调整大小，
>     * 长度一直是2的幂
>     */
>    transient Node<K, V>[] table;
>
>    /**
>     * 用于缓存entrySet()输出的结果，
>     * 但是在AbstractMap中是被keySet()和values()使用的
>     * 
>     * 下面这个对象只会被创建一次，用于存储entrySet()函数里面创建的EntrySet对象
>     * 主要的是这个对象的实现大部分都是借助HashMap内部的函数和数据结构，
>     * 所以此对象内部的属性基本上创建一次之后就不会在改变。
>    */
>    transient Set<Map.Entry<K, V>> entrySet;
>
>    /**
>     * key-value 的数量
>     */
>    transient int size;
>
>    /**
>     * HashMap被修改的次数
>     */
>    transient int modCount;
>
>    /**
>     * 下一次调整容量的临界值(capacity * load factor).
>     */
>
>    int threshold;
>
>    /**
>     * HashTable 的加载因子
>     */
>    final float loadFactor;
>
>    /* ---------------- Public operations -------------- */
>    // table 的初始化，不是在对象创建的时候初始化，是在第一个插入key-value
>    // 时进行初始化，将初始化的过程推迟到第一个使用时，可以加快对象的创建和
>    // 节省内存空间
>
>    /**
>     * 自定义初始容量和加载因子并初始化
>     */
>    public HashMap(int initialCapacity, float loadFactor) {
>        if (initialCapacity < 0)
>            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
>        if (initialCapacity > MAXIMUM_CAPACITY)
>            initialCapacity = MAXIMUM_CAPACITY;
>        if (loadFactor <= 0 || Float.isNaN(loadFactor))
>            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
>        this.loadFactor = loadFactor;
>        this.threshold = tableSizeFor(initialCapacity);
>    }
>
>    /**
>     * 使用默认的加载因子和指定初始容量构造一个空的HashMap
>     */
>    public HashMap(int initialCapacity) {
>        this(initialCapacity, DEFAULT_LOAD_FACTOR);
>    }
>
>    /**
>     * 使用默认的加载因子和初始容量构造一个空的HashMap
>     */
>    public HashMap() {
>        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
>    }
>
>    /**
>     * 使用默认的加载因子和初始容量构造一个空的HashMap
>     * 并向集合中添加 m
>     */
>    public HashMap(Map<? extends K, ? extends V> m) {
>        this.loadFactor = DEFAULT_LOAD_FACTOR;
>        putMapEntries(m, false);
>    }
>
>    /**
>     * 实现putAll方法和Map的创建
>     *
>     * @param m the map
>     * @param evict false when initially constructing this map, else
>     * true (relayed to method afterNodeInsertion).
>     */
>    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
>        int s = m.size();
>        if (s > 0) {
>            if (table == null) { // 当table为空时，初始化threshold
>                float ft = ((float) s / loadFactor) + 1.0F;
>                int t = ((ft < (float) MAXIMUM_CAPACITY) ? (int) ft : MAXIMUM_CAPACITY);
>                if (t > threshold)
>                    threshold = tableSizeFor(t);
>            } else if (s > threshold) //重新构建table的大小
>                resize();
>            //添加元素到集合
>            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
>                K key = e.getKey();
>                V value = e.getValue();
>                putVal(hash(key), key, value, false, evict);
>            }
>        }
>    }
>
>    /**
>     * 返回集合中key-value Map的数量，而不是table数组的大小
>     */
>    public int size() {
>        return size;
>    }
>
>    /**
>     * 判断Map是否为空
>     */
>    public boolean isEmpty() {
>        return size == 0;
>    }
>
>    /**
>     * Returns the value to which the specified key is mapped,
>     * or {@code null} if this map contains no mapping for the key.
>     *
>     * <p>More formally, if this map contains a mapping from a key
>     * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
>     * key.equals(k))}, then this method returns {@code v}; otherwise
>     * it returns {@code null}.  (There can be at most one such mapping.)
>     *
>     * <p>A return value of {@code null} does not <i>necessarily</i>
>     * indicate that the map contains no mapping for the key; it's also
>     * possible that the map explicitly maps the key to {@code null}.
>     * The {@link #containsKey containsKey} operation may be used to
>     * distinguish these two cases.
>     *
>     * @see #put(Object, Object)
>     */
>    public V get(Object key) {
>        Node<K, V> e;
>        return (e = getNode(hash(key), key)) == null ? null : e.value;
>    }
>
>    /**
>     * Implements Map.get and related methods
>     *
>     * @param hash hash for key
>     * @param key the key
>     * @return the node, or null if none
>     */
>    final Node<K, V> getNode(int hash, Object key) {
>        Node<K, V>[] tab;
>        Node<K, V> first, e;
>        int n;
>        K k;
>        if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
>            if (first.hash == hash && // always check first node
>                    ((k = first.key) == key || (key != null && key.equals(k))))
>                return first;
>            if ((e = first.next) != null) {
>                if (first instanceof TreeNode)
>                    return ((TreeNode<K, V>) first).getTreeNode(hash, key);
>                do {
>                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
>                        return e;
>                } while ((e = e.next) != null);
>            }
>        }
>        return null;
>    }
>
>    /**
>     * Returns <tt>true</tt> if this map contains a mapping for the
>     * specified key.
>     *
>     * @param   key   The key whose presence in this map is to be tested
>     * @return <tt>true</tt> if this map contains a mapping for the specified
>     * key.
>     */
>    public boolean containsKey(Object key) {
>        return getNode(hash(key), key) != null;
>    }
>
>    /**
>     * Associates the specified value with the specified key in this map.
>     * If the map previously contained a mapping for the key, the old
>     * value is replaced.
>     *
>     * @param key key with which the specified value is to be associated
>     * @param value value to be associated with the specified key
>     * @return the previous value associated with <tt>key</tt>, or
>     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
>     *         (A <tt>null</tt> return can also indicate that the map
>     *         previously associated <tt>null</tt> with <tt>key</tt>.)
>     */
>    public V put(K key, V value) {
>        return putVal(hash(key), key, value, false, true);
>    }
>
>    /**
>     * 实现Map.put方法
>     *
>     * @param hash hash for key
>     * @param key the key
>     * @param value the value to put
>     * @param onlyIfAbsent if true, don't change existing value
>     * @param evict if false, the table is in creation mode.
>     * @return previous value, or null if none
>     */
>    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
>        Node<K, V>[] tab;
>        Node<K, V> p;
>        int n, i;
>
>        //判断table是否为空，并初始化table
>        if ((tab = table) == null || (n = tab.length) == 0)
>            n = (tab = resize()).length;
>
>        // 如果hash值对应的table中的索引位置没有元素，
>        // 说明这个元素是第一个添加到此Hash值对应的table中的索引位置，则直接添加即可
>        if ((p = tab[i = (n - 1) & hash]) == null)
>            tab[i] = newNode(hash, key, value, null);
>        else { //说明table中的此Hash对应的索引已经有元素，
>               // 则要进行判断，是否需要转换成红黑树
>            Node<K, V> e;
>            K k;
>
>            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
>                e = p;
>            else if (p instanceof TreeNode)
>                e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
>            else {
>                for (int binCount = 0;; ++binCount) {
>                    if ((e = p.next) == null) {
>                        p.next = newNode(hash, key, value, null);
>                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
>                            treeifyBin(tab, hash);
>                        break;
>                    }
>                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
>                        break;
>                    p = e;
>                }
>            }
>
>            if (e != null) { // existing mapping for key
>                V oldValue = e.value;
>                if (!onlyIfAbsent || oldValue == null)
>                    e.value = value;
>                afterNodeAccess(e);
>                return oldValue;
>            }
>        }
>        ++modCount;
>        if (++size > threshold)
>            resize();
>        afterNodeInsertion(evict);
>        return null;
>    }
>
>    /**
>     * 初始化或者调整table的大小。
>     * 如果是调整table的大小，则table中索hash值对应的索引位置也需要进行调整，
>     * 也就是整个hashMap都会被调整
>    
>     * @return the table
>     */
>    final Node<K, V>[] resize() {
>        Node<K, V>[] oldTab = table;
>        int oldCap = (oldTab == null) ? 0 : oldTab.length;
>        int oldThr = threshold;
>        int newCap, newThr = 0;
>        // 下面这段代码是改变容量和改变界限值
>        if (oldCap > 0) {
>            if (oldCap >= MAXIMUM_CAPACITY) {
>                threshold = Integer.MAX_VALUE;
>                return oldTab;
>            } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
>                newThr = oldThr << 1; // double threshold
>        } else if (oldThr > 0) // initial capacity was placed in threshold
>            newCap = oldThr;
>        else { // zero initial threshold signifies using defaults
>            newCap = DEFAULT_INITIAL_CAPACITY;
>            newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
>        }
>        if (newThr == 0) {
>            float ft = (float) newCap * loadFactor;
>            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ? (int) ft : Integer.MAX_VALUE);
>        }
>        threshold = newThr;
>        @SuppressWarnings({ "rawtypes", "unchecked" })
>        Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
>        table = newTab;
>        if (oldTab != null) {
>            for (int j = 0; j < oldCap; ++j) {
>                Node<K, V> e;
>                if ((e = oldTab[j]) != null) {
>                    oldTab[j] = null;
>                    if (e.next == null)
>                        newTab[e.hash & (newCap - 1)] = e;
>                    else if (e instanceof TreeNode)
>                        ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
>                    else { // preserve order
>                        Node<K, V> loHead = null, loTail = null;
>                        Node<K, V> hiHead = null, hiTail = null;
>                        Node<K, V> next;
>                        do {
>                            next = e.next;
>                            if ((e.hash & oldCap) == 0) {
>                                if (loTail == null)
>                                    loHead = e;
>                                else
>                                    loTail.next = e;
>                                loTail = e;
>                            } else {
>                                if (hiTail == null)
>                                    hiHead = e;
>                                else
>                                    hiTail.next = e;
>                                hiTail = e;
>                            }
>                        } while ((e = next) != null);
>
>                        if (loTail != null) {
>                            loTail.next = null;
>                            newTab[j] = loHead;
>                        }
>                        if (hiTail != null) {
>                            hiTail.next = null;
>                            newTab[j + oldCap] = hiHead;
>                        }
>                    }
>                }
>            }
>        }
>        return newTab;
>    }
>
>    /**
>     * 判断是否把hash对应下的索引转换成红黑树
>     */
>    final void treeifyBin(Node<K, V>[] tab, int hash) {
>        int n, index;
>        Node<K, V> e;
>        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
>            resize();
>        else if ((e = tab[index = (n - 1) & hash]) != null) {
>            TreeNode<K, V> hd = null, tl = null;
>            do {
>                TreeNode<K, V> p = replacementTreeNode(e, null);
>                if (tl == null)
>                    hd = p;
>                else {
>                    p.prev = tl;
>                    tl.next = p;
>                }
>                tl = p;
>            } while ((e = e.next) != null);
>
>            if ((tab[index] = hd) != null)
>                hd.treeify(tab);
>        }
>    }
>
>    /**
>     * 将 m 存储到此HashMap中
>     */
>    public void putAll(Map<? extends K, ? extends V> m) {
>        putMapEntries(m, true);
>    }
>
>    /**
>     * 删除指定key，如果有则返回删除key下面的value值，没有返回null
>     */
>    public V remove(Object key) {
>        Node<K, V> e;
>        return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
>    }
>
>    /**
>     * 实现Map.removeNode 删除指定节点
>     *
>     * @param hash hash for key
>     * @param key the key
>     * @param value the value to match if matchValue, else ignored
>     * @param matchValue if true only remove if value is equal
>     * @param movable if false do not move other nodes while removing
>     * @return the node, or null if none
>     */
>    final Node<K, V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
>        Node<K, V>[] tab;
>        Node<K, V> p;
>        int n, index;
>        if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) {
>            Node<K, V> node = null, e;
>            K k;
>            V v;
>            // 查找节点
>            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
>                node = p;
>            else if ((e = p.next) != null) {
>                if (p instanceof TreeNode)
>                    node = ((TreeNode<K, V>) p).getTreeNode(hash, key);
>                else {
>                    do {
>                        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
>                            node = e;
>                            break;
>                        }
>                        p = e;
>                    } while ((e = e.next) != null);
>                }
>            }
>            // 删除节点
>            if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
>                if (node instanceof TreeNode)
>                    ((TreeNode<K, V>) node).removeTreeNode(this, tab, movable);
>                else if (node == p)
>                    tab[index] = node.next;
>                else
>                    p.next = node.next;
>                ++modCount;
>                --size;
>                afterNodeRemoval(node);
>                return node;
>            }
>        }
>        return null;
>    }
>
>    /**
>     * 清空Map集合
>     */
>    public void clear() {
>        Node<K, V>[] tab;
>        modCount++;
>        if ((tab = table) != null && size > 0) {
>            size = 0;
>            for (int i = 0; i < tab.length; ++i)
>                tab[i] = null;
>        }
>    }
>
>    /**
>     *如果Map集合中包含一个或多个和value值相同的值
>     */
>    public boolean containsValue(Object value) {
>        Node<K, V>[] tab;
>        V v;
>        if ((tab = table) != null && size > 0) {
>            for (int i = 0; i < tab.length; ++i) {
>                for (Node<K, V> e = tab[i]; e != null; e = e.next) {
>                    if ((v = e.value) == value || (value != null && value.equals(v)))
>                        return true;
>                }
>            }
>        }
>        return false;
>    }
>
>    /**
>     * 返回整个Map集合的key的set集合，此HashMap集合的任何key的改变
>     * 也会反应的keySet集合上，反之亦然
>     */
>    public Set<K> keySet() {
>        Set<K> ks = keySet;
>        if (ks == null) {
>            ks = new KeySet();
>            keySet = ks;
>        }
>        return ks;
>    }
>
>    //实现简单的keySet类
>    //继承了AbstractSet，实现了一些通用的方法
>    final class KeySet extends AbstractSet<K> {
>        public final int size() {
>            return size;
>        }
>
>        public final void clear() {
>            HashMap.this.clear();
>        }
>
>        public final Iterator<K> iterator() {
>            return new KeyIterator();
>        }
>
>        public final boolean contains(Object o) {
>            return containsKey(o);
>        }
>
>        public final boolean remove(Object key) {
>            return removeNode(hash(key), key, null, false, true) != null;
>        }
>
>        public final Spliterator<K> spliterator() {
>            return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
>        }
>
>        public final void forEach(Consumer<? super K> action) {
>            Node<K, V>[] tab;
>            if (action == null)
>                throw new NullPointerException();
>            if (size > 0 && (tab = table) != null) {
>                int mc = modCount;
>                for (int i = 0; i < tab.length; ++i) {
>                    for (Node<K, V> e = tab[i]; e != null; e = e.next)
>                        action.accept(e.key);
>                }
>                if (modCount != mc)
>                    throw new ConcurrentModificationException();
>            }
>        }
>    }
>
>    /**
>     * 返回此HashMap集合中所有value的Collection集合
>     * 此HashMap集合的任何value的改变，也会改变的value值
>     * 的集合上，反之亦然
>     */
>    public Collection<V> values() {
>        Collection<V> vs = values;
>        if (vs == null) {
>            vs = new Values();
>            values = vs;
>        }
>        return vs;
>    }
>
>    //实现一个简单的内部Collection类，用于缓存当前HashSet的value
>    final class Values extends AbstractCollection<V> {
>        public final int size() {
>            return size;
>        }
>
>        public final void clear() {
>            HashMap.this.clear();
>        }
>
>        public final Iterator<V> iterator() {
>            return new ValueIterator();
>        }
>
>        public final boolean contains(Object o) {
>            return containsValue(o);
>        }
>
>        public final Spliterator<V> spliterator() {
>            return new ValueSpliterator<>(HashMap.this, 0, -1, 0, 0);
>        }
>
>        public final void forEach(Consumer<? super V> action) {
>            Node<K, V>[] tab;
>            if (action == null)
>                throw new NullPointerException();
>            if (size > 0 && (tab = table) != null) {
>                int mc = modCount;
>                for (int i = 0; i < tab.length; ++i) {
>                    for (Node<K, V> e = tab[i]; e != null; e = e.next)
>                        action.accept(e.value);
>                }
>                if (modCount != mc)
>                    throw new ConcurrentModificationException();
>            }
>        }
>    }
>
>    /**
>     * 返回此HashMap的key-value的Entry集合，
>     * 此HashMap集合的任何的改变，也会改变的Entry
>     * 的集合上，反之亦然
>     */
>    public Set<Map.Entry<K, V>> entrySet() {
>        Set<Map.Entry<K, V>> es;
>        return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
>    }
>
>    // 实现一个简单的内部AbstractSet，用于缓存当前HashMap的key-value的Entry集合
>    // 同时实现一些简单的操作
>    final class EntrySet extends AbstractSet<Map.Entry<K, V>> {
>        public final int size() {
>            return size;
>        }
>
>        public final void clear() {
>            HashMap.this.clear();
>        }
>
>        public final Iterator<Map.Entry<K, V>> iterator() {
>            return new EntryIterator();
>        }
>
>        public final boolean contains(Object o) {
>            if (!(o instanceof Map.Entry))
>                return false;
>            Map.Entry<?, ?> e = (Map.Entry<?, ?>) o;
>            Object key = e.getKey();
>            Node<K, V> candidate = getNode(hash(key), key);
>            return candidate != null && candidate.equals(e);
>        }
>
>        public final boolean remove(Object o) {
>            if (o instanceof Map.Entry) {
>                Map.Entry<?, ?> e = (Map.Entry<?, ?>) o;
>                Object key = e.getKey();
>                Object value = e.getValue();
>                return removeNode(hash(key), key, value, true, true) != null;
>            }
>            return false;
>        }
>
>        public final Spliterator<Map.Entry<K, V>> spliterator() {
>            return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
>        }
>
>        public final void forEach(Consumer<? super Map.Entry<K, V>> action) {
>            Node<K, V>[] tab;
>            if (action == null)
>                throw new NullPointerException();
>            if (size > 0 && (tab = table) != null) {
>                int mc = modCount;
>                for (int i = 0; i < tab.length; ++i) {
>                    for (Node<K, V> e = tab[i]; e != null; e = e.next)
>                        action.accept(e);
>                }
>                if (modCount != mc)
>                    throw new ConcurrentModificationException();
>            }
>        }
>    }
>
>    // Overrides of JDK8 Map extension methods
>
>    @Override
>    public V getOrDefault(Object key, V defaultValue) {
>        Node<K, V> e;
>        return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
>    }
>
>    @Override
>    public V putIfAbsent(K key, V value) {
>        return putVal(hash(key), key, value, true, true);
>    }
>
>    @Override
>    public boolean remove(Object key, Object value) {
>        return removeNode(hash(key), key, value, true, true) != null;
>    }
>
>    @Override
>    public boolean replace(K key, V oldValue, V newValue) {
>        Node<K, V> e;
>        V v;
>        if ((e = getNode(hash(key), key)) != null && ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
>            e.value = newValue;
>            afterNodeAccess(e);
>            return true;
>        }
>        return false;
>    }
>
>    @Override
>    public V replace(K key, V value) {
>        Node<K, V> e;
>        if ((e = getNode(hash(key), key)) != null) {
>            V oldValue = e.value;
>            e.value = value;
>            afterNodeAccess(e);
>            return oldValue;
>        }
>        return null;
>    }
>
>    @Override
>    public V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) {
>        if (mappingFunction == null)
>            throw new NullPointerException();
>        int hash = hash(key);
>        Node<K, V>[] tab;
>        Node<K, V> first;
>        int n, i;
>        int binCount = 0;
>        TreeNode<K, V> t = null;
>        Node<K, V> old = null;
>        if (size > threshold || (tab = table) == null || (n = tab.length) == 0)
>            n = (tab = resize()).length;
>        if ((first = tab[i = (n - 1) & hash]) != null) {
>            if (first instanceof TreeNode)
>                old = (t = (TreeNode<K, V>) first).getTreeNode(hash, key);
>            else {
>                Node<K, V> e = first;
>                K k;
>                do {
>                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
>                        old = e;
>                        break;
>                    }
>                    ++binCount;
>                } while ((e = e.next) != null);
>            }
>            V oldValue;
>            if (old != null && (oldValue = old.value) != null) {
>                afterNodeAccess(old);
>                return oldValue;
>            }
>        }
>        V v = mappingFunction.apply(key);
>        if (v == null) {
>            return null;
>        } else if (old != null) {
>            old.value = v;
>            afterNodeAccess(old);
>            return v;
>        } else if (t != null)
>            t.putTreeVal(this, tab, hash, key, v);
>        else {
>            tab[i] = newNode(hash, key, v, first);
>            if (binCount >= TREEIFY_THRESHOLD - 1)
>                treeifyBin(tab, hash);
>        }
>        ++modCount;
>        ++size;
>        afterNodeInsertion(true);
>        return v;
>    }
>
>    public V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
>        if (remappingFunction == null)
>            throw new NullPointerException();
>        Node<K, V> e;
>        V oldValue;
>        int hash = hash(key);
>        if ((e = getNode(hash, key)) != null && (oldValue = e.value) != null) {
>            V v = remappingFunction.apply(key, oldValue);
>            if (v != null) {
>                e.value = v;
>                afterNodeAccess(e);
>                return v;
>            } else
>                removeNode(hash, key, null, false, true);
>        }
>        return null;
>    }
>
>    @Override
>    public V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
>        if (remappingFunction == null)
>            throw new NullPointerException();
>        int hash = hash(key);
>        Node<K, V>[] tab;
>        Node<K, V> first;
>        int n, i;
>        int binCount = 0;
>        TreeNode<K, V> t = null;
>        Node<K, V> old = null;
>        if (size > threshold || (tab = table) == null || (n = tab.length) == 0)
>            n = (tab = resize()).length;
>        if ((first = tab[i = (n - 1) & hash]) != null) {
>            if (first instanceof TreeNode)
>                old = (t = (TreeNode<K, V>) first).getTreeNode(hash, key);
>            else {
>                Node<K, V> e = first;
>                K k;
>                do {
>                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
>                        old = e;
>                        break;
>                    }
>                    ++binCount;
>                } while ((e = e.next) != null);
>            }
>        }
>        V oldValue = (old == null) ? null : old.value;
>        V v = remappingFunction.apply(key, oldValue);
>        if (old != null) {
>            if (v != null) {
>                old.value = v;
>                afterNodeAccess(old);
>            } else
>                removeNode(hash, key, null, false, true);
>        } else if (v != null) {
>            if (t != null)
>                t.putTreeVal(this, tab, hash, key, v);
>            else {
>                tab[i] = newNode(hash, key, v, first);
>                if (binCount >= TREEIFY_THRESHOLD - 1)
>                    treeifyBin(tab, hash);
>            }
>            ++modCount;
>            ++size;
>            afterNodeInsertion(true);
>        }
>        return v;
>    }
>
>    @Override
>    public V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
>        if (value == null)
>            throw new NullPointerException();
>        if (remappingFunction == null)
>            throw new NullPointerException();
>        int hash = hash(key);
>        Node<K, V>[] tab;
>        Node<K, V> first;
>        int n, i;
>        int binCount = 0;
>        TreeNode<K, V> t = null;
>        Node<K, V> old = null;
>        if (size > threshold || (tab = table) == null || (n = tab.length) == 0)
>            n = (tab = resize()).length;
>        if ((first = tab[i = (n - 1) & hash]) != null) {
>            if (first instanceof TreeNode)
>                old = (t = (TreeNode<K, V>) first).getTreeNode(hash, key);
>            else {
>                Node<K, V> e = first;
>                K k;
>                do {
>                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
>                        old = e;
>                        break;
>                    }
>                    ++binCount;
>                } while ((e = e.next) != null);
>            }
>        }
>        if (old != null) {
>            V v;
>            if (old.value != null)
>                v = remappingFunction.apply(old.value, value);
>            else
>                v = value;
>            if (v != null) {
>                old.value = v;
>                afterNodeAccess(old);
>            } else
>                removeNode(hash, key, null, false, true);
>            return v;
>        }
>        if (value != null) {
>            if (t != null)
>                t.putTreeVal(this, tab, hash, key, value);
>            else {
>                tab[i] = newNode(hash, key, value, first);
>                if (binCount >= TREEIFY_THRESHOLD - 1)
>                    treeifyBin(tab, hash);
>            }
>            ++modCount;
>            ++size;
>            afterNodeInsertion(true);
>        }
>        return value;
>    }
>
>    @Override
>    public void forEach(BiConsumer<? super K, ? super V> action) {
>        Node<K, V>[] tab;
>        if (action == null)
>            throw new NullPointerException();
>        if (size > 0 && (tab = table) != null) {
>            int mc = modCount;
>            for (int i = 0; i < tab.length; ++i) {
>                for (Node<K, V> e = tab[i]; e != null; e = e.next)
>                    action.accept(e.key, e.value);
>            }
>            if (modCount != mc)
>                throw new ConcurrentModificationException();
>        }
>    }
>
>    @Override
>    public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
>        Node<K, V>[] tab;
>        if (function == null)
>            throw new NullPointerException();
>        if (size > 0 && (tab = table) != null) {
>            int mc = modCount;
>            for (int i = 0; i < tab.length; ++i) {
>                for (Node<K, V> e = tab[i]; e != null; e = e.next) {
>                    e.value = function.apply(e.key, e.value);
>                }
>            }
>            if (modCount != mc)
>                throw new ConcurrentModificationException();
>        }
>    }
>
>    /* ------------------------------------------------------------ */
>    // Cloning and serialization
>
>    /**
>     * Returns a shallow copy of this <tt>HashMap</tt> instance: the keys and
>     * values themselves are not cloned.
>     *
>     * @return a shallow copy of this map
>     */
>    @SuppressWarnings("unchecked")
>    @Override
>    public Object clone() {
>        HashMap<K, V> result;
>        try {
>            result = (HashMap<K, V>) super.clone();
>        } catch (CloneNotSupportedException e) {
>            // this shouldn't happen, since we are Cloneable
>            throw new InternalError(e);
>        }
>        result.reinitialize();
>        result.putMapEntries(this, false);
>        return result;
>    }
>
>    // These methods are also used when serializing HashSets
>    final float loadFactor() {
>        return loadFactor;
>    }
>
>    final int capacity() {
>        return (table != null) ? table.length : (threshold > 0) ? threshold : DEFAULT_INITIAL_CAPACITY;
>    }
>
>    /**
>     * Save the state of the <tt>HashMap</tt> instance to a stream (i.e.,
>     * serialize it).
>     *
>     * @serialData The <i>capacity</i> of the HashMap (the length of the
>     *             bucket array) is emitted (int), followed by the
>     *             <i>size</i> (an int, the number of key-value
>     *             mappings), followed by the key (Object) and value (Object)
>     *             for each key-value mapping.  The key-value mappings are
>     *             emitted in no particular order.
>     */
>    private void writeObject(java.io.ObjectOutputStream s) throws IOException {
>        int buckets = capacity();
>        // Write out the threshold, loadfactor, and any hidden stuff
>        s.defaultWriteObject();
>        s.writeInt(buckets);
>        s.writeInt(size);
>        internalWriteEntries(s);
>    }
>
>    /**
>     * Reconstitute the {@code HashMap} instance from a stream (i.e.,
>     * deserialize it).
>     */
>    private void readObject(java.io.ObjectInputStream s) throws IOException, ClassNotFoundException {
>        // Read in the threshold (ignored), loadfactor, and any hidden stuff
>        s.defaultReadObject();
>        reinitialize();
>        if (loadFactor <= 0 || Float.isNaN(loadFactor))
>            throw new InvalidObjectException("Illegal load factor: " + loadFactor);
>        s.readInt(); // Read and ignore number of buckets
>        int mappings = s.readInt(); // Read number of mappings (size)
>        if (mappings < 0)
>            throw new InvalidObjectException("Illegal mappings count: " + mappings);
>        else if (mappings > 0) { // (if zero, use defaults)
>            // Size the table using given load factor only if within
>            // range of 0.25...4.0
>            float lf = Math.min(Math.max(0.25f, loadFactor), 4.0f);
>            float fc = (float) mappings / lf + 1.0f;
>            int cap = ((fc < DEFAULT_INITIAL_CAPACITY) ? DEFAULT_INITIAL_CAPACITY
>                    : (fc >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : tableSizeFor((int) fc));
>            float ft = (float) cap * lf;
>            threshold = ((cap < MAXIMUM_CAPACITY && ft < MAXIMUM_CAPACITY) ? (int) ft : Integer.MAX_VALUE);
>            @SuppressWarnings({ "rawtypes", "unchecked" })
>            Node<K, V>[] tab = (Node<K, V>[]) new Node[cap];
>            table = tab;
>
>            // Read the keys and values, and put the mappings in the HashMap
>            for (int i = 0; i < mappings; i++) {
>                @SuppressWarnings("unchecked")
>                K key = (K) s.readObject();
>                @SuppressWarnings("unchecked")
>                V value = (V) s.readObject();
>                putVal(hash(key), key, value, false, false);
>            }
>        }
>    }
>
>    /* ------------------------------------------------------------ */
>    // iterators
>
>    //实现Iteractor中大部分方法，方便后面类的书写
>    abstract class HashIterator {
>        Node<K, V> next; // next entry to return
>        Node<K, V> current; // current entry
>        int expectedModCount; // for fast-fail
>        int index; // current slot
>
>        HashIterator() {
>            expectedModCount = modCount;
>            Node<K, V>[] t = table;
>            current = next = null;
>            index = 0;
>            if (t != null && size > 0) { // advance to first entry
>                do {
>                } while (index < t.length && (next = t[index++]) == null);
>            }
>        }
>
>        public final boolean hasNext() {
>            return next != null;
>        }
>
>        final Node<K, V> nextNode() {
>            Node<K, V>[] t;
>            Node<K, V> e = next;
>            if (modCount != expectedModCount)
>                throw new ConcurrentModificationException();
>            if (e == null)
>                throw new NoSuchElementException();
>            if ((next = (current = e).next) == null && (t = table) != null) {
>                do {
>                } while (index < t.length && (next = t[index++]) == null);
>            }
>            return e;
>        }
>
>        public final void remove() {
>            Node<K, V> p = current;
>            if (p == null)
>                throw new IllegalStateException();
>            if (modCount != expectedModCount)
>                throw new ConcurrentModificationException();
>            current = null;
>            K key = p.key;
>            removeNode(hash(key), key, null, false, false);
>            expectedModCount = modCount;
>        }
>    }
>
>    final class KeyIterator extends HashIterator implements Iterator<K> {
>        public final K next() {
>            return nextNode().key;
>        }
>    }
>
>    final class ValueIterator extends HashIterator implements Iterator<V> {
>        public final V next() {
>            return nextNode().value;
>        }
>    }
>
>    final class EntryIterator extends HashIterator implements Iterator<Map.Entry<K, V>> {
>        public final Map.Entry<K, V> next() {
>            return nextNode();
>        }
>    }
>
>    /* ------------------------------------------------------------ */
>    // spliterators
>
>    static class HashMapSpliterator<K, V> {
>        final HashMap<K, V> map;
>        Node<K, V> current; // current node
>        int index; // current index, modified on advance/split
>        int fence; // one past last index
>        int est; // size estimate
>        int expectedModCount; // for comodification checks
>
>        HashMapSpliterator(HashMap<K, V> m, int origin, int fence, int est, int expectedModCount) {
>            this.map = m;
>            this.index = origin;
>            this.fence = fence;
>            this.est = est;
>            this.expectedModCount = expectedModCount;
>        }
>
>        final int getFence() { // initialize fence and size on first use
>            int hi;
>            if ((hi = fence) < 0) {
>                HashMap<K, V> m = map;
>                est = m.size;
>                expectedModCount = m.modCount;
>                Node<K, V>[] tab = m.table;
>                hi = fence = (tab == null) ? 0 : tab.length;
>            }
>            return hi;
>        }
>
>        public final long estimateSize() {
>            getFence(); // force init
>            return (long) est;
>        }
>    }
>
>    static final class KeySpliterator<K, V> extends HashMapSpliterator<K, V> implements Spliterator<K> {
>        KeySpliterator(HashMap<K, V> m, int origin, int fence, int est, int expectedModCount) {
>            super(m, origin, fence, est, expectedModCount);
>        }
>
>        public KeySpliterator<K, V> trySplit() {
>            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
>            return (lo >= mid || current != null) ? null
>                    : new KeySpliterator<>(map, lo, index = mid, est >>>= 1, expectedModCount);
>        }
>
>        public void forEachRemaining(Consumer<? super K> action) {
>            int i, hi, mc;
>            if (action == null)
>                throw new NullPointerException();
>            HashMap<K, V> m = map;
>            Node<K, V>[] tab = m.table;
>            if ((hi = fence) < 0) {
>                mc = expectedModCount = m.modCount;
>                hi = fence = (tab == null) ? 0 : tab.length;
>            } else
>                mc = expectedModCount;
>            if (tab != null && tab.length >= hi && (i = index) >= 0 && (i < (index = hi) || current != null)) {
>                Node<K, V> p = current;
>                current = null;
>                do {
>                    if (p == null)
>                        p = tab[i++];
>                    else {
>                        action.accept(p.key);
>                        p = p.next;
>                    }
>                } while (p != null || i < hi);
>                if (m.modCount != mc)
>                    throw new ConcurrentModificationException();
>            }
>        }
>
>        public boolean tryAdvance(Consumer<? super K> action) {
>            int hi;
>            if (action == null)
>                throw new NullPointerException();
>            Node<K, V>[] tab = map.table;
>            if (tab != null && tab.length >= (hi = getFence()) && index >= 0) {
>                while (current != null || index < hi) {
>                    if (current == null)
>                        current = tab[index++];
>                    else {
>                        K k = current.key;
>                        current = current.next;
>                        action.accept(k);
>                        if (map.modCount != expectedModCount)
>                            throw new ConcurrentModificationException();
>                        return true;
>                    }
>                }
>            }
>            return false;
>        }
>
>        public int characteristics() {
>            return (fence < 0 || est == map.size ? Spliterator.SIZED : 0) | Spliterator.DISTINCT;
>        }
>    }
>
>    static final class ValueSpliterator<K, V> extends HashMapSpliterator<K, V> implements Spliterator<V> {
>        ValueSpliterator(HashMap<K, V> m, int origin, int fence, int est, int expectedModCount) {
>            super(m, origin, fence, est, expectedModCount);
>        }
>
>        public ValueSpliterator<K, V> trySplit() {
>            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
>            return (lo >= mid || current != null) ? null
>                    : new ValueSpliterator<>(map, lo, index = mid, est >>>= 1, expectedModCount);
>        }
>
>        public void forEachRemaining(Consumer<? super V> action) {
>            int i, hi, mc;
>            if (action == null)
>                throw new NullPointerException();
>            HashMap<K, V> m = map;
>            Node<K, V>[] tab = m.table;
>            if ((hi = fence) < 0) {
>                mc = expectedModCount = m.modCount;
>                hi = fence = (tab == null) ? 0 : tab.length;
>            } else
>                mc = expectedModCount;
>            if (tab != null && tab.length >= hi && (i = index) >= 0 && (i < (index = hi) || current != null)) {
>                Node<K, V> p = current;
>                current = null;
>                do {
>                    if (p == null)
>                        p = tab[i++];
>                    else {
>                        action.accept(p.value);
>                        p = p.next;
>                    }
>                } while (p != null || i < hi);
>                if (m.modCount != mc)
>                    throw new ConcurrentModificationException();
>            }
>        }
>
>        public boolean tryAdvance(Consumer<? super V> action) {
>            int hi;
>            if (action == null)
>                throw new NullPointerException();
>            Node<K, V>[] tab = map.table;
>            if (tab != null && tab.length >= (hi = getFence()) && index >= 0) {
>                while (current != null || index < hi) {
>                    if (current == null)
>                        current = tab[index++];
>                    else {
>                        V v = current.value;
>                        current = current.next;
>                        action.accept(v);
>                        if (map.modCount != expectedModCount)
>                            throw new ConcurrentModificationException();
>                        return true;
>                    }
>                }
>            }
>            return false;
>        }
>
>        public int characteristics() {
>            return (fence < 0 || est == map.size ? Spliterator.SIZED : 0);
>        }
>    }
>
>    static final class EntrySpliterator<K, V> extends HashMapSpliterator<K, V> implements Spliterator<Map.Entry<K, V>> {
>        EntrySpliterator(HashMap<K, V> m, int origin, int fence, int est, int expectedModCount) {
>            super(m, origin, fence, est, expectedModCount);
>        }
>
>        public EntrySpliterator<K, V> trySplit() {
>            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
>            return (lo >= mid || current != null) ? null
>                    : new EntrySpliterator<>(map, lo, index = mid, est >>>= 1, expectedModCount);
>        }
>
>        public void forEachRemaining(Consumer<? super Map.Entry<K, V>> action) {
>            int i, hi, mc;
>            if (action == null)
>                throw new NullPointerException();
>            HashMap<K, V> m = map;
>            Node<K, V>[] tab = m.table;
>            if ((hi = fence) < 0) {
>                mc = expectedModCount = m.modCount;
>                hi = fence = (tab == null) ? 0 : tab.length;
>            } else
>                mc = expectedModCount;
>            if (tab != null && tab.length >= hi && (i = index) >= 0 && (i < (index = hi) || current != null)) {
>                Node<K, V> p = current;
>                current = null;
>                do {
>                    if (p == null)
>                        p = tab[i++];
>                    else {
>                        action.accept(p);
>                        p = p.next;
>                    }
>                } while (p != null || i < hi);
>                if (m.modCount != mc)
>                    throw new ConcurrentModificationException();
>            }
>        }
>
>        public boolean tryAdvance(Consumer<? super Map.Entry<K, V>> action) {
>            int hi;
>            if (action == null)
>                throw new NullPointerException();
>            Node<K, V>[] tab = map.table;
>            if (tab != null && tab.length >= (hi = getFence()) && index >= 0) {
>                while (current != null || index < hi) {
>                    if (current == null)
>                        current = tab[index++];
>                    else {
>                        Node<K, V> e = current;
>                        current = current.next;
>                        action.accept(e);
>                        if (map.modCount != expectedModCount)
>                            throw new ConcurrentModificationException();
>                        return true;
>                    }
>                }
>            }
>            return false;
>        }
>
>        public int characteristics() {
>            return (fence < 0 || est == map.size ? Spliterator.SIZED : 0) | Spliterator.DISTINCT;
>        }
>    }
>
>    /* ------------------------------------------------------------ */
>    // LinkedHashMap support
>
>    /*
>     * 
>     * The following package-protected methods are designed to be
>     * overridden by LinkedHashMap, but not by any other subclass.
>     * Nearly all other internal methods are also package-protected
>     * but are declared final, so can be used by LinkedHashMap, view
>     * classes, and HashSet.
>     */
>
>    // Create a regular (non-tree) node
>    Node<K, V> newNode(int hash, K key, V value, Node<K, V> next) {
>        return new Node<>(hash, key, value, next);
>    }
>
>    // For conversion from TreeNodes to plain nodes
>    Node<K, V> replacementNode(Node<K, V> p, Node<K, V> next) {
>        return new Node<>(p.hash, p.key, p.value, next);
>    }
>
>    // 创建一个新的节点
>    TreeNode<K, V> newTreeNode(int hash, K key, V value, Node<K, V> next) {
>        return new TreeNode<>(hash, key, value, next);
>    }
>
>    // For treeifyBin，使用node类型的节点，创建Treenode类型的及诶单
>    TreeNode<K, V> replacementTreeNode(Node<K, V> p, Node<K, V> next) {
>        return new TreeNode<>(p.hash, p.key, p.value, next);
>    }
>
>    /**
>     * 用于反序列化时对属性进行初始化
>     */
>    void reinitialize() {
>        table = null;
>        entrySet = null;
>        keySet = null;
>        values = null;
>        modCount = 0;
>        threshold = 0;
>        size = 0;
>    }
>
>    // 回调函数，用于LinkedHashMap中设置
>    // 键值对的排序，
>    // 其中有俩种排序，一种是按照插入的顺序排序
>    // 一种是按照access的先后顺序，
>    void afterNodeAccess(Node<K, V> p) {
>    }
>
>    void afterNodeInsertion(boolean evict) {
>    }
>
>    void afterNodeRemoval(Node<K, V> p) {
>    }
>
>    // Called only from writeObject, to ensure compatible ordering.
>    void internalWriteEntries(java.io.ObjectOutputStream s) throws IOException {
>        Node<K, V>[] tab;
>        if (size > 0 && (tab = table) != null) {
>            for (int i = 0; i < tab.length; ++i) {
>                for (Node<K, V> e = tab[i]; e != null; e = e.next) {
>                    s.writeObject(e.key);
>                    s.writeObject(e.value);
>                }
>            }
>        }
>    }
>
>    /* ------------------------------------------------------------ */
>    // 树节点
>
>    /**
>     * 树节点定义，继承的是LinkedHashMap.Entry
>     * 整个的继承体系如下
>     *  class HashMap.Node<K,V>  implements Map.Entry<K,V>
>     * 
>     *  class LinkedHashMap.Entry<K,V> extends HashMap.Node<K,V>
>     * 
>     *   class TreeNode<K, V> extends LinkedHashMap.Entry<K, V>
>     * 这样做的好处是，可以在LinkedHashMap里面不用定义重复的类，来实现TreeNode
>     * 的定义，同时又可以在LinkedHashMap里面使用这个类。让LinkedHashMap最大程度
>     * 的简化实现。充分的利用了类的泛化。
>     * 
>     */
>    static final class TreeNode<K, V> extends LinkedHashMap.Entry<K, V> {
>        TreeNode<K, V> parent; // red-black tree links 父节点
>        TreeNode<K, V> left;
>        TreeNode<K, V> right;
>        TreeNode<K, V> prev; // needed to unlink next upon deletion
>        boolean red; // 是否是红色节点
>
>        TreeNode(int hash, K key, V val, Node<K, V> next) {
>            super(hash, key, val, next);
>        }
>
>        /**
>         * 返回树的根节点
>         * 要仔细看下面这个函数，
>         * TreeNode<K, V> r = this, p;
>         * 上面那句话就是定义r为当前对象代表的节点，
>         * p是空的TreeNode对象，
>         * 循环向上遍历，最后找到父节点为空的根节点
>         */
>        final TreeNode<K, V> root() {
>            for (TreeNode<K, V> r = this, p;;) {
>                if ((p = r.parent) == null)
>                    return r;
>                r = p;
>            }
>        }
>
>        /**
>         * 确信给定的root节点是数组指定索引下面的第一个节点
>         */
>        static <K, V> void moveRootToFront(Node<K, V>[] tab, TreeNode<K, V> root) {
>            int n;
>            if (root != null && tab != null && (n = tab.length) > 0) {
>
>                int index = (n - 1) & root.hash; // 找到索引值
>                TreeNode<K, V> first = (TreeNode<K, V>) tab[index]; //获取当前索引下面的第一个节点
>                if (root != first) { // 如果不是第一个节点，则将该节点移动的root
>
>                    Node<K, V> rn; //下面这一部分就是从链表中删除节点root
>                    tab[index] = root;
>                    TreeNode<K, V> rp = root.prev;
>                    if ((rn = root.next) != null)
>                        ((TreeNode<K, V>) rn).prev = rp;
>                    if (rp != null)
>                        rp.next = rn;
>
>                    if (first != null) // 将roor和first链接在一起
>                        first.prev = root;
>                    root.next = first;
>                    root.prev = null;
>                }
>                assert checkInvariants(root); //判断是否是root节点，在真事环境运行时可以不管
>            }
>        }
>
>        /**
>         * 找到从当前节点开始和给定的Hash和key值相同的节点
>         * kc 参数是第一次使用comparableClassFor(key) 时所缓存的
>         */
>        final TreeNode<K, V> find(int h, Object k, Class<?> kc) {
>            TreeNode<K, V> p = this;
>            do {
>                int ph, dir;
>                K pk;
>                TreeNode<K, V> pl = p.left, pr = p.right, q;
>
>                if ((ph = p.hash) > h)
>                    p = pl;
>                else if (ph < h)
>                    p = pr;
>                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
>                    return p;
>                else if (pl == null)
>                    p = pr;
>                else if (pr == null)
>                    p = pl;
>                else if ((kc != null || (kc = comparableClassFor(k)) != null)
>                        && (dir = compareComparables(kc, k, pk)) != 0)
>                    p = (dir < 0) ? pl : pr;
>                else if ((q = pr.find(h, k, kc)) != null)
>                    return q;
>                else
>                    p = pl;
>            } while (p != null);
>            return null;
>        }
>
>        /**
>         * 从根节点开始查找和指定的对象相同的节点
>         */
>        final TreeNode<K, V> getTreeNode(int h, Object k) {
>            return ((parent != null) ? root() : this).find(h, k, null);
>        }
>
>        /**
>         * Tie-breaking utility for ordering insertions when equal
>         * hashCodes and non-comparable. We don't require a total
>         * order, just a consistent insertion rule to maintain
>         * equivalence across rebalancings. Tie-breaking further than
>         * necessary simplifies testing a bit.
>         */
>        static int tieBreakOrder(Object a, Object b) {
>            int d;
>            if (a == null || b == null || (d = a.getClass().getName().compareTo(b.getClass().getName())) == 0)
>                d = (System.identityHashCode(a) <= System.identityHashCode(b) ? -1 : 1);
>            return d;
>        }
>
>        /**
>         * Forms tree of the nodes linked from this node.
>         * @return root of tree
>         */
>        final void treeify(Node<K, V>[] tab) {
>            TreeNode<K, V> root = null;
>            for (TreeNode<K, V> x = this, next; x != null; x = next) {
>                next = (TreeNode<K, V>) x.next;
>                x.left = x.right = null;
>                if (root == null) {
>                    x.parent = null;
>                    x.red = false;
>                    root = x;
>                } else {
>                    K k = x.key;
>                    int h = x.hash;
>                    Class<?> kc = null;
>                    for (TreeNode<K, V> p = root;;) {
>                        int dir, ph;
>                        K pk = p.key;
>                        if ((ph = p.hash) > h)
>                            dir = -1;
>                        else if (ph < h)
>                            dir = 1;
>                        else if ((kc == null && (kc = comparableClassFor(k)) == null)
>                                || (dir = compareComparables(kc, k, pk)) == 0)
>                            dir = tieBreakOrder(k, pk);
>
>                        TreeNode<K, V> xp = p;
>                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
>                            x.parent = xp;
>                            if (dir <= 0)
>                                xp.left = x;
>                            else
>                                xp.right = x;
>                            root = balanceInsertion(root, x);
>                            break;
>                        }
>                    }
>                }
>            }
>            moveRootToFront(tab, root);
>        }
>
>        /**
>         * Returns a list of non-TreeNodes replacing those linked from
>         * this node.
>         */
>        final Node<K, V> untreeify(HashMap<K, V> map) {
>            Node<K, V> hd = null, tl = null;
>
>            for (Node<K, V> q = this; q != null; q = q.next) {
>
>                Node<K, V> p = map.replacementNode(q, null);
>                if (tl == null)
>                    hd = p;
>                else
>                    tl.next = p;
>                tl = p;
>            }
>            return hd;
>        }
>
>        /**
>         * 添加节点，当节点已经存在时，就不会添加
>         */
>        final TreeNode<K, V> putTreeVal(HashMap<K, V> map, Node<K, V>[] tab, int h, K k, V v) {
>            Class<?> kc = null;
>            boolean searched = false;
>            TreeNode<K, V> root = (parent != null) ? root() : this; // 找到根节点
>
>            for (TreeNode<K, V> p = root;;) { //从根节点开始遍历，寻找插入的地点
>                int dir, ph;
>                K pk;
>                if ((ph = p.hash) > h)
>                    dir = -1;
>                else if (ph < h)
>                    dir = 1;
>                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
>                    return p;
>                else if ((kc == null && (kc = comparableClassFor(k)) == null)
>                        || (dir = compareComparables(kc, k, pk)) == 0) {
>                    if (!searched) {
>                        TreeNode<K, V> q, ch;
>                        searched = true;
>                        if (((ch = p.left) != null && (q = ch.find(h, k, kc)) != null)
>                                || ((ch = p.right) != null && (q = ch.find(h, k, kc)) != null))
>                            return q;
>                    }
>                    dir = tieBreakOrder(k, pk);
>                }
>
>                TreeNode<K, V> xp = p;
>                if ((p = (dir <= 0) ? p.left : p.right) == null) {
>                    Node<K, V> xpn = xp.next;
>                    TreeNode<K, V> x = map.newTreeNode(h, k, v, xpn);
>                    if (dir <= 0)
>                        xp.left = x;
>                    else
>                        xp.right = x;
>                    xp.next = x;
>                    x.parent = x.prev = xp;
>                    if (xpn != null)
>                        ((TreeNode<K, V>) xpn).prev = x;
>                    moveRootToFront(tab, balanceInsertion(root, x)); // balance返回的还是root节点
>                    return null;
>                }
>            }
>        }
>
>        /**
>         * Removes the given node, that must be present before this call.
>         * This is messier than typical red-black deletion code because we
>         * cannot swap the contents of an interior node with a leaf
>         * successor that is pinned by "next" pointers that are accessible
>         * independently during traversal. So instead we swap the tree
>         * linkages. If the current tree appears to have too few nodes,
>         * the bin is converted back to a plain bin. (The test triggers
>         * somewhere between 2 and 6 nodes, depending on tree structure).
>         */
>        final void removeTreeNode(HashMap<K, V> map, Node<K, V>[] tab, boolean movable) {
>            int n;
>            if (tab == null || (n = tab.length) == 0)
>                return;
>            int index = (n - 1) & hash;
>            TreeNode<K, V> first = (TreeNode<K, V>) tab[index], root = first, rl;
>            TreeNode<K, V> succ = (TreeNode<K, V>) next, pred = prev;
>            if (pred == null)
>                tab[index] = first = succ;
>            else
>                pred.next = succ;
>            if (succ != null)
>                succ.prev = pred;
>            if (first == null)
>                return;
>            if (root.parent != null)
>                root = root.root();
>            if (root == null || root.right == null || (rl = root.left) == null || rl.left == null) {
>                tab[index] = first.untreeify(map); // too small
>                return;
>            }
>            TreeNode<K, V> p = this, pl = left, pr = right, replacement;
>            if (pl != null && pr != null) {
>                TreeNode<K, V> s = pr, sl;
>                while ((sl = s.left) != null) // find successor
>                    s = sl;
>                boolean c = s.red;
>                s.red = p.red;
>                p.red = c; // swap colors
>                TreeNode<K, V> sr = s.right;
>                TreeNode<K, V> pp = p.parent;
>                if (s == pr) { // p was s's direct parent
>                    p.parent = s;
>                    s.right = p;
>                } else {
>                    TreeNode<K, V> sp = s.parent;
>                    if ((p.parent = sp) != null) {
>                        if (s == sp.left)
>                            sp.left = p;
>                        else
>                            sp.right = p;
>                    }
>                    if ((s.right = pr) != null)
>                        pr.parent = s;
>                }
>                p.left = null;
>                if ((p.right = sr) != null)
>                    sr.parent = p;
>                if ((s.left = pl) != null)
>                    pl.parent = s;
>                if ((s.parent = pp) == null)
>                    root = s;
>                else if (p == pp.left)
>                    pp.left = s;
>                else
>                    pp.right = s;
>                if (sr != null)
>                    replacement = sr;
>                else
>                    replacement = p;
>            } else if (pl != null)
>                replacement = pl;
>            else if (pr != null)
>                replacement = pr;
>            else
>                replacement = p;
>            if (replacement != p) {
>                TreeNode<K, V> pp = replacement.parent = p.parent;
>                if (pp == null)
>                    root = replacement;
>                else if (p == pp.left)
>                    pp.left = replacement;
>                else
>                    pp.right = replacement;
>                p.left = p.right = p.parent = null;
>            }
>
>            TreeNode<K, V> r = p.red ? root : balanceDeletion(root, replacement);
>
>            if (replacement == p) { // detach
>                TreeNode<K, V> pp = p.parent;
>                p.parent = null;
>                if (pp != null) {
>                    if (p == pp.left)
>                        pp.left = null;
>                    else if (p == pp.right)
>                        pp.right = null;
>                }
>            }
>            if (movable)
>                moveRootToFront(tab, r);
>        }
>
>        /**
>         * Splits nodes in a tree bin into lower and upper tree bins,
>         * or untreeifies if now too small. Called only from resize;
>         * see above discussion about split bits and indices.
>         *
>         * @param map the map
>         * @param tab the table for recording bin heads
>         * @param index the index of the table being split
>         * @param bit the bit of hash to split on
>         */
>        final void split(HashMap<K, V> map, Node<K, V>[] tab, int index, int bit) {
>            TreeNode<K, V> b = this;
>            // Relink into lo and hi lists, preserving order
>            TreeNode<K, V> loHead = null, loTail = null;
>            TreeNode<K, V> hiHead = null, hiTail = null;
>            int lc = 0, hc = 0;
>            for (TreeNode<K, V> e = b, next; e != null; e = next) {
>                next = (TreeNode<K, V>) e.next;
>                e.next = null;
>                if ((e.hash & bit) == 0) {
>                    if ((e.prev = loTail) == null)
>                        loHead = e;
>                    else
>                        loTail.next = e;
>                    loTail = e;
>                    ++lc;
>                } else {
>                    if ((e.prev = hiTail) == null)
>                        hiHead = e;
>                    else
>                        hiTail.next = e;
>                    hiTail = e;
>                    ++hc;
>                }
>            }
>
>            if (loHead != null) {
>                if (lc <= UNTREEIFY_THRESHOLD)
>                    tab[index] = loHead.untreeify(map);
>                else {
>                    tab[index] = loHead;
>                    if (hiHead != null) // (else is already treeified)
>                        loHead.treeify(tab);
>                }
>            }
>            if (hiHead != null) {
>                if (hc <= UNTREEIFY_THRESHOLD)
>                    tab[index + bit] = hiHead.untreeify(map);
>                else {
>                    tab[index + bit] = hiHead;
>                    if (loHead != null)
>                        hiHead.treeify(tab);
>                }
>            }
>        }
>
>        /* ------------------------------------------------------------ */
>        // 红黑树所需要使用的方法
>
>        //p节点右旋
>        static <K, V> TreeNode<K, V> rotateLeft(TreeNode<K, V> root, TreeNode<K, V> p) {
>            TreeNode<K, V> r, pp, rl;
>            if (p != null && (r = p.right) != null) {
>                if ((rl = p.right = r.left) != null)
>                    rl.parent = p;
>                if ((pp = r.parent = p.parent) == null)
>                    (root = r).red = false;
>                else if (pp.left == p)
>                    pp.left = r;
>                else
>                    pp.right = r;
>                r.left = p;
>                p.parent = r;
>            }
>            return root;
>        }
>
>        //p节点左旋
>        static <K, V> TreeNode<K, V> rotateRight(TreeNode<K, V> root, TreeNode<K, V> p) {
>            TreeNode<K, V> l, pp, lr;
>            if (p != null && (l = p.left) != null) {
>                if ((lr = p.left = l.right) != null)
>                    lr.parent = p;
>                if ((pp = l.parent = p.parent) == null)
>                    (root = l).red = false;
>                else if (pp.right == p)
>                    pp.right = l;
>                else
>                    pp.left = l;
>                l.right = p;
>                p.parent = l;
>            }
>            return root;
>        }
>
>        // 平衡插入的节点
>        static <K, V> TreeNode<K, V> balanceInsertion(TreeNode<K, V> root, TreeNode<K, V> x) {
>            x.red = true;
>            for (TreeNode<K, V> xp, xpp, xppl, xppr;;) {
>                //定义节点 xp x的父节点             xpp x的祖父节点
>                //       xppl x的祖父节点的左节点   xppr祖父节点的右节点
>
>                if ((xp = x.parent) == null) { // 如果x的父节点为空，说明是根节点
>                    x.red = false;
>                    return x;
>                } else if (!xp.red || (xpp = xp.parent) == null) //插入的头三个节点，
>                    return root;
>
>                if (xp == (xppl = xpp.left)) { // 如果x的父节点是左节点
>                    if ((xppr = xpp.right) != null && xppr.red) { // case 1 :x的叔叔节点是红色节点，
>                        xppr.red = false;
>                        xp.red = false;
>                        xpp.red = true;
>                        x = xpp;
>                    } else {
>                        if (x == xp.right) { // case 2 ：x 是右节点，进行左旋变成case 3
>                            root = rotateLeft(root, x = xp);
>                            xpp = (xp = x.parent) == null ? null : xp.parent;
>                        }
>                        if (xp != null) { //case 3：x是左节点 此时就结束循环
>                            xp.red = false;
>                            if (xpp != null) {
>                                xpp.red = true;
>                                root = rotateRight(root, xpp);
>                            }
>                        }
>                    }
>                } else { // x的父节点是右节点
>                    if (xppl != null && xppl.red) { //case 1 ：x的父节点是右节点，且叔叔节点是红色
>                        xppl.red = false;
>                        xp.red = false;
>                        xpp.red = true;
>                        x = xpp;
>                    } else {
>                        if (x == xp.left) { //case 2 ：x是左节点，右旋编程case3
>                            root = rotateRight(root, x = xp);
>                            xpp = (xp = x.parent) == null ? null : xp.parent;
>                        }
>                        if (xp != null) { //case 4
>                            xp.red = false;
>                            if (xpp != null) {
>                                xpp.red = true;
>                                root = rotateLeft(root, xpp);
>                            }
>                        }
>                    }
>                }
>            }
>        }
>
>        //删除平衡
>        static <K, V> TreeNode<K, V> balanceDeletion(TreeNode<K, V> root, TreeNode<K, V> x) {
>            for (TreeNode<K, V> xp, xpl, xpr;;) {
>                if (x == null || x == root)
>                    return root;
>                else if ((xp = x.parent) == null) {
>                    x.red = false;
>                    return x;
>                } else if (x.red) {
>                    x.red = false;
>                    return root;
>                } else if ((xpl = xp.left) == x) {
>                    if ((xpr = xp.right) != null && xpr.red) {
>                        xpr.red = false;
>                        xp.red = true;
>                        root = rotateLeft(root, xp);
>                        xpr = (xp = x.parent) == null ? null : xp.right;
>                    }
>                    if (xpr == null)
>                        x = xp;
>                    else {
>                        TreeNode<K, V> sl = xpr.left, sr = xpr.right;
>                        if ((sr == null || !sr.red) && (sl == null || !sl.red)) {
>                            xpr.red = true;
>                            x = xp;
>                        } else {
>                            if (sr == null || !sr.red) {
>                                if (sl != null)
>                                    sl.red = false;
>                                xpr.red = true;
>                                root = rotateRight(root, xpr);
>                                xpr = (xp = x.parent) == null ? null : xp.right;
>                            }
>                            if (xpr != null) {
>                                xpr.red = (xp == null) ? false : xp.red;
>                                if ((sr = xpr.right) != null)
>                                    sr.red = false;
>                            }
>                            if (xp != null) {
>                                xp.red = false;
>                                root = rotateLeft(root, xp);
>                            }
>                            x = root;
>                        }
>                    }
>                } else { // symmetric
>                    if (xpl != null && xpl.red) {
>                        xpl.red = false;
>                        xp.red = true;
>                        root = rotateRight(root, xp);
>                        xpl = (xp = x.parent) == null ? null : xp.left;
>                    }
>                    if (xpl == null)
>                        x = xp;
>                    else {
>                        TreeNode<K, V> sl = xpl.left, sr = xpl.right;
>                        if ((sl == null || !sl.red) && (sr == null || !sr.red)) {
>                            xpl.red = true;
>                            x = xp;
>                        } else {
>                            if (sl == null || !sl.red) {
>                                if (sr != null)
>                                    sr.red = false;
>                                xpl.red = true;
>                                root = rotateLeft(root, xpl);
>                                xpl = (xp = x.parent) == null ? null : xp.left;
>                            }
>                            if (xpl != null) {
>                                xpl.red = (xp == null) ? false : xp.red;
>                                if ((sl = xpl.left) != null)
>                                    sl.red = false;
>                            }
>                            if (xp != null) {
>                                xp.red = false;
>                                root = rotateRight(root, xp);
>                            }
>                            x = root;
>                        }
>                    }
>                }
>            }
>        }
>
>        /**
>         * Recursive invariant check
>         */
>        static <K, V> boolean checkInvariants(TreeNode<K, V> t) {
>            TreeNode<K, V> tp = t.parent, tl = t.left, tr = t.right, tb = t.prev, tn = (TreeNode<K, V>) t.next;
>            if (tb != null && tb.next != t)
>                return false;
>            if (tn != null && tn.prev != t)
>                return false;
>            if (tp != null && t != tp.left && t != tp.right)
>                return false;
>            if (tl != null && (tl.parent != t || tl.hash > t.hash))
>                return false;
>            if (tr != null && (tr.parent != t || tr.hash < t.hash))
>                return false;
>            if (t.red && tl != null && tl.red && tr != null && tr.red)
>                return false;
>            if (tl != null && !checkInvariants(tl))
>                return false;
>            if (tr != null && !checkInvariants(tr))
>                return false;
>            return true;
>        }
>    }
>
>}
>
>```
>
>1. 从上面可以看出，HashMap的实现方式是先使用除余法找到key值对应的索引位置，然后将元素添加到数组索引位置对应的链表上。当链表的长度大于一个给定的值时使用红黑树来进行存储，这样就可以在O(lg(n))的时间范围内找到节点。同时当元素过多时，红黑树的深度可能过大，所以为了加快查找速度，就需要调整数组的大小和对所有节点进行从新hash计算调整位置。
>2. ​

### 3. HashMap遍历方式

> **遍历HashMap的键值对**
>
> 第一步：**根据entrySet()获取HashMap的“键值对”的Set集合。**
>
> 第二步：**通过Iterator迭代器遍历“第一步”得到的集合。**
>
> ```java
> // 假设map是HashMap对象
> // map中的key是String类型，value是Integer类型
> Integer integ = null;
> Iterator iter = map.entrySet().iterator();
> while(iter.hasNext()) {
>     Map.Entry entry = (Map.Entry)iter.next();
>     // 获取key
>     key = (String)entry.getKey();
>         // 获取value
>     integ = (Integer)entry.getValue();
> }
> ```
>
> **遍历HashMap的键**
>
> 第一步：**根据keySet()获取HashMap的“键”的Set集合。**
> 第二步：**通过Iterator迭代器遍历“第一步”得到的集合。**
>
> ```
> // 假设map是HashMap对象
> // map中的key是String类型，value是Integer类型
> String key = null;
> Integer integ = null;
> Iterator iter = map.keySet().iterator();
> while (iter.hasNext()) {
>         // 获取key
>     key = (String)iter.next();
>         // 根据key，获取value
>     integ = (Integer)map.get(key);
> }
> ```
>
> **遍历HashMap的值**
>
> 第一步：**根据value()获取HashMap的“值”的集合。**
> 第二步：**通过Iterator迭代器遍历“第一步”得到的集合**
>
> ```
> // 假设map是HashMap对象
> // map中的key是String类型，value是Integer类型
> Integer value = null;
> Collection c = map.values();
> Iterator iter= c.iterator();
> while (iter.hasNext()) {
>     value = (Integer)iter.next();
> }
> ```
>
> 代码如下
>
> ```
> import java.util.Collection;
> import java.util.HashMap;
> import java.util.Iterator;
> import java.util.Map;
> import java.util.function.Consumer;
>
> @FunctionalInterface
> interface testTime {
>     void apply(Map map);
> }
>
> /**************************************
>  *      Author : zhangke
>  *      Date   : 2018/3/13 10:36
>  *      Desc   : 用于测试HashMap 遍历的快慢
>  *
>  *      三种遍历方式
>  *      1. 通过entrySet()遍历key、value,参考实现函数
>  *          iteratorHashMapByEntrySet
>  *      2. 通过keySet()去遍历key，value，参考实现函数：
>  *          IteratorHashMapByKeySet
>  *      3. 通过values()去遍历value，参考实现函数：
>  *          iteratorHashMapByValues
>  ***************************************/
> public class HashMapIteratorTest {
>     public static void main(String[] args) {
>         int val = 0;
>
>         HashMap<String, Integer> map = new HashMap();
>
>         for (int i = 0; i < 1000000; i++) {
>             // 随机获取一个[0,100)之间的数字
>
>             // 添加到HashMap中
>             map.put(Integer.toString(i), i);
>             //System.out.println(" key:" + key + " value:" + value);
>         }
>         test((hashmap) -> {
>             iteratorHashMapByEntrySet(hashmap);
>         }, map);
>         test((hashmap) -> {
>             iteratorHashMapJustValues(hashmap);
>         }, map);
>         test((hashmap) -> {
>             IteratorHashMapByKeySet(hashmap);
>         }, map);
>     }
>
>     private static void test(Consumer<HashMap<String, Integer>> consumerm, HashMap map) {
>         long start = System.currentTimeMillis();
>         consumerm.accept(map);
>         long end = System.currentTimeMillis();
>         System.out.println(end - start);
>     }
>
>     /**
>      * 通过entry set遍历HashMap
>      */
>     private static void iteratorHashMapByEntrySet(HashMap<String, Integer> map) {
>         if (map == null)
>             return;
>         System.out.println("iterator HashMap By EntrySet");
>         String key = null;
>         Integer integ = null;
>         Iterator iterator = map.entrySet().iterator();
>         while (iterator.hasNext()) {
>             Map.Entry<String, Integer> entry = (Map.Entry) iterator.next();
>             key = entry.getKey();
>             integ = entry.getValue();
>             // System.out.println(key + "--- " + integ.intValue());
>         }
>
>     }
>
>     /**
>      * 通过keySet来遍历HashMap
>      */
>     private static void IteratorHashMapByKeySet(HashMap<String, Integer> map) {
>         if (map == null)
>             return;
>         System.out.println("iterator HashMap By keyset");
>         String key = null;
>         Integer integ = null;
>         Iterator<String> iter = map.keySet().iterator();
>         while (iter.hasNext()) {
>             key = iter.next();
>             integ = map.get(key);
>             // System.out.println(key + "--- " + integ.intValue());
>         }
>     }
>
>     /**
>      * 遍历HashMap的values
>      */
>     private static void iteratorHashMapJustValues(HashMap map) {
>         if (map == null)
>             return;
>         System.out.println("iterator by values");
>         Collection c = map.values();
>         Iterator iter = c.iterator();
>         while (iter.hasNext()) {
>             iter.next();
>             //  System.out.println(iter.next());
>         }
>     }
> }
> ```
>
> 其实你运行上面的代码，会发现三种遍历方式的速度是相同的。因为三种方式得到的Iterator的实现方式是一样的。在HashMap源码里面首先定义了HashIterator抽象类，这里面实现了Iterator的所有方法，接着为了得到Key，value，Entry，只需在Iterator.next()后面加上key或者value就可以。所以三种遍历方式的速度大小相同

###  4. HashMap使用示例

>```java
>import java.util.Map;
>import java.util.Random;
>import java.util.Iterator;
>import java.util.HashMap;
>import java.util.HashSet;
>import java.util.Map.Entry;
>import java.util.Collection;
>
>public class HashMapTest {
>
>    public static void main(String[] args) {
>        testHashMapAPIs();
>    }
>    
>    private static void testHashMapAPIs() {
>        // 初始化随机种子
>        Random r = new Random();
>        // 新建HashMap
>        HashMap map = new HashMap();
>        // 添加操作
>        map.put("one", r.nextInt(10));
>        map.put("two", r.nextInt(10));
>        map.put("three", r.nextInt(10));
>
>        // 打印出map
>        System.out.println("map:"+map );
>
>        // 通过Iterator遍历key-value
>        Iterator iter = map.entrySet().iterator();
>        while(iter.hasNext()) {
>            Map.Entry entry = (Map.Entry)iter.next();
>            System.out.println("next : "+ entry.getKey() +" - "+entry.getValue());
>        }
>
>        // HashMap的键值对个数        
>        System.out.println("size:"+map.size());
>
>        // containsKey(Object key) :是否包含键key
>        System.out.println("contains key two : "+map.containsKey("two"));
>        System.out.println("contains key five : "+map.containsKey("five"));
>
>        // containsValue(Object value) :是否包含值value
>        System.out.println("contains value 0 : "+
>                 map.containsValue(new Integer(0)));
>
>        // remove(Object key) ： 删除键key对应的键值对
>        map.remove("three");
>
>        System.out.println("map:"+map );
>
>        // clear() ： 清空HashMap
>        map.clear();
>
>        // isEmpty() : HashMap是否为空
>        System.out.println((map.isEmpty()?"map is empty":"map is not empty") );
>    }
>}
>```
>
>











 