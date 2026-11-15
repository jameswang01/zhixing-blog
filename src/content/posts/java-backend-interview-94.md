---
title: "Java 后端面试复习指南"
description: "按七个方向浏览 Java 后端面试复习题，进入对应专题查看完整答案。"
date: "2026-11-15"
category: "Java 后端"
tags: ["Java", "面试准备", "目录"]
---

> 原有 94 道题现按方向整理成独立专题；题号、答案和本页原有锚点予以保留。

## 1 Java 基础与并发

[进入「Java 基础与并发」专题 →](../java-foundations-concurrency/)

### 1 ArrayList 和 LinkedList 如何选择

[查看这道题的答案 →](../java-foundations-concurrency/#1-arraylist-和-linkedlist-如何选择)

### 2 如何解决 Hash 冲突 HashMap 怎样处理

[查看这道题的答案 →](../java-foundations-concurrency/#2-如何解决-hash-冲突-hashmap-怎样处理)

### 3 HashMap 在多线程下会出现哪些问题

[查看这道题的答案 →](../java-foundations-concurrency/#3-hashmap-在多线程下会出现哪些问题)

### 4 volatile 和 synchronized 分别保证什么

[查看这道题的答案 →](../java-foundations-concurrency/#4-volatile-和-synchronized-分别保证什么)

### 5 Condition 与 Object.wait notify 有什么区别

[查看这道题的答案 →](../java-foundations-concurrency/#5-condition-与-objectwait-notify-有什么区别)

### 6 sleep yield join interrupt 分别做什么

[查看这道题的答案 →](../java-foundations-concurrency/#6-sleep-yield-join-interrupt-分别做什么)

### 7 线程安全的单例有哪些实现

[查看这道题的答案 →](../java-foundations-concurrency/#7-线程安全的单例有哪些实现)

### 8 常见线程池与 ThreadPoolExecutor 七个参数是什么

[查看这道题的答案 →](../java-foundations-concurrency/#8-常见线程池与-threadpoolexecutor-七个参数是什么)

### 9 四种拒绝策略分别怎样处理任务

[查看这道题的答案 →](../java-foundations-concurrency/#9-四种拒绝策略分别怎样处理任务)

### 10 线程池有哪些状态 怎样正确关闭

[查看这道题的答案 →](../java-foundations-concurrency/#10-线程池有哪些状态-怎样正确关闭)

### 11 AQS 是什么 常见工具怎样使用 state

[查看这道题的答案 →](../java-foundations-concurrency/#11-aqs-是什么-常见工具怎样使用-state)

### 12 Java SPI 怎样发现实现

[查看这道题的答案 →](../java-foundations-concurrency/#12-java-spi-怎样发现实现)

### 13 反射和 JavaBeans 内省是什么关系

[查看这道题的答案 →](../java-foundations-concurrency/#13-反射和-javabeans-内省是什么关系)

### 14 新增 == equals hashCode 的契约及 String 不可变性

[查看这道题的答案 →](../java-foundations-concurrency/#14-新增--equals-hashcode-的契约及-string-不可变性)

### 15 新增 ConcurrentHashMap 如何保证线程安全

[查看这道题的答案 →](../java-foundations-concurrency/#15-新增-concurrenthashmap-如何保证线程安全)

### 16 新增 ThreadLocal 原理及线程池中的泄漏风险

[查看这道题的答案 →](../java-foundations-concurrency/#16-新增-threadlocal-原理及线程池中的泄漏风险)

### 17 新增 CAS 原理 ABA 问题与适用边界

[查看这道题的答案 →](../java-foundations-concurrency/#17-新增-cas-原理-aba-问题与适用边界)

### 18 新增 死锁的必要条件 预防与定位

[查看这道题的答案 →](../java-foundations-concurrency/#18-新增-死锁的必要条件-预防与定位)

## 2 JVM 与诊断

[进入「JVM 与诊断」专题 →](../java-jvm-diagnostics/)

### 19 JDK 8 为什么用元空间替代永久代

[查看这道题的答案 →](../java-jvm-diagnostics/#19-jdk-8-为什么用元空间替代永久代)

### 20 Tomcat 为什么自定义类加载器

[查看这道题的答案 →](../java-jvm-diagnostics/#20-tomcat-为什么自定义类加载器)

### 21 Java GC 的基本过程怎样回答

[查看这道题的答案 →](../java-jvm-diagnostics/#21-java-gc-的基本过程怎样回答)

### 22 G1 怎样尽量控制停顿时间

[查看这道题的答案 →](../java-jvm-diagnostics/#22-g1-怎样尽量控制停顿时间)

### 23 怎样排查 Java 进程 CPU 持续过高

[查看这道题的答案 →](../java-jvm-diagnostics/#23-怎样排查-java-进程-cpu-持续过高)

### 24 新增 JVM 运行时内存区域怎样划分

[查看这道题的答案 →](../java-jvm-diagnostics/#24-新增-jvm-运行时内存区域怎样划分)

### 25 新增 类加载过程与双亲委派是什么

[查看这道题的答案 →](../java-jvm-diagnostics/#25-新增-类加载过程与双亲委派是什么)

### 26 新增 对象创建与内存分配有哪些要点

[查看这道题的答案 →](../java-jvm-diagnostics/#26-新增-对象创建与内存分配有哪些要点)

### 27 新增 怎样诊断 OOM 与频繁 Full GC

[查看这道题的答案 →](../java-jvm-diagnostics/#27-新增-怎样诊断-oom-与频繁-full-gc)

## 3 MySQL 事务 索引与复制

[进入「MySQL 事务与索引」专题 →](../java-mysql/)

### 28 ACID 分别靠什么保证

[查看这道题的答案 →](../java-mysql/#28-acid-分别靠什么保证)

### 29 四种事务隔离级别和三类读异常是什么

[查看这道题的答案 →](../java-mysql/#29-四种事务隔离级别和三类读异常是什么)

### 30 MVCC 快照读 当前读与间隙锁如何配合

[查看这道题的答案 →](../java-mysql/#30-mvcc-快照读-当前读与间隙锁如何配合)

### 31 redo undo binlog 有什么区别

[查看这道题的答案 →](../java-mysql/#31-redo-undo-binlog-有什么区别)

### 32 B 树与 B+ 树有什么区别 为什么适合数据库索引

[查看这道题的答案 →](../java-mysql/#32-b-树与-b-树有什么区别-为什么适合数据库索引)

### 33 为什么常用短小 递增的主键

[查看这道题的答案 →](../java-mysql/#33-为什么常用短小-递增的主键)

### 34 MySQL 主从复制怎么工作 为什么会延迟

[查看这道题的答案 →](../java-mysql/#34-mysql-主从复制怎么工作-为什么会延迟)

### 35 主从延迟时 如何保证写后读和业务正确性

[查看这道题的答案 →](../java-mysql/#35-主从延迟时-如何保证写后读和业务正确性)

### 36 哪些情况可能不使用预期索引

[查看这道题的答案 →](../java-mysql/#36-哪些情况可能不使用预期索引)

### 37 如何有依据地优化慢 SQL

[查看这道题的答案 →](../java-mysql/#37-如何有依据地优化慢-sql)

### 38 大分页为什么慢 如何优化

[查看这道题的答案 →](../java-mysql/#38-大分页为什么慢-如何优化)

### 39 新增 InnoDB 死锁如何形成 定位和处理

[查看这道题的答案 →](../java-mysql/#39-新增-innodb-死锁如何形成-定位和处理)

### 40 新增 EXPLAIN 怎么看 覆盖索引 回表和索引下推如何串起来

[查看这道题的答案 →](../java-mysql/#40-新增-explain-怎么看-覆盖索引-回表和索引下推如何串起来)

## 4 Redis 编码 内存与高可用

[进入「Redis 与缓存」专题 →](../java-redis-cache/)

### 41 五种常见数据类型的底层编码是什么

[查看这道题的答案 →](../java-redis-cache/#41-五种常见数据类型的底层编码是什么)

### 42 压缩列表 listpack 字典和跳表分别解决什么问题

[查看这道题的答案 →](../java-redis-cache/#42-压缩列表-listpack-字典和跳表分别解决什么问题)

### 43 Redis 如何删除过期键

[查看这道题的答案 →](../java-redis-cache/#43-redis-如何删除过期键)

### 44 常见八种内存淘汰策略如何分类

[查看这道题的答案 →](../java-redis-cache/#44-常见八种内存淘汰策略如何分类)

### 45 主从与 Sentinel 有什么区别

[查看这道题的答案 →](../java-redis-cache/#45-主从与-sentinel-有什么区别)

### 46 Redis Cluster 如何分片 有哪些限制

[查看这道题的答案 →](../java-redis-cache/#46-redis-cluster-如何分片-有哪些限制)

### 47 Redis 全量复制与部分重同步如何工作

[查看这道题的答案 →](../java-redis-cache/#47-redis-全量复制与部分重同步如何工作)

### 48 新增 RDB AOF 如何取舍 恢复时有哪些持久性边界

[查看这道题的答案 →](../java-redis-cache/#48-新增-rdb-aof-如何取舍-恢复时有哪些持久性边界)

### 49 新增 缓存穿透 击穿 雪崩分别怎么治理

[查看这道题的答案 →](../java-redis-cache/#49-新增-缓存穿透-击穿-雪崩分别怎么治理)

### 50 新增 缓存与数据库一致性如何设计

[查看这道题的答案 →](../java-redis-cache/#50-新增-缓存与数据库一致性如何设计)

### 51 新增 Pipeline MULTI EXEC WATCH 与 Lua 有什么区别

[查看这道题的答案 →](../java-redis-cache/#51-新增-pipeline-multi-exec-watch-与-lua-有什么区别)

### 52 新增 如何识别和治理热 key 大 key

[查看这道题的答案 →](../java-redis-cache/#52-新增-如何识别和治理热-key-大-key)

### 53 新增 Redis 为什么快 单线程边界和阻塞风险是什么

[查看这道题的答案 →](../java-redis-cache/#53-新增-redis-为什么快-单线程边界和阻塞风险是什么)

## 5 Spring 与 MyBatis

[进入「Spring 与消息队列」专题 →](../java-framework-messaging/)

### 54 Spring MVC 的请求处理流程

[查看这道题的答案 →](../java-framework-messaging/#54-spring-mvc-的请求处理流程)

### 55 七种事务传播行为与实现原理

[查看这道题的答案 →](../java-framework-messaging/#55-七种事务传播行为与实现原理)

### 56 事务未生效或没有回滚时怎样排查

[查看这道题的答案 →](../java-framework-messaging/#56-事务未生效或没有回滚时怎样排查)

### 57 JDK 动态代理与 CGLIB 的区别

[查看这道题的答案 →](../java-framework-messaging/#57-jdk-动态代理与-cglib-的区别)

### 58 循环依赖与三级缓存的边界

[查看这道题的答案 →](../java-framework-messaging/#58-循环依赖与三级缓存的边界)

### 59 Spring 与 Spring MVC 的父子容器

[查看这道题的答案 →](../java-framework-messaging/#59-spring-与-spring-mvc-的父子容器)

### 60 MyBatis 怎样记录执行耗时

[查看这道题的答案 →](../java-framework-messaging/#60-mybatis-怎样记录执行耗时)

### 61 新增 IoC DI 与 Bean 生命周期怎样串起来

[查看这道题的答案 →](../java-framework-messaging/#61-新增-ioc-di-与-bean-生命周期怎样串起来)

### 62 新增 Spring 单例 Bean 为什么不一定线程安全

[查看这道题的答案 →](../java-framework-messaging/#62-新增-spring-单例-bean-为什么不一定线程安全)

### 63 新增 Spring AOP 怎样工作 常见通知有什么区别

[查看这道题的答案 →](../java-framework-messaging/#63-新增-spring-aop-怎样工作-常见通知有什么区别)

### 64 新增 Spring Boot 自动配置与 starter 是什么关系

[查看这道题的答案 →](../java-framework-messaging/#64-新增-spring-boot-自动配置与-starter-是什么关系)

### 65 新增 MyBatis 参数绑定与字符串替换有什么区别

[查看这道题的答案 →](../java-framework-messaging/#65-新增-mybatis-参数绑定与字符串替换有什么区别)

## 6 消息队列与 Kafka

[进入「Spring 与消息队列」专题 →](../java-framework-messaging/)

### 66 MQ 有什么作用 代价是什么

[查看这道题的答案 →](../java-framework-messaging/#66-mq-有什么作用-代价是什么)

### 67 Kafka 为什么能够实现高吞吐

[查看这道题的答案 →](../java-framework-messaging/#67-kafka-为什么能够实现高吞吐)

### 68 Kafka 如何降低消息丢失风险

[查看这道题的答案 →](../java-framework-messaging/#68-kafka-如何降低消息丢失风险)

### 69 Kafka 如何保证业务消息的顺序

[查看这道题的答案 →](../java-framework-messaging/#69-kafka-如何保证业务消息的顺序)

### 70 至多一次 至少一次 恰好一次怎么区分

[查看这道题的答案 →](../java-framework-messaging/#70-至多一次-至少一次-恰好一次怎么区分)

### 71 Kafka 事务消息的核心流程是什么

[查看这道题的答案 →](../java-framework-messaging/#71-kafka-事务消息的核心流程是什么)

### 72 Kafka 为什么移除 ZooKeeper KRaft 做了什么

[查看这道题的答案 →](../java-framework-messaging/#72-kafka-为什么移除-zookeeper-kraft-做了什么)

## 7 分布式事务

[进入「分布式与系统设计」专题 →](../java-distributed-systems/)

### 73 2PC 是什么 有哪些主要问题

[查看这道题的答案 →](../java-distributed-systems/#73-2pc-是什么-有哪些主要问题)

### 74 3PC 与 2PC 有什么区别

[查看这道题的答案 →](../java-distributed-systems/#74-3pc-与-2pc-有什么区别)

### 75 TCC 是什么 适用于什么场景

[查看这道题的答案 →](../java-distributed-systems/#75-tcc-是什么-适用于什么场景)

### 76 TCC 如何处理幂等 空回滚和悬挂

[查看这道题的答案 →](../java-distributed-systems/#76-tcc-如何处理幂等-空回滚和悬挂)

### 77 新增 CAP 与 BASE 如何用于系统选型

[查看这道题的答案 →](../java-distributed-systems/#77-新增-cap-与-base-如何用于系统选型)

## 8 分布式锁与 RPC

[进入「分布式与系统设计」专题 →](../java-distributed-systems/)

### 78 如何用 MySQL 实现分布式锁

[查看这道题的答案 →](../java-distributed-systems/#78-如何用-mysql-实现分布式锁)

### 79 Redis 分布式锁怎样实现 边界是什么

[查看这道题的答案 →](../java-distributed-systems/#79-redis-分布式锁怎样实现-边界是什么)

### 80 ZooKeeper 分布式锁的流程是什么

[查看这道题的答案 →](../java-distributed-systems/#80-zookeeper-分布式锁的流程是什么)

### 81 Dubbo 的服务暴露 引入和调用如何串起来

[查看这道题的答案 →](../java-distributed-systems/#81-dubbo-的服务暴露-引入和调用如何串起来)

### 82 常见负载均衡算法如何选择

[查看这道题的答案 →](../java-distributed-systems/#82-常见负载均衡算法如何选择)

## 10 业务设计与稳定性

[进入「分布式与系统设计」专题 →](../java-distributed-systems/)

### 90 未支付订单怎样自动取消

[查看这道题的答案 →](../java-distributed-systems/#90-未支付订单怎样自动取消)

### 91 怎样建设系统稳定性

[查看这道题的答案 →](../java-distributed-systems/#91-怎样建设系统稳定性)

### 92 新增 接口幂等怎样抵御重复请求与并发

[查看这道题的答案 →](../java-distributed-systems/#92-新增-接口幂等怎样抵御重复请求与并发)

### 93 新增 本地事务与 MQ 发消息如何保持一致

[查看这道题的答案 →](../java-distributed-systems/#93-新增-本地事务与-mq-发消息如何保持一致)

### 94 新增 限流 熔断 降级有什么区别 重试为何放大故障

[查看这道题的答案 →](../java-distributed-systems/#94-新增-限流-熔断-降级有什么区别-重试为何放大故障)

## 9 网络与身份认证

[进入「网络基础」专题 →](../java-networking/)

### 83 HTTPS 怎样保证传输安全

[查看这道题的答案 →](../java-networking/#83-https-怎样保证传输安全)

### 84 HTTP 各版本的主要变化

[查看这道题的答案 →](../java-networking/#84-http-各版本的主要变化)

### 85 TCP 为什么三次握手和通常四次挥手

[查看这道题的答案 →](../java-networking/#85-tcp-为什么三次握手和通常四次挥手)

### 86 Cookie Session Token 与 JWT

[查看这道题的答案 →](../java-networking/#86-cookie-session-token-与-jwt)

### 87 新增 TCP 如何保证可靠传输 流量控制与拥塞控制有何区别

[查看这道题的答案 →](../java-networking/#87-新增-tcp-如何保证可靠传输-流量控制与拥塞控制有何区别)

### 88 新增 输入网址到得到响应经历什么 怎样定位慢请求

[查看这道题的答案 →](../java-networking/#88-新增-输入网址到得到响应经历什么-怎样定位慢请求)

### 89 新增 BIO NIO AIO 与多路复用怎样区分

[查看这道题的答案 →](../java-networking/#89-新增-bio-nio-aio-与多路复用怎样区分)

## 参考资料

各专题文章附有相关参考资料。
