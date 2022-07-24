---
title: "Prometheus Exporter"
date: 2021-12-05T11:30:03+00:00
tags: ["prometheus"]
series: []
categories: ["时序数据库","数据库"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS:  false
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---


time series metrics collect toolkit

提供了UI服务，默认端口30313

![image.png](https://b3logfile.com/file/2020/12/image-d99479ea.png)

主要作用就是监控一下我们服务的各项指标，也有许多包已经集成了这个功能，基本上查找一下配置一下就可以快速的展示出各项指标了

# 业务指标

但例如 java_gc,go_gc,node_request_total这种指标对我们来说没有特别大的用处，我们的着力点应该在于提取业务指标，各个业务系统数据源不同逻辑不同导致无法使用通用型第三方包来构建我们的指标。

这个时候就出现了 `prometheus-exporter`用来放出自定义的业务指标，等待prometheus来此处收集。

# prometheus exporter

其实很简单，就是攒一些prometheus数据格式的数据放入 `/metrics`路由中等待scrape就好

下面使用Go来举例子

![image.png](https://b3logfile.com/file/2020/12/image-23569ab7.png)

这里使用了godror驱动oracle，prometheus/client_golang 客户端,logrus日志模块

![image.png](https://b3logfile.com/file/2020/12/image-6b8b790c.png)

一些命令行参数

![image.png](https://b3logfile.com/file/2020/12/image-260cc485.png)

main函数主要就是构造exporter对象然后启动http服务

![image.png](https://b3logfile.com/file/2020/12/image-ee6c3853.png)

exporter 结构体中加入了同步锁，封装好的oracle客户端比较重要的是 `metricDescriptions`和 `metricMap`这几个指标map

作为prometheus的exporter需要实现两个接口,一个是输入描述的 `Describe`一个是收集数据的 `Collect`

![image.png](https://b3logfile.com/file/2020/12/image-af83cebb.png)![image.png](https://b3logfile.com/file/2020/12/image-df7bf1e2.png)

Collect，循环map然后将数据和字符串传入ch管道中

![image.png](https://b3logfile.com/file/2020/12/image-76d4b075.png)

Describe相同都是向ch放入构建好的 指针类型对象

接下来看map中到底是什么

![image.png](https://b3logfile.com/file/2020/12/image-989ae789.png)

map为string对应的匿名函数,函数类型为MetricHandler

很好理解，将管道传入匿名函数，结构体传入匿名函数，然后利用结构体的oracleClient执行sql构建Metric结构体传入ch中即可

# Over

整个流程非常简单，由于公司的exporter不可能开源，所以这里提供redis-exporter地址以供参考

https://github.com/oliver006/redis_exporter

2020结束，最后一篇简单的参考教程