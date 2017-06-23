---
layout:     post
title:      "软件工程系列-调度管理"
subtitle:   ""
date:       2017-06-02 12:00:00
author:     "JerryMinds"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 云端
    - 软件工程
    - 调度管理
---

> 非人为触发的行为的管理

### Quartz的实现原理
调度器，任务，触发器3个核心概念,并通过接口，类对这些核心概念进行描述
* JOB: 只有一个方法void execute(JobExecutionContext context)，开发者实现该接口定义运行任务，JobExecutionContext类提供了调度上下文的各种信息。Job运行时的信息保存在JobDataMap实例中.
* JobDetail: Quartz在每次执行Job时，都重新创建一个Job实例，所以它不直接接受一个Job的实例，相反它接收一个Job实现类，以便运行时通过newInstance()的反射机制实例化Job。因此需要通过__一个类来描述Job的实现类及其它相关的静态信息__，如Job名字、描述、关联监听器等信息，JobDetail承担了这一角色。

> 通过该类的构造函数可以更具体地了解它的功用：JobDetail(java.lang.String name, java.lang.String group, java.lang.Class   jobClass)，该构造函数要求指定Job的实现类，以及任务在Scheduler中的组名和Job名称。
>

* Trigger: 描述触发Job执行的时间触发规则。主要有SimpleTrigger和CronTrigger这两个子类。当仅需触发一次或者以固定时间间隔周期执行，SimpleTrigger是最适合的选择；而CronTrigger则可以通过Cron表达式定义出各种复杂时间规则的调度方案：如每早晨9:00执行，周一、周三、周五下午5:00执行等。

* Calendar:  Calendar：org.quartz.Calendar和java.util.Calendar不同，它是一些日历特定时间点的集合（可以简单地将org.quartz.Calendar看作java.util.Calendar的集合——java.util.Calendar代表一个日历时间点，无特殊说明后面的Calendar即指org.quartz.Calendar）。一个Trigger可以和多个Calendar关联，以便排除或包含某些时间点。假设，我们安排每周星期一早上10:00执行任务，但是如果碰到法定的节日，任务则不执行，这时就需要在Trigger触发机制的基础上使用Calendar进行定点排除。针对不同时间段类型，Quartz在org.quartz.impl.calendar包下提供了若干个Calendar的实现类，如__AnnualCalendar__、__MonthlyCalendar__、__WeeklyCalendar__分别针对每年、每月和每周进行定义。

* Scheduler：代表一个Quartz的独立运行容器，__Trigger__和__JobDetail__可以注册到__Scheduler__中，两者在Scheduler中拥有各自的组及名称，组及名称是Scheduler查找定位容器中某一对象的依据，Trigger的组及名称必须唯一，__JobDetail__的组和名称也必须唯一（但可以和Trigger的组和名称相同，因为它们是不同类型的）。__Scheduler__定义了多个接口方法，允许外部通过组及名称访问和控制容器中__Trigger__和__JobDetail__。

Scheduler可以将Trigger绑定到某一JobDetail中，这样当Trigger触发时，对应的Job就被执行。一个Job可以对应多个Trigger，但一个Trigger只能对应一个Job。可以通过SchedulerFactory创建一个Scheduler实例。Scheduler拥有一个SchedulerContext，它类似于ServletContext，保存着Scheduler上下文信息，Job和Trigger都可以访问__SchedulerContext__内的信息。__SchedulerContext__内部通过一个Map，以键值对的方式维护这些上下文数据，SchedulerContext为保存和获取数据提供了多个put()和getXxx()的方法。可以通过Scheduler# getContext()获取对应的SchedulerContext实例。



* ThreadPool: Scheduler使用一个线程池作为任务运行的基础设施，任务通过共享线程池中的线程提高运行效率。



###Ｑuartz所用到的表

|         表名    |    描述    |   
|    ------: |    :-------:    |   
|     QRTZ_BLOG_TRIGGERS   |    Trigger作为Blob类型存储(用于Quartz用户用JDBC创建他们自己定制的Trigger类型，JobStore 并不知道如何存储实例的时候)
    |    
|    QRTZ_CALENDARS    |    以Blob类型存储Quartz的Calendar信息
    |    
|    QRTZ_CRON_TRIGGERS    |    存储Cron Trigger，包括Cron表达式和时区信息
    | 
