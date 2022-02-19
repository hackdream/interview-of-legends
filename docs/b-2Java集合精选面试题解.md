# Java的集合类

这题一般用在开头，让你热热身，你也不需要说的那么详细，大致讲下，面试官会根据你回答的点继续挖的。

Java集合从分类上看，有 collection 和 map 两种，前者是存储对象的集合类，后者存储的是键值对（key-value）

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202800.png)

## Collection
### **Set**

 主要功能是保证存储的集合不会重复，至于集体是有序还是无序的，需要看具体的实现类，比如 TreeSet 就是有序的，HashSet 是无序的（所以网上有些说 set 是无序集合是在扯淡）

###  **List**

 这个很熟悉，具体的实现类有 ArrayList 和 LinkedList，两者的区别在于底层实现不同，前者是数组，后者是双向链表，所以引申出来就是将数组和链表的区别。

####  数组 VS 链表
**数组**的内存是连续的，且存储的元素大小是固定的，实现上是基于一个内存地址，然后由于元素固定大小，支持利用下标的直接访问。

具体是通过`下标 * 元素大小+内存基地址`算出一个访问地址，然后直接访问，所以随机访问的效率很高，O(1)。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202821.png)

而由于要保持内存连续这个特性，不能在内存中间空一块，所以删除中间元素时就需要搬迁元素，需进行内存拷贝，所以说删除的效率不高。

**链表**的内存不需要连续，它们是通过指针相连，这样对内存的要求没那么高（数组的申请需要一块**连续**的内存），链表就可以**散装**内存，不过链表需要额外存储指针，所以总体来说，链表的占用内存会大一些。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202832.png)


且由于是指针相连，所以直接无法随机访问一个元素，必须从头（如双向链表，可尾部）开始遍历，所以随机查找的效率不高，O(n)。

也由于指针相连这个特性，单方面删除的效率高，因为只需要改变指针即可，没有额外的内存拷贝动作(但是要找到这个元素，费劲儿呀，除非你顺序遍历删)。

> 两者大致的特点就如上所说，再扯地深一点，就要说到 CPU 亲和性问题

各位应该都听过空间局部性。

空间局部性（spatial locality）：如果一个存储器的位置被引用，那么将来它附近的位置也会被引用。

根据这个原理，就会有预读功能，像 CPU 缓存就会读连续的内存，这样一来如果你本就要遍历数组的，那么你后面的数据就已经被上一次读取前面数据的时候，一块被加载了，这样就是 CPU 亲和性高。

反观链表，由于内存不连续，所以预读不到，所以 CPU 亲和性低。

对了，链表（数组）加了点约束的话，还可以用作栈、队列和双向队列。

像 LinkedList 就可以用来作为栈或者队列使用。

### **queue**

队列，有序，严格遵守先进先出，就像往常的排队，没啥别的好说的。

常用的实现类就是 LinkedList，没错这玩意还实现了 Queue 接口。

还有一个值得一提的是优先队列，即 PriorityQueue，内部是基于数组构建的，用法就是你自定义一个 comparator ，自己定义对比规则，这个队列就是按这个规则来排列出队的优先级。

## Map
存储的是键值对，也就是给对象（value）搞了一个 key，这样通过 key 可以找到那个 value。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202845.png)


最出名的，平日里使用最多的应该就是 HashMap，这个是无序的。

还有两个实现类，LinkedHashMap 和 TreeMap，前者里面搞了个链表，这样塞入顺序就被保存下来了，后者是红黑树实现了，所以有序。

最后还有个 IdentityHashMap ，这个好像网上文章都提的比较少，不过我们也来盘一下，有备无患。

### HashMap
这玩意是面试高频点，可以说几乎被问烂了... 有的题目还很刁难，比如问默认初始容量（16）是多少，哈希函数怎么设计的..没点准备肯定蒙，所以我们来个一网打尽。


### 能说下 HashMap 的实现原理吗

其实就是有个 Entry 数组，Entry 保存了 key 和 value。当你要塞入一个键值对的时候，会根据一个 hash 算法计算 key 的 hash 值，然后通过数组大小 n-1 & hash 值之后，得到一个数组的下标，然后往那个位置塞入这个 Entry。

然后我们知道，hash 算法是可能产生冲突的，且数组的大小是有限的，所以很可能通过不同的 key 计算得到一样的下标，因此为了解决 Entry 冲突的问题，采用了链表法，如下图所示：

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202856.png)

