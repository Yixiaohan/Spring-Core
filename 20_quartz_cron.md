# Quartz 实现定时任务

定义一个定时任务，需要 `Task`, `Job`, `Trigger`, `Scheduler` 4 个类.

其中 Task 是我们**自定义的类**，是任务的实现逻辑，另外 3 个类则是由 Spring 和 Quartz 提供的，在 Bean 的配置文件里配置即可。

## 1. pom.xml

Quartz 需要 spring-context-support，spring-tx 和 quartz 的依赖。

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>4.1.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.1.1.RELEASE</version>
</dependency>

<!-- Quartz framework -->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.2.1</version>
</dependency>
```

## 2. spring-beans.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--1. Task: 我们自定义的 Task 类-->
    <bean id="fooTask" class="com.xtuer.scheduler.FooTask"/>

    <!--2. Job-->
    <bean id="jobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
        <property name="targetObject" ref="fooTask"/> <!--自定义的 Task 类-->
        <property name="targetMethod" value="execute"/> <!--Task 的方法名-->
        
        <!--false表示等上一个任务执行完后再开启新的任务-->
        <property name="concurrent" value="false"/>
    </bean>

    <!--3. Trigger: run every 5 seconds-->
    <bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="jobDetail"/>
        <property name="cronExpression" value="0/5 * * * * ?"/>
    </bean>

    <!--4. Scheduler-->
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
            <list>
                <ref bean="cronTrigger"/>
            </list>
        </property>
    </bean>
</beans>
```
## 3. Task 类

Task 类就是一个普通的类，不需要继承或者实现其它任何的类或者接口。

```
package com.xtuer.scheduler;

import java.text.SimpleDateFormat;
import java.util.Date;

public class FooTask {
    private SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    public void execute() {
        System.out.println(format.format(new Date()));
    }
}
```

## 4. 测试


```
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        new ClassPathXmlApplicationContext("spring-beans.xml");
    }
}
```

## 输出

```
2015-06-19 08:16:10
2015-06-19 08:16:15
2015-06-19 08:16:20
2015-06-19 08:16:25
2015-06-19 08:16:30
...省略其他输出
```

## Cron 的语法
   
   
``` 
<!--3. Trigger: run every 5 seconds-->
    <bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="jobDetail"/>
        <property name="cronExpression" value="0/5 * * * * ?"/>
    </bean>
```
`Quartz-Cron` 的语法类似于 Linux 中的 Cron 的语法，但有些的区别，例如 ? 表达式中同时有日和周时，它们不能相同，所以用 ? 来区别开，不能同时为 * 等。 
```
秒 分 时 日 月 周 年
```

表达式 | 意义
------- | -------
0 0 12 * * ? | 每天中午12点触发
0 15 10 ? * * | 每天上午10:15触发
0 15 10 * * ? | 每天上午10:15触发
0 15 10 * * ? * | 每天上午10:15触发
0 15 10 * * ? 2005 | 2005年的每天上午10:15触发
0 * 14 * * ? | 在每天下午2点到下午2:59期间的每1分钟触发
0 0/5 14 * * ? | 在每天下午2点到下午2:55期间的每5分钟触发
0 0/5 14,18 * * ? | 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
0 0-5 14 * * ? | 在每天下午2点到下午2:05期间的每1分钟触发
0 10,44 14 ? 3 WED | 每年三月的星期三的下午2:10和2:44触发
0 15 10 ? * MON-FRI | 周一至周五的上午10:15触发
0 15 10 15 * ? | 每月15日上午10:15触发
0 15 10 L * ? | 每月最后一日的上午10:15触发
0 15 10 ? * 6L | 每月的最后一个星期五上午10:15触发
0 15 10 ? * 6L 2002-2005 | 2002年至2005年的每月的最后一个星期五上午10:15触发
0 15 10 ? * 6#3 | 每月的第三个星期五上午10:15触发
0 6 * * * | 每天早上6点
0 */2 * * * | 每两个小时
0 23-7/2，8 * * * | 晚上11点到早上8点之间每两个小时，早上八点
0 11 4 * 1-3 | 每个月的4号和每个礼拜的礼拜一到礼拜三的早上11点
0 4 1 1 * | 1月1日早上4点



## cronExpression中问号(?)的解释

http://blog.csdn.net/chh_jiang/article/details/4603529

1. 如官方文档解释的那样，`问号(?)`的作用是指明该字段‘**没有特定的值**’；

2. 星号(*)和其它值，比如数字，都是给该字段指明特定的值，只不过用星号(*)代表所有可能值；

3. `cronExpression` 对日期和星期字段的处理规则是它们必须互斥，即只能且必须有一个字段有特定的值，另一个字段必须是‘没有特定的值’；

4. `问号(?)`就是用来对日期和星期字段做**互斥**的。
 
#### 基于以上结论就可以解释下列情况：


- 当星期和日期都为`*`或数字时，报错
 `Support for specifying both a day-of-week AND a day-of-month parameter is not implemented.`

即两个字段不能都指明的特定的值，必须互斥。这里的`*`和数字是一样的，如果都指明特定的数字，也是报一样的错。

- 当星期和日期都为`?`时，报错 `'?' can only be specfied for Day-of-Month -OR- Day-of-Week.`

即两个字段不能都‘**没有特定的值**’。
 
这个是spring的实现，没有什么道理，Spring就是**规定**这两个字段必须这样互斥的设置。

**这与UNIX的crontab设置不一样**，crontab的规则是日期和星期中只要满足一个就触发，所以不存在互斥的问题。


