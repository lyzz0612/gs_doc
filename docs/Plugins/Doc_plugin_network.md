---
sidebar_label: '网络通信'
sidebar_position: 5
---


# network - 网络通信插件

## 概述
本插件提供network的命令接口。

## 功能使用说明
___________________________________________________________________________________________________________________________
下面列出network类型一些常用的外部函数以及用法。

|序号|函数原型|函数作用|
| :---: | --- | --- |
|1|socket network.bind_to_port(handle h, int port_type)                |socket绑定端口号                   |
|2|void network.close_port(handle port_no)                             |关闭端口号                         |
|3|map network.get_port_info(handle port_no)                           |获取端口信息                       |
|4|int network.get_send_queue_length(handle port_no)                   |获取端口的发送队列大小              |
|5|int network.get_send_queue_max_length(handle port_no)               |获取端口的最大发送队列大小           |
|6|bool network.is_port_alive(handle port_no)                          |端口是否打开                       |
|7|bool network.send_to_port(handle port_no, int msg_no, buffer buf)   |发送数据                           |
|8|bool network.send_to_port_batch(handle port_no, array buf_arr)      |批量发送数据包                      |
|9|void network.set_callback(function conn_func, function disconn_func)|设置回调                           |
|10|int network.set_port_attrib(handle port_no, map port_attrib)       |设置端口属性                       |
|11|bool network.set_port_callback(handle port_no, function conn_func, function disconn_func, function message_func)|设置端口回调|
|12|int network.set_send_queue_max_length(handle port_no, int len)     |设置端口的最大发送队列大小           |
|13|handle network.start_service_on(int port, int max_conn, function conn_func, function disconn_func)|开启网络服务|
___________________________________________________________________________________________________________________________
**注：第1项port_type可取三个值：`1 << 12`, `1 << 13`和`1 << 14`，分别表示kClientUser, kInternalComm和kFromServer。**

**例子:[net](https://m68gitlab.g-bits.com/pkgs/net)**
