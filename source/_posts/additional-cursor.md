---
title: 几天前写的比较
date: 2016-11-06 15:23:39
tags: java
---
## 需求分析
最近做了一个学生成绩排序的功能，我们的需求是首先按照学生作业是否完成排序，然后按照学生成绩高低排序，最后按照学生的名字进行排序，并且可以使用 Spinner 进行来回选择，让老师可以自由对学生进行排序。

## 我的做法
在 Java 中对于集合或者数组不是单纯的数字时，一般可以选择使用 Collections.sort 或者 Arrays.sort 函数来对对象进行排序。
Collections.sort 函数的用法是将一个 Comparator 对象传入。
首先是自定义 Comparator 对象，一般是方法有两种，通常可以使用Comparator或Comparable，在这里我选择了实现 Comparator 接口来完成排序。如下是一个学生分数升序的写法。

```java
inner class SortByPaperScopeAsc : Comparator<Student> {
    override fun compare(w1: Student, w2: Student): Int {
        val v1 = h1?.score ?: "-1"
        val v2 = h2?.score ?: "-1"
        return v2.compareTo(v1)
    }
}
```
但这里仅仅是满足一项需求，而不能做到满足多种条件排序。如我们再写一个需求：
```java
inner class SortByStatusDesc : Comparator<Student> {
    override fun compare(h1: Student, h2: Student): Int {
      val v1 = if (h1?.homeworkIsFinished) 1 else 0
      val v2 = if (h2?.homeworkIsFinished) 1 else 0
        return v2.compareTo(v1)
    }
}
```
我们需要满足已完成作业的排在前面，未完成作业的排在后面，然后对已完成作业的学生按分数升序排列。
毫无疑问的写法是直接定义一个排序的Comparator满足这样的需求。
```java
inner class SortByStatusAndScore : Comparator<Student> {
  override fun compare(w1: Student, w2: Student): Int {
    val compare = SortByStatusDesc().compare(w1, w2)
        if (compare == 0) {
            val v1 = w1?.score ?: "-1"
            val v2 = w2?.score ?: "-1"
            return v1.compareTo(v2)
        }
        return compare
  }
}
```
但是这样一来当我们按多种条件排序时，这样的写法无疑会出现代码出现大量的重复，这个时候我们可以考虑对这种方法进行封装，可以把不同的排序条件进行组合。
下面是我写的一个比较类
```java
class ComposeComparator<T> : Comparator<T> {
  val comparators: Array<out Comparator<T>>
  constructor(vararg comparators: Comparator<T>) {
      this.comparators = comparators
  }
  override fun compare(o1: T, o2: T): Int {
      comparators.forEach {
          val result = it.compare(o1, o2)
          if (result != 0) {
              return result
          }
      }
      return 0
  }
}
```
使用vararg，也就是可变长参数将不同的条件依次传入，可以做到不同单一排序方法的组合使用，比如
```java
ComposeComparator(SortByStatusDesc(), SortByScopeAsc(), SortByNameAsc()))
```
