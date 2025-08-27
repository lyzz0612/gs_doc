---
sidebar_label: 'GS枚举类型命名推荐'
sidebar_position: 6
---


# GS枚举类型命名推荐

## 简介

推荐几种枚举类型的命名方法。一般来说，枚举类型命名和枚举项命名使用相同的规则。

### 大驼峰命名法
枚举类型和枚举项使用大驼峰命名法：
- 单词首字母大写。

```c++
enum ServerState
{
    Unknown,
    PowerOn,
    PowerOff
}
```

### 蛇形命名法
枚举类型和枚举项使用蛇形命名法：
- 全小写，单词使用下划线 `_` 连接。

```c++
enum server_state
{
    unknown,
    power_on,
    power_off
}
```

### 大写蛇形命名法
枚举类型和枚举项使用大写蛇形命名法：
- 全大写，单词使用下划线 `_` 连接。

```c++
enum SERVER_STATE
{
    UNKNOWN,
    POWER_ON,
    POWER_OFF
}
```