---
title: Springboot集成Quartz定时任务管理器
date: 2024-06-06
categories:
  - 学习笔记
  - 01-Spring学习笔记
tags:
  - quartz
  - 定时任务
---

# Quartz简介

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/20240606/Snipaste_2024-06-06_15-20-56.7493xv9cdao0.jpg)

**Job** 表示一个工作，要执行的具体<font color="red">**业务内容**</font>。

**JobDetail** 表示一个具体的可执行的调度程序，Job 是这个可执行程调度程序所要执行的内容，另外 JobDetail 还包含了这个任务调度的方案和策略。

**Trigger** 代表一个调度参数的配置，什么时候去调。

**Scheduler** 代表一个调度容器，一个调度容器中可以注册多个 JobDetail 和 Trigger。当 Trigger 与 JobDetail 组合，就可以被 Scheduler 容器调度了。

# 案例

## 背景

系统中有一张自定义的定时任务表，需要实现动态的添加、修改、删除、启停定时任务等功能，定时任务里包含了业务需要执行的按设置周期执行的代码

由于不想使用Quartz的数据存储功能，所以下面实现里直接使用了这张自定义的表，以及使用了Quartz的<font color=red>**任务调度**</font>和<font color=red>**触发**</font>功能

## 实现步骤

### 1. 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
    <version>2.7.18</version>