|    QRTZ_FIRED_TRIGGERS    |    存储与已触发的Trigger相关的状态信息，以及相联Job的执行信息
    | 
|    QRTZ_JOB_DETAILS    |    存储每一个已配置的Job的详细信息
    | 
|    QRTZ_LOCKS    |    存储程序的悲观锁的信息(假如使用了悲观锁)
    | 
|    QRTZ_PAUSED_TRIGGER_GRPS    |    存储已暂停的Trigger组的信息
    | 
|    QRTZ_SCHEDULER_STATE    |    存储少量的有关   Scheduler的状态信息，和别的 Scheduler 实例(假如是用于一个集群中)
    | 
|    QRTZ_JOB_LISTENERS    |    存储有关已配置的   JobListener的信息
    | 
|    QRTZ_SIMPLE_TRIGGERS    |    存储简单的 Trigger，包括重复次数，间隔，以及已触的次数
    | 
|    QRTZ_SIMPROP_TRIGGERS   |        | 
|    QRTZ_TRIGGERS    |    存储已配置的 Trigger的信息
    | 


### 相关的数据结构信息

* qrtz_job_details : 
|         字段名    |    描述    |   
|    ------: |    :-------:    |  
|job_name|集群中job的名字,该名字用户自己可以随意定制,无强行要求。|
|job_group|集群中job的所属组的名字,该名字用户自己随意定制,无强行要求。|
|job_class_name|集群中job实现类的完全包名,quartz就是根据这个路径到classpath找到该job类的。|
|is_durable|是否持久化,把该属性设置为1，quartz会把job持久化到数据库中|
|job_data|一个blob字段，存放持久化job对象。|


* qrtz_scheduler_state
|         字段名    |    描述    |   
|    ------: |    :-------:    |  
|instance_name|配置文件中org.quartz.scheduler.instanceId配置的名字，就会写                入该字段，如果设置为AUTO,quartz会根据物理机名和当前时间产生一个名字|
|last_checkin_time|上次检查时间|
|checkin_interval|检查间隔时间|


* qrtz_triggers

|         字段名    |    描述    |   
|    ------: |    :-------:    |  
|trigger_name|trigger的名字,该名字用户自己可以随意定制,无强行要求|
|trigger_group| trigger所属组的名字,该名字用户自己随意定制,无强行要求|
|job_name|qrtz_job_details表job_name的外键|
|job_group|qrtz_job_details表job_group的外键|
|trigger_state|当前trigger状态设置为ACQUIRED,如果设为WAITING,则job不会触发|
|trigger_cron|触发器类型,使用cron表达式|


###　线程模型
Scheduler调度线程，任务执行线程

＊　任务执行线程 ：Quartz不会在主线程(QuartzSchedulerThread)中处理用户的Job。
Quartz把线程管理的职责委托给ThreadPool，一般的设置使用SimpleThreadPool。
SimpleThreadPool创建了一定数量的WorkerThread实例来使得Job能够在线程中进行处理。
WorkerThread是定义在SimpleThreadPool类中的内部类，它实质上就是一个线程。

＊　QuartzSchedulerThread调度主线程 ：QuartzScheduler被创建时创建一个QuartzSchedulerThread实例。

###  集群源码分析

如何保证集群情况下trgger处理的信息同步？业界基本有两种方案，一种是通过ZK做分布式一致性,一种通过数据库锁同步状态

QuartzSchedulerThread包含有决定何时下一个Job将被触发的处理循环，主要逻辑在其run()方法中：
```java

public void run() {
   boolean lastAcquireFailed = false;
   while (!halted.get()) {
    ......
    int availThreadCount = qsRsrcs.getThreadPool().blockForAvailableThreads();
    if(availThreadCount > 0) { 
    ......

    //调度器在trigger队列中寻找30秒内一定数目的trigger(需要保证集群节点的系统时间一致)
    triggers = qsRsrcs.getJobStore().acquireNextTriggers(
                       now + idleWaitTime, Math.min(availThreadCount, qsRsrcs.getMaxBatchSize()), qsRsrcs.getBatchTimeWindow());
    ......
    //触发trigger
    List<TriggerFiredResult> res = qsRsrcs.getJobStore().triggersFired(triggers);
    ......
    //释放trigger
    for (int i = 0; i < triggers.size(); i++) {
        qsRsrcs.getJobStore().releaseAcquiredTrigger(triggers.get(i));
    }
   }             
}

```
由此可知，QuartzScheduler调度线程不断获取trigger，触发trigger，释放trigger。下面分析trigger的获取过程，qsRsrcs.getJobStore()返回对象是JobStore，集群环境配置如下：
```xml
<!-- JobStore 配置 -->
<prop key="org.quartz.jobStore.class">org.quartz.impl.jdbcjobstore.JobStoreTX</prop>
```

