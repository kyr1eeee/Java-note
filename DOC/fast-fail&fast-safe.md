### fast fail

```java
public void test(kk arg) {	
	if (arg == null) {
		throw new RuntimeException("wanger 不能为空");
	}
	
	System.out.println(arg.toString());
}
```

一旦检测到 arg 为 null，就立马抛出异常，让调用者来决定这种情况下该怎么处理，下一步 `arg.toString()` 就不会执行了——避免更严重的错误出现。这就是fast fail 的一种体现。

#### for each 集合的remove

```java
List<String> list = new ArrayList<>();
list.add("沉默王二");
list.add("沉默王三");
list.add("一个文章真特么有趣的程序员");

for (String str : list) {
	if ("沉默王二".equals(str)) {
		list.remove(str);
	}
}

System.out.println(list);

```

这段代码看起来没有任何问题,但运行起来出现运行时异常。

```java
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at com.kyrie.test.extendTest.ExtendTest1.main(ExtendTest1.java:16)
```

分析ArrayList的源码会发现，remove 的时候执行了 `checkForComodification` 方法，该方法对 modCount 和 expectedModCount 进行了比较，发现两者不等，就抛出了 `ConcurrentModificationException` 异常。



到底哪里执行了`checkForComodification`方法？反编译for循环的代码，

```java
List<String> list = new ArrayList();
list.add("沉默王二");
list.add("沉默王三");
list.add("一个文章真特么有趣的程序员");
Iterator var3 = list.iterator();

while (var3.hasNext()) {
	String str = (String) var3.next();
	if ("沉默王二".equals(str)) {
		list.remove(str);
	}
}

System.out.println(list);
```

原来用到了迭代器，而迭代器的hasNext方法

```java
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    Itr() {}

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();//在这！
        int i = cursor;
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }
}
```

也就是说 `new Itr()` 的时候 expectedModCount 被赋值为 modCount，而 modCount 是 List 的一个成员变量，表示集合被修改的次数。由于 list 此前执行了 3 次 add 方法，所以 modCount 的值为 3；expectedModCount 的值也为 3。

可当执行 `list.remove(str)` 后，modCount 的值变成了 4。

```java
private void fastRemove(int index) {//使用集合的remove方法调用了fastRemove，修改了modCount
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

**总结一下**。在 `for each` 循环中，集合遍历其实是通过迭代器 Iterator 配合 while 循环实现的，但是元素的 remove 却直接使用的集合类自身的方法。这就导致 Iterator 在遍历的时候，会发现元素在自己不知情的情况下被修改了，就抛出了异常。**Iterator**就使用了fast-fail的保护机制

#### 使用迭代器的remove方法避开fast-fail的保护机制

```java
List<String> list = new ArrayList<>();
list.add("沉默王二");
list.add("沉默王三");
list.add("一个文章真特么有趣的程序员");

Iterator<String> itr = list.iterator();

while (itr.hasNext()) {
	String str = itr.next();
	if ("沉默王二".equals(str)) {
		itr.remove();
	}
}
```

看一下迭代器的remove做了什么

```java
public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();

    try {
        ArrayList.this.remove(lastRet);//还是使用集合自身的remove方法
        cursor = lastRet;
        lastRet = -1;
        expectedModCount = modCount;//只不过还顺便把expectedModCount更新了
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}

```

从广义上讲，fail-fast 指的是当有异常或者错误发生时就立即中断执行的这种设计。

### fail safe

为了避免触发fail-fast机制，导致异常，我们可以使用Java中提供的一些采用了fail-safe机制的集合类。

这样的集合容器在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

java.util.concurrent包下的容器都是fail-safe的，可以在多线程下并发使用，并发修改。同时也可以在foreach中进行add/remove 。

但是，虽然基于拷贝内容的优点是避免了ConcurrentModificationException，但同样地，迭代器并不能访问到修改后的内容。

```java
List<String> userNames = new CopyOnWriteArrayList<String>() {{
            add("Hollis");
            add("hollis");
            add("HollisChuang");
            add("H");
        }};

        Iterator it = userNames.iterator();

        for (String userName : userNames) {
            if (userName.equals("Hollis")) {
                userNames.remove(userName);
            }
        }

        System.out.println(userNames);
        
        while(it.hasNext()){
            System.out.println(it.next());
        }
```

debug

![](C:\Users\kyrie\Pictures\迭代器快照.PNG)

可以看到迭代器保存了一份快照

#### CopyOnWriteArrayList

###### CopyOnWriteArrayList的add/remove等方法都已经加锁了，还要copy一份再修改干嘛？多此一举？同样是线程安全的集合，这玩意和Vector有啥区别呢？

Copy-On-Write简称COW，是一种用于程序设计中的优化策略。其基本思路是，从一开始大家都在共享同一个内容，当某个人想要修改这个内容的时候，才会真正把内容Copy出去形成一个新的内容然后再改，这是一种延时懒惰策略。

CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。

CopyOnWriteArrayList中add/remove等写方法是需要加锁的，目的是为了避免Copy出N个副本出来，导致并发写。但是，CopyOnWriteArrayList中的读方法是没有加锁的。也就是可以支持读写并发。

这样做的好处是我们可以对CopyOnWrite容器进行并发的读，当然，这里读到的数据可能不是最新的。因为写时复制的思想是通过延时更新的策略来实现数据的**最终一致性的，并非强一致性。**

**所以CopyOnWrite容器是一种读写分离的思想，读和写不同的容器。**而Vector在读写的时候使用同一个容器，读写互斥，同时只能做一件事儿。

