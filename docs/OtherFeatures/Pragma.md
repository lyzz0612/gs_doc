---
sidebar_label: 'pragma（编译注记）'
sidebar_position: 1
---

## pragma (编译注记)

与C/C++的pragma预处理指令类似，GS也有一些pragma指令，例如：

```cpp
#pragma parallel

void foo()
{

}
```

`#pragma parallel` 是一条常用的预处理命令，作用是让当前脚本实例化出的对象是一个parallel对象。调用对象的方法不需要跨域，同时也限制了这个对象不能有非parallel/readonly的成员变量。

除此之外，GS还有其他预处理指令，名称和功能如下表所示：

名称|功能描述|备注
---|---|---
parallel|指示这个脚本对应的对象是parallel的|-
default_object|这个脚本中可以定义全局可见的函数/枚举|default_object与parallel应同时存在
disable_debug|使得调试器无法调试这个program|-
enable_handle_index|允许从object中直接获取成员|仅限调试器附加时使用
disable_error_trace|阻止回溯调用堆栈，在报错时不会打印此对象中的调用栈|-
export_imports|其他脚本import此脚本时，会同时import当前脚本import的其他脚本|-
export_macros|其他脚本import此脚本可以使用本脚本中定义的宏|-
see_all_import|使得词法分析器能够观察到所有program，而不仅局限于import的|-
auto_declare_var|允许使用变量前不需要进行声明|不推荐使用
env_object_func|允许调用shell中定义的函数|不推荐使用
env_object_var|允许访问shell中定义的变量|不推荐使用
disable_jit|指定脚本不会被jit编译，保留使用解释执行|-
enable_cross_domain_warn|如果当前函数是被跨域调用的，那么写入和读取成员变量时发生运行时警告|-
location|为当前脚本重新指定名称和行号，行为类似C/C++的#line|*注释[1]

------
## 注释

[1] location 的用法如下所示
```cpp
//test.gs

public void foo1()
{
    error("Error happend!");
}

public void foo2()
{
#pragma location example.gs : 1234
    error("Error happend!");
}

```
分别执行foo1和foo2，会发现报错时，foo2位置的调用栈显示的文件名和行号变成了"example.gs"的第1235行。
