---
sidebar_label: 'gsbind'
sidebar_position: 16
---

# gsbind

## 概述

`gsbind` 不是一个插件，`gsbind`是一个轻量级的头文件库，用于支持以非`ffi`的方式进行`gs`向`C/C++`的跨语言函数调用。

> **提示**: 
`gsbind`仅包含两个头文件，`gsbind.h` 与 `memory_mgr.h`。于项目 [Gsbind](https://m68gitlab.g-bits.com/pkgs/gsbind) 下获取。当前 `gip buildcpp` 指令流程已做平，指令执行时默认会自动拉取最新的 gsbind 头文件至 `your_pkg_name/cpp/gsbind/` 目录下。

## 对比FFI
`gsbind` 性能优势的核心在于 “将 `FFI` 的运行时成本转移到编译时”：

`gsbind` 充分利用了C++的模板和元编程特性，在编译期完成了大量传统 FFI 在运行时才执行的工作，当通过 gsbind 声明绑定代码时，这些模板会在编译时实例化并生成针对特定类型和函数的、高度优化的跨语言调用代码。

`FFI` 相对于 `gsbind` 的优势在于其不需要侵入修改 C++ 代码， 无需源码和编译，快速集成相应动态库。

`FFI` 需要在运行时动态查找函数符号、解析函数签名、确定参数类型和调用约定等，相对于 `gsbind` 具有额外开销。

**gsbind 与 FFI 对比总览表**

| 对比维度     | gsbind | FFI |
|-------------|--------|-----|
| **实现机制**     | 基于 C++ 模板元编程，**编译时生成绑定代码**，自动映射 C++ 函数到 gs efun| **运行时动态加载动态库**（`.so`/`.dll`），需手动声明函数签名与数据类型 |
| **开发体验**     | 🔧 **中等复杂度**：需编译绑定代码，但语法简洁（声明式绑定）| 🚀 **快速上手**：无需编译，直接调用现有库，适合快速原型 |
| **适用场景**     | 高性能计算, 需复杂交互（如操作gs复杂数据类型）| 调用闭源动态库，快速原型验证，轻量级跨语言集成 |
| **性能**         | ⭐⭐⭐⭐ **接近原生 C++**：调用无运行时解析开销 | ⭐⭐ **固定调用开销**：每次调用需跨语言边界，参数转换耗时|
| **可变参数函数** | ⚠️ 必须通过高级功能函数实现 | ✅ 完整支持C++可变参数函数|
| **OS_PENDING_CALL**| ✅ 支持 | ✅ 支持|
| **SAFE_CALL**| ✅ 支持 | ✅ 支持|
| **复杂结构值传递**| ✅ 支持传递struct | ✅ ffi调用支持传递struct|
| **高级功能支持** | ✅ 当前支持对 gs array/map/string/buffer 的基本操作 | ❌ 仅基础函数调用|

> **可变参数**：当前 gsbind 带有高级功能的函数可以支持 [pkg:hiredis](https://m68gitlab.g-bits.com/pkgs/hiredis/-/blob/master/src/hiredis.ffi?ref_type=heads) 中的 redisCommand 函数的可变参数功能（需额外修改cpp代码，将可变参数作为gs数组传递和处理）。

## 注意事项

> **名称要求**: 要求生成的动态库命名与 INIT_MODULE(module_name) 声明的 module_name 名称一致

> **内存管理**: gsbind 默认注册 gs 的内存管理函数

> **参数传递**: struct，class 等类型数据请以 C++ 指针传递

## 宏简介
此处简单介绍下基本的 C++ 侧 gsbind 支持必须的宏。
* INIT_MODULE(module_name)
  * 该宏会声明和定义内存管理器必要的函数或变量
  * 该宏会声明，定义并导出 module_init_##module_name 和 module_shutdown_##module_name 函数
  * 该宏以module用于注册函数的函数声明`void detail::module_def_funcs(ModuleHolder* m)`作为结尾
  
* DEFINE_FUNC(ret_type, func, name, arguments, doc, ...)
  * 该宏用于调用`void def(Func&& f, const char* name, const char* prototype, const char* doc)`添加注册函数
  * 以 `int add(int i, int j)` 函数为例, DEFINE_FUNC 应声明为 `DEFINE_FUNC(int, add, add, (int i, int j), "Add func");`
  * 注意 ret_type 和 arguments 中的类型应为gs类型而不是C++类型
  * 该宏末尾参数为可选参数 `PendingMode mode, bool is_safe_call`
  * `PendingMode mode` 默认为 `PendingMode::NORMAL`
    
    | PendingMode | 使用场景 |
    |-------------|---------|
    |PendingMode::NORMAL| 普通模式：直接调用函数，不进行任何额外处理。|
    |PendingMode::PENDING| 挂起模式：携程进入挂起状态，允许垃圾回收器（GC）在不等待当前协程的情况下进行回收。（当gsbind函数耗时较长时应使用此模式）|
    |PendingMode::LEAVE_DOMAIN| 离开域模式：携程离开当前域并进入挂起状态。（在 PENDING 基础上希望携程放开当前域，被放开的域需用于进行其他操作时）|
    |PendingMode::POST_TO_MAIN| 投递到主线程模式：将函数投递到主线程（zero）队列中执行。协程会离开当前域并进入挂起状态。（在 LEAVE_DOMAIN 基础上期望 gsbind 函数在 zero 携程执行）。|
  * `bool is_safe_call` 默认为 `false`

    | is_safe_call | 使用场景 |
    |----|----|
    |false|默认值，在调用动态库函数过程中，若程序崩溃时不进行任何处理|
    |true|在调用动态库函数过程中，若程序崩溃时尝试将崩溃作为异常抛出|

* DEFINE_FUNC_SIMPLE(ret_type, name, arguments, ...)
  * 功能同DEFINE_FUNC，会将`prototype` 参数直接作为`doc`参数传入

* DEFINE_FUNC_LAZY(name, ...)
  * 尽管使用此宏定义 gsbind 绑定函数最方便，
  * 功能同DEFINE_FUNC, 函数原型的`prototype` 参数由模板展开自动获取，`prototype` 参数直接作为`doc`参数传入



* 由于 C++ 函数类型信息中没有参数名称，以 `DEFINE_FUNC_LAZY` 定义的函数参数列表会被替换为 arg1，arg2...
* 类成员函数已支持，定义方式形如 `DEFINE_FUNC_LAZY(Class::get_a);`

## 简单样例及性能对比
在windows下开发一个简单的，仅有 `int add(int i, int j) `函数的动态库。同时为动态库支持ffi调用方式与gsbind调用方式，比较两者性能。
* 工作目录下运行 `gip new hello_gsbind -d` 指令。此指令通过 gip 使用模板新建 pkg，其会创建一个 hello_gsbind 文件夹。
* 在 /hello_gsbind/cpp/ 路径下打开终端，执行命令 `git submodule add https://m68gitlab.g-bits.com/jszx/gsbind.git`，以获取 gsbind 头文件。
* 在 /hello_gsbind/cpp/src/ 路径下修改 hello_gsinbd.cpp 文件，文件内容如下
```cpp
//hello_gsbind.cpp
#include "gsbind.h"

//Support for FFI
extern "C"
{
    GSBIND_API int add(int i, int j);
}

int add(int i, int j) {
    return i + j;
}

//Support for gsbind
//Module name is gsbind_sample
INIT_MODULE(gsbind_sample)
{
    DEFINE_FUNC_LAZY(add);
}
```
* 在 /hello_gsbind/cpp 路径下修改CMakeLists.txt，内容如下
```cpp
cmake_minimum_required(VERSION 3.16)
project(gsbind_sample)
set(CMAKE_CXX_STANDARD 14)
if (WIN32)
    set(SUFFIX_NAME "dll")
elseif (APPLE)
    set(SUFFIX_NAME "dylib")
else ()
    set(SUFFIX_NAME "so")
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/gsbind)

aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src DIR_SRCS)
add_library(${PROJECT_NAME} SHARED ${DIR_SRCS})

set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")

```
* 在 /hello_gsbind/cpp 路径下执行 `cmake .`
* 在 /hello_gsbind/cpp 打开 gsbind_sample.sln，生成 gsbind_sample.dll 动态库，将动态库移动至 /hello_gsbind/src 路径
* 在 /hello_gsbind/ 路径下执行 `code .` 打开 Vscode
* 在 /hello_gsbind/src 路径下新建 gsbind_sample.ffi 文件为如下内容（在只支持gsbind，可不撰写此 .ffi 文件）
```cpp
//gsbind_sample.ffi
module(gsbind_sample)
{
    int add(int i, int j);
}
```
* 在 /hello_gsbind/src 路径下编写gs脚本 hello_gsbind.gs 内容如下:
```cpp
// hello_gsbind.gs
// Import gsbind sample 
import .gsbind_sample;

// Import ffi sample
import builtin.ffi.gsffi;
const string dll = "/__dll/gsbind_samlpe.gs";
gsffi.load_library(dll, __DIR__ "gsbind_sample.ffi", this_domain(), this.name());
dll.add_dependent(this);

void benchmark()
{
    int start_time, end_time;
    for(int iter = 0 upto 3)
    {
        start_time = time.time_ms();
        for(int i = 0 upto 10000000)
        {
            dll.add(1,2);
        }
        end_time = time.time_ms();
        printf(HIC"FFI call add func(10M) time cost: %dms\n"NOR, end_time - start_time);
    }

    for(int iter = 0 upto 3)
    {
        start_time = time.time_ms();
        for(int i = 0 upto 10000000)
        {
            gsbind_sample.add(1,2);
        }
        end_time = time.time_ms();
        printf(HIC"Gsbind call add func(10M) time cost: %dms\n"NOR, end_time - start_time);
    }
}
benchmark();
```
* 运行 hello_gsbind.gs，运行结果如下:
```cpp
Create page memory pool, size = 0x200000000.
>>> WINPOWER attached to the console
FFI call add func(10M) time cost: 2069ms
FFI call add func(10M) time cost: 2131ms
FFI call add func(10M) time cost: 2104ms
FFI call add func(10M) time cost: 2138ms
Gsbind call add func(10M) time cost: 194ms
Gsbind call add func(10M) time cost: 194ms
Gsbind call add func(10M) time cost: 192ms
Gsbind call add func(10M) time cost: 189ms

Welcome driver shell.
GS 1.32.250427 Copyright (C) G-bits
Shell>
```

从以上内容我们可以看到gs侧函数调用效率有一个数量级的提升。

## 类型转换
目前 gsbind 支持的类型转换如下:

1. 函数参数或返回值的类型转换方式最终由C++侧的返回值或参数类型决定

2. 函数调用参数从gs类型转换为c++类型操作如下`(gs type -> c++ type)`:

    | gs 类型       | C++ 类型 |
    | -----------   | ----------- |
    |int    | int8_t(带有范围检查过程)|
    |int    | int16_t(带有范围检查过程)|
    |int    | int32_t(带有范围检查过程)|
    |int    | uint8_t(带有范围检查过程)|
    |int    | uint16_t(带有范围检查过程)|
    |int    | uint32_t(带有范围检查过程)|
    |int    | char(带有范围检查过程)|
    |int    | bool(带有范围检查过程)|
    |int    | long (带有范围检查过程)|
    |int    | unsigned long(带有范围检查过程)|
    |int    | int64_t 或 uint64_t |
    |raw_pointer|int64_t 或 uint64_t (raw_pointer.get_int_ptr)|
    |string   | int64_t 或 uint64_t (string.data_ptr)|
    |buffer|int64_t 或 uint64_t (buffer.data_ptr)|
    |float|float(带有范围检查过程)|
    |float|double|
    |float|long double|
    |int|指针类型|
    |raw_pointer|指针类型(raw_pointer.get_int_ptr)|
    |string|指针类型(string.data_ptr)|
    |buffer|指针类型 (buffer.data_ptr)|

* 当由 gs int 类型转换为 C++ 各个整型类型时，转换带有有效范围检查，当超出有效范围时值被设置为有效范围内的最大或最小值。由下图可见传入的`INT64_MAX`超过了`int32_t`的有效范围，参数被设为`INT32_MAX`, 在 +1 后溢出为`INT32_MIN`并返回。
* 当由 gs float 类型转换为 C++ 各个浮点类型时，转换带有的检查过程同上。
* 当由 gs 传递带有 struct_id 的 raw_pointer类型，且在 gsbind 中函数原型中对应参数声明为 mixed 时，转换函数会进行内存空间大小检查。

3. 函数调用返回值中 c++类型 -> gs类型转换`(c++ type -> gs type)`：

    | C++ 类型     | gs 类型 |
    | ----------- | ----------- |
    | 所有的整型或 | int |
    | 所有的浮点型 | float |
    | 所有的指针类型| raw_pointer(find subtype from registerd struct map)|
    | void | void | 

## struct 声明问题
由于 gsbind 包括转换函数的主要工作都是在 C++ 编译期间通过宏和元编程构建的，而 `gs struct` 的 `struct id` 是在启动 gs 后动态分配的，所以 gsbind 无法自动处理并设置值的 `struct id`。  
gsbind 返回的任何指针类型都会被 gsbind 自动类型默认转换设置为 `subtype` 为空的 `raw_pointer` 类型，此种情况下只能使用 `ffi.ref_struct` 手动处理 struct 转换。  
目前的解决方案是所有的 gsbind 支持的动态库都会默认注册一个 register_struct 函数，用于手动指定 `struct name` 与 `struct id` 关系，这样 gsbind 就可通过查表动态设置`struct id`。  
需通过类似如下方式转换为对应struct类型:
```cpp
// module_name 为 sample
import .sample;

// 以 struct Vector 为例
EMBED_FFI_BEGIN
struct Vector
{
    int x;
    int y;
    int z;
};
EMBED_FFI_END

// 假设 C++ 动态库有函数 Vector* get_new_vector(); , 函数返回 Vector* 指针的情况下。
void test_gsbind_return_pointer()
{
    Vector result_vec;
    raw_pointer result_raw_ptr;

    // 1. 直接使用Vector接收，隐式转换失败
    result_vec = sample.get_new_vector();

    // 2. 强制类型转换失败，语法禁止
    result_vec = (Vector)sample.get_new_vector();

    // 3. 使用raw_pointer类型接受返回值，使用 ffi.ref_struct 手动处理struct转换
    result_raw_ptr = sample.get_new_vector();
    result_vec = ffi.ref_struct(ffi.get_struct_id("Vector", __FILE__), result_raw_ptr.get_int_ptr());
}
```

这太麻烦了，目前 gsbind 的解决方案是提供默认注册函数 `void register_struct(const char* name, cmm::StructId struct_id)`。  
函数允许注册 `struct name` 向 `struct id`的映射，这样当返回`struct`指针时，gsbind 可以通过查表的方式得到正确的 `struct id`，并将返回类型正确设置, 具体用法见下样例:

```cpp
// module_name 为 sample
import .sample;

// 以 struct Vector 为例
EMBED_FFI_BEGIN
struct Vector
{
    int x;
    int y;
    int z;
};
EMBED_FFI_END

// 预先注册 struct 名称至 struct id 映射
sample.register_struct("Vector", ffi.get_struct_id("Vector",__FILE__));

// 假设 C++ 动态库有函数 Vector* get_new_vector(); , 函数返回 Vector* 指针的情况下。
void test_gsbind_return_pointer()
{
    Vector result_vec;

    // 直接使用Vector接收，gsbind查表得到正确类型 struct id
    result_vec = sample.get_new_vector();
}

```

对于 gsbind 来说由于在 C++ 编译期间无法获取的信息造成的麻烦无法避免。目前 gsbind 有一套实验性的解决方案，`gsbind.h` 中有如下宏:
* `REGISTER_STRUCT(struct_name)`
  * 该宏用于确保 C++ struct 不被优化掉，在 Centos -g 编译的情况下，dwarf 中保留有对应的 struct 信息。
  * 使用该宏请确保 Centos 编译时带有 -g 参数
以 [bind_excel](https://m68gitlab.g-bits.com/pkgs/bind_excel/-/blob/master/cpp/src/cmm_excel_export.cpp?ref_type=heads) 为例，在cpp文件中以如下方式声明需要导出的 C struct
```cpp
REGISTER_STRUCT(CellPosition)
REGISTER_STRUCT(RangePosition)
REGISTER_STRUCT(XlsBorderProp)
REGISTER_STRUCT(XlsBorders)
REGISTER_STRUCT(XlsFont)
REGISTER_STRUCT(SheetTitle)

INIT_MODULE(excel_module)
{
    DEFINE_FUNC_LAZY(init);

    DEFINE_FUNC_LAZY(shutdown);
    //... other codes
```
gsbind 项目的 tools 文件夹下有一个 [python](https://m68gitlab.g-bits.com/pkgs/gsbind/-/blob/master/tools/auto_gs_ffi_struct_generate.py?ref_type=heads) 脚本 `auto_gs_ffi_struct_generate.py`。  
将 Centos 编译好的 *.so 文件与该 python 脚本放在同一目录下。以 `excel_module.so` 为例，在该目录下运行以下命令 `python auto_gs_ffi_struct_generate.py excel_module.so`, 稍等后会有 `excel_module_structs.gs` 文件被自动生成。  
`excel_module_structs.gs` 文件将所有在 C++ 侧以 `REGISTER_STRUCT(struct_name)` 宏声明的 struct 自动解析为对应的 gs struct 并生成注册必须的 gs 代码。
文件内容如下：
```cpp
import .excel_module
EMBED_FFI_BEGIN

struct CellPosition {  // size=8
    int32_t row;  // offset=0, size=4
    int32_t col;  // offset=4, size=4
};

struct RangePosition {  // size=16
    CellPosition lt_cell_pos;  // offset=0, size=8
    CellPosition rd_cell_pos;  // offset=8, size=8
};

struct XlsFont {  // size=64
    bool has_name;  // offset=0, size=1
    bool has_size;  // offset=1, size=1
    bool has_color;  // offset=2, size=1
    bool has_family;  // offset=3, size=1
    bool has_charset;  // offset=4, size=1
    bool has_scheme;  // offset=5, size=1
    bool bold;  // offset=6, size=1
    bool italic;  // offset=7, size=1
    bool superscript;  // offset=8, size=1
    bool strikethrough;  // offset=9, size=1
    bool outline;  // offset=10, size=1
    bool shadow;  // offset=11, size=1
    char underline;  // offset=12, size=1
    char* name;  // offset=16, size=8
    char* color;  // offset=24, size=8
    double size;  // offset=32, size=8
    size_t family;  // offset=40, size=0
    size_t charset;  // offset=48, size=0
    char* scheme;  // offset=56, size=8
};

struct XlsBorderProp {  // size=16
    bool has_style;  // offset=0, size=1
    bool has_color;  // offset=1, size=1
    char style;  // offset=2, size=1
    char* color;  // offset=8, size=8
};

struct XlsBorders {  // size=72
    bool has_left;  // offset=0, size=1
    bool has_top;  // offset=1, size=1
    bool has_right;  // offset=2, size=1
    bool has_bottom;  // offset=3, size=1
    XlsBorderProp left;  // offset=8, size=16
    XlsBorderProp top;  // offset=24, size=16
    XlsBorderProp right;  // offset=40, size=16
    XlsBorderProp bottom;  // offset=56, size=16
};

struct SheetTitle {  // size=8
    char* title;  // offset=0, size=8
};

EMBED_FFI_END

excel_module.register_struct("CellPosition", ffi.get_struct_id("CellPosition", __FILE__));
excel_module.register_struct("RangePosition", ffi.get_struct_id("RangePosition", __FILE__));
excel_module.register_struct("XlsFont", ffi.get_struct_id("XlsFont", __FILE__));
excel_module.register_struct("XlsBorderProp", ffi.get_struct_id("XlsBorderProp", __FILE__));
excel_module.register_struct("XlsBorders", ffi.get_struct_id("XlsBorders", __FILE__));
excel_module.register_struct("SheetTitle", ffi.get_struct_id("SheetTitle", __FILE__));
```


## 高级功能
### 对 gs 数据类型操作支持
#### 简介
​​gsbind 的当前通过 gs 向动态库注册回调函数指针的方式以支持在动态库中操作 gs 中的 array，map，string，buffer数据结构。当动态库需要操作 gs 管理的数据结构时，它通过调用这些注册的回调函数间接进行。实现尽量保留了近似于 gs efun开发的语法习惯。

> **参数及返回值转换**：gsbind带有高级功能的函数的函数参数，返回值处理需自行手动处理，而不再自动生成转换逻辑。

> **PendingMode**：gsbind带有高级功能的函数的 PendingMode 不支持 PENDING/LEAVE_DOMAIN/POST_TO_MAIN

#### 简单样例
以下为一个gsbind高级功能函数的简单样例，函数会使用接收的参数构建一个简单的map并返回,编译流程与前述一致。
```c
#include "gsbind.h"
// DECL_TFUNC function prototype is 'void (cmm::Value* ret, cmm::Value* __args, size_t __n)'
DECL_TFUNC(map, construct_map, (array keys, array values), "Construct a simple map")
{
	// Reserve '2' value space on value stack
	// Make the stack ReserveStackImpl operator 'gs_stack'
	RESERVE_STACK(gs_stack, 3);
	// Get arguments from gs_stack
	auto& keys_arr = gs_stack.get_arg_arr(0);
	auto& values_arr = gs_stack.get_arg_arr(1);
	// Use the reserved value space
	auto& tmp = gs_stack.mixed();	// Reserved stack value pos:1
	auto& tmp1 = gs_stack.mixed();	// Reserved stack value pos:2
	auto& map = gs_stack.mixed();	// Reserved stack value pos:3

	auto& map_impl = map.bind_map(keys_arr.size()); // Bind stack value (pos:3) to map

	// Construct map from arrays
	for (int i = 0; i < keys_arr.size(); i++)
	{
		// Keys array get index element to tmp 
		keys_arr.get(i, tmp);
		if (i < values_arr.size())
		{
			// Values array get index element to tmp 
			values_arr.get(i, tmp1);
			// Map set key value pair
			map_impl.set(tmp, tmp1);
		}
	}

	auto& str = tmp.bind_string("Hello advanced gsbind.");	// Bind stack value (pos:1) to string
	const Uint8* new_buf = reinterpret_cast<const uint8_t*>("Create a buffer.");
	auto& buf = tmp1.bind_buffer(new_buf, strlen("Create a buffer."));	// Bind stack value (pos:2) to buffer
	map_impl.set(str, buf);
	gs_stack.set_ret(map_impl);
}

INIT_MODULE(gsbind_sample)
{
	DEFINE_TFUNC(construct_map);
}
```
编写 test.gs 脚本
```cpp
// test.gs
import .gsbind_sample;
write(gsbind_sample.construct_map(["aa", 11], ["bb", 22]));
```
运行 test.gs 脚本，结果如下：
```cpp

{ /* sizeof() == 3 */
    "aa" : "bb",
    11 : 22,
    "Hello advanced gsbind." : 43 72 65 61 74 65 20 61-20 62 75 66 66 65 72 2E    Create a buffer.
,
}
```
#### 宏简介
此处简单介绍下基本的 C++ 侧 gsbind 支持高级功能必须的宏。
* `DECL_TFUNC(ret_gs_type, func_name, arguments, doc) { your function body...}`
  * 该宏用于声明带有gsbind高级功能的函数，其函数原型为 `void (cmm::Value* ret, cmm::Value* __args, size_t __n)`
  * 该宏后需紧跟函数体实现对应的 gsbind 函数
  * 注意 ret_type 和 arguments 中的类型应为 gs 类型而不是C++类型

* `RESERVE_STACK(stack_name, stack_reserved_space)`
  * 该宏用于封装 gs 栈或携程操作
  * 所有 gsbind 高级功能均在该封装基础上实现

* `DEFINE_TFUNC(func_name)`
    * 与`DECL_TFUNC`宏对应使用, prototype 及 doc 参数实际来自于对应的`DECL_TFUNC`定义
    * 该宏用于调用`void def_efunc(Func&& f, const char* name, const char* prototype, const char* doc)`添加注册带有高级功能的函数

#### RESERVE_STACK 栈空间申请封装
`RESERVE_STACK(stack_name, stack_reserved_space)` 实际是创建了一个`ReserveStackImpl`类，该类在初始化时会获取当前 `Coroutine`，`ValueStack` 并在`ValueStack`上预留指定的空间。

| 成员函数接口    | 接口作用 |
| ----------- | ----------- |
|StackValue& mixed()| 从预留的`ValueStack`上获取一个`mixed`类型的`StackValue`，其栈上位置随调用次数递增，当栈上位置超过预留空间限制时报错|
|StackValue& get_arg(ArgNo idx, ValueType type = ValueType::MIXED)|从`ValueStack`上获取函数参数，带有可选类型检查，参数数量超范围报错|
|ArrayImpl& get_arg_arr(ArgNo idx)|从`ValueStack`上获取函数`array`参数，带有`array`类型检查，参数数量超范围报错|
|MapImpl& get_arg_map |从`ValueStack`上获取函数`map`参数，带有`map`类型检查，参数数量超范围报错|
|StringImpl& get_arg_str(ArgNo idx) |从`ValueStack`上获取函数`string`参数，带有`string`类型检查，参数数量超范围报错|
|BufferImpl& get_arg_buf(ArgNo idx) |从`ValueStack`上获取函数`buffer`参数，带有`buffer`类型检查，参数数量超范围报错|
|void set_ret(StackValue& ret_val) |设置返回值|

#### StackValue 操作支持
注意 `StackValue`, `ArrayImpl`, `MapImpl`, `StingImpl`,`BufferImpl` 本质都是 `Value`，只不过允许调用的方法不同。

| 成员函数接口    | 接口作用 |
| ----------- | ----------- |
| StringImpl& bind_string(const char* str)| 绑定C字符串到当前值，创建 `string`，返回 `StringImpl` 类型引用 |
| StringImpl& bind_string(const char* str, size_t len)| 绑定指定长度的C字符串到当前值，，返回 `StringImpl` 类型引用 |
| BufferImpl& bind_buffer(size_t len, size_t reserved = 0)|创建指定长度的 `buffer`，绑定到当前值，返回 `BufferImpl` 类型引用|
| BufferImpl& bind_buffer(const Uint8* p, size_t len)| 创建指定长度的`buffer`并绑定现有字节数据，返回 `BufferImpl` 类型引用 |
| ArrayImpl& bind_array(size_t reserved)|创建新 `array` 指定 `capacity`，绑定到当前值，返回`ArrayImpl`引用|
| MapImpl& bind_map(size_t reserved)|创建新映射 `map` 指定 `capacity`，绑定到当前值，返回`MapImpl`引用|
| void dup_to(StackValue& to)| 浅拷贝当前值到目标栈值 `to`|
| void deep_dup_to(StackValue& to)| 深拷贝当前值到目标栈值 `to`|
| StackValue& set_int(Integer v)| 将当前值设置为整数值，返回自身 `StackValue` 引用|
| StackValue& set_real(Real v)| 将当前值设置为浮点数值，返回自身 `StackValue` 引用|
| StackValue& set_void()| 将当前值设置为 `void` 状态，返回自身 `StackValue` 引用|
| StackValue& set_raw_pointer(const void* pointer, StructId struct_id = 0)| 将当前值设置为原始指针，返回自身 `StackValue` 引用|
| StackValue& as_string() | 检查是否为 `string` 类型，若不是则报错。返回自身 `StackValue` 引用|
| StackValue& as_buffer() | 检查是否为 `buffer` 类型，若不是则报错。返回自身 `StackValue` 引用|
| StackValue& as_array()| 检查是否为 `array` 类型，若不是则报错。返回自身 `StackValue` 引用|
| StackValue& as_map()| 检查是否为 `map` 类型，若不是则报错。 返回自身 `StackValue` 引用|
| StackValue& set_nil()| 检查是否为 `nil` 类型，若不是则报错。 返回自身 `StackValue` 引用|
| ArrayImpl& get_array()| 检查当前  `StackValue` 是否为 `array`，若不是则报错，返回`ArrayImpl`引用 |
| MapImpl& get_map()| 检查当前 `StackValue` 是否为 `map` 若不是则报错，返回`MapImpl`引用 |
| StringImpl& get_string()|检查当前 `StackValue` 是否为 `sting` 若不是则报错，返回`StringImpl`引用 |
| BufferImpl& get_buffer()|检查当前 `StackValue` 是否为 `buffer` 若不是则报错，返回`BufferImpl`引用 |
| bool  is_nil()|检查当前 `StackValue` 是否为 `nil`|
| bool  is_int()|检查当前 `StackValue` 是否为 `int`|
| bool  is_real()|检查当前 `StackValue` 是否为 `real`|
| bool  is_raw_pointer()|检查当前 `StackValue` 是否为 `raw_pointer`|
| bool  is_string()|检查当前 `StackValue` 是否为 `string`|
| bool  is_buffer()|检查当前 `StackValue` 是否为 `buffer`|
| bool  is_array()|检查当前 `StackValue` 是否为 `array`|
| bool  is_map()|检查当前 `StackValue` 是否为 `map`|
#### int/float 操作支持

| 成员函数接口    | 接口作用 |
| ----------- | ----------- |
| StackValue& set_int(Integer v) | 将当前 StackValue 值设置为 int|
| StackValue& set_real(Real v) | 将当前 StackValue 值设置为 float|
| Integer get_int() | 获取当前 StackValue 的整型数|
| Real    get_real() | 获取当前 StackValue 的浮点数|

#### array 操作支持
| 成员函数接口    | 接口作用 |
| ----------- | ----------- |
| void get(Integer idx, StackValue& p1) | 获取 `array` 在 `idx` 处的 element 值至 `p1`|
| void set(Integer idx, StackValue& p1) | 将 `array` 在 `idx` 处的 element 置为 `p1`|
| size_t size() | 获取 `array` 大小|
| ArrayImpl& resize(size_t size) | 重设 `array` 大小|
| void insert(Integer idx, StackValue& val) | 在 `array` `idx` 处插入数据|
| void clear() | 清空 `array`|
| void delete_at(Integer idx) | 移除在 `array` `idx` 处数据|
| void init_value(StackValue& val, size_t size, bool dup = false) | 以 `val` 初始化大小为 `size` 的 `array`| 

#### map 操作支持
| 成员函数接口    | 接口作用 |
| ----------- | ----------- |
| clear() | 清空 `map` |
| bool contains_key(StackValue& key) | 检查 `map` 是否包含键值 |
| void get(StackValue& key, StackValue& ret) | 获取 `key` 对应的键值至 `ret`|
| void set(StackValue& key, StackValue& val) | 添加或更新 `key` 对应的键值至 `val`|
| void keys(StackValue& ret_keys) | 获取 `map` 所有键构成的数组至 `ret_keys` |
| void values(StackValue& ret_values) | 获取 `map `所有值构成的数组至 `ret_vals` |
| bool erase(StackValue& key) | 移除在 `key` 对应的键值对，如果成功移除，返回 `true` | 

#### string 操作支持
| 成员函数接口    | 接口作用 |
| ----------- | ----------- |
| const char* get_data_read(size_t pos = 0) | 获取 `string` 只读的 `char*` 指针|
| size_t length() | 获取 `string` 长度|
| int compare(StringImpl& other)| 以 `gs` 规则比较 `string` |

#### buffer 操作支持
| 成员函数接口    | 接口作用 |
| ----------- | ----------- |
| Uint8* data_to_modify(size_t len)| 获取可写 `buffer` 的 `Uint8*` 指针，检查 `buffer` 长度 >= `len`，若为不可写 `buffer` 则报错|
| const Uint8* data_to_read() | 获取可写 `buffer` 的只读 `const Uint8*` 指针|
| size_t length() | 获取 `buffer` 的长度|
| BufferImpl& resize(size_t size) | 调整可写 `buffer` 的大小|

## 样例项目

   * [gsbind_sample](https://m68gitlab.g-bits.com/pkgs/gsbind_sample)
   * [bind_excel](https://m68gitlab.g-bits.com/pkgs/bind_excel)
## QA
目前遇到的一些问题总结:
 
## Todo list
`gsbind` 目前仅支持基础的跨语言函数调用，但其实现方式有很大的拓展支持与优化潜力，以下是后续可能进行的拓展的todo list。
* ~~gsbind 尝试支持 gs pending call（完成）~~
* ~~gsbind 尝试添加 safe call 机制（完成）~~
* ~~gsbind 尝试支持在 C++ 动态库中操作 gs 数据类型（完成） ~~
* gsbind 支持传递值传递 struct，class 等
* gsbind cicd流程自动生成 dll 函数描述文件 gslang 需求
* gsbind 尝试支持通过 ffi 自动生成 cpp 代码，自动封装及编译
* gsbind 尝试支持 C++ 函数重载，默认参数，Lambda表达式等
* gsbind 尝试支持 C++ 类绑定至 gs
* gsbind 尝试支持 C++ 智能指针

