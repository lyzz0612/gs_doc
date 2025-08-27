---
sidebar_label: 'GS关键字、运算符书写格式'
sidebar_position: 2
---

# 简介
这里主要是展示 GS 关键字、运算符等等的书写格式。

## 关键字

### if
```cpp
if (arr.is_empty())
    arr.push_back(1);

if (arr.is_empty())
{
    arr.push_back(1);
    arr.push_back(2);
}
```

### if else
```cpp
if (arr.is_empty())
    arr.push_back(1);
else
    arr.clear();

if (arr.is_empty())
{
    arr.push_back(1);
    arr.push_back(2);
}
else
{
    arr.clear();
    arr.shrink();
}
```

### if else if
```cpp
if (arr.length() > 64)
{
    arr.resize(64);
}
else if (arr.length() > 32)
{
    arr.resize(32);
}
else if (arr.length() > 16)
{
    arr.resize(16);
}
else
{
    arr.clear();
    arr.shrink();
}
```

### switch case
```cpp
switch (value)
{
case 1:
    arr.push_back(1);
    break;

case 2:
    arr.push_back(2);
    break;

case 3:
    arr.push_back(3);
    return;

case 4:
    arr.push_back(4);
    break;

default:
    arr.push_back(value);
    break;    
}
```

### for
```cpp
for (int i = 1 upto 999)
    arr.push_back(i * 10);

for (int i = 1; i < 999; i++)
{
    arr.push_back(i * 10);
}

for (;;)
{
    coroutine.sleep(0.01);
}

for (int value : arr)
{
    total += value;
}
```

### while
```cpp
while (arr.length() > 0)
{
    arr.clear();
    arr.shrink();
}

while (arr.is_empty())
    arr.push_back(1);
```

### do while
```cpp
do
{
    arr.push_back(arr.length());
    arr.push_back(arr.length() + 1);
} while (arr.length() < 32);
```

### defer
```cpp
queue q = queue.create();
defer q.close();

array arr = nil;
defer {
    if (arr != nil)
        arr.clear();
}
```

### catch
```cpp
catch(arr.clear());

catch
{
    arr.clear();
    arr.shrink();
}
```

### try catch
```cpp
try
{
    error("test\n");
}
catch
{
    printf("error happen\n");
}

try
{
    error("test\n");
}
catch (err)
{
    printf("error: %M\n", err)
}
```

### try_lock
```cpp
try_lock(share_value1)
{
    //
}

try_lock(share_value1, share_value2)
{
    //
}

try_lock(read: share_value1)
{
    //
}

try_lock(read: share_value1, share_value2)
{
    //
}
```

### let
```cpp
let int a, int b = [ 1, 2 ];
int a, b;
let a, b = [ 1, 2 ];
```

`let` 是GS的一个语法糖，用于从数组中解构赋值。可以快速从具有多个返回值的函数中提取结果。

目前 `let` 的规则是：

* 当从非数组或者nil中解构时，抛出异常
* 当从空数组中解构时，抛出异常
* 当从数组中解构时，按照数组的长度进行解构，如果数组长度小于解构变量的数量，则剩余变量赋值为nil

考虑到空数组/长度不足时的行为有些诡异，后续版本的driver会进行调整；目前建议使用 `let` 时，尽可能确保数组和解构变量的数量一致。

### enum
```cpp
enum ServerState
{
    Unknown,
    Online,
    Offline
}
```

### class
```cpp
class Queue
{
    array list = nil;
    int size = 0;
    
    void __new(Queue self, int size)
    {
        self.list = array.allocate(size);
        self.size = size;
    }
}

parallel class Queue
{
    array list = nil;
    int size = 0;

    void __new(Queue self, int size)
    {
        self.list := make_parallel(array.allocate(size));
        self.size := size;
    }
}
```

## 运算符

### `!`
```cpp
bool flag = ! arr.is_empty();
if (! arr.is_empty())
if (! flag)
```

### `~`
```cpp
int value1 = ~0x7FFF;
int value2 = ~value1;
```

### `++`
```cpp
int n = 1;
n++;
++n;
```

### `--`
```cpp
int n = 1;
n--;
--n;
```

### `||`
```cpp
bool flag1 = (! arr1.is_empty()) || (arr2.length() > 32);

// 条件一行写的下时
if (flag1 || flag2)
{
}

// 条件一行写不下时
if (flag1 || flag2 ||
    flag3 || flag4)
{
}
```

### `&&`
```cpp
bool flag1 = (! arr.is_empty()) && (arr.length() > 32);

// 条件一行写的下时
if (flag1 && flag2)
{
}

// 条件一行写不下时
if (flag1 && flag2 &&
    flag3 && flag4)
{
}
```

### `+` 和 `+=`
```cpp
int a = 1 + 2;
a += 10;
```

### - 和 -=
```cpp
int a = 3 - 1;
a -= 10;
```

### `*` 和 `*=`
```cpp
int a = 2 * 3;
a *= 10;
```

### `/` 和 `/=`
```cpp
int a = 100 / 5;
a /= 5;
```

### `%` 和 `%=`
```cpp
int a = 100 % 6;
a %= 3;
```

### `>`
```cpp
bool flag = a > 0;
if (a > 0)
```

### `>=`
```cpp
bool flag = a >= 0;
if (a >= 0)
```

### `<`
```cpp
bool flag = a < 0;
if (a < 0)
```

### `<=`
```cpp
bool flag = a <= 0;
if (a <= 0)
```

### `==`
```cpp
bool flag = a == 0;
if (a == 0)
```

### `!=`
```cpp
bool flag = a != 0;
if (a != 0)
```

### `=`
```cpp
int a = 10;
int b = a;
a = 11;
b = a;
```

### `^`
```cpp
int a = 0 ^ 1;
```

### `|`
```cpp
int a = 0x01 | 0x10;
```

### `&`
```cpp
int a = 0x01 & 0x11;
```

### `<<`
```cpp
int a = 1 << 16;

map m1 = {};
m1 << m2;
```

### `>>`
```cpp
int a = 0xFFFF << 8;
```

### `??`
```cpp
string a = b ?? "hello world";
```

### `? :`
```cpp
int a = b > 0 ? b : -b;
```