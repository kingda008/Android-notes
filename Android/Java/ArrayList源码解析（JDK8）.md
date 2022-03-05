概述
很久没有写博客了，准确的说17年以来写博客的频率降低到一个不忍直视的水平。这个真不怪我，给大家解释一下。
一是自从做了leader，整天各种事，开会，过需求，无限循环。心很累，时间也被无线压榨
二 我本身也在学习一些其他的技术，比如ReactNative，也看了半天的kotlin，撸了几个groovy脚本、gradle插件。
三 是打算找工作了。又要开始复习数据结构和算法。

得，脑海中有很多躺了无数天的，甚至半成稿的博客，比如组件化、Rxjava一些使用注意点，都被我搁置delay了。
这眼瞧着，很久没写了，写点啥吧。

正巧最近在看jdk的Collection集合源码，这种单个类的源码解析，写起来还算比较方便。
关键代码处加上注释，核心处做个总结，就可以成文，拿出来和大家讨论分享。
且网上绝大多数都是JDK7甚至之前的源码解析文章。咱们也要与时俱进。
这似乎是我回归博客的一个不错选择。

那下面就跟我一起撸起ArrayList的源码吧。

本文将从几个常用方法下手，来阅读ArrayList的源码。
按照从构造方法->常用API（增、删、改、查）的顺序来阅读源码，并会讲解阅读方法中涉及的一些变量的意义。了解ArrayList的特点、适用场景。

如果本文中有不正确的结论、说法，请大家提出和我讨论，共同进步，谢谢。

概要
概括的说，ArrayList 是一个动态数组，它是线程不安全的，允许元素为null。
其底层数据结构依然是数组，它实现了List<E>, RandomAccess, Cloneable, java.io.Serializable接口，其中RandomAccess代表了其拥有随机快速访问的能力，ArrayList可以以O(1)的时间复杂度去根据下标访问元素。

因其底层数据结构是数组，所以可想而知，它是占据一块连续的内存空间（容量就是数组的length），所以它也有数组的缺点，空间效率不高。

由于数组的内存连续，可以根据下标以O1的时间读写(改查)元素，因此时间效率很高。

当集合中的元素超出这个容量，便会进行扩容操作。扩容操作也是ArrayList 的一个性能消耗比较大的地方，所以若我们可以提前预知数据的规模，应该通过public ArrayList(int initialCapacity) {}构造方法，指定集合的大小，去构建ArrayList实例，以减少扩容次数，提高效率。

或者在需要扩容的时候，手动调用public void ensureCapacity(int minCapacity) {}方法扩容。
不过该方法是ArrayList的API，不是List接口里的，所以使用时需要强转:
((ArrayList)list).ensureCapacity(30);

当每次修改结构时，增加导致扩容，或者删，都会修改modCount。

