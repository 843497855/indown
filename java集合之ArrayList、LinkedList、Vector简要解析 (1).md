# ***\*List、Map、Set三者区别\****

## ***\*List\****

List 接口存储一组不唯一（可以有多个元素引用相同的对象），有序对象。

1.也可以插入多个重复元素以及null元素。

2.有序是指输出时的顺序就是插入时的顺序。

3.常用的实现类有ArrayList、LinkedList和Vector。

## ***\*Set\****

不允许重复的集合。不会有多个元素引用相同的对象。

\1. 里面不允许有重复的元素；

\2. 无序容器，无法保证每个元素的存储顺序，TreeSet是通过Comparator或者Comparable维护了排列顺序；

\3. 只允许有一个null元素；

\4. 常用实现类有HashSet、LinkedHashSet以及TreeSet。

## ***\*Map\****

使用键值对存储。Map会维护与Key有关联的值。两个Key可以引用相同的对象，但Key不能重复，典型的Key是String类型，但也可以是任何对象。

\1. Map不是继承或者实现了Collection接口，它是一个接口；

\2. Map中的每个Entry都是持有两个对象，一个是key，一个是value，Map中的key必须是唯一的，但可以有相同的多个value；

\3. 常用实现类有HashMap、LinkedHashMap、HashTable和TreeMap.

# ***\*ArraList与LinkedList的区别\****

## ***\*线程是否安全\****

从底层源码来看都是不同步的，所以都没有保证线程安全。

 

## ***\*底层数据结构\****

ArrayList使用的Object数组结构默认为10（DEFAULT_CAPACITY=10	），LinkedList使用的是双向链表（JDK1.6之前为双向循环链表，JDK1.7取消了循环，为双向链表）。两种链表区别见下文。

## ***\*插入和删除是否受元素位置影响\****

1.ArrayList底层结构是数组，所以插入或删除的时间复杂度受到元素位置的影响，比如执行add(E e)方法时，ArrayList会默认将此元素追加到此数组的末尾，此时时间复杂度为O(1)。如果将此元素插入到指定的i位置add(int index,E e)，此时的时间复杂度为O(n-i)，因为在插入或删除时集合中的第i个和第i个元素之后的都会向后/向前移动一个位置。

add(E e)方法：size为当前集合容量大小，默认在集合最后一位追加。

```java
public boolean add(E e) {
    // 判断数组是否需要扩容		
	ensureCapacityInternal(size + 1);  // Increments modCount!!
	// 将元素添加到后一位
    elementData[size++] = e;
    return true;
}
```

add(int index,E element)方法：

```java
    public void add(int index, E element) {
        // 检查指定位置是否在范围之内
        rangeCheckForAdd(index);
        // 当前数组长度+1
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 复制成一个新的数组
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```

\2. LinkedList底层为双向链表结构，所以默认插入add(E e)方法，删除元素时间复杂度不受元素位置影响，近似O(1)，如果要在指定位置i插入和删除元素的话（add(int index,E element)）时间复杂度为O(n),因为要先移动到指定位置在插入。

add(E e)方法：默认将元素追加到最后一位，此时的速度和ArrayList差不多，甚至ArrayList还要好一些。

```java
 public boolean add(E e) {
     	//调用末尾追加方法
        linkLast(e);
        return true;
    }

 void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```



add(int index,E element)方法：

```java
public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
```

LinkedBefore(E e , Node<E> succ)方法

```java
 void linkBefore(E e, Node<E> succ) {
      
        // 将原位置的succ前驱指针改变指向newNode，将原succ前一个Node的后驱指针指向newNode
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

说明：两者的删除方法和添加指定元素时原理差不多。



## ***\*是否支持快速随机访问\****

1.ArrayList底层结构是数组，获取指定位置元素时就是获取数组指定下表的元素。如下get(int index)方法就可以看出ArrayList支持快速随机访问：

```java
	public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
  
    E elementData(int index) {
        // 就是取数组中index位置的数据
        return (E) elementData[index];
    }
