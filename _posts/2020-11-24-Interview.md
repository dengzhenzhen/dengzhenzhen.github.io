---
layout:     post
title:      记录今天面试遇到的几个问题
subtitle:   
date:       2020-11-24
author:     dengzhenzhen
header-img: img/post-bg-keybord.jpg
catalog:     true
tags:
    - Python
    - numpy
---


##今天面试遇到几个问题，记录一下

今天面试时，遇到这么个问题：
给了一段代码：


```python
A = [1, 2, 3, 4, 5]
B = []
for i in A:
    B.append(i * i)
```

要求能否用效率更高的方式来写

一般这种情况，就直接祭出列表推导式了：


```python
B = [i * i for i in A]
```

然后面试官又问，能否用别的方式，提示道可以用其他库

那我猜肯定是要考我numpy的用法和向量化了，也没多想，直接在纸上写：


```python
import numpy as np
B = np.array(A).dot(np.array(A))
```

其实这时候已经错了，毕竟也很久没和矩阵打交道了，我忘记了点乘每一项之后是要加起来的( ╯□╰ )

这问题也是一直到上地铁才想到，两个1维向量，点乘应该是一个数才对

这种写法，B其实变成了一个数


```python
B
```




    55



面试官这时候分享了她想要的答案：


```python
A = np.array(A)
B = A * A
B
```




    array([ 1,  4,  9, 16, 25])



事实上，如果想用点乘来做，应该这么写：


```python
A = [1, 2, 3, 4, 5]
A = np.array(A)
B = A.dot(A * np.identity(A.shape[0]))
B
```




    array([ 1.,  4.,  9., 16., 25.])



其中 *np.identity()* 是单位矩阵

那 *, np.dot, array.dot之间到底什么关系，做个实验


```python
X = np.array([
    [1,2],
    [3,4]
])
Y = np.array([
    [5,6],
    [7,8]
])
print(X * Y)
print('--------------')
print(X.dot(Y))
print('--------------')
print(Y.dot(X))
print('--------------')
print(np.dot(X, Y))
print('--------------')
print(np.dot(Y, X))
```

    [[ 5 12]
     [21 32]]
    --------------
    [[19 22]
     [43 50]]
    --------------
    [[23 34]
     [31 46]]
    --------------
    [[19 22]
     [43 50]]
    --------------
    [[23 34]
     [31 46]]
    

可以看出

\* 是相同下标相乘, 如果一边是一个数，则整个矩阵乘那个数

*X.dot(Y)* 是 X 左乘 Y 
*np.dot(X, Y)* 是左边左乘右边


之后又问了个问题：


```python
def func(A):
    B = A * A
    return B
A = np.array([1, 2, 3, 4, 5])
B = func(A)
C = B
# A,B,C内存地址的关系？
```

C和B指向同一个地址很明显：


```python
id(B) == id(C)
```




    True



关键是A和B，在这里我并不清楚A * A返回的到底是一个新的array对象还是在A的基础上修改的对象，所以这里只好老实回答不清楚

面试官这时候说A和BC一样指向的同一个地址，我先记下这个结论

回来打开notebook试一下：


```python
id(A) == id(B)
```




    False



竟然不一样？  我蒙蔽了

大概是因为对于np.array的乘法的理解偏差，这里做个实验：


```python
A = np.array([[1,2,3,4,5]])
B = A * A
print(id(A))
print(id(B))
```

    1533489663152
    1533489660272
    

两个id并不一样

在这里应该是想考察函数里面改变了对象, 跟原来对象的关系, 但实际上乘号是不会影响原array的值的

如果把题目改成这样，那就可以达到想要的效果


```python
def func(A):
    A[3] = 100
    B = A
    return B
A = np.array([1, 2, 3, 4, 5])
B = func(A)
C = B
print(id(A) == id(B) == id(C))
A
```

    True
    




    array([  1,   2,   3, 100,   5])




```python

```
