### 1.HashMap类的定义

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
```

如上述代码所示, HashMap继承了AbstractMap类, 实现了Map, Cloneable, [Serializable](https://so.csdn.net/so/search?q=Serializable&spm=1001.2101.3001.7020)接口.

### 2. HashMap中定义的常量

```java
//默认容量大小为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 

//最大容量为2^30
static final int MAXIMUM_CAPACITY = 1 << 30;

//默认装载因子为0.75, 0.75在时间与空间开销之间提供了很好的平衡;
//如果值太高, 虽说会提高空间利用率, 但是加大查找的开销
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//主要用于resize()扩容过程中, 当对原来的红黑树根据hash值拆分成两条链表后,
//如果拆分后的链表长度 <=UNTREEIFY_THRESHOLD, 那么就采用链表形式管理hash值冲突;
//否则, 采用红黑树管理hash值冲突.
static final int UNTREEIFY_THRESHOLD = 6;
```

### 3. HashMap中定义的变量

```java
//节点表
transient Node<K,V>[] table;

transient Set<Map.Entry<K,V>> entrySet;

//映射对的数量
transient int size;

//修改的次数, 主要用于迭代的快速失败
transient int modCount;

//reHash的临界值, ( =capacity * loadFactor)
int threshold;
final float loadFactor;
```

### 4. 构造方法

```java
//构造器参数:
//initialCapacity: 初始化容量
//loadFactor: 加载因子
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    //调用tableSizeFor方法, 该方法的作用是将输入的initialCapacity修改为相近的2的幂次方数, 因为HashMap的容量必须为2的幂次方
    this.threshold = tableSizeFor(initialCapacity);
}

public HashMap(int initialCapacity) {
    //采用默认的装载因子
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap() {
    //所有参数采用默认参数
    this.loadFactor = DEFAULT_LOAD_FACTOR; 
}
```

构造[hash](https://so.csdn.net/so/search?q=hash&spm=1001.2101.3001.7020)数组时, 必须保证数组的长度为2的幂次方, 如果传入的初始化长度不是2的幂次方, HashMap类内部会调用tableSizeFor函数自动将传入的初始化长度修剪成相近的2的幂次方数.

```
//将用户输入的hashMap容量cap进行修剪, 返回容量2^n >= cap
    static final int tableSizeFor(int cap) {
      //先减去1,然后将最高位1不断右移进行或操作, 最终得到最高位之后全都是1, 最后再加上1, 便得到新容量2^n.
        int n = cap - 1; 
        n |= n >>> 1;  
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

为什么要设置hash数组长度必须为2的幂次方呢? 考虑如下情况:
当得到key对应的hash值时, hash &(cap-1) 得到该hash值对应的位置.
现在假设hash数组长度为15, 16, 现在有两个hash值分别为8, 9, 计算结果如下:
hash & ( table.length -1 )　　　hash　　&　table.length-1　　位置
　　8 & (15-1)　　　　　　　1000　　　　　1110　　　　　1000
　　9 & (15-1)　　　　　　　1001　　　　　1110　　　　　1000

------

　　8 & (16-1)　　　　　　　1000　　　　　1111　　　　　1000
　　9 & (16-1)　　　　　　　1001　　　　　1111　　　　　1001

------

由上述结果可见, 当hash数组长度为15时, hash值8和9发生了冲突, 均存放到了相同的数组位置8, 那么就需要将hash值为8, 9采用链表管理起来. 而hash数组长度为16时, hash值为9存放到9号位置, hash值为8存放到8号位置, 并没有产生冲突. 因此, 在需要查找hash值为8时, 长度为15的hash数组需要遍历链表, 比长度为16的数组查找效率要低下.
通过进一步的观察, 我们可以发现长度为15的hash数组 length-1 后, 最低位为0, 也就以为 hash & (table.length-1) 的低位永远为0, 也就是说序号最低位为1的位置会被浪费, 0001, 0011, 0101, 0111, 1001, 1011, 1101这些最低位为1不会被使用.
而当hash数组的长度为 2^n 时, table.length -1 后, 最低的 n-1 位全部为1, 说明hash值存放的位置由hash值的低 n-1 位决定. 而HashMap计算hash值时, 同时兼顾了hash值的高位与低位. HashMap计算hash值的源代码如下:

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  //兼顾高低位, 优化hash值位置冲突
    }
```

### 5. 键值对

HashMap采用内部静态类Node来保存键值对, Node类定义如下:

```java
   static class Node<K,V> implements Map.Entry<K,V> {
        final int hash; //保存Hash值
        final K key;  //保存键
        V value;   //保存值
        Node<K,V> next;  //指向下一个键值对

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

### 6. 扩容

HashMap内部提供了reszie函数, 该函数主要用于初始化生成table数组; 或者是将原来的table数组进行扩容, 扩展为原来数组大小的两倍.
在扩容两倍之后, 原来数组中存放的Node要存放到新的数组中, 原来数组中Node的位置在新数组中可以保持不变, 或者一致加上大小为原数组长度的偏移量.
resize函数源代码如下:

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;  //oldTab变量指向原来的数组
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;  //oldThr变量保存原来数组的临界值
        int newCap, newThr = 0;
        if (oldCap > 0) {   //说明将要进行扩容操作
            if (oldCap >= MAXIMUM_CAPACITY) {  //由于最大容量不能超过 MAXMUM_CAPACITY, 当原来数组的容量达到这个值后不能再进行扩容
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && 
                     oldCap >= DEFAULT_INITIAL_CAPACITY)  // 进行两倍扩容
                newThr = oldThr << 1; 
        }
        else if (oldThr > 0) // oldCap=0, 说明原来的table数组为null 
            newCap = oldThr;   // 新创建的容器容量为原来容器中设定的临界值
        else {      //oldCap=0, oldThr=0,所以一切参数采用默认值
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor; //新容器的临界值
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];  //创建新容量的数组
        table = newTab; 
        if (oldTab != null) {  //如果原来的数组中存在值, 需要将原来数组中的值保存到新数组中
            for (int j = 0; j < oldCap; ++j) {  //遍历原来的数组
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {  //如果原来数组位置中的值不为null, 则需要进行转移
                    oldTab[j] = null;   //置为null, 方便进行GC
                    if (e.next == null)   //说明原来数组中保存的hash值是没有冲突的, 也就是Node类型变量
                        newTab[e.hash & (newCap - 1)] = e;  //将e的hash值和(newCap-1)进行与操作, 从而获取在新数组中的位置
                    else if (e instanceof TreeNode)  // 说明原来数组中保存的hash值存在冲突, 是红黑树 TreeNode 类型变量, 采用红黑树管理冲突的键值对
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // 这说明原来数组中保存的hash值存在冲突, 但是并没有采用红黑树对冲突的Hash值进行管理, 而是采用Node链表进行管理
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                             //因为需要根据冲突链表中的hash值存放到新数组中,而新数组的长度是原数组长度的2倍, newTable.length-1 比 oldTable.length-1 多oldCap, 因此 hash&(newTable.length-1) 等价于 hash&(oldTable.length-1) + (hash&oldCap ==0 ? 0 : oldCap)
                            if ((e.hash & oldCap) == 0) {  
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);  //将链表复制到新数组中
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;  //返回新数组的引用
    }
```

### 7. put放置方法

```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

如上述代码所示, HashMap方法调用内部的putVal方法进行插入键值对, putVal方法如下:

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;  //当数组table为null时, 调用resize生成数组table, 并令tab指向数组table
        if ((p = tab[i = (n - 1) & hash]) == null)  //如果新存放的hash值没有冲突
            tab[i] = newNode(hash, key, value, null);  //则只需要生成新的Node节点并存放到table数组中即可
        else {  //否则就是产生了hash冲突
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k)))) 
                e = p;  //如果hash值相等且key值相等, 则令e指向冲突的头节点
            else if (p instanceof TreeNode)  //如果头节点的key值与新插入的key值不等, 并且头结点是TreeNode类型,说明该hash值冲突是采用红黑树进行处理.
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  //向红黑树中插入新的Node节点
            else {  //否则就是采用链表处理hash值冲突
                for (int binCount = 0; ; ++binCount) {  //遍历冲突链表, binCount记录hash值冲突链表中节点个数
                    if ((e = p.next) == null) {  //当遍历到冲突链表的尾部时
                        p.next = newNode(hash, key, value, null);  //生成新节点添加到链表末尾
                        if (binCount >= TREEIFY_THRESHOLD - 1) //如果binCount即冲突节点的个数大于等于 (TREEIFY_THRESHOLD(=8) - 1),便将冲突链表改为红黑树结构, 对冲突进行管理, 否则不需要改为红黑树结构
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))  //如果在冲突链表中找到相同key值的节点, 则直接用新的value覆盖原来的value值即可
                        break;
                    p = e;
                }
            }
            if (e != null) { // 说明原来已经存在相同key的键值对
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)  //onlyIfAbsent为true表示仅当<key,value>不存在时进行插入, 为false表示强制覆盖;
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;  //修改次数自增
        if (++size > threshold) //当键值对数量size达到临界值threhold后, 需要进行扩容操作.
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

