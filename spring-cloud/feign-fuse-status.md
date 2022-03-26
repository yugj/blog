# Hystrix Feign 特定状态码不熔断

## 背景

部分业务校验规范或疏忽场景，传入bad request 导致接口熔断，影响接口正常流量

## 处理

重写Feign error decoder逻辑

```java
import com.netflix.hystrix.exception.HystrixBadRequestException;
import feign.Response;
import feign.Util;
import feign.codec.ErrorDecoder;

import static java.lang.String.format;

/**
 * @author yugj
 * @date 2020/4/29 9:13 上午.
 */
public class SkipHttpStatusErrorDecoder extends ErrorDecoder.Default {

    public SkipHttpStatusErrorDecoder() {
        super();
    }

    @Override
    public Exception decode(String methodKey, Response response) {

        int status = response.status();
        if (status == 400 || status == 404) {
            String message = statusFormat(methodKey, response);
            return new HystrixBadRequestException(message);
        }

        return super.decode(methodKey, response);
    }

    private String statusFormat(String methodKey, Response response) {

        String message = format("status %s reading %s", response.status(), methodKey);
        if (response.body() != null) {
            try {
                String body = Util.toString(response.body().asReader());
                message += "; content:\n" + body;
            } catch (Exception e) {
                //do nothing
            }
        }

        return message;
    }
}
```



```java
@Configuration
public class SkipHttpStatusConfiguration {
    @Bean
    public SkipHttpStatusErrorDecoder errorDecoder() {
        return new SkipHttpStatusErrorDecoder();
    }

```



原理

com.netflix.hystrix.AbstractCommand#executeCommandAndObserve异常控制HystrixBadRequestException没有走到熔断逻辑