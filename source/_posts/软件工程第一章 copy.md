---
title: 软件工程第一章
date: 2021-07-18 10:12:47
tags: ['软件工程']
category: 软件工程
article: 软件工程第一章
---

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