---
sidebar_label: 'gs_py'
sidebar_position: 3
---

# gs_py

---

## 简介

用于Linux环境下的GS崩溃后的coredump调试工具。

在Linux环境下，当程序运行的过程中出现异常中断或崩溃时，操作系统会将程序当时的运行状态信息记录下来并保存在coredump。Coredump中保存有内存信息、寄存器信息、调用栈信息等，并可以通过gdb的thread、info等命令查看。这是定位程序异常崩溃问题的重要手段。 

对于gs编译器而言，查看操作系统层次的信息是不够的，还需要driver层的信息，因为driver是业务层的低层，业务层内存、栈、寄存器等信息都保存在driver层中。而目前使用gdb工具，driver层的信息只能像普通应用程序一样通过类似p命令的方式获取，十分不方便。因此考虑基于gdb提供的python接口，为gs开发一个coredump可视化工具，方便在gdb中快速查看drive的信息，从而帮助快速定位问题。

gs.py 文件位置: https://m68gitlab.g-bits.com/pkgs/gs_py.git


## 使用方式

在gdb环境下，使用gs.py脚本分析coredump信息。步骤如下：

- 将coredump文件(一般命名为 core.xxxx )、coredump对应的driver、gs.py文件拷贝到同一目录。
- 在terminal终端进入步骤1的目录下，输入命令gdb gs core.xxx文件名，进入gdb调试模式。
- 输入命令so gs.py加载脚本。
- 用 "命令列表" 中的命令获取需要的信息

若出现警告 **warning: RTTI symbol not found for class**, 请以如下形式启动coredump调试： 

gdb gs core.xxx 2>warn.txt

将所有警告信息重定向到文件以便于查看 gs.py 分析结果，driver的编译过程中默认保留了RTTI(运行时类型信息)，目前查阅到的信息显示该警告由gdb的bug产生。


## 命令列表

- ps [true] // 显示当前协程信息，带参数true表示只显示running状态的协程，ps指令会尝试寻找crash发生的coroutine， 若找到会用红色高亮

- get_call_stacks [coroutine_ptr] // 显示协程的调用栈,  co参数为协程指针

- get_obj_var_info [obj_name / obj_ptr] [varname] // 显示object的某个变量信息

- get_obj_vars [obj_name / obj_ptr] // 打印object下所有components的变量内容

- get_obj_info [obj_name / obj_ptr] // 显示某个object的信息

- value_stack [coroutine_ptr] [ip] // 显示指定协程的变量栈上的变量信息，第一个参数为需要显示协程的指针，第二个参数表示要显示第几层栈上的变量，第二个参数可选，默认显示所有层

- value_print [value] // 打印某个Value的信息，参数value表示要打印的Value的指针

- box_value_print [boxvalue] //打印某个BoxValue信息, 当遇到带有 (Boxed) 标记的变量时使用该方法输出

- get_all_handles [handle_type] // 打印所有的coroutine 或 object的信息，目前handle_type 仅可选为 "co" , "object" 或 空


## 使用样例

```
//在centos下克隆并打开本项目
git clone https://m68gitlab.g-bits.com/pkgs/gs_py.git
cd gs_py
git submodule update --init      
cd sample
gdb gs core.22450                      // 开启 gdb 调试
so gs.py                               // 加载 gs.py 命令
ps true                                // 查看RUNNING状态的协程信息，得到协程指针 0x400055a9680
get_call_stacks 0x400055a9680          // 查看协程 call_stack
value_stack 0x400055a9680              // 查看协程 call_stack value_stack
get_all_handles "ojbect"               // 查看所有ojbect信息, 得到test_c_acrash.gs 的 object 指针
get_obj_info 0x400060a63c0             // 获取object信息
get_obj_vars 0x400060a63c0             // 打印object下所有变量内容
get_obj_var_info 0x400060a63c0 b       // 获取object下变量b的信息
// 上述 0x400060a63c0 指针均可用objec的全名 "/unit_test/test_c_acrash.gs" 替代
box_value_print 0x4000608de00          //cmm::MapImple(Boxed)* 带有box标记
value_print 0x40005381a10



```