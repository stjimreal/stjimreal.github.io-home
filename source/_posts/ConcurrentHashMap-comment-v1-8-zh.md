---
title: ConcurrentHashMap_comment_v1.8_zh
date: 2021-12-04 20:41:27
tags: [data structure & algorithms, java, infrastructure]
---

# Java ConcurrentHashMap注释中文翻译(作者：Doug Lea)

> Doug Lea 主页传送门：http://gee.cs.oswego.edu/

## 生词

| 词条 | 释义 |
| - | - |
| retrieval | 索引 |
| interoperable | 互操作 |
| functional specification | 功能规范 |
| transient state| 瞬态 |
| sth. modulo numb. | 以 ... 为模 |
| bulk operations | 批量操作 |
| scalar | 标量 |
| contention | 争用 |

## 功能注释

A hash table supporting full concurrency of retrievals and high expected concurrency for updates. This class obeys the same functional specification as Hashtable, and includes versions of methods corresponding to each method of Hashtable. However, even though all operations are thread-safe, retrieval operations do not entail locking, and there is not any support for locking the entire table in a way that prevents all access. This class is fully interoperable with Hashtable in programs that rely on its thread safety but not on its synchronization details.  
ConcurrentHashMap是一个哈希表，能够支持检索的完全并发性和更新的高预期并发性。该类遵循与Hashtable相同的**功能规范**，包括与Hashtable的每个方法对应的方法版本。但是，即使所有操作都是线程安全的，检索操作也不需要锁定，并且不支持以防止所有访问的方式**锁定整个表**。在依赖其线程安全但不依赖于其同步细节的程序中，此类与 Hashtable 完全**可互操作**。  
Retrieval operations (including get) generally do not block, so may overlap with update operations (including put and remove). Retrievals reflect the results of the most recently completed update operations holding upon their onset. (More formally, an update operation for a given key bears a happens-before relation with any (non-null) retrieval for that key reporting the updated value.) For aggregate operations such as putAll and clear, concurrent retrievals may reflect insertion or removal of only some entries. Similarly, Iterators, Spliterators and Enumerations return elements reflecting the state of the hash table at some point at or since the creation of the iterator/enumeration. They do not throw ConcurrentModificationException. However, iterators are designed to be used by only one thread at a time. Bear in mind that the results of aggregate status methods including size, isEmpty, and containsValue are typically useful only when a map is not undergoing concurrent updates in other threads. Otherwise the results of these methods reflect transient states that may be adequate for monitoring or estimation purposes, but not for program control.  
检索操作（包括获取）一般不会阻塞，因此可能与更新操作（包括插入和删除）重叠。检索反映了最近完成的更新操作的结果。 （更正式地说，给定键的更新操作与报告更新值的该键的任何（非空）检索具有**happens-before**关系，注：满足JMM约束？。）对于诸如 putAll 和 clear 之类的聚合操作，并发检索可能只反映一些条目的插入或删除。类似地，迭代器、拆分器和枚举返回反映哈希表在迭代器/枚举**创建时**或**创建后的**某个时刻的状态的元素。它们不会抛出 ConcurrentModificationException。然而，迭代器被设计为**一次只能被一个线程**使用，注：过多地使用迭代器会导致线程频繁阻塞？。请记住，包括 size、isEmpty 和 containsValue 在内的聚合状态方法的结果通常仅在哈希表未在其他线程中进行并发更新时才发挥正常的函数效用。否则，这些方法的结果反映的瞬态状态可能足以用于监测或估计目的，但不适用于程序控制。  
The table is dynamically expanded when there are too many collisions (i.e., keys that have distinct hash codes but fall into the same slot modulo the table size), with the expected average effect of maintaining roughly two bins per mapping (corresponding to a 0.75 load factor threshold for resizing). There may be much variance around this average as mappings are added and removed, but overall, this maintains a commonly accepted time/space tradeoff for hash tables. However, resizing this or any other kind of hash table may be a relatively slow operation. When possible, it is a good idea to provide a size estimate as an optional initialCapacity constructor argument. An additional optional loadFactor constructor argument provides a further means of customizing initial table capacity by specifying the table density to be used in calculating the amount of space to allocate for the given number of elements. Also, for compatibility with previous versions of this class, constructors may optionally specify an expected concurrencyLevel as an additional hint for internal sizing. Note that using many keys with exactly the same hashCode() is a sure way to slow down performance of any hash table. To ameliorate impact, when keys are Comparable, this class may use comparison order among keys to help break ties.  
当有太多冲突（即具有不同散列码但落入同一个以表大小为模的键槽）时，该表动态扩展，预期平均效果是每个mapping（注：mapping就是一个key->value实体，bin就是实际内存中的结构）保持大约两个 bin（对应于 0.75 负载因子阈值用于调整大小，注：负载达到0.75的时候按照50%扩张，大小会来到1.5倍，0.75/1.5正好是每个mapping两个bin，见[stackoverflow讨论](https://stackoverflow.com/questions/26112957/why-would-loadfactor-be-0-75)）。随着映射的添加和删除，这个平均值可能会有很大差异，但总的来说，这保持了哈希表的普遍接受的时间/空间权衡。然而，调整这个或任何其他类型的哈希表的大小可能是一个相对较慢的操作。如果可能，最好使用的时候提供出估计的大小作为可选的构造函数参数`initialCapacity`。还有一个可选的构造函数参数 `loadFactor` 通过指定表密度与给定数量的元素计算来得到分配长度，这提供了一种自定义初始表容量的进一步的方法。此外，为了与此类的先前版本兼容，构造函数可以选择指定预期的 `concurrencyLevel` 作为内部大小的附加提示。请注意，使用许多具有完全相同 hashCode() 的键是一定会降低任何哈希表的性能的。为了减轻这个影响，当键满足 Comparable 接口时，这个类(ConcurrentHashMap)可以使用键之间的比较顺序来帮助打破联系。  
A Set projection of a ConcurrentHashMap may be created (using newKeySet() or newKeySet(int)), or viewed (using keySet(Object) when only keys are of interest, and the mapped values are (perhaps transiently) not used or all take the same mapping value.  
如果只对键值感兴趣，可以得到ConcurrentHashMap相应的 Set 投射（通过 `newKeySet()` 或者 `newKeySet(int)` 接口去创建，或者通过 keySet(Object)去查看），并且映射的值（或许暂时地）不会使用或者采用一样的值。  
A ConcurrentHashMap can be used as scalable frequency map (a form of histogram or multiset) by using java.util.concurrent.atomic.LongAdder values and initializing via computeIfAbsent. For example, to add a count to a ConcurrentHashMap<String,LongAdder> freqs, you can use freqs.computeIfAbsent(k -> new LongAdder()).increment();  
ConcurrentHashMap可以在Java中用作可扩展的频率表（一种直方图或者多值集合的形式），通过采用 `util.concurrent.atomic.LongAdder` 作为值，并且通过`computeIfAbsent`机制初始化。例如，如果要增加一个 `ConcurrentHashMap<String, LongAdder> freqs` 的计数，你可以用`freqs.computeIfAbsent(k -> new LongAdder()).increment();`  
This class and its views and iterators implement all of the optional methods of the Map and Iterator interfaces.  
这个类（注：ConcurrentHashMap）以及它的视图和迭代器实现了Map和迭代器的所有可选的接口。  
Like Hashtable but unlike HashMap, this class does not allow null to be used as a key or value.  
与 Hashtable 类似但与 HashMap 不同的是，此类不允许将 null 用作键或值。  
ConcurrentHashMaps support a set of sequential and parallel bulk operations that, unlike most Stream methods, are designed to be safely, and often sensibly, applied even with maps that are being concurrently updated by other threads; for example, when computing a snapshot summary of the values in a shared registry. There are three kinds of operation, each with four forms, accepting functions with Keys, Values, Entries, and (Key, Value) arguments and/or return values. Because the elements of a ConcurrentHashMap are not ordered in any particular way, and may be processed in different orders in different parallel executions, the correctness of supplied functions should not depend on any ordering, or on any other objects or values that may transiently change while computation is in progress; and except for forEach actions, should ideally be side-effect-free. Bulk operations on Map.Entry objects do not support method setValue.  
ConcurrentHashMaps 支持一组顺序和并行的批量操作，与大多数 Stream 方法不同，这些操作被设计为安全且通常明智地进行应用，即使哈希表正在由其他线程并发更新；例如，在计算共享注册表中值的快照和时。共有三种操作，每种有四种形式，接受带有键、值、条目（注：Entries）和（键，值）参数和/或返回值的函数。因为 ConcurrentHashMap 的元素没有以任何特定的方式排序，并且可能在不同的并行执行中以不同的顺序进行处理，所提供函数的正确性不应该依赖于任何排序，或者依赖于其他任何会在计算过程中暂时改变的对象或值；除了 forEach 操作，理想情况下应该是无副作用的。 `Map.Entry` 对象上的批量操作不支持 `setValue` 方法。  


forEach: Perform a given action on each element. A variant form applies a given transformation on each element before performing the action.  
+ `forEach`: 对每个元素执行给定的操作。变体形式在执行操作之前对每个元素应用给定的转换。  
search: Return the first available non-null result of applying a given function on each element; skipping further search when a result is found.  
+ `search`: 返回在每个元素上应用给定函数的第一个可用非空结果；找到结果时就略过进一步搜索。  
reduce: Accumulate each element. The supplied reduction function cannot rely on ordering (more formally, it should be both associative and commutative). There are five variants:  
+ `reduce`：累加每个元素。提供的归约函数不能依赖于排序（更正式地说，它应该是结合的和可交换的）。有五种变体：  
Plain reductions. (There is not a form of this method for (key, value) function arguments since there is no corresponding return type.)  
1. 普通reduce归约。 （因为没有相应的返回类型，所以 (key, value) 函数参数没有这种方法的形式。）  
Mapped reductions that accumulate the results of a given function applied to each element.  
2. 映射归约，累积应用于每个元素的给定函数的结果。  
Reductions to scalar doubles, longs, and ints, using a given basis value.  
3. 使用给定的基值归约到到标量双精度、长整数和整数。  


These bulk operations accept a parallelismThreshold argument. Methods proceed sequentially if the current map size is estimated to be less than the given threshold. Using a value of Long.MAX_VALUE suppresses all parallelism. Using a value of 1 results in maximal parallelism by partitioning into enough subtasks to fully utilize the ForkJoinPool.commonPool() that is used for all parallel computations. Normally, you would initially choose one of these extreme values, and then measure performance of using in-between values that trade off overhead versus throughput.  
这些批量操作接受一个 parallelismThreshold 参数。如果估计当前哈希表大小小于给定阈值，则方法按顺序进行。使用 Long.MAX_VALUE 值会抑制所有并行性。使用值 1 可通过划分为足够多的子任务以充分利用用于所有并行计算的 `ForkJoinPool.commonPool()` 来实现最大并行度。通常，您最初会选择这些极值之一，然后使用测量性能得到的在开销与吞吐量之间进行权衡的中间值。  
The concurrency properties of bulk operations follow from those of ConcurrentHashMap: Any non-null result returned from get(key) and related access methods bears a happens-before relation with the associated insertion or update. The result of any bulk operation reflects the composition of these per-element relations (but is not necessarily atomic with respect to the map as a whole unless it is somehow known to be quiescent). Conversely, because keys and values in the map are never null, null serves as a reliable atomic indicator of the current lack of any result. To maintain this property, null serves as an implicit basis for all non-scalar reduction operations. For the double, long, and int versions, the basis should be one that, when combined with any other value, returns that other value (more formally, it should be the identity element for the reduction). Most common reductions have these properties; for example, computing a sum with basis 0 or a minimum with basis MAX_VALUE.  
批量操作的并发属性遵循 ConcurrentHashMap 的并发属性：从 get(key) 和相关访问方法返回的任何非空结果都与相关的插入或更新具有`happens-before`关系。任何批量操作的结果都反映了这些每个元素关系的组成（但对于整个映射而言，不一定是原子的，除非以某种方式知道它是静止的）。相反，由于映射中的键和值永远不会为空，因此空可以作为当前缺少任何结果的可靠原子指示符。为了保持这个属性，null 作为所有非标量（注：非数值量）归约操作的隐式基础。对于 double、long 和 int 版本，基础应该是与任何其他值组合时返回该其他值的基础（更正式地说，它应该是归约的标识元素）。最常见的归约具有这些特性；例如，计算以 0 为基础的总和或以 MAX_VALUE 为基础的最小值。  
Search and transformation functions provided as arguments should similarly return null to indicate the lack of any result (in which case it is not used). In the case of mapped reductions, this also enables transformations to serve as filters, returning null (or, in the case of primitive specializations, the identity basis) if the element should not be combined. You can create compound transformations and filterings by composing them yourself under this "null means there is nothing there now" rule before using them in search or reduce operations.  
作为参数提供的搜索和转换函数应该类似地返回 null 以指示缺少任何结果（在这种情况下不使用它）。在“映射归约”的情况下，这也使转换能够充当过滤器，如果元素不应该被组合，则返回 null（或者，在原始特化的情况下，作为特定的基础）。在将它们用于搜索或归约操作之前，您可以通过在此“null 意味着现在没有任何东西”规则下自己组合它们来创建复合转换和过滤。  
Methods accepting and/or returning Entry arguments maintain key-value associations. They may be useful for example when finding the key for the greatest value. Note that "plain" Entry arguments can be supplied using new AbstractMap.SimpleEntry(k,v).  
接受和/或返回 Entry 参数的方法维护键值关联。例如，在查找最大值的键时，它们可能很有用。请注意，可以使用 `new AbstractMap.SimpleEntry(k,v)` 提供“普通” `Entry` 参数。  
Bulk operations may complete abruptly, throwing an exception encountered in the application of a supplied function. Bear in mind when handling such exceptions that other concurrently executing functions could also have thrown exceptions, or would have done so if the first exception had not occurred.  
批量操作可能会突然完成，抛出在提供的函数中遇到的异常。请记住，在处理此类异常时，其他并发执行的函数也可能抛出异常，或者在第一个异常没有发生的情况下抛出一个异常。  
Speedups for parallel compared to sequential forms are common but not guaranteed. Parallel operations involving brief functions on small maps may execute more slowly than sequential forms if the underlying work to parallelize the computation is more expensive than the computation itself. Similarly, parallelization may not lead to much actual parallelism if all processors are busy performing unrelated tasks.  
与顺序形式相比，并行通常都可以实现加速，但不能保证。如果并行化计算的基础工作比计算本身更昂贵，则涉及小映射上的简短函数的并行操作可能比顺序形式执行得更慢。 类似地，如果所有处理器都忙于执行不相关的任务，并行化可能不会带来太多实际的并行性。  
All arguments to all task methods must be non-null.  
所有任务方法的参数都必须为非空。  
This class is a member of the Java Collections Framework.  
这个类是 Java 集合框架的成员。  
Since:  
1.5  
Author:  
Doug Lea  
Type parameters:  
<K> – the type of keys maintained by this map  
<V> – the type of mapped values  


## 实现注释

Overview:

The primary design goal of this hash table is to maintain concurrent readability (typically method get(), but also iterators and related methods) while minimizing update contention. Secondary goals are to keep space consumption about the same or better than java.util.HashMap, and to support high initial insertion rates on an empty table by many threads.  

这个哈希表的主要设计目标是维护并发可读性（通常是 get() 方法，但也有迭代器和相关方法）同时最小化更新争用。次要目标是将空间消耗保持在
与 java.util.HashMap 相同或更好，并且支持多线程对空表的**高初始插入率**。

This map usually acts as a binned (bucketed) hash table.  Each
key-value mapping is held in a Node.  Most nodes are instances
of the basic Node class with hash, key, value, and next
fields. However, various subclasses exist: TreeNodes are
arranged in balanced trees, not lists.  TreeBins hold the roots
of sets of TreeNodes. ForwardingNodes are placed at the heads
of bins during resizing. ReservationNodes are used as
placeholders while establishing values in computeIfAbsent and
related methods.  The types TreeBin, ForwardingNode, and
ReservationNode do not hold normal user keys, values, or
hashes, and are readily distinguishable during search etc
because they have negative hash fields and null key and value
fields. (These special nodes are either uncommon or transient,
so the impact of carrying around some unused fields is
insignificant.)

该映射通常用作分箱（分桶）哈希表。每个键值映射保存在一个节点中。大多数节点是实例具有散列、键、值和下一个的基本 Node 类领域。但是，存在各种子类：TreeNodes 排列在平衡的树中，而不是列表中。 TreeBins 持有根的树节点集。 ForwardingNodes 被放置在头部调整大小期间的 bin 数量。 ReservationNodes 用作在 computeIfAbsent 中建立值时的占位符和相关方法。类型 TreeBin、ForwardingNode 和 ReservationNode 不持有普通的用户键、值或散列，并且在搜索等过程中很容易区分，因为它们有负散列字段和空键和值领域。 （这些特殊节点要么不常见，要么短暂，所以携带一些未使用的字段的影响是微不足道的。）

The table is lazily initialized to a power-of-two size upon the
first insertion.  Each bin in the table normally contains a
list of Nodes (most often, the list has only zero or one Node).
Table accesses require volatile/atomic reads, writes, and
CASes.  Because there is no other way to arrange this without
adding further indirections, we use intrinsics
(sun.misc.Unsafe) operations.


表被延迟初始化为 2 的幂大小
第一次插入。表中的每个 bin 通常包含一个
节点列表（大多数情况下，列表只有零个或一个节点）。
表访问需要易失性/原子读、写和
案例。因为没有其他方法可以安排这个
添加进一步的间接性，我们使用内在函数
(sun.misc.Unsafe) 操作。

We use the top (sign) bit of Node hash fields for control
purposes -- it is available anyway because of addressing
constraints.  Nodes with negative hash fields are specially
handled or ignored in map methods.


我们使用节点哈希字段的顶部（符号）位进行控制
目的 - 由于寻址，它无论如何都可用
约束。具有负哈希字段的节点是特别的
在地图方法中处理或忽略。


Insertion (via put or its variants) of the first node in an
empty bin is performed by just CASing it to the bin.  This is
by far the most common case for put operations under most
key/hash distributions.  Other update operations (insert,
delete, and replace) require locks.  We do not want to waste
the space required to associate a distinct lock object with
each bin, so instead use the first node of a bin list itself as
a lock. Locking support for these locks relies on builtin
"synchronized" monitors.

将第一个节点插入（通过 put 或其变体）
空 bin 是通过将其 CASing 到 bin 来执行的。这是
到目前为止，最常见的放置操作情况
密钥/哈希分布。其他更新操作（插入、
删除和替换）需要锁。我们不想浪费
将不同的锁对象与关联所需的空间
每个 bin，所以改为使用 bin 列表本身的第一个节点作为
一把锁。对这些锁的锁定支持依赖于内置
“同步”监视器。


Using the first node of a list as a lock does not by itself
suffice though: When a node is locked, any update must first
validate that it is still the first node after locking it, and
retry if not. Because new nodes are always appended to lists,
once a node is first in a bin, it remains first until deleted
or the bin becomes invalidated (upon resizing).


使用列表的第一个节点作为锁本身并不
足够了：当一个节点被锁定时，任何更新都必须首先
验证它在锁定后仍然是第一个节点，并且
如果没有，请重试。因为新节点总是附加到列表中，
一旦一个节点在一个 bin 中是第一个，它就会保持第一个直到被删除
或者 bin 失效（调整大小时）。


The main disadvantage of per-bin locks is that other update
operations on other nodes in a bin list protected by the same
lock can stall, for example when user equals() or mapping
functions take a long time.  However, statistically, under
random hash codes, this is not a common problem.  Ideally, the
frequency of nodes in bins follows a Poisson distribution
(http://en.wikipedia.org/wiki/Poisson_distribution) with a
parameter of about 0.5 on average, given the resizing threshold
of 0.75, although with a large variance because of resizing
granularity. Ignoring variance, the expected occurrences of
list size k are (exp(-0.5) pow(0.5, k) / factorial(k)). The
first values are:

per-bin 锁的主要缺点是其他更新
对受相同保护的 bin 列表中的其他节点进行操作
锁定可能会停止，例如当用户 equals() 或映射时
函数需要很长时间。然而，统计上，在
随机哈希码，这不是一个常见问题。理想情况下，
箱中节点的频率遵循泊松分布
(http://en.wikipedia.org/wiki/Poisson_distribution) 与
给定调整大小阈值，参数平均约为 0.5
0.75，虽然由于调整大小而有很大的差异
粒度。忽略方差，预期出现的
列表大小 k 是 (exp(-0.5) pow(0.5, k) / factorial(k))。这
第一个值是：

0:    0.60653066
1:    0.30326533
2:    0.07581633
3:    0.01263606
4:    0.00157952
5:    0.00015795
6:    0.00001316
7:    0.00000094
8:    0.00000006
more: less than 1 in ten million

Lock contention probability for two threads accessing distinct
elements is roughly 1 / (8 #elements) under random hashes.

两个线程对于不同元素的锁争用大概是 1 / (8 #elements)在随机哈希的情况下

Actual hash code distributions encountered in practice
sometimes deviate significantly from uniform randomness.  This
includes the case when N > (1<<30), so some keys MUST collide.
Similarly for dumb or hostile usages in which multiple keys are
designed to have identical hash codes or ones that differs only
in masked-out high bits. So we use a secondary strategy that
applies when the number of nodes in a bin exceeds a
threshold. These TreeBins use a balanced tree to hold nodes (a
specialized form of red-black trees), bounding search time to
O(log N).  Each search step in a TreeBin is at least twice as
slow as in a regular list, but given that N cannot exceed
(1<<64) (before running out of addresses) this bounds search
steps, lock hold times, etc, to reasonable constants (roughly
100 nodes inspected per operation worst case) so long as keys
are Comparable (which is very common -- String, Long, etc).
TreeBin nodes (TreeNodes) also maintain the same "next"
traversal pointers as regular nodes, so can be traversed in
iterators in the same way.

The table is resized when occupancy exceeds a percentage
threshold (nominally, 0.75, but see below).  Any thread
noticing an overfull bin may assist in resizing after the
initiating thread allocates and sets up the replacement array.
However, rather than stalling, these other threads may proceed
with insertions etc.  The use of TreeBins shields us from the
worst case effects of overfilling while resizes are in
progress.  Resizing proceeds by transferring bins, one by one,
from the table to the next table. However, threads claim small
blocks of indices to transfer (via field transferIndex) before
doing so, reducing contention.  A generation stamp in field
sizeCtl ensures that resizings do not overlap. Because we are
using power-of-two expansion, the elements from each bin must
either stay at same index, or move with a power of two
offset. We eliminate unnecessary node creation by catching
cases where old nodes can be reused because their next fields
won't change.  On average, only about one-sixth of them need
cloning when a table doubles. The nodes they replace will be
garbage collectable as soon as they are no longer referenced by
any reader thread that may be in the midst of concurrently
traversing table.  Upon transfer, the old table bin contains
only a special forwarding node (with hash field "MOVED") that
contains the next table as its key. On encountering a
forwarding node, access and update operations restart, using
the new table.

Each bin transfer requires its bin lock, which can stall
waiting for locks while resizing. However, because other
threads can join in and help resize rather than contend for
locks, average aggregate waits become shorter as resizing
progresses.  The transfer operation must also ensure that all
accessible bins in both the old and new table are usable by any
traversal.  This is arranged in part by proceeding from the
last bin (table.length - 1) up towards the first.  Upon seeing
a forwarding node, traversals (see class Traverser) arrange to
move to the new table without revisiting nodes.  To ensure that
no intervening nodes are skipped even when moved out of order,
a stack (see class TableStack) is created on first encounter of
a forwarding node during a traversal, to maintain its place if
later processing the current table. The need for these
save/restore mechanics is relatively rare, but when one
forwarding node is encountered, typically many more will be.
So Traversers use a simple caching scheme to avoid creating so
many new TableStack nodes. (Thanks to Peter Levart for
suggesting use of a stack here.)

The traversal scheme also applies to partial traversals of
ranges of bins (via an alternate Traverser constructor)
to support partitioned aggregate operations.  Also, read-only
operations give up if ever forwarded to a null table, which
provides support for shutdown-style clearing, which is also not
currently implemented.

Lazy table initialization minimizes footprint until first use,
and also avoids resizings when the first operation is from a
putAll, constructor with map argument, or deserialization.
These cases attempt to override the initial capacity settings,
but harmlessly fail to take effect in cases of races.

The element count is maintained using a specialization of
LongAdder. We need to incorporate a specialization rather than
just use a LongAdder in order to access implicit
contention-sensing that leads to creation of multiple
CounterCells.  The counter mechanics avoid contention on
updates but can encounter cache thrashing if read too
frequently during concurrent access. To avoid reading so often,
resizing under contention is attempted only upon adding to a
bin already holding two or more nodes. Under uniform hash
distributions, the probability of this occurring at threshold
is around 13%, meaning that only about 1 in 8 puts check
threshold (and after resizing, many fewer do so).

TreeBins use a special form of comparison for search and
related operations (which is the main reason we cannot use
existing collections such as TreeMaps). TreeBins contain
Comparable elements, but may contain others, as well as
elements that are Comparable but not necessarily Comparable for
the same T, so we cannot invoke compareTo among them. To handle
this, the tree is ordered primarily by hash value, then by
Comparable.compareTo order if applicable.  On lookup at a node,
if elements are not comparable or compare as 0 then both left
and right children may need to be searched in the case of tied
hash values. (This corresponds to the full list search that
would be necessary if all elements were non-Comparable and had
tied hashes.) On insertion, to keep a total ordering (or as
close as is required here) across rebalancings, we compare
classes and identityHashCodes as tie-breakers. The red-black
balancing code is updated from pre-jdk-collections
(http://gee.cs.oswego.edu/dl/classes/collections/RBCell.java)
based in turn on Cormen, Leiserson, and Rivest "Introduction to
Algorithms" (CLR).

TreeBins also require an additional locking mechanism.  While
list traversal is always possible by readers even during
updates, tree traversal is not, mainly because of tree-rotations
that may change the root node and/or its linkages.  TreeBins
include a simple read-write lock mechanism parasitic on the
main bin-synchronization strategy: Structural adjustments
associated with an insertion or removal are already bin-locked
(and so cannot conflict with other writers) but must wait for
ongoing readers to finish. Since there can be only one such
waiter, we use a simple scheme using a single "waiter" field to
block writers.  However, readers need never block.  If the root
lock is held, they proceed along the slow traversal path (via
next-pointers) until the lock becomes available or the list is
exhausted, whichever comes first. These cases are not fast, but
maximize aggregate expected throughput.

Maintaining API and serialization compatibility with previous
versions of this class introduces several oddities. Mainly: We
leave untouched but unused constructor arguments refering to
concurrencyLevel. We accept a loadFactor constructor argument,
but apply it only to initial table capacity (which is the only
time that we can guarantee to honor it.) We also declare an
unused "Segment" class that is instantiated in minimal form
only when serializing.

Also, solely for compatibility with previous versions of this
class, it extends AbstractMap, even though all of its methods
are overridden, so it is just useless baggage.

This file is organized to make things a little easier to follow
while reading than they might otherwise: First the main static
declarations and utilities, then fields, then main public
methods (with a few factorings of multiple public methods into
internal ones), then sizing methods, trees, traversers, and
bulk operations.
