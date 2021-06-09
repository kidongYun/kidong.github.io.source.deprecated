---
layout: post
title:  "Spring Schedule"
date:   2021-06-08 00:0054 +0900
categories: java
---

## @EnableScheduling

If you want to use @Scheduled annotation then EnableScheduling annotation should be written at the entry point like Application.java file.

## @Scheduled

This annotation has three options for setting period.

```
cron, fixedDelay, fixedRate
```

you can use these options like the below code.

## cron

```java

@Component
public class Scheduler {
   @Scheduled(cron = "0 * 9 * * ?")
   public void cronJobSch() {
       // Do Something
   }
}

```

cron option can specify date and time using 6th digits.

```
second(0-59)   minute(0-59)   hour(0-23)    day(1-31)   month(1-12)     week(0-7)
```

If you don't want to set at the specfic digit, then just use asterisk.


## fixedDelay

```java

@Scheduled(fixedDelay = 1000) 
public void scheduleFixedRateTask() {
    // Do Something
}

```

## fixedRate

```java

@Scheduled(fixedRate = 1000)
public void scheduleFixedRateTask() {
  // Do something
}

```

## The problem of Thread pool

If you don't do any setting for @Scheduled annotation then, it will be operated at only one thread. so They can't operate together at same time.
We should implement Schedulingconfigurer to solve this.

```java

@Configuration
public class SchedulerConfig implements SchedulingConfigurer {
    private final int POOL_SIZE = 10;

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();

        threadPoolTaskScheduler.setPoolSize(POOL_SIZE);
        threadPoolTaskScheduler.setThreadNamePrefix("my-scheduled-task-pool-");
        threadPoolTaskScheduler.initialize();

        scheduledTaskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
    }
}

```