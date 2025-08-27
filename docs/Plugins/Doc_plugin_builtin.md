---
sidebar_label: '内置脚本'
sidebar_position: 1
---

# builtin - 脚本插件

## 概述
本插件可将一些脚本打包到插件中，直接以插件的形式提供外部使用\
如果没有builtin插件，如果我们想要增加一些偏底层的脚本库文件（比如说telnet，性能测试等），我们必须以源码的形式提供，不太方便也不安全

## 实现思路
* 先编译minshell，作为基础的执行环境
* 使用minshell执行script/generate_builtin_file.gs脚本，遍历builtin/builtin_scripts目录，将其中的所有脚本编译为.o，并转为.h文件
* 自动生成auto_gen_builtin_files.hpp文件，其中的_register_all_builtin_files()方法，将所有生成的内嵌脚本数据注册到内存文件设备中
* 当我们导入builtin插件后，我们就能通过 `/v/<文件名>` 访问到已注册的内嵌脚本内容

## 如何扩展
将需要打包的脚本放入builtin/builtin_scripts目录中即可

## 常用builtin脚本

### gs.util.logger
`gs.util.logger` 用于实现日志的记录，可以实现按照配置过滤指定的日志等级等辅助调试的功能。

| 函数名 | 说明 |
|:----|----|
| `void logger.init(map params)` | 使用参数指定的配置初始化logger，具体的参数说明见说明1 |
| `void logger.set_log_colors(map colors)` | 使用参数配置对应等级使用的日志标志颜色控制字符，参数的map，key应该是日志等级枚举，对应的value是颜色控制字符（例如 `HIR`） |
| `void logger.set_log_level(enum LogLevel level)` | 设置全局关注的日志等级，若日志等级低于指定的等级则不记录 |
| `void logger.add_focus_program(string path)` | 设置关注的program路径，只有日志相关的调用栈涉及到此program时才记录。 |
| `void logger.remove_focus_program(string path)` | 通过路径移除指定的关注program |
| `array logger.get_focus_programs()` | 获取所有正在关注的program路径 |
| `array logger.get_focus_programs()` | 获取所有正在关注的program路径 |
| `void logger.log(enum LogLevel level, string msg, bool with_prefix = true)` | 以`level`指定的日志等级记录`msg`到日志中，若`with_prefix`为false，则输出的日志不带有对应的等级前缀 |

* 说明：
  1. logger初始化时的参数map应该如下所示：
  ```
  {
    "log_level" : 最低日志等级
    "log_device" : 需要输出日志的文件或设备句柄，可以是由句柄组成的数组，
    "log_prefix" : 输出日志条目的前缀设置，默认是 : "auto"
         "auto" : 使用日志等级作为前缀
         "no"   : 没有前缀
         others : 使用配置项本身作为前缀
    "time_format" : 设置时间的格式 (TimeType enum, 参考 man("ctime")), 默认 : nil (没有时间格式)
    "enable_color" : 是否允许日志等级颜色，默认 : true
    "log_level_configs" : 针对具体的日志等级进行配置
        {
          <log_level> : {
            log_device : Device for specified log level
            log_prefix : Prefix for speicified log level
            time_format: Time format for specified log level
        }
    }
  }
  ```