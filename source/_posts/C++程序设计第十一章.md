---
title: 学堂在线C++程序设计第十一章学习笔记
date: 2021-08-18 10:12:47
tags: ['C++']
category: C++
article: 学堂在线C++程序设计第十一章学习笔记
---

# 学堂在线C++程序设计第十一章学习笔记

## 模板和群体数据

### 模板

#### 函数模板

如果不使用函数模板，需要写多个函数，比如求绝对值
- 整数绝对值
- 浮点数绝对值

```C++
int abs(int x) {
    return x < 0 ? -x : x;
}

double abs(double x) {
    return x < 0 ? -x : x;
}
```

使用模板函数可以只写一个函数

```C++
template<typename T>
T abs(T x) {
    return x < 0 ? -x : x;
}
```

如果使用int参数调用，那么编译器会根据上面的模板生成一个int型的绝对值函数，也就是将T类型换成int类型

函数模板定义语法
- 语法
    template <模板参数表>
    函数定义
- 模板参数表的内容
    - 类型参数 class (或 typename) 标识符
    - 常量参数 类型说明符 标识符
    - 模板参数 template<参数表> class 标识符

注意：
- 一个函数模板并非自动可以处理所有类型的数据
- 只有能够进行函数模板中运算的类型 可以作为类型实参
- 自定义的类，需要重载模板中的运算符，才能作为类型实参

#### 类模板

类模板的作用
使用类模板使用户可以为类声明一种模式，使得类中的某些数据成员，某些成员函数的参数，某些成员函数的返回值，能取任意类型

声明
```C++
template<模板参数表>
class 类名
{

}
```

如果需要再类模板以外定义其成员函数，则要求采用以下的形式：
```C++
template<模板参数表>
类型名 类名<模板参数标识符列表>::函数名(参数)
```

定义一个类模板
```C++
template <class T>
class Store{
    private:
        T item;
        bool haveValue;
    public:
        Store();
        T &getElem();
        void putElem(const T &x);
}

template<class T>
T Store<T>::getElem() {
    return item;
}

template<class T>
void Store<T>::putElem(const T *x) {
    item = x;
}

template <class T>
Store<T>::Store():haveValue(false){}
```

调用类模板

```C++
int main() {
    Store<int> s1,s2;
    s1.putElem(3);
    s2.putElem(4);
    cout << s1.getElem();
}

```

### 线性群体

群体的概念
- `群体`是指由多个数据元素组成的集合体
    群体可以分为两个大类：`线性群体`和`非线性群体`
- `线性群体`中的元素按位置排列有序
    可以分为第一个元素，第二个元素等
- `非线性群体`不用位置顺序来标识元素

线性群体中的元素次序和逻辑位置关系是对应的。按照访问元素的不同方法分为`直接访问`,`顺序访问`和`索引访问`

### 数组

直接访问的线性群体--数组类
- 静态数组是具有固定元素个数的群体，其中的元素可以通过下标直接访问。
- 缺点：大小在编译时确定，在运行时无法修改
- 动态数组由一系列位置连续的，任意数量相同类型的元素组成
- 优点：其元素个数可在程序运行时改变


### 链表类

链表是一种动态数据结构，可以用来表示顺序访问的线性群体
链表是由系列结点组成的，结点可以在运行时动态生成
每一个结点包括数据域和指向链表中下一个结点的指针
如果链表每个结点只有一个指向后继结点的指针，则为单链表

单链表结点模板

```C++
template<class T>
class Node{
    private:
        Node<T> *next;
    public:
        T data;
        Node(const T& item, Node<T>* next = 0);
        void insertAfter(Node<T> *p);
        Node<T> *deleteAfter();
        Node<T> *nextNode() const;
}

template<class T>
void Node<T>::insertAfter(Node<T> *p) {
    p->next = next;
    next = p;
}

template<class T>
Node<T>* Node<T>::deleteAfter() {
    Node<T> *tempPtr = next;
    if (next == 0) {
        return 0;
    }
    next = tempPtr->next;
    return tempPtr;
}
```

## 多态

### 链表类模板

链表的基本操作
- 生成链表
- 插入结点
- 查找结点
- 删除结点
- 遍历链表
- 清空链表

### 栈类模板

### 队列模板

### 插入排序

每一步将一个待排序元素插入已排序序列的适当位置

### 选择排序

