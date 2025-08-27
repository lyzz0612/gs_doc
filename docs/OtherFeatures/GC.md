---
sidebar_label: '垃圾回收'
sidebar_position: 5
---

## 垃圾回收

在GS中，垃圾回收机制是自动执行的，通过可达性分析来确定哪些对象可以被回收，当一个对象不再被任何引用时，垃圾回收器会自动将其标记为可回收对象。并在独立的GC协程中释放内存。

GS支持分代回收机制，当满足以下条件时，GS会触发一次垃圾回收：

* 当执行 `gc.start(false)` 时，触发一次 Minor GC。
* 当执行 `gc.start(true)` 时，触发一次 Full GC。
* 当新分配的内存（即自上一次GC完成之后的内存增长量）**大于或等于** MinorGC 阈值<sup>1</sup>时，触发一次 Minor GC。
* 当新分配的内存（即自上一次GC完成之后的内存增长量）**满足** FullGC 触发条件<sup>2</sup>时，触发一次 Full GC。
* 当尝试为GC实例分配内存失败时，触发一次 Full GC。

---
注释：
1. MinorGC 阈值默认为：

    ```
    OLD_SIZE / 16 + YOUNG_SIZE / 2
    ```

    其中，`OLD_SIZE` 是当前的老年代体积，`YOUNG_SIZE` 是上一轮GC完成后的年轻代体积。如果距离上一次GC的时间少于 5ms 并且有足够的 GC 工作线程，实际检查时还会有额外的修正系数，以避免过于频繁的GC触发。

2. FullGC 触发条件为：

    **先决条件**
    
    新分配的内存大于等于：

    ```
    LAST_FULL_GC_OLD_SIZE + OLD_SIZE / 16 + YOUNG_SIZE / 2
    ```

    **满足先决条件后**，还需满足下列三个条件之一：

    ```
    OLD_SIZE >= LAST_FULL_GC_OLD_SIZE * 2
    ```

    ```
    距离上一次 FullGC 3秒以上，且 OLD_SIZE >= LAST_FULL_GC_OLD_SIZE * 3/2
    ```

    ```
    距离上一次 FullGC 180秒以上，且 OLD_SIZE >= LAST_FULL_GC_OLD_SIZE * 5/4
    ```
    
    LAST_FULL_GC_OLD_SIZE 是上一次 Full GC 之后的老年代体积。

### GC配置

可以通过命令行参数配置GC的阈值，常用的相关的配置项如下：

```
--gc-quota-to-minor-gc <MinorGC 阈值>

单位为字节，合法取值范围为 16KByte 到 1TByte；用于替换默认的 MinorGC 阈值计算公式。
```

```
--gc-count-upper-bound <单次内存计数分配上界>

设置单次内存分配计数的上限，单位是字节，默认值为 1MByte；设置为0表示不设上限。

当一次分配超过此值的内存大小时，计数最大只会计入此值，超过部分不计入内存分配计数。

此机制能降低部分场景下的GC触发频率，但如果设置得过低会抑制GC的触发，导致内存占用偏高。
```
