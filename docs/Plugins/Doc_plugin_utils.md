---
sidebar_label: '常用操作'
sidebar_position: 12
---


# utils - 常用操作插件

## 概述
本插件提供了一个基于MD5算法（Message Digest Algorithm 5）的散列计算函数。  

MD5中文名为信息摘要算法第五版，主要用于确保信息传输完整一致，提供信息完整性保护。

## 功能使用说明

```cpp
buffer md5(string|buffer val|constexpr);   // Calculate the md5 of input data
```

提供计算传入的string或buffer的md5值的功能。底层同时提供了编译时常量函数版本和运行时外部函数版本。
如果传入的是常量，则会在编译时计算，如果传入的是变量，则在运行时计算,如:
```cpp
md5("sdf");           // 此方法在编译时计算
md5(get_exec_path()); // 此方法在运行时计算
```

举个例子：
```cpp
string str = "xxx", md5_str = "";
md5_str = md5("sdf").to_hex();   // md5_str = "d9729feb74992cc3482b350163a1a010");
md5_str = md5(str).to_hex();     // md5_str = "f561aaf6ef0bf14d4208bb46a4ccb3ad");
buffer buf = (buffer)str;               // md5(buf) == md5(str)
```