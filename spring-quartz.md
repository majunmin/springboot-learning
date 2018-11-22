#springBoot


##quartz
  @Schedule 
  Spring Schedule是Spring自带的任务框架，简单说他就是一个简化版本的Quartz，但是正因为它有个 与Springboot的集成非常简单，
  
  Quartz:
  springBoot2.0 以前
  
  配置文件  quartz.properties
  
  spirngBoot 2.0 以后， springBoot 集成了Quartz
```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
  ```
### quartz 配置详解
  见 quartz-learning.properties
1. 调度器属性

    第一部分有两行，分别设置调度器的实例名(instanceName) 和实例 ID (instanceId)。属性 org.quartz.scheduler.instanceName 可以是你喜欢的任何字符串。它用来在用到多个调度器区分特定的调度器实例。多个调度器通常用在集群环境中。现在的话，设置如下的一个字符串就行：org.quartz.scheduler.instanceName :DefaultQuartzScheduler实际上，这也是当你没有该属性配置时的默认值。

    调度器的第二个属性是 org.quartz.scheduler.instanceId。和 instaneName 属性一样，instanceId 属性也允许任何字符串。这个值必须是在所有调度器实例中是唯一的，尤其是在一个集群当中。假如你想 Quartz 帮你生成这个值的话，可以设置为 AUTO。如果 Quartz 框架是运行在非集群环境中，那么自动产生的值将会是 NON_CLUSTERED。假如是在集群环境下使用 Quartz，这个值将会是主机名加上当前的日期和时间。大多情况下，设置为 AUTO 即可。

2. 线程池属性

    接下来的部分是设置有关线程必要的属性值，这些线程在 Quartz 中是运行在后台担当重任的。threadCount 属性控制了多少个工作者线程被创建用来处理 Job。原则上是，要处理的 Job 越多，那么需要的工作者线程也就越多。threadCount 的数值至少为 1。Quartz 没有限定你设置工作者线程的最大值，但是在多数机器上设置该值超过100的话就会显得相当不实用了，特别是在你的 Job 执行时间较长的情况下。这项没有默认值，所以你必须为这个属性设定一个值。

    threadPriority 属性设置工作者线程的优先级。优先级别高的线程比级别低的线程更优先得到执行。threadPriority 属性的最大值是常量 java.lang.Thread.MAX_PRIORITY，等于10。最小值为常量 java.lang.Thread.MIN_PRIORITY，为1。这个属性的正常值是 Thread.NORM_PRIORITY，为5。大多情况下，把它设置为5，这也是没指定该属性的默认值。

    最后一个要设置的线程池属性是 org.quartz.threadPool.class。这个值是一个实现了 org.quartz.spi.ThreadPool 接口的类的全限名称。Quartz 自带的线程池实现类是 org.quartz.smpl.SimpleThreadPool，它能够满足大多数用户的需求。这个线程池实现具备简单的行为，并经很好的测试过。它在调度器的生命周期中提供固定大小的线程池。你能根据需求创建自己的线程池实现，如果你想要一个随需可伸缩的线程池时也许需要这么做。这个属性没有默认值，你必须为其指定值。


3. 作业存储设置

    作业存储部分的设置描述了在调度器实例的生命周期中，Job 和 Trigger 信息是如何被存储的。我们还没有谈论到作业存储和它的目的；因为对当前例子是非必的，所以我们留待以后说明。现在的话，你所要了解的就是我们存储调度器信息在内存中而不是在关系型数据库中就行了。

    把调度器信息存储在内存中非常的快也易于配置。当调度器进程一旦被终止，所有的 Job 和 Trigger 的状态就丢失了。要使 Job 存储在内存中需通过设置  org.quartz.jobStrore.class 属性为 org.quartz.simpl.RAMJobStore。假如我们不希望在 JVM 退出之后丢失调度器的状态信息的话，我们可以使用关系型数据库来存储这些信息。这需要另一个作业存储(JobStore) 实现，我们在后面将会讨论到。第五章“Cron Trigger 和其他”和第六章“作业存储和持久化”会提到你需要用到的不同类型的作业存储实现。



### quartz 持久化 与 集群
```
QRTZ_CALENDARS 以 Blob 类型存储 Quartz 的 Calendar 信息 
QRTZ_CRON_TRIGGERS 存储 Cron Trigger，包括Cron表达式和时区信息 
QRTZ_FIRED_TRIGGERS 存储与已触发的 Trigger 相关的状态信息，以及相联 Job的执行信息QRTZ_PAUSED_TRIGGER_GRPS 存储已暂停的 Trigger组的信息 
QRTZ_SCHEDULER_STATE 存储少量的有关 Scheduler 的状态信息，和别的Scheduler实例(假如是用于一个集群中) 
QRTZ_LOCKS 存储程序的悲观锁的信息(假如使用了悲观锁) 
QRTZ_JOB_DETAILS 存储每一个已配置的 Job 的详细信息 
QRTZ_JOB_LISTENERS 存储有关已配置的 JobListener的信息 
QRTZ_SIMPLE_TRIGGERS存储简单的Trigger，包括重复次数，间隔，以及已触的次数 
QRTZ_BLOG_TRIGGERS Trigger 作为 Blob 类型存储(用于 Quartz 用户用JDBC创建他们自己定制的 Trigger 类型，JobStore并不知道如何存储实例的时候) 
QRTZ_TRIGGER_LISTENERS 存储已配置的 TriggerListener的信息 
QRTZ_TRIGGERS 存储已配置的 Trigger 的信息 
```

