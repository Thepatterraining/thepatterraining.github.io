---
title: 学堂在线C++程序设计第四章学习笔记
date: 2021-07-21 10:12:47
tags: ['C++']
category: C++
article: 学堂在线C++程序设计第四章学习笔记
---

# 学堂在线C++程序设计第四章学习笔记

## 函数定义

函数：定义好的可重用功能模块

定义函数：将一个模块的算法用C++描述出来

函数的返回值：需要返回的计算结果

定义函数的语法：
```
类型标识符 函数名（形式参数表）
{
    语句
}
```

形式参数表
- <类型名> 参数名，...

返回值
- return 一个计算结果
- 返回的类型是类型标识符的类型
- 没有返回值，类型标识符写`void`


## 函数调用

调用函数前需要先声明函数原型

函数原型
- 类型标识符
- 被调用函数名
- （类型说明的形参表）

函数调用
- 函数名（实参列表）

### 计算x的n次方

```C++
#include <iostream>
using namespace std;

double pow(double x, int n) {
    double sum = 1.0;
    for(int i = 1; i <= n; i++) {
        sum = x * sum;
    }
    return sum;
}

int main() {
    double sum = pow(2,4);
    cout << sum;
    return 0;
}
```

### 进制转换

输入一个8位二进制，输出十进制

```C++
#include <iostream>
using namespace std;

double pow(double x, int n) {
    double sum = 1;
    for (int i = 1; i <= n; i++) {
        sum *= x;
    }
    cout << "x:"<<x<<"n:"<<n<<endl;
    return sum;
} 

int main() {
    int h = 0;
    for (int i = 7; i >= 0; i--) {
        char d;
        cout << "输入：";
        cin >> d;
        if (d == '1') {
            //转换进制
            h += static_cast<int>(pow(2, i));
        }
    }
    cout << h;
    return 0;
}
```

### 计算圆周率

```C++
#include <iostream>

using namespace std;

double arctan(double x) {

          double sqr = x * x;

          double e = x;

          double r = 0;

          int i = 1;

          while (e / i > 1e-15) {

                   double f = e / i;

                   r = (i % 4 == 1) ? r + f : r - f;

                   e = e * sqr;

                   i += 2;

          }

          return r;

}

int main() {

          double a = 16.0 * arctan(1/5.0);

          double b = 4.0 * arctan(1/239.0);

          //注意：因为整数相除结果取整，如果参数写1/5，1/239，结果就都是0

 

          cout << "PI = " << a - b << endl;

          return 0;

}
```

### 求回文

寻找并输出11~999之间的数M，它满足M、M<sup>2</sup>和M<sup>3</sup>均为回文数。

```C++
#include <iostream>
using namespace std;

/**
 * 判断是否是回文
 */
bool symm(unsigned int n) {
    unsigned int i = n;
    unsigned int m = 0;
    while (i > 0)
    {
        /* code */
        m = m * 10 + i % 10;
        i /= 10;
    }
    return m == n;
}

int main () {
    for (int i = 11; i <= 999; i++) {
        if (symm(i) && symm(i * i) && symm(i * i *i)) {
            cout << "m = " << i << endl;
            cout << "m * m =" << i * i << endl;
            cout << "m * m * m = " << i * i*i <<endl;
        }
    }
    return 0;
}
```

### 计算分段函数，并输出结果

```C++
#include <iostream>
#include <cmath>
using namespace std;

const double IINY_VALUE = 1e-10;

double tsin(double x) {
    double g = 0;
    double t = x;
    int n = 1;
    do {
        g += t;
        n++;
        t = -t * x * x/(2*n-1)/(2 * n -2);
    } while (fabs(t) >= IINY_VALUE);
    return g;
}

int main() {
    double k,r,s;
    cout << "r =";
    cin >> r;
    cout << "s=";
    cin >> s;
    if (r * r <= s * s) {
        k = sqrt(tsin(r) * tsin(r) + tsin(s) * tsin(s));
    } else {
        k = tsin(r * s) /2;
    }
    cout << k <<endl;
    return 0;
}
```

### 摇筛子游戏

```C++
#include <iostream>
#include <cstdlib>
using namespace std;

enum gameStatus {WIN,LOSE,PLAYING};

int rollDice() {
    unsigned int rand1 = rand() % 6 + 1;
    unsigned int rand2 = rand() % 6 + 1;
    cout << "rand1 + rand2 = " << rand1 + rand2 << endl;
    return rand1 + rand2;
}


int main() {
    unsigned int seed, sum, myPoint;
    gameStatus status;
    cout << "enter is seed:";
    cin >> seed;
    srand(seed);
    sum = rollDice();
    switch (sum)
    {
    case 7:
    case 11:
        status = WIN;
        break;
    case 2:
    case 3:
    case 12:
        status = LOSE;
        break;
    default:
        status = PLAYING;
        myPoint = sum;
        break;
    }

    while (status == PLAYING)
    {
        /* code */
        sum = rollDice();
        if (sum == 7) {
            status = LOSE;
        } else if (sum == myPoint) {
            status = WIN;
        }
    }

    if (status == WIN) {
        cout << "WIN";
    } else if (status == LOSE) {
        cout << "LOSE";
    }
    return 0;

}
```

