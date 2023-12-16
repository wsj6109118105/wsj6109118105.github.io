---
title: HashMap
date: 2021-08-25 16:24:51
tags: 数据结构
---
# B-树，B+树
首先来介绍B-树与B+树。
## B-树
1970年，R.Bayer和E.mccreight提出了一种适用于外查找的树，它是一种平衡的多叉树，称为B树（或B-树、B_树）。
**m阶的B树必须满足以下条件**
- 每个节点最多只有m个子节点。
- 每个非叶子节点（除了根）具有至少⌈ m/2⌉子节点。
- 如果根不是叶节点，则根至少有两个子节点。
- 具有k个子节点的非叶节点包含k -1个键。
- 所有叶子都出现在同一水平，没有任何信息（高度一致）。
>B树的阶

B树中一个节点的子节点数目的最大值，即为B树的阶。
![](https://note.youdao.com/yws/api/personal/file/WEBf22f1cb5061a690642e4ed9132946430?method=download&shareKey=dad5244c7980e03f8dff86749c4d99af)
图中，结点[13,16,19]拥有的子节点数最多，有4个，所以是一棵4阶B树。
>根节点

节点[10]即为根节点，根结点的儿子数为[2, m]，包含的元素数量为[1,m-1]
>内部节点

除了根节点与叶子节点，其他节点均为内部节点拥有父节点和子节点，根节点的儿子数为[m/2, m],元素数量为[m/2-1,m-1],m/2向上取整

>叶子节点

节点[11,12],[14,15]等最后一层的都是叶子节点，元素数量为[m/2-1,m-1],m/2向上取整  
**插入**  
对于m阶的B数，插入一个元素时，首先在B树中是否存在，如果不存在，会在叶子节点处结束，然后再叶子节点处插入。
- 如果该节点元素的数量小于m-1,直接插入。
- 如果该节点元素的数量等于m-1,则以中间元素分裂，上移至父节点。
- 重复第二步的动作，直到所有节点都符合B-树规则，最坏的情况一直分裂，生成新的根节点，高度加1。  

**删除**(参考http://xianzilei.cn/blog/31)  
- 1）如果当前需要删除的key位于非叶子节点上，则用后继key（这里的后继key均指后继记录的意思）覆盖要删除的key，然后在后继key所在的子支中删除该后继key。此时后继key一定位于叶子节点上，这个过程和二叉搜索树删除节点的方式类似。删除这个记录后执行第2步
- 2）该节点key个数大于等于(m/2)-1，结束删除操作，否则执行第3步。
- 3）如果兄弟节点key个数大于(m/2)-1，则父节点中的key下移到该节点，兄弟节点中的一个key上移，删除操作结束。
- 4）否则，将父节点中的key下移与当前节点及它的兄弟节点中的key合并，形成一个新的节点。原父节点中的key的两个孩子指针就变成了一个孩子指针，指向这个新节点。然后当前节点的指针指向父节点，重复上第2步。（有些节点它可能即有左兄弟，又有右兄弟，那么我们任意选择一个兄弟节点进行操作即可）

一棵含有N个总关键字数的m阶的B树的最大高度是多少？

log（m/2）(N+1)/2 + 1  ，log以（m/2）为低，(N+1)/2的对数再加1
## B+树
B+树其实是B树的一种变体。
B+树的特征：

- 有m个子树的中间节点包含有m个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引；
- 所有的叶子结点中包含了全部关键字的信息，及指向含有这些关键字记录的指针，且叶子结点本身依关键字的大小自小而大的顺序链接。 (而B 树的叶子节点并没有包括全部需要查找的信息)；
- 所有的非终端结点可以看成是索引部分，结点中仅含有其子树根结点中最大（或最小）关键字。 (而B 树的非终节点也包含需要查找的有效信息)；

为什么说B+树比B树更适合数据库索引？  
1）B+树的磁盘读写代价更低

　　B+树的内部结点并没有指向关键字具体信息的指针。因此其内部结点相对B 树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说IO读写次数也就降低了；

2）B+树查询效率更加稳定

　　由于非终结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当；

3）B+树便于范围查询（最重要的原因，范围查找是数据库的常态）

　　B树在提高了IO性能的同时并没有解决元素遍历的我效率低下的问题，正是为了解决这个问题，B+树应用而生。B+树只需要去遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而B树不支持这样的操作或者说效率太低；  
**B+树：**
![](https://note.youdao.com/yws/api/personal/file/WEB5451482d05f869e84f6cb92ebe55ef71?method=download&shareKey=83392c9a0e73bfb22ce228bc7941b493)
# 红黑树
## 红黑树的定义和性质
红黑树是一种含有红黑结点并能自平衡的二叉查找树。它必须满足下面性质：
- 性质1：每个节点要么是黑色，要么是红色。
- 性质2：根节点是黑色。
- 性质3：每个叶子节点（NIL）是黑色。（在Java中，叶子结点是为null的结点。）
- 性质4：每个红色结点的两个子结点一定都是黑色。
- 性质5：任意一结点到每个叶子结点的路径都包含数量相同的黑结点。
![红黑树](https://note.youdao.com/yws/api/personal/file/WEBeaaa1ac52bc5fad3104072cc6043daba?method=download&shareKey=91e95d61c9423af61807d0569af6b165)

**注意图中的黑色方格代表值为空的叶子节点**  
红黑树并不是一棵完美平衡二叉树，但是左右子树的黑色节点的层数是相同的，即任意一个节点到每个叶子节点所经过的黑色节点的数量是相同的，这种平衡被称为黑色完美平衡。

红黑树有三种操作：左旋，右旋，变色。正是通过这三种操作实现了自平衡。
- 左旋：以某个结点作为支点(旋转结点)，其右子结点变为旋转结点的父结点，右子结点的左子结点变为旋转结点的右子结点，左子结点保持不变。
- 右旋：以某个结点作为支点(旋转结点)，其左子结点变为旋转结点的父结点，左子结点的右子结点变为旋转结点的左子结点，右子结点保持不变。
- 变色：结点的颜色由红变黑或由黑变红。

![左旋](https://note.youdao.com/yws/api/personal/file/WEB41ccdf109878f374fb9ad331fd9b2487?method=download&shareKey=c94ce0910416847be42ee97ed4f40e04)
![右旋](https://note.youdao.com/yws/api/personal/file/WEB9078573835d74117eb4bd093b75d8b3e?method=download&shareKey=5a1d1f994a9a84c9b0ff8faad7872767)

### 红黑树的查找
因为红黑树是一颗二叉平衡树，并且查找不会破坏树的平衡，所以查找跟二叉平衡树的查找无异：

- 1.从根结点开始查找，把根结点设置为当前结点；
- 2.若当前结点为空，返回null；
- 3.若当前结点不为空，用当前结点的key跟查找key作比较；
- 4.若当前结点key等于查找key，那么该key就是查找目标，返回当前结点；
- 5.若当前结点key大于查找key，把当前结点的左子结点设置为当前结点，重复步骤2；
- 6.若当前结点key小于查找key，把当前结点的右子结点设置为当前结点，重复步骤2；

### 红黑树的插入
插入的节点再插入时会被染成红色，因为这样不会破坏性质五

**情况一**：红黑树为空树，直接插入到根节点，结束

**情况二**：插入的key已经存在，把要插入的节点设置为将要代替的节点的颜色，将节点更新

**情况三**：插入节点的父节点为黑色，这种情况不会影响平衡，直接插入。

**情况四**：插入节点的父节点为红色，且叔叔节点存在为红色，那么将祖父节点变为红色，父节点，叔叔节点变为黑色，这是因为祖父节点变为红色，可能依然破坏了平衡，所以可能需要接着平衡，知道平衡为止。

**情况五**：插入节点为父节点的左子节点且父节点为红色，且叔叔节点不存在或为黑色，将祖父节点染为红色，父节点染为黑色，绕祖父节点右旋。
![](https://note.youdao.com/yws/api/personal/file/WEB191aaf2581a7120fd64dc98f63ae0a7a?method=download&shareKey=db17dbe15fbdd2f400d6525a1424f3ec)

**情况六**：插入节点为父节点的右子节点且父节点为红色，且叔叔节点不存在或为黑色，先绕父节点进行左旋，然后变为情况五。。
![](https://note.youdao.com/yws/api/personal/file/WEBb395efd39768666d42f5920b57e33967?method=download&shareKey=29543c51192574f4a92bc9a26ec1479b)

**情况七**：为情况五的对称情况，与其进行对应的反操作即可
![](https://note.youdao.com/yws/api/personal/file/WEB3c87725fb5fd228924b4a1628338928e?method=download&shareKey=60c61b8d52b31f0b81ffda6fd134fb2c)
**情况八**：为情况六的对此情况，与其进行对应的反操作即可
![](https://note.youdao.com/yws/api/personal/file/WEBcedb8a70eaee0340a81ec56577e29e7e?method=download&shareKey=71f4abb15c3e2fee24088aa7aa6484b7)

### 红黑树删除
红黑树的删除操作也包括两部分工作：一查找目标结点；二删除后自平衡。当不存在目标结点时，忽略本次操作；当存在目标结点时，删除后就得做自平衡处理了。删除了结点后我们还需要找结点来替代删除结点的位置，不然子树跟父辈结点断开了，除非删除结点刚好没子结点，那么就不需要替代。

二叉树删除结点找替代结点有3种情情景：

- 情景1：若删除结点无子结点，直接删除
- 情景2：若删除结点只有一个子结点，用子结点替换删除结点
- 情景3：若删除结点有两个子结点，用后继结点（大于删除结点的最小结点）替换删除结点(也可以找小于删除节点的最大节点)

**如果直接删除节点，那么将会破坏树，修复十分麻烦，但是如果将需要删除的节点用需要替换的节点替换，然后删除替换的节点，则非常容易。**

## HashMap原理分析

>实现原理

采用数组加链表来实现对数据的存储  
HashMap采⽤Entry数组来存储key-value对，每⼀个键值对组成了⼀个Entry实体，Entry类实际上是⼀个单向的链表结 构，它具有Next指针，可以连接下⼀个Entry实体。 只是在JDK1.8中，链表⻓度⼤于8的时候，链表会转成红⿊树！

### 对key进行hash
```java
static final int hash(Object key) {
    int h;
    //将生成的hashCode的低16位与高16位进行异或
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
### put操作
```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
        //HashMap懒加载策略，在进行put操作时才检测table数组的初始化
            n = (tab = resize()).length;
        //采用长度减1，然后与得到的key的hash值相与可以得到0-tab.length-1的所有值，可以将tab全部取到
        if ((p = tab[i = (n - 1) & hash]) == null)
        //如果位置刚好没有，则将put的值放入tab[i]
            tab[i] = newNode(hash, key, value, null);
        else {
        //此时p中存储这tab[i]中的node
            Node<K,V> e; K k;
            //p的hash与要填入的hash相等，且key也相等，判断是否时相同key不同value
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)   //如果p为TreeNode,则调用putTreeVal
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //遍历链表，与输入的key进行比较
                for (int binCount = 0; ; ++binCount) {
                    //如果p的下一个为空，那么将其插入之后
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //如果长度超过TREEIFY_THRESHOLD则转化为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //判断是否与p之后的某个节点为相同key不同value,是则退出
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        //如果找到相同的key则退出
                        break;
                    //将p向后指
                    p = e;
                }
            }
            //走完上面的代码，如果e为空，则说明是一个新节点，并且已经插入，如果e不为空则说明输入的key已经存在，需要更新value，此时e中存储的为已存在的节点
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //长度增加进行扩容检测
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
### resize扩容
```java
    final Node<K,V>[] resize() {
        //将老table取别名
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        //如果数组容量大于0
        if (oldCap > 0) {
            //如果老的数组容量大于最大值
            if (oldCap >= MAXIMUM_CAPACITY) {
                //直接将阈值设置为Int最大值
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //如果老数组的容量大于默认的值，并且将老数组容量左移一位之后小于最大容量，那么将这个容量赋给newThr
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        //如果数组容量为0，但阈值不为0，直接将阈值设置为新容量
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        //表示初始化的时候，使用默认值，容量为默认容量，阈值为默认容量*默认负载因子
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //如果新阈值为0，则用新容量*负载因子来计算新阈值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //开始对新的hash表进行操作
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            //如果老的hash表不为空，遍历老的表，移入新的表中
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    //判断当前只有一个元素，将其的hash与新的容量重新散列，放入新表中
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    //如果是一个红黑树，则新的表也需要为一个树结构
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            //因为cap为2的m次幂，所以m+1位为1，其余位位0，如果与hash相与为0，则说明hash的m+1位为0，
                            //那么hash在与newcap-1相与时结果与hash在与oldcap-1相与时结果一致
                            /*
                            eg: oldcap = 10000  newcap = 100000  hash = xxxxx
                            oldcap&hash = 0 则  hash = 0xxxx
                            则hash&oldcap-1为  0xxxx &  01111 = xxxx
                            hash&newcap-1 为   0xxxx & 011111 = xxxx
                            */
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            /*
                            同理若相与不为0，则说明hash最高位为1   hash = 1xxxx
                            则两个相与结果如下：
                            hash&oldcap-1为 1xxxx &  01111 = xxxx
                            hash&newcap-1为 1xxxx & 011111 = 1xxxx = xxxx + 10000(oldcap)
                            */
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        /*
                        通过上面分析可知节点的位置要么与原来相同，要么为原来位置加上oldcap,
                        为哪种则取决于  hash&oldcap 为0还是不为0
                        */
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                        //至此将old表中的节点全部移入new表中
                    }
                }
            }
        }
        return newTab;
    }
```
### get操作
```java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```
```java
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //判断table不为空,且长度大于0,key对应的节点不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //当前第一个节点的hash于key所对应的hash一致，且key相同，那么返回这个节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //否则继续向后查找
            if ((e = first.next) != null) {
                //如果对应为数则进行树的查找
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //链表进行循环查找
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        //如果找不到返回空
        return null;
    }
```
### containskey方法

根据get来查找返回
```java
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
```
