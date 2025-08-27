---
sidebar_label: 'GS常量、宏定义命名推荐'
sidebar_position: 7
---


# GS常量、宏定义命名推荐

## 简介

推荐常量、宏定义的命名方法。一般的，常量使用全大写或者全小写的蛇形命名法；宏定义使用全大写的蛇形命名法。

### 常量全大写蛇形命名

```cpp
const string HELLO_WORLD = "hello world";
const int MAX_USER_COUNT = 100;
const int WEEKS = 10;
```

### 常量全小写蛇形命名

```cpp
const string hello_world = "hello world";
const int max_user_count = 100;
const int weeks = 10;
```

### 宏定义全大写蛇形命名

```cpp
#define TRACE(...)
#define TRACE_INFO(...)
```