---
title: C++程序设计第三章
date: 2021-07-19 10:12:47
tags: ['C++']
category: C++
article: C++程序设计第三章
---

# C++程序设计第三章

## 数据的输入输出

输入输出看成是数据的流动

IO流
- 将数据从一个对象到另一个对象的流动抽象为`流`
- 流在使用前要建立，使用后要删除
- 数据的输入输出是通过IO流来实现的，cin和cout是预定义的流类对象。cin用来处理标准输入，即键盘输入。cout用来处理标准输出，即屏幕输出
- 从流中获取数据的操作称为

预定义的插入符和提取符
- `<<` 是预定义的插入符，作用在流对象cout上就可以向标准输出设备输出
- `>>` 是预定义的提取符，作用在流对象cin上
- 可以写多个


常用的IO流类库操纵符

| 操纵符名|含义|
|:---|:----|
|dec|数值数据采用十进制表示|
|hex|数值数据采用十六进制表示|
|oct|数值数据采用八进制表示|
|ws|提取空白符|
|endl|插入换行符并刷新流|
|ends|插入空字符|
|setprecision(int)| 设置浮点数的小数位数(包括小数点)|
|setw(int)| 设置域宽|


## 选择结构

### if语句

if语句的语法形式
- if (表达式)语句
    if (x > y) cout  << x;
- if (表达式) 语句1 else 语句2
    if (x < y) cout << x;
    else cout <<y;
- if (表达式1) 语句1 else if (表达式2) 语句2 else 语句n

```C++
if (表达式1) 
    语句1
else if (表达式2) 
    语句2 
else 
    语句n
```

上面的表达式等同于下面的表达式
```C++
if (表达式1) 
    语句1
else 
    if (表达式2) 
        语句2 
    else 
        语句n
```

### switch语句

```C++
switch （表达式1） {
    case 常量值1：
        语句1
        break;
    case 常量值2：
        语句2
        break;
    default:
        默认语句
}
```

表达式和常量值都是int或char型

## 循环结构

### while语句

计算0到10之和

```C++
int sum = 0;
while （int i <= 10） {
    sum += i;
    i++;
}
```

执行顺序
- 先判断while表达式的值，若为true，执行语句
- 执行完语句在判断while表达式的值，直到为false,不再执行

while里面可以是`复合语句`，其中必须含有改变`条件表达式`值得语句

### do-while语句

计算0到10之和

```C++
int sum = 0;
int i = 0;
do {
    sum += i;
    i++;
} while （ i <= 10）
```

执行顺序
- 先执行语句
- 执行完语句判断while表达式的值，如果为true，那么接着执行语句
- 执行完语句后再次判断while表达式的值，直到为false
- 语句至少会执行一次


## for语句

输入一个整数，求出他的所有因子

```C++
#include <iostream>
using namespace std;

int main() {
    int n;
    cin >> n;
    cout << "Number:" << n << "Factors\n";
    for (int k = 1;k <= n;k++) {
        if (n % k == 0) {
            cout << k << endl;
        }
    }
    cout << "end" << endl;
    return 0;
}
```

### 嵌套的控制结构

输入一系列整数，统计出正整数个数i和负整数个数j,读入0则结束

```C++
#include <iostream>
using namespace std;

int main() {
    int n;int z = 0;int f = 0;
    cin >> n;
    while (n != 0)
    {
        /* code */
        if (n > 0) {
            z++;
        } else if (n < 0) {
            f++;
        }
        cin >> n;
    }
    cout << "n:" << n << endl;
    cout << "正数数量：" << z << endl;
    cout << "负数数量：" << f << endl;
    
    return 0;
}
```

## 自定义类型

类型别名：为已有类型另外命名
- typedef 已有类型名 新类型名表
- 例子：typedef double area
- using 新类型名 = 已有类型名
- 例子：using area = double


枚举类型：
- 定义方式：
    将全部可取值列出来
- 语法形式：
    enum 枚举类型名 {变量值列表}
- 例子
    enum week {1,2,3,4,5,6,7}

C++包含两种枚举类型：
- 不限定作用域
- 限定作用域


不限定作用域枚举类型
- 枚举元素是常量，不能赋值
- 枚举元素具有默认值，依次为0,1,2,3
- 也可以在声明时另行指定枚举元素的值
- 枚举值可以进行关系运算
- 整数值不能赋值为枚举变量，如需要，应该进行强制转换
- 枚举可以给整形赋值

```C++
#include <iostream>
using namespace std;

enum GameResult {WIN,LOSE,TIE,CANCEL};

int main() {
    GameResult Result;
    enum GameResult omit = CANCEL;
    for (int count = WIN; count <= CANCEL; count++) {
        Result = GameResult(count);
        if (Result == omit) {
            cout << "The game was canceld" << endl;
        } else {
            cout << "The game was played" << endl;
            if (Result == WIN) cout << "and we Win" << endl;
            if (Result == LOSE) cout << "and we lose" << endl;
        }

    }
    
    return 0;
}
```

auto 类型 与 decltype类型
- auto ：编译器通过初始值自动推断变量的类型
    例如：auto val = val1 + val2
    如果val1 val2是int，那么val是int
    如果val1 val2是double，那么val是double
- decltype:定义一个变量与某一表达式的类型相同，单并不用该表达式初始化变量
    例如：decltype(i)j = 2;
    表示j 用 2 作为初始值，但是类型和i一致










