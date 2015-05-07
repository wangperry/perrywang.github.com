---
layout: post
title: Spark 1.3.0 API Map & Reduce 说明[pyspark]
category: Spark
tags: [Map, Reduce, pyspark]
---

------

以pyspark 为例说明：

## map

map 的lambda函数应用到RDD中的每一个元素，transmition 后得到一个新的RDD. __原始RDD中的元素在新的RDD中有且只有一个元素与之对应__.
```
In [125]: lc = sc.parallelize(range(10), 3)

In [126]: lc.collect()
Out[126]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [127]: lcs = lc.map(lambda x: x + 5)

In [128]: lcs.collect()
Out[128]: [5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
```

## mapPartitions

mapPartitions **中的函数应用到RDD中的每一个分区**。函数的输入为一个可迭代的对象，输出为可迭代对象。

```
In [98]: a = sc.parallelize(range(10), 3)

In [113]: a.collect()
Out[113]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [117]: def myfunc(lists_range):
   .....:     final_li = []
   .....:     it = iter(lists_range)
   .....:     try:
   .....:         while True:
   .....:             pre = it.next()
   .....:             cur = it.next()
   .....:             final_li.append((pre, cur))
   .....:     except StopIteration:
   .....:         pass
   .....:     return iter(final_li)
   .....: 
   
In [120]: mfc = a.mapPartitions(myfunc)

In [121]: mfc.collect()
Out[121]: [(0, 1), (3, 4), (6, 7), (8, 9)]
```

## mapPartitionsWithIndex

Return a new RDD by applying a function to each partition of this RDD, while tracking the index of the original partition.
类似于mapPartitions, 但func函数还有一个整形参数表示分块的索引值。

```
In [113]: a.collect()
Out[113]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [169]: def f(splitIndex, it):
    yield splitIndex
    
In [170]: fc = a.mapPartitionsWithIndex(f)

In [171]: fs = fc.collect()

In [172]: fs
Out[172]: [0, 1, 2]
```

## mapPartitionsWithSplit

- Deprecated: use mapPartitionsWithIndex instead.

## mapValues

mapValues函数应用到每个key-value pairs中的value。   原RDD中的Key保持不变，与新的Value一起组成新的RDD中的元素。因此，该函数只适用于元素为KV对的RDD。

```
In [3]: kv1 = sc.parallelize([('a', 1), ('b', 2), ('c', 3), ('b', 22)])
In [7]: kv1.mapValues(lambda x: x % 2).collect()
Out[7]: [('a', 1), ('b', 0), ('c', 1), ('b', 0)]
```

## flatMap

flatMap 的动作可以拆分成两步，第一步类似于map,先映射,一条输入返回一 个对象。第二步扁平化，把所有的对象合并为一个对象。

```
In [3]: kv1 = sc.parallelize([('a', 1), ('b', 2), ('c', 3), ('b', 22)])
In [12]: kv1.flatMap(lambda x: (x[0] + '1', x[1] + 1)).collect()
Out[12]: ['a1', 2, 'b1', 3, 'c1', 4, 'b1', 23]     
```

## flatMapValues

flatMapValues类似于mapValues，不同的在于flatMapValues应用于元素为KV对的RDD中Value。每个一元素的Value被输入函数映射为一系列的值，然后这些值再与原RDD中的Key组成一系列新的KV对。【行为类似于mapValues】

## reduce

reduce将RDD中元素两两传递给输入函数，同时产生一个新的值，新产生的值与RDD中下一个元素再被传递给输入函数直到最后只有一个值为止。自己定义reduce的行为。

```
In [1]: kv = sc.parallelize([('a', 1), ('b', 2), ('c', 3)])
In [2]: kv.reduce(lambda x, y: (x[0] + y[0], x[1] + y[1]))
Out[2]: ('abc', 6) 
```

## reduceByKey

reduceByKey 按key相同的pairs两两值传入函数func中，得到新的值和原来的key重新组合得到新的kv对。

```
In [3]: kv1 = sc.parallelize([('a', 1), ('b', 2), ('c', 3), ('b', 22)])
In [4]: kv1_rd = kv1.reduceByKey(lambda x, y : x + y)
In [6]: kv1_rd.collect()
Out[6]: [('c', 3), ('b', 24), ('a', 1)]  
```











