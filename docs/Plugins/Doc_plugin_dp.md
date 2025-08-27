---
sidebar_label: '数据处理'
sidebar_position: 8
---


# dp插件

## 概述
本插件提供一些数据处理的接口，包括压缩解压，加密解密（DES算法和Rijndael算法），


## 功能使用说明

### 压缩解压
压缩解压函数原型为：
```cpp
buffer dp.compress(string|buffer data, CompressMode mode)   // 数据压缩
// 其中mode包括：CompressMode.LZSS, CompressMode.LZ77, CompressMode.BZIP

buffer dp.decompress(buffer data)                           // 数据解压
```

举个例子：
```cpp
string key = "123456789";                                   // key
string source_str = "abc";                                  // 源数据
buffer zip = dp.compress(source_str, CompressMode.LZ77);    // 压缩
buffer raw = dp.decompress(zip);                            // 解压
string decompress_str = raw.to_string();                    // decompress_str == source_str
```


## 加密解密（DES算法）
DES算法加密解密函数原型为：
```cpp
buffer dp.des(string|buffer data, Crypt action, string|buffer key, int enc_no = 1, bool padding = false, bool cbc = false, string|buffer initial_vector = "1234567")

// 参数说明：
// data   : String or buffer to be process
// action : Crypt.ENCODE, Crypt.DECODE
// key    : 1-8bytes, 9-16bytes valid for DES3
// enc_no : 1 for DES, 3 for DES3
// padding: Padding
// cbc    : False for Electronic Code Book, True for Cipher Block Chain
// inital_vector: Valid for cbc mode
```
举个例子：
```cpp
string key = "123456789";                                   // key
string source_str = "abc";                                  // 源数据
buffer crypted = dp.des(source_str, Crypt.ENCODE, key, 1);  // 加密
buffer raw = dp.des(crypted, Crypt.DECODE, key, 1);         // 解密
string decode_str = raw.to_string();                        // decode_str = source_str
```


## 加密解密（Rijndael算法）
Rijndael算法加密解密函数原型为：
```cpp
buffer dp.rdl(string|buffer data, Crypt action, string|buffer key)

// 参数说明：
// data   : String or buffer to be process
// action : Crypt.ENCODE, DECODE
// key    : 1-16bytes
```

举个例子：
```cpp
string key = "123456789";                                   // key
string source_str = "abc";                                  // 源数据
buffer crypted = dp.rdl(source_str, Crypt.ENCODE, key);     // 加密
buffer raw = dp.rdl(crypted, Crypt.DECODE, key);            // 解密
string decode_str = raw.to_string();                        // decode_str = source_strF                                                   
```

## sha256加密算法
```cpp
buffer dp.sha256(string|buffer data)
```