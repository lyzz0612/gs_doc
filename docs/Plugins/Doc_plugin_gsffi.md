---
sidebar_label: 'FFI'
sidebar_position: 15
---

# FFI

## 概述

ffi实际上不是一个插件，而是gs的核心库组件，其定义了一系列静态函数，并通过核心内嵌脚本（*internal scripts*）中的gsffi等进行封装

如何通过FFI的方式支持plugins
==========================

什么是FFI?
----------  
FFI(Foregin Function Interface) 是用来与其他语言交互的接口。由于现实中很多程序是由不同的编程语言写的，必然会涉及到跨语言调用，比如A语言写的函数如果想在B语言中调用，这时就可以通过FFI调用，可以直接将A语言的接口内嵌到B语言中直接调用。

dll是怎么提供给gsffi的?
----------------------
[Group pkgs](https://m68gitlab.g-bits.com/pkgs)下面放了许多``pkg``的源码，在``pkg``中编译`cpp`目录下的源码生成动态库文件然后拷贝到`src`中, 并提供对应API的ffi文件，提供给gsffi解析。例如excel、glad、glft、toml等。  

ffi使用技巧
-----------
1. ffi文件（函数声明）  
    ffi文件模板如下，在module中写需要使用的API的函数声明。  
    如果源码中有对结构体的宏定义，需要在module前面标明。

    ```cpp
    typedef type_name1 other_name1;
    typedef type_name2 other_name2;
    ...

    //functions
    module(dll_name)
    {
        return_type function_name1 (argument_list1);
        return_type function_name2 (argument_list2);
        ...
    }
    ```
    在gs文件中会调用``gsffi.load_library(string name, string path, domain dom, string st_mod)``  
    解释一下这个函数各个参数的意义：  
    name是一个虚拟路径，如``"/_dll/xxx.gs"``，且必须是``"/_dll/"``开头的。  
    path是ffi文件的路径，load_library方法会对ffi文件进行编译。   
    st_mode一般传入当前文件名，结构体声明一般也约定写在这个文件中。load_library也会导入这个文件中的struct。
    dom传入当前域，最后"/_dll/xxx.gs"对象将会创建在这个域里。
2. 结构体的声明   
    结构体声明格式如下, 和在源码中定义的struct基本一致。需要注意的是，union直接声明在gs中无法识别，写成空指针或者int即可。
    ```cpp
    EMBED_FFI_BEGIN
    struct struct_name{
        type_name var_name;
        ...
    };
    EMBED_FFI_END
    ```
3. 函数的使用  
    1. 调用函数
        在本小结的第1点中提到，我们最后会创建dll对象在当前域里面。可以通过dll对象直接调用函数。
    2. 对于指针传递的函数  
        需要找到对应类型的``new_ptr(); new_int64(); new_double();``或者在知道数据结构size的情况下使用``buffer.allocate(size)``得到一个buffer类型传入。  
        前者需要对应方法``get_ptr(); get_int64(); get_double() ``获取到数据。后者则可以通过``data_ptr()``获取到buffer的指针根据数据结构的的size依次还原数据。
    3. 对于返回值为结构体指针的函数
        底层处理的时候实际上是把指针是传回来一个地址，所以可以用int型存储。 
        一般可以通过``ffi.ref_struct(int struct_id, int ptr)``来通过结构体指针还原结构体。struct_id可以通过 ``ffi.get_struct_id(string name, string mod)``来得到结构体id。string mod指声明结构体的文件。
