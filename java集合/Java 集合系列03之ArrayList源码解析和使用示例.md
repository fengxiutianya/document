### 概要

>本篇博客主要的内容是介绍ArrayList的使用和对其源码进行分析，并比ArrayList不同迭代器之间的性能
>
>内容包括：
>
>1. ArrayList 简介
>2. ArrayList数据结构
>3. ArrayList源码分析
>4. ArrayList遍历方式分析
>5. toArray 异常
>6. ArrayList 基本使用示例

### 1. ArrayList 简介

>ArrayList 是一个数组队列，可以动态的改变大小，相当于动态数组。与Java中的数组相比，它的容量能动态增长。他继承于AbstractList，实现了List，RandomAccess，Cloneable，java.io.Serializable接口。
>
>ArrayList 继承了AbstractList，实现了List。因此就具有相关的添加、删除、修改、遍历等功能。
>
>ArrayList 实现了RandmoAccess接口，即提供了随机访问功能。RandmoAccess是java中用来被List实现，为List提供快速访问功能的。在ArrayList中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。稍后，我们会比较List的“快速随机访问”和“通过Iterator迭代器访问”的效率。
>
>ArrayList 实现了Cloneable接口，即覆盖了函数clone()，能被克隆。
>
>ArrayList 实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输。
>
>**注意的一点是：ArrayList不是线程安全的！所以不要再并发程序中使用**

### 2. ArrayList 数据结构

