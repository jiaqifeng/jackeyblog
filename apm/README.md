# 综述

从嵌入式转行后做了3年APM,一年开发商业版,2年开发pinpoint,是时候总结一下了.

在我入门的时候,在网络上找不到比较好的资料,导致一路很多东西都是自己摸索的.在pinpoint的QQ群中,也看到大家总是提出很多问题,看得出大部分人对很多基本的概念也不太理解,所以有必要写出来和大家共享一下.

下面只是讨论技术,无意冒犯任何人.如果有什么地方写的不合适,无意冒犯了谁,可私信或PR给我,我会更正过来.

# 什么是APM

全称是Application Performance Management & Monitoring,翻译过来就是应用性能管理和监控.目前国内国外用多家公司在此领域深耕,有名的外国公司有NewRelic, Dynatrace, AppDynamic, 老牌企业IBM, CA, HP也都有相关产品, 国内的公司就不说了,大家上网查查就知道了.

从我的经历而言,狭义的APM是指,能够深入应用内部,取得性能相关数据后,完成性能展示/性能分析/性能定位.所谓"深入内部",即可搜集单个代码函数某次调用的起始时间,有用的参数/返回/异常信息.同时,要求是在生产环境下有效,而不是临时使用各种perf类的工具.一般而言,也不包括对TCP/UDP/IP网络报文时间的监控,网络这块往往会使用专门的设备来监控,而且已经是个很成熟的市场了.

此外,基于APM的技术,也很容易实现各种安全性上的需求,比如检测/预防网络攻击等.

除了函数调用信息外,往往也会提供CPU/MEM/DISK,以及各种VM的运行状态,比如GC/DeadLock之类的信息,以协助定位问题.

由于搜集到的函数调用信息数量巨大,一般会期望对数据做各种汇聚操作后再展示,以及实时对满足特定条件的情况作出报警.

对于非单机应用,比如微服务体系,还需要能够将一次用户访问涉及到的所有服务调用能够串联起来.

Google的Dapper论文推荐大家读一下,中文版可参考 http://bigbully.github.io/Dapper-translation/ .

# 基本原理

## 传递txId

无论是单体应用,或是微服务体系,若要将多个函数调用串联起来,就需要用一个txId将其串联起来.我这里说的txId是个概念,不对应具体实现.在具体实现中可以只是一个UUID,也可以还包括调用深度(depth),采样标志(sampleFlag)等等.

在单一应用单一线程(注:OS级的线程,不是协程)中,txId信息一般会放在类似ThreadLocal中传递.在异步线程或跨应用的RPC中,一般将txId信息加入到RPC报文头中,或者携带异步信息的数据结构/类之中,比如Runnable/Callable.


## 植入收集监控信息的代码的方式

### 基于JVM的应用

对于Java应用,java代码会编译成class文件,其中的可执行部分为字节码.如果把Java比喻成C,那么字节码就是汇编码.最主要额区别就是汇编码主要是面向寄存器分配,而字节码是面向堆栈的.

Java虚拟机提供了Instrumentation功能,也常常被称为javaagent.它相当于在JVM加载类字节码的时候留了一个HOOK,允许你在加载应用代码之前,修改应用的字节码,加入自己的逻辑.大部分APM项目都是使用此功能,有点是对应用开发者而言完全透明,对应用无侵入.需要注意的是,instrument功能在JavaSE5/JavaSE6之间有较大改进,目前见到的产品基本都是基于JavaSE6以上的版本实现的.

### nodejs

javascript可以很方便的支持polyfill能力,即把一个api包装一下再替换原始api即可.问题主要是javascript的容忍度太高,应用或者第三方库也常常做各种polyfill,导致稳定性不像java那样让人放心.

有兴趣的可以去github找一下NewRelic的实现看一看.其代码是我看到的最规范的js代码项目之一,而且对nodejs的版本兼容性也做了很好的处理.

还有一个重要问题是js全部是异步操作,导致所有的系统异步调用,都要处理以便传递txId信息.这个可能导致性能损耗有点大(对比java),但我没有实测过.最新的nodejs版本,提供了更多的调试功能,支持了asycId,基于此来实现txId功能,应该可以改进性能损耗.

### php

我见过的有全部用php脚本实现,在应用启动时初始化.这种方式简单方便,难以推广.更好的办法是在zend层拦截,缺点是复杂度增加,以及处理zend的版本和平台的兼容性问题.

### python/ruby/C#

基本未接触过.

### go/c

此类编译型语言,往往需要修改源代码,至少需要在代码中调用一个初始化的API.

我曾经在面试的时候,有一个面试者对C程序实现了最简单计时功能,但涉及复杂的汇编指令替换和栈处理,还要考虑编译器的优化.一般而言,有成熟的各种perf/trace类的工具可用.

### android/ios

不了解.肯定是要在应用代码中增加一个调用.

# 开源产品

排名不分先后, 有Pinpoint, SkyWalking, Zipkin, 等等.具体可百度.

# pinpoint