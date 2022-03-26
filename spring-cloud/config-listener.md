# Spring Cloud 配置变化监听

## 背景

开发中遇到个需求，期望可以在配置变更的时候，监听配置的变化，做一些逻辑处理，原生ApplicationEvent已经有发出对应的配置更新事件，但是包含的是所有的变更，开发人员一般只关心自己需要的配置变更

原生事件发出EnvironmentChangeEvent（如spring cloud config）或RefreshEvent（nacos 最终也会发EnvironmentChangeEvent事件）

对此我们可以基于EnvironmentChangeEvent，最一层包装，封装我们自己需要的事件

## 使用

先看下效果，直接基于EventListener捕获ConfigRefreshEvent，condition使用el表达式配置需要监听的key，针对集合或map 可以使用正则匹配

```java
@EventListener(condition = "#event.key eq 'sys.loglevel.root'")
void handleConditionalListener(ConfigRefreshEvent event) {
    // 业务逻辑 balabala
    System.out.println("handleConditionalListener event key :" + event.getKey()
    + ", before :" + event.getBeforeRefresh()
    + ", after :" + event.getAfterRefresh());
}

			/**
     * LIST 或者 MAP 通过正则表达式匹配前缀
     * LIST key为 sys.loglevel.clazzLevel[0] sys.loglevel.clazzLevel[1] 类似格式
     * MAP key 为sys.loglevel.clazzLevel.key1 sys.loglevel.clazzLevel.key2 类似格式
     * @param event
     */
    @EventListener(condition = "{#event.key matches '^sys.loglevel.clazzLevel.*'}")
    void clazzLogChangeListener(ConfigRefreshEvent event) {

        log.info("receive log level change event, key :{}, value :{}", event.getKey(), event.getAfterRefresh());
    }
```

需要的依赖

```xml
<dependency>
    <groupId>com.github.yugj</groupId>
    <artifactId>config-refresh-listener-core</artifactId>
    <version>2.0.0</version>
</dependency>
```

参考代码：https://github.com/yugj/spring-cloud-config-refresh-listener

## 实现原理

代码很简单直接一个监听

```
/**
 * 配置刷新监听
 * @author yugj
 * @since 1.0-SNAPSHOT
 */
@Service
public class ConfigRefreshListener implements Ordered {

    @Autowired
    private Environment environment;
    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @EventListener(EnvironmentChangeEvent.class)
    public void listener(EnvironmentChangeEvent event) {
        for (String refreshKey : event.getKeys()) {
        		//查询刷新配置的值，重新包装为自定义的ConfigRefreshEvent
            Object afterRefreshed = environment.getProperty(refreshKey);
            eventPublisher.publishEvent(new ConfigRefreshEvent(this,refreshKey, afterRefreshed, afterRefreshed));
        }
    }

    @Override
    public int getOrder() {
        return LOWEST_PRECEDENCE - 1;
    }
}
```

直接监听最后的EnvironmentChangeEvent事件（spring cloud各自配置服务刷新最终都会转换为该事件），在该事件触发的时候，包装为我们定义的ConfigRefreshEvent，然后依赖spring 事件监听即可

项目中考虑到代码简单通用，没有获取变化前的数据

若需要获取变化之前的数据可以参考org.springframework.cloud.context.refresh.ContextRefresher

```java
public synchronized Set<String> refresh() {
		Set<String> keys = refreshEnvironment();
		this.scope.refreshAll();
		return keys;
	}

	public synchronized Set<String> refreshEnvironment() {
    //刷新之前的值,
		Map<String, Object> before = extract(
				this.context.getEnvironment().getPropertySources());
		addConfigFilesToEnvironment();
    //变化的key值，在上面this.scope.refreshAll();之前可以通过变化的key去查询before值获取到变化之前的值
		Set<String> keys = changes(before,
				extract(this.context.getEnvironment().getPropertySources())).keySet();
		this.context.publishEvent(new EnvironmentChangeEvent(this.context, keys));
		return keys;
	}
```

## 扩展使用

类似上面demo可以基于config server做日志级别的调整及其他需要在进程内维护的数据监听变更