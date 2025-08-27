---
sidebar_label: 'component'
sidebar_position: 2
---

# component

`component`是组件的, 一个类作为,另外一个类的补充.

主类可以调用组件类的`public`函数和`protected`函数.

可以在组件类声明**虚函数**, 然后在主类中实现该函数, 在主类中可以可以通过`base`来调用**组件类**的虚函数实现.

PS: 不存在纯虚函数，一般在虚函数声明中直接抛出异常作为替代。

### 虚函数

虚函数是可以被覆写的函数。虚函数可以是 `public` 或 `protected` 的。覆写时必须使用相同的访问修饰符（即 `public` 虚函数必须用 `public` 覆写，`protected` 虚函数必须用 `protected` 覆写）。


## 简单例子

```cpp title="bar.gs"

string world = "world";

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

public virtual string foo_bar()
{
    write ("bar\n");
}

```

```cpp title="foo.gs"
component  .bar;
public string foo()
{
    return "Hello ";
}

public override void bar.foo_bar()
{
    write(foo());
    base.foo_bar();                 // 调用 bar.gs中的实现
}

write(foo(), bar());

foo_bar();          // 输出 Hello World

// bar1();          // error, 默认函数为private
// bar.bar2();      // error, 无法访问private函数
bar.bar3();         // OK, 可以访问protected函数
// bar.world;       // error, 不可以访问其他脚本中的成员变量
```

## component路径

component的路径规则和[import](./Import)完全一致.