构造方法
    //默认构造函数里的空数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    //存储集合元素的底层实现：真正存放元素的数组
    transient Object[] elementData; // non-private to simplify nested class access
    //当前元素数量
    private int size;
    
    //默认构造方法
    public ArrayList() {
        //默认构造方法只是简单的将 空数组赋值给了elementData
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
    //空数组
    private static final Object[] EMPTY_ELEMENTDATA = {};
    //带初始容量的构造方法
    public ArrayList(int initialCapacity) {
        //如果初始容量大于0，则新建一个长度为initialCapacity的Object数组.
        //注意这里并没有修改size(对比第三个构造函数)
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {//如果容量为0，直接将EMPTY_ELEMENTDATA赋值给elementData
            this.elementData = EMPTY_ELEMENTDATA;
        } else {//容量小于0，直接抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
    
    //利用别的集合类来构建ArrayList的构造函数
    public ArrayList(Collection<? extends E> c) {
        //直接利用Collection.toArray()方法得到一个对象数组，并赋值给elementData 
        elementData = c.toArray();
        //因为size代表的是集合元素数量，所以通过别的集合来构造ArrayList时，要给size赋值
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)//这里是当c.toArray出错，没有返回Object[]时，利用Arrays.copyOf 来复制集合c中的元素到elementData数组中
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            //如果集合c元素数量为0，则将空数组EMPTY_ELEMENTDATA赋值给elementData 
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
小结一下，构造函数走完之后，会构建出数组elementData和数量size。

这里大家要注意一下Collection.toArray()这个方法，在Collection子类各大集合的源码中，高频使用了这个方法去获得某Collection的所有元素。

关于方法：Arrays.copyOf(elementData, size, Object[].class)，就是根据class的类型来决定是new 还是反射去构造一个泛型数组，同时利用native函数，批量赋值元素至新数组中。
如下：

    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        //根据class的类型来决定是new 还是反射去构造一个泛型数组
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        //利用native函数，批量赋值元素至新数组中。
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
1
2
3
4
5
6
7
8
9
10
11
值得注意的是，System.arraycopy也是一个很高频的函数，大家要留意一下。

    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
1
2
3
常用API
1 增
每次 add之前，都会判断add后的容量，是否需要扩容。

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;//在数组末尾追加一个元素，并修改size
    return true;
}
    private static final int DEFAULT_CAPACITY = 10;//默认扩容容量 10
    private void ensureCapacityInternal(int minCapacity) {
        //利用 == 可以判断数组是否是用默认构造函数初始化的
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }


private void ensureExplicitCapacity(int minCapacity) {
    modCount++;//如果确定要扩容，会修改modCount 

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

//需要扩容的话，默认扩容一半
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);//默认扩容一半
    if (newCapacity - minCapacity < 0)//如果还不够 ，那么就用 能容纳的最小的数量。（add后的容量）
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);//拷贝，扩容，构建一个新数组，
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36

public void add(int index, E element) {
    rangeCheckForAdd(index);//越界判断 如果越界抛异常

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index); //将index开始的数据 向后移动一位
    elementData[index] = element;
    size++;
}
1
2
3
4
5
6
7
8
9
10
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount //确认是否需要扩容
    System.arraycopy(a, 0, elementData, size, numNew);// 复制数组完成复制
    size += numNew;
    return numNew != 0;
}
1
2
3
4
5
6
7
8
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);//越界判断

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount //确认是否需要扩容
    
    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);//移动（复制)数组
    
    System.arraycopy(a, 0, elementData, index, numNew);//复制数组完成批量赋值
    size += numNew;
    return numNew != 0;
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
总结：
add、addAll。
先判断是否越界，是否需要扩容。
如果扩容， 就复制数组。
然后设置对应下标元素值。

值得注意的是：
1 如果需要扩容的话，默认扩容一半。如果扩容一半不够，就用目标的size作为扩容后的容量。
2 在扩容成功后，会修改modCount

2 删
public E remove(int index) {
    rangeCheck(index);//判断是否越界
    modCount++;//修改modeCount 因为结构改变了
    E oldValue = elementData(index);//读出要删除的值
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);//用复制 覆盖数组数据
    elementData[--size] = null; // clear to let GC do its work  //置空原尾部数据 不再强引用， 可以GC掉
    return oldValue;
}
    //根据下标从数组取值 并强转
    E elementData(int index) {
        return (E) elementData[index];
    }

//删除该元素在数组中第一次出现的位置上的数据。 如果有该元素返回true，如果false。
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);//根据index删除元素
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
//不会越界 不用判断 ，也不需要取出该元素。
private void fastRemove(int index) {
    modCount++;//修改modCount
    int numMoved = size - index - 1;//计算要移动的元素数量
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);//以复制覆盖元素 完成删除
    elementData[--size] = null; // clear to let GC do its work  //置空 不再强引用
}

//批量删除
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);//判空
    return batchRemove(c, false);
}
//批量移动
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;//w 代表批量删除后 数组还剩多少元素
    boolean modified = false;
    try {
        //高效的保存两个集合公有元素的算法
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement) // 如果 c里不包含当前下标元素， 
                elementData[w++] = elementData[r];//则保留
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) { //出现异常会导致 r !=size , 则将出现异常处后面的数据全部复制覆盖到数组里。
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;//修改 w数量
        }
        if (w != size) {//置空数组后面的元素
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;//修改modCount
            size = w;// 修改size
            modified = true;
        }
    }
    return modified;
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
从这里我们也可以看出，当用来作为删除元素的集合里的元素多余被删除集合时，也没事，只会删除它们共同拥有的元素。

