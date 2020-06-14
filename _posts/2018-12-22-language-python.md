---
layout: post
title:  "Python学习笔记"
date:   2018-12-22 00:00:00 +0000
categories: python
---

## 学习资料

点了百度排名最上面的[廖雪峰](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)的讲解，先看着。

菜鸟教程系列：[Python 3 教程](http://www.runoob.com/python3/python3-tutorial.html)

Python官方文档：[Python 3.7.1 documentation](https://docs.python.org/3/index.html)



## 工具

测试 HTTP 请求及响应：[httpbin](https://github.com/Runscope/httpbin)



## 语言基础

语言中的一些值得注意的特性。



### list的浅拷贝与深拷贝

Python中没有指针的概念，需要自己注意是否传入了一个共享的变量地址。如果直接将一个list传递给另一个变量，不管是通过赋值或者作为函数的参数，都是以地址的形式传递的。如果需要使用一个全新的list变量，需要使用拷贝函数：

```python
# -*- coding: utf-8 -*-
import copy
list1 = [1, [2, ]]
# 只拷贝第一层
list2 = copy.copy(list1)
# 拷贝全部内容
list3 = copy.deepcopy(list1)
# 
list1[1].append(3)
list1.append(4)
print(list1)
print(list2)
print(list3)

```



### class



#### \_\_new\_\_与\_\_init\_\_方法

new负责创建一个对象，此时的传入参数为类属性（区别于实例属性）。

new方法需要返回一个实例，这个实例的init函数会被调用，而实例本身则成为self属性的值。

init函数才对应c++中的构造函数，该函数传入对象被创建时的初始化参数。

因此，在python中，类的init函数被调用与否取决于new函数。同时，我们也可以在new函数中修改实例的默认属性。



#### super函数

会按一定顺序调用所有基类的相应函数。



#### 成员函数

成员函数的第一个参数必须是指向自身的self，这一点与c++自动添加this指针有所不同。