JobStoreTX继承自JobStoreSupport，而JobStoreSupport的acquireNextTriggers、triggersFired、releaseAcquiredTrigger方法负责具体trigger相关操作，都必须获得TRIGGER_ACCESS锁。核心逻辑在executeInNonManagedTXLock方法中：
```
protected <T> T executeInNonManagedTXLock(
        String lockName, 
        TransactionCallback<T> txCallback, final TransactionValidator<T> txValidator) throws JobPersistenceException {
    boolean transOwner = false;
    Connection conn = null;
    try {
        if (lockName != null) {
            if (getLockHandler().requiresConnection()) {
                conn = getNonManagedTXConnection();
            }

            //获取锁
            transOwner = getLockHandler().obtainLock(conn, lockName);
        }
        if (conn == null) {
            conn = getNonManagedTXConnection();
        }
        final T result = txCallback.execute(conn);
        try {
            commitConnection(conn);
        } catch (JobPersistenceException e) {
            rollbackConnection(conn);
            if (txValidator == null || !retryExecuteInNonManagedTXLock(lockName, new TransactionCallback<Boolean>() {
                @Override
                public Boolean execute(Connection conn) throws JobPersistenceException {
                    return txValidator.validate(conn, result);
                }
            })) {
                throw e;
            }
        }
        Long sigTime = clearAndGetSignalSchedulingChangeOnTxCompletion();
        if(sigTime != null && sigTime >= 0) {
            signalSchedulingChangeImmediately(sigTime);
        }
        return result;
    } catch (JobPersistenceException e) {
        rollbackConnection(conn);
        throw e;
    } catch (RuntimeException e) {
        rollbackConnection(conn);
        throw new JobPersistenceException("Unexpected runtime exception: "
                + e.getMessage(), e);
    } finally {
        try {
            releaseLock(lockName, transOwner);    //释放锁
        } finally {
            cleanupConnection(conn);
        }
    }
}
```

由上代码可知Quartz集群基于数据库锁的同步操作流程如下图所示：
![img](/img/in-post/post-sofeware/quartz流程图.png)

一个调度器实例在执行涉及到分布式问题的数据库操作前，首先要获取QUARTZ_LOCKS表中对应的行级锁，获取锁后即可执行其他表中的数据库操作，随着操作事务的提交，行级锁被释放，供其他调度实例获取。集群中的每一个调度器实例都遵循这样一种严格的操作规程。

getLockHandler()方法返回的对象类型是Semaphore，获取锁和释放锁的具体逻辑由该对象维护

``` java
public interface Semaphore {

     boolean obtainLock(Connection conn, String lockName) throws LockException;

     void releaseLock(String lockName) throws LockException;

     boolean requiresConnection();
}

```

该接口的实现类完成具体操作锁的逻辑，在JobStoreSupport的初始化方法中注入的Semaphore具体类型是StdRowLockSemaphore

``` java
setLockHandler(new StdRowLockSemaphore(getTablePrefix(), getInstanceName(), getSelectWithLockSQL()));
```

StdRowLockSemaphore的源码如下所示：

