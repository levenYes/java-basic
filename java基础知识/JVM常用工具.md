# JVM常用工具
1. jps：JVM Process Status Tool，显示指定系统内所有的HotSpot虚拟机进程。
2. jstat: JVM Statistics Monitoring Tool，用于收集HotSpot虚拟机各方面的运行数据。它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
3. jinfo：Configuration Info for Java，显示虚拟机配置信息
4. jmap：Memory Map for Java，生成虚拟机的内存转储快照（heapdump文件）。还可以查询finalize执行队列、Java堆和永久代的详细信息，如空间使用率、当前用的是哪种收集器等。
5. jstack：Stack Trace for Java，显示虚拟机的线程快照。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待，等等。线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者等待着什么资源。

常见面试题：
1. 频繁FULL GC怎么办？
是否人为原因，主动调用gc方法；堆设置太小，导致频繁FULL GC。
2. CPU100%怎么办？
TOP命令看进程；PS命令确定哪个java进程；jstack看哪个线程导致。
