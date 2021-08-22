---
title: 学堂在线C++程序设计第十章学习笔记
date: 2021-08-12 10:12:47
tags: ['C++']
category: C++
article: 学堂在线C++程序设计第十章学习笔记
---

# 学堂在线C++程序设计第十章学习笔记

## 多态

### 运算符重载

#### 重载规则

C++ 几乎可以重载全部的运算符，而且只能够重载C++中已经有的
- 不能重载的：".",".*","::","?:"

重载之后运算符的优先级和结合性都不会改变

运算符重载是针对新类型数据的实际需要，对原有运算符进行适当的改造。

例如：
- 使复数类的对象可以用 + 运算符实现加法
- 是时钟类对象可以用 ++ 运算符实现时间增加1秒

重载为类的非静态成员函数
重载为非成员函数

#### 双目运算符重载为成员函数

重载为类成员的运算符函数定义形式：
```C++
函数类型 operator 运算符(形参)
{

}
```
参数个数 = 原操作数个数 - 1(后置++,--除外)


双目运算符重载规则
- 如果要重载B为类成员函数，使之能够实现表达式 oprd1 B oprd2,其中 oprd1 为 A类对象，则B则应被重载为A的成员函数，形参类型应该是 oprd2 所属类型
- 经重载后，表达式 oprd1 B oprd2 相当于 oprd1.operator B(oprd2)


例子8-1 复数类加减法运算重载为成员函数
- 要求
    将 +，-运算重载为复数类的成员函数
- 规则
    实部和虚部分别相加减
- 操作数
    两个操作数都是复数类的对象

```C++
#include <iostream>
using namespace std;

class Complex{
    public:
        Complex(double r = 0.0, double i = 0.0):real(r),imag(i) {};
        //运算符+重载成员函数
        Complex operator +(const Complex &c2) const;
        //运算符-重载
        Complex operator -(const Complex &c2) const;
        //运算符 * 重载
        Complex operator *(const Complex &c2) const;
        void display() const;
    private:
        double real;
        double imag;
};

Complex Complex::operator+(const Complex &c2) const {
    return Complex(real + c2.real, imag + c2.imag);
}

Complex Complex::operator-(const Complex &c2) const {
    return Complex(real - c2.real, imag - c2.imag);
}

Complex Complex::operator*(const Complex &c2) const{
    return Complex(real * c2.real, imag * c2.imag);
}

void Complex::display() const {
    cout << "(" << real << "," << imag << ")" << endl;
}

int main() {
    Complex c1(1,2),c2(5,6);
    c1.display();
    c2.display();
    Complex c3 = c1 + c2;
    c3.display();
    Complex c4 = c1 * c2;
    c4.display();
}

```

#### 单目运算符重载成为成员函数

前置单目运算符重载规则
- 如果要重载U为类成员函数，使之能够实现表达式 U oprd, 其中 oprd 为 A 类对象，则 U 应被重载为 A 类的成员函数，无形参。
- 重载后，表达式 U oprd 相当于 oprd.operator U()

后置单目运算符 ++ 和 -- 重载规则
- 如果要重载 ++ 或 -- 为类成员函数，使之能实现表达式 oprd++ 或 oprd--，其中 oprd 为 A 类对象，则 ++,--应被重载为A类的成员函数，且具有一个int类型的形参
- 经重载后，表达式 oprd++ 相当于 oprd.operator ++(0)

```
Complex & Complex::operator++() {
    real++;
    imag++;
    return *this;
}

Complex Complex::operator++(int ) {
    Complex that = *this;
    ++(*this);
    return that;
}
```


#### 运算符重载为非成员函数

规则
- 函数的形参代表依自左至右次序排列的各操作数
- 重载为非成员函数时
    - 参数个数 = 原操作个数（后置++，--除外）
    - 至少应该有一个自定义类型的参数
- 后置单目运算符 ++ 和 -- 的重载函数，形参列表中要增加一个int, 但不必写形参名
- 如果在运算符的重载函数中需要操作某类对象的私有成员，可以将此函数声明为该类的友元

双目运算符重载
- oprd1 B oprd2 等于 operator B(oprd1,oprd2)

前置单目重载
- B oprd 等于 operator B(oprd)

后置单目++，--重载
- oprd B 等于 operator B(oprd, 0)

### 虚函数

虚函数
- virtual 定义虚函数 使用运行时多态
- 虚函数必须在类外实现函数体
- 虚函数必须是非静态的成员函数，经过派生后，就可以实现运行时多态

什么函数可以是虚函数
- 一般成员函数
- 构造函数不能是虚函数
- 析构函数可以

virtual 关键字
- 派生类可以不显式的用virtual声明虚函数，这时系统就会用以下规则来判断派生类的一个函数成员是不是虚函数：
    - 该函数是否与基类的虚函数有相同的名称，参数个数及对应参数类型
    - 该函数是否与基类的虚函数有相同的返回值或者满足类型兼容规则的指针，引用型的返回值
- 如果从名称，参数及返回值三个方面检查之后，派生类的函数满足上述条件，就会自动确定为虚函数。这时，派生类虚函数便会覆盖基类的虚函数
    - 派生类中的虚函数还会隐藏基类中同名函数的所有其他重载形式
    - 一般习惯于在派生类的函数中也使用 virtual 关键字，以增加程序的可读性


#### 虚析构函数

通过基类指针调用对象的析构函数，就需要让基类的析构函数成为虚函数，否则 delete 的结果是不确定的

写成 虚析构函数，那么delete就会执行派生类的析构函数，不然只会静态绑定，永远执行基类的析构函数，派生类的成员就无法delete

```C++
class Base{
    public:
        //基类 虚析构函数
        virtual ~Base();
}

class A:public Base{
    public:
        virtual ~A();
}

void fun(Base* B) {
    delete B;
}

int main() {
    Base* b = new A();
    fun(b);
    return 0;
}
```


#### 虚表和动态绑定

虚表
- 每个多态类有一个虚表
- 虚表中有当前类的各个虚函数的入口地址
- 每个对象有一个指向当前类的虚表的指针（虚指针 vptr）

动态绑定的实现
- 构造函数中为对象的虚指针赋值
- 通过多态类型的指针或引用调用成员函数时，通过虚指针找到虚表，进而找到所调用的虚函数的入口地址
- 通过该入口地址调用虚函数


#### 抽象类

`纯虚函数`是一个在基类中声明的虚函数，它在该基类中没有定义具体的操作内容，要求各派生类根据实际需要定义自己的版本

纯虚函数的声明：
virtual 函数类型 函数名（参数表） = 0;

带有纯虚函数的就是`抽象类`

抽象类的作用
- 将有关的数据和行为组织在一个继承层次结构中，保证派生类具有要求的行为
- 对于暂时无法实现的函数，可以声明为纯虚函数，留给派生类去实现

注意
- 抽象类只能作为基类使用
- 不能定义抽象类的对象

#### override

override
- 多态行为的基础：基类声明虚函数，派生类声明一个函数覆盖该虚函数
- 覆盖要求：函数签名（signature）完全一致
- 函数签名包括：函数名 参数列表 const


- C++引入显式函数覆盖，在编译器而非运行期捕获此类错误
- 在虚函数显式重载中运用，编译器会检查基类是否存在一虚拟函数，与派生类中带有声明override的虚拟函数，有相同的函数签名；若不存在，则回报错误

#### final

final 类
- 不能被继承
- 继承出现编译错误

final 方法
- 不能被覆盖
- 覆盖出现编译错误