``` java
public class StdRowLockSemaphore extends DBSemaphore {
//锁定SQL语句
public static final String SELECT_FOR_LOCK = "SELECT * FROM "
        + TABLE_PREFIX_SUBST + TABLE_LOCKS + " WHERE " + COL_LOCK_NAME
        + " = ? FOR UPDATE";
public static final String INSERT_LOCK = "INSERT INTO " + TABLE_PREFIX_SUBST 
        + TABLE_LOCKS + "(" + COL_SCHEDULER_NAME + ", " 
        + COL_LOCK_NAME + ") VALUES (" + SCHED_NAME_SUBST + ", ?)"; 
//指定锁定SQL
protected void executeSQL(Connection conn, String lockName, String expandedSQL) throws LockException {
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        ps = conn.prepareStatement(expandedSQL);
        ps.setString(1, lockName);
        ......
        rs = ps.executeQuery();
        if (!rs.next()) {
            throw new SQLException(Util.rtp(
                "No row exists in table " + TABLE_PREFIX_SUBST +
                TABLE_LOCKS + " for lock named: " + lockName, getTablePrefix()));
        }
    } catch (SQLException sqle) {

    } finally {
      ...... //release resources
    }
  }
}
//获取QRTZ_LOCKS行级锁
public boolean obtainLock(Connection conn, String lockName) throws LockException {
    lockName = lockName.intern();

    if (!isLockOwner(conn, lockName)) {
        executeSQL(conn, lockName, expandedSQL);

        getThreadLocks().add(lockName);
    }
    return true;
}
//释放QRTZ_LOCKS行级锁
public void releaseLock(Connection conn, String lockName) {
    lockName = lockName.intern();

    if (isLockOwner(conn, lockName)) {
        getThreadLocks().remove(lockName);
    }
    ......
}
```
至此，总结一下Quartz集群同步机制：每当要进行与某种业务相关的数据库操作时，先去QRTZ_LOCKS表中查询操作相关的业务对象所需要的锁，在select语句之后加for update来实现。例如，TRIGGER_ACCESS表示对任务触发器相关的信息进行修改、删除操作时所需要获得的锁。这时，执行查询这个表数据的SQL形如：

```sql
select * from QRTZ_LOCKS t where t.lock_name='TRIGGER_ACCESS' for update

```

当一个线程使用上述的SQL对表中的数据执行查询操作时，若查询结果中包含相关的行，数据库就对该行进行ROW LOCK；若此时，另外一个线程使用相同的SQL对表的数据进行查询，由于查询出的数据行已经被数据库锁住了，此时这个线程就只能等待，直到拥有该行锁的线程完成了相关的业务操作，执行了commit动作后，数据库才会释放了相关行的锁，这个线程才能继续执行。

通过这样的机制，在集群环境下，结合悲观锁的机制就可以防止一个线程对数据库数据的操作的结果被另外一个线程所覆盖，从而可以避免一些难以觉察的错误发生。当然，达到这种效果的前提是需要把Connection设置为手动提交，即autoCommit为false。


### 调度模块剖析
#### quartz的不足
- 问题1：调用API的方式操作任务,不人性化
- 问题2：持久化业务QuartzJobBean到底层数据中，系统侵入性大
- 问题3：调度逻辑和QuartzJobBean耦合在同一个项目中，当任务过多时，影响调度性能会受业务系统影响


#### RemoteHttpJobBean
常规Quartz的开发，任务逻辑一般维护在QuartzJobBean中，耦合很严重。XXL-JOB中“调度模块”和“任务模块”完全解耦，调度模块中的所有调度任务使用同一个QuartzJobBean，即RemoteHttpJobBean。不同的调度任务将各自参数维护在各自扩展表数据中，当触发RemoteHttpJobBean执行时，将会解析不同的任务参数发起远程调用，调用各自的远程执行器服务。

这种调用模型类似RPC调用，RemoteHttpJobBean提供调用代理的功能，而执行器提供远程服务的功能


#### 调度中心HA（集群）
基于Quartz的集群方案，数据库选用Mysql；集群分布式并发环境中使用QUARTZ定时任务调度，会在各个节点会上报任务，存到数据库中，执行时会从数据库中取出触发器来执行，如果触发器的名称和执行时间相同，则只有一个节点去执行此任务。

```
# for cluster
org.quartz.jobStore.tablePrefix = XXL_JOB_QRTZ_
org.quartz.scheduler.instanceId: AUTO
org.quartz.jobStore.class: org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.isClustered: true
org.quartz.jobStore.clusterCheckinInterval: 1000
```

#### 调度线程池
默认线程池中线程的数量为10个，避免单线程因阻塞而引起任务调度延迟。
```
org.quartz.threadPool.class: org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount: 10
org.quartz.threadPool.threadPriority: 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread: true
```

XXL-JOB系统中业务逻辑在远程执行器执行，调度中心每次调度仅仅负责一次调度请求，执行器会将请求存入执行队列并且立即响应调度中心；相比直接在quartz的QuartzJobBean中执行业务逻辑，差别就像大象和羽毛；

