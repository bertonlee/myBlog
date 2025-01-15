## 1. HashMap 概述
`HashMap` 是 Java 集合框架中最常用的数据结构之一，基于**哈希表（Hash Table）**实现。它以**键值对（Key-Value）**存储数据，允许 `null` 键和 `null` 值，且**无序**。

##### [👉👉👉点击获取2024Java学习汁源](https://pan.quark.cn/s/0a774039f8f9)
### 1.1 HashMap 的特性
- **基于哈希表（Hash Table）实现**
- **允许 `null` 键和 `null` 值**
- **非线程安全**
- **默认初始容量 `16`，负载因子 `0.75`**
- **JDK 1.8 之后采用 **数组 + 链表 + 红黑树** 结构**

```java
Map<String, Integer> map = new HashMap<>();
map.put("Java", 1);
map.put("Python", 2);
System.out.println(map.get("Java")); // 1
```

---

## 2. HashMap 的底层数据结构
### 2.1 JDK 1.7 结构（数组 + 链表）
在 JDK 1.7 及之前，`HashMap` 采用 **数组 + 链表** 实现：
```text
table[0]  →  (key1, value1) → (key2, value2) → (key3, value3) → null
table[1]  →  (key4, value4) → null
table[2]  →  (key5, value5) → null
...
```
### 2.2 JDK 1.8 结构（数组 + 链表 + 红黑树）
JDK 1.8 进行了优化：
- 当链表长度 **超过 8**，转换为 **红黑树** 以提高查询性能（O(n) → O(log n)）。
- 当链表长度 **小于 6**，恢复为 **链表**。

```text
table[0]  →  (key1, value1) → (key2, value2) → 红黑树
table[1]  →  (key3, value3) → null
table[2]  →  (key4, value4) → null
```

---

## 3. HashMap 核心源码解析

### 3.1 `put(K key, V value)` 方法
`put()` 方法用于向 `HashMap` 添加键值对，底层步骤如下：
1. **计算哈希值**：调用 `hash(key)` 方法，计算 `key` 在数组中的索引。
2. **索引定位**：确定该 `key` 对应的数组位置 `table[index]`。
3. **检查是否冲突**：
   - **无冲突**：直接插入（数组无元素）。
   - **有冲突**（已有元素）：使用**链表或红黑树**存储。
4. **扩容判断**：如果元素数量超过 `threshold`（容量 * 负载因子），触发 **resize()** 扩容。

源码分析：
```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```
- `hash(key)` 计算键的哈希值。
- `putVal()` 负责存储键值对。

---

### 3.2 `hash()` 计算哈希值
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
**解释**：
- `key.hashCode()` 获取哈希码。
- 右移 16 位后与自身异或（`^`），减少哈希冲突。

---

### 3.3 `resize()` 扩容机制
当 `HashMap` 的**元素数量超过阈值**（默认 `0.75 * capacity`），会进行**扩容**：
1. **容量翻倍**（`newCapacity = oldCapacity * 2`）。
2. **重新计算索引位置**，将所有元素重新映射到新数组。

源码：
```java
void resize(int newCapacity) {
    Node<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    int newThreshold = (int)(newCapacity * loadFactor);
    table = new Node[newCapacity];
    for (Node<K,V> e : oldTable) {
        if (e != null) {
            int newIndex = e.hash & (newCapacity - 1);
            table[newIndex] = e;
        }
    }
}
```

---

## 4. JDK 1.7 vs JDK 1.8 的区别
| 版本 | 结构 | 主要优化 |
|------|------|---------|
| **JDK 1.7** | 数组 + 链表 | 头插法（导致死循环） |
| **JDK 1.8** | 数组 + 链表 + 红黑树 | 1. 采用 **尾插法** 避免死循环 2. 长链表转换为 **红黑树** |

---

## 5. HashMap 的并发问题
### 5.1 JDK 1.7 的并发问题
- **并发扩容导致死循环**（线程同时 `resize()`）。
- **多线程同时插入数据，导致数据丢失**。

### 5.2 线程安全的替代方案
- **`ConcurrentHashMap`**：推荐使用，线程安全，性能高。
- **`Collections.synchronizedMap(new HashMap<>())`**：使用同步包装类。

示例：
```java
Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());
```

---

## 6. HashMap 面试高频问题
### **Q1：HashMap 为什么使用 `hash & (length - 1)` 而不是 `hash % length`？**
**A**：`&` 运算比 `%` 更高效，`length` 为 2 的幂时，`hash & (length - 1)` 能保证均匀分布。

---

### **Q2：HashMap 什么时候扩容？**
**A**：当元素个数 **超过 `capacity * loadFactor`**（默认 `16 * 0.75 = 12`）时。

---

### **Q3：为什么 JDK 1.8 使用尾插法？**
**A**：
- 头插法在多线程环境下可能导致 **死循环**（链表形成环）。
- 尾插法能避免这个问题，提高稳定性。

---

### **Q4：为什么 JDK 1.8 使用红黑树？**
**A**：
- 当链表长度**超过 8** 时，查找效率降为 O(n)。
- **红黑树** 使查找变为 O(log n)，提高性能。

---

## 7. 总结
1. **HashMap 采用**数组 + 链表 + 红黑树** 结构。
2. **JDK 1.7 使用头插法，JDK 1.8 使用尾插法，避免死循环。**
3. **默认负载因子 `0.75`，超过 `capacity * 0.75` 时扩容。**
4. **多线程环境下请使用 `ConcurrentHashMap` 代替 `HashMap`**。