在 JDK1.7 及之前链表的插入采用的是头插法，即在链表的头部插入新的 Entry。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202908.png)



在 JDK1.8 的时候，改成了尾插法，并且引入了红黑树。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202916.png)




当链表的长度大于 8 且数组大小大于等于 64 的时候，就把链表转化成红黑树，当红黑树节点小于 6 的时候，又会退化成链表。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202925.png)


### 为什么 JDK 1.8 要对 HashMap 做红黑树这个改动？

主要是避免 hash 冲突导致链表的长度过长，这样 get 的时候时间复杂度严格来说就不是 O(1) 了，因为可能需要遍历链表来查找命中的 Entry。

#### 为什么定义链表长度为 8 且数组大小大于等于 64 才转红黑树？不要链表直接用红黑树不就得了吗？
![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202947.png)

因为**红黑树节点的大小是普通节点大小的两倍**，所以为了节省内存空间不会直接只用红黑树，只有当节点到达一定数量才会转成红黑树，这里定义的是 8。

为什么是 8 呢？这个其实 HashMap 注释上也有说的，和泊松分布有关系，这个大学应该都学过。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219202956.png)

简单翻译下就是在默认阈值是 0.75 的情况下，冲突节点长度为 8 的概率为 0.00000006，也就概率比较小（毕竟红黑树耗内存，且链表长度短点时遍历的还是很快的）。

这就是基于时间和空间的平衡了，红黑树占用内存大，所以节点少就不用红黑树，如果万一真的冲突很多，就用红黑树，选个参数为 8 的大小，就是为了平衡时间和空间的问题。

#### 为什么节点少于 6 要从红黑树转成链表？
也是为了平衡时间和空间，节点太少链表遍历也很快，没必要成红黑树，变成链表节约内存。

为什么定了 6 而不是小于等于 8 就变？

是因为要**留个缓冲余地**，避免反复横跳。举个例子，一个节点反复添加，从 8 变成 9 ，链表变红黑树，又删了，从 9 变成 8，又从红黑树变链表，再添加，又从链表变红黑树？

所以**余**一点 ，毕竟树化和反树化都是有开销的。

### 那 JDK 1.8 对 HashMap 除了红黑树这个改动，还有哪些改动？
1. hash 函数的优化
2. 扩容 rehash 的优化
3. 头插法和尾插法
4. 插入与扩容时机的变更

#### hash 函数的优化

1.7是这样实现的：
```java
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

而 1.8 是这样实现的：
```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
具体而言就是 1.7 的操作太多了，经历了四次异或，所以 1.8 优化了下，它将 key 的哈希码的高16位和低16位进行了异或，得到的 hash 值同时拥有了高位和低位的特性，这样做得出的码比较均匀，不容易冲突。

这也是 JDK 开发者根据速度、实用性、哈希质量所做的权衡来做的实现：

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203008.png)
#### 扩容 rehash 的优化
按照我们的思维，正常扩容肯定是先申请一个更大的数组，然后将原数组里面的每一个元素重新 hash 判断在新数组的位置，然后一个一个搬迁过去。

在 1.7 的时候就是这样实现的，然而 1.8 在这里做了优化，**关键点就在于数组的长度是 2 的次方，且扩容为 2 倍**。

因为数组的长度是 2 的 n 次方，所以假设以前的数组长度（16）二进制表示是 010000，那么新数组的长度（32）二进制表示是 100000，这个应该很好理解吧？

它们之间的差别就在于高位多了一个 1，而我们通过 key 的 hash 值定位其在数组位置所采用的方法是 `(数组长度-1) & hash`。我们还是拿 16 和 32 长度来举例：

16-1=15，二进制为 001111

32-1=31，二进制为 011111

所以重点就在 key 的 hash 值的从右往左数第五位是否是 1，如果是 1 说明需要搬迁到新位置，且新位置的下标就是原下标+16（原数组大小），如果是 0 说明吃不到新数组长度的高位，那就还是在原位置，不需要迁移。

所以，我们刚好拿老数组的长度（010000）来判断高位是否是 1，这里只有两种情况，要么是 1 要么是 0  。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203020.png)

从上面的源码可以看到，链表的数据是一次性计算完，然后一堆搬运的，因为扩容时候，节点的下标变化只会是原位置，或者原位置+老数组长度，不会有第三种选择。

