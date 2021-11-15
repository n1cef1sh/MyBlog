---
layout: post
title: "Hashmap中用对象作为key的几点问题"
categories: [Java]

---



### Object类中的equals & hashcode

Java中所有的类都继承于Object类，当子类调用一个方法时，如果该方法没有被重写则需要往上面找到父类中的方法执行。而Object类中equals和hashcode的源码如下。

```JAVA
 public boolean equals(Object obj) {
        return (this == obj);
 }
 public native int hashCode();

```

hashcode是根据对象的内存地址经哈希算法得来的，equals比较规则就是比较两个对象的内存地址。

