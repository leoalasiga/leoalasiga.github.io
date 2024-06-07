---
date: 2024-06-07
authors:
  - leoalasiga
tags:
 - redis
 - java
 - springboot
comments: false
draft: true
---

# Redis开启工作空间

redis开启工作空间通知

redis的配置文件设置：**notify-keyspace-events** 

notify-keyspace-events的参数值可以是以下字符的任意组合，它指定了服务器该发送哪些类型的通知。该参数将针对整个实例（所有DB）启用通知，启用后会额外消耗CPU，更多信息请参见[Redis keyspace notifications](https://redis.io/docs/manual/keyspace-notifications/)。

- K：键空间通知，所有通知以__keyspace@__为前缀。
- E：键事件通知，所有通知以__keyevent@__为前缀。
- g：**DEL**、**EXPIRE**、**RENAME**等类型无关的通用命令的通知。
- $：字符串命令的通知，会发送关于字符串的创建、修改、删除等操作的通知。
- l：列表命令的通知。
- s：集合命令的通知。
- h：哈希命令的通知。
- z：有序集合命令的通知。
- x：过期事件，不一定在键过期时发送，而是在过期键被删除时发送。
- e：驱逐（evict）事件，每当有键因为maxmemory政策而被删除时发送。
- A：参数**g$lshzxe**的别名，表示监听上述所有事件，设置示例为**AKE**。

**重要**

输入的参数中至少包含K或E， 否则不会有任何通知被分发。

例如您希望订阅过期事件，您可以在参数设置中将该参数设置为Ex。设置参数后，在客户端执行对应的订阅命令：PSUBSCRIBE __keyevent@0__*，表示订阅DB0的键事件通知。

然后springboot整合spring-data-redis，实现工作空间消息的接收

```java
@Component
public class RedisKeyExpirationListener extends KeyExpirationEventMessageListener {

    public RedisKeyExpirationListener(RedisMessageListenerContainer listenerContainer) {
        super(listenerContainer);
    }

    @Override
    public void onMessage(Message message, byte[] pattern) {
        String expiredKey = message.toString();
        System.out.println("Key expired: " + expiredKey);

        // 在此处添加你的业务逻辑，比如清理本地缓存、发送通知等
    }
}
```

然后你就可以按照你的需要做一些工作了

然后这边的redis的listener，来自于这个**RedisMessageListenerContainer**

```java
public class KeyExpirationEventMessageListener extends KeyspaceEventMessageListener implements ApplicationEventPublisherAware {
    private static final Topic KEYEVENT_EXPIRED_TOPIC = new PatternTopic("__keyevent@*__:expired");
    @Nullable
    private ApplicationEventPublisher publisher;

    public KeyExpirationEventMessageListener(RedisMessageListenerContainer listenerContainer) {
        super(listenerContainer);
    }

    protected void doRegister(RedisMessageListenerContainer listenerContainer) {
        listenerContainer.addMessageListener(this, KEYEVENT_EXPIRED_TOPIC);
    }

    protected void doHandleMessage(Message message) {
        this.publishEvent(new RedisKeyExpiredEvent(message.getBody()));
    }

    protected void publishEvent(RedisKeyExpiredEvent event) {
        if (this.publisher != null) {
            this.publisher.publishEvent(event);
        }

    }

    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.publisher = applicationEventPublisher;
    }
}
```

它里面维护了一个线程池

```java
public class RedisMessageListenerContainer implements InitializingBean, DisposableBean, BeanNameAware, SmartLifecycle {
    ···
    @Nullable
    private Executor taskExecutor;
    ···
    
    public void afterPropertiesSet() {
        Assert.state(!this.afterPropertiesSet, "Container already initialized.");
        if (this.connectionFactory == null) {
            throw new IllegalArgumentException("RedisConnectionFactory is not set");
        } else {
            if (this.taskExecutor == null) {
                this.manageExecutor = true;
                this.taskExecutor = this.createDefaultTaskExecutor();
            }
    
            if (this.subscriptionExecutor == null) {
                this.subscriptionExecutor = this.taskExecutor;
            }
    
            this.subscriber = this.createSubscriber(this.connectionFactory, this.subscriptionExecutor);
            this.afterPropertiesSet = true;
        }
    }

    protected TaskExecutor createDefaultTaskExecutor() {
        String threadNamePrefix = this.beanName != null ? this.beanName + "-" : DEFAULT_THREAD_NAME_PREFIX;
        return new SimpleAsyncTaskExecutor(threadNamePrefix);
    }

    public void destroy() throws Exception {
        this.afterPropertiesSet = false;
        this.stop();
        if (this.manageExecutor && this.taskExecutor instanceof DisposableBean) {
            ((DisposableBean)this.taskExecutor).destroy();
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Stopped internally-managed task executor.");
            }
         }
    }
    ···
}
```

然后这个**SimpleAsyncTaskExecutor**的核心方法是这个，相当于他执行doExecute方法的时候，会触发创建线程的任务，相当于这个listener是多线程的任务，可以一直跑动

```java
public class SimpleAsyncTaskExecutor extends CustomizableThreadCreator implements AsyncListenableTaskExecutor, Serializable {
    ···
    public SimpleAsyncTaskExecutor(String threadNamePrefix) {
        super(threadNamePrefix);
    }
    ···

    protected void doExecute(Runnable task) {
        Thread thread = this.threadFactory != null ? this.threadFactory.newThread(task) : this.createThread(task);
        thread.start();
    }
    ···
}
```



## 参考

[阿里云redis配置参数列表](https://help.aliyun.com/zh/redis/user-guide/supported-parameters)