### 监听器
[Quartz 监听器](https://blog.csdn.net/Evankaka/article/details/45498363)
扩展点 钩子函数(Hook)
Job Listener
```java
public interface JobListener {  
    /**
    * getName() 方法返回一个字符串用以说明 JobListener 的名称。对于注册为全局的监听器，getName() 主要用于记录日志，对于由特定 Job 引用的 JobListener，注册在 JobDetail 上的监听器名称必须匹配从监听器上 getName() 方法的返回值。在你看完一些例子之后就会很清楚了。
    * @return 
    */
    String getName();  
    /**
    * Scheduler 在 JobDetail 将要被执行时调用这个方法。
    * @param context
    */
    void jobToBeExecuted(JobExecutionContext context);  
    
    /**
    * Scheduler 在 JobDetail 即将被执行，但又被 TriggerListener 否决了时调用这个方法。
    * @param context
    */
    void jobExecutionVetoed(JobExecutionContext context);   
      
    
    /**
    * Scheduler 在 JobDetail 被执行之后调用这个方法。
    * @param context
    * @param jobException
    */
    void jobWasExecuted(JobExecutionContext context,   
           JobExecutionException jobException);   
    }  
```

Trigger Listener
```java
public interface TriggerListener {   
    
    String getName();   
      
    /**
    * 当与监听器相关联的 Trigger 被触发，Job 上的 execute() 方法将要被执行时，Scheduler 就调用这个方法。在全局 TriggerListener 情况下，这个方法为所有 Trigger 被调用。
    * @param trigger
    * @param context
    */
    void triggerFired(Trigger trigger, JobExecutionContext context);   
      
    /**
    * 在 Trigger 触发后，Job 将要被执行时由 Scheduler 调用这个方法。TriggerListener 给了一个选择去否决 Job 的执行。假如这个方法返回 true，这个 Job 将不会为此次 Trigger 触发而得到执行。
    * @param trigger
    * @param context
    * @return 
    */
    boolean vetoJobExecution(Trigger trigger, JobExecutidonContext context);   
      
    /**
    * Scheduler 调用这个方法是在 Trigger 错过触发时。如这个方法的 JavaDoc 所指出的，你应该关注此方法中持续时间长的逻辑：在出现许多错过触发的 Trigger 时，长逻辑会导致骨牌效应。你应当保持这上方法尽量的小。
    */
    void triggerMisfired(Trigger trigger);   
      
    /**
    * Trigger 被触发并且完成了 Job 的执行时，Scheduler 调用这个方法。这不是说这个 Trigger 将不再触发了，而仅仅是当前 Trigger 的触发(并且紧接着的 Job 执行) 结束时。这个 Trigger 也许还要在将来触发多次的。
    * @param trigger
    * @param context
    * @param triggerInstructionCode
    */
    void triggerComplete(Trigger trigger, JobExecutionContext context,int triggerInstructionCode);   
}  

```

Scheduler Listener
```java
    public interface SchedulerListener {   
        /**
        * Scheduler 在有新的 JobDetail 部署或卸载时调用这两个中的相应方法。
        * @param trigger
        */
        public void jobScheduled(Trigger trigger);   
        public void jobUnscheduled(String triggerName, String triggerGroup);  
        /**
        * 当一个 Trigger 来到了再也不会触发的状态时调用这个方法。除非这个 Job 已设置成了持久性，否则它就会从 Scheduler 中移除。
        * @param trigger
        */
        public void triggerFinalized(Trigger trigger);   
        /**
        * Scheduler 调用这个方法是发生在一个 Trigger 或 Trigger 组被暂停时。假如是 Trigger 组的话，triggerName 参数将为 null。
        * @param triggerName
        * @param triggerGroup
        */
        public void triggersPaused(String triggerName, String triggerGroup); 
        /**
        * Scheduler 调用这个方法是发生成一个 Trigger 或 Trigger 组从暂停中恢复时。假如是 Trigger 组的话，triggerName 参数将为 null。
        * @param triggerName
        * @param triggerGroup
        */
        public void triggersResumed(String triggerName,String triggerGroup);  
        /**
        * 当一个或一组 JobDetail 暂停时调用这个方法。
        * @param jobName
        * @param jobGroup
        */
        public void jobsPaused(String jobName, String jobGroup);   
        /**
        * 当一个或一组 Job 从暂停上恢复时调用这个方法。假如是一个 Job 组，jobName 参数将为 null。
        * @param jobName
        * @param jobGroup
        */
        public void jobsResumed(String jobName, String jobGroup);   
        /**
        * 在 Scheduler 的正常运行期间产生一个严重错误时调用这个方法。错误的类型会各式的，但是下面列举了一些错误例子：
         *     ·初始化 Job 类的问题
          *    ·试图去找到下一 Trigger 的问题
           *   ·JobStore 中重复的问题
            *  ·数据存储连接的问题
        *  你可以使用 SchedulerException 的 getErrorCode() 或者 getUnderlyingException() 方法或获取到特定错误的更详尽的信息
        **/
        public void schedulerError(String msg, SchedulerException cause);   
        /**
        * Scheduler 调用这个方法用来通知 SchedulerListener Scheduler 将要被关闭。
        */
        public void schedulerShutdown();   
    }  

```


    


