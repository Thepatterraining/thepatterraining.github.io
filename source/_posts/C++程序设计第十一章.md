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




