---
sidebar_label: 'GS函数命名、书写格式'
sidebar_position: 5
---

# GS命名和书写格式推荐

## 简介

展示一下函数的命名、函数参数的命名和定义时的书写格式。

## 函数命名以及书写格式

- 一般情况下，函数和函数参数命名都使用全小写的蛇形命名法：

```c++
void boot()
{
}

public bool register_hook(string op, function func, bool call_if_exist = false)
{
}
```

- 在书写格式上，函数参数之间用空格隔开：

```c++
public bool register_hook(string op, function func, bool call_if_exist = false)
{
}
```