上面的位操作，包括为什么是原下标+老数组长度等，如果你不理解的话，可以举几个数带进去算一算，就能理解了。

总结一下，1.8 的扩容不需要每个节点重写 hash 算下标，而是通过和老数组长度的**&**计算是否为 0 ，来判断新下标的位置。

额外再补充一个问题：**为什么 HashMap 的长度一定要是 2 的 n 次幂**？

原因就在于数组下标的计算，由于下标的计算公式用的是 `i = (n - 1) & hash`，即位运算，一般我们能想到的是 %（取余）计算，但相比于位运算而言，效率比较低，所以推荐用位运算，而要满足上面这个公式，n 的大小就必须是 2 的 n 次幂。

即：**当 b 等于 2 的 n 次幂时，a % b 操作等于 a & ( b - 1 )**

#### 头插法和尾插法
1.7是头插法，上面的图已经展示了。

头插法的好处就是插入的时候不需要遍历链表，直接替换成头结点，但是缺点是扩容的时候会逆序，而逆序在多线程操作下可能会出现环，然后就死循环了。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203032.png)

然后 1.8 是尾插法，每次都从尾部插入的话，扩容后链表的顺序还是和之前一致，所以不可能出现多线程扩容成环的情况。

其实我在网上找了找，很多文章说尾插法的优化就是避免多线程操作成环的问题，我表示怀疑。因为 HashMap 本就不是线程安全的，我要还优化你多线程的情况？我觉得开发者应该不会做这样的优化。

那为什么要变成尾插法呢？我也没找到官方解答，如果有谁知道可以教我一下。

那再延伸一下，**改成尾插法之后 HashMap 就不会死循环了吗**？

好像还是会，这次是红黑树的问题 ，我在网上看到这篇文章，有兴趣的可以深入了解下

- *https://blog.csdn.net/qq_33330687/article/details/101479385*

#### 插入与扩容时机的变更  

1.7 是先判断 put 的键值对是新增还是替换，如果是替换则直接替换，如果是新增会判断当前元素数量是否大于等于阈值，如果超过阈值且命中数组索引的位置已经有元素了，那么就进行扩容。

```java
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }
    createEntry(...)
```

所以 1.7 是先扩容，然后再插入。

而 1.8 则是先插入，然后再判断 size 是否大于阈值，若大于则扩容。

就这么个差别，至于为什么，好吧，我查了下没查出来，我自己也不知道，我个人觉得两者没差。。可能是重构（引入红黑树）的时候改了下顺序而已...其实没什么影响，我自己也脑补不出什么别的了。

网上也没找到什么比较有信服力的答案，所以如果有谁知道可以再教我一下。

HashMap 大概就这么些个点了，建议看下源码，再巩固一下。

### LinkedHashMap

LinkedHashMap 的父类是 HashMap，所以 HashMap 有的它都有，然后基于 HashMap 做了一些扩展。

首先它把 HashMap 的 Entry 加了两个指针：before 和 after。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203048.png)

这目的已经很明显了，就是要把塞入的 Entry 之间进行关联，串成双向链表，如下图红色的就是新增的两个指针

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203058.png)


并且内部还有个  accessOrder 成员，默认是 false， 代表链表是顺序是按插入顺序来排的，如果是 true 则会根据访问顺序来进行调整，就是咱们熟知的 LRU 那种，如果哪个节点访问了，就把它移到最后，代表最近访问的节点。

具体实现其实就是 HashMap 埋了几个方法，然后 LinkedHashMap 实现了这几个方法做了操作，比如以下这三个，从方法名就能看出了：访问节点之后干啥；插入节点之后干啥；删除节点之后干啥。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203123.png)

举个 afterNodeInsertion 的例子，它埋在 HashMap 的 put 里，在塞入新节点之后，会调用这个方法

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203133.png)

然后 LinkedHashMap 实现了这个方法，可以看到**这个方法主要用来移除最老的节点**。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203142.png)

看到这你能想到啥？假如你想用 map 做个本地缓存，由于缓存的数量不可能无限大，所以你就能**继承** LinkedHashMap 来实现，当节点超过一定数量的时候，在插入新节点的同时，移除最老最久没有被访问的节点，这样就实现了一个 LRU 了。

