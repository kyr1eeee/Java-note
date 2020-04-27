### ThreadLocal

ThreadLocal可以设置存储属于当前线程的对象，存在当前线程（CurrentThread）的ThreadLocals中，ThreadLocals是ThreadLocalMap类的对象。

ThreadLocal的作用：

存储单个线程上下文信息。比如存储id等；减少参数传递。比如做一个trace工具，能够输出工程从开始到结束的整个一次处理过程中所有的信息，从而方便debug。由于需要在工程各处随时取用，可放入ThreadLocal。

#### 原理

```java
public void set(T value) {//为ThreadLocal对象设置value
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);//ThreadLocalMap对象的引用是由当前线程持有，ThreadLocal对象没有map引用!!!!
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();//如果map中没有或map为空，则返回初始值，注意初始值可能为null，也可能为之前设置的初始值。要想设置初始值，得使用子类SuppliedThreadLocal，也就是调用ThreadLocal.withInitial()创建对象。
    }
```



```java
    public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {
            return new SuppliedThreadLocal<>(supplier);
        }

    static final class SuppliedThreadLocal<T> extends ThreadLocal<T> {

        private final Supplier<? extends T> supplier;

        SuppliedThreadLocal(Supplier<? extends T> supplier) {
            this.supplier = Objects.requireNonNull(supplier);
        }

        @Override
        protected T initialValue() {
            return supplier.get();
        }
    }

private static  ThreadLocal<String> INT_OF_THREAD = ThreadLocal.withInitial(() ->"kyrie");
//像上述这样构造ThreadLocal对象，将返回子类SuppliedThreadLocal的对象，使用到了ThreadLocal里面的静态内部类SuppliedThreadLocal。
```

#### 内存泄漏问题

```java
static class ThreadLocalMap {

        
        static class Entry extends WeakReference<ThreadLocal<?>> {
           
            Object value;

            Entry(ThreadLocal<?> k, Object v) {//map中存放的key是ThreadLocal的弱引用
                super(k);
                value = v;
            }
        }
```

![](C:\Users\kyrie\Pictures\threadlocal.PNG)

上图是ThreadLocal对象及引用，ThreadLocalMap对象及引用的分布。

##### map中key使用弱引用的原因

如果使用强引用，当ThreadLocal对象（假设为ThreadLocal@123456）的引用（即：TL_INT，是一个强引用，指向ThreadLocal@123456）被回收了，ThreadLocalMap本身依然还持有ThreadLocal@123456的强引用，如果没有手动删除这个key，则ThreadLocal@123456不会被回收，所以只要当前线程不消亡，ThreadLocalMap引用的那些对象就不会被回收，可以认为这导致Entry内存泄漏。
如果使用弱引用，那指向ThreadLocal@123456对象的引用就两个：TL_INT强引用，和ThreadLocalMap中Entry的弱引用。一旦TL_INT被回收，则指向ThreadLocal@123456的就只有弱引用了，在下次gc的时候，这个ThreadLocal@123456就会被回收。
key被删了之后，变成了null，value更是无法被访问到了！针对这一问题，ThreadLocalMap类的设计本身已经有了这一问题的解决方案，那就是在每次get()/set()/remove()ThreadLocalMap中的值的时候，会自动清理key为null的value。如此一来，value也能被回收了。

如果面试问为什么不将value也设置为弱引用，方便清除处理？

那是因为在map外部很有可能没有value的强引用，value就可能随时被GC，那就没有使用ThreadLocal的必要了。（key在map外部一般会有一个全局的强引用，和value的情况不一样）

#### ThreadLocal与线程池

#### 参考文章

原文链接：https://blog.csdn.net/puppylpg/article/details/80433271