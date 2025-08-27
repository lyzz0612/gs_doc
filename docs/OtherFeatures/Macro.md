---
sidebar_label: 'GS中的宏'
sidebar_position: 3
---

# gs中的宏

本文介绍gs中的常用宏。所有宏均区分大小写，且都以双下划线开头，双下划线结尾。
假设本gs文件 `test.gs` 位于路径 ` ./test_scripts/path/test.gs`中,  /r 设置的根目录为 `./test_scripts`， 当前宏位于第 3 行,  在 `test_func` 函数中。


#### \_\_FILE\_\_

`__FILE__` 宏展开后为相对于 /r 设置的根路径的当前文件路径 + 当前文件名。

参照假设，宏展开结果为`/path/test.gs`。


#### \_\_PURE_FILE\_\_
`__PURE_FILE__` 宏展开后为当前文件的文件名。

参照假设，宏展开结果为`test.gs`。

#### \_\_PURE_FILE_NAME\_\_
`__PURE_FILE_NAME__` 宏展开后为当前文件的文件名（不含扩展名）。

参照假设，宏展开结果为`test`。

#### \_\_CLASS_NAME\_\_

在一个类的作用域中，`__CLASS_NAME__` 宏展开后为当前类的名称：

```cpp
class Example
{
	string msg = __CLASS_NAME__;
}

writeln(Example.msg); // 输出 "Example"
```

#### \_\_DIR\_\_
`__DIR__`  宏展开后为相对于 /r 设置的根路径的当前文件路径。

参照假设，宏展开结果为`/path/`。


#### \_\_LINE\_\_
`__LINE__` 宏展开后为当前所在的行号。

参照假设，宏展开结果为`3`。


#### \_\_LOCATION\_\_
`__LOCATION__`宏展开后相当于 `__FILE__ + ":" + __LINE__`

参照假设，宏展开结果为`/path/test.gs:3`。


#### \_\_FUN\_\_
`__FUN__`宏展开后为当前所在函数的名字。

参照假设，宏展开结果为`test_func`。


#### \_\_SRC_PREFIX\_\_
`_SRC_PREFIX__`行展开后相当于 `"#pragma location " + __FILE__ + ":" + __LINE__`

参照假设，宏展开结果为`#pragma location /path/test.gs:3`。


#### \_\_COUNTER\_\_
`__COUNTER__`宏展开后为在此之前的`__COUNTER__`的数量。 

如下代码:
```cpp 
write(__COUNTER__);
write(__COUNTER__);
```
展开后为:
```cpp 
write(0);
write(1);
```


#### \_\_result\_\_
`__result__` 严格来说是一个语法糖， 其专用于处于非entry函数的defer中，用于获取当前函数的返回值。

如下代码:
```cpp 
int test_ret()
{
	defer {write(__result__)};
	reutrn 1;
}
```
其中write会输出 test_ret 函数的返回结果， 即为 1, 其实际作用相当于在 test_ret 函数中插入了一个新的`__result__`变量， 并在return时给变量赋值，语法糖效果如下：

```cpp 
int test_ret()
{
	mixed __result__ = nil;
	defer {write(__result__)};
	reutrn __result__ = 1;
}
```
