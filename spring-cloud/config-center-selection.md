# 发现&配置中心选型

2020-06-08 

## 配置中心产品功能对比

| 功能           | spring cloud config                                        | apollo                          | nacos                      | consul                |
| -------------- | ---------------------------------------------------------- | ------------------------------- | -------------------------- | --------------------- |
| 管理端配置管理 | 自己开发基于gitlab管理                                     | 支持                            | 支持                       | 支持                  |
| 配置刷新       | 依赖Git的WebHook<br />Spring Cloud Bus和客户端/bus/refresh | http poll                       | http poll                  | http poll             |
| 权限控制       | 依赖gitlab                                                 | 支持                            | 简单                       | 不支持                |
| 灰度发布       | 依赖destination 不完整                                     | 支持                            | 支持                       | 不支持                |
| 配置回滚       | 不支持                                                     | 支持                            | 支持                       | 不支持                |
| 程序支持       | java                                                       | java .net                       | java、go、node、Python、c# |                       |
| 最小依赖       | config3 + kafka3 + zk3 + gitlab2                           | Config*2+Admin*3+Portal*2+Mysql | nacos3 + mysql             | consul server、 agent |
| 厂商           | netflix                                                    | 携程                            | 阿里                       | HashiCorp             |
| 多环境支持     | 支持                                                       | 支持                            | 支持                       | 支持                  |
| 多项目支持     | 可以支持                                                   | 支持                            | 支持                       | 可以支持              |
| 配置共享       | 支持                                                       | 支持                            | 支持                       | 不支持                |
| 注册发现功能   | 无                                                         | 无                              | 有                         | 有                    |

## 注册中心对比

|                 | **Nacos**                  | **Eureka**  | **Consul**        |
| :-------------- | :------------------------- | :---------- | :---------------- |
| 一致性协议      | CP+AP                      | AP          | CP                |
| 健康检查        | TCP/HTTP/MYSQL/Client Beat | Client Beat | TCP/HTTP/gRPC/Cmd |
| 负载均衡策略    | 权重/ metadata/Selector    | Ribbon      | Fabio             |
| 雪崩保护        | 有                         | 有          | 无                |
| 自动注销实例    | 支持                       | 支持        | 不支持            |
| 访问协议        | HTTP/DNS                   | HTTP        | HTTP/DNS          |
| 监听支持        | 支持                       | 支持        | 支持              |
| 多数据中心      | 支持                       | 支持        | 支持              |
| 跨注册中心同步  | 支持                       | 不支持      | 支持              |
| SpringCloud集成 | 支持                       | 支持        | 支持              |
| Dubbo集成       | 支持                       | 不支持      | 不支持            |
| K8S集成         | 支持                       | 不支持      | 支持              |

单从配置中心方面来看 Apollo目前更完善，从服务治理方面nacos集成了注册发现，另外对k8s、和监控支持友好

另外nacos提供权重负载均衡策略，比较适合我们这边物理机和虚拟机性能不均衡场景；ribbon扩展基于权重需要维护元数据和编写客户端权重负载均衡，目前有灰度框架支持，但是管理不方便

国产的产品比较倾向于提供丰富的功能，整体使用和集成会相对简单



**选型建议**

鉴于Eureka、hystrix不继续迭代，考虑基于nacos + sentinel的服务治理方案，它们对Spring Cloud灰度发布和路由更具出色的兼容性和友好性

结合灰度管理框架[Nepxion/Discovery](https://github.com/Nepxion/Discovery) 让整体灰度策略管理更加方便

考虑nacos主要考虑点：

* 配置刷新基于poll减少中间件依赖
* 灰度配置
* 注册中心
* 主机权重配置、
* 雪崩保护
* K8S集成
* 多语言程序支持
* 配置管理控制台

## 参考文档

https://yq.aliyun.com/articles/698930?spm=a2c6h.12873639.0.0.428675c7XlEhlY

https://jimmysong.io/blog/what-is-a-service-mesh/