```

2.LinkedList底层为链表结构，在访问index位置的元素时，需要根据index所在范围去遍历整个集合去查找，所以说此集合是不支持随机访问的，get(int index)方法如下：

```java
 	public E get(int index) {
        // 检查index是否在集合长度范围内
        checkElementIndex(index);
        return node(index).item;
   	}
	//根据索引分段查找元素
	Node<E> node(int index) {
       if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```



## ***\*内存空间占用\****

 ArrayList的空间浪费主要体现在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。



## ***\*关于循环使用\****

 首先说一下RandomAccess接口，查看源码发现RandomAccess接口未实现，如下：

```java
/**
 * RandomAccess接口
 * @since1.4
 */
public interface RandomAccess {
}
```

基本上我们就可以断定它是一个标识接口，类似于Spring中的AwareBeanFactory接口，RandomAccess接口可以说是标识一个集合是否可以快速随机访问。

而官方的解释为：

```properties
RandomAccess接口是一个标识接口，本身并没有提供任何方法，任何实现它的对象都可以认为是支持随机访问的对象。此接口的主要目的时标识那些可支持快速随机访问的List实现。
```

在Collections类中 binarySearch()方法中，它要判断传入的list类型是否实现RamdomAccess 接口，如果是，调用indexedBinarySearch()方法，如果不是，那么调用iteratorBinarySearch()方法。

```java
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key)  {
	// 判断是否实现了RandomAccess接口
    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        return Collections.indexedBinarySearch(list, key);
	else
		return Collections.iteratorBinarySearch(list, key);
}
```

在ArrayList类中实现了RandomAccess接口，而LinkedList没有实现，因为是与底层的数据结构有关，数组是天然支持随机访问，时间复杂度为O(1)，所以可以称之为快速随机访问。而LinkedList底层为链表，需要遍历到指定的位置才能找到，时间复杂度为O(n),所以不能称之为快速随机访问。

由此我们可以说实现了 RandomAccess 接口的list，优先选择普通 for 循环 ，其次 foreach；未实现 RandomAccess接口的list，优先选择iterator遍历（foreach遍历底层也是通过iterator实现的），大size的数据，千万不要使用普通for循环。 

 

## ***\*补充：循环链表和双向循环链表\****

1.***\*双向链表：\**** 包含两个指针，一个prev指向前一个节点，一个next指向后一个节点。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508154802710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE0OTgwOTc=,size_16,color_FFFFFF,t_70)

2.***\*双向循环链表：\**** 最后一个节点的 next 指向head，而 head 的prev指向最后一个节点，构成一个环。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508155001301.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE0OTgwOTc=,size_16,color_FFFFFF,t_70)

# ***\*ArrayList和Vector的区别\****

## ***\*线程是否安全\****

Vector实现类中的方法都有Synchronized关键字，所以说相对于ArrayList，Vector是线程安全的。

## ***\*性能\****

Vector实现类中的方法都有Synchronized关键字，在保持同步操作上会耗费大量的时间的，所以ArrayList的性能要优于Vector。

由两个线程安全地访问一个Vector对象。但是一个线程访问Vector的话，代码要在同步操作上耗费大量的时间。Arraylist不是同步的，所以在不需要保证线程安全时建议使用Arraylist。

## ***\*扩容\****

ArrayList 和 Vector 都会根据实际的需要动态的调整容量，只不过在 Vector 扩容每次会增加 1 倍，而 ArrayList 只会增加 50%。具体参看ArrayList扩容机制。

# ***\*ArrayList扩容机制\****

## ***\*ArrayList构造函数\****

1.无参构造函数：以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10

```java
    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    /**
     *默认构造函数，使用初始容量10构造一个空列表(无参数构造)
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
```
2.带有初始容量的构造函数

```java
 public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
        	//如果传入的容量大于0，创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
        	// 传入为0  则是创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

3. 带有初始化集合的构造函数

```java
 public ArrayList(Collection<? extends E> c) {
 		// 将用户传入的集合转为数组
        elementData = c.toArray();
        // 判断数组长度
        if ((size = elementData.length) != 0) {
        	//数组不为空 数组不为Object类型 复制为Object类型数组 
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 将数组置为空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
## ***\*扩容分析\****

这里以无参构造函数创建的ArrayList为例分析。

1.先看add(E e)方法

2.再来看看ensureCapacityInternal(int minCapacity)方法

 可以看到当数组为空的时候获取默认值和传入的值比较 得到较大的值，也就是创建一个为容量为10 的数组

```java
   //得到最小扩容量
   private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取默认的容量和传入参数的较大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
```

3. ensureExplicitCapacity(int minCapacity)方法

   如果调用了`ensureCapacityInternal()`方法一定会调用此方法判断是否需要扩容。

```java
 //判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容，调用此方法代表已经开始扩容了
            grow(minCapacity);
    }
```

根据创建的ArrayList看一下此方法源码：

- 当我们要add一个元素到ArrayList时，`elementData`的长度现在0，当此时调用了`ensureCapacityInternal()`方法，此时长度为10，所以minCapacity此时为10，此时方法中的`minCapacity - elementData.length > 0`条件成立，调用`grow(int minCapacity)`方法。
- 当add第2个元素时,调用`ensureCapacityInternal(int minCapacity)`方法，此时的minCapacity=2，此时`elementData.length`(容量)在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0` 条件不成立，不会调用`grow(int minCapacity)` 方法。
- 同理，直到add第11个元素时，此时minCapacity=11,`elementData.length`(容量)为10，`minCapacity - elementData.length > 0` 条件成立，此时需要调用`grow(int minCapacity)` 方法进行扩容。

4.grow方法

```java
  /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
       // 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
       //如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
在此方法中我们可以看到在oldCapacity + (oldCapacity >> 1); 对原来的容量进行了位运算，向右移动一位，也就是说新的容量是原来的1.5倍。
**补充：**
> ">>"（移位运算符）：>>1 右移一位相当于除2，右移n位相当于除以 2 的 n 次方。这里 oldCapacity 明显右移了1位所以相当于oldCapacity /2。对于大数据的2进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源 

 - 当add第1个元素时，oldCapacity 为0，经比较后第一个if判断成立，newCapacity = minCapacity(为10)。但是第二个if判断 不会成立，即newCapacity 不比 MAX_ARRAY_SIZE大，则不会进入 hugeCapacity 方法。数组容量为10，add方法中 return true,size增为1。  
 - 当add第11个元素进入grow方法时，newCapacity为15，比minCapacity（为11）大，第一个if判断不成立。新容量没有大于数组最大size，不会进入hugeCapacity方法。数组容量扩为15，add方法中return true,size增为11。
 - 如果新的扩容后的容量大于了最大的值MAX_ARRAY_SIZE ,则进入hugeCapacity方法进行判断，返回较大的值
5. hugeCapacity(int minCapacity)方法
```java
 private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
 private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //判断数组的长度和最大值哪个大就返回哪个
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```
 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) hugeCapacity() 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，如果minCapacity大于最大容量，则新容量则为Integer.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 Integer.MAX_VALUE - 8。
## ***\*System.arrayCopy()和Arrays.copyOf()\****
```java
  //elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的起始位置； size -      	  //index：要复制的数组元素的数量；
System.arraycopy(elementData, index, elementData, index + 1, size - index);
 
 //elementData：要复制的数组；size：要复制的长度
Arrays.copyOf(elementData, size);
```
在Arrays.copyOf中也是调用了System.arraycopy()方法,但是两者参数不同，arraycopy() 需要目标数组，将原数组拷贝到你自己定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置 copyOf() 是系统自动在内部新建一个数组，并返回该数组。
## *ensureCapacity方法*
```java
	/**
	 * 若有必要，则对集合进行扩容，以确保他能容纳由minCapacity指定的元素数
	 */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
```
实际上我们可以在插入大量数据之前调用此方法，对集合进行一次性的扩容操作，避免添加数据时 由于集合多次扩容操作，造成性能影响。

参考：
	https://snailclimb.gitee.io/javaguide/#/?id=java
	https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/ArrayList-Grow.md
