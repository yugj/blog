# Dubbo Tag

## 自定义URL参数

方式1

```shell script
-Ddubbo.provider.parameters.xxxTag=abcd
```

方式2，重写ZookeeperRegistry

## 降级约定
1，dubbo.tag=tag1 时优先选择 标记了tag=tag1 的 provider。若集群中不存在与请求标记对应的服务，默认将降级请求 tag为空的provider；如果要改变这种默认行为，即找不到匹配tag1的provider返回异常，需设置dubbo.force.tag=true。

2，dubbo.tag未设置时，只会匹配tag为空的provider。即使集群中存在可用的服务，若 tag 不匹配也就无法调用，这与约定1不同，携带标签的请求可以降级访问到无标签的服务，但不携带标签/携带其他种类标签的请求永远无法访问到其他标签的服务。

## 参考
* https://dubbo.apache.org/zh/docs/advanced/routing-rule/