# 82 springboot： 线程池ThreadPoolTaskExecutor



```cpp
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);//核心线程大小
        executor.setMaxPoolSize(2);//最大线程大小
        executor.setQueueCapacity(500);//d队列最大容量
   executor.setThreadNamePrefix("Gith");
         executor.setKeepAliveSeconds(3000);//最大存活 时间
        executor.initialize();
        executor.submit(new ThreadDemo());
        return executor;
    }
```

CorePoolSize 核心线程大小
keepAliveSeconds 线程池所能允许的空闲时间
maxPoolSize 线程池维护线程的最大数量
queueCapacity 线程池所能使用的缓冲队列

当一个任务通过execute(方法，欲加到线程池中时；)
1 如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。

2，如果线程池中的数量等于corePoolsize，缓冲队列workQueue未满，那么任务被放入缓冲队列
3，如果此时线程池中的数量大于coreSize,缓冲队列workQueue满，并且线程池的数量小于maximumPoolSize,建立新的线程来处理被添加的任务。

4 如果此时线程池中的数量大于corePoolSize,缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过handler所指定的策略来处理任务。也就是：处理任务的优先级为: 核心线程corePoolSize, 任务队列workQueue,最大线程maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。

5，当线程池中的线程数量大于corePoolSize 时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。



```kotlin
package ch2.taskexecutor;
 
 
//执行器
import java.util.concurrent.Executor;
//异步捕获助手
import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
 
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ComponentScan;
 
//配置
import org.springframework.scheduling.annotation.AsyncConfigurer;
//异步支持注解
import org.springframework.scheduling.annotation.EnableAsync;
//线程池
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
 
 
//声明这是一个配置类
@Configuration
//引入ch2.taskexecutor下面的@service,@component,@repository,@controller注册为bean
@ComponentScan("ch2.taskexecutor")
//开启注解：开启异步支持
@EnableAsync
 
//配置类实现AsyncConfigurer接口并重写AsyncConfigurer方法，并返回一个ThreadPoolTaskExecutor
//这样我们就得到了一个基于线程池的TaskExecutor
public class TaskExecutorConfig implements AsyncConfigurer {
 
     
    //配置类实现AsyncConfigurer接口并重写AsyncConfigurer方法，并返回一个ThreadPoolTaskExecutor
    //这样我们就得到了一个基于线程池的TaskExecutor
    @Override
    public Executor getAsyncExecutor() {
        // TODO Auto-generated method stub
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        //如果池中的实际线程数小于corePoolSize,无论是否其中有空闲的线程，都会给新的任务产生新的线程
        taskExecutor.setCorePoolSize(5);
        //连接池中保留的最大连接数。Default: 15 maxPoolSize  
        taskExecutor.setMaxPoolSize(10);
        //queueCapacity 线程池所使用的缓冲队列
        taskExecutor.setQueueCapacity(25);
        taskExecutor.initialize();
        return taskExecutor;
    }
 
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        // TODO Auto-generated method stub
        return null;
    }
     
 
}
```



```java
package ch2.taskexecutor;
//组件声明类
import org.springframework.stereotype.Service;
//异步声明,如果在方法表示是异步方法，如果在类表示异步类。
//这里的方法自动被注入使用ThreadPoolTaskExecutor作为TaskExecutor（线程池）
import org.springframework.scheduling.annotation.Async;
 
//声明为组件
@Service
public class AsyncService {
 
     
    //异步声明,如果在方法处表示是异步方法，如果在类处表示异步类（所有的方法都是异步方法）。
    //这里的方法自动被注入使用ThreadPoolTaskExecutor作为TaskExecutor（线程池）
    @Async
    public void executorAsyncTask(Integer i)
    {
        System.out.println("执行异步：" + i);
    }
     
     
    //异步声明,如果在方法处表示是异步方法，如果在类处表示异步类（所有的方法都是异步方法）。
    //这里的方法自动被注入使用ThreadPoolTaskExecutor作为TaskExecutor（线程池）
    @Async
    public void executorAsyncTaskPlus(Integer i)
    {
        System.out.println("执行异步任务+1: " + (i+1));
    }
     
}
```



```java
package ch2.taskexecutor;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
 
 
public class Main {
 
     
    public static void main(String[] args)
    {
         
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(TaskExecutorConfig.class);
        AsyncService asyncService = context.getBean(AsyncService.class);
         
        for(int i = 0; i<10; i++)
        {
            asyncService.executorAsyncTask(i);
            asyncService.executorAsyncTaskPlus(i);
        }
         
        context.close();
         
    }
     
}
```

运行



```undefined
执行异步：0
执行异步任务+1: 1
执行异步任务+1: 3
执行异步：3
执行异步任务+1: 4
执行异步任务+1: 5
执行异步：2
执行异步：4
执行异步任务+1: 6
执行异步：5
执行异步：1
执行异步任务+1: 2
执行异步任务+1: 8
执行异步：7
执行异步任务+1: 9
执行异步任务+1: 7
执行异步任务+1: 10
执行异步：6
执行异步：9
执行异步：8
```