>Arraylist的继承关系：
>
>```java
>java.lang.Object
>   ↳     java.util.AbstractCollection<E>
>         ↳     java.util.AbstractList<E>
>               ↳     java.util.ArrayList<E>
>
>public class ArrayList<E> extends AbstractList<E>
>        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
>```
>
>Arraylist的类图如下
>
>![](https://images0.cnblogs.com/blog/497634/201401/272343457973489.jpg)
>
>ArrayList包含了两个重要的对象：elementData 和 size。
>
>1. elementData 是"Object[]类型的数组"，它保存了添加到ArrayList中的元素。实际上，elementData是个动态数组，我们能通过构造函数 ArrayList(int initialCapacity)来执行它的初始容量为initialCapacity；如果通过不含参数的构造函数ArrayList()来创建ArrayList，则elementData的容量默认是10。elementData数组的大小会根据ArrayList容量的增长而动态的增长，具体的增长方式，请参考源码分析中的ensureCapacity()函数。
>
>2. size 则是动态数组的实际大小。
>
>   ​

### 3. ArrayList源码分析

>下面是我基于java 8对于ArrayList的分析
>
>```java
>package java.util;
>
>import java.util.function.Consumer;
>import java.util.function.Predicate;
>import java.util.function.UnaryOperator;
>
>public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
>
>    private static final long serialVersionUID = 8683452581122892189L;
>
>    //默认初始容量
>    private static final int DEFAULT_CAPACITY = 10;
>
>    //如果是空实例，这个就会减少创建的开销，
>    private static final Object[] EMPTY_ELEMENTDATA = {};
>
>    //使用new ArrayList()创建时，elementData会指向下面这个数组
>    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
>
>    //当开始添加任何元素时，此属性就会指向EMPTY_ELEMENTDATA
>       // non-private to simplify nested class access
>    transient Object[] elementData; 
>
>
>    //集合包含元素的数量
>    private int size;
>
>    //设置ArrayList的初始容量，当时负数时抛出错误
>    public ArrayList(int initialCapacity) {
>        if (initialCapacity > 0) {
>            this.elementData = new Object[initialCapacity];
>        } else if (initialCapacity == 0) {
>            this.elementData = EMPTY_ELEMENTDATA;
>        } else {
>            throw
>               new IllegalArgumentException("Illegal Capacity: " + 
>                                            initialCapacity);
>        }
>    }
>
>    //初始化ArrayList
>    public ArrayList() {
>        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
>    }
>
>    //添加指定的集合到此集合中并初始化
>    public ArrayList(Collection<? extends E> c) {
>        elementData = c.toArray();
>        if ((size = elementData.length) != 0) {
>            //c.toArray 返回的数组类型有可能和此集合的类型不相同，
>            //则重新拷贝元素到本集合的数组中去
>            if (elementData.getClass() != Object[].class)
>                elementData = Arrays.copyOf(elementData, size, 
>                                            Object[].class);
>        } else {
>            // 利用空集合来初始化
>            this.elementData = EMPTY_ELEMENTDATA;
>        }
>    }
>
>    //缩短集合数组的长度，和现在集合的size相同
>    public void trimToSize() {
>        modCount++;
>        if (size < elementData.length) {
>            elementData = (size == 0) ? EMPTY_ELEMENTDATA :
>                    Arrays.copyOf(elementData, size);
>        }
>    }
>
>    //是集合中数组的长度为当前参数指定的长度
>    public void ensureCapacity(int minCapacity) {
>        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
>                // 如果没有任何元素在此集合中
>                ? 0
>                // 如果集合中有元素，则集合的长度必须大于次默认值，10
>                : DEFAULT_CAPACITY;
>
>        if (minCapacity > minExpand) {
>            ensureExplicitCapacity(minCapacity);
>        }
>    }
>
>    //比较当前指定的长度和默认长度的大小，如果小于则设置当前的长度等于默认长度
>    private void ensureCapacityInternal(int minCapacity) {
>        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
>            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
>        }
>
>        ensureExplicitCapacity(minCapacity);
>    }
>
>    //初始化一个数组为当前指定的长度，并修改modCount 防止出现fail-fast
>    private void ensureExplicitCapacity(int minCapacity) {
>        modCount++;
>
>        // overflow-conscious code
>        if (minCapacity - elementData.length > 0)
>            grow(minCapacity);
>    }
>
>    //数组的最大长度，默认比Integer的长度小8，因为数组有一些信息需要存储在头部
>    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
>
>    // 增加数组的长度，并拷贝数组中的元素到新的数组
>    //下面增加数组，实际上不一定得到的是指定的长度，
>    //有可能比这个长度大
>    private void grow(int minCapacity) {
>        // overflow-conscious code
>        int oldCapacity = elementData.length;
>        int newCapacity = oldCapacity + (oldCapacity >> 1);
>        if (newCapacity - minCapacity < 0)
>            newCapacity = minCapacity;
>        if (newCapacity - MAX_ARRAY_SIZE > 0)
>            newCapacity = hugeCapacity(minCapacity);
>        // minCapacity is usually close to size, so this is a win:
>        elementData = Arrays.copyOf(elementData, newCapacity);
>    }
>
>    //确认最大集合的最大长度
>    private static int hugeCapacity(int minCapacity) {
>        if (minCapacity < 0) // overflow
>            throw new OutOfMemoryError();
>        return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : 
>                                                      MAX_ARRAY_SIZE;
>    }
>
>    // 返回集合的大小
>    public int size() {
>        return size;
>    }
>
>    //通过size来判断集合是否为空
>    public boolean isEmpty() {
>        return size == 0;
>    }
>
>    //判断集合中是否包含和参数相等的元素
>    public boolean contains(Object o) {
>        return indexOf(o) >= 0;
>    }
>
>    // 返回和参数相等的对象位置，
>    // 只要有一个相等，就立即返回在集合总的索引值
>    public int indexOf(Object o) {
>        if (o == null) {
>            for (int i = 0; i < size; i++)
>                if (elementData[i] == null)
>                    return i;
>        } else {
>            for (int i = 0; i < size; i++)
>                if (o.equals(elementData[i]))
>                    return i;
>        }
>        return -1;
>    }
>
>    //返回和参数相等的对象位置，
>    // 是从后往前找，也就是找最后一个，
>    // 只要有一个相等，就立即返回在集合总的索引值
>    public int lastIndexOf(Object o) {
>        if (o == null) {
>            for (int i = size - 1; i >= 0; i--)
>                if (elementData[i] == null)
>                    return i;
>        } else {
>            for (int i = size - 1; i >= 0; i--)
>                if (o.equals(elementData[i]))
>                    return i;
>        }
>        return -1;
>    }
>
>    //返回这个对象的一个浅拷贝
>    public Object clone() {
>        try {
>            ArrayList<?> v = (ArrayList<?>) super.clone();
>            v.elementData = Arrays.copyOf(elementData, size);
>            v.modCount = 0;
>            return v;
>        } catch (CloneNotSupportedException e) {
>            // this shouldn't happen, since we are Cloneable
>            throw new InternalError(e);
>        }
>    }
>
>    // 返回包含集合所有元素的数组
>    public Object[] toArray() {
>        return Arrays.copyOf(elementData, size);
>    }
>
>    // 返回和参数中指定数组类型相同的数组
>    @SuppressWarnings("unchecked")
>    public <T> T[] toArray(T[] a) {
>        if (a.length < size)
>            // Make a new array of a's runtime type, but my contents:
>            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
>        System.arraycopy(elementData, 0, a, 0, size);
>        if (a.length > size)
>            a[size] = null;
>        return a;
>    }
>
>    // 指定位置操作对象
>
>    //得到指定位置的元素
>    @SuppressWarnings("unchecked")
>    E elementData(int index) {
>        return (E) elementData[index];
>    }
>
>    //得到指定位置的元素
>    public E get(int index) {
>        rangeCheck(index);
>
>        return elementData(index);
>    }
>
>    //替换指定位置的元素
>    public E set(int index, E element) {
>        rangeCheck(index);
>
>        E oldValue = elementData(index);
>        elementData[index] = element;
>        return oldValue;
>    }
>
>    //在集合末尾添加元素，并修改modCount
>    public boolean add(E e) {
>        ensureCapacityInternal(size + 1); // Increments modCount!!
>        elementData[size++] = e;
>        return true;
>    }
>
>    //在指定索引位置添加元素
>    public void add(int index, E element) {
>        rangeCheckForAdd(index);
>
>        ensureCapacityInternal(size + 1); // Increments modCount!!
>        System.arraycopy(elementData, index, elementData, index + 1, size - index);
>        elementData[index] = element;
>        size++;
>    }
>
>    // 删除指定索引位置的元素并返回元素
>    public E remove(int index) {
>        rangeCheck(index);
>
>        modCount++;
>        E oldValue = elementData(index);
>
>        int numMoved = size - index - 1;
>        if (numMoved > 0)
>            System.arraycopy(elementData, index + 1, 
>                             elementData, index, numMoved);
>        elementData[--size] = null; // 方便GC处理
>
>        return oldValue;
>    }
>
>    // 移除所有和参数相等的元素
>    public boolean remove(Object o) {
>        if (o == null) {
>            for (int index = 0; index < size; index++)
>                if (elementData[index] == null) {
>                    fastRemove(index);
>                    return true;
>                }
>        } else {
>            for (int index = 0; index < size; index++)
>                if (o.equals(elementData[index])) {
>                    fastRemove(index);
>                    return true;
>                }
>        }
>        return false;
>    }
>
>    //使用native方法快速
>    private void fastRemove(int index) {
>        modCount++;
>        int numMoved = size - index - 1;
>        if (numMoved > 0)
>            System.arraycopy(elementData, index + 1,
>                             elementData, index, numMoved);
>        elementData[--size] = null; // 方便GC处理
>    }
>
>    //清空所有的元素
>    public void clear() {
>        modCount++;
>
>        // 方便GC处理
>        for (int i = 0; i < size; i++)
>            elementData[i] = null;
>
>        size = 0;
>    }
>
>    //添加集合c到此集合中
>    public boolean addAll(Collection<? extends E> c) {
>        Object[] a = c.toArray();
>        int numNew = a.length;
>        ensureCapacityInternal(size + numNew); // Increments modCount
>        System.arraycopy(a, 0, elementData, size, numNew);
>        size += numNew;
>        return numNew != 0;
>    }
>
>    //在index索引之后插入集合c
>    public boolean addAll(int index, Collection<? extends E> c) {
>        rangeCheckForAdd(index);
>
>        Object[] a = c.toArray();
>        int numNew = a.length;
>        ensureCapacityInternal(size + numNew); // Increments modCount
>
>        int numMoved = size - index;
>        if (numMoved > 0)
>            System.arraycopy(elementData, index, 
>                             elementData, index + numNew, numMoved);
>
>        System.arraycopy(a, 0, elementData, index, numNew);
>        size += numNew;
>        return numNew != 0;
>    }
>
>    // 删除在所以呢formIndex和toIndex之间的元素
>    protected void removeRange(int fromIndex, int toIndex) {
>        modCount++;
>        int numMoved = size - toIndex;
>        System.arraycopy(elementData, toIndex, 
>                         elementData, fromIndex, numMoved);
>
>        // 方便GC处理
>        int newSize = size - (toIndex - fromIndex);
>        for (int i = newSize; i < size; i++) {
>            elementData[i] = null;
>        }
>        size = newSize;
>    }
>
>    //检查索引是否有效
>    private void rangeCheck(int index) {
>        if (index >= size)
>            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
>    }
>
>    //检查索引添加位置是否有效
>    private void rangeCheckForAdd(int index) {
>        if (index > size || index < 0)
>            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
>    }
>
>    //构造错误消息模板
>    private String outOfBoundsMsg(int index) {
>        return "Index: " + index + ", Size: " + size;
>    }
>
>    //移除所有和集合c相等的元素
>    public boolean removeAll(Collection<?> c) {
>        Objects.requireNonNull(c);
>        return batchRemove(c, false);
>    }
>
>    //保留所有和集合c相等的元素
>    public boolean retainAll(Collection<?> c) {
>        Objects.requireNonNull(c);
>        return batchRemove(c, true);
>    }
>
>    //true 保留和集合c相等的元素
>    //false 删除和集合c相等的元素
>    private boolean batchRemove(Collection<?> c, boolean complement) {
>        final Object[] elementData = this.elementData;
>        int r = 0, w = 0;
>        boolean modified = false;
>        try {
>            for (; r < size; r++)
>                if (c.contains(elementData[r]) == complement)
>                    elementData[w++] = elementData[r];
>        } finally {
>            //即使上面出现错误，也拷贝完整的数组，保持集合原子性
>            if (r != size) {
>                System.arraycopy(elementData, r, elementData, w, size - r);
>                w += size - r;
>            }
>            if (w != size) {
>                // 方便GC处理
>                for (int i = w; i < size; i++)
>                    elementData[i] = null;
>                modCount += size - w;
>                size = w;
>                modified = true;
>            }
>        }
>        return modified;
>    }
>
>    //序列化对象
>    private void writeObject(java.io.ObjectOutputStream s) 
>             throws java.io.IOException {
>        // Write out element count, and any hidden stuff
>        int expectedModCount = modCount;
>        s.defaultWriteObject();
>
>        // Write out size as capacity for behavioural 
>            //compatibility with clone()
>        s.writeInt(size);
>
>        // Write out all elements in the proper order.
>        for (int i = 0; i < size; i++) {
>            s.writeObject(elementData[i]);
>        }
>
>        if (modCount != expectedModCount) {
>            throw new ConcurrentModificationException();
>        }
>    }
>
>    //反序列化集合
>    private void readObject(java.io.ObjectInputStream s) throws java.io.IOException, ClassNotFoundException {
>        elementData = EMPTY_ELEMENTDATA;
>
>        // Read in size, and any hidden stuff
>        s.defaultReadObject();
>
>        // Read in capacity
>        s.readInt(); // ignored
>
>        if (size > 0) {
>            // be like clone(), allocate array based upon size not capacity
>            ensureCapacityInternal(size);
>
>            Object[] a = elementData;
>            // Read in all elements in the proper order.
>            for (int i = 0; i < size; i++) {
>                a[i] = s.readObject();
>            }
>        }
>    }
>
>    //从指定位置开始返回一个ListIterator
>    public ListIterator<E> listIterator(int index) {
>        if (index < 0 || index > size)
>            throw new IndexOutOfBoundsException("Index: " + index);
>        return new ListItr(index);
>    }
>
>    //返回ListIterator
>    public ListIterator<E> listIterator() {
>        return new ListItr(0);
>    }
>
>    //返回Iterator
>    public Iterator<E> iterator() {
>        return new Itr();
>    }
>
>    //专门为Arraylist实现的一个Iterator
>    private class Itr implements Iterator<E> {
>        int cursor; // index of next element to return
>        int lastRet = -1; // index of last element returned; -1 if no such
>        int expectedModCount = modCount;
>
>        public boolean hasNext() {
>            return cursor != size;
>        }
>
>        @SuppressWarnings("unchecked")
>        public E next() {
>            checkForComodification();
>            int i = cursor;
>            if (i >= size)
>                throw new NoSuchElementException();
>            Object[] elementData = ArrayList.this.elementData;
>            if (i >= elementData.length)
>                throw new ConcurrentModificationException();
>            cursor = i + 1;
>            return (E) elementData[lastRet = i];
>        }
>
>        public void remove() {
>            if (lastRet < 0)
>                throw new IllegalStateException();
>            checkForComodification();
>
>            try {
>                ArrayList.this.remove(lastRet);
>                cursor = lastRet;
>                lastRet = -1;
>                expectedModCount = modCount;
>            } catch (IndexOutOfBoundsException ex) {
>                throw new ConcurrentModificationException();
>            }
>        }
>
>        @Override
>        @SuppressWarnings("unchecked")
>        public void forEachRemaining(Consumer<? super E> consumer) {
>            Objects.requireNonNull(consumer);
>            final int size = ArrayList.this.size;
>            int i = cursor;
>            if (i >= size) {
>                return;
>            }
>            final Object[] elementData = ArrayList.this.elementData;
>            if (i >= elementData.length) {
>                throw new ConcurrentModificationException();
>            }
>            while (i != size && modCount == expectedModCount) {
>                consumer.accept((E) elementData[i++]);
>            }
>            // update once at end of iteration to reduce heap write traffic
>            cursor = i;
>            lastRet = i - 1;
>            checkForComodification();
>        }
>
>        final void checkForComodification() {
>            if (modCount != expectedModCount)
>                throw new ConcurrentModificationException();
>        }
>    }
>
>    //专门为Arraylist优化实现的ListIterator
>    private class ListItr extends Itr implements ListIterator<E> {
>        ListItr(int index) {
>            super();
>            cursor = index;
>        }
>
>        public boolean hasPrevious() {
>            return cursor != 0;
>        }
>
>        public int nextIndex() {
>            return cursor;
>        }
>
>        public int previousIndex() {
>            return cursor - 1;
>        }
>
>        @SuppressWarnings("unchecked")
>        public E previous() {
>            checkForComodification();
>            int i = cursor - 1;
>            if (i < 0)
>                throw new NoSuchElementException();
>            Object[] elementData = ArrayList.this.elementData;
>            if (i >= elementData.length)
>                throw new ConcurrentModificationException();
>            cursor = i;
>            return (E) elementData[lastRet = i];
>        }
>
>        public void set(E e) {
>            if (lastRet < 0)
>                throw new IllegalStateException();
>            checkForComodification();
>
>            try {
>                ArrayList.this.set(lastRet, e);
>            } catch (IndexOutOfBoundsException ex) {
>                throw new ConcurrentModificationException();
>            }
>        }
>
>        public void add(E e) {
>            checkForComodification();
>
>            try {
>                int i = cursor;
>                ArrayList.this.add(i, e);
>                cursor = i + 1;
>                lastRet = -1;
>                expectedModCount = modCount;
>            } catch (IndexOutOfBoundsException ex) {
>                throw new ConcurrentModificationException();
>            }
>        }
>    }
>
>    // 返回formIndex和toIndex索引之间的元素
>    public List<E> subList(int fromIndex, int toIndex) {
>        subListRangeCheck(fromIndex, toIndex, size);
>        return new SubList(this, 0, fromIndex, toIndex);
>    }
>
>    //检测索引位置是否正确
>    static void subListRangeCheck(int fromIndex, int toIndex, int size) {
>        if (fromIndex < 0)
>            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
>        if (toIndex > size)
>            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
>        if (fromIndex > toIndex)
>            throw new IllegalArgumentException("fromIndex(" + fromIndex + ")
>                                               > toIndex(" + toIndex + ")");
>    }
>
>    //一个类似于ArrayList的子ArrayList，方便返回子集和
>    private class SubList extends AbstractList<E> implements RandomAccess {
>        private final AbstractList<E> parent;
>        private final int parentOffset;
>        private final int offset;
>        int size;
>
>        SubList(AbstractList<E> parent, int offset, 
>                int fromIndex, int toIndex) {
>            this.parent = parent;
>            this.parentOffset = fromIndex;
>            this.offset = offset + fromIndex;
>            this.size = toIndex - fromIndex;
>            this.modCount = ArrayList.this.modCount;
>        }
>
>        public E set(int index, E e) {
>            rangeCheck(index);
>            checkForComodification();
>            E oldValue = ArrayList.this.elementData(offset + index);
>            ArrayList.this.elementData[offset + index] = e;
>            return oldValue;
>        }
>
>        public E get(int index) {
>            rangeCheck(index);
>            checkForComodification();
>            return ArrayList.this.elementData(offset + index);
>        }
>
>        public int size() {
>            checkForComodification();
>            return this.size;
>        }
>
>        public void add(int index, E e) {
>            rangeCheckForAdd(index);
>            checkForComodification();
>            parent.add(parentOffset + index, e);
>            this.modCount = parent.modCount;
>            this.size++;
>        }
>
>        public E remove(int index) {
>            rangeCheck(index);
>            checkForComodification();
>            E result = parent.remove(parentOffset + index);
>            this.modCount = parent.modCount;
>            this.size--;
>            return result;
>        }
>
>        protected void removeRange(int fromIndex, int toIndex) {
>            checkForComodification();
>            parent.removeRange(parentOffset + fromIndex, parentOffset + toIndex);
>            this.modCount = parent.modCount;
>            this.size -= toIndex - fromIndex;
>        }
>
>        public boolean addAll(Collection<? extends E> c) {
>            return addAll(this.size, c);
>        }
>
>        public boolean addAll(int index, Collection<? extends E> c) {
>            rangeCheckForAdd(index);
>            int cSize = c.size();
>            if (cSize == 0)
>                return false;
>
>            checkForComodification();
>            parent.addAll(parentOffset + index, c);
>            this.modCount = parent.modCount;
>            this.size += cSize;
>            return true;
>        }
>
>        public Iterator<E> iterator() {
>            return listIterator();
>        }
>
>        public ListIterator<E> listIterator(final int index) {
>            checkForComodification();
>            rangeCheckForAdd(index);
>            final int offset = this.offset;
>
>            return new ListIterator<E>() {
>                int cursor = index;
>                int lastRet = -1;
>                int expectedModCount = ArrayList.this.modCount;
>
>                public boolean hasNext() {
>                    return cursor != SubList.this.size;
>                }
>
>                @SuppressWarnings("unchecked")
>                public E next() {
>                    checkForComodification();
>                    int i = cursor;
>                    if (i >= SubList.this.size)
>                        throw new NoSuchElementException();
>                    Object[] elementData = ArrayList.this.elementData;
>                    if (offset + i >= elementData.length)
>                        throw new ConcurrentModificationException();
>                    cursor = i + 1;
>                    return (E) elementData[offset + (lastRet = i)];
>                }
>
>                public boolean hasPrevious() {
>                    return cursor != 0;
>                }
>
>                @SuppressWarnings("unchecked")
>                public E previous() {
>                    checkForComodification();
>                    int i = cursor - 1;
>                    if (i < 0)
>                        throw new NoSuchElementException();
>                    Object[] elementData = ArrayList.this.elementData;
>                    if (offset + i >= elementData.length)
>                        throw new ConcurrentModificationException();
>                    cursor = i;
>                    return (E) elementData[offset + (lastRet = i)];
>                }
>
>                @SuppressWarnings("unchecked")
>                public void forEachRemaining(Consumer<? super E> consumer) {
>                    Objects.requireNonNull(consumer);
>                    final int size = SubList.this.size;
>                    int i = cursor;
>                    if (i >= size) {
>                        return;
>                    }
>                    final Object[] elementData = ArrayList.this.elementData;
>                    if (offset + i >= elementData.length) {
>                        throw new ConcurrentModificationException();
>                    }
>                    while (i != size && modCount == expectedModCount) {
>                        consumer.accept((E) elementData[offset + (i++)]);
>                    }
>                    // update once at end of iteration to reduce heap write traffic
>                    lastRet = cursor = i;
>                    checkForComodification();
>                }
>
>                public int nextIndex() {
>                    return cursor;
>                }
>
>                public int previousIndex() {
>                    return cursor - 1;
>                }
>
>                public void remove() {
>                    if (lastRet < 0)
>                        throw new IllegalStateException();
>                    checkForComodification();
>
>                    try {
>                        SubList.this.remove(lastRet);
>                        cursor = lastRet;
>                        lastRet = -1;
>                        expectedModCount = ArrayList.this.modCount;
>                    } catch (IndexOutOfBoundsException ex) {
>                        throw new ConcurrentModificationException();
>                    }
>                }
>
>                public void set(E e) {
>                    if (lastRet < 0)
>                        throw new IllegalStateException();
>                    checkForComodification();
>
>                    try {
>                        ArrayList.this.set(offset + lastRet, e);
>                    } catch (IndexOutOfBoundsException ex) {
>                        throw new ConcurrentModificationException();
>                    }
>                }
>
>                public void add(E e) {
>                    checkForComodification();
>
>                    try {
>                        int i = cursor;
>                        SubList.this.add(i, e);
>                        cursor = i + 1;
>                        lastRet = -1;
>                        expectedModCount = ArrayList.this.modCount;
>                    } catch (IndexOutOfBoundsException ex) {
>                        throw new ConcurrentModificationException();
>                    }
>                }
>
>                final void checkForComodification() {
>                    if (expectedModCount != ArrayList.this.modCount)
>                        throw new ConcurrentModificationException();
>                }
>            };
>        }
>
>        public List<E> subList(int fromIndex, int toIndex) {
>            subListRangeCheck(fromIndex, toIndex, size);
>            return new SubList(this, offset, fromIndex, toIndex);
>        }
>
>        private void rangeCheck(int index) {
>            if (index < 0 || index >= this.size)
>                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
>        }
>
>        private void rangeCheckForAdd(int index) {
>            if (index < 0 || index > this.size)
>                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
>        }
>
>        private String outOfBoundsMsg(int index) {
>            return "Index: " + index + ", Size: " + this.size;
>        }
>
>        private void checkForComodification() {
>            if (ArrayList.this.modCount != this.modCount)
>                throw new ConcurrentModificationException();
>        }
>
>        public Spliterator<E> spliterator() {
>            checkForComodification();
>            return new ArrayListSpliterator<E>(ArrayList.this, offset, offset + this.size, this.modCount);
>        }
>    }
>
>    //移除所有和集合c相等的元素
>    //一下都是方便java8流的使用、、 
>    @Override
>    public void forEach(Consumer<? super E> action) {
>        Objects.requireNonNull(action);
>        final int expectedModCount = modCount;
>        @SuppressWarnings("unchecked")
>        final E[] elementData = (E[]) this.elementData;
>        final int size = this.size;
>        for (int i = 0; modCount == expectedModCount && i < size; i++) {
>            action.accept(elementData[i]);
>        }
>        if (modCount != expectedModCount) {
>            throw new ConcurrentModificationException();
>        }
>    }
>
>    @Override
>    public Spliterator<E> spliterator() {
>        return new ArrayListSpliterator<>(this, 0, -1, 0);
>    }
>
>    /** Index-based split-by-two, lazily initialized Spliterator */
>    static final class ArrayListSpliterator<E> implements Spliterator<E> {
>
>        private final ArrayList<E> list;
>        private int index; // current index, modified on advance/split
>        private int fence; // -1 until used; then one past last index
>        private int expectedModCount; // initialized when fence set
>
>        /** Create new spliterator covering the given  range */
>        ArrayListSpliterator(ArrayList<E> list, int origin, int fence, int expectedModCount) {
>            this.list = list; // OK if null unless traversed
>            this.index = origin;
>            this.fence = fence;
>            this.expectedModCount = expectedModCount;
>        }
>
>        private int getFence() { // initialize fence to size on first use
>            int hi; // (a specialized variant appears in method forEach)
>            ArrayList<E> lst;
>            if ((hi = fence) < 0) {
>                if ((lst = list) == null)
>                    hi = fence = 0;
>                else {
>                    expectedModCount = lst.modCount;
>                    hi = fence = lst.size;
>                }
>            }
>            return hi;
>        }
>
>        public ArrayListSpliterator<E> trySplit() {
>            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
>            return (lo >= mid) ? null : // divide range in half unless too small
>                    new ArrayListSpliterator<E>(list, lo, index = mid, expectedModCount);
>        }
>
>        public boolean tryAdvance(Consumer<? super E> action) {
>            if (action == null)
>                throw new NullPointerException();
>            int hi = getFence(), i = index;
>            if (i < hi) {
>                index = i + 1;
>                @SuppressWarnings("unchecked")
>                E e = (E) list.elementData[i];
>                action.accept(e);
>                if (list.modCount != expectedModCount)
>                    throw new ConcurrentModificationException();
>                return true;
>            }
>            return false;
>        }
>
>        public void forEachRemaining(Consumer<? super E> action) {
>            int i, hi, mc; // hoist accesses and checks from loop
>            ArrayList<E> lst;
>            Object[] a;
>            if (action == null)
>                throw new NullPointerException();
>            if ((lst = list) != null && (a = lst.elementData) != null) {
>                if ((hi = fence) < 0) {
>                    mc = lst.modCount;
>                    hi = lst.size;
>                } else
>                    mc = expectedModCount;
>                if ((i = index) >= 0 && (index = hi) <= a.length) {
>                    for (; i < hi; ++i) {
>                        @SuppressWarnings("unchecked")
>                        E e = (E) a[i];
>                        action.accept(e);
>                    }
>                    if (lst.modCount == mc)
>                        return;
>                }
>            }
>            throw new ConcurrentModificationException();
>        }
>
>        public long estimateSize() {
>            return (long) (getFence() - index);
>        }
>
>        public int characteristics() {
>            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
>        }
>    }
>
>    @Override
>    public boolean removeIf(Predicate<? super E> filter) {
>        Objects.requireNonNull(filter);
>        // figure out which elements are to be removed
>        // any exception thrown from the filter predicate at this stage
>        // will leave the collection unmodified
>        int removeCount = 0;
>        final BitSet removeSet = new BitSet(size);
>        final int expectedModCount = modCount;
>        final int size = this.size;
>        for (int i = 0; modCount == expectedModCount && i < size; i++) {
>            @SuppressWarnings("unchecked")
>            final E element = (E) elementData[i];
>            if (filter.test(element)) {
>                removeSet.set(i);
>                removeCount++;
>            }
>        }
>        if (modCount != expectedModCount) {
>            throw new ConcurrentModificationException();
>        }
>
>        // shift surviving elements left over the spaces left by removed elements
>        final boolean anyToRemove = removeCount > 0;
>        if (anyToRemove) {
>            final int newSize = size - removeCount;
>            for (int i = 0, j = 0; (i < size) && (j < newSize); i++, j++) {
>                i = removeSet.nextClearBit(i);
>                elementData[j] = elementData[i];
>            }
>            for (int k = newSize; k < size; k++) {
>                elementData[k] = null; // Let gc do its work
>            }
>            this.size = newSize;
>            if (modCount != expectedModCount) {
>                throw new ConcurrentModificationException();
>            }
>            modCount++;
>        }
>
>        return anyToRemove;
>    }
>
>    @Override
>    @SuppressWarnings("unchecked")
>    public void replaceAll(UnaryOperator<E> operator) {
>        Objects.requireNonNull(operator);
>        final int expectedModCount = modCount;
>        final int size = this.size;
>        for (int i = 0; modCount == expectedModCount && i < size; i++) {
>            elementData[i] = operator.apply((E) elementData[i]);
>        }
>        if (modCount != expectedModCount) {
>            throw new ConcurrentModificationException();
>        }
>        modCount++;
>    }
>
>    @Override
>    @SuppressWarnings("unchecked")
>    public void sort(Comparator<? super E> c) {
>        final int expectedModCount = modCount;
>        Arrays.sort((E[]) elementData, 0, size, c);
>        if (modCount != expectedModCount) {
>            throw new ConcurrentModificationException();
>        }
>        modCount++;
>    }
>}
>
>```
>
>1. ArrayList 实际上是通过一个数组去保存数据的。但我们初始化数组的时候，如果使用空构造函数，则数组大小为10
>2. 当数组容量不足时，Arraylist会重新设置容量。不同版本Jdk自动增加容量的算法不同。

### 4. ArrayList 遍历方式

>(01) 第一种，**通过迭代器遍历**。即通过Iterator去遍历。
>
>```
>Integer value = null;
>Iterator iter = list.iterator();
>while (iter.hasNext()) {
>    value = (Integer)iter.next();
>}
>```
>
>(02) 第二种，**随机访问，通过索引值去遍历。**
>由于ArrayList实现了RandomAccess接口，它支持通过索引值去随机访问元素。
>
>```
>Integer value = null;
>int size = list.size();
>for (int i=0; i<size; i++) {
>    value = (Integer)list.get(i);        
>}
>```
>
>(03) 第三种，**for循环遍历**。如下：
>
>```
>Integer value = null;
>for (Integer integ:list) {
>    value = integ;
>}
>```
>
>下面通过一个实例来演示效率问题
>
>```java
>package Collections.cnblog.collection.list;
>
>import java.util.ArrayList;
>import java.util.Iterator;
>import java.util.List;
>
>/**************************************
> *      Author : zhangke
> *      Date   : 2018/1/18 19:37
> *      Desc   : 测出ArrayList三种哪一种遍历最快
> ***************************************/
>public class StudyArrayList {
>
>    public static void main(String[] args) {
>        List<Integer> list = new ArrayList<>();
>        for (int i = 0; i < 100000; i++) {
>            list.add(i);
>        }
>        iteratorThroughFor(list);
>        iteratorThroughFor2(list);
>        iteratorThroughRandomAccess(list);
>    }
>
>    public static void iteratorThroughRandomAccess(List list) {
>        long startTime;
>        long endTime;
>        startTime = System.currentTimeMillis();
>        for (int i = 0; i < list.size(); i++) {
>            list.get(i);
>        }
>        endTime = System.currentTimeMillis();
>        System.out.println("iteractorRandomAccess interval: " + (endTime - startTime));
>    }
>
>    public static void iteratorThroughFor2(List list) {
>        long startTime;
>        long endTime;
>        startTime = System.currentTimeMillis();
>        for (Object object : list) {
>
>        }
>        endTime = System.currentTimeMillis();
>        System.out.println("iteractorfor interval: " + (endTime - startTime));
>    }
>
>    public static void iteratorThroughFor(List list) {
>        long startTime;
>        long endTime;
>        startTime = System.currentTimeMillis();
>        for (Iterator iterator = list.iterator(); iterator.hasNext(); ) {
>            iterator.next();
>        }
>        endTime = System.currentTimeMillis();
>        System.out.println("iteractor interval: " + (endTime - startTime));
>    }
>}
>
>```
>
>结果如下
>
>```
>iteractor interval: 7
>iteractorfor2 interval: 4
>iteractorRandomAccess interval: 4
>```
>
>我测试了几次，感觉java8里面随机访问和foreach访问比较快，不过相差也不是太大。不过数据量大的话，还是推介使用随机访问。
>
>

### 5. toArray 异常

>```java
>Object[] toArray()
><T> T[] toArray(T[] arr)
>```
>
>调用 toArray() 函数会抛出“java.lang.ClassCastException”异常，但是调用 toArray(T[] contents) 能正常返回 T[]。
>
>toArray() 会抛出异常是因为 toArray() 返回的是 Object[] 数组，将 Object[] 转换为其它类型(类如，将Object[]转换为的Integer[])则会抛出“java.lang.ClassCastException”异常，因为**Java不支持向下转型**。具体的可以参考前面ArrayList.java的源码介绍部分的toArray()。
>解决该问题的办法是调用 <T> T[] toArray(T[] contents) ， 而不是 Object[] toArray()。

### 6. ArrayList基本使用

>```java
>import java.util.*;
>
>public class ArrayListTest {
>
>    public static void main(String[] args) {
>        
>        // 创建ArrayList
>        ArrayList list = new ArrayList();
>
>        // 将“”
>        list.add("1");
>        list.add("2");
>        list.add("3");
>        list.add("4");
>        // 将下面的元素添加到第1个位置
>        list.add(0, "5");
>
>        // 获取第1个元素
>        System.out.println("the first element is: "+ list.get(0));
>        // 删除“3”
>        list.remove("3");
>        // 获取ArrayList的大小
>        System.out.println("Arraylist size=: "+ list.size());
>        // 判断list中是否包含"3"
>        System.out.println("ArrayList contains 3 is: "+ list.contains(3));
>        // 设置第2个元素为10
>        list.set(1, "10");
>
>        // 通过Iterator遍历ArrayList
>        for(Iterator iter = list.iterator(); iter.hasNext(); ) {
>            System.out.println("next is: "+ iter.next());
>        }
>
>        // 将ArrayList转换为数组
>        String[] arr = (String[])list.toArray(new String[0]);
>        for (String str:arr)
>            System.out.println("str: "+ str);
>
>        // 清空ArrayList
>        list.clear();
>        // 判断ArrayList是否为空
>        System.out.println("ArrayList is empty: "+ list.isEmpty());
>    }
>}
>```
>
>







