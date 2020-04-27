### 读写锁

```
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable {
    private static final long serialVersionUID = -6992448646407690164L;
    //内部类 ReadLock
    private final ReentrantReadWriteLock.ReadLock readerLock;
    //内部类 WriteLock
    private final ReentrantReadWriteLock.WriteLock writerLock;
    //注意的是不管是读锁还是写锁，获取锁的线程都被封装为Node节点在sync中排队
    final Sync sync;
    
    //构造方法
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);//读锁写锁公用ReentrantReadWriteLock 的sync
        writerLock = new WriteLock(this);
    }
```

读锁为共享锁，写锁为独占锁



### 锁降级&锁升级

```java
public static void main(String[] args) {
        ReentrantReadWriteLock rtLock = new ReentrantReadWriteLock();  
        rtLock.writeLock().lock();  
        System.out.println("writeLock");  
          
        rtLock.readLock().lock();  //允许锁降级
        System.out.println("get read lock");
}

public static void main(String[] args) {
        ReentrantReadWriteLock rtLock = new ReentrantReadWriteLock();
        rtLock.readLock().lock();
        System.out.println("get readLock.");
        rtLock.writeLock().lock();//不允许锁升级，会被阻塞
        System.out.println("blocking");
    }
```



