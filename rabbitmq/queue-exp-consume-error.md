## RabbitMq queue异常导致rabbitmq consumer停止消费问题处理

## 问题

偶发性rabbitmq出问题或者认为操作错误，访问不了queue，导致消费端停止消费

```
org.springframework.amqp.rabbit.listener.QueuesNotAvailableException: Cannot prepare queue for listener. Either the queue doesn't exist or the broker will not allow us to use it.
	at org.springframework.amqp.rabbit.listener.BlockingQueueConsumer.start(BlockingQueueConsumer.java:599)
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$AsyncMessageProcessingConsumer.run(SimpleMessageListenerContainer.java:1424)
	at java.lang.Thread.run(Thread.java:745)
Caused by: org.springframework.amqp.rabbit.listener.BlockingQueueConsumer$DeclarationException: Failed to declare queue(s):[s.message.like.add]
	at org.springframework.amqp.rabbit.listener.BlockingQueueConsumer.attemptPassiveDeclarations(BlockingQueueConsumer.java:672)
	at org.springframework.amqp.rabbit.listener.BlockingQueueConsumer.start(BlockingQueueConsumer.java:571)
	... 2 common frames omitted
Caused by: java.io.IOException: null
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:105)
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:101)
	at com.rabbitmq.client.impl.AMQChannel.exnWrappingRpc(AMQChannel.java:123)
	at com.rabbitmq.client.impl.ChannelN.queueDeclarePassive(ChannelN.java:992)
	at com.rabbitmq.client.impl.ChannelN.queueDeclarePassive(ChannelN.java:50)
	at sun.reflect.GeneratedMethodAccessor711.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory$CachedChannelInvocationHandler.invoke(CachingConnectionFactory.java:980)
	at com.sun.proxy.$Proxy231.queueDeclarePassive(Unknown Source)
	at org.springframework.amqp.rabbit.listener.BlockingQueueConsumer.attemptPassiveDeclarations(BlockingQueueConsumer.java:651)
	... 3 common frames omitted
Caused by: com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - failed to perform operation on queue 'q.hell' in vhost '/' due to timeout, class-id=50, method-id=10)
	at com.rabbitmq.utility.ValueOrException.getValue(ValueOrException.java:66)
	at com.rabbitmq.utility.BlockingValueOrException.uninterruptibleGetValue(BlockingValueOrException.java:32)
	at com.rabbitmq.client.impl.AMQChannel$BlockingRpcContinuation.getReply(AMQChannel.java:366)
	at com.rabbitmq.client.impl.AMQChannel.privateRpc(AMQChannel.java:229)
	at com.rabbitmq.client.impl.AMQChannel.exnWrappingRpc(AMQChannel.java:117)
	... 11 common frames omitted
Caused by: com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - failed to perform operation on queue 's.message.like.add' in vhost '/' due to timeout, class-id=50, method-id=10)
	at com.rabbitmq.client.impl.ChannelN.asyncShutdown(ChannelN.java:505)
	at com.rabbitmq.client.impl.ChannelN.processAsync(ChannelN.java:336)
	at com.rabbitmq.client.impl.AMQChannel.handleCompleteInboundCommand(AMQChannel.java:143)
	at com.rabbitmq.client.impl.AMQChannel.handleFrame(AMQChannel.java:90)
	at com.rabbitmq.client.impl.AMQConnection.readFrame(AMQConnection.java:634)
	at com.rabbitmq.client.impl.AMQConnection.access$300(AMQConnection.java:47)
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:572)
	... 1 common frames omitted
	
	Stopping container from aborted consumer
```

## 跟踪

SimpleMessageListenerContainer

```java 
catch (QueuesNotAvailableException ex) {
				logger.error("Consumer received fatal=" + SimpleMessageListenerContainer.this.mismatchedQueuesFatal
						+ " exception on startup", ex);
				if (SimpleMessageListenerContainer.this.missingQueuesFatal) {
					logger.error("Consumer received fatal exception on startup", ex);
					this.startupException = ex;
					// Fatal, but no point re-throwing, so just abort.
					aborted = true;
				}
				publishConsumerFailedEvent("Consumer queue(s) not available", aborted, ex);
			}
```

SimpleMessageListenerContainer.this.missingQueuesFatal = true 将aborted

查代码跟踪，如下配置设置

```java
package indi.yugj.test.springcloud.rabbitmq;

import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.boot.autoconfigure.amqp.SimpleRabbitListenerContainerFactoryConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author yugj
 * @date 2020-04-21 18:29.
 */
@Configuration
public class RabbitConfiguration {

    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(
            SimpleRabbitListenerContainerFactoryConfigurer configurer,
            ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();

        //Whether to fail if the queues declared by the container are not available on the broker and/or whether to stop the container if one or more queues are deleted at runtime.
        factory.setMissingQueuesFatal(false);
        configurer.configure(factory, connectionFactory);
        return factory;
    }

}
```

至此涉及queue访问失败场景 会一直重试 直到可用，默认5次