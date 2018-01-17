---
title: Java虚拟机(二)：性能监控与故障处理工具
date: 2018-01-15 14:26:35
---

知识、经验是关键基础，数据是依据，工具是运用知识处理数据的手段。这里说的数据包括：运行日志，异常堆栈，GC日志，线程快照（headdump/hprof文件）等。使用适当监控和分析工具可以加快分析数据，定位解决问题的速度。

<!--more-->

####JDK命令行工具

- jps：显示HotSpot虚拟机进程

- jstat：用于收集HotSpot虚拟机各方面的运行数据

- jinfo：显示虚拟机配置信息

- jmap：生成虚拟机的内存转储快照 （heapdump文件）

- jhat：用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器查看分析结果

- jstack：显示虚拟机的线程快照

- JConsole：java监视与管理控制台

  - “内存”页签相当于可视化的jstat命令，用于监视受收集器管理的虚拟机内存的变化趋势。

    ```
    static class OOMObjec {
    	public byte[] placehplder = new byte[64 * 1024];
    }

    public static void fillHeap(int num) throws InterruptedException {
      List<OOMObjec> list = new ArrayList<>();
      for (int i = 0; i < num; i++){
        Thread.sleep(1000);
        list.add(new OOMObjec());
      }
      System.gc();
    }
    ```

  - "线程"页签相当于可视化的jstack命令，遇到线程停顿时可以使用这个页签进行监控分析。

- JVisualVM：多合一故障处理工具。