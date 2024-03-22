### 一.java.lang

------

#### 1.Object源码

##### 1.native

带**native**的方法是java调用本地库（C的库），通过JIN（Java interface native）实现。这部分方法Java将其交给本地去实现。

* JIN调用C流程图

![](.\jdkimg\1.png)



##### 2.equals和==

```
   public boolean equals(Object obj) {
        return (this == obj);
    }
```

== 的作用：
　　基本类型（int，char，double，long，short，float，byte，boolean）：比较值是否相等
　　引用类型：比较内存地址值是否相等

equals 的作用:
　　引用类型：默认情况下，比较内存地址值是否相等。可以按照需求逻辑，重写对象的equals方法。



##### 3.clone

浅拷贝

> 被复制对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。换言之，浅复制仅仅复制所考虑的对象，而不复制它所引用的对象。

深拷贝

> 被复制对象的所有变量都含有与原来的对象相同的值，除去那些引用其他对象的变量。那些引用其他对象的变量将指向被复制过的新对象，而不再是原有的那些被引用的对象。换言之，深复制把要复制的对象所引用的对象都复制了一遍。

> 当然实现深拷贝还有一种方式是进行对象的**序列化**。

```
protected native Object clone() throws CloneNotSupportedException;
```



```
x.clone() != x		true
x.clone().getClass() == x.getClass();		true
x.clone().equals(x);		false
```



**对象要实现clone方法要先实现`Cloneable`接口**

如果一个类没有实现Cloneable接口（这个接口里面没有任何方法的声明，是一个标记接口），那么对此类的对象进行复制时，在运行时会出现CloneNotSupportedException异常。


```
class Person implements Cloneable {
    private String name;
    private House house;
    public Person(String name, House house) {
        this.setName(name);
        this.setHouse(house);
    }
    @Override
    public Object clone() {
        //浅拷贝
        try {
            return super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
            return null;
        }   
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public House getHouse() {
        return house;
    }
    public void setHouse(House house) {
        this.house = house;
    }

}
```



##### 4.toString

```
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

可以看到Object中toString方法的实现是返回类的名称（权限定名称）加上@，然后 加上此类的哈希码的16进制表示，例如：`com.ll.client.House@2a139a55`



##### 5.wait

timeout毫秒，nanos纳秒

```
public final native void wait(long timeout) throws InterruptedException;

public final void wait(long timeout, int nanos) throws InterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
                            "nanosecond timeout value out of range");
    }

    if (nanos > 0) {
        timeout++;
    }

    wait(timeout);
}

public final void wait() throws InterruptedException {
    wait(0);
}
```

可见wait()和wait(long timeout, int nanos)都在在内部调用了wait(long timeout)方法。

下面主要是说说wait(long timeout)方法

**wait方法会引起当前线程阻塞，直到另外一个线程在对应的对象上调用notify或者notifyAll()方法，或者达到了方法参数中指定的时间。**

调用wait方法的当前线程一定要拥有对象的监视器锁。

wait方法会把当前线程T放置在对应的object上的等到队列中，在这个对象上的所有同步请求都不会得到响应。线程调度将不会调用线程T，在以下四件事发生之前，线程T一直处于休眠状态（线程T是在其代码中调用wait方法的那个线程）

1. 当其他的线程在对应的对象上调用notify方法，而在此对象的对应的等待队列中将会任意选择一个线程进行唤醒。
2. 其他的线程在此对象上调用了notifyAll方法。
3. 其他的线程调用了interrupt方法来中断线程T。
4. 等待的时间已经超过了wait中指定的时间。如果参数timeout的值为0，不是指真实的等待时间是0，而是线程等待直到被另外一个线程唤醒。

被唤醒的线程T会被从对象的等待队列中移除并且重新能够被线程调度器调度。之后，线程T会像平常一样跟其他的线程竞争获取对象上的锁；一旦线程T获得了此对象上的锁，那么在此对象上的所有同步请求都会恢复到之前的状态，也就是恢复到wait被调用的情况下。然后线程T从wait方法的调用中返回。因此，当从wait方法返回时，对象的状态以及线程T的状态跟wait方法被调用的时候一样。



##### 6.notify方法

通知可能等待该对象的对象锁的其他线程。由**JVM(与优先级无关)随机挑选一个处于wait状态的线程**。

- 在调用notify()之前，线程必须获得该对象的对象级别锁
- 执行完notify()方法后，不会马上释放锁，要直到退出synchronized代码块，当前线程才会释放锁
- notify()一次只随机通知一个线程进行唤醒

##### 7.notifyAll()方法

和notify()差不多，只不过是使所有正在等待池中等待同一共享资源的全部线程从等待状态退出，进入可运行状态
让它们竞争对象的锁，只有获得锁的线程才能进入就绪状态
每个锁对象有两个队列：就绪队列和阻塞队列

- 就绪队列：存储将要获得锁的线程
- 阻塞队列：存储被阻塞的线程



#### 2.Boolean

##### 1.属性

```
private final boolean value;
```

##### 2.构造方法

```
public Boolean(boolean value) {
    this.value = value;
}

