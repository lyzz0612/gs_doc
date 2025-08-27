---
sidebar_label: '标准IO'
sidebar_position: 1
---

# 标准输入输出

介绍常用的输入输出方法。

## 格式化IO

`printf`类似于C语言中的格式化输出，根据格式化字符串，对可变参数列表进行格式化。

`broadcast`可以将消息广播到所有设备，它同样也支持格式化输出。

```bash
Shell> printf("%O", [1, 2]);
[ /* sizeof() == 2 */
    1,
    2,
]
Shell> printf("%M\n", [1, 2]);
[1,2]

Shell> object obj = load_static("/test.gs");
Shell> printf("%O", obj);
object[32:v26]/test.gs<Static>
Shell> broadcast("%ip", 123456789);
7.91.205.21
```

`sprintf`将格式化输出以字符串返回。

```bash
Shell> string str1 = "hello";
Shell> int year = 2017;
Shell> string str2 = "world";
Shell> string str = sprintf("%s%d%s", str1, year, str2); // str = "hello2017world"
```

:::tip

GS的格式化输出部分细节与C语言中不同：

+ 几个扩展的格式化说明符 `%O` `%H` `%M`等。

  + `%O`：任意类型格式化，输出详细格式
  + `%M`：任意类型格式化，输出紧凑格式
  + `%H`：格式化handle类型
  + `%ip`：ip地址格式化

+ 不支持转换浮点数为十进制指数计数法 `%E` `%e`。

+ 不支持转换浮点数为十六进制记法 `%A` `%a`。

+ 格式化浮点数输出`%g` 等效于`%f` , 不会转换为十进制指数计数法输出。

+ 字符串格式化输出中无截断操作
  
  ```cpp
  printf("%2s", "Hello"); // 结果为 “Hello” 而不是 “He”
  ```
  
+ `.`  dot类拓展整型时默认补空格而不是数字0
  
  ```cpp
  printf("%.5d", 1); // 结果为 “    1” 而不是 “00001”
  ```

+ 方便的输出颜色控制
  
  ```cpp
  printf(HIY "RED" NOR); // 输出为红色
  ```

+ 居中输出控制符 '|'
  
  ```cpp
  printf(".%|11s.", "Hello"); //输出结果为 ".   Hello   ."
  ```

+ 左对齐输出控制符 '-'
  
  ```cpp 
  printf(".%-11s.", "Hello"); //输出结果为 ".Hello      ."
  ```
  
+ 浮点型格式化输出不支持小数精度为0， 不会进行4舍5入
  
  ```cpp
  printf("%.0f", 1.5); //输出结果为 "1.5" 而不是 "2"
  ```

:::

## write

write作为一种设备输出方式，不需要格式化字符串，可以输出任意类型值，且支持可变参数：

```bash
Shell> string val1 = "G-bits";
Shell> int val2 = 2017;
Shell> write(val1, "to", val2, 1234);
G-bitsto20171234
```

## input_string

`input_string`支持按行读取标准输入：

```bash
Shell> import gs.lang.io;
Shell> string name = input_string("Your name: ");
Your name: Zhang3

Shell> printf("Hello, %s\n", name);
Hello, Zhang3

```

## sscanf

`sscanf`根据格式化字符串从指定字符串中提取内容，返回含有提取到的内容的`array`，其中第一个元素为格式化字符串中`%`说明符的个数。GS还增加了另外一种匹配方式%\*d, %\*s等等，\*号表示任意个。这种匹配方式不会映射到返回数组当中，而是舍弃这个匹配，但是返回数组第一个参数会+1。

```bash
Shell> array arr1 = sscanf("123abc456", "%dabc%d"); // arr1 = [2, 123,456]
Shell> // 带*号匹配
Shell> array arr2 = sscanf("g1b2i3ts", "g%db%*si%cts"); // arr2 = [3, 1, 51], 3对应的ASCII为51
Shell> array arr3 = sscanf("g1b2i3ts", "g%db%*s%d"); // arr2 = [3, 1, 2]， %*s匹配到0个字符
```

:::tip

sscanf与c的实现也有一些区别:

+ 返回值为接收参数的数量与参数构成的数组，不需要额外的输入用于接收参数的变量。

+ 不支持如 %9s 此类最多有9个非空白符的字符串匹配的限制描述

  ```cpp
  sscanf("aaaa", "%2s"); // 报错
  ```

+ 整型溢出后结果不是默认值-1，而是保留溢出值

  ```cpp
  sscanf("9223372036854775808", "%d"); // 结果为 -9223372036854775808
  ```

+ 不支持如 %3[0-9] 类的匹配至多3个十进制数字的字符串的描述

  ```cpp
  sscanf("4444", "3[0-9]"); // 报错
  ```

+ 不以空格作为匹配字符串的默认分隔符

  ```cpp
  sscanf("aa bb", "%s"); // 结果为 "aa bb" 而不是 "aa"
  ```

+ 当需要匹配`%`时采用`%`作为转义字符
  
  ```cpp
  sscanf("%aa", "%%%s"); // 结果为 "aa"
  ```

+ `%*`表示匹配任意个并抛弃，但记录的匹配数会+1

  ```cpp
  sscanf("aa:11", "%*s:%d"); // 结果为 11 ，但记录的匹配数为2
  ```

:::
