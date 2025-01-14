# Java String、StringBuffer、StringBuilder 详解（含 Java 7 和 Java 8 变化）

在 Java 开发中，**String** 是最常用的数据类型之一，而 **StringBuffer** 和 **StringBuilder** 也在字符串操作中扮演着重要角色。从 **Java 7 到 Java 8**，它们的底层实现发生了一些变化，影响了性能和使用方式。本文将深入解析它们的区别，并介绍 **Java 7 和 Java 8 的优化点**。

---

## 1. String 的特性与不可变性

### 1.1 **String 是不可变的（Immutable）**
```java
public class StringTest {
    public static void main(String[] args) {
        String str = "Hello";
        str += " World";
        System.out.println(str); // 输出: Hello World
    }
}
```
每次对 `String` 进行修改（如拼接）时，都会创建新的字符串对象，而原来的对象不会改变。这种设计可以：
- **保证线程安全**（不可变对象天然线程安全）
- **字符串常量池优化内存占用**

### 1.2 **Java 7 及 Java 8 对 String 的优化**
- **Java 6 及之前**：`String` 内部使用 `char[]` 存储字符。
- **Java 7（JEP 192）**：`char[]` 仍然是底层实现，但进行了小优化。
- **Java 8（JEP 254）**：将 `char[]` **改为 byte[]**，并新增 `coder` 字段用于存储编码信息，减少了 25%-50% 的内存占用。

**Java 8 源码对比**
```java
// Java 7 及之前（String.java）
private final char[] value;

// Java 8 及之后（String.java）
private final byte[] value;
private final byte coder;
```

#### Java 8 String 内存优化测试：
```java
public class StringMemoryTest {
    public static void main(String[] args) {
        String str1 = new String("Java7");
        String str2 = new String("Java8");

        System.out.println(str1.getClass().getDeclaredFields()[0]); // [C
        System.out.println(str2.getClass().getDeclaredFields()[0]); // [B (Java 8 变为 byte[])
    }
}
```
**结果：**
```
char[]（Java 7）
byte[]（Java 8）
```
这样 **Java 8 版本的 String 在内存占用上更加优化，尤其适用于 UTF-8 编码**。

---

## 2. StringBuffer：可变字符串（线程安全）

### 2.1 **StringBuffer 适用于多线程环境**
```java
public class StringBufferTest {
    public static void main(String[] args) {
        StringBuffer sb = new StringBuffer("Hello");
        sb.append(" World");
        System.out.println(sb); // 输出: Hello World
    }
}
```
**特点：**
- **线程安全**，适用于多线程环境（使用 `synchronized`）
- **比 String 性能高**，但同步机制会影响速度

### 2.2 **Java 7/8 对 StringBuffer 的优化**
- **Java 6 及之前**：底层依赖 `char[]`，每次扩容都会复制原数组到新数组，开销较大。
- **Java 7**：引入**延迟分配（Lazy Allocation）**，避免不必要的数组创建。
- **Java 8**：整体无明显改动，但 JVM 进行了 JIT（即时编译）优化，使 `StringBuffer` 的同步开销在特定情况下被优化掉。

---

## 3. StringBuilder：可变字符串（非线程安全）

### 3.1 **StringBuilder 适用于单线程环境**
```java
public class StringBuilderTest {
    public static void main(String[] args) {
        StringBuilder sb = new StringBuilder("Hello");
        sb.append(" World");
        System.out.println(sb); // 输出: Hello World
    }
}
```
**特点：**
- **非线程安全，但性能更高**
- **适用于单线程的字符串拼接**
- **底层与 StringBuffer 类似，但去掉了同步锁**

### 3.2 **Java 7/8 对 StringBuilder 的优化**
- **Java 7**：
  - **优化了扩容机制**，减少了不必要的数组复制，提高性能。
  - **append() 方法 JIT 编译优化**，减少性能损耗。
- **Java 8**：
  - **进一步优化 JIT，使无锁优化更明显**，在单线程环境下几乎没有额外开销。

---

## 4. String vs StringBuffer vs StringBuilder 性能对比

### 4.1 **性能测试**
```java
public class PerformanceTest {
    public static void main(String[] args) {
        long startTime, endTime;
        
        // String 测试
        startTime = System.currentTimeMillis();
        String str = "";
        for (int i = 0; i < 10000; i++) {
            str += i;
        }
        endTime = System.currentTimeMillis();
        System.out.println("String 执行时间：" + (endTime - startTime) + " ms");
        
        // StringBuffer 测试
        startTime = System.currentTimeMillis();
        StringBuffer sbf = new StringBuffer();
        for (int i = 0; i < 10000; i++) {
            sbf.append(i);
        }
        endTime = System.currentTimeMillis();
        System.out.println("StringBuffer 执行时间：" + (endTime - startTime) + " ms");
        
        // StringBuilder 测试
        startTime = System.currentTimeMillis();
        StringBuilder sbd = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            sbd.append(i);
        }
        endTime = System.currentTimeMillis();
        System.out.println("StringBuilder 执行时间：" + (endTime - startTime) + " ms");
    }
}
```

**运行结果（示例，仅供参考）：**
```
String 执行时间：500+ ms
StringBuffer 执行时间：10 ms
StringBuilder 执行时间：5 ms
```
可以看出：
- **String 的性能最差**，因为每次拼接都会创建新对象。
- **StringBuffer 由于同步开销，稍微比 StringBuilder 慢**。
- **StringBuilder 的性能最佳，适用于单线程环境**。

---

## 5. 结论：如何选择？

| 操作类型 | String | StringBuffer | StringBuilder |
|----------|--------|-------------|--------------|
| **可变性** | 不可变 | 可变 | 可变 |
| **线程安全** | 线程安全 | 线程安全（synchronized） | 非线程安全 |
| **适用场景** | 适用于少量字符串拼接 | 适用于多线程环境 | 适用于单线程环境 |
| **性能** | 最低 | 中等 | 最高 |
| **Java 7/8 变化** | 内存优化 | 延迟分配优化 | JIT 编译优化 |

### **推荐使用规则**
1. **默认使用 `StringBuilder`**（单线程场景，性能最佳）
2. **如果需要线程安全，则使用 `StringBuffer`**
3. **如果字符串不会改变，使用 `String`**

---

## 6. 总结
本篇文章深入解析了 **Java 7 和 Java 8** 以来对 **String、StringBuffer、StringBuilder** 的优化，并提供了性能测试和最佳实践建议。希望这篇文章能帮助你更好地理解 Java 的字符串操作方式！🚀

##### [👉👉👉点击获取2024Java学习资料](https://pan.quark.cn/s/0a774039f8f9)
>https://pan.quark.cn/s/0a774039f8f9
