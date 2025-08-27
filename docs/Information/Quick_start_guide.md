---
sidebar_label: '快速开始'
sidebar_position: 2.1
---

# 快速上手

## driver的启动

### 查看全部启动参数
使用 `--help`（或者 `-h`, `-H`） 作为命令行参数启动driver可以获取启动参数说明：

```bash
gs.exe --help
```

### 常用的启动参数
下面将介绍一些常用的启动参数，更详细的启动参数见文末。

* `/r` 设置根目录
* `/e` 启动脚本
* `/eq` 启动脚本，在脚本执行完成后退出
* `/m` 可以带多个预加载的脚本，如果有多个用逗号隔开(`,`)
* `--mount` 挂载目录
* `/D` 预定义宏
* `--require` || `--require all` 导入内置插件（用于兼容旧代码，详细说明见 `import` 章节中的相关说明）
* `/c1` 开启命令行交互模式

其中`/c1` 常用于*linux*系统中，**windows**下是默认开启的。

下面给出一个例子更好的理解上述的其他参数，可以不用关注语法细节，使用的语法也非常简单，需要关注的是启动参数的使用方法。

目录结构如下：
```
sample-server
├─start_server.bat
├─server
|   ├─preload.gsh
|   └server.gs
├─etc
|  └test.txt
├─bin
|  └gs.exe
```
start-server.bat 内容如下
```powershell
.\bin\gs.exe /r ./server/ /e /server.gs /m /preload.gsh --mount ./etc::/etc_m
```

可以看到这句批处理将根目录设置为`./server/`, 后面`/e` 和 `/m`都是相对于这个根目录来说，`mount`是将当前目录下的`etc`文件夹挂载到根目录下的*etc*。

preload.gsh 内容如下
```js
write("load preload.gsh\n");
```
sever.gs 内容如下
```js
write("sample server\n");
```

启动脚本后可以看到先输出了`load preload.gsh` 然后输出了`sample server`, 也就是说预加载脚本和启动脚本都被执行了，且预加载脚本是在启动脚本之前执行的。

接下来我们来验证一下根目录是否被设置为./server以及etc是否有挂载在根目录下。

在控制台上敲一句`'get_system_info()`，这里的`'`用于控制台上的快捷打印，`get_system_info()`可以返回**driver**现在的一些信息。查看输出中的*root_path*和*mount_info*即可。进一步验证etc是否成功挂载成根目录下的etc_m可以在控制台上敲一下下面这句话。
```js
'file.exist("/etc_m/test.txt")
```

现在我们已经对启动参数以及简单的项目结构有一个整体的认知了，并且能对shell进行操作，接下来看看shell上的常用方法吧。

## shell常用方法

* man 用于查看函数说明  
比如想查看array相关的函数，这时可以在控制台敲`man("array")` 或者敲`man array`, 两种写法是等价的（但在telnet时只能用前者）。可以看到函数的属性(包括定义在c++中还是定义在script中，是static函数还是member函数)，参数列表以及描述。

* `cls` 用于清空屏幕

* `。`|| `.` 用于退出driver

* `=>` 用于切换 shell 执行时的域
  * `=> ob.get_domain()` 可以切换到 ob 所属的域，之后shell的操作将在该域中执行
  * `=> nil` 可以切换到 shell 的默认域
  * 切换到目标域后的 shell 并不会持续占有域锁，而是仅当执行 shell 命令时占有域锁，执行完毕后会自动释放域锁。
  * 特别注意，如果被切换的目标域始终不让出，可能导致 shell 无法进入而阻塞

* `clear env` 可以清除此前在shell中定义的变量、函数和宏

* `#define` 可以定义在shell 中使用的宏
  * 出于安全考虑，覆盖同名的宏（无论是之前在shell中定义的还是环境中定义的），都会导致之后执行的代码发出警告
  
* `monkey.patch(program_name)` 或 `monkey.patch("代码文件路径")`
  * 可以尝试通过`handle.get_all("program")`查询当前有哪些program
  * 然后通过`monkey.patch(program_name)`去热更新
  * `update()`也可以用于热更新， 但是实现原理不太相同， update的逻辑更简单compile → destruct_object → load_static，但是对于有状态的对象，成员变量无法恢复，无法进行热更新。`monkey.patch`更为通用。

* `find_object(handle|string name_or_id)` 查找一个object
* `get_obj_var(object ob, string var_name)` 查看某个object的私有变量
* `set_obj_var(object ob, string var_name, mixed val)` 设置某个私有变量的值 避免添加一个set函数->patch->删除这个set函数->patch这样的工作
* `child_objects(string | object name)` 某个对象的所有实例