### 8. 将冲突链表改为红黑树

```java
//该方法的主要作用是将冲突链表改为红黑树
final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)  //当数组的长度< MIN_TREEIFY_CAPACITY(64) 时,只是单纯将数组扩容, 而没有直接将链表改为红黑树. 因为hash数组长度还太小时导致多冲突的主要原因, 增大hash数组长度可以改善冲突情况
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

### 9. 获取get方法

```java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```

如上述代码所示, get方法调用getNode方法获取对应的value值, getNode方法如下:

```java
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {  //first指向hash值对应数组位置中的Node节点
            if (first.hash == hash && // 如果first节点对应的hash和key的hash相等(在数组相同位置,只是说明 hash&(n-1) 操作结果相等, 说明hash值的部分低位相等, 并不代表整个hash值相等), 并且first对应的key也相等的话, first节点就是要查找的
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {  //说明存在hash冲突
                if (first instanceof TreeNode)  //说明由红黑树对hash值冲突进行管理
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);  //查找红黑树
                do {  //说明hash值冲突是由链表进行管理
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);  //对链表进行遍历
            }
        }
        return null;
    }
```

### 10. remove方法

```java
//删除key对应的键值对
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }
```

由上述代码可见, remove方法调用了内部的removeNode方法, removeNode方法如下:

```java
//参数hash为key的hash值;
//参数key为要删除的key键;
//参数value为key对应的value;
//参数matchValue为true表明只有key在HashMap中对应值为value时才删除; 为false表示强制删除;
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {   //在table中查找对应hash值
            Node<K,V> node = null, e; K k; V v;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {   //说明hash值存在冲突
                if (p instanceof TreeNode)   //hash值冲突由红黑树进行管理
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);   //查找红黑树并返回该节点
                else {   //hash值冲突由链表管理
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);  //从红黑树中删除该节点
                else if (node == p)
                    tab[index] = node.next;   //直接修改
                else
                    p.next = node.next;   //修改冲突链表
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```