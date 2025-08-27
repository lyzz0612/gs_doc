---
sidebar_label: '归档'
sidebar_position: 14
---


# archive -- 归档（打包文件夹）为zip/pak

## 概述

对文件夹中的内容打包归档为zip或pak格式的文件

参见[复杂数据类型-archive](../ComplexTypes/Archive.mdx)

## 接口

### Static

```cpp
// Find an archive by path, auto create if need
archive find(string path, bool auto_create = false, string mode = "");
```

```cpp
// Open an archive by path, the archive type will be auto detected by suffix of the file path
archive open(string path, string mode);
```

例子：
```cpp
string name = "test.zip";
archive arch = archive.open(name, "wr");
defer
{
    // Remove the arch files
    if (arch)
        arch.close();
    // Closed the archive opened by file.read_all()
    handle auto_opened = archive.find(name);
    if (auto_opened)
        auto_opened.close();
}

// Do pack

```

### member

```cpp
// Get the file list of dir in archive
array get_dir(string path, bool detail = false);
```

```cpp
// Read the content of file in archive
buffer read_file(string path);
```

```cpp
// Write the content of file in archive
bool write_file(string path, buffer buf);
```



```cpp
/*
tree_files = {
    "dir1" : {
        "file1.dat" : "/data/file1.dat",
        "file2.dat" : "/data/file2.dat",
        ...
    },
    "dir2" : ...,
    "filex" : (buffer)... // data of filex
    ...
}
*/
// Pack files organized by map to a archive file
void pack_tree_files(map tree_files);
```

```cpp
// Dump the full structure of the archive
void dump();
```

```cpp
// Save the archive
void flush();
```

## 归档文件的加载流程

- 通过启动参数/e ./game_server.zip来指定需要运行的zip包。
- 通过archive_loader.load(archive_path)来加载zip包内部的BOOT_PLIST文件。
- 逐行执行boot.plist文件中的执行命令。
- 加载gs脚本时，会优先比较本地文件和zip包内文件的新旧情况，加载最新的文件。

### boot.plist 文件格式

boot.plist 里存放着若干指令，在装载归档时，会自动执行这里面的命令；一个可能的文件内容如下：

```
unpack /@root/etc/** /@local/etc/
unpack /@root/config/** /@local/config
set some_env_var 1
run /main.gs
```

boot.plist 通常用于指示将一些必须的文件释放到指定位置，或者指示driver的启动参数和入口脚本，支持的所有指令如下：

| 指令 | 说明 |
| ---- | ---- |
| preload | 预加载指定的文件，通常用于预加载一些必须的脚本文件 |
| set | 调用 set_env 设置 driver 的环境变量 |
| run | 执行指定的脚本文件 |
| unpack | 将指定的文件解包到指定目录，支持通配符 |
