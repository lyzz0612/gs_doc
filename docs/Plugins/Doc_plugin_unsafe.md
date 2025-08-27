---
sidebar_label: 'unsafe'
sidebar_position: 11
---


# unsafe -- Unsafe type convertion & operation

## 概述
主要应用于与其他语言交互（ffi）时的类型转换和操作

## 接口

```cpp
// Convert cstr to gs string
string cstr_to_string(int ptr);
```

```cpp
// Peek memory as bytes (only used for return value of function in dynamic lib)
buffer data_to_buffer(int ptr, int len);
```

```cpp
// Put int64 from a buffer
void memcpy(int dest, int src, int len);
```

```cpp
// FFI module must be initialized before this module
mixed ref_cstruct(int id, int ptr);
```