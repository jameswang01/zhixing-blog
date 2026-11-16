---
title: "Java 后端面试：网络基础"
description: "HTTP、TCP、TLS、认证与 I/O；共 7 道复习题。"
date: "2026-11-15"
category: "Java 后端"
javaDirection: "network"
tags: ["Java", "面试准备"]
---

> **版本说明：** 本篇概括常见网络协议机制；具体浏览器、操作系统与协议版本可能改变实际处理过程。

[← 返回 Java 后端题目总目录](../java-backend-interview-94/)
<span id="9-网络与身份认证"></span>

## 网络与身份认证

<span id="83-https-怎样保证传输安全"></span>

### 1 HTTPS 怎样保证传输安全

<span class="java-label java-insight">理解与追问：</span> 客户端需要验证证书链和域名，并在握手中验证对端对密钥的控制。TLS 保护传输中的数据，但无法阻止已授权用户滥用接口或服务端自身的数据泄露。

HTTPS 是 HTTP 通过 TLS 传输。TLS 通过证书验证对端身份,通过握手协商会话密钥,再用对称认证加密保护应用数据的机密性和完整性。

- 证书把主体身份与公钥绑定,CA 对证书内容签名。客户端验证证书链、域名、有效期等,并验证服务端确实持有相应私钥。签名验证应表述为验签,不能泛化为用公钥解密。

- 现代 TLS 常使用临时 ECDHE 等密钥协商,双方推导密钥;TLS 1.3 不再采用旧式 RSA 密钥传输。应用数据通常使用 AES-GCM 或 ChaCha20-Poly1305 等算法。

<span class="java-label java-warning">易错提醒：</span> 对称密钥不必明文传输;公私钥不能笼统说成可以互相加解密;HTTPS 不保证业务端点自身安全。

<span class="java-label java-example">具体场景：</span> 访问银行网站时，浏览器校验证书链、有效期和域名，再通过 TLS 握手建立会话密钥。若客户端关闭证书校验，传输加密本身也不能可靠证明对端身份。

**追问：HTTPS 能防止所有中间人攻击吗？** 前提是客户端正确校验证书链、域名与握手信息。TLS 保护传输，并不能修复服务端越权、客户端泄漏或业务逻辑问题。

<span id="84-http-各版本的主要变化"></span>

### 2 HTTP 各版本的主要变化

<span class="java-label java-insight">理解与追问：</span> HTTP/2 在同一 TCP 连接内复用多个流，但 TCP 丢包仍可能影响该连接上的流；HTTP/3 使用 QUIC，可以降低不同流之间的传输层阻塞。两者仍共享网络带宽与拥塞控制。

主要演进方向是连接复用、并发传输和减少队头阻塞。HTTP 版本本身不等于是否加密,实际传输还需看 TLS 和协议使用方式。

| 版本 | 关键特征 | 复习重点 |
| --- | --- | --- |
| HTTP 1.0 | 文本报文，通常每次请求新建连接 | 可有持久连接扩展，不宜绝对化 |
| HTTP 1.1 | 默认持久连接，支持管线化 | 管线化响应需按序返回，存在 HTTP 层队头阻塞 |
| HTTP 2 | 二进制帧、多流复用、HPACK | 共享 TCP，丢包仍可能阻塞连接上的多个流 |
| HTTP 3 | 基于 QUIC、QPACK，集成 TLS | QUIC 基于 UDP，流间通常不因单流丢包而相互等待 |

- QUIC 提供可靠传输、流控和拥塞控制;连接 ID 支持在条件满足时迁移连接。HTTP 2 曾支持服务端推送,但不能据此推断所有客户端仍支持。

<span class="java-label java-warning">易错提醒：</span> 二进制帧不等于整体数据必然更少;HTTP 3 也没有消除所有应用和拥塞层面的等待。

<span class="java-label java-example">具体场景：</span> HTTP/2 可在一条 TCP 连接复用多个流，但底层丢包仍可能影响该连接的数据交付；HTTP/3 通过 QUIC 提供独立流，降低流之间这类阻塞，但它们仍共享网络带宽。

**追问：HTTP/2 多路复用后为什么仍可能等待？** 多个流共享 TCP 连接时，TCP 丢包会影响字节流交付；HTTP/3 的 QUIC 可以降低不同流之间这类传输层阻塞，但仍共享网络容量。

<span id="85-tcp-为什么三次握手和通常四次挥手"></span>

### 3 TCP 为什么三次握手和通常四次挥手

<span class="java-label java-insight">理解与追问：</span> 可以分别追问：握手为何确认双方序列号、半关闭是什么意思、主动关闭者为何常进入 TIME_WAIT。具体报文可能合并，因此四次挥手是常见过程而非固定抓包数量。

