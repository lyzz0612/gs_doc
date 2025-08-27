# GS变量命名推荐

## 简介

推荐普通变量(非成员变量)、成员变量、class map 字段的命名方法。

### 普通变量(非成员变量)

一般的，使用全小写的蛇形命名法命名。

```c++
int level;
string user_name;
function save_callback;
```

### 成员变量

一般的，以一个下划线 `_` 开头标识成员变量，后面的单词使用全小写的蛇形命名法命名。

```c++
readonly int _level;
string _user_name;
function _save_callback;
```

### class map 字段

一般的，class map 的字段（也就是 `class_map` 的 key）使用全小写的蛇形命名法命名。

```c++
class CircularQueue
{
    array list = nil;
    int head = 0;
    int tail = 0;
    int size = 0;    
}
```