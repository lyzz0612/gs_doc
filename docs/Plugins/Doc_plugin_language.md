---
sidebar_label: '编码转换'
sidebar_position: 4
---


# language - 编码转换插件

## 概述
本插件主要为程序转换编码。

## 功能使用说明
language为编码转换插件，其功能有：  
1. 提供utf8字符串相关的操作接口  
2. 提供输入/输出编码转换的接口  

### utf8字符串操作
```cpp
array language.break_utf8(string utf8_str)

string language.combine_utf8(array unicodes)

int language.strwidth(string str, string encoding)      // Get the width of string by specified encoding
```


### 设置输入/输出编码转换的接口  

获取，设置内部编码函数原型为：
```cpp
string language.get_internal_encoding()                 // 获取内部编码

string language.set_internal_encoding(string encoding)  // 设置内部编码
```

举个例子：
```cpp
language.set_internal_encoding("ASCII");

string inter_encode = language.get_internal_encoding(); // inter_encode = "ASCII"
```

设置输入输出编码转换函数原型为：
```cpp
void language.set_io_translator(handle fd, mixed internal_encoding, string? external_encoding = nil)
/* Since the terminal may has other encoding different with internal, so
 * set the translator of an IO is useful.
 * When read, the data should translate from external to internal;
 * When write, the data should translate from internal to external
 * Remove the translator if internal_encoding == 0
*/
```

举个例子：
```cpp
language.set_io_translator(coroutine.get_output(), "UTF-8", "GBK"); // internal = "UTF-8", external = "GBK"
```


iconv类似于Linux平台上的iconv，它是用来转换文件的编码方式的，原型为：
```cpp
mixed language.iconv(string|buffer text, string to_code, string from_code)
```
举个例子：
```cpp
array lines = [];
string str = "";
while (str = fd.gets())
{
    str = language.iconv(str, "gbk", "utf-8");      // 编码转换，从utf-8转换成gbk
    lines << [str];
}
```