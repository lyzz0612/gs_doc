---
sidebar_label: '常见问题'
sidebar_position: 2.9
---

# 常见问题

---

<details>
<summary>
Q: 为什么要重新做一个语言，而不是用已有的语言比如c++，java，go，python，javascript，lua等成熟的语言？
</summary>
<ul>
<li>我们不考虑使用静态语言，主要是使用门槛较高，容易在出现bug的时候导致crash等严重问题，并且热更新比较麻烦，要做一般就是做使用共享内存，服务器本身做成无状态的这种方案来做，对于开发来讲会有比较大地心智负担</li>
<li>动态语言方面，比较广泛使用的lua/python/javascript等语言对于多线程支持不友好，如果要充分利用多核性能，必须使用分布式的方案，开多进程来做，或者开多线程，每个线程运行一个脚本的runtime，然后再通过共享内存或网络通信，本质上和分布式的结构没太大区别，且热更新上并不是天然支持，还是要自己做不少工作；除了typescript外的语言类型都过于松散，一些基础类型都没有约束，在项目较大时不容易维护</li>
<li>
我们还是希望有一个动态语言，适合游戏开发，用起来顺手简单简单，逻辑清晰，性能优秀，易于扩展，对多线程友好，可以方便地进行热更新，可以随意连接到后台查看和修改线上数据
</li>
</ul>
</details>

<details>
<summary>
Q: 为什么底层有一套和stl库类似的simple库
</summary>
<ul>
<li>stl库中的数据结构藏得比较深，内存布局我们自己控制不了，gdb调试起来需要依赖一些额外的工作才能比较友好，用simple库实现一套简化版的stl不是出于性能考虑，主要就是为了方便在出问题的时候查看内存，快速定位问题，一些基础的容器类比如list, vector, hash_map(unordered_map)等建议使用simple库，而一些对于调试没有影响或者实现较为复杂的比如std::function, std::thread等没必要在simple中再造一遍轮子</li>
<li>另外有内存分配的stl类型要注意自定义allocator，使用我们自己的allocator，申请和释放效率都会高不少</li>
</ul>
</details>

<details>
<summary>
Q: '域(domain)'、'协程(coroutine)'、'线程(thread)'都是什么？相互之间有什么关系？
</summary>
<ul>
<li>协程（coroutine），从表现上说，其实属于“用户态线程”，GS协程和线程的关系，类似线程和CPU核心的关系。只不过线程是操作系统负责调度，而GS协程是由调度器在用户态进行上下文切换调度。</li>
<li>域（domain）类似锁，或者也可以比喻成一个罩子，任何情况下一个域之中只能有一个协程在执行，非parallel/readonly的array/map通过参数/返回值跨域时也会进行复制，避免两个协程同时操作一个实例。域和协程没有其他特别的关系。</li>
</ul>
</details>

<details>
<summary>
Q: parallel修饰方法有什么意义？
</summary>
<ul>
<li>默认情况下，方法是非parallel的，如果需要调用某个方法，就需要进行跨域操作，将协程切换绑定到方法所属object所在的域，而跨域操作往往意味着参数和返回值的deep_dup等操作，开销较大。</li>
<li>因此对于一些不涉及线程安全问题，或者做了额外保障的方法，用parallel修饰可以避免跨域操作，一方面降低开销，另一方面让这个方法可以被多个协程同时调用。</li>
<li>调用parallel方法时，协程仍然保持在之前的域中。</li>
</ul>
</details>

<details>
<summary>
Q: `readonly/parallel`修饰的变量属于什么域？
</summary>
<ul>
<li>
简单类型的值（int, float, map, array 等）本身不属于任何域，readonly/parallel修饰的成员变量有两个主要目的：

```
1. 用于在编译期保证读取和写入有额外的安全保证。

2. 用于保证这个变量只能保存readonly/parallel的实例（特指array/map这样的引用类型需要先make_readonly/make_parallel之后才能被赋值给这些成员变量）。
```

</li>
<li>如果一个对象是parallel的，那么它的成员变量也必须是parallel/readonly的，以此保证同样一份实例不会发生同时操作——保障线程安全。</li>
</ul>
</details>

<details>
<summary>
Q: 使用parallel修饰匿名函数有何意义？
</summary>
<ul>
<li>用parallel修饰的匿名函数保证其执行时不需要跨域，如同parallel修饰的方法一样。</li>
<li>需要注意的是parallel修饰的是匿名函数的函数体，而不是产生的function实例，闭包也是同理。如果有需要还是要使用 `make_parallel(parallel (){...})`。</li>
</ul>
</details>