每次从待排序序列中选择一个最小的元素，顺序排在已排序序列的最后




### 迭代器

迭代器是算法和容器的桥梁
- 迭代器用作访问容器中的元素
- 算法不直接操作容器中的数据，而是通过迭代器间接操作

算法和容器独立
- 增加新的算法，无需影响容器的实现
- 增加新的容器，原有的算法也可以适用


输入流迭代器
- istream_iterator<T>
- 以输入流 如 cin 为参数构造
- 可用 *(p++) 获得下一个输入的元素

输出流迭代器
- ostream_iterator<T>
- 构造时需要提供输出流 如 cout
- 可用 (*p++) = x 将 x 输出到输出流

二者都属于适配器
- 适配器是用来为已有对象提供新的接口的对象
- 输入流适配器和输出流适配器为流对象提供了迭代器的接口


### 容器的基本功能和分类

- 容器类是容纳，包含一组元素或元素集合的对象
- 基于容器中元素的组织方式分类：顺序容器，关联容器
- 按照与容器所关联的迭代器类型分类：可逆容器，随机访问容器


### 顺序容器

##### 顺序容器的基本功能

- STL中的顺序容器
    - 向量
    - 双端队列(deque)
    - 列表(list)
    - 单向链表(forward_list)
    - 数组(array)
- 元素线性排列，可以随时在指定位置插入元素和删除元素
- 必须符合 Assignable 这一概念（具有共有的复制构造函数并可以用 '=' 赋值）
- array 对象的大小固定，forward_list有特殊的添加和删除操作

顺序容器的接口（不包含单向链表和数组）
- 构造函数
- 赋值函数：assign
- 插入函数: insert, push_front(只对list和deque)，push_back, emplace, emplace_front
- 删除函数：erase, clear, pop_front, pop_back, emplace_back
- 首尾元素的直接访问：front, back
- 改变大小：resize


#### 顺序容器的特征

向量
- 特点
    - 一个可以扩展的动态数组
    - 随机访问，在尾部插入或删除元素块
    - 在中间或头部插入或删除元素慢
- 容量
    - 容量：实际分配空间的大小
    - s.capacity: 返回当前容量
    - s.reserve(n): 容量小于n，则对s进行扩展，使其容量至少为n

双端队列
- 特点
    - 在两端插入或删除元素快
    - 在中间插入或删除元素慢
    - 随机访问较快，但比向量慢

列表
- 特点
    - 在任意位置插入和删除元素都很快
    - 不支持随机访问
- 接合操作（splice）
    - s.splice(p, s2, q1, q2): 将s2中[q1,q2)移动到s1中p所指向元素之前


#### 顺序容器的插入迭代器与适配器

顺序容器的插入迭代器
- 用于向容器头部，尾部或中间指定位置插入元素的迭代器
- 包括前插迭代器（front_inserter），后插迭代器（back_inserter）和任意位置插入迭代器（inserter）
- 例子：
    list<int> s;
    back_iserter iter(s);
    *(iter++) = 5; //通过iter把5插入s末尾


顺序容器的适配器
- 以顺序容器为基础构建一些常用数据结构，是对顺序容器的封装
    - 栈
    - 队列
    - 优先级队列：最 `大` 的先弹出

栈和队列模板
- 栈模板
    template <class T, class Sequence = deque<T>> class stack;
- 队列模板
    template <class T,class FrontInsertionSequence = deque<T>> class queue;
- 栈可以用任何一种顺序容器作为基础容器，而队列只允许用前插顺序容器（双端队列或列表）

栈和队列共同支持的操作
- 比较
- 返回元素个数
- empty
- push
- pop
- 不支持迭代器，因为不允许对任意元素访问


栈和队列不同的操作
- 栈的操作
    s.top 返回栈顶元素的引用
- 队列操作
    s.front 获得队头元素的引用
    s.back 获得队尾元素的引用

优先级队列
- 优先级队列也像栈和队列一样支持元素的压入和弹出，但元素弹出顺序和元素大小有关，每次弹出最大的
- template <class T, class Sequence = Vector<T>> class priority_queue
- 优先级队列的基础容器必须是支持随机访问的顺序容器
- 支持栈和队列的size,empty, push, pop几个函数
- 优先级队列并不支持比较操作
- 与栈类似，优先级队列提供一个 top 函数，可以获得下一个即将被弹出元素的引用