</dependency>
```

### 2. 添加配置类

```java
@Configuration
public class ScheduleQuartzConfig {
    @Bean
    public Scheduler scheduler() throws SchedulerException {
        Scheduler scheduler = schedulerFactoryBean().getScheduler();
        return scheduler;
    }
    @Bean
    public SchedulerFactoryBean schedulerFactoryBean() {
        Properties properties = new Properties();
        properties.setProperty("org.quartz.threadPool.threadCount", "10");
        properties.setProperty("org.quartz.threadPool.threadNamePrefix","quartz_worker");
        SchedulerFactoryBean factory = new SchedulerFactoryBean();
        factory.setSchedulerName("QUARTZ_SCHEDULER");
        factory.setQuartzProperties(properties);
        return factory;
    }
}
```



### 3. 编写Job类

该类就是将来要定时执行的业务代码，具体代码路径根据实际情况规划即可，重点是继承 `QuartzJobBean`，重写 `executeInternal` 方法

```java
//锁定机制，以确保在同一时间只有一个任务实例运行
@DisallowConcurrentExecution
public class MyJob extends QuartzJobBean
    private final static Logger logger = LoggerFactory.getLogger(PushPvImitateDataJob.class);

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        //业务代码
        logger.info("开始执行业务代码了。。。");
		try {
            Thread.sleep(6000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }	
        logger.info("业务代码执行完成了！！！！");
    }
}
```

### 4. 编写定时任务表的实体类

ScheduleLog.java

```java
@Data
public class ScheduleLog {
    private Integer id;
    private String name; //任务名称
    private String jobClassName; //任务的实现类 如：xxx.xxx.xx.MyJob
    private String cronExpression; //触发时机表达式，比如每5秒执行一次 0/5 * * * * ?
    private Integer status; //状态:0 启动 1 禁用
    private String remark;
    private Long createTime;
    private Long updateTime;
    private Long lastTime; //上一次执行时间
    private Long nextTime; //下一次执行时间
    private String lastTimeText;
    private String nextTimeText;
}
```

### 5. 创建管理定时任务的工具类

```java
@Component
public class ScheduleQuartzManage {
    private static final Logger logger = LoggerFactory.getLogger(ScheduleQuartzManage.class);
    /**
     * 创建定时任务 定时任务创建之后默认启动状态
     *
     * @param scheduler:    调度器
     * @param scheduleLog: 报告订阅对象
     * @return: void
     **/
    public void createScheduleJob(Scheduler scheduler, ScheduleLog scheduleLog) throws SchedulerException {

        //获取到定时任务的执行类  必须是类的绝对路径名称
        //定时任务类需要是job类的具体实现 QuartzJobBean是job的抽象类。
//        Class<? extends Job> jobClass = PushPvImitateDataJob.class;
        Class<? extends Job> jobClass = null;
        try {
            jobClass = (Class<? extends Job>) Class.forName(scheduleLog.getJobClassName());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        // 构建定时任务信息
        JobDetail jobDetail = JobBuilder.newJob(jobClass)
                .withIdentity(scheduleLog.getId().toString(),"jobGroup")
                .usingJobData("id",scheduleLog.getId())
                .build();
        // 设置定时任务执行方式
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(scheduleLog.getCronExpression());
        // 构建触发器trigger
        // 如果已经有下一次时间，就设置为下一次时间为触发时间
        CronTrigger trigger;
        if (!Objects.isNull(scheduleLog.getNextTime())) {

            Date date = new Date(scheduleLog.getNextTime());
            trigger = TriggerBuilder.newTrigger()
                    .startAt(date)
                    .withIdentity(scheduleLog.getId().toString(), "jobGroup")
                    .withSchedule(scheduleBuilder)
                    .build();
        } else {
            trigger = TriggerBuilder.newTrigger()
                    .withIdentity(scheduleLog.getId().toString(),"jobGroup")
                    .withSchedule(scheduleBuilder)
                    .build();
        }

        scheduler.scheduleJob(jobDetail, trigger);

        // 设置下次执行时间
        //long nextTime = trigger.getNextFireTime().getTime();
        //LocalDateTime nextTime = DateUtil.dateToLocalDate(trigger.getNextFireTime());
        //logger.info("下次一执行时间:{}",DateUtil.formatDateTime(new Date(nextTime)));
        //scheduleLog.setNextTime(nextTime);


    }

    /**
     * 根据任务名称暂停定时任务
     *
     * @param scheduler 调度器
     * @param jobName  定时任务名称（这里直接用ReportSubscribePO的Id）
     * @throws SchedulerException
     */
    public void pauseScheduleJob(Scheduler scheduler, String jobName) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName, "jobGroup");
        scheduler.pauseJob(jobKey);

    }

    /**
     * 根据任务名称恢复定时任务
     *
     * @param scheduler 调度器
     * @param scheduleLog  定时任务名称（这里直接用ReportSubscribePO的Id）
     * @throws SchedulerException
     */
    public void resumeScheduleJob(Scheduler scheduler, ScheduleLog scheduleLog) throws SchedulerException {

        // 判断当前任务是否在调度中
        Set<JobKey> jobKeys = scheduler.getJobKeys(GroupMatcher.groupEquals("jobGroup"));
        List<JobKey> thisNameJobs = jobKeys.stream().filter(jobKey -> Objects.equals(scheduleLog.getId().toString(), jobKey.getName())).collect(Collectors.toList());

        if (thisNameJobs.size() > 0){
            JobKey jobKey = JobKey.jobKey(scheduleLog.getId().toString(), "jobGroup");
            scheduler.resumeJob(jobKey);
            // 下一次执行时间设置回去
            Trigger trigger = scheduler.getTrigger(TriggerKey.triggerKey(scheduleLog.getId().toString(), "jobGroup"));


            long nextTime = trigger.getNextFireTime().getTime();
            scheduleLog.setNextTime(nextTime);
        }else {
            createScheduleJob(scheduler, scheduleLog);
        }

    }

    /**
     * 更新定时任务
     *
     * @param scheduler    调度器
     * @param scheduleLog 报告订阅对象
     * @throws SchedulerException
     */
    public void updateScheduleJob(Scheduler scheduler, ScheduleLog scheduleLog) throws SchedulerException {

        //获取到对应任务的触发器
        TriggerKey triggerKey = TriggerKey.triggerKey(scheduleLog.getId().toString(),"jobGroup");
        //设置定时任务执行方式
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(scheduleLog.getCronExpression());
        //重新构建任务的触发器trigger
        CronTrigger trigger = (CronTrigger) scheduler.getTrigger(triggerKey);
        if (trigger == null){
            return;
        }
        //  trigger = trigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(scheduleBuilder).build();
        trigger = TriggerBuilder.newTrigger().startNow()
                .withIdentity(scheduleLog.getId().toString(), "jobGroup").withSchedule(scheduleBuilder).build();

        //重置对应的job
        scheduler.rescheduleJob(triggerKey, trigger);
        scheduleLog.setNextTime(trigger.getNextFireTime().getTime());
    }

    /**
     * 根据定时任务名称从调度器当中删除定时任务
     *
     * @param scheduler 调度器
     * @param jobName  定时任务名称 （这里直接用 ScheduleLog 的Id）
     * @throws SchedulerException
     */
    public void deleteScheduleJob(Scheduler scheduler, String jobName) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName,"jobGroup");
        scheduler.deleteJob(jobKey);
    }
}
```

### 6. 项目启动时做任务初始化

这里我直接让service实现了**CommandLineRunner**，然后在run()方法中，将初始化逻辑写入进来，让数据库中的持久化的任务全部添加进内存中。

```java
@Service
public class ScheduleLogServiceImpl implements ScheduleLogService, CommandLineRunner
    private final static Logger logger = LoggerFactory.getLogger(ScheduleLogServiceImpl.class);
    @Autowired
    private ScheduleLogDao scheduleLogDao;
    @Autowired
    private ScheduleQuartzManage scheduleQuartzManage;
    @Autowired
    private Scheduler scheduler; //这里的scheduler来源于配置

    @Override
    public void run(String... args) throws Exception {
        // 初始化所有的已经启用的订阅
        List<ScheduleLog> enableSubs = getEnableScheduleList();
        logger.info("需要初始化的任务个数:{}", enableSubs.size());
        for (ScheduleLog sub : enableSubs) {
            try {
                logger.info("开始初始化订阅任务,任务name:{}", sub.getName());
                scheduleQuartzManage.createScheduleJob(scheduler, sub);
            } catch (Exception e) {
                logger.error("启动时初始化订阅任务失败:{}", e.getMessage());
            }
        }
    }
	@Override
    public List<ScheduleLog> getEnableScheduleList() {
        HashMap<String, Object> param = new HashMap<>();
        param.put("status",1);
        List<ScheduleLog> list = scheduleLogDao.getList(param);
        return list;
    }
	@Override
    public boolean add(Map data) {
        data.put("createTime", DateUtil.currentSeconds());
        Integer add = scheduleLogDao.add(data);
        if(add > 0){
            //创建调度器
            data.put("id",add);
            ScheduleLog scheduleLog = BeanUtil.mapToBean(data, ScheduleLog.class,false,null);
            if(scheduleLog.getStatus() < 1){
                return true;
            }
            try {
                scheduleQuartzManage.createScheduleJob(scheduler, scheduleLog);
            } catch (SchedulerException e) {
                e.printStackTrace();
            }
            return true;
        }
        return false;
    }

    @Override
    public boolean del(Integer id) {
        Integer del = scheduleLogDao.del(id);
        if(del > 0){
            try {
                scheduleQuartzManage.deleteScheduleJob(scheduler,String.valueOf(id));
            } catch (SchedulerException e) {
                e.printStackTrace();
            }
            return true;
        }
        return false;
    }

    @Override
    public boolean edit(Map data) {
        data.put("updateTime",DateUtil.currentSeconds());
        Integer save = scheduleLogDao.save(data);
        if(save > 0){
            try {
                ScheduleLog scheduleLog = getDetail((Integer) data.get("id"));
                if(data.get("status").equals(0)){
                    //关闭
                    scheduleQuartzManage.deleteScheduleJob(scheduler,data.get("id").toString());
                }else{
                    //开启
                    scheduleQuartzManage.deleteScheduleJob(scheduler,data.get("id").toString());
                    scheduleQuartzManage.createScheduleJob(scheduler,scheduleLog);
                }
            } catch (SchedulerException e) {
                e.printStackTrace();
            }
            return true;
        }
        return false;
    }
}
```

至此基本就实现了定时任务的管理了，controller 里的内容包含了对定时任务进行管理的接口，就不写了

# 总结

1. 同一个任务是否可以并行执行，可参考第3步设置
2. 每次项目重新部署后，自动加载数据库中的定时任务
3. Quartz在发生异常时会重试一次，注意异常处理，可在第3步中处理

参考链接：https://juejin.cn/post/7054762566035193869#heading-7