小结：
1 删除操作一定会修改modCount，且可能涉及到数组的复制，相对低效。
2 批量删除中，涉及高效的保存两个集合公有元素的算法，可以留意一下。

3 改
不会修改modCount，相对增删是高效的操作。

public E set(int index, E element) {
    rangeCheck(index);//越界检查
    E oldValue = elementData(index); //取出元素 
    elementData[index] = element;//覆盖元素
    return oldValue;//返回元素
}
1
2
3
4
5
6
4 查
不会修改modCount，相对增删是高效的操作。

public E get(int index) {
    rangeCheck(index);//越界检查
    return elementData(index); //下标取数据
}
E elementData(int index) {
    return (E) elementData[index];
}
1
2
3
4
5
6
7
5 清空，clear
会修改modCount。

public void clear() {
    modCount++;//修改modCount
    // clear to let GC do its work
    for (int i = 0; i < size; i++)  //将所有元素置null
        elementData[i] = null;

    size = 0; //修改size 
}
1
2
3
4
5
6
7
8
6 包含 contain
//普通的for循环寻找值，只不过会根据目标对象是否为null分别循环查找。
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
//普通的for循环寻找值，只不过会根据目标对象是否为null分别循环查找。
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
7 判空 isEmpty()
public boolean isEmpty() {
    return size == 0;
}
1
2
3
8 迭代器 Iterator.
public Iterator<E> iterator() {
    return new Itr();
}
/**
 * An optimized version of AbstractList.Itr
 */
 private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return //默认是0
    int lastRet = -1; // index of last element returned; -1 if no such  //上一次返回的元素 (删除的标志位)
    int expectedModCount = modCount; //用于判断集合是否修改过结构的 标志

    public boolean hasNext() {
        return cursor != size;//游标是否移动至尾部
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)//判断是否越界
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)//再次判断是否越界，在 我们这里的操作时，有异步线程修改了List
            throw new ConcurrentModificationException();
        cursor = i + 1;//游标+1
        return (E) elementData[lastRet = i];//返回元素 ，并设置上一次返回的元素的下标
    }

    public void remove() {//remove 掉 上一次next的元素
        if (lastRet < 0)//先判断是否next过
            throw new IllegalStateException();
        checkForComodification();//判断是否修改过

        try {
            ArrayList.this.remove(lastRet);//删除元素 remove方法内会修改 modCount 所以后面要更新Iterator里的这个标志值
            cursor = lastRet; //要删除的游标
            lastRet = -1; //不能重复删除 所以修改删除的标志位
            expectedModCount = modCount;//更新 判断集合是否修改的标志，
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
 //判断是否修改过了List的结构，如果有修改，抛出异常
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
 }
 1
 2
 3
 4
 5
 6
 7
 8
 9
 10
 11
 12
 13
 14
 15
 16
 17
 18
 19
 20
 21
 22
 23
 24
 25
 26
 27
 28
 29
 30
 31
 32
 33
 34
 35
 36
 37
 38
 39
 40
 41
 42
 43
 44
 45
 46
 47
 48
 总结
 增删改查中， 增导致扩容，则会修改modCount，删一定会修改。 改和查一定不会修改modCount。
 扩容操作会导致数组复制，批量删除会导致 找出两个集合的交集，以及数组复制操作，因此，增、删都相对低效。 而 改、查都是很高效的操作。
 因此，结合特点，在使用中，以Android中最常用的展示列表为例，列表滑动时需要展示每一个Item（element）的数组，所以 查 操作是最高频的。相对来说，增操作 只有在列表加载更多时才会用到 ，而且是在列表尾部插入，所以也不需要移动数据的操作。而删操作则更低频。 故选用ArrayList作为保存数据的结构。
 在面试中还有可能会问到和Vector的区别，我大致看了一下Vector的源码，内部也是数组做的，区别在于Vector在API上都加了synchronized所以它是线程安全的，以及Vector扩容时，是翻倍size，而ArrayList是扩容50%。
 ————————————————
 版权声明：本文为CSDN博主「张旭童」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
 原文链接：https://blog.csdn.net/zxt0601/article/details/77281231