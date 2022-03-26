# Ribbon 全局修改irule
1.x版本
```
/**
 * @author Spencer Gibb
 */
@Configuration
@Import({ PropertyPlaceholderAutoConfiguration.class, ArchaiusAutoConfiguration.class,
		UtilAutoConfiguration.class, RibbonAutoConfiguration.class })
@RibbonClients(defaultConfiguration = DefaultRibbonConfig.class)
public class RibbonClientDefaultConfigurationTestsConfig {

	public static class BazServiceList extends ConfigurationBasedServerList {
		public BazServiceList(IClientConfig config) {
			super.initWithNiwsConfig(config);
		}
	}
}

@Configuration
class DefaultRibbonConfig {

	@Bean
	public IRule ribbonRule() {
		return new BestAvailableRule();
	}

	@Bean
	public IPing ribbonPing() {
		return new PingUrl();
	}

	@Bean
	public ServerList<Server> ribbonServerList(IClientConfig config) {
		return new RibbonClientDefaultConfigurationTestsConfig.BazServiceList(config);
	}

	@Bean
	public ServerListSubsetFilter serverListFilter() {
		ServerListSubsetFilter filter = new ServerListSubsetFilter();
		return filter;
	}

}
```
[https://github.com/spring-cloud/spring-cloud-netflix/issues/1741](https://github.com/spring-cloud/spring-cloud-netflix/issues/1741)

2.x版本

重写irule
```
@Bean public IRule ribbonRule() {
    return new NacosWeightRandomRule();
}```