# Loadbalance核心接口

## 接口列表

* IRule
* ILoadBalancer
* ServerList
* ServerListFilter
* IPing

## 功能介绍

配置入口：RibbonClientConfiguration

合适的方式选择合适的节点

### ILoadBalancer

接口定义：com.netflix.loadbalancer.ILoadBalancer

核心方法：

```java
	/**
	 * Choose a server from load balancer.
	 * 
	 * @param key An object that the load balancer may use to determine which server to return. null if 
	 *         the load balancer does not use this parameter.
	 * @return server chosen
	 */
	public Server chooseServer(Object key);
/**
 * @return Only the servers that are up and reachable.
    */
   public List<Server> getReachableServers();

   /**
    * @return All known servers, both reachable and unreachable.
    */
public List<Server> getAllServers();
```

核心作用：维护全量服务列表及可用列表，依赖IPing接口标记失效节点，标识失效依赖IPing实现

默认实现：ZoneAwareLoadBalancer

### IRule

接口定义：com.netflix.loadbalancer.IRule

核心方法：

```java
/*
 * choose one alive server from lb.allServers or
 * lb.upServers according to key
 * 
 * @return choosen Server object. NULL is returned if none
 *  server is available 
 */

public Server choose(Object key);
```

核心作用：从loadbalance里面选取一个server；提供多种选择策略

默认实现：ZoneAvoidanceRule



### ServerList

接口定义：com.netflix.loadbalancer.ServerList

核心方法：

```java
/**
 * Return updated list of servers. This is called say every 30 secs
 * (configurable) by the Loadbalancer's Ping cycle
 * 
 */
public List<T> getUpdatedListOfServers(); 
```

核心作用：获取更新服务列表

默认实现：ConfigurationBasedServerList

### ServerListFilter

接口定义：com.netflix.loadbalancer.ServerListFilter

核心方法：

```java
public List<T> getFilteredListOfServers(List<T> servers);
```

核心作用：

默认实现：ZonePreferenceServerListFilter

### IPing

接口定义：com.netflix.loadbalancer.IPing

核心方法：

```java
/**
 * Checks whether the given <code>Server</code> is "alive" i.e. should be
 * considered a candidate while loadbalancing
 * 
 */
public boolean isAlive(Server server);
```

核心作用：检测特定服务是否活着，在ILoadBalance中做失效服务检测作用

默认实现：DummyPing

## 扩展思考

Ribbon最大的有点就是允许用户定制化各种规则，很好的管理了节点状态及访问规则，基于规则扩展做一些好用的产品，比如灰度管理、目标节点能力不对等下的权重控制

## ref
https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-ribbon.html
