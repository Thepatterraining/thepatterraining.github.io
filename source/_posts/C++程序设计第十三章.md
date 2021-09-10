---
title: 学堂在线C++程序设计第十三章学习笔记
date: 2021-08-31 10:12:47
tags: ['C++']
category: C++
article: 学堂在线C++程序设计第十三章学习笔记
---

# 学堂在线C++程序设计第十三章学习笔记

## 流类库与输入输出

### IO流的概念

当程序与外界环境进行信息交换时，存在着两个对象，一个是程序中的对象，另一个是文件对象

`流`是信息流动的抽象，负责在数据的`生产者`和`消费者`之间建立联系，并管理数据的流动

流对象与文件操作
- 建立流对象
- 指定这个流对象与某个文件对象建立连接
- 程序操作流对象
- 流对象通过文件系统对所连接的文件对象产生作用

提取与插入
- 读操作从流中提取
- 写操作向流中写入

### 输出流

最重要的三个输出流
- ostream
- ofstream
- ostringstream

预先定义的输出流对象
- cout 标准输出
- cerr 标准错误输出，没有缓冲，发送给它的内容立即被输出
- clog 类似cerr，但是有缓冲，缓冲区满时被输出

```C++
ofstream fout('123.log');
streamBuf* old = cout.rdbuf(fout.rdbuf());
//cout 输出到123.log文件
cout.rdbuf(old); //恢复输出到标准输出
```

构造输出流对象
- ofstream 类支持磁盘文件输出
- 如果在构造函数中指定文件名，当构造这个文件时该文件时自动打开的
    ofstream fout('123.log');
- 可以在调用默认构造函数后使用open打开文件
    ofstream fout;
    fout.open(123.log);
- 打开文件可以指定模式
    ofstream fout('123.log', ios_base::out|ios_base::binary);

文件输出流成员函数的三种类型
- 与操纵符等价的成员函数
- 执行非格式化写操作的成员函数
- 其他修改流状态且不同于操纵符或插入运算符的成员函数

成员函数
- open
    - 把流与磁盘文件关联
    - 需要指定打开模式
- put
    - 把一个字符写到输出流
- write
    - 将内存中的一块内容写到一个文件输出，可以写二进制
- seekp和 tellp
    - 操作文件流的内部指针
- close
    - 关闭磁盘文件
- 错误处理函数
    - 在写到一个流时进行错误处理

#### 向文本文件输出

插入运算符 <<
- 为所有标准C++数据类型设计的，用于传送一个字节到输出流对象

操纵符（manipulator）
- 插入运算符和操纵符一起
    控制输出格式
- 很多操纵符都定义在 ios_base类中，如hex()，<iomanip>头文件中，如setprecision()
- 控制输出宽度
    在流中放入setw操纵符或调用width成员函数
- setw和 width只影响紧随其后的输出项，但其他流格式操纵符保持有效直到发生改变
- dec,oct和 hex操纵符设置输入输出的默认进制

精度
- 浮点数输出精度的默认值是6
- 改变精度使用 setprecision操纵符，定义在 <iomanip> 头文件中
- 如果不指定 fixed 或 scientifc,精度值表示有效数字位数
- 如果设置了 ios_base::fixed 定点形式 或 ios_base::scientifc 科学计数法 ，精度值表示小数点之后的位数

#### 向二进制文件输出

二进制文件流
- 使用ofstream构造函数中的模式指定二进制
- 以通常方式构造一个流，然后使用setmode成员函数，在文件打开后改变模式
- 通过二进制文件输出流对象完成输出

```C++
int main() {
    int b[3] ={912,2456,37890};
    ofstream file("date.bat", ios_base::binary);
    file.write(reinterpret_cast<char *> (&b), sizeof(b));
    file.close();
    return 0;
}
```

#### 向字符串输出

- 用于构造字符串
- 功能
    - 支持ofstream类的除了open,close外的所有操作
    - str函数可以返回当前已构造的字符串
- 典型应用
    - 将数值转换为字符串

```C++
ostringstream os;
os << 123;
string s = os.str();
```

### 输入流

重要的输入流类
- istream 最适用于顺序文本输入，cin是他的实例
- ifstream 支持磁盘文件输入
- istringstream 字符串输入

构造输入流对象
- 如果在构造函数中指定文件，在构造对象时自动打开
    ifstream ifs('data.bat');
- 调用默认构造函数后使用open打开
    ifstream ifs;
    ifs.open('data.bat');
- 打开文件时可以指定模式

输入流相关函数
- open 把输入流和文件关联
- get 函数的功能与提取运算符(>>)很像，主要的不同点是get函数在读入数据时包括空白字符
- getLine 的功能是读取多个字符，并且可以指定终止字符，读取完成后，从读取内容中删除终止字符
- read 函数从一个文件读字节到内存中的指定区域，由长度确定要读的字节数。当遇到文件结束或在文本模式文件中遇到文件结束标记字符时结束读取。可以读取二进制。

```C++
int main() {
    int b[3] ={912,2456,37890};
    ofstream file("date", ios_base::binary);
    file.write(reinterpret_cast<char *> (b), sizeof(b));
    file.close();
    int i2;
    ifstream mfile("date", ios_base::binary);
    mfile.seekg(2 * sizeof(int));
    mfile.read(reinterpret_cast<char *>(&i2), sizeof(int));
    cout << i2 <<endl;
    mfile.close();
    return 0;
}
```

#### 字符串输入

用于从字符串读取数据


#### 输入输出流

输入输出流都是从iostream派生的
- fstream
- stringstream


