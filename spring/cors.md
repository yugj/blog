# cors
## WebFilter跨域配置

### 配置代码
```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.web.cors.reactive.CorsUtils;
import org.springframework.web.server.ServerWebExchange;
import org.springframework.web.server.WebFilter;
import org.springframework.web.server.WebFilterChain;
import reactor.core.publisher.Mono;

/**
 * @author yugj
 * @date 2022/3/14 09:35.
 */
@Order(-100)
public class GatewayCorsFilter implements WebFilter {

    @Value("${sisyphus.cors.allowedHeaders}")
    private String allowedHeaders;

    private static final String ALLOWED_METHODS = "GET, PUT, POST, DELETE, OPTIONS";
    private static final String ALLOWED_ORIGIN = "http://api.demo.cn";

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String origin = request.getHeaders().getOrigin();
        if (origin == null) {
            origin = ALLOWED_ORIGIN;
        }
        //允许所有请求跨域
        if (CorsUtils.isCorsRequest(request)) {
            ServerHttpResponse response = exchange.getResponse();

            HttpHeaders headers = response.getHeaders();
            headers.add(HttpHeaders.ACCESS_CONTROL_ALLOW_ORIGIN, origin);
            headers.add(HttpHeaders.ACCESS_CONTROL_ALLOW_METHODS, ALLOWED_METHODS);
            headers.add(HttpHeaders.ACCESS_CONTROL_ALLOW_HEADERS, allowedHeaders);

            headers.add(HttpHeaders.ACCESS_CONTROL_ALLOW_CREDENTIALS, "true");
            if (request.getMethod() == HttpMethod.OPTIONS) {
                response.setStatusCode(HttpStatus.OK);
                return Mono.empty();
            }
        }
        return chain.filter(exchange);
    }
}

```

说明：
```
ACCESS_CONTROL_ALLOW_CREDENTIALS, "true" 该属性将允许跨域设置cookie，
ACCESS_CONTROL_ALLOW_ORIGIN不能为*，需要根据客户端请求设置origin，_ACCESS_CONTROL_ALLOW_HEADERS在option请求过来，
将探测允许请求的header头，若服务端未返回给客户端，流量器将提示头信息不在允许范围_

```

### 前端测试代码

```javascript
var xhr = new XMLHttpRequest();  
xhr.open('post', 'http://88.localhost.cn:8089/xxx',false);  
xhr.setRequestHeader("appId", "2");  
xhr.setRequestHeader("poolId", "2");  
xhr.setRequestHeader("api-tag", "xxx");

xhr.setRequestHeader("content-type", "application/json");
// 该属性允许跨域cookie传输到服务端，允许跨域设置响应cookie（同一个根域名下
xhr.withCredentials=true;   
xhr.send('{"code":"xx","phone":"xxx"}');  
console.log(xhr.responseText);  
console.log(xhr.getAllResponseHeaders());
```

cookie在http协议中基于header头传输，key为Set-Cookie 值格式

```shell script
%s=%s; domain=%s; path=%s
// key=value; domain=xxx.google.com; path=/
```


参考文档：

[https://stackoverflow.com/questions/32500073/request-header-field-access-control-allow-headers-is-not-allowed-by-itself-in-pr](https://stackoverflow.com/questions/32500073/request-header-field-access-control-allow-headers-is-not-allowed-by-itself-in-pr)