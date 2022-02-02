## 1.说说 Java 的集合类吧？

这种问题一般大致提一下，然后等着面试官深挖。

常用的集合有 List、Set、Map、Queue 等。

List 常见实现类有 ArrayList 和 LinkedList。

- ArrayList 基于动态数组实现，支持下标随机访问，对删除不友好。

- LinkedList 基于双向链表实现，不支持随机访问，只能顺序遍历，但是支持O(1)插入和删除元素。


Set 常见实现类有：HashSet、TreeSet、LinkedHashSet。

- HashSet 其实就是 HashMap 包了层马甲，支持 O(1)查询，无序。

- TreeSet 基于红黑树实现，支持范围查询，不过基于红黑树的查找时间复杂度是O(lgn)，有序。

- LinkedHashSet，比 HashSet 多了个双向链表，通过链表保证有序。


Map 常见实现类有：HashMap、TreeMap、LinkedHashMap

- HashMap：基于哈希表实现，支持 O(1) 查询，无序。

- TreeMap：基于红黑树实现，O(lgn)查询，有序。

- LinkedHashMap：同样也是多了双向链表，支持有序，可以很好的支持 lru 的实现。


设置有序，并且重写LinkedHashMap中的 removeEldestEntry 方法，即可实现 lru。

> 这里有一点要提一下，如果你对某个东西比较熟悉就要在合适的地方抛出来。比如通过 LinkedHashMap 你还能延伸到 lru ，这表明你对 LinkedHashMap 有研究并且也知晓 lru，面试官自己可能都不清楚，会觉得你有点东西。

> 而且面试官基本会追问  lru 然后接着延伸，比如延伸到改进的 lru ，mysql 缓存中的 lru 等等，这就是通过你的引导把问题领域迁移到你自身熟悉的地方，这岂不美哉？如果你不熟悉，那少 bb。

Queue 常见的实现类有：LinkedList、PriorityQueue。

PriorityQueue：优先队列，是基于堆实现的，底层其实就是数组。

基本上回答不了这么全，稍微讲几个可能就被打断，然后深挖了，届时只能见招拆招。

## 2.如果让你设计一个 HashMap 如何设计？

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



---

TBC。这部分之前都没写，我过几天补上，很快的。

有更多相关的Java集合面试题，可以提PR哈，有错误欢迎联系我。

除了这个系列，我的公众号每周至少都会有一篇原创，欢迎关注~

![](https://gitee.com/yessimida/interview-of-legends/raw/master/pic/16034279-e6ebb79b5a0b8fe7.png)

最近已经汇集了近 500 名朋友交流各大小厂面试真题，也期待各位的面试题分享，公众号有我的联系方式，如果有兴趣可以加我备注 **面霸**，拉你进真题交流群。

![](https://cdn.jsdelivr.net/gh/yessimida/cdn_image/img/image-20210228190741512.png)