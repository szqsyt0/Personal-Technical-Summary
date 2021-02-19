## JVM性能监控与故障处理工具

- jps

  jps位于jdk的bin目录下，用于显示当前系统的java进程及其id

- jstack

  jstack用于生成JVM当前时刻的线程快照，用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源的长时间等待等

  ```bash
  jstack [ option ] pid
  Options:
  -F 当’jstack [-l] pid’没有响应的时候强制打印栈信息
  -l 长列表，打印关于锁的附加信息
  -m 打印java和native c/c++框架的所有栈信息
  ```

- jmap

  jmap用于生成堆转储快照，即dump文件

  ```bash
  jmap [ option ] pid
  Options:
  -heap：打印jvm heap的情况
  -histo：打印jvm heap的直方图。其输出信息包括类名，对象数量，对象占用大小
  -histo：live ：同上，但是只打印存活对象的情况
  -dump:<dump-options>: 生成hprof格式的dump文件
        dump-options：
          live: 只dump存活对象
          format=b: 二进制格式
          file=<file>: 指定dump文件路径
  -F: 强制dump heap信息
  ```

- jstat

  jstat用于对Java应用程序的资源和性能进行实时的命令行监控，包括对heap size和垃圾回收状况的监控

  ```bash
  jstat -class pid 显示加载class的数量，及所占空间
  jstat -compiler pid 显示VM实时编译的数量等信息
  jstat -gc pid 显示gc信息
  ```

- jconsole

  可视化Java应用程序监控工具，可以实时显示CPU使用了、内存、线程、类、VM等信息

  ![img](../../../%25E7%259F%25A5%25E8%25AF%2586%25E7%25AE%25A1%25E7%2590%2586/assets/98860363-DDB9-4C0C-8FF3-AE2016201279.png)

- jinfo

  用于查看和调整虚拟机的各项参数

  ```bash
  jinfo [ option ] pid
  no option 打印命令行参数和系统属性
  -flags 打印命令行参数
  -sysprops 打印系统属性
  ```

- jhat

  jhat是java虚拟机自带的一种虚拟机堆转储快照分析工具



## 参考

《深入理解Java虚拟机_JVM高级特性与最佳实践（第3版）》