public Boolean(String s) {
//通过this调用构造方法
    this(parseBoolean(s));
}

public static boolean parseBoolean(String s) {
    return ((s != null) && s.equalsIgnoreCase("true"));
}
```

##### 3.hashcode

```
public int hashCode() {
    return Boolean.hashCode(value);
}
public static int hashCode(boolean value) {
    return value ? 1231 : 1237;
}
```

##### 4.Comparable<Boolean>接口

实现接口的`public int compareTo(T o);`方法

```
public int compareTo(Boolean b) {
    return compare(this.value, b.value);
}

public static int compare(boolean x, boolean y) {
    return (x == y) ? 0 : (x ? 1 : -1);
}
```



#### 3.String

**String类被final所修饰，也就是说String对象是不可变类，是线程安全的。**

```
public final class String implements java.io.Serializable, Comparable<String>, CharSequence
```

##### 1.常量

```
//用于存储字符串
private final char value[];
```

##### 2.构造函数

> String str1 = "abc";
> String str2 = new String("abc");
>
> **这两种创建方式的区别，str1地址指向字符串常量池，str2地址存放在栈中指向堆，两者地址不相同**

> String内部的char[]是final类型，意味着他不能被改变。由于String字符串的不可变性我们可以十分肯定常量池中一定不存在两个相同的字符串。那么上述两个str1和str2的字符串怎么内存地址不一样呢。可以这样理解。str1的内存地址指向字符串常量池是没错，str2的内存地址在栈中，指向堆中“abc”，然后堆中“abc”又引用了常量池中“abc”。但是str1和str2的内存地址确实不同。

```
//从bytes数组中的offset位置开始，将长度为length的字节，以charsetName格式编码，拷贝到value
public String(byte bytes[], int offset, int length, String charsetName)
        throws UnsupportedEncodingException {
    if (charsetName == null)
        throw new NullPointerException("charsetName");
    checkBounds(bytes, offset, length);
    this.value = StringCoding.decode(charsetName, bytes, offset, length);
}

//调用public String(byte bytes[], int offset, int length, String charsetName)构造函数
public String(byte bytes[], String charsetName)
        throws UnsupportedEncodingException {
    this(bytes, 0, bytes.length, charsetName);
}

//使用默认编码String csn = (charsetName == null) ? "ISO-8859-1" : charsetName;
public String(byte bytes[], int offset, int length) {
    checkBounds(bytes, offset, length);
    this.value = StringCoding.decode(bytes, offset, length);
}

```



##### 3.常用方法

equals

```
1. 内存地址相同，则为真。
2. 如果对象类型不是String类型，则为假。否则继续判断。
3. 如果对象长度不相等，则为假。否则继续判断。
4. 从后往前，判断String类中char数组value的单个字符是否相等，有不相等则为假。如果一直相等直到第一个数，则返回真。

  public boolean equals(Object anObject)
```



CompareTo

```
1.当返回值为0时表示两个字符串值一样
2.该方法可以判断三种情况

public int compareTo(String anotherString) {
    //自身对象字符串长度len1
    int len1 = value.length;
    //被比较对象字符串长度len2
    int len2 = anotherString.value.length;
    //取两个字符串长度的最小值lim
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    //从value的第一个字符开始到最小长度lim处为止，如果字符不相等，返回自身（对象不相等处字符-被比较对象不相等字符）
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    //如果前面都相等，则返回（自身长度-被比较对象长度）
    return len1 - len2;
}
```



hashcoede

1.s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1] ，对每个字符进行，31的幂运算
2.String类重写了hashCode方法，Object中的hashCode方法是一个Native调用。String类的hash采用多项式计算得来，我们完全可以通过不相同的字符串得出同样的hash，所以**两个String对象的hashCode相同，并不代表两个String是一样的**。

```
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```



startsWith

```
public boolean startsWith(String prefix, int toffset) {
    char ta[] = value;
    int to = toffset;
    char pa[] = prefix.value;
    int po = 0;
    int pc = prefix.value.length;
    // Note: toffset might be near -1>>>1.
    //如果起始地址小于0或者（起始地址+所比较对象长度）大于自身对象长度，返回假
    if ((toffset < 0) || (toffset > value.length - pc)) {
        return false;
    }
    //从所比较对象的末尾开始比较
    while (--pc >= 0) {
        if (ta[to++] != pa[po++]) {
            return false;
        }
    }
    return true;
}