具体做法是把 accessOrder 设置为  true，这样每次访问节点就会把刚访问的节点移动到尾部，然后再重写 removeEldestEntry 方法，LinkedHashMap 默认的实现是直接返回 true。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203154.png)

你可以搞个：

```java
 protected boolean removeEldestEntry(Entry<K, V> eldest) {
       return this.size() > this.maxCacheSize;
   }
```

这样就简单的实现一个 LRU 了！下面展示下完整的代码，非常简单：

```java
    private static final class LRUCache<K, V> extends LinkedHashMap<K, V> {
        private final int maxCacheSize;

        LRUCache(int initialCapacity, int maxCacheSize) {
            super(initialCapacity, 0.75F, true);
            this.maxCacheSize = maxCacheSize;
        }

        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return this.size() > this.maxCacheSize;
        }
    }
```
这里还能引申出一个笔试题，手写实现一个 LRU 算法，来我给你写！

```java
public class LRUCache<K,V> {
    class Node<K,V> {
        K key;
        V value;
        Node<K,V> prev, next;
        public Node(){}
        public Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
    private int capacity;
    private HashMap<K,Node> map;
    private Node<K,V> head;
    private Node<K,V> tail;
    public LRUCache(int capacity) {
        this.capacity = capacity;
        map = new HashMap<>(capacity);
        head = new Node<>();
        tail = new Node<>();
        head.next = tail;
        tail.prev = head;
    }

    public V get(K key) {
        Node<K,V> node = map.get(key);
        if (node == null) {
            return null;
        }
        moveNodeToHead(node);
        return node.value;
    }

    public void put(K key, V value) {
         Node<K,V> node = map.get(key);
       if (node == null) {
            if (map.size() >= capacity) {
                map.remove(tail.prev.key);
                removeTailNode();
            }
            Node<K,V> newNode = new Node<>(key, value);
            map.put(key, newNode);
            addToHead(newNode);
        } else {
            node.value = value;
            moveNodeToHead(node);
        }
    }

    private void addToHead(Node<K,V> newNode) {
        newNode.prev = head;
        newNode.next = head.next;
        head.next.prev = newNode;
        head.next = newNode;
    }

    private void moveNodeToHead(Node<K,V> node) {
        removeNode(node);
        addToHead(node);
    }

    private void removeNode(Node<K,V> node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void removeTailNode() {
        removeNode(tail.prev);
    }

    public static void main(String[] args) {
        LRUCache<Integer,Integer> lruCache = new LRUCache<>(3);
        lruCache.put(1,1);
        lruCache.put(2,2);
        lruCache.put(3,3);
        lruCache.get(1);
        lruCache.put(4,4);
        System.out.println(lruCache); // toString 我就没贴了，代码太长了
    }
}
```

### TreeMap

TreeMap 内部是通过红黑树实现的，可以让 key 的实现 Comparable 接口或者自定义实现一个 comparator 传入构造函数，这样塞入的节点就会根据你定义的规则进行排序。

这个用的比较少，我常用在跟加密有关的时候，有些加密需要根据字母序排，然后再拼接成字符串排序，在这个时候就可以把业务上的值统一都塞到 TreeMap 里维护，取出来就是有序的。

具体就不深入了，一般不会问太多。

### IdentityHashMap
理解这个 map 的关键就在于它的名字 Identity，也就是它判断是否相等的依据不是靠 equals ，而是对象本身是否是它自己。

什么意思呢？

首先看它覆盖的 hash 方法：

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203206.png)

可以看到，它用了个 `System.identityHashCode(x)`，而不是x.hashCode()。

而这个方法会返回原来默认的 hashCode 实现，不管对象是否重写了 hashCode 方法

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203221.png)

默认的实现返回的值是：对象的内存地址转化成整数**，是不是有点感觉了？

然后我们再看下它的 get 方法：

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203229.png)

可以看到，**它判断 key 是否相等并不靠 hash 值和 equals，而是直接用了 ==** 。

而 == 其实就是地址判断！

只有相同的对象进行 == 才会返回 true。

因此我们得知，**IdentityHashMap 的中的 key 只认它自己（对象本身）**。

即便你伪造个对象，就算值都相等也没用，put 进去 IdentityHashMap 只会多一个键值对，而不是替换，这就是 Identity 的含义。

比如以下代码，identityHashMap 会存在两个 Yes：

