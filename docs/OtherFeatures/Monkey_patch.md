---
sidebar_label: '热更新'
sidebar_position: 2
---

# monkey_patch


热更新可以快速修复bug、改善用户体验。
___________________________________________________________________________________________________________________________

gs语言支持热更新，底层热更新的处理过程是这样的：

* 暂停gc

* 挂起所有协程（除shell和当前协程外）

* 清理所有program的函数调用缓存（m_call_caches）

* 编译patch文件，生成new program，将old program指向new program，old function指向new function，为new program的全局变量建立新旧对应关系（m_monkey_patch_object_var_map）

* 若有全局变量发生变化，对所有依赖该program的object，更新全局变量的补丁信息（m_extend_object_vars），根据新program的m_monkey_patch_object_var_map，将变量的旧值拷贝到m_extend_object_vars中。热更新完成后，后续对全局变量的操作都将在m_extend_object_vars上进行

* 恢复挂起的协程

* 开启gc

热更新成功之后立即生效。

注意：patch不能改变原始文件的component布局。
___________________________________________________________________________________________________________________________



举个例子：
```
const string src = __SRC_PREFIX__ """P
public int test()
{
	return 1;
}

public int get()
{
	return 2;
}
"""P;

const string src2 = __SRC_PREFIX__ """P
int i = 1;

public int test()
{
    return i + i;
}

"""P;

compile_program("/p1.gs", src, true);
object ob = new_object("/p1.gs", this_domain());
write(ob.test()); //1
write(ob.get()); //2

monkey.patch("/p1.gs",src2);

write(ob.test());//0
write(ob.get());//error occured
```

上面的程序中，第一次调用ob.test()执行的是src中的程序，第二次调用ob.test()执行的是src2中的程序，返回0，是因为补丁中新增的int类型全局变量"i"默认初始化成了0。

第一次调用ob.get()返回2，patch后第二次调用时报错，是因为src2中没有get()这个接口了。