public boolean startsWith(String prefix) {
    return startsWith(prefix, 0);
}

public boolean endsWith(String suffix) {
    return startsWith(suffix, value.length - suffix.value.length);
}
```



concat

```
 1.先判断str是否为空，空就返回原对象
 2.不空就创建一新的字符数组，将俩个字符串串联，返回一个新String
 3."cares".concat("s") returns "caress"
  "to".concat("get").concat("her") returns "together"
  
 public String concat(String str)
```



trim

```
1.去掉字符串首尾空格字符
2.先判断首位出现非''字符位置，再用subString取出

public String trim() {
        int len = value.length;
        int st = 0;
        char[] val = value;    /* avoid getfield opcode */

        while ((st < len) && (val[st] <= ' ')) {
            st++;
        }
        while ((st < len) && (val[len - 1] <= ' ')) {
            len--;
        }
        return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
    }
```



intern

```
//intern方法是Native调用，它的作用是在方法区中的常量池里通过equals方法寻找等值的对象，如果没有找到则在常量池中开辟一片空间存放字符串并返回该对应String的引用，否则直接返回常量池中已存在String对象的引用。

public native String intern();
```





#### 4.abstractStringbuilder

​	我们在jdk1.8的源码中可以看到，StringBuilder类和StringBuffer类都是继承了AbstractStringBuilder类的，并且很多方法都是直接使用的父类AbstractStringBuilder的方法，所以在学习StringBuilder类和StringBuffer类之前，先来研究一下AbstractStringBuilder的源码。

##### 1.申明

```
abstract class AbstractStringBuilder implements Appendable, CharSequence
```

类名用abstract修饰说明是一个抽象类，只能被继承，不能直接创建对象。查了里面的方法你会发现它就一个抽象方法，toString方法。 同时可以看到AbstractStringBuilder实现了Appendable和CharSequence接口。

* 实现了 Appendable能够被追加 char 序列和值的对象，实现的几个方法如下：

>  a. append(CharSequence csq) throws IOException:如何添加一个字符序列
>
> b. append(CharSequence csq, int start, int end) throws IOException：如何添加一个字符序列的一部分
>
> c. append(char c) throws IOException:如何添加一个字符

*  实现CharSequence字符序列的接口，代表该类，或其子类是一个字符序列 ：

> a. 规定了需要实现该字符序列的长度:length()；
>
> b. 可以取得下标为index的的字符：charAt(int index)；
>
> c. 可以得到该字符序列的一个子字符序列： subSequence(int start, int end)；
>
> d. 规定了该字符序列的String版本（重写了父类Object的toString()）：toString();



##### 2.成员变量

```
    //定义存放内容的字符数组，数组的长度即为容量
    char[] value;
    //定义内容实际的长度count<=value.length
    int count;