三次握手让双方交换并确认初始序列号,建立双向连接状态,并减少旧连接报文造成的误建立。关闭时两个方向分别结束,因此通常需要四个报文段。

- 握手:SYN seq=x → SYN ACK seq=y ack=x+1 → ACK ack=y+1。SYN 会消耗一个序列号。

- 关闭:主动方 FIN → 对方 ACK → 对方准备完毕后 FIN → 主动方 ACK。FIN 表示该方向不再发送数据,并不代表另一方向立即关闭。

- ACK 与 FIN 有时可以合并,因此不是每次抓包都恰好四个报文。主动关闭方通常进入 TIME_WAIT,便于重传最终 ACK,并等待旧报文失效。

<span class="java-label java-warning">易错提醒：</span> 挥手次数由双向关闭和确认决定;不要只用验证收发能力解释全部握手设计。

<span class="java-label java-example">具体场景：</span> 客户端发 SYN，服务端回应 SYN+ACK，客户端再 ACK，双方建立连接状态；关闭时两个方向分别结束，因此常见四段交互，某些确认和 FIN 也可能合并。

**追问：四次挥手一定对应四个独立包吗？** 不一定，确认和结束报文在某些条件下可以合并。TCP 支持两个方向分别关闭，TIME_WAIT 也要结合主动关闭与具体状态分析。

<span id="86-cookie-session-token-与-jwt"></span>

### 4 Cookie Session Token 与 JWT

<span class="java-label java-insight">理解与追问：</span> 先分清载体与状态：Cookie 是浏览器发送数据的机制，Session 是服务端会话状态，JWT 是一种令牌格式。把 JWT 放进 Cookie 时，仍需按 Cookie 的发送行为评估 CSRF 风险。

先区分三个概念：

- **Cookie**：浏览器存储和发送数据的机制。
- **Session**：服务端保存会话状态的方案。
- **Token**：访问凭证。

它们并不互斥：同一系统可以把会话 ID 或 Token 放入 Cookie。

- Session 常由随机会话 ID 关联服务端状态。多实例部署要设计共享会话或路由策略;Cookie 是否跨站发送受 Domain、Path、SameSite、Secure 等规则约束。

- 常见签名 JWT 由 Header、Payload、Signature 三部分组成。前两部分是 Base64URL 编码,可直接解码,不应放密码和敏感秘密。签名可使用 HMAC 或非对称算法,并不总是私钥加密。

- 服务端要验证签名、允许的算法、有效期,以及适用的 issuer 和 audience。签名 JWT 不自带即时撤销能力,可用短有效期及服务端状态设计退出和封禁。

- HttpOnly 可降低脚本直接读取 Cookie 的风险;Cookie 自动携带的认证需考虑 CSRF,浏览器可读存储中的 Token 则需考虑被 XSS 窃取。

<span class="java-label java-warning">易错提醒：</span> 常见的签名 JWT 不等于加密，Base64URL 不是加密，Token 也不一定采用 JWT。JWT 也可以按规范加密；不要接受不符合服务端安全策略的未签名令牌。

<span class="java-label java-example">具体场景：</span> 网站把会话 ID 存在 Cookie，浏览器按规则自动带上；服务端再用 ID 找到 Session。若使用签名 JWT，Payload 仍可被解码，因此不能把密码等秘密直接放进去。

**追问：JWT 的内容别人看不到吗？** 普通签名 JWT 的载荷只是编码，可被解码读取；签名主要用于完整性和身份验证，不等于加密。还要验证算法、有效期及适用的签发者和受众。

<span id="87-新增-tcp-如何保证可靠传输-流量控制与拥塞控制有何区别"></span>

### 5 TCP 如何保证可靠传输 流量控制与拥塞控制有何区别

<span class="java-label java-insight">理解与追问：</span> 流量控制主要看接收端处理能力，拥塞控制看网络路径承载能力。发送方可发送的数据同时受相关窗口限制；TCP 的有序字节流也不替应用定义业务消息边界。

TCP 在连接内提供有序字节流,通过序列号、确认、重传和校验等机制应对丢包、重复与乱序。流量控制避免发送方压垮接收端,拥塞控制避免网络路径持续过载,两者共同限制发送速率。

- 接收窗口 rwnd 反映接收端可接收能力;拥塞窗口 cwnd 由发送端根据网络反馈调整,在途数据受两者共同约束。传统拥塞控制包括慢启动、拥塞避免、快速重传等,具体算法依系统实现。

- TCP 没有业务消息边界,一次 write 不对应一次 read;应用协议需用长度字段、分隔符等方式分帧。TCP ACK 只确认传输层接收,不代表订单等业务已经处理成功。

<span class="java-label java-warning">易错提醒：</span> 重传不是重新执行业务的许可;连接中断后重试请求,仍需防止服务端已执行而客户端未收到结果。