```java
Map<String, String> identityHashMap = new IdentityHashMap<>();
identityHashMap.put(new Yes("1"), "1");
identityHashMap.put(new Yes("1"), "2");
```

这里眼尖的小伙伴发现，为什么返回值是 `tab[i+1]`?

这是因为 IdentityHashMap 的存储方式有点不一样，它是将 value 存在 key 的后面。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203238.png)

认识到这就差不多了，具体不深入了，有兴趣的小伙伴们自行研究~

# 如果让你设计一个 HashMap 如何设计？

这个问题我觉得可以从 HashMap 的一些关键点入手，例如 hash函数、如何处理冲突、如何扩容。

可以先说下你对 HashMap 的理解。

比如：HashMap 无非就是一个存储 <key,value> 格式的集合，用于通过 key 就能快速查找到 value。

基本原理就是将 key 经过 hash 函数进行散列得到散列值，然后通过散列值对数组取模找到对应的 index 。

所以 hash 函数很关键，不仅运算要快，还需要分布均匀，减少 hash 碰撞。

而因为输入值是无限的，而数组的大小是有限的所以肯定会有碰撞，因此可以采用拉链法来处理冲突。

为了避免恶意的 hash 攻击，当拉链超过一定长度之后可以转为红黑树结构。

当然超过一定的结点还是需要扩容的，不然碰撞就太严重了。

而普通的扩容会导致某次 put 延时较大，特别是 HashMap 存储的数据比较多的时候，所以可以考虑和 redis 那样搞两个 table 延迟移动，一次可以只移动一部分。

不过这样内存比较吃紧，所以也是看场景来 trade off 了。

不过最好使用之前预估准数据大小，避免频繁的扩容。

基本上这样答下来差不多了，HashMap 几个关键要素都包含了，接下来就看面试官怎么问了。

可能会延伸到线程安全之类的问题，反正就照着 currentHashMap 的设计答。

# ConcurrentHashMap 1.7

其实大体的哈希表实现跟 HashMap 没有本质的区别，都是经过 key 的 hash 定位到一个下标，然后获取元素，如果冲突了就用链表相连。

差别就在于引入了一个 Segments 数组，我们来看下大致的结构。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203435.png)


原理就是先通过 key 的 hash 判断得到 Segment 数组的下标，**然后将这个 Segment 上锁**，然后再次通过 key 的 hash 得到 Segment 里 HashEntry 数组的下标，下面这步其实就是 HashMap 一致了，所以我说差别就是引入了一个 Segments 数组。

因此可以简化的这样理解：每个 Segment 数组存放的就是一个单独的 HashMap。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203444.png)


可以看到，图上我们有 6 个 Segment ，那么等于有六把锁，因此共可以有六个线程同时操作这个 ConcurrentHashMap，并发度就是 6，相比于直接将 put 方法上锁，并发度就提高了，这就是**分段锁**。

具体上锁的方式来源于 Segment ，这个类实际继承了 ReentrantLock，因此它自身具备加锁的功能。

可以看出，1.7 的分段锁已经有了细化锁粒度的概念，它的一个缺陷是 Segment 数组一旦初始化了之后不会扩容，只有 HashEntry 数组会扩容，这就导致并发度过于死板，不能随着数据的增加而提高并发度。

# ConcurrentHashMap 1.8
1.8  ConcurrentHashMap 做了更细粒度的锁控制，可以理解为 1.8 HashMap 的数组的每个位置都是一把锁，这样扩容了锁也会变多，并发度也会增加。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203455.png)


思想的转变就是把粒度更加细化，我不分段了，我直接把 Node 数组的每个节点分别上一把锁，这样并发度不就更高了吗？

并且 1.8 也不借助于  ReentrantLock 了，直接用 synchronized，这也侧面证明，**都 1.8 了 synchronized 优化后的速度已经不下于 ReentrantLock 了**。

具体实现思路也简单，当塞入一个值的时候，先计算 key 的 hash 后的下标，如果计算到的下标还未有 Node ，那么就通过 cas 塞入新的 Node。如果已经有 node 则通过 synchronized 将这个 node 上锁，这样别的线程就无法访问这个 node 及其之后的所有节点。

然后判断 key 是否相等，相等则是替换 value ，反之则是新增一个 node，这个操作和 HashMap 是一样的。

## 1.8 的扩容也是有点意思的，它允许协助扩容，也就是**多线程扩容**。

