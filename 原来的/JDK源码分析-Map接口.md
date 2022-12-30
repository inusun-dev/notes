Map接口以键值方式存储，常用的实现类有HashMap，Hashtable，[ConcurrentHashMap](https://so.csdn.net/so/search?q=ConcurrentHashMap&spm=1001.2101.3001.7020)，TreeMap，LinkedHashMap等。

**jdk1.7**的Map接口的方法有如下，到了jdk1.8还有额外的增加。

```java
package java.util;
 
 
public interface Map<K,V> {
 
    //获取集合代下
    int size();
 
    //判断集合是否为空
    boolean isEmpty();
 
    //是否包含key值
    boolean containsKey(Object key);
   
    //是否包含value值
    boolean containsValue(Object value);
 
    //通过键获取值
    V get(Object key);
 
    //增加新元素
    V put(K key, V value);
 
    //通过键移除数据
    V remove(Object key);
   
    //添加一个新的集合
    void putAll(Map<? extends K, ? extends V> m);
 
    //清空集合数据
    void clear();
 
    //返回此map集合【包含的键】的 Set 视图。
    Set<K> keySet();
    
    //返回此map集合的 Collection 视图。
    Collection<V> values();
 
    //返回此map集合的 Set 视图。
    Set<Map.Entry<K, V>> entrySet();
   
    //内置Entry接口，以键，值方式进行存储的，相当于Map集合存储元数据的。
    interface Entry<K,V> {
        //设置key
        K getKey();
 
        //获取value
        V getValue();
 
        //设置value
        V setValue(V value);
 
        //判断是否相等
        boolean equals(Object o);
 
       //获得哈希码
        int hashCode();
    }
 
 
    //判断集合对象是否相等
    boolean equals(Object o);
 
    //获得集合的哈希码
    int hashCode();
 
}
```

到了jdk1.8，增加以下的几个方法：

```java
   // 获取 key 对应的 value，若 value 为 null，则返回 defaultValue 
   default V getOrDefault(Object key, V defaultValue) {
        V v;
        return (((v = get(key)) != null) || containsKey(key))
            ? v
            : defaultValue;
    }
 
   // 遍历 Map 中的元素
   default void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }
 
    // 通过给定的函数计算出新的 Entry 替换所有旧的 Entry
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
 
            // ise thrown from function is not a cme.
            v = function.apply(k, v);
 
            try {
                entry.setValue(v);
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
        }
    }
 
 
   // 若 key 对应的 value 不存在，则把 key-value 存入 Map，否则无操作
    default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }
 
        return v;
    }
 
   // 若 key 对应的值等于 value，则移除 key；否则无操作
    default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }
 
 
  // 若 key 对应的值等于 oldValue，则将其替换为 newValue；否则无操作 
    default boolean replace(K key, V oldValue, V newValue) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }
 
 
    // Map 中存在 key 时，将 key-value 存入，相当于：
   /*
     if (map.containsKey(key)) {
         return map.put(key, value);
     } else
       return null;
     }
   */
    default V replace(K key, V value) {
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }
 
    // 当 key 对应的 value 不存在时，使用给定的函数计算得出 newValue
   // 并将 key-newValue 存入 Map
    default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }
 
        return v;
    }
 
   // 当 key 对应的 value 存在时，使用给定的函数计算得出 newValue，
   // 当 newValue 不为 null 时将 key-newValue 存入 Map；否则移除 key
    default V computeIfPresent(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue;
        if ((oldValue = get(key)) != null) {
            V newValue = remappingFunction.apply(key, oldValue);
            if (newValue != null) {
                put(key, newValue);
                return newValue;
            } else {
                remove(key);
                return null;
            }
        } else {
            return null;
        }
    }
   
 
   // 根据 key 和其对应的 oldValue，使用给定的函数计算出 newValue
    // 若 newValue 为 null
   //   若 oldValue 不为空或 key 存在，则删除 key-oldValue
   //   否则无操作
   // 若 newValue 不为 null，用 newValue 替换 oldValue
    default V compute(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue = get(key);
 
        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue == null) {
            // delete mapping
            if (oldValue != null || containsKey(key)) {
                // something to remove
                remove(key);
                return null;
            } else {
                // nothing to do. Leave things as they were.
                return null;
            }
        } else {
            // add or replace old mapping
            put(key, newValue);
            return newValue;
        }
    }
 
   //当map中不存在指定的key时，便将传入的value设置为key的值，当key存在值时，执行一个方法该方法 
   //接收key的旧值和传入的value，执行自定义的方法返回最终结果设置为key的值。
    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }
```

小结：
jdk1.8对于许多常用的类都扩展了一些面向函数，lambda表达式，方法引用的功能，使得java面向函数编程更为方便。
map接口按照key-value的键值对形式进行存储。
Entry接口是定义在Map接口内，其实Entry真正就是键-值结构的存储，用来存储map的元数据的。
