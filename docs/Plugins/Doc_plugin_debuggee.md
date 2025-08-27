---
sidebar_label: 'vscode调试插件'
sidebar_position: 2
---


# debuggee - VsCode调试插件

## 概述
桥接编译器debugger与vscode-debug-adapter，实现在vscode界面中的调试功能

## 主要内容

### EFUN
```cpp
/**
 * 启用debugger并设置端口，相对于启动参数 --enable-debugger <listen_port>::<pause_when_start>::<remote_port> (后两个参数为可选)
 * @param listen_port 监听端口，监听vscode发送的请求
 * @param remote_prot vscode端的监听端口，有默认值，若vscode端设置了自定义值，则在debugger中也要保持一致
 */
void dbg.enable_module_debuggee(int listen_port, int remote_port = -1);
```

该函数一般用在运行中的服务器（未开启enable-debugger功能）中，启动debugger并设置监听端口后，vscode端的调试框架便能够attach到服务器并发送调试请求

### 通信
debuggee插件启用时会创建两个协程:

* 用来监听调试请求的**listen_from_gstools**:
  * 即socket服务端，vscode的debug-adapter通过与其建立短连接来发送调试操作请求并接收响应
  * 目前主要的调试请求包括（在vscode调试界面都有相应的操作选项）：
    * Start 开始调试
    * Pause 暂停
    * Continue 继续
    * StepIn 逐语句
    * StepOver 逐过程
    * StepOut 跳出
    * Stop 停止调试
    * AddBreakpoint 添加断点
    * AddFileBreakpoints 批量添加文件中断点
    * DisableBreakpoint 删除断点
    * InfoWatchVariable 获取变量信息
    * InfoThreads 获取协程信息
    * InfoPath 获取driver运行环境路径信息
    * AttachToProcess 附加
    * DeleteAllBreakpoint 删除所有断点

* 用来发送trap等事件的**send_to_gstools**:
  * 即socket客户端，将调试运行过程中的事件（主要是trap状态下的堆栈帧）发送给debug-adapter

### 调试器
debuggee插件定义了一个Debugger的子类DebuggerBridge，用于适配vscode调试框架

## GsLang调试说明

[参数配置](https://wiki.g-bits.com/pages/viewpage.action?pageId=594980681) (待完善)

[运行/调试功能例子：mt_random](https://leiting.feishu.cn/docs/doccnScBfiJLgXm6dy6jhi25C1e)

[运行/调试功能例子：server0](https://leiting.feishu.cn/docs/doccnqrxGOHeIPWcpsQIw7bOnyc)