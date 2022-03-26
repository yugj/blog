# zuul http请求跟踪

```java
@Configuration
@ConditionalOnClass(name = "org.apache.http.client.HttpClient")
@ConditionalOnProperty(name = "ribbon.httpclient.enabled", matchIfMissing = true)
public class HttpClientRibbonConfiguration {

    @Bean
    @ConditionalOnMissingBean(HttpClientConnectionManager.class)
    public HttpClientConnectionManager httpClientConnectionManager(
        IClientConfig config,
        ApacheHttpClientConnectionManagerFactory connectionManagerFactory) {
      Integer maxTotalConnections = config.getPropertyAsInteger(
          CommonClientConfigKey.MaxTotalConnections,
          DefaultClientConfigImpl.DEFAULT_MAX_TOTAL_CONNECTIONS);
      Integer maxConnectionsPerHost = config.getPropertyAsInteger(
          CommonClientConfigKey.MaxConnectionsPerHost,
          DefaultClientConfigImpl.DEFAULT_MAX_CONNECTIONS_PER_HOST);
      Integer timerRepeat = config.getPropertyAsInteger(
          CommonClientConfigKey.ConnectionCleanerRepeatInterval,
          DefaultClientConfigImpl.DEFAULT_CONNECTION_IDLE_TIMERTASK_REPEAT_IN_MSECS);
      Object timeToLiveObj = config
          .getProperty(CommonClientConfigKey.PoolKeepAliveTime);
      Long timeToLive = DefaultClientConfigImpl.DEFAULT_POOL_KEEP_ALIVE_TIME;
      Object ttlUnitObj = config
          .getProperty(CommonClientConfigKey.PoolKeepAliveTimeUnits);
      TimeUnit ttlUnit = DefaultClientConfigImpl.DEFAULT_POOL_KEEP_ALIVE_TIME_UNITS;
      if (timeToLiveObj instanceof Long) {
        timeToLive = (Long) timeToLiveObj;
      } else if (timeToLiveObj instanceof String) {
        timeToLive = Long.valueOf((String)timeToLiveObj);
      }
      if (ttlUnitObj instanceof TimeUnit) {
        ttlUnit = (TimeUnit) ttlUnitObj;
      }
      final HttpClientConnectionManager connectionManager = connectionManagerFactory
          .newConnectionManager(false, maxTotalConnections,
              maxConnectionsPerHost, timeToLive, ttlUnit, registryBuilder);
      this.connectionManagerTimer.schedule(new TimerTask() {
        @Override
        public void run() {
          connectionManager.closeExpiredConnections();
        }
      }, 30000, timerRepeat);
      return connectionManager;
    }
​
    @Bean
    @ConditionalOnMissingBean(CloseableHttpClient.class)
    public CloseableHttpClient httpClient(ApacheHttpClientFactory httpClientFactory,
                       HttpClientConnectionManager connectionManager, IClientConfig config) {
      Boolean followRedirects = config.getPropertyAsBoolean(
          CommonClientConfigKey.FollowRedirects,
          DefaultClientConfigImpl.DEFAULT_FOLLOW_REDIRECTS);
      Integer connectTimeout = config.getPropertyAsInteger(
          CommonClientConfigKey.ConnectTimeout,
          DefaultClientConfigImpl.DEFAULT_CONNECT_TIMEOUT);
      RequestConfig defaultRequestConfig = RequestConfig.custom()
          .setConnectTimeout(connectTimeout)
          .setRedirectsEnabled(followRedirects).build();
      this.httpClient = httpClientFactory.createBuilder().
          setDefaultRequestConfig(defaultRequestConfig).
          setConnectionManager(connectionManager).build();
      return httpClient;
    }
}
```