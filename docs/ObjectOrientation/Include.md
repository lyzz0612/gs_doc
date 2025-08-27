---
sidebar_label: 'include'
sidebar_position: 3
---

# #include

`#include`的作用和`c++`中的`#include`完全相同. 在编译阶段将该语句替换为对应文件内容.

```cpp title="bar.gs"
public string bar()
{
    return "World";
}

string bar1()
{
    return "World";
}

private string bar2()
{
    return "World";
}

protected string bar3()
{
    return "World";
}
```

```cpp title="foo.gs"
#include  "bar.gs"

public string foo()
{
    return "Hello ";
}

write(foo(), bar(), "\n");          // collect
write(foo(), bar1(), "\n");         // collect
write(foo(), bar2(), "\n");         // collect
write(foo(), bar3(), "\n");         // collect
```

foo.gs 等同于:

```cpp title="foo.gs"
public string bar()
{
    return "World";
}

string bar1()
{
    return "World";
}

private string bar2()
{
    return "World";
}

protected string bar3()
{
    return "World";
}

public string foo()
{
    return "Hello ";
}

write(foo(), bar(), "\n");          // collect
write(foo(), bar1(), "\n");         // collect
write(foo(), bar2(), "\n");         // collect
write(foo(), bar3(), "\n");         // collect

```

## include的路径说明

**include** 是不能省略末尾的`.gs`的

**include**的路径一般是从运行**driver**时的根目录开始算的. 例如 `/pkg/cjson/cjosn`加载的就是`/pkg/cjosn/cjson.gs`文件.
PS: 注意`mount`导致的路径变化

### 例子
|路径|说明|
|-----|-----|
|`/pkg/cjson/cjson.gs`|加载`/pkg/cjosn/cjson.gs`|
|`./bar.gs`|加载当前文件夹下的`bar.gs`|
|`../bar.gs`|加载当前文件夹的父文件夹下的`bar.gs`|
