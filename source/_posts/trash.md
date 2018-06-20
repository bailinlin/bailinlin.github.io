---
title: 你需要知道的 javascript 垃圾回收机制
date: 2018-06-12 15:11:38
tags:[javascript, 思维导图 ]
---

>我们通常理解的 javascript 垃圾回收机制都停留在表面，"会释放不被引用变量内存"，最近在读《深入浅出node.js》的书，详细了解了下 v8 垃圾回收的算法，记录了一些学习笔记。


### 敲黑板：v8引擎的垃圾回收算法
![this](/images/blog/v8/fds.jpg)

V8的垃圾回收策略主要基于分代式垃圾回收机制，现代的垃圾回收算法中按对象的存活时间将内存的垃圾回收进行不同的分代,然后分别对不同分代的内存施以更高效的算法。在V8中,主要将内存分为新生代和老生代两代。新生代中的对象为存活时间较短的对象, 老生代中的对象为存活时间较长或常驻内存的对象。

### Scavenge算法


>在分代的基础上,新生代中的对象主要通过Scavenge算法进行垃圾回收,在Scavenge的具体 实现中,主要采用了Cheney算法

![this](/images/blog/v8/csf.jpg)

Cheney 算法是一种采用复制的方式实现的垃圾回收算法。它将堆内存一分为二,每一部分空间称为 semispace。在这两个 semispace 空间中,只有一个处于使用中,另一个处于闲置状态。处于使用状态的 semispace 空间称为 From 空间,处于闲置状态的空间称为 To 空间。当我们分配对象时,先是在 From 空间中进行分配。当开始进行垃圾回收时,会检查 From 空间中的存活对象,这 些存活对象将被复制到 To 空间中,而非存活对象占用的空间将会被释放。完成复制后,From 空 间和To空间的角色发生对换。 简而言之, 在垃圾回收的过程中, 就是通过将存活对象在两个 semispace 空间之间进行复制。


### Mark-Sweep & Mark-Compact

>Scavenge算法通过牺牲空间换时间的算法非常适合生命周期短的新生代，但是，当一个对象经过多次复制，生命周期较长的时候或则To空间不足的时候，对象会被分配到进入到老生代中，需要采用新的算法进行垃圾回收。

![this](/images/blog/v8/mssf.jpg)

Mark-Sweep 并不将内存空间划分为两半,所以不存在浪费一半空间的行为。与 Scavenge 复制活着的对象不同, Mark-Sweep 在标记阶段遍历堆中的所有对象,并标记活着的对象,在随后的清除阶段中,只清除没有被标记的对象。可以看出,Scavenge 中只复制活着的对象,而 Mark-Sweep 只清理死亡对象。

Mark-Sweep 在进行一次标记清除回收后,内存空间会出现不连续的状态。这种内存碎片会对后续的内存分配造成问题,因为很可能出现需要分配一个大对象的情况,这时所有的碎片空间都无法完成此次分配,就会提前触发垃圾回收,而这次回收是不必要的。Mark-Compact 对象在标记为死亡后,在整理的过程中,将活着的对象往一端移动,移动完成后,直接清理掉边界外的内存

### 增量标记

为了避免出现 JavaScript 应用逻辑与垃圾回收器看到的不一致的情况,垃圾回收的 3 种基本算法都需要将应用逻辑暂停下来,待执行完垃圾回收后再恢复执行应用逻辑,这种行为被称为“全停顿"，长时间的"全停顿"垃圾回收会让用户感受到明显的卡顿，带来体验的影响。以1.5 GB的垃圾回收堆内存为例,V8做一次小的垃圾回收需要50毫秒以上,做一次非增量式的垃圾回收甚至要1秒以上。这是垃圾回收中引起JavaScript线程暂停执行的时间,在 这样的时间花销下,应用的性能和响应能力都会直线下降。

为了降低全堆垃圾回收带来的停顿时间,V8先从标记阶段入手,将原本要一口气停顿完成的动作改为增量标记(incremental marking),也就是拆分为许多小“步进”,每做完一“步进” 就让 JavaScript 应用逻辑执行一小会儿,垃圾回收与应用逻辑交替执行直到标记阶段完成


### 扩展阅读

[JavaScript 内存泄漏教程](http://www.ruanyifeng.com/blog/2017/04/memory-leak.html)

[精读《深入浅出Node.js》](https://bailinlin.github.io/2018/06/08/node-notes/)

[《深入浅出Node.js》PDF](chrome-extension://ikhdkkncnoglghljlkmcimlnlhkeamad/pdf-viewer/web/viewer.html?file=http%3A%2F%2Fblog.songqingbo.cn%2Fpdf%2Fnodejs%2F%25E6%25B7%25B1%25E5%2585%25A5%25E6%25B5%2585%25E5%2587%25BANode.js.pdf)

