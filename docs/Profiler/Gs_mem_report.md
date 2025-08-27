---
sidebar_label: '内存统计'
sidebar_position: 1
---

# 内存统计

---

## 简介

用于统计driver运行时内存占用情况的工具。

当程序有内存泄漏产生时， mem_snapshot 方法仅统计不同handle下相对大小的改变，部分内存泄漏点仍难以找到和排查。本工具通过记录pdb中代码位置信息与内存申请指针的对应关系统计内存占用情况。统计内存占用的单位为字节，可生成日志文件，并行解析，获取以100ms为间隔的局部最大内存占用情况。

目前内存统计有两种形式， MemRecordType.LOG_RECORD MemRecordType.MEM_RECORD。
MEM_RECORD状态下内存统计工具仅在内存中存储当前driver的内存申请情况，而LOG_RECORD则会将所有历史申请数据保存在日志中。内存统计工具保存信息所占用空间不是由 gs 内存分配器所分配，其不会影响 mem_state 或 get_mem_report 结果。

需要注意的是，由于gs的内存申请极为频繁且统计工具需要保存内存申请位置与所申请空间指针的对应关系，开启内存统计工具后内存占用较大且运行速度明显减慢。

## 开启内存统计

开启内存统计后，会在driver内存管理器的申请处记录内存指针对应的申请位置及申请大小，在释放处减少所释放内存指针对应申请位置的已申请大小。
一般情况下调用 set_enable_mem_profiler(MemRecordType.MEM_RECORD)开启 driver内部内存记录。并在内存泄露前后通过 ’get_mem_report() 查看当前申请未释放的内存信息即可定位内存泄露具体的申请位置。

```
system.core.set_enable_mem_profiler [native|static]
 - bool set_enable_mem_profiler(int enable_type = 4)
 - Enable internal memory record and get_mem_report, default : MemRecordType.MEM_RECORD
        MemRecordType.CLOSED       - close internal memory record
        MemRecordType.LOG_RECORD   - record memory info in log files and hash map
        MemRecordType.LOG_FAST     - record memory info in log files and array
        MemRecordType.MEM_RECORD   - record memory info only in hash map
        MemRecordType.MEM_FAST     - record memory info only in array
```
开启模式的划分:

* 以 "RECORD" 结尾的模式通过 hash 映射进行内存申请记录，一般来说其占用内存相对 "FAST" 模式更小，但hash映射有额外的性能消耗，同时hash表扩张时需要进行rehash同样有一定性能消耗。
* 以 "FAST" 结尾的模式会直接申请 1/4 page-alloc 大小的内存，用于存储所有可能的内存申请记录，空间消耗相对 "RECORD" 更大，但由于不需要 hash 和 rehash, 速度相对 "RECORD" 模式会快很多。
* 以 "LOG" 开头的模式相对以 "MEM" 开头的模式，会额外将所有内存申请记录写入driver 根路径下的本地日志文件中，且记录带有以100ms为间隔的时间戳。通过调用函数 mem_profiler.start_mem_log_profiler() 进行日志文件解析后，可以通过 mem_profiler.get_all_time_size() 查看内存数量随运行时间的变化情况。

具体的模式选择策略:
* 大部分情况下通过 MemRecordType.MEM_RECORD 模式启动 mem_record 后，在可能泄露的逻辑前后分别调用 get_mem_report() 进行对比即可拿到泄露的具体内存申请位置。
* 若内存申请较为频繁，以 MemRecordType.MEM_RECORD 模式启动后程序运行速度很慢，且机器有额外 1/4 page-alloc 内存的情况下，请以 带有 "FAST" 结尾的模式启动 mem_record。
* 若内存泄露问题复现后没有足够的时间窗口调用 get_mem_report 或者需排查长时间偶现的内存异常增长之后又恢复正常的情况，则可以选择带有 "LOG"开头的模式。将 "LOG" 模式下生成的以 memlog 开头的日志文件复制至driver根目录后即可解析, 详情见下文内存日志的使用。

简单的使用样例:

```
import gs.mem_profiler;
set_enable_mem_profiler(MemRecordType.MEM_RECORD);
array test_arr = [];
for(int i = 0; i < 10000; i++)
{
    test_arr.push_back(get_system_info());// Internal mem_profiler will record the memory alloc place
}
map report = mem_profiler.get_mem_report();
write(report);
```

结果如下:

