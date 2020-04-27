## String

### JDK6的intern()

所有经过intern()处理的字符串被保存在PermGen中，其是Java堆的一个固定大小部分，存储类的信息和字符串常量池。

而JDK6+后的字符串常量池不再放在永久代，而移到了堆内存。

![intern](C:\Users\kyrie\Pictures\intern方法.PNG)

其中JDK6下，上述结果为false false

JDK7+下，结果为false true;

JDK6下，new 字符串时，字符串常量池和堆都会有"a",因此intern方法不会在常量池中拷贝一份"a"，则s!=s2;而在new s3时，字符串常量池中会有"a" ，堆中会有"aa"，intern方法就会在常量池中生产"aa"，s4指向常量池的"aa"，因此不等；

JDK7+下，第一种情况一样；第二种情况下，intern方法不会在常量池生成字符串"aa"，而是将生成引用指向堆中的"aa"，所有s3 s4都指向堆的"aa"，因此为true。

#### String 的hashcode()选用31作为乘数的原因

* 31是一个不大不小的质数，是作为 hashCode 乘子的优选质数之一。另外一些相近的质数，比如37、41、43等等，也都是不错的选择
* 31可以被 JVM 优化，31 * i = (i << 5) - i。