---
sidebar_label: 'gc内存快照'
sidebar_position: 2
---

# gc 内存快照

---

## 简介

用于统计 driver 运行时内存占用情况的工具。

mem_snapshot 方法统计不同 snapshot 下的 handle 数量的相对改变，缺点是无法获取具体的内存大小仅能获得某类成员的相对数量。

mem_record 则是记录某一内存申请位置已申请但未释放的内存大小，缺点是只能记录内存申请位置无法知晓已申请内容具体被哪个 handle 引用，需要自己分析代码，且 mem_record 必须在内存申请前就开启。

gc_snapshot 则是通过日志记录的内存 block 描述，大小，引用关系，生成 block-ref 日志。再通过 profiler-tool 解析后以可视化的形式展示出来。

[**profiler-tool 下载**](http://tools.jszx.g-bits.com:3000/profiler-tool)

## gc 内存快照

gs 支持的 snapshot 方法有两个

- gc.snapshot_target 方法
  - gc.snapshot_target 用于内存申请后，记录某一指定 unit 引用到的内存的 block-ref 日志。
    ```
    void snapshot_target, (mixed search_unit, int max_round = 3, int max_log_limit = 2097152, bool record_handle_ref = true,  int element_limit = 3, int layer_limit = 2)
        在driver 的 root 路径生成 search_unit 的 block-ref日志
        search_unit: snapshot_target 会记录 search_unit 所能 gc mark 到的所有 block 内存成员并生成日志。
        max_round: snapshot_target 的 gc_search 执行轮次。
        max_log_limit: 最大的 block-ref 日志数量, 在生成日志的过程中可能的最大内存使用量为 512 * max_log_limit 字节。
        record_handle_ref: 决定 snapshot_target 是否会记录跨handle的引用。
        element_limit：block生成Array/Map等reference类型描述的最大成员数量。
        layer_limit：block生成Array/Map等reference类型描述的最大层数限制。
    ```
- gc.snapshot_alloc 方法
  - gc.snapshot_alloc 用于内存申请前，记录到接下来 n 秒内的申请未释放内存的引用 block-ref 日志。
    ```
    void snapshot_alloc (float record_time, int max_round = 3, int max_log_limit = 2097152,int element_limit = 3, int layer_limit = 2)
        在根路径生成接下来n秒内申请未释放内存的block-ref日志
        record_time: snapshot_alloc会记录接下来record_time时间内申请未释放内存的block-ref日志。
        max_round: snapshot_alloc 的 gc_search 执行轮次。
        max_log_limit: 最大的 block-ref 日志数量, 在生成日志的过程中可能的最大内存使用量为 512 * max_log_limit 字节。
        element_limit：block生成Array/Map等reference类型描述的最大成员数量。
        layer_limit：block生成Array/Map等reference类型描述的最大层数限制。
    ```

简单的使用样例:

```
//with_sysinfo_ob.gs
#pragma parallel
parallel array arr := make_parallel([]);
void create_data()
{
    array new_arr = [];
    for(int i = 0 upto 32 * 1024)
    {
        new_arr.push_back(get_system_info());
    }
    arr := make_parallel(new_arr);
}
create_data();

//test.gs
import .with_sysinfo_ob;
array ob_arr = [];
void test()
{
    //gc.snapshot_alloc(5); // 记录之后5s中申请未释放内存的 block-ref 信息
    for(int i = 0 upto 8)
    {
        ob_arr.push_back(new_object(with_sysinfo_ob, domain.create()));
    }
    //gc.snapshot_target(this_object()); // 记录 this_object() 能 mark 到的内存的 block-ref 信息
}
test();
```

如上代码申请 2GB 左右的内存数据，内存引用关系如下:

```
- object[test.gs] -> array[ob_arr] -> 8 * object[with_sysinfo_ob.gs]
- object[with_sysinfo_ob.gs] -> array[arr] -> 32 * 1024 * map[get_system_info]
```

根据内存申请前或内存申请后的内存统计需要，放开 snapshot_alloc 或 snapshot_target 代码处的注释进行 snapshot。
snapshot 成功时会提示`Gc snapshot finished,please checkout the root path`，此时检查 drvier 根路径会发现以`gc_snapshot_log`开头的文件夹，将文件夹中的任意`.br`文件拖入新版 profiler-tool 解析工具启动页面中即可，profiler-tool 会解析处于相同目录下的所有.br 文件。

## 工具使用演示

以下链接会自动跳转至 profiler-tool 视频部分:
[**gc_snapshot 工具演示**](https://leiting.feishu.cn/wiki/Tw4NwYUTri1uQUkurAecpfYjnmg#share-LJGkd0xcFoxwchxbJZIc7MNFneb)

## driver 版本

目前该 snapshot 工具仅在 driver 1.23.240729-m72fix11 与 1.31.250330 版本后提供支持，若为之前的版本可联系 jszx 的同学 pick 相关修改以启用该工具。

## Block 信息

| 成员名  | 描述                                         |
| ------- | -------------------------------------------- |
| Describ | 最上方显示为 block 内容描述信息              |
| Type    | 代表 block 对应的 Value 类型                 |
| Size    | 代表当前 block 的内存大小                    |
| Usage   | 当前 block 能引用到的 block 的 Size 大小之和 |

:::warning
Profiler 工具显示的矩形大小是按照 Usage 大小来计算并显示的。
:::

## BadType 类型

profiler-tool 显示的信息中，可能会出现 Describ 描述为 mem，Type 为 BadType 的内存空间。
其实际代表 map 或 array 容器在 C++ 实现侧申请的用于保存 Value 的 buckets 内存空间。

## 日志数量限制

可能会出现 block 内存块 Usage 大小在实际上应该是相等的，但 profiler-tool 解析结果却不同的情况。其成因是由于 block-ref 的日志数量限制，部分内存 block 的 block-ref 信息未被完全统计，导致相应的 Usage 实际偏小。

所以在申请内存较多，需要记录的 block-ref 日志较多导致触发日志数量限制的情况下，请尽量选择合适的目标进行 snapshot，尽可能利用有限的日志收集有效的信息。

或者考虑一定程度增大 max_log_limit 日志数量限制。

## Block 索引

profiler 工具中矩形图上显示的数字其实是`内存索引`，其计算为：

`（block 地址 - 堆内存起始地址） / 堆内存对齐大小`

代表 object 的 `block 内存索引`计算结果是可以与 `object.info().index` 对应的。

## 文档

[**带有图片及视频的文档**](https://leiting.feishu.cn/wiki/Tw4NwYUTri1uQUkurAecpfYjnmg)
