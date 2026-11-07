![](http://www.itcast.cn/2018czgw/images/logo.png)
![](http://www.itcast.cn/2020gw/images/slogan2.jpg)

集信达-短信平台
=========================

- [后端Gitee地址](https://gitee.com/itcastopen/itcast-sms.git)
- [后端Github地址](https://github.com/itcastopen/itcast-sms.git)
- [前端Gitee地址](https://gitee.com/itcastopen/itcast-sms-web.git)
- [前端Github地址](https://github.com/itcastopen/itcast-sms-web.git)


**在线演示**
=========================
- [官方主页](https://pip.itcast.cn/jxd)
- [演示地址](http://jxd.itheima.net)


**项目背景**
=========================

- 现在各种账户越来越多越来越重要，短信服务能够避免商家被恶意攻击，避免个人账户被盗。当账户出现异常和高风险操作时，系统自动触发验证码到用户手机上，确认是否是用户本人操作。短信本身被破解、被劫持的难度就比较高，正规的短信平台也会通过多种手段来保障短信安全,收到验证码输入几个数字即可，因此短信验证码是非常方便的一种验证方式。 对于企业商户来说，通过与正规的短信群发平台合作，低成本、低门槛、不需要太多人力财力投入，直接把短信API接入自己的网站/app里就可以了。

- 随着企业对短信规模化使用，传统短信平台的接入方式和单一的信息发送功能，已不能满足现代企业管理的需求，所以统一入口、减少对接成本、简单易行的操作与维护、高稳定、高可靠的移动信息化应用成为短信平台发展趋势。

- [了解更多项目](https://project-dev.itheima.net/java)，改变命运，少走弯路，进入真项目知识海洋畅游。




功能特性
=========================
![](docs/img/功能特性.png)


功能结构图
=========================
![](docs/img/功能结构图.png)
![](docs/img/功能结构图-发送服务.png)
![](docs/img/功能结构图-接收服务.png)
![](docs/img/功能结构图-管理服务.png)

技术架构
=========================
![](docs/img/技术架构图.png)

工程结构
=========================
``` 
itcast-sms
  ├
  ├── docs -- 相关image、nacos、mysql
  ├
  ├── itcast-sms-api -- 接收端
  ├
  ├── itcast-sms-entity -- 实体类
  ├
  ├── itcast-sms-manage -- 管理端
  ├
  ├── itcast-sms-sdk -- 对外开放sdk
  ├
  └── itcast-sms-server-- 服务端 
```

环境要求
=========================
- JDK ： 1.8 +
- Maven： 3.3 +
  http://maven.apache.org/download.cgi
- Mysql： 5.6.0 +
  https://downloads.mysql.com/archives/community
- Redis： 4.0 +
  https://redis.io/download
- Nacos： 1.1.4
  https://github.com/alibaba/nacos/releases


安装运行步骤
=========================
- [安装部署手册（建议下载后查看，避免格式问题）](docs/安装手册/install.md)

页面一览
=========================
![](docs/img/页面/首页.png)
![](docs/img/页面/通道管理.png)
![](docs/img/页面/应用管理.png)
![](docs/img/页面/发送记录.png)
![](docs/img/页面/黑名单.png)

新的惊喜
=========================
想进步、要关注。关注老铁抖音号，成为'铁丝' ：）

<img src="docs/img/douyin.jpeg" width="300"  alt="抖音号" />
<br><br>
加微信、加入Java开源技术交流群
<br>
<img src="docs/img/weixin.jpeg" width="300"  alt="微信交流群" />
<br><br>

更多福利
=========================

[了解更多项目](https://pip.itcast.cn/home)

---

# 项目深度分析（补充章节）

> **说明**：本章节在**完整保留以上原有 README 内容**的前提下追加，原文未删除、未改写。
> **分析依据**：对仓库当前源码（`pom.xml`、3 份 `bootstrap.yml`、Mapper XML、`docs/` 部署资料、204 个 Java 源文件）的静态审阅，分析时间 2026-11-07。
> **阅读约定**：文中路径均相对仓库根目录；行号仅用于快速定位，随代码改动可能漂移。凡未经运行时验证的推断，均在对应位置明确标注。

## 0. 结论速览

**集信达（itcast-sms）** 是一套面向企业多业务系统接入的**短信中台**：把"接入鉴权、签名/模板、通道调度、供应商适配、发送日志、运营统计"收敛为统一入口，业务系统只需通过 SDK 或 HTTP 提交短信，无需逐一对接各家短信服务商。

工程形态是 **Spring Cloud 多模块 Maven 项目**，但运行期并**不依赖服务间 RPC**：三个 Spring Boot 应用通过**共享 MySQL + Redis List 队列 + Redis Pub/Sub 广播**解耦协作。

| 维度 | 现状 |
| --- | --- |
| Maven 描述文件 | 6 个（1 个聚合 POM + 5 个模块 POM） |
| 聚合模块（reactor） | 4 个：`entity` / `server` / `api` / `manage`（`sdk` 不在其中，见 §2.2） |
| 可启动应用 | 3 个：`SmsApiApplication`、`SmsServerApplication`、`SmsManageApplication` |
| Java 源文件 | 204 个：`entity` 60、`api` 31、`server` 56、`manage` 50、`sdk` 7 |
| 领域模型 | 16 个 `@TableName` 实体 / 16 个 Mapper 接口 / 16 份 Mapper XML / 16 张表 |
| REST 控制器 | 13 个（api 1、server 1、manage 11）；类级路径 13 个 |
| 短信通道适配 | 6 个具体供应商实现 + 1 个抽象基类 |
| 单元测试 | 0 个（三个应用仅声明 `spring-boot-starter-test` 依赖，无 `*Test.java`） |
| 配置方式 | 仓库内**只有 `bootstrap.yml`，没有 `application.yml`**；数据源、Redis、端口、口令全部由 Nacos 下发 |
| 消息队列 | 无 Kafka / RabbitMQ / RocketMQ；用 Redis List 做队列、Redis Pub/Sub 做广播 |
| 定时任务 | Spring `@Scheduled`（非 XXL-Job / Quartz） |
| 事务 / ORM | MyBatis-Plus 3.2.0 + Druid + MySQL 8，三个应用各自直连同一个库 |
| 许可证 | Apache License 2.0（仓库根目录 `LICENSE`） |

**一条短信的旅程（核心链路）**：

```
业务系统                          接入层                         队列                     发送层                    供应商
   │                                │                            │                         │                        │
   ├─ SDK / HTTP POST /sms/send ────▶ itcast-sms-api              │                         │                        │
   ├─ HTTP POST /sms/batchSend ─────▶  ① 平台注册与 AK/SK 签名鉴权 │                         │                        │
   └─ Netty TCP :10000 (JSON) ─────▶  ② 黑名单 / 敏感词            │                         │                        │
                                      ③ 签名 + 模板 → 可用通道 ID  │                         │                        │
                                      ④ 模板参数渲染校验            │                         │                        │
                                      ⑤ 写 receive_log 接收日志    │                         │                        │
                                      ⑥ 分类并按优先级分发 ─────────▶ TOPIC_HIGH_SMS ───────▶ HighSmsListener ───┐  │
                                      定时短信 → timing_push 表     └─ TOPIC_GENERAL_SMS ──▶ GeneralSmsListener ─┤  │
                                                                                                                ▼  │
                                                                         SmsFactory 按 level 逐级选通道            │
                                                                         校验通道是否支持该签名+模板               │
                                                                         AbstractSmsService 具体供应商实现 ────────▶ 阿里云/梦网/飞鸽/京东云/华为/乐信
                                                                         保存 send_log（成功/失败/耗时）            │
                                                                                                                │
管理后台 itcast-sms-manage ── 共用 entity Mapper 直接读写同库 ── 通道/模板/签名/黑名单/营销/日志/统计/Excel 导入导出
```

**关键澄清**：项目**没有** Dubbo、`@FeignClient` 业务接口、Kafka、RabbitMQ、MongoDB、Elasticsearch、XXL-Job，也没有独立的计费与限流模块；`docs/dockerfile/sentinel/` 是 **Redis Sentinel 哨兵高可用**，不是 Alibaba Sentinel 流控组件。"送达状态"目前只有供应商**同步返回值**判定，没有异步状态回执回调接口（见 §9）。

---

## 1. 业务能力地图

| 能力域 | 具体实现 | 主要代码位置 |
| --- | --- | --- |
| 开放接入 | SDK（`SmsSendService`）+ HTTP `POST /sms/send`、`/sms/batchSend` + Netty TCP `:10000` | `itcast-sms-sdk/.../service/impl/SmsSendServiceImpl.java`、`itcast-sms-api/.../controller/SmsSendController.java`、`itcast-sms-api/.../netty/` |
| 接入鉴权 | `accessKeyId` 注册校验 + `timestamp + AK + SK` 双次 MD5 签名，按 `platform.need_auth` 开关 | `itcast-sms-api/.../service/impl/SmsSendServiceImpl.java:78-103,209,229`、`utils/SmsEncryptionUtils.java` |
| 内容风控 | 手机号正则 `^[1]\d{10}$`、黑名单、敏感词 | 同上 `:111-136`；`BlackListServiceImpl`、`SensitiveListServiceImpl`（Redis 缓存 60s） |
| 签名 / 模板 | 签名表、模板表、`config_signature` / `config_template` 关联；`${}` 占位符渲染 | `entity/.../mapper/ConfigMapper.xml` 的 `findByTemplateSignature`；`StringHelper.renderString` |
| 通道管理 | 通道配置（`level` 优先级、启停、`channel_type` 1 文字 / 2 语音 / 3 推送）、按级别降级、失败计数、热切换与选举 | `itcast-sms-server/.../factory/SmsFactory.java`、`SmsConnectLoader.java`、`config/RedisLock.java`、`SubscriberConfig.java` |
| 供应商适配 | 6 家 HTTP(S)/厂商 SDK 实现，反射按 `platform` 值实例化 | `itcast-sms-server/.../sms/` |
| 优先级队列 | 验证码进 `TOPIC_HIGH_SMS`，营销/通知进 `TOPIC_GENERAL_SMS`；定时短信入 `timing_push` 表由定时任务补投 | `itcast-sms-api/.../SmsSendServiceImpl.java:312-360`；`itcast-sms-server/.../redismq/`、`job/SendSmsJob.java` |
| 日志与统计 | `receive_log`（接入日志）、`send_log`（发送日志、耗时、失败原因）、`manual_process`（人工处理）、送达率/Top10/趋势统计 | `entity/.../mapper/SendLogMapper.xml`、`ReceiveLogMapper.xml`；`manage/.../controller/StatisticsController.java` |
| 运营后台 | 通道、应用（接入平台）、签名、模板、黑名单、敏感词、通讯组/通讯录、营销（审核/重发/Excel 导入）、日志、统计 | `itcast-sms-manage/.../controller/` 共 11 个 Controller |
| 高可用辅助 | Redis 分布式锁（`SET NX PX` + Lua 解锁）、服务实例注册上报与失联清理、通道变更广播 | `server/.../config/RedisLock.java`、`factory/ServerRegister.java`、`HighServerReceiver.java` |

---

## 2. 工程结构与构建

### 2.1 目录全景（在原文"工程结构"基础上的细化）

```
itcast-sms
├── pom.xml                         # 聚合 POM：parent、4 个 modules、3 套 profiles、dependencyManagement
├── README.md / LICENSE             # 说明文档 / Apache-2.0
├── docs/
│   ├── mysql/itcast_sms.sql        # 16 张表的建表语句 + 演示数据（含演示密钥，见 §9）
│   ├── nacos/nacos_config_export_.zip  # Nacos 配置导出包（common.yml / redis.yml / mysql.yml 的来源）
│   ├── dockerfile/redis/           # docker-compose：Redis 1 主 2 从
│   ├── dockerfile/sentinel/        # docker-compose + 3 份 sentinel.conf：3 哨兵
│   ├── 安装手册/install.md         # 环境安装、导数据、编译、启动全流程（含 26 张步骤截图）
│   └── img/                        # 功能/架构图、后台页面截图、二维码
├── itcast-sms-entity/              # 共享领域层（jar，无启动类）
│   ├── src/main/java/com/itheima/sms/{entity,mapper,dto,vo,enumeration,exception,model}
│   └── src/main/resources/mapper/  # 16 份 Mapper XML（pom 特意只把 **/*.xml 打进 jar）
├── itcast-sms-api/                 # 接收/接入服务（Spring Boot 应用）
│   └── src/main/java/com/itheima/sms/{controller,service,netty,config,utils}
├── itcast-sms-server/              # 发送/通道调度服务（Spring Boot 应用）
│   └── src/main/java/com/itheima/sms/{factory,sms,redismq,job,service,config,properties,controller}
├── itcast-sms-manage/              # 管理后台 REST 服务（Spring Boot 应用 + Dockerfile）
│   └── src/main/java/com/itheima/sms/{controller,service,aspect,excel,utils,config}
└── itcast-sms-sdk/                 # 独立发布的客户端 SDK（普通 jar，无 resources）★★ 不在 reactor 中
    └── src/main/java/com/itheima/sms/{service,dto,utils}
```

### 2.2 模块职责与依赖方向

| 模块 | 坐标 / 打包 | 职责 | 被谁依赖 |
| --- | --- | --- | --- |
| `itcast-sms-entity` | `com.itheima:itcast-sms-entity:1.0.0` / jar | 实体、DTO/VO、枚举、Mapper 接口与 XML、`SmsException`、`ServerTopic` | `api`、`server`、`manage` |
| `itcast-sms-api` | `itcast-sms-api:1.0.0` / 可执行 jar | 开放接入：校验 9 步、写 `receive_log`、按优先级入 Redis 队列；附带 Netty TCP 入口 | 无（运行期由 SDK / 管理后台通过 HTTP 调用） |
| `itcast-sms-server` | `itcast-sms-server:1.0.0` / 可执行 jar | 消费队列、通道选择与降级、供应商适配、写 `send_log`、定时补投、通道热切换 | 无 |
| `itcast-sms-manage` | `itcast-sms-manage:1.0.0` / 可执行 jar | 11 组后台 REST 接口、营销审核与重发、Excel 导入导出、统计 | 无（依赖 `entity` + `sdk`） |
| `itcast-sms-sdk` | `com.itheima:itcast-sms-sdk:1.0.1` / 普通 jar | `sendSms` / `batchSendSms`、统一 `R<T>` 返回、签名工具，Apache HttpClient 裸调用 | `manage`（`MarketingController` 审核后发短信） |

```
编译期：api ─┐
            server ─┼──▶ entity        manage ──▶ entity + sdk(1.0.1)
            manage ─┘
运行期：manage ──(SDK/HTTP)──▶ api ──(Redis List)──▶ server ──(HTTP/厂商 SDK)──▶ 短信服务商
数据面：三个应用各自使用 entity 的 Mapper 直连同一个 MySQL，共享 16 张表
```

**容易踩坑的事实**：根 `pom.xml:20-25` 的 `<modules>` **只有 4 个**，`itcast-sms-sdk` 有独立的 `spring-boot-starter-parent` 与版本号 `1.0.1`（`itcast-sms-sdk/pom.xml:6-25`，并配置 `distributionManagement` 发布到 `http://repo.itheima.net/repository/maven-releases/`）。而 `itcast-sms-manage/pom.xml:18-22` 以**外部制品**方式依赖它。因此全新环境下直接执行根 `mvn clean install` 会因找不到 `itcast-sms-sdk:1.0.1` 失败，需先在 `itcast-sms-sdk/` 目录单独 `mvn clean install`（或部署到私服）。安装手册 `install.md:134` 只给出一条根构建命令，未提示这一点。

### 2.3 版本清单（取自 `pom.xml` 与各模块 POM）

| 组件 | 版本 | 说明 |
| --- | --- | --- |
| JDK / 编码 | Java 1.8 / UTF-8 | `pom.xml:74-75`，`maven-compiler-plugin 3.8.1` |
| Spring Boot | 2.2.2.RELEASE | 根 POM parent；SDK 也单独继承同一版本 |
| Spring Cloud | Hoxton.SR1 | `spring-cloud-dependencies` BOM |
| Spring Framework | 5.2.2.RELEASE | `spring-framework-bom` BOM |
| Spring Cloud Alibaba | 2.1.1.RELEASE | Nacos Discovery / Config |
| Nacos | 1.1.4（README 环境要求） | SDK 里显式依赖 `nacos-common:1.1.4` |
| MyBatis-Plus | 3.2.0 | 三个应用引入 `mybatis-plus-boot-starter` |
| MySQL / Druid | mysql-connector 8.0.11 / druid 1.2.5 | 数据库脚本来自 MySQL 8.0.20 导出 |
| Web 容器 | Undertow | 三个应用均从 `spring-boot-starter-web` 排除 Tomcat 后引入 `spring-boot-starter-undertow` |
| Redis | `spring-boot-starter-data-redis` + `commons-pool2`（Lettuce） | 队列、缓存、Pub/Sub、分布式锁全靠它 |
| Hystrix / OpenFeign | `spring-cloud-starter-netflix-hystrix` / SDK 引入 openfeign 2.2.2 | 只有 `@EnableHystrix`、`@EnableFeignClients` 注解，**无 `@HystrixCommand` / 无 `@FeignClient` 业务接口** |
| 接口文档 | springfox 2.9.2 + `itcast-tools-swagger2` | 启动后 `/doc.html`，另有 `/druid` 监控 |
| 厂商 SDK | `aliyun-java-sdk-core:4.5.0`、`com.jdcloud.sdk:sms:1.3.2` | 其余供应商走 `HttpPost` 裸 HTTP |
| 工具库 | `itcast-tools-* 2.1.9`（common/user/log/swagger2/validator/xss/databases/dozer/jwt/j2cache/core）、hutool 5.1.0、fastjson 1.2.62、guava 20.0、lombok 1.18.10、easypoi 4.1.3 | `itcast-tools` 来自私服 `http://repo.itheima.net/repository/maven-public/` |
| 其它（仅 BOM 声明，未使用） | minio 8.3.3、qiniu 7.2.18、fastdfs-client 1.27.2、easy-captcha 1.6.2、mybatis 3.5.2、jsr310 typehandlers 1.0.2、`itcast-auth-api:2.1.6` | `pom.xml:86-101,140-144,203-259` |

### 2.4 构建、配置占位符与启动

- **资源过滤**：根 `pom.xml:302-311` 对 `src/main/resources/**/*` 开启 `filtering=true`，因此 `bootstrap.yml` 中的 `@pom.nacos.ip@`、`@pom.profile.name@`、`@project.artifactId@` 与 `banner.txt` 中的 `@project.version@` 会在构建时被替换；`maven-resources-plugin` 同时声明 `pem/pfx/p12/key` 不做过滤（避免证书被破坏）。
- **三套环境 profiles**（`pom.xml:27-69`，均使用 Nacos 8848）：`dev`（默认激活，`localhost:8848`）、`test`（`nacos-test.itcast.cn:8848`）、`prod`（`172.17.2.120:8848`），各带独立 namespace ID。切换方式：`mvn clean install -DskipTests -Ptest`。
- **构建命令**：`mvn clean install -DskipTests`（`install.md:134`）；`finalName=${project.artifactId}`，产物形如 `itcast-sms-manage/target/itcast-sms-manage.jar`，与 `itcast-sms-manage/Dockerfile` 的 `ADD ./target/itcast-sms-manage.jar ./app.jar` 对应。
- **启动顺序**（`install.md:139-151`）：先基础设施（Nacos → MySQL → Redis 主从哨兵）→ 导入 Nacos 配置与 SQL → 修改根 POM namespace → 构建 → 依次启动 `SmsApiApplication`、`SmsManageApplication`、`SmsServerApplication`。**注意**安装手册中的 `SmsManagerApplication` 是笔误，实际类名为 `SmsManageApplication`；`docs/naocs/` 亦是 `docs/nacos/` 的笔误。
- **验证**：三个应用启动时会打印 `http://{ip}:{server.port}/doc.html`（Swagger）与 `/druid`（数据源监控）。应用自己的 HTTP 端口**不在仓库代码里**，由 Nacos 配置下发。

> 本节为静态分析结论，未在本机执行 Maven 构建与三个应用的联调启动。

---

## 3. 核心业务流程详解

### 3.1 接入层：三种入口

| 入口 | 说明 | 代码 |
| --- | --- | --- |
| HTTP（推荐） | `POST /sms/send`（单条，`SmsParamsDTO`）、`POST /sms/batchSend`（批量，`SmsBatchParamsDTO`），`BaseParamsDTO` 提供 `accessKeyId / timestamp / encryption / signature / template / mobile / params / sendTime` 等公共字段 | `itcast-sms-api/.../controller/SmsSendController.java:27-58` |
| SDK | `SmsSendService.sendSms / batchSendSms`，配置键 `itheima.sms.{auth,domain,accessKeyId,accessKeySecret,signature}`，用 `CloseableHttpClient` 拼 `domain + /sms/send` | `itcast-sms-sdk/.../service/impl/SmsSendServiceImpl.java:26-39,66,121` |
| Netty TCP | 默认 `:10000`（`netty.port` 可覆盖），`DelimiterBasedFrameDecoder` + `StringDecoder/StringEncoder` 处理换行分隔的 JSON，收到后转交同一 `SmsSendServiceImpl` | `itcast-sms-api/.../netty/NettyServer.java:35,44-63`、`NettyServerHandler.java:27-38` |

三者最终都汇聚到 `itcast-sms-api` 的 `SmsSendServiceImpl.send / batchSend`，保证校验逻辑只有一份。

### 3.2 接入层「9 步校验」（类注释见 `api/.../service/impl/SmsSendServiceImpl.java:29-40`）

| # | 校验 | 关键逻辑 |
| --- | --- | --- |
| 1 | 注册校验 | 按 `accessKeyId` 查 `platform`，不存在抛"系统未注册"，`is_active=0` 抛"系统不可用"（`:78-87`，平台信息 Redis 缓存 60s） |
| 2 | 签名鉴权 | `need_auth=1` 时，`SmsEncryptionUtils.encode(timestamp, accessKeyId, accessKeySecret)`（双次 MD5）与请求 `encryption` 比对，不符抛"鉴权失败"（`:97-103,209,229`） |
| 3 | 手机号 + 黑名单 | 正则 `^[1]\d{10}$`；命中 `black_list`（type=1）抛"黑名单手机号"（`:111-122`） |
| 4 | 敏感词 | 对所有模板参数逐个检查 `sensitive_list`（`:129-136`） |
| 5 | 签名 | 按 `signature` 编码查 `signature` 表，不存在抛异常 |
| 6 | 模板 → 通道 | 按 `template` 编码查 `template` 表，再经 `ConfigMapper.findByTemplateSignature` 联查 `config / config_template / config_signature`，按 `c.level asc` 返回支持该「签名 + 模板」的可用通道 ID 列表 `configIds`（`:146-165`） |
| 7 | 参数匹配 | `StringHelper.renderString` 渲染 `${}` 占位符，渲染后若仍残留 `${` 判为"参数不匹配"（`:170-178`） |
| 8 | 短信分类 | 验证码（`TemplateType.VERIFICATION`）→ 高优队列；其余（通知/营销）→ 普通队列 |
| 9 | 分发 + 落日志 | 生成 `apiLogId`，`finally` 中写 `receive_log`；实时消息 `leftPush` 到 Redis List，定时消息写 `timing_push` 表（`:312-360`） |

定时发送另有前置校验：`sendTime` 必须晚于「当前时间 + 1 分钟」（格式 `yyyy-MM-dd HH:mm`，`:185-194`）。批量接口按 `mobile / signature / template / params` 四个集合迭代，逐条走同样的校验与分发，错误逐条收集而不是整批失败。

### 3.3 队列与消费

- 队列名：`TOPIC_HIGH_SMS`（验证码）、`TOPIC_GENERAL_SMS`（营销/通知），均为 **Redis List**，`leftPush` 生产、`rightPop(queueKey, ${spring.redis.queue.pop.timeout})` 阻塞消费（默认 8000ms）。
- 消费者：`HighSmsListener` / `GeneralSmsListener`，二者都 `extends Thread`，在 `@PostConstruct` 中直接 `start()`，`while(true)` 循环消费并调用 `smsFactory.send(message)`（`itcast-sms-server/.../redismq/`）。
- 定时短信：`job/SendSmsJob.java` 上 `@Scheduled(cron = "10 0/1 * * * ?")`，每分钟第 10 秒扫描 `timing_push`，用 `RedisLock` 加 50s 锁防止多实例重复投递，`SendTimingSmsImpl` 以 `@Async` 补投队列。

### 3.4 发送层：通道选择、降级与热切换

`itcast-sms-server/.../factory/SmsFactory.java:71-215` 是发送核心，循环逻辑如下：

1. `level` 从 **1** 开始，`SmsConnectLoader.getConnectByLevel(level)` 取 `CONNECT_LIST[level-1]`。
2. `checkConnectLevel(level)` 判定所有通道是否已尝试完 → 写 `manual_process` 人工处理任务 + 一条 `status=0` 的 `send_log`（`configId=404`），返回失败。
3. 判断当前级别通道 ID 是否包含在 `smsSendDTO.configIds` 中（即是否支持本次签名+模板），不支持则 `level++` 继续。
4. 调用 `AbstractSmsService.send(...)` 获得供应商同步响应；`SendLogEntity.checkResponse` 解析 `FAIL@#@` 前缀，命中则抛 `SmsException`。
5. 异常路径：`resetChannel(level)` 用 Redis `config_level_{level}` 计数（10 分钟过期），超过 `configLevelFailNum` 触发通道重排广播；超过 `messageErrorNum` 阈值则 `level++` 换下一级通道；超过 `configBuildScale` 倍阈值则 `buildNewConnect()` 启用备用通道。阈值来自 `properties/SmsProperties.java`（`prefix=itcast.sms`，`@RefreshScope`，可由 Nacos 动态刷新）。
6. `finally` 统一保存 `send_log`（含 `useTime` 耗时、`status`、`response`、`error`、通道快照字段）。

**通道实例化**：`SmsConnectLoader.initConnect()`（`factory/SmsConnectLoader.java:63-117`）是 `@Order(101)` 的 `CommandLineRunner`，读取 `config` 表可用通道（`ConfigServiceImpl.listForConnect`，60s 缓存），把 `domain/name/platform/accessKeyId/accessKeySecret/other(JSON)` 组装成 `SmsConfig`，再 `Class.forName("com.itheima.sms." + platform + "SmsService")` 反射创建实例，并反射注入 `signatureService` / `templateService`。因此**新增供应商的类名必须与数据库中 `platform` 字段严格对应**（`Aliyun` → `AliyunSmsService`），没有编译期约束。

**通道选举与广播**：`ConfigServiceImpl.listForNewConnect()` 依据近 1 小时成功数重新排序，把不可用地后置、原第一通道置 `level=99, is_enable=0`；`buildNewConnect()` 用 `RedisLock`（`SET NX PX` + Lua 原子解锁）保证只有一个实例计算，并通过 `TOPIC_HIGH_SERVER` Pub/Sub 广播 `USE_NEW_CONNECT` / `INIT_CONNECT`（`entity/.../model/ServerTopic.java`），`HighServerReceiver` 收到后各实例重建通道。`ServerRegister` 每 3 分钟上报实例（Redis Hash `SERVER_ID_HASH`），每 10 分钟清理失联 5 分钟的实例。

### 3.5 供应商适配清单

| 实现类 | 平台值（数据库 `config.platform`） | 接入方式 |
| --- | --- | --- |
| `AliyunSmsService` | `Aliyun` | 阿里云 `aliyun-java-sdk-core:4.5.0` |
| `MengWangSmsService` | `MengWang` | 梦网 HTTP API（`api02.monyun.cn:7901`） |
| `FeigeSmsService` | `Feige` | 飞鸽 HTTP 模板接口 |
| `JdSmsService` | `Jd` | 京东云 `com.jdcloud.sdk:sms:1.3.2` |
| `HuaweiSmsService` | `Huawei` | 华为云 `/sms/batchSendSms/v1` HTTPS |
| `LexinSmsService` | `Lexin`（代码存在，SQL 未预置） | HTTP |
| `AbstractSmsService` | — | 抽象基类，统一 `send / sendSms / sendSmsBatch`，失败响应前缀 `FAIL@#@` |

全部位于 `itcast-sms-server/src/main/java/com/itheima/sms/sms/`。协议形态是 **HTTP(S) + 厂商 SDK**，另有一个自研 Netty TCP 接入协议；仓库中**没有** CMPP / SMPP / SGIP / SMGP 等运营商网关协议实现。

### 3.6 运营后台闭环

- **配置类**：通道配置 `/config`（含 `level` 调整、`other` JSON 参数修改）、接入平台 `/platform`、签名 `/signature`、模板 `/template`（含 `paramFields` 占位符解析、`build` 构建）、黑名单 `/blacklist`、敏感词 `/sensitivelist`。
- **营销类**：通讯组 `/mailgroup`、通讯录 `/maillist`、营销活动 `/marketing`。`MarketingController.audit`（`:194-215`）审核通过后组装 `SmsBatchParamsDTO` 调 SDK `batchSendSms`；`reSend`（`:225-250`）依据 `receive_log` 中 `status=0` 的记录重发；Excel 名单导入用 easypoi（`excel/`、`utils/ExcelUtils.java`）。
- **日志/统计**：`/log`（`sendPage`、`receivePage`、`sendLogPage`）、`/statistics`（`count`、`trend`、`countForConfig`、`rateForConfig` 通道送达率、`top10`、`marketingTrend`），复杂 SQL 集中在 `SendLogMapper.xml` 与 `ReceiveLogMapper.xml`。

---

## 4. REST 接口清单

### 4.1 接入服务 `itcast-sms-api`

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| POST | `/sms/send` | 单条短信发送 |
| POST | `/sms/batchSend` | 批量短信发送 |

类级 `@RequestMapping("sms")`；异常 `SmsException` 在 Controller 内被 `try/catch` 转成 `R.fail`，其余异常由全局 `ExceptionConfiguration`（继承 `DefaultGlobalExceptionHandler`）处理。

### 4.2 发送服务 `itcast-sms-server`

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| POST | `/sms/simulation` | **调试用**：随机批量生成 `receive_log` / `send_log` 模拟数据，代码中未见鉴权注解（见 §9） |

### 4.3 管理后台 `itcast-sms-manage`（11 个 Controller）

| Controller | 基础路径 | 主要能力 |
| --- | --- | --- |
| `ConfigController` | `/config` | 分页、详情、新增、级别调整、修改、`other` 参数修改、删除 |
| `BlackListController` | `/blacklist` | CRUD + Excel 导入（`upload`）/ 导出（`export`） |
| `TemplateController` | `/template` | 分页、`customPage`、`list`、详情、`paramFields`、新增、`build`、修改、删除 |
| `SignatureController` | `/signature` | 分页、`customPage`、`list`、详情、新增、修改、删除 |
| `PlatformController` | `/platform` | 接入平台（AK/SK、IP 白名单、是否需要鉴权）CRUD |
| `SensitiveListController` | `/sensitivelist` | 敏感词 CRUD |
| `MailGroupController` | `/mailgroup` | 通讯组 CRUD |
| `MailListController` | `/maillist` | 通讯录 CRUD + `list` |
| `MarketingController` | `/marketing` | 分页、详情、新增、修改、审核（`audit`）、重发（`reSend`）、删除、名单导入 |
| `LogController` | `/log` | `sendPage`、`receivePage`、详情、`sendLogPage` |
| `StatisticsController` | `/statistics` | 概览计数、趋势、按通道统计、送达率、Top10、营销趋势 |

通用端点：`/doc.html`（Swagger UI）、`/druid`（Druid 监控）。所有路径均来自类级 `@RequestMapping`，未包含 `server.servlet.context-path`（若 Nacos 下发上下文路径则需叠加）。

---

## 5. 数据模型

`docs/mysql/itcast_sms.sql` 是 MySQL 8.0.20 导出的初始化脚本（InnoDB、`utf8mb4`、逻辑删除 `is_delete`、通用审计字段），包含 16 张表的 `DROP + CREATE + INSERT` 演示数据；脚本内**不含建库语句**，需先手工创建数据库（`install.md:1.2.2`）。16 个实体与 16 份 Mapper XML 一一对应，位于 `itcast-sms-entity/`。

| 分类 | 表 / 实体 | 用途 |
| --- | --- | --- |
| 接入与风控 | `platform` / `PlatformEntity` | 接入平台、AK/SK、IP 白名单、`need_auth`、`is_active` |
| 接入与风控 | `black_list` / `BlackListEntity` | 黑名单手机号（type 区分短信等） |
| 接入与风控 | `sensitive_list` / `SensitiveListEntity` | 敏感词 |
| 内容配置 | `signature` / `SignatureEntity` | 短信签名（编码如 `DXQM_0000000xx`） |
| 内容配置 | `template` / `TemplateEntity` | 短信模板（`${}` 占位符、类型：验证码/通知/营销） |
| 通道配置 | `config` / `ConfigEntity` | 通道：平台、域名、AK/SK、`other`(JSON)、`level`、`is_active`、`is_enable`、`channel_type` |
| 通道配置 | `config_signature` / `ConfigSignatureEntity` | 通道—签名关联 |
| 通道配置 | `config_template` / `ConfigTemplateEntity` | 通道—模板关联（含供应商侧模板 ID） |
| 营销 | `marketing` / `MarketingEntity` | 营销活动（审核状态、目标名单、发送时间） |
| 营销 | `mail_group` / `MailGroupEntity` | 通讯组 |
| 营销 | `mail_list` / `MailListEntity` | 通讯录 |
| 营销 | `mail_list_group` / `MailListGroupEntity` | 通讯录—通讯组关联 |
| 运行数据 | `receive_log` / `ReceiveLogEntity` | 接入日志（请求、状态、`apiLogId`、平台） |
| 运行数据 | `send_log` / `SendLogEntity` | 发送日志（通道快照、响应、失败原因、耗时、状态 0/1） |
| 运行数据 | `timing_push` / `TimingPushEntity` | 定时待发队列（由 `SendSmsJob` 扫描补投） |
| 运行数据 | `manual_process` / `ManualProcessEntity` | 通道全部失败后的人工处理任务 |

`BaseEntity`（`entity/.../entity/base/BaseEntity.java`）统一提供主键、`createUser/createTime/updateUser/updateTime`、`isDelete`；`DefaultParamsAspect` 在保存/修改前通过 AOP 反射自动填充这些审计字段（配合 `@DefaultParams` 注解，约 23 处使用）。

演示数据要点：`config` 预置 5 条通道（阿里云、梦网、飞鸽、京东云、华为）、`platform` 预置 13 个业务系统、`signature` 13 条、`template` 9 条、`black_list` 11 个手机号，并包含**演示用的明文 AccessKey/Secret**——真实部署前必须替换（见 §9）。

---

## 6. 配置、部署与基础设施

### 6.1 配置从哪来

仓库里**没有 `application.yml`**；三个应用各有一份内容一致的 `src/main/resources/bootstrap.yml`，它只负责"引导"，真实配置在 Nacos：

| 配置段 | 值 / 作用 |
| --- | --- |
| `itcast.nacos.{ip,port,namespace}` | `${NACOS_IP:@pom.nacos.ip@}` 等：优先环境变量，否则用构建期由 Maven profile 注入的值 |
| `itcast.local-ip` | `${LOCAL_IP:${spring.cloud.client.ip-address}}`，Docker 部署时需显式指定宿主机 IP |
| `spring.application.name` | `@project.artifactId@` → `itcast-sms-api` / `itcast-sms-server` / `itcast-sms-manage`，也是 Nacos 服务名 |
| `spring.profiles.active` | `@pom.profile.name@` → dev / test / prod |
| `spring.cloud.nacos.config` | `server-addr`、`file-extension: yml`、`namespace`、`shared-dataids: common.yml,redis.yml,mysql.yml`、`refreshable-dataids: common.yml` |
| `spring.cloud.nacos.discovery` | 同地址同 namespace 做服务注册发现 |
| `spring.main.allow-bean-definition-overriding` | `true`（允许 `itcast-tools` 的自动配置覆盖） |
| `logging.file.{path,name}` | `/data/projects/logs`、`${spring.application.name}/root.log` |

也就是说：**MySQL 连接串、Redis 地址与口令、服务 HTTP 端口、日志级别**都在 `docs/nacos/nacos_config_export_.zip` 导出的 `common.yml / redis.yml / mysql.yml` 里，需按 `install.md:1.2.1` 在 Nacos 控制台创建命名空间并导入。

代码内可覆盖的配置键（全仓 `@Value` / `@ConfigurationProperties` 扫描）：

| 键 | 默认值 | 使用处 |
| --- | --- | --- |
| `netty.port` | `10000` | `api/.../netty/NettyServer.java:35` |
| `spring.redis.queue.pop.timeout` | 无默认（Nacos 下发，通常 8000ms） | `server/.../redismq/HighSmsListener.java:29`、`GeneralSmsListener.java:29` |
| `itcast.sms.messageErrorNum` / `configLevelFailNum` / `configBuildScale` | 由 Nacos 下发 | `server/.../properties/SmsProperties.java`（`@RefreshScope`） |
| `itheima.sms.{auth,domain,accessKeyId,accessKeySecret,signature}` | `signature` 默认空串 | `sdk/.../SmsSendServiceImpl.java:26-34`（调用方配置） |
| `spring.datasource.druid.*` | 由 Nacos 下发 | 3 份 `config/datasource/SmsXxxDatabaseAutoConfiguration.java` |

### 6.2 端口一览（区分代码内 / 部署文档 / 外部服务）

| 端口 | 归属 | 来源 |
| --- | --- | --- |
| 8848 | Nacos（dev/test/prod 三套地址相同端口） | `pom.xml:40,52,64`；`install.md:11,16` |
| 3306 | MySQL（Docker 映射） | `install.md:24,29` |
| 16380 / 16381 / 16382 | Redis 一主两从 | `docs/dockerfile/redis/docker-compose.yml` |
| 26380 / 26381 / 26382 | 3 个 Redis Sentinel | `docs/dockerfile/sentinel/docker-compose.yml`、`sentinel{1,2,3}.conf` |
| 10000 | Netty TCP 接入（代码默认） | `NettyServer.java:35` |
| 8770 | manage 容器 `EXPOSE`（示例） | `itcast-sms-manage/Dockerfile` |
| 应用 HTTP 端口 | **不在仓库内**，由 Nacos `server.port` 下发 | 启动类读取 `env.getProperty("server.port")` 打印 Swagger 地址 |
| 7901 / 10743 / 9960 | 外部服务商/控制台端口（梦网、华为） | `docs/mysql/itcast_sms.sql`、`MengWangSmsService.java` |

### 6.3 部署资料

- `docs/dockerfile/redis/docker-compose.yml`：Redis 1 主 2 从（`--port 16380/16381/16382`，`--slaveof`）。
- `docs/dockerfile/sentinel/docker-compose.yml` + `sentinel1~3.conf`：3 哨兵监控 `mymaster`，`quorum=2`。这两处是 **Redis Sentinel**，与 Alibaba Sentinel 流控无关。
- `itcast-sms-manage/Dockerfile`：基础镜像 `anapsix/alpine-java:8_server-jre_unlimited`，`EXPOSE 8770`，`ADD ./target/itcast-sms-manage.jar ./app.jar`，JVM `-Xms256m -Xmx256m`。其余两个应用没有 Dockerfile。
- 3 份 `logback-spring.xml`：引入 `itcast-tools-log` 的 `itcast-defaults.xml`，按 `springProfile` 区分 dev（同步 appender）与 test/docker/prod（异步 appender）。
- 3 份 `spy.properties`：P6Spy SQL 监控，`com.mysql.cj.jdbc.Driver`，慢 SQL 阈值 2 秒，仅 dev 环境启用 `P6DataSource`。
- 3 份 `banner.txt`：ASCII 字符画 + 由资源过滤填入的应用版本与 Spring Boot 版本；`dozer/sms.dozer.xml`（api/server）目前是空 `<mappings/>` 占位。

---

## 7. 横切机制与基础设施代码

| 机制 | 实现位置 | 说明 |
| --- | --- | --- |
| 全局异常 | 3 份 `config/ExceptionConfiguration.java`，`@ControllerAdvice` + 继承 `DefaultGlobalExceptionHandler`；业务异常 `entity/.../exception/SmsException.java` | api Controller 另把 `SmsException` 就地转成 `R.fail` |
| 鉴权 | `api/.../utils/SmsEncryptionUtils.java`（api / server / sdk 共 3 份拷贝）；后台用户上下文 `BaseContextHandler.getUserId()` | 平台级 AK/SK + 时间戳签名；用户级上下文来自 `itcast-tools-user` |
| 参数填充 AOP | `annotation/DefaultParams.java` + `aspect/DefaultParamsAspect.java` | `@Before` 反射注入 createUser/createTime/updateUser/updateTime |
| 事务 | 3 个启动类 `@EnableTransactionManagement` + `SmsXxxDatabaseAutoConfiguration`（`TransactionInterceptor`） | 由 `itcast-tools-databases` 提供基础自动配置 |
| MyBatis | 3 份 `SmsXxxMybatisAutoConfiguration` 继承 `BaseMybatisConfiguration` | 注册 MyBatis-Plus 分页等拦截器 |
| 缓存 | `@EnableCaching` 的 `config/LettuceConfig.java`；业务侧手写 60s Redis 缓存 | `PlatformServiceImpl`、`BlackListServiceImpl`、`TemplateServiceImpl`、`ConfigServiceImpl` |
| 分布式锁 | `server/.../config/RedisLock.java` | `SET NX PX` 加锁 + Lua 原子解锁，用于通道重建与定时补投 |
| 定时任务 | `SmsServerApplication` 上 `@EnableScheduling`；`SendSmsJob`（每分钟第 10 秒）、`ServerRegister`（3 分钟上报 / 10 分钟清理） | 无 XXL-Job / Quartz |
| 异步 | `@EnableAsync` + `SendTimingSmsImpl` 上 `@Async` | **全仓没有自定义 `ThreadPoolTaskExecutor`**，使用 Spring 默认执行器 |
| 消息广播 | `config/SubscriberConfig.java`（`RedisMessageListenerContainer` + `PatternTopic("TOPIC_HIGH_SERVER")`）→ `HighServerReceiver` | 通道热切换广播 |
| 队列消费 | 2 个 `extends Thread` 的 Listener，`@PostConstruct` 中 `start()`，`while(true)` 无停机钩子 | 见 §9 的改进建议 |
| 校验 / XSS / 参数解析 | `itcast-tools-validator`、`itcast-tools-xss`、`@EnableLoginArgResolver`、`@EnableFormValidator` | 三个 `SmsXxxWebConfiguration` 均继承 `tools-common` 的 `BaseConfig`，项目内无自定义拦截器 |

---

## 8. `docs/` 资源索引

| 资源 | 内容 |
| --- | --- |
| `docs/mysql/itcast_sms.sql` | 16 张表建表语句 + 演示数据（含 5 条通道配置、13 个接入平台、签名/模板、黑名单） |
| `docs/nacos/nacos_config_export_.zip` | Nacos 配置导出包：`common.yml` / `redis.yml` / `mysql.yml` 及三个应用的端口配置 |
| `docs/dockerfile/redis/docker-compose.yml` | Redis 主从编排 |
| `docs/dockerfile/sentinel/*` | 3 哨兵编排 + `sentinel1/2/3.conf` |
| `docs/安装手册/install.md` | Docker 安装 Nacos/MySQL/Redis → 导入 Nacos 配置与 SQL → 改 namespace → `mvn clean install -DskipTests` → 启动 3 个应用 → Swagger 验证 |
| `docs/img/功能特性.png`、`功能结构图*.png`、`技术架构图.png` | 原文"功能特性 / 功能结构图 / 技术架构"章节配图 |
| `docs/img/页面/*.png` | 首页、通道管理、应用管理、发送记录、黑名单 5 张后台截图 |
| `docs/img/douyin.jpeg`、`weixin.jpeg` | 原文末尾的社群二维码 |
| `docs/安装手册/install.assets/*.png`（26 张） | install.md 的操作步骤截图 |

---

## 9. 已知问题与改进建议（静态代码审阅，未运行验证）

> 以下问题均给出源码位置，供二次开发时优先核查；**未在本机做编译/运行时验证**，其中"功能性缺陷"建议在联调环境复现后再修改。

### 9.1 功能性 / 构建阻断类（高优先级）

1. **`sdk` 未纳入聚合构建**：根 `pom.xml:20-25` 只列 4 个模块，`manage/pom.xml:18-22` 却依赖 `itcast-sms-sdk:1.0.1`。全新环境需先单独 `mvn -f itcast-sms-sdk/pom.xml clean install`，否则根构建失败；`install.md:134` 未说明。
2. **SDK 签名口径与服务端不一致**：服务端 `api/.../SmsSendServiceImpl.java:98` 用 `encode(timestamp, ak, sk)`，SDK `sdk/.../SmsSendServiceImpl.java:52,107` 却用 `encode(smsParamsDTO.getEncryption(), ...)`，且 `BaseParamsDTO.encryption` 初始为 `null`、`encode` 首行即对参数取字节 → 开启 `auth` 时可能空指针，且签名永不匹配。
3. **SDK 空值校验不生效**：`sdk/.../SmsSendServiceImpl.java:49,56,104,111` 的 `R.fail(...)` 只构造返回值没有 `return`，`accessKey` / `domain` 为空时仍继续发请求。
4. **通道耗尽判定疑似 off-by-one**：`server/.../factory/SmsConnectLoader.java:123-125` 为 `CONNECT_LIST.size() <= level`，而 `SmsFactory.java:72` 的 `level` 从 1 起、`getConnectByLevel` 用 `level-1` 取下标；当 `level == size`（最后一个通道）时会被误判为"已全部尝试"，直接转人工处理。
5. **人工处理记录的请求报文被覆盖**：`SmsFactory.java:95-96` 先 `setRequest(JSON.toJSONString(params))` 又 `setRequest(smsSendDTO.getSendTime())`。
6. **模板参数边界缺陷**：`api/.../SmsSendServiceImpl.java:173` 用 `content.indexOf("${") > 0`，占位符出现在首位时漏判（应为 `>= 0`）；`:130-135` 敏感词用 `sensitive.contains(param + ",")` 拼接判定，依赖存储格式，边界脆弱。
7. **文档笔误**：`install.md:92` 的 `docs/naocs/` 应为 `docs/nacos/`，`install.md:141` 的 `SmsManagerApplication` 应为 `SmsManageApplication`；`TemplateType.java:9` 注释"1：验证码，2：验证码"与枚举不符。

### 9.2 安全类（部署前必须处理）

8. **演示密钥与手机号入库**：`docs/mysql/itcast_sms.sql:31-41,74-78,401-413` 含明文 AccessKey/Secret、供应商域名、真实格式手机号；`server/.../sms/HuaweiSmsService.java:68-69` 注释残留 APP_Key / APP_Secret 样例串。上线前必须全量替换并限制 SQL 脚本访问范围。
9. **可生成假数据的调试端点**：`server/.../controller/SmsSendController.java:61-63` 的 `POST /sms/simulation` 无鉴权注解，会随机批量写 `receive_log` / `send_log`，生产环境应关闭或加认证。
10. **依赖仓库使用 HTTP**：`pom.xml:294-300` 的 `http://repo.itheima.net/...` 与 `sdk/pom.xml:23` 的发布地址均为明文 HTTP，存在依赖投毒面，建议改 HTTPS 并校验制品。
11. **敏感信息进日志**：`sdk/.../SmsSendServiceImpl.java:63-64` 以 debug 级别打印 `accessKeyId` / `accessKeySecret`；`manage/.../controller/MarketingController.java:218` 硬编码默认审核人姓名。

### 9.3 架构与工程化类

12. **零自动化测试**：全仓无 `*Test.java` / `@Test`，仅有 test 依赖；校验链、通道降级、签名算法均缺回归保护。
13. **线程模型粗糙**：2 个队列消费者 `extends Thread` + `@PostConstruct start()` + `while(true)` 无优雅停机；`@EnableAsync` 未定义业务线程池；`RedisLock.lock` 用 `Thread.sleep(50)` 忙等抢锁。
14. **能力缺口**：无独立计费/余额模块、无限流组件、无异步状态回执回调（`HuaweiSmsService.java:81` 的 `statusCallBack = ""` 使回调分支永不生效）、Hystrix 只有依赖与 `@EnableHystrix` 而无熔断注解。
15. **反射硬约定**：`SmsConnectLoader.java:84` 按 `"com.itheima.sms." + platform + "SmsService"` 拼类名并反射注入私有字段，新增供应商无编译期校验；注释里的示例包名 `com.itheima.tools.sms.*` 与实际包不符。
16. **重复与失效配置**：3 份近乎相同的 `SmsXxxDatabaseAutoConfiguration` / `ExceptionConfiguration` / `SmsEncryptionUtils`、api 与 server 各重复声明 `spring-boot-starter-json`、空 `dozer/sms.dozer.xml`、`LettuceConfig` 中被注释的序列化设置、`logback-spring.xml` 里 `com.itheima.sms.dao`（实际包是 `mapper`）的 logger、`dependencyManagement` 中未被引用的 minio/qiniu/fastdfs/easy-captcha、`@EnableFeignClients` 却无 `@FeignClient`。
17. **能力孤岛**：`Netty TCP :10000` 入口与 HTTP 入口并存，但没有协议文档与鉴权说明；管理后台与 api/server 共享数据库表，缺少服务边界与迁移版本管理（无 Flyway/Liquibase）。

### 9.4 建议的整改顺序

1. 先补**构建可复现性**：把 `sdk` 纳入 reactor 或在文档中固化单独 install 步骤；补最小单元测试（签名、9 步校验、通道级别）。
2. 修 §9.1 的功能缺陷（签名口径、off-by-one、空值校验、边界判断），并补充端到端联调用例。
3. 部署前清理 §9.2：替换演示密钥、关闭 `/sms/simulation`、私服改 HTTPS、关闭敏感日志。
4. 中长期：引入真实 MQ 与回执回调、可配置线程池与优雅停机、限流/计费、状态回执表，并把重复的自动配置抽取到公共 starter。

---

## 10. 版本控制与忽略策略

仓库根目录 `.gitignore` 已在原有规则基础上按类别整理，**原有规则全部保留**，覆盖以下本地产物：

| 类别 | 忽略项 |
| --- | --- |
| IntelliJ IDEA | `.idea/`（含 `.idea/.gitignore`）、`*.iml`、`*.iws`、`*.ipr`、`out/` |
| Maven / 编译产物 | `target/`、`**/target/`、`*.class`、`.flattened-pom.xml`、`dependency-reduced-pom.xml`、`pom.xml.versionsBackup`、`release.properties`、`.mvn/timing.properties`；**不忽略 `*.jar`**，避免误伤 Maven Wrapper 与需入库依赖 |
| 日志 / 运行时 | `logs/`、`*.log`、`*.log.*`、`log.path_IS_UNDEFINED`、`derby.log`、`hs_err_pid*`、`replay_pid*` |
| Eclipse / STS / NetBeans | `.classpath`、`.project`、`.settings`、`.springBeans`、`.factorypath`、`.apt_generated`、`nbproject/private/`、`build/`、`nbbuild/`、`dist/`、`nbdist/`、`.nb-gradle/`、`generatorConfig.xml` |
| 编辑器 / 操作系统 | `.vscode/`、`*.code-workspace`、`*.swp`、`*.swo`、`*~`、`*.bak`、`*.orig`、`*.rej`、`.DS_Store`、`Thumbs.db`、`ehthumbs.db` |
| 其它 | `rebel.xml`（JRebel）、`offline_user.md`、`third-party/nacos/{derby.log,data/,work/}` |

`.mvn/wrapper/maven-wrapper.jar` 通过 `!` 规则保留（若将来引入 Maven Wrapper）。可用 `git check-ignore -v .idea/misc.xml itcast-sms-api/target/ xxx.iml` 验证规则命中；已跟踪文件不受忽略规则影响，如需从索引移除应使用 `git rm --cached`（本仓库当前**没有** `.class`、`target/`、`.idea/`、`*.iml` 被跟踪）。

---

## 11. 二次开发与排错速查

**新增一个短信供应商**

1. 在 `itcast-sms-server/src/main/java/com/itheima/sms/sms/` 新建 `XxxSmsService extends AbstractSmsService`，提供 `SmsConfig` 构造器，实现 `sendSms` / `sendSmsBatch`；失败返回以 `FAIL@#@` 开头。
2. 在 `config` 表插入一行，`platform` 字段值必须为 `Xxx`（与类名前缀严格一致），并填 `domain`、AK/SK、`other`(JSON)、`level`、`is_active/is_enable`。
3. 在 `config_signature` / `config_template` 建立该通道与签名、模板的关联（供应商侧模板 ID 也在这里配置）。
4. 重启 `itcast-sms-server` 或等待通道热切换广播（`TOPIC_HIGH_SERVER` → `INIT_CONNECT`），观察启动日志"通道初始化成功"。

**常见问题**

| 现象 | 排查方向 |
| --- | --- |
| 根构建报找不到 `itcast-sms-sdk:1.0.1` | SDK 不在 reactor，需先单独 install（§2.2） |
| 启动连不上 Nacos / 拉不到配置 | 检查 Maven profile（`-Pdev/test/prod`）、`@pom.nacos.*@` 是否被资源过滤替换、`NACOS_IP/NACOS_PORT/NACOS_ID` 环境变量、namespace 是否为 **ID** 而非名称 |
| 应用端口不明 / Swagger 打不开 | `server.port` 在 Nacos 配置里；启动日志会打印 `/doc.html` 与 `/druid` 完整地址 |
| 提示"未找到支持当前签名和模板的通道" | 检查 `config_signature` / `config_template` 关联及 `config.is_active/is_enable` |
| 短信一直进 `manual_process` | 检查通道级别判定与 `configIds` 匹配（§9.1 第 4 条） |
| 开启鉴权后客户端调用失败 | 核对 SDK 与服务端签名口径（§9.1 第 2 条） |
| 定时短信不发送 | 检查 `timing_push`、`SendSmsJob`（每分钟第 10 秒）、Redis 分布式锁是否被占用 |
| SQL 慢 / 想看 SQL | dev 环境启用 P6Spy（`spy.properties`，慢 SQL 阈值 2s） |

**常用命令**

```bash
# 全量构建（需先单独安装 sdk）
mvn -f itcast-sms-sdk/pom.xml clean install -DskipTests
mvn clean install -DskipTests

# 切换环境
mvn clean install -DskipTests -Ptest

# 启动 Redis 主从与哨兵
cd docs/dockerfile/redis && docker-compose -f docker-compose.yml up -d
cd ../sentinel && docker-compose -f docker-compose.yml up -d

# 检查忽略规则是否命中
git check-ignore -v .idea/misc.xml itcast-sms-api/target/ foo.iml
```

---

> **文档维护**：本章节为静态分析产物，代码、依赖版本或部署方式变更后请同步更新对应小节；原始项目介绍、演示地址与安装手册链接仍以本文档上半部分为准。
