---
sidebar_label: 'msgpack'
sidebar_position: 9
---


# msgpack - 高效二进制序列化格式pack/unpack

## 概述
[MessagePack](https://msgpack.org/)简介：高效的二进制序列化格式。它让你像 JSON 一样可以在各种语言之间交换数据。但是它比 JSON 更快、更小。


## 功能使用说明

### pack
```cpp
buffer pack(mixed val);
```

### unpack
```cpp
mixed unpack(buffer val);
```