---
sidebar_label: 'import'
sidebar_position: 1
---

# import

`import` 用于引入一个新的模块或者插件，其本质是一种语法糖。如下代码所示，下面的两个代码是**基本**等价的（细节上略有差别，假定`example.gs`放在指定的根目录下）：

```js
import example;
example.func();
```

**基本**等价于

```js
"/example.gs".func(); 
```

`import` 在导入时，会按照一定规则查找到对应的文件路径，并得到一个等价于字符串常量（内容为导入文件的路径，以`'/'`开头）的标识符。也正因为此，如下的代码是可以工作的。

```js
import example;

//...
object obj_1 = new_object(example, this_domain());
// example 是一个字符串常量，其内容为"/example.gs"

//...
```

当调用模块中的函数时，模块会发生 load_static。

```js
// example.gs
void create()
{
    printf("example Instance.\n");
}

public string func()
{
    return "Helloworld";
}

// src.gs
import example;

printf("Now test: %s\n" ,example);

printf("%s\n", example.func());
// 此处等效于
// printf("%s\n", load_static(example, this_domain()).func());

printf("%s\n", example.func());
```

运行`src.gs`得到：

```
Now test: /example.gs
example Instance.
Helloworld
Helloworld
```

总而言之，利用`import`，可以在另外的脚本中封装一些操作，将之整体`import`进来，然后调用其中的方法。

## import别名

`import import_name` 在导入模块时，可在其后添加 `as alias_name`,其将 alias_name 视为 import_name 的别名，除部分别名冲突的情况下，两者的使用是等效的。如下代码所示:

```js
import example as example_alias;// example_alias 作为 example的别名
example.func();
example_alias.func(); //等效于 example.func()
```

:::info

**当alias_name与已有 变量，import_name, 关键字，其他别名 冲突时，其表现如下:**

* 与已有变量名冲突时，该冲突别名在变量的作用域内被视为变量，作用域外则可正常作为别名使用。

```js
import example as example_alias;// example_alias 作为 example的别名
if(true)
{
	int example_alias = 0;
	example.func();
	example_alias = 1; //在变量作用域内example_alias为int变量
}
example.func();
example_alias.func(); //在变量作用域外example_alias正常作为别名
```

* 与已有的 import_name 冲突，使用该冲突名称时，报错"Ambiguous import name ..."

```js
import example_alias;
import example as example_alias;// example_alias作为别名与已导入的模块名冲突
example.func();
//example_alias.func();  使用 example_alias 时会报错"Ambiguous import name ..."
```

* 与已有关键字冲突时，关键字的优先级更高，此时别名无法作为import_name 使用。

```js
import example as while;
import example1 as if;

// while.func(); while 无法作为 import_name 使用
// if.func(); if 无法作为 import_name 使用
```

* 有不同文件对应相同的别名时，使用该冲突名称时会导致"Ambiguous import name"错误。


```js
import example0 as example_alias;
import example1 as example_alias;

//example_alias.func();  使用 example_alias 时会报错"Ambiguous import name 
```

:::

## 使用import导入插件（plugins）

新版gs（0.1.220125.01）将以下插件从gs内部拆离：

* archive
* debuggee
* dp
* json
* language
* msgpack
* network
* profiler
* regex
* unsafe
* utils
* xml

因此新版gs在使用这些插件中的函数时，需要import对应插件。

```js
import gs.bin.regex;  // import gs.bin.PLUGINS_NAME；

regex.match("abc", "abc");
```

如果需要兼容旧版的gs代码，可以使用以下命令行参数：

`--require PLUGINS_1;PLUGINS_2`  

导入指定的插件（注意分号在shell内的含义），或者使用：

`--require all`

导入全部内置插件

## 导出脚本

一些情况下，希望将一个脚本的import项导出出去，让别的脚本只需import一个脚本就能引入其他。

在旧版本中，类似行为可以通过 `#pragma export_imports` 来实现：

```js title="a.gs"
#pragma export_imports

import b;
import c;
import d;
```

如上所示，其他脚本import a之后，连带着也会将b、c、d导入进来。

新版本driver可以使用 export 关键字修饰，导出指定的脚本：

```js title="a.gs"
export import b;
export import c;
import d;
```

或

```js title="a.gs"
export 
{
    import b;
    import c;
}
import d;
```

如上，import a之后，同时也会导入 b和c，而d因为没有标注，所以不会导入。

需要注意：`#pragma export_imports` 和 手动标注的export是不可以同时使用的。同时使用会在编译时报告错误。

## import的路径说明
**import** 是省略路径末尾的`.gs`的

**import**的路径一般是从运行**driver**时的根目录开始算的（可通过命令行参数等强制指定）. 例如 `pkg.cjson.cjson`加载的就是`/pkg/cjson/cjson.gs`文件. 用`.`替代`/`作为文件夹划分.

若要加载的脚本文件名与所在目录相同，可以省略一层，如下所示：

```js
import pkg.cjson.json; //想要加载 /pkg/cjson/cjson.gs
```
等效于
```js
import pkg.cjson; //同样加载 /pkg/cjson/cjson.gs
```

PS: 注意`mount`导致的路径变化

### 例子
|路径|说明|
|-----|-----|
|`pkg.cjson.cjson`|加载`/pkg/cjson/cjson.gs`|
|`pkg.cjson`|对于上面的情况，也可以这样加载`/pkg/cjson/cjson.gs`|
|`pkg.cjson.*`|加载`/pkg/cjson/`下的所有`gs`文件|
|`.bar`|加载当前脚本所在目录下的`bar.gs`|
|`..bar`|加载当前脚本所在目录的父目录下的`bar.gs`|