XXL-JOB调度中心中每个JOB逻辑非常 “轻”，单个JOB一次运行平均耗时基本在 "100ms" 之内（基本是网络开销）；因此，可以保证使用有限的线程支撑大量的JOB并发运行；上面配置的10个线程至少可以支撑100个JOB正常运行；

####  @DisallowConcurrentExecution

XXL-JOB调度模块的“调度中心”默认不使用该注解，即默认开启并行机制，因为RemoteHttpJobBean为公共QuartzJobBean，这样在多线程调度的情况下，调度模块被阻塞的几率很低，大大提高了调度系统的承载量。

XXL-JOB的每个调度任务虽然在调度模块是并行调度执行的，但是任务调度传递到任务模块的“执行器”确实串行执行的，同时支持任务终止。


#### misfire
错过了触发时间，处理规则。 可能原因：服务重启；调度线程被QuartzJobBean阻塞，线程被耗尽；某个任务启用了@DisallowConcurrentExecution，上次调度持续阻塞，下次调度被错过；

quartz.properties中关于misfire的阀值配置如下，单位毫秒：
```
org.quartz.jobStore.misfireThreshold: 60000
```

Misfire规则： 
* withMisfireHandlingInstructionDoNothing：不触发立即执行，等待下次调度； 
* withMisfireHandlingInstructionIgnoreMisfires：以错过的第一个频率时间立刻开始执行； 
* withMisfireHandlingInstructionFireAndProceed：以当前时间为触发频率立刻触发一次执行；

XXL-JOB默认misfire规则为：withMisfireHandlingInstructionDoNothing

``` java
CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(jobInfo.getJobCron()).withMisfireHandlingInstructionDoNothing();
CronTrigger cronTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).withSchedule(cronScheduleBuilder).build();
```

#### 日志回调服务
调度模块的“调度中心”作为Web服务部署时，一方面承担调度中心功能，另一方面页为执行器提供API服务。

调度中心提供的"日志回调服务API服务"代码位置如下：

```
xxl-job-admin#com.xxl.job.admin.controller.JobApiController.callback
```
“执行器”在接收到任务执行请求后，执行任务，在执行结束之后会将执行结果回调通知“调度中心”，代码位置为：



####　任务HA（Failover）
执行器如若集群部署，调度中心将会感知到在线的所有执行器，如“127.0.0.1:9997, 127.0.0.1:9998, 127.0.0.1:9999”。

当任务"路由策略"选择"故障转移(FAILOVER)"时，当调度中心每次发起调度请求时，会按照顺序对执行器发出心跳检测请求，第一个检测为存活状态的执行器将会被选定并发送调度请求。 
调度成功后，可在日志监控界面查看“调度备注”，如下； 
```
触发调度:
address: 127.0.0.1:9999
code:200
msg:null
```

“调度备注”可以看出本地调度运行轨迹，执行器的"注册方式"、"地址列表"和任务的"路由策略"。"故障转移(FAILOVER)"路由策略下，调度中心首先对第一个地址进行心跳检测，心跳失败因此自动跳过，第二个依然心跳检测失败…… 直至心跳检测第三个地址“127.0.0.1:9999”成功，选定为“目标执行器”；然后对“目标执行器”发送调度请求，调度流程结束，等待执行器回调执行结果。




###  Quartz的升级版XXL
|         表名    |    描述    |   
|    ------: |    :-------:    |   
|     QRTZ_TRIGGER_GROUP   |    执行器信息表，维护任务执行器信息；app_name为所在应用名称
    |    
|    QRTZ_TRIGGER_INFO    |    调度扩展信息表： 用于保存JOB调度任务的扩展信息，如任务分组、任务名、机器地址、执行器、执行入参和报警邮件等等，核心表
    |    
|    QRTZ_TRIGGER_LOG    |    调度日志表： 用于保存JOB任务调度的历史信息，如调度结果、执行结果、调度入参、调度机器和执行器等等；
    | 
|    QRTZ_TRIGGER_LOGGLUE    |    任务GLUE日志：用于保存GLUE更新历史，用于支持GLUE的版本回溯功能；
    | 
|    QRTZ_TRIGGER_REGISTRY    |    执行器注册表，维护在线的执行器和调度中心机器地址信息；
    | 


