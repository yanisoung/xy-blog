---
title: 实现自己的HashMap
date: 2019-09-21 20:35:53
categories:
- 编程
tags:
- java
- 基础
keywords: hashmap,java,实现自己的HashMap
---

HsahMap1.7jdk原理分析：https://www.jianshu.com/p/068b6d25b593

* HashMap中重要的概念：
初始化容量直： 1 << 4 map的初始化容量
最大容量值：1 << 30 map可容纳的最大容量
扩容因子 ： 0.75  即容量已达到该设置条件,put时会进行扩容
扩容阈值：扩容阈值=容量(table的size)*扩容因子
size:map中存储的键值对的数量
table：存储实体
实际扩容因子：创建时可由有参构造给出
实际容量：创建时可由有参构造给出
扩容次数：可限制扩容
* key允许为null,存储时key为Null,在table中的下标为0.取值时从table[0]中获取链表，查找链表中k为null的节点。
* HashMap使用数组+链表的形式实现的，通过计算key的hash值获取key在数组中对应的下标，添加新的节点到该下标的链表中。
* 取值时，通过计算key的hash值获取key在数组中对应的下标，获取该下标上的链表，通过eques方法从链表中找到对应的value。


<!-- more -->

*以下是代码实现：


    public class MyHashMap<K, V> {
        /**
         * 初始化容量 1的4次方
         */
        private final Integer DEFAULT_INITIAL_CAPACITY = 1 << 4;

        /**
         * 最大容量 1的30次方 超过最大量将被最大量所赋值
         */
        private final Integer MAXIMUM_CAPACITY = 1 << 30;

        /**
         * 实际容量
         */
        private Integer capacity;

        /**
         * 默认扩容因子 四分之三
         */
        private final float DEFAULT_LOAD_FACTOR = 0.75f;

        /**
         * 实际扩容因子
         */
        private float loadFactor;

        /**
         * 扩容阈值=容量(table的size)*扩容因子
         */
        private int threshold;

        /**
         * 实际存储
         */
        private MyEntry<K, V>[] table = null;

        /**
         * map 的size  既Map中键值对的数量
         */
        private static int size = 0;
        /**
         * 扩容次数：可做扩容限制
         */
        private static int resizeIndex = 0;

        /**
         * 无参构造
         */
        public MyHashMap() {
            //实际扩容因子
            this.loadFactor = DEFAULT_LOAD_FACTOR;
            //实际容量
            this.capacity = DEFAULT_INITIAL_CAPACITY;
            //扩容阈值
            this.threshold = tableSizeFor(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
            //初始化table
            this.table = new MyEntry[DEFAULT_INITIAL_CAPACITY];
        }

        /**
         * @param initialCapacity 初始化容量
         * @param loadFactor      扩容因子
         */
        public MyHashMap(int initialCapacity, float loadFactor) {
            //如果初始化容量大于默认最大容量 则等于最大容量
            if (initialCapacity >= MAXIMUM_CAPACITY) {
                initialCapacity = MAXIMUM_CAPACITY;
            }
            //实际扩容因子
            this.loadFactor = loadFactor;
            //实际容量
            this.capacity = initialCapacity;
            //计算扩容阈值
            this.threshold = tableSizeFor(initialCapacity, loadFactor);
            //初始化table
            this.table = new MyEntry[initialCapacity];
        }

        /**
         * @param initialCapacity 初始化容量
         */
        public MyHashMap(int initialCapacity) {
            if (initialCapacity >= MAXIMUM_CAPACITY) {
                initialCapacity = MAXIMUM_CAPACITY;
            }
            //实际容量
            this.capacity = initialCapacity;
            //实际扩容因子
            this.loadFactor = DEFAULT_LOAD_FACTOR;
            //扩容阈值
            this.threshold = tableSizeFor(initialCapacity, loadFactor);
            //初始化table
            this.table = new MyEntry[initialCapacity];
        }

        public V get(K k) {
            if (null == table) {
                return null;
            }
            if (null == k) {
                return getForNullKey();
            }
            //计算k对应的hash值,根据hash计算在table中对应下标
            int hash = k.hashCode();
            int index = getIndex(hash, capacity);
            MyEntry<K, V> entry = table[index];
            while (null != entry) {
                //hash值相同 同时key相同
                if (hash == entry.getHash() && (k == entry.getK() || k.equals(entry.getK()))) {
                    return entry.getV();
                }
                entry = entry.getNext();
            }
            return null;
        }

        /**
         * 获取key为null时的值
         *
         * @return
         */
        private V getForNullKey() {
            MyEntry<K, V> entry = table[0];
            while (null != entry) {
                if (null == entry.getHash() && null == entry.getK()) {
                    return entry.getV();
                }
                entry = entry.getNext();
            }
            return null;
        }

        /**
         * 添加数据
         *
         * @param k
         * @param v
         * @return
         */
        public V put(K k, V v) {
            //put时扩容
            resize();
            //key==null时
            if (null == k) {
                return putForNullKey(k, v);
            }
            //计算k的hash
            int hash = k.hashCode();
            //计算k在数组中对应的下标
            int index = getIndex(hash, capacity);

            //获取对应下标上的Entry:若新增时 则为下个节点
            MyEntry<K, V> nextEntry = table[index];
            for (MyEntry<K, V> entry = table[index]; entry != null; entry = entry.getNext()) {
                //判断是否已经存在对应的key
                if (hash == entry.getHash() && (k == entry.getK() || k.equals(entry.getK()))) {
                    V oldValue = entry.getV();
                    entry.setV(v);
                    return oldValue;
                }
            }
            size++;
            //新增
            table[index] = new MyEntry<>(k, v, nextEntry, hash);
            //新增时返回Null
            return null;
        }

        /**
         * 存储key为null的值
         *
         * @param k
         * @return
         */
        private V putForNullKey(K k, V v) {
            MyEntry<K, V> entry = table[0];
            //判断是否已经存在key为null的键值对，如果存在则替换
            while (null != entry) {
                if (null == entry.getHash() && null == entry.getK()) {
                    V oldValue = entry.getV();
                    entry.setV(v);
                    entry.setNext(entry);
                    return oldValue;
                }
                entry = entry.getNext();
            }
            //新增
            size++;
            table[0] = new MyEntry<>(k, v, entry, null);
            return null;
        }

        /**
         * 计算k在数组中对应的下标
         *
         * @param hash     hash值
         * @param capacity table容量
         * @return
         */
        private int getIndex(int hash, int capacity) {
            //table的容量
            int index = hash % capacity;
            return index >= 0 ? index : -index;
        }

        /**
         * 扩容
         */
        private void resize() {
            //已是最大扩容，不再扩容
            if (capacity >= MAXIMUM_CAPACITY) {
                //将阈值设置为Inter的最大容量
                threshold = Integer.MAX_VALUE;
                return;
            }
            //未达到最大阈值 无需扩容
            if (size < threshold) {
                return;
            }
            int newCapacity = capacity * 2;
            if (newCapacity >= MAXIMUM_CAPACITY) {
                newCapacity = MAXIMUM_CAPACITY;
            }
            //扩容次数
            resizeIndex++;
            //重新计算扩容阈值
            threshold = tableSizeFor(newCapacity, loadFactor);
            //新容量
            capacity = newCapacity;
            MyEntry<K, V>[] oldEntry = table;
            //创建新数组
            MyEntry<K, V>[] newEntry = new MyEntry[newCapacity];
            //将旧数组的值转移到新数组
            transition(oldEntry, newEntry);
            //重新定义table
            table = newEntry;
        }

        /**
         * 转移数组上的数据
         *
         * @param oldEntries
         * @param newEntries
         */
        private void transition(MyEntry<K, V>[] oldEntries, MyEntry<K, V>[] newEntries) {
            for (int i = 0; i < oldEntries.length; i++) {
                MyEntry<K, V> entry = oldEntries[i];
                while (null != entry) {
                    MyEntry<K, V> next = entry.getNext();
                    //扩容之后重新计算在数组中的下标
                    int index = getIndex(entry.getK().hashCode(), capacity);
                    //将元素放在数组上：
                    //采用单链表的头插入方式 = 在链表头上存放数据 = 将数组位置的原有数据放在后1个指针、将需放入的数据放到数组位置中
                    // 即 扩容后，可能出现逆序：按旧链表的正序遍历链表、在新链表的头部依次插入
                    entry.setNext(newEntries[index]);
                    //赋值到新数组
                    newEntries[index] = entry;
                    entry = next;
                }
            }
        }

        /**
         * 计算扩容阈值
         */
        private int tableSizeFor(int initialCapacity, float loadFactor) {
            BigDecimal init = new BigDecimal(initialCapacity);
            BigDecimal bigLoad = new BigDecimal(loadFactor);
            BigDecimal multiply = init.multiply(bigLoad);
            return multiply.intValue();
        }
    }



* entry组成了链表


        public final class MyEntry<K, V> {

        private K k;

        private V v;

        private MyEntry<K, V> next;

        private Integer hash;

        public MyEntry(K k, V v, MyEntry<K, V> next, Integer hash) {
            this.k = k;
            this.v = v;
            this.next = next;
            this.hash = hash;
        }

        public MyEntry() {

        }


        public MyEntry(K k, V v) {
            this.k = k;
            this.v = v;
        }

        public MyEntry(K k, V v, MyEntry<K, V> next) {
            this.k = k;
            this.v = v;
            this.next = next;
        }

        public Integer getHash() {
            return hash;
        }

        public void setHash(Integer hash) {
            this.hash = hash;
        }

        public K getK() {
            return k;
        }

        public void setK(K k) {
            this.k = k;
        }

        public V getV() {
            return v;
        }

        public void setV(V v) {
            this.v = v;
        }

        public MyEntry<K, V> getNext() {
            return next;
        }

        public void setNext(MyEntry<K, V> next) {
            this.next = next;
        }
    }


* 最后测试一下：

        @Test
        public void demo() {
            MyHashMap<String, String> map = new MyHashMap();
            System.out.println("-------------------MyHashMap begin-------------------------");
            Long t1 = System.currentTimeMillis();
            for (int i = 0; i < 1000; i++) {
                map.put("key" + i, "value" + i);
            }
            for (int i = 0; i < 1000; i++) {
                String value = map.get("key" + i);
                System.out.println("key: " + "key" + i + "---" + "value: " + value);
                assert (value.equals("value" + i));
            }
            //null key
            map.put(null, "value:key is null");
            System.out.printf(map.get(null));
            Long t2 = System.currentTimeMillis();
            System.out.println("MyHashMap耗时：" + (t2 - t1));
            System.out.println("-------------------MyHashMap end-------------------------");
            Map<String, String> hashMap = new HashMap();
            Long t3 = System.currentTimeMillis();
            for (int i = 0; i < 1000; i++) {
                hashMap.put("key" + i, "value" + i);
            }
            for (int i = 0; i < 1000; i++) {
                System.out.println("key: " + "key" + i + "---" + "value: " + hashMap.get("key" + i));
            }
            Long t4 = System.currentTimeMillis();
            System.out.println("JDK的HashMap耗时：" + (t4 - t3));

        }