```





##### 3.ensureCapacity()扩容方法以及内部调用的方法

每次扩容大小为2n+2

```
public void ensureCapacity(int minimumCapacity) {
        //判断扩充的目标容量值是否大于0
        if (minimumCapacity > 0)
            ensureCapacityInternal(minimumCapacity);
    }

    private void ensureCapacityInternal(int minimumCapacity) {
        //如果目标容量大于原本容量，则调用Arrays.copyOf，长度则由newCapacity(minimumCapacity)方法得出
        if (minimumCapacity - value.length > 0) {
            value = Arrays.copyOf(value,
                    newCapacity(minimumCapacity));
        }
    }
    //先定义字符数组的最大存储容量，减去8是因为数组作为一个对象，需要一定的内存存储对象头信息，对象头信息最大占用内存不可超过8个字节，数组的对象头信息相较于其他Object，多了一个表示数组长度的信息。
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    //这个方法是计算容量的核心逻辑
    private int newCapacity(int minCapacity) {
        //这里表示每次扩容都是2n+2,就有可能出现int越界，导致newCapacity从正数变为了负数
        int newCapacity = (value.length << 1) + 2;
        //这里判断相比较于newCapacity < minCapacity的写法，规避了newCapacity越界，让newCapacity 也能够被赋予一个正值。
        //而正常情况的时候，就是newCapacity扩容后还是没有目标容量minCapacity大，就直接设置扩容后的容量为目标容量minCapacity。
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        //如果newCapacity是正常的正数，直接返回；如果不是，则进入hugeCapacity()方法
        return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
            ? hugeCapacity(minCapacity)
            : newCapacity;
    }
    //如果目标容量大于了数组最大容量MAX_ARRAY_SIZE，则返回目标容量，小于则返回数组最大容量MAX_ARRAY_SIZE
    private int hugeCapacity(int minCapacity) {
        if (Integer.MAX_VALUE - minCapacity < 0) {
            throw new OutOfMemoryError();
        }
        return (minCapacity > MAX_ARRAY_SIZE)
            ? minCapacity : MAX_ARRAY_SIZE;
    }
```

**注意：看到很多判断都是采用 a - b < 0 而不是直接用 a < b 。是因为本身与0比较大小运算更快一些，还能有效的规避一些int值越界的问题。**



##### 4.trimToSize()

trimToSize()方法用来存放实际用的容量的值，没有值的容量被释放，执行后count = value.length

```
public void trimToSize() {
        if (count < value.length) {
            value = Arrays.copyOf(value, count);
        }
    }
```

值得注意的是， 原来的数组由于没有被任何引用所指向，之后会被gc回收。



##### 5.append(String str)

```
 public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
```

>a. 首先判断所传参数是否为null，如果为null则调用appendNull方法，实际上就是在原字符序列后加上"null"序列。
>
>b. 如果不为null则进行扩容操作，最小值为count+len，这一步可能增加容量也可能不增加，当count+len小于或等于capacity就不用进行扩容。
>
>c. 然后再将参数的字符串序列添加到value中。
>
>d. **最后返回this,注意这里返回的是this**，即AbstractStringBuilder对象本身，也就意味者，可以在一条语句中多次调用append方法。



##### 6.delete(int start, int end)

```
public AbstractStringBuilder delete(int start, int end) {
        if (start < 0)
            throw new StringIndexOutOfBoundsException(start);
        if (end > count)
            end = count;
        if (start > end)
            throw new StringIndexOutOfBoundsException();
        int len = end - start;
        if (len > 0) {
            System.arraycopy(value, start+len, value, start, count-end);
            count -= len;
        }
        return this;
    }
```

此方法可以删掉value数组的[start,end)部分，并将end后面的数据移到start位置。 删除字符序列指定区间的内容后不改变原序列的容量。



##### 7.deleteCharAt(int codePoint)方法可以删除下标为index的数据，并将后面的数据前移一位。

```
public AbstractStringBuilder deleteCharAt(int index) {
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        System.arraycopy(value, index+1, value, index, count-index-1);
        count--;
        return this;
    }
```



#### 5.StringBuilder

##### 1.构造方法

```
    public StringBuilder() {
        super(16);
    }
 
    public StringBuilder(int capacity) {
        super(capacity);
    }
 
    public StringBuilder(String str) {
        super(str.length() + 16);
        append(str);
    }
 
    public StringBuilder(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }
```



2.

### 二.java.util

------

#### 1.集合源码

![](.\jdkimg\2.png)

##### 1.ArrayList

* ArrayList使用**数组**来存储数据，是线程不安全的。

默认分配10的大小

```
/**
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;
```

按1.5倍进行扩容

```
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        
        //oldCapacity >> 1变为一半+一个oldCapacity，ArrayList的Capacity变为原来1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);  
        
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```



##### 2.Vector

* 实现方法和ArrayList很像，但是线程安全，效率低于ArrayList

```
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
    
    public synchronized boolean removeElement(Object obj) {
        modCount++;
        int i = indexOf(obj);
        if (i >= 0) {
            removeElementAt(i);
            return true;
        }
        return false;
    }
```



##### 3.LinkedList

使用**双向链表**，增删很快，但是查找较慢，因为其实现的是双向链表，所以，可以把它当栈和队列去使用

```
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    //在链表头插入node
    private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}

```