```
{ /* sizeof() == 2 */
    "total_size" : 79850464,
    "mem_report" : [ /* sizeof() == 11 */
        "/test/test.gs                                                       line:6       size:79558144",
        "G:\gs_master\master\gs\std\include\std_template\simple_vector.h     line:251     size:259424",
        "G:\gs_master\master\gs\std\include\std_template\simple_vector.h     line:353     size:32768",
        "G:\gs_master\master\gs\std\include\std_template\simple_vector.h     line:132     size:32",
        "/gs/mem_profiler.gs                                                 line:1109    size:32",
        "/gs/mem_profiler.gs                                                 line:1107    size:32",
        "/test/test.gs                                                       line:3       size:32",
        "G:\gs_master\master\gs\mts\cmm_gc_gray_page.h                       line:59      size:0",
        "G:\gs_master\master\gs\std\include\std_template\simple_list.h       line:478     size:0",
        "G:\gs_master\master\gs\std\src\std_template\simple_string.cpp       line:405     size:0",
        "G:\gs_master\master\gs\mts\cmm_gc_gray_page.h                       line:287     size:0",
    ],
}
```
## 内存日志使用

当前内存申请日志以近似100ms为时间间隔，统计时间间隔内局部内存最大值处的内存占用情况。日志文件每占用80 - 100MB左右后会创新的日志文件，**默认总日志文件数量限制为10**。日志文件可放在任何driver的 /r 路径进行解析，不同文件之间相对独立，可以并行解析。

内存日志一般用于泄露产生时间不确定且泄露所需测试运行时间过长的情况下，在以 LOG_RECORD 形式开启内存记录时，内存日志会不断产生，**在日志文件数量达到上限后最旧的内存日志文件会被删除**。start_mem_log_profiler 仅会解析 root 目录下最新的连续的日志文件。

以如下方式开启内存统计日志:

```
set_enable_mem_profiler(MemRecordType.LOG_RECORD)
```

此时查看启动driver时设置的 /r 所在路径，可以看到以 memlog 开头的 txt 内存日志文件，保留该文件在 /r 位置或另起一driver将该文件复制至该driver的 /r 位置，默认总日志文件数量限制为10，超过日志数量限制时最老的日志文件会被删除。可通过get_mem_profiler_log_number() ，set_mem_profiler_log_number( log_num ) 获取或日志文件限制数量。

运行 mem_profiler.start_mem_log_profiler() 解析二进制日志文件，注意默认情况下，start_mem_log_profiler会创建出日志文件数量的协程进行并行解析，也可手动指定协程数量。

```
Shell> mem_profiler.start_mem_log_profiler() //mem_profiler.start_mem_log_profiler(coroutine_num)
Processing, please wait a minute
Parse memlog20230209_5F43FAFD5D7FF-0.txt complete
Process complete, profiler contains record for 34412 ms, start from 2023-02-09 15:54:1
```

解析完成后运行 mem_profiler.get_all_time_size() 可查看以近似100ms 为间隔统计到局部最大内存数量。运行mem_profiler.get_mem_report_with_time(time_ms) 可查看time_ms所在间隔内局部最大内存的具体占用情况。

```
Shell> 'get_all_time_size()
[ /* sizeof() == 339 */
    [ /* sizeof() == 2 */
        0,
        10752,
    ],
    [ /* sizeof() == 2 */
        102,
        10752,
    ],
    [ /* sizeof() == 2 */
        203,
        10752,
    ],
    [ /* sizeof() == 2 */
        305,
        10752,
    ],
    [ /* sizeof() == 2 */
        406,
        10752,
    ]...
Shell> 'mem_profiler.get_mem_report_with_time(305)
{ /* sizeof() == 2 */
    "total_size" : 10752,
    "mem_report" : [ /* sizeof() == 8 */
        "G:\gs_master\master\gs\std\include\std_template\simple_vector.h  line:251              size:5376",
        "/builtin/shell.gs                                                line:22               size:4128",
        "G:\gs_master\master\gs\mts\cmm_shell_debugger.cpp                line:40               size:704",
        "/builtin/shell.gs                                                line:107              size:448",
        "G:\gs_master\master\gs\mts\cmm_debugger.cpp                      line:130              size:64",
        "G:\gs_master\master\gs\std\include\std_template\simple_vector.h  line:132              size:32",
        "G:\gs_master\master\gs\std\src\std_template\simple_string.cpp    line:405              size:0",
        "G:\gs_master\master\gs\mts\cmm_temp_buffer.h                     line:36               size:0",
    ],
}
    
```

## 问题

上述例子可以看到部分内存申请记录定位到了 driver 内部的 .h 或者 .cpp 文件，这是正常的，一般是driver内部用于 gc 工作相关的内存。但在内存泄露产生且内存泄漏定位在 .h 或 .cpp文件情况下是有问题的，目前所测得的所有该类情况都是coroutine创建时以 efunc 作为函数参数，该情况下内部实际调用并不经过 gs 脚本层所以没有对应的pdb信息，所以申请定位在driver内部文件中。

若是上述之外的内存泄露定位在 .h .cpp 文件的情况，请尽可能准备可复现脚本或测例，并联系@chuyx。
