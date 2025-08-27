---
sidebar_label: 'GS对象脚本布局推荐'
sidebar_position: 3
---

## 简介

推荐对象脚本内的 `#pragma` 列表、`component` 列表、`import` 列表、`#define` 列表、`enum` 列表、`const` 列表、成员变量列表、构造函数、析构函数的布局。

## 布局

- 布局顺序按先后如下：
    1. 预处理列表 `#pragma`
    2. `component` 列表
    3. `import` 列表
    4. `#define` 列表
    5. `enum` 列表
    6. `const` 列表
    7. 成员变量列表
    8. 构造函数
    9. 析构函数

## 示例

以下仅是演示布局：

```c++
// 预处理
#pragma parallel
#pragma disable_debug
#pragma default_object
#pragma export_macros
#pragma export_imports

// 组件
component engine.components.FEntity;
component pkg.common_event.FCommonEvent;

// import 列表
import engine.import;
import engine.mods.net;
import gs.bin.msgpack;
import pkg.util_lib;

// 宏定义
#define TRACE(...)
#define TRACE_INFO(...)
#define TRACE_WARN(...)

// 枚举列表
enum ServerState
{
    Unknown,
    Online,
    Offline,
}

// 常量
const int SCAN_INTERVAL = 60;
const int MAX_COUNT = 100000;

// 成员变量
readonly int _total_count := 0;
readonly map _scan_obs := make_readonly({});

// 构造函数
void create()
{
}

// 析构函数
void destruct()
{
}
```