* 常规的输出操作为`write("xxxx")`或`printf("xxxxx")`
  * 但是在*shell*中可以使用 `'` 进行快捷打印
  * 例如 `'foo(a, b, c)` 可以直接打印出表达式 `foo(a, b, c)` 的值

## 如何快速测试

在下一章中将会有很多需要测试的例子，快速测试并不需要写上面提到那种相对复杂的项目结构。下面介绍常用的两种方法，以如下这个简单的目录结构为例
```
sample-helloworld
├─test.gs
├─gs.exe
```
一. 使用启动参数启动  
启动语句
```powershell
gs.exe /r ./ /e /test.gs
```
二. 使用load_static   
启动参数什么也不写，把driver启动起来  
在控制台上敲
```js
load_static("/test.gs", this_domain())
```

## 全部启动参数简介

| 选项/配置                          | 描述                                                                 |
|------------------------------------|----------------------------------------------------------------------|
| **选项**                           |                                                                      |
| `-c1`                             | 启用命令行。                                                         |
| `-c2`                             | 启用命令行，并在启动完成前允许输入。                                 |
| `-d`                              | 在指定端口启用调试器。                                               |
| `-e[q]`                           | 设置虚拟机的启动脚本（如果设置了 'q' 则在完成后退出）。               |
| `-l`                              | 设置日志文件名。                                                     |
| `-o`                              | 设置基础脚本（在启动脚本之前执行）。                                 |
| `-p`                              | 设置 Telnet 端口。                                                   |
| `-r`                              | 设置虚拟机根目录。                                                   |
| `-s`                              | 设置默认线程池大小。                                                 |
| `-sm`                             | 设置 GC 静态工作线程数。                                             |
| `-w`                              | 设置 Telnet 密码。                                                   |
| `-wd`                             | 设置工作目录。                                                       |
| `-D`                              | 声明一个定义。                                                       |
| `-v`                              | 打印版本信息。                                                       |
| **配置**                           |                                                                      |
| `--bin-alloc`                     | 使用二进制分配器。                                                   |
| `--call-stack-size %d`           | 设置函数调用上下文的深度。                                           |
| `--enable-threads`               | 如果为 0，则禁用除并行调度器和主线程外的其他线程，禁止套接字守护进程和新线程创建；通常与 `/s 0` 一起使用。 |
| `--enable-color-error`           | 将错误输出消息设置为红色。                                           |
| `--enable-compiler-debug`        | 启用编译器调试，显示生成的汇编代码。                                 |
| `--enable-debugger [%d]::[%d]`  | 允许嵌入式调试器工作，可选调试器端口和状态（1 表示随机端口，1 或 0 表示启动时暂停）。 |
| `--enable-gc-log`                | 启用 GC 日志。                                                       |
| `--enable-jit`                   | 启用 JIT（0 表示禁用，1 表示 asmjit，2 表示 llvmjit）。              |
| `--enable-jit-debug`             | 启用 JIT 调试。                                                      |
| `--enable-jit-log`               | 启用 JIT 日志。                                                      |
| `--enable-multi-cpu`             | 使用多 CPU（可能导致性能下降）。                                     |
| `--enable-system-command`        | 启用 efun "system"（存在安全风险）。                                 |
| `--enable-coverage`              | 启用覆盖率统计。                                                     |
| `--gc-count-upper-bound`         | 设置分配大小计数上限，0 表示无限制。                                 |
| `--gc changedset`                | 使用变更集进行次要 GC 标记。                                         |
| `--gc cardtable`                 | 使用卡表进行次要 GC 标记。                                           |
| `--gc changedset+cardtable`      | 同时使用变更集和卡表进行次要 GC 标记。                               |
| `--gc full`                      | 仅使用完全 GC，不进行次要 GC。                                       |
| `--gc-force-full-gc`             | 启用强制完全 GC。                                                    |
| `--gc-lms-cached`                | 设置每个 LMS 释放内存页时缓存的删除页大小（字节）。                  |
| `--gc-global-cached`             | 设置所有 LMS 释放内存页时缓存的删除页大小（字节）。                  |
| `--gc-gray-page-size`            | 设置灰色页中的单元数（0 表示不使用灰色页）。                         |
| `--gc-max-dynamic-work-us`       | 设置动态工作线程的最大工作时间（微秒）。                             |
| `--gc-max-dynamic-worker`        | 设置动态工作线程的最大数量。                                         |
| `--gc-max-ub-pool`               | 设置单元/块池的最大数量。                                            |
| `--gc-min-gc-interval`           | 设置 GC 间隔的最小时间（毫秒）。                                     |
| `--gc-quota-to-full-gc`          | 提示进行完全 GC（当旧单元大小超过限制时）。                          |
| `--gc-quota-to-minor-gc`         | 提示进行次要 GC（当新单元大小超过限制时）。                          |
| `--gc-max-quota-to-minor-gc`     | 限制次要 GC（当新单元大小超过限制时）。                              |
| `--gc-stat-recycle`              | 在指定轮次后打印回收信息以进行调试。                                 |
| `--gc-time-to-long-live`         | 设置 GC 引用值的默认长期存活时间。                                   |
| `--gc-thread-threshold`          | 限制动态工作线程（当 GC 期间分配大小超过限制时）。                   |
| `--ignore-inexist-import`        | 当导入文件缺失时不报编译错误。                                       |
| `--jit-mem-size`                 | 设置 JIT 内存池保留大小（MB，默认：1024）。                          |
| `--lcs-buf-size`                 | 设置默认 LCS 缓冲区大小。                                            |
| `--no-internal-scripts`          | 启动时不加载内部脚本。                                               |
| `--mount`                        | 挂载设备或路径（真实目录::挂载点）。                                 |
| `--max-coroutines`               | 设置调度器的最大协程数。                                             |
| `--max-device`                   | 设置可打开的最大设备数。                                             |
| `--max-sync_object`              | 设置调度器的最大同步对象数。                                         |
| `--max-rudp`                     | 设置最大 RUDP 连接数。                                               |
| `--max-socket`                   | 设置最大套接字数。                                                   |
| `--optimize`                     | 设置优化选项，格式：opt1=0/1;opt2=0/1...（默认：eps_efun=1）。       |
| `--page-alloc`                   | 使用页内存分配器。                                                   |
| `--page-keep-committed`          | 保持所有页已提交。                                                   |
| `--pause-startup-script`         | 等待某些操作完成后再执行启动脚本。                                   |
| `--publish`                      | 启用某些功能以通过指定的 pmake 文件发布。                            |
| `--publish-builder`              | 设置 pmake 构建器脚本。                                              |
| `--publish-loader`               | 设置 pmake 加载器脚本。                                              |
| `--quick-exit-on-windows`        | 在 Windows 上启用快速退出（通过 TerminateProcess，默认：true）。     |
| `--relinquish`                   | 设置调度器的释放时间。                                               |
| `--require`                      | 添加必需的插件，格式：`--require plugin1;plugin2;...` 或 `--require all`。 |
| `--root-password`                | 设置 `os.reset_euid_to_root` 的根密码。                              |
| `--script-args`                  | 设置脚本参数，格式：`--script-args "args..."`。                      |
| `--search-path`                  | 设置打开设备的搜索路径，格式：`path1;path2;path3...`。               |
| `--socket-coroutine`             | 设置套接字协程数。                                                   |
| `--stack-size %d`               | 设置堆栈大小。                                                       |
| `--treat-warning-as-error`       | 让编译器将警告视为错误。                                             |
| `--use-single-lms`               | 使用单一本地内存存储以节省内存。                                     |
| `--use-large-page`               | 使用大页（Intel 上为 2M 页大小）。                                   |
| `--with-malloc 1[xAll\|xNonGC]` | 在分配内存时执行额外的 malloc 操作。                                 |
| `--use-client-mode`              | 使用客户端模式。                                                     |
| `--timer-scheduler`              | 设置 TimerScheduler 的值（level1_unit_num/level2_unit_num/max_timers）。 |
| `--parallel-scheduler`           | 设置 ParallelScheduler 的值（level1_unit_num/level2_unit_num/pool_size）。 |
| `--thread-scheduler`             | 设置 ThreadScheduler 的值（level1_unit_num/level2_unit_num/pool_size）。 |
| `--mixed-func-ret-check`         | 设置是否检查混合函数中的返回类型。                                   |
| `--disable-dump-window`          | 在 Windows 上禁用崩溃时的消息框窗口（默认：false）。                 |
| `--default-full-dump`            | 在 Windows 上崩溃时自动完全转储（默认：false）。                     |
| `--config`                       | 加载配置文件作为启动设置。                                           |
| `--not-opt-write-back`           | 不优化写回堆栈变量。                                                 |