<span class="java-label java-example">具体场景：</span> 客户端发出支付请求后连接断开，服务端可能已处理但响应丢失。TCP 确认数据传输不代表业务完成；客户端重试仍需要支付请求号、幂等与结果查询。

**追问：TCP 已可靠传输，为什么业务还需幂等？** TCP 确认传输层数据接收，不代表业务事务已成功。响应丢失或连接中断后重试仍可能重复执行服务端操作。

<span id="88-新增-输入网址到得到响应经历什么-怎样定位慢请求"></span>

### 6 输入网址到得到响应经历什么 怎样定位慢请求

<span class="java-label java-insight">理解与追问：</span> 排查慢请求时先拆 DNS、建连、TLS、首字节和响应下载，再对照服务端链路追踪。连接复用和缓存命中会改变实际路径，不是每次请求都执行完整解析和握手。

以首次访问 HTTPS 服务为例:解析 URL 与缓存策略,通过 DNS 获取目标地址,建立传输连接与TLS 会话,发送 HTTP 请求,经过代理或网关到服务端处理,再接收响应。浏览器随后按资源与页面内容继续加载和渲染。

- DNS 可能命中浏览器、系统或递归解析器缓存;未命中时解析器按域名层级查询。已有连接可复用,HTTP 3 使用 QUIC,实际流程不会每次重复完整 TCP 与 TLS 建连。

- 排查时拆分 DNS、连接、TLS、首字节和下载时间,并结合服务端排队、应用处理、数据库与下游调用耗时。首字节慢可能包含网络与服务端多段等待,不能只看总时长下结论。

<span class="java-label java-warning">易错提醒：</span> 浏览器收到响应不等于页面已经渲染完成;接口总耗时也不等于业务代码的执行耗时。

<span class="java-label java-example">具体场景：</span> 用户打开页面很慢，可先分解 DNS、连接、TLS、首字节和下载，再看网关、应用及数据库耗时。若连接复用，本次请求可能没有完整握手过程，不能把每次访问画成固定链路。

**追问：首字节时间长就一定是服务端慢吗？** 不一定，它可能包含网络和多个服务端阶段。拆分 DNS、建连、TLS、服务端排队与应用执行，再用链路追踪交叉定位。

<span id="89-新增-bio-nio-aio-与多路复用怎样区分"></span>

### 7 BIO NIO AIO 与多路复用怎样区分

<span class="java-label java-insight">理解与追问：</span> 阻塞与非阻塞描述调用等待行为，多路复用负责等待多个连接的就绪事件，异步强调完成后通知。就绪不等于一次读写已完成，应用仍需处理部分数据和协议拆包。

可以从调用者如何等待和获取结果来区分：

- **阻塞 I/O**：调用线程通常等待数据或操作完成。
- **非阻塞 I/O**：暂时无法进行时及时返回。
- **I/O 多路复用**：一个线程等待多个连接的就绪事件，再按就绪情况处理。
- **异步 I/O**：由系统或实现负责发起的操作，并在完成后通知调用者。

- Java NIO 常用 Channel、Buffer、Selector。Selector 报告就绪不等于已替应用完成所有读写,仍需处理部分读取、部分写入和协议分帧;NIO 中的通道也并非全部只能非阻塞。

- Linux 常用 epoll 管理关注集合及就绪事件,避免应用反复检查全部空闲连接;Reactor 线程负责事件分发时,耗时业务应合理交给工作线程,防止阻塞其他连接。

<span class="java-label java-warning">易错提醒：</span> NIO 不是异步 I/O 的同义词,epoll 也不能保证所有负载下都更快;事件线程同样可能被业务阻塞。

<span class="java-label java-example">具体场景：</span> 一次 TCP read 只拿到消息前半段时，应用需暂存数据，等待其余部分，再按长度或分隔符组装消息。Selector 提示可读不等于已经获得完整业务请求。

**追问：Selector 报告可读是否代表已经拿到完整消息？** 不是。就绪表示可以尝试相关 I/O，实际仍可能只读到部分字节；协议需要长度或分隔符等分帧机制。

## 参考资料

- [RFC 9293：TCP](https://www.rfc-editor.org/rfc/rfc9293.html)
- [RFC 7519：JWT](https://datatracker.ietf.org/doc/html/rfc7519.html)

- [Java 8 ThreadPoolExecutor 文档](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html)
- [MySQL 8.0 InnoDB 隔离级别](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)与 [锁机制](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
- [Redis 内存淘汰策略](https://redis.io/docs/latest/develop/reference/eviction/)
- [Spring 事务注解与代理](https://docs.spring.io/spring/reference/6.2/data-access/transaction/declarative/annotations.html)
