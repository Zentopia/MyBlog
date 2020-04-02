# WeakReference  在 WeakHashMap 和 ThreadLocalMap 中的使用

`WeakHashMap`
```java
static class Entry extends WeakReference<ThreadLocal<?>> {
           /** The value associated with this ThreadLocal. */
           Object value;

           Entry(ThreadLocal<?> k, Object v) {
               super(k);
               value = v;
           }
       }
```

`TheadLocalMap`
```java
 private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
        V value;
        final int hash;
        Entry<K,V> next;
...
}
```

`WeakHashMap` 和 `ThreadLocalMap` 中 Entry 的 key 使用的都是弱引用，如果 key 没有被强引用，那么在发生 GC 的情况下，key 对象会被回收。所以弱引用通常用来指向使用周期短暂的数据，以提高内存的使用效率。

key 被回收后，`WeakHashMap` 和 `ThreadLocalMap` 中就存在 key 为 null 的 entry，通常称之为 stale entry。Entry 中的 value 也无法被使用，如果被不能清理就会发生内存泄漏。

在 `WeakHashMap` 和 `ThreadLocalMap` 中，对 stale entry 的清理方式是不同的。

`WeakHashMap` 使用 expungeStaleEntries() 方法来清理：

```java
private void expungeStaleEntries() {
        for (Object x; (x = queue.poll()) != null; ) {
            synchronized (queue) {
                @SuppressWarnings("unchecked")
                    Entry<K,V> e = (Entry<K,V>) x;
                int i = indexFor(e.hash, table.length);

                Entry<K,V> prev = table[i];
                Entry<K,V> p = prev;
                while (p != null) {
                    Entry<K,V> next = p.next;
                    if (p == e) {
                        if (prev == e)
                            table[i] = next;
                        else
                            prev.next = next;
                        // Must not null out e.next;
                        // stale entries may be in use by a HashIterator
                        e.value = null; // Help GC
                        size--;
                        break;
                    }
                    prev = p;
                    p = next;
                }
            }
        }
    }
```

 其中的 queue 为 reference queue，用来存放需要被清理的 WeakEntries，即那些 stale entries。通过遍历来移除 table 中的 stale entries。

expungeStaleEntries() 方法会在 table 获取、扩容等场景下调用。

在 LocalThreadMap 中也有相应的 expungeStaleEntries() 方法：
```java
/**
         * Expunge a stale entry by rehashing any possibly colliding entries
         * lying between staleSlot and the next null slot.  This also expunges
         * any other stale entries encountered before the trailing null.  See
         * Knuth, Section 6.4
         *
         * @param staleSlot index of slot known to have null key
         * @return the index of the next null slot after staleSlot
         * (all between staleSlot and this slot will have been checked
         * for expunging).
         */
        private int expungeStaleEntry(int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;

            // expunge entry at staleSlot
            tab[staleSlot].value = null;
            tab[staleSlot] = null;
            size--;

            // Rehash until we encounter null
            Entry e;
            int i;
            for (i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                if (k == null) {
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    int h = k.threadLocalHashCode & (len - 1);
                    if (h != i) {
                        tab[i] = null;

                        // Unlike Knuth 6.4 Algorithm R, we must scan until
                        // null because multiple entries could have been stale.
                        while (tab[h] != null)
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            return i;
        }
```

 expungeStaleEntries()  在扩容rehash、set 等场景下会被调用，但是从源码中可以发现，expungeStaleEntries() 方法被调用的条件比较苛刻。因此，在使用 ThreadLocal 时，通常需要我们显示调用 ThreadLocal 的 remove() 方法来触发 expungeStaleEntries()。

当使用线程池时，因为存在线程重用机制，所以在使用完ThreadLocal 后，需要调用其 remove() 方法来避免内存泄漏。