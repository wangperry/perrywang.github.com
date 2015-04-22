---
layout: post
title: Spark 使用combinByKey
category: Spark
tags: [spark, combinByKey]
---

# Spark 使用combinByKey说明

------
示例，处理key-value pair数据并求出对应key的平均值。

``` python
kvpair =[("a", 3), ("b", 4), ("a", 1)]
kvdc = sc.parallelize(kvpair)

sumCount = kvdc.combineByKey(lambda x: (x, 1), lambda x, y: (x[0]+ y, x[1] + 1), lambda x, y :(x[0] + y[0], x[1] + y[1]))

averageByKey = sumCount.map(lambda (label, (value_sum, count)):(label, value_sum /count))

print averageByKey.collectAsMap()
```

 计算结果：
```
{'a': 2, 'b': 4}
```

为了并行的聚合RDD中的元素，Spark中提供了一个高度抽象的combineByKey方法.想要快速的读懂combineByKey方法中的三个lambda参数并不是这么简单的事，在此为自己做个笔记。

combineByKey 方法定义了三个函数：

>* createCombiner
>* mergeValue
>* mergeCombiner

### createCombine 函数
```
lambda x: (x, 1)
```
createCombine函数是对应于combinByKey方法中的第一个参数，lambda中的参数是对应于key-value对中value值来说的。假如要使用combineByKey来求和与计数，则创建的"combiner"形如(sum, count)。sum 值就对应于key-value中value值:"x", 并设置"1"初始化key次数。

### mergeValue 函数
```
lambda x, y: (x[0]+ y, x[1] + 1)
```
这个函数是用来告诉combinByKey方法，对combiner给定一个新的值时该做些什么样的工作。这个lambda函数接收两个参数：一个combiner, 一个new value.即x是一个"combiner"【combiner的结构对应createCombine中的(sum, count)】, y 对应着一个新的值。

### mergeCombiner 函数
```
lambda x, y :(x[0] + y[0], x[1] + y[1])
```
合并两人个Combiners,lambda的两个参数都指的是combiner。




