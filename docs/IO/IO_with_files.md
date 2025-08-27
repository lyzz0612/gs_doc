---
sidebar_label: '文件读写'
sidebar_position: 2
---

# 文件操作

在GS中，打开的文件实例表示为一种handle类型file。和其他语言一样，在GS中进行文件读写先需要打开文件，获得文件实例；当对文件的操作结束后，关闭文件以释放资源。读写失败时函数返回整数错误码，读取到文件末尾时返回nil。

|序号|函数原型|函数作用|
|:---:|---|---|
|1|`void file.append(string file_name, mixed data)`|添加数据到指定文件|
|2|`mixed file_instance.cntl(string opt_name, mixed opt_value)`|设置打开文件的相关选项，平台相关|
|3|`bool file.delete(string path)`|删除指定文件|
|4|`mixed file_instance.gets(int n = 65535)`|读取文件内容，出错时返回整数错误码，否则返回字符串|
|5|`string file.get_normalize_path(string path)`|规范化文件路径|
|6|`string file.get_os_path_from_script_path(string path)`|将脚本路径转换为操作系统绝对路径|
|7|`string file.get_script_path_from_os_path(string path)`|将操作系统路径转换为脚本路径|
|8|`file file.get_stdin()`|获取标准输入的文件实例|
|9|`file file.get_stdout()`|获取标准输出的文件实例|
|10|`mixed file_instance.flush()`|刷新缓冲区|
|11|`int file.log(string file_name, string format, ...)`|在文件中记录日志信息|
|12|`file file.open(string file_name, string mode, string? in_memory_file = nil)`|按指定模式打开文件并返回文件handle|
|13|`string file.regular_path(string file_name)`|获取规范路径|
|14|`bool file.rename(string old_name, string new_name)`|文件重命名|
|15|`int file_instance.printf(string fmt ...)`|格式化输出到文件|
|16|`handle file.open_user_io(string device_ob_name)`|打开用户IO|
|17|`mixed file_instance.read(int n, bool line_mode = false)`|读取文件内容，返回buffer|
|18|`int file_instance.seek(int offset, int mode)`|设置文件流位置|
|19|`int file.size(mixed file)`|获取文件大小，亦可用文件实例调用`int file_instance.size()` gs|
|20|`mixed file.stat(mixed file)`|获取文件信息，亦可用文件实例调用`int file_instance.stat()` gs|
|21|`int file_instance.tell()`|获取文件流位置|
|22|`mixed file.read_all(string path, string flag, int start, int size)`|读入整个文件，返回buffer|
|23|`int file.utime(string file_name, int access_time, int modification_time)`|改变文件的访问及修改时间戳|
|24|`int file_instance.write(mixed val)`|向文件写入内容，接受任意类型|
|25|`void, file.write_all(string file_name, mixed data, int offset = 0)`|向指定文件中写入内容，无需打开文件|

```bash
Shell> file fw = file.open("test.txt", "w");
Shell> fw.write("Hello, world\n");
Shell> fw.close();
Shell> fw = file.open("test.txt", "a");   # Open with append mode
Shell> fw.write("New line\n");
Shell> fw.close();
Shell> file fr = file.open("test.txt", "r");
Shell> mixed data = fr.read(5);
Shell> 'is_buffer(data);
true
Shell> 'data
48 65 6C 6C 6F                                     Hello...........
Shell> 'data.length();
5
Shell> 'fr.tell();
5
Shell> fr.seek(0, SeekOrigin.BEGIN);  # Rewind
Shell> 'fr.tell();
0
Shell> fr.read(65535);
Shell> 'fr.read(65535);   # End of file
nil
Shell> fr.close();
Shell>
Shell> 'file.size("test.txt");
22
Shell> 'file.stat("test.txt");
{ /* sizeof() == 5 */
    "path" : "test.txt",
    "size" : 22,
    "time" : 1626428364,
    "is_dir" : false,
    "type" : "file",
}
Shell> 'file.delete("test.txt");
true
Shell>
```