当 put 的时候，发现当前 node hash 值是 -1，则表明当前的节点正在扩容，则会触发协助扩容机制

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203508.png)

这个要详细说起来还真的有点麻烦，而且不太好说清，代码有点大，我就说个大概。

其实大家大致理解下就够了：

扩容无非就是搬迁 Node，假设当前数组长度为 32，那就可以分着搬迁，31-16 这几个下标的 Node 都由线程A来搬迁，然后线程 B 来搬迁 15-0 这几个下标的 Node。

简单说就是会维护一个 transferIndex 变量，来的线程死循环 cas 争抢下标，如果下标已经分配完了，那自然就不需要搬迁了，如果 cas 抢到了要搬运的下标，那就去帮忙搬就好了，就是这么个事儿。

## 还有 1.8 的 size 方法和 1.7 也不一样

1.7 有个尝试的思想，当调用 size 方法的时候不会加锁，而是先尝试三次不加锁获取 sum。

如果三次总数一样，说明当前数量没有变化，那就直接返回了。如果总数不一样，那说明此时有线程在增删 map，于是加锁计算，这时候其他线程就操作不了 map 了。

```java
  if (retries++ == RETRIES_BEFORE_LOCK) {
       for (int j = 0; j < segments.length; ++j)
           ensureSegment(j).lock(); // force creation
   }
   ...再累加数量
```


而 1.8 不一样，它就是直接计算返回结果，具体采用的是类似 LongAdder 的思想，累加不再是基于一个成员变量，而是搞了一个数组，每个线程在自己对应的下标地方进行累加，等最后的时候把数组里面的数据统一一下，就是最终的值了。

所以这是一个分解的思想，分而治之。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203519.png)

在 put 的时候，就会通过 addCount 方法维护 counterCells 的数量，当然 remove 的时候也会调用此方法。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/20220219203527.png)

总而言之，就是平日的操作会维护 map 里面的节点数量，会先通过 CAS 修改 baseCount ，如果成功就直接返回，如果失败说明此时有线程在竞争，那么就通过 hash 选择一个 CounterCell 对象就行修改，最终 size 的结果就是  baseCount + 所有 CounterCell 。

这种通过 counterCell 数组来减少并发下场景下对单个成员变量的竞争压力，提高了并发度，提升了性能，这也就是 LongAdder  的思想。

这里还有个能问的就是 `@sun.misc.Contended` 注解了，这个和伪共享有关系，具体可以看看我之前写的这篇文章：

[填之前的坑，伪共享解析](https://mp.weixin.qq.com/s/x1jDDD967bvMWR2jYshBKw)

# ConcurrentHashMap#get需要加锁吗？

不需要加锁。

保证 put 的时候线程安全之后，get 的时候只需要**保证可见性即可**，而可见性不需要加锁。

具体是通过Unsafe#getXXXVolatile 和用  volatile 来修饰节点的 val 和 next 指针来实现的。

# 为什么 ConcurrentHashMap 不支持 key 或者 value 为 null ？

首先， key 为什么也不能为 null ？我不知道，没想明白，可能是作者 lea 佬不喜欢 null 值。

那 value 为什么不能为 null ？

因为在多线程情况下， null 值会**产生二义性**，因为你不清楚 map 里到底是不存在在这个 key ，还是说被put(key，null)。

这里可能有人会说，那 HashMap 不一样有这个问题？ HashMap 可以通过 containsKey 来判断是否存在这个 key，而多线程使用的 ConcurrentHashMap 就不能够。

比如你 get（key） 得到了null，此时map里面没有这个 key 的，但是你不知道，所以你想调用 containsKey 看看，而恰巧在你调用之前，别的线程 put 了这个 key ，这样你 containsKey 就发现有这个 key，这是不是就发生“误会”了。

---

TBC。

有更多相关的Java集合面试题，可以提PR哈，有错误欢迎联系我。

除了这个系列，我的公众号每周至少都会有一篇原创，欢迎关注~

![](https://gitee.com/yessimida/interview-of-legends/raw/master/pic/16034279-e6ebb79b5a0b8fe7.png)

最近已经汇集了近 500 名朋友交流各大小厂面试真题，也期待各位的面试题分享，公众号有我的联系方式，如果有兴趣可以加我备注 **面霸**，拉你进真题交流群。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/image-20210228190741512.png)