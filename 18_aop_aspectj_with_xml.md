# AspectJ with XML -- 使用 XML 配置 AOP

## 添加 aop 相关依赖

```
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aop</artifactId>
       <version>4.1.1.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.8.1</version>
   </dependency>
```
## spring-beans.xml

pointcut 表示需要织入切面的地方

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="service" class="service.Service"/>
    <bean id="callAdvice" class="aop.CallAdvice"/>

    <!--使用 XML 配置 AOP-->
    <aop:config>
        <aop:aspect ref="callAdvice">
            <aop:pointcut id="methodPointCut" expression="execution(* service.*.*(..))"/>
            <aop:before pointcut-ref="methodPointCut" method="doBefore"/>
            
            <!--<aop:after  pointcut-ref="methodPointCut" method="doAfter"/>-->
            <!--<aop:around pointcut-ref="methodPointCut" method="doAround"/>-->
            <!--<aop:after-throwing pointcut-ref="methodPointCut"
                     method="doThrowing" throwing="ex"/>-->
        </aop:aspect>
    </aop:config>
</beans>
```
## Service

```
package service;

public class Service 
{
    public void foo() 
    {
        System.out.println("service.Service.foo()");
    }
}
```
## CallAdvide

```
package aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;

public class CallAdvice 
{
    public static int steps = 1;

    /**
     * 方法被执行前调用
     * @param jp
     */
    public void doBefore(JoinPoint jp) 
    {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: BEFORE  METHOD]: %s.%s()", steps++, className, methodName));
    }

    /**
     * 方法被执行后调用
     * @param jp
     */
    public void doAfter(JoinPoint jp) 
    {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: AFTER   METHOD]: %s.%s()\n", steps++, className, methodName));
    }

    /**
     * 方法被执行时调用
     * @param pjp
     * @return
     * @throws Throwable
     */
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable 
    {
        String className = pjp.getTarget().getClass().getName();
        String methodName = pjp.getSignature().getName();
        int sn = steps++;

        System.out.println(String.format("[%d: METHOD BEFORE]: %s.%s()", sn, className, methodName));

        long time = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        time = System.currentTimeMillis() - time;

        System.out.println(String.format("[%d: METHOD  AFTER]: %s.%s() : %dms\n", sn, className, methodName, time));

        return retVal;
    }

    /**
     * 异常抛出后调用
     * @param jp
     * @param ex
     */
    public void doThrowing(JoinPoint jp, Throwable ex) 
    {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: THROW EXCEPTION]: %s.%s()\n", steps++, className, methodName));
        System.out.println(ex.getMessage());
    }
}
```

## 测试 AspectJXmlAOPTest

```
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import service.Service;

public class AspectJXmlAOPTest 
{
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() 
    {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() 
    {
        // service 是 Spring 生成的代理类的对象
        Service service = context.getBean("service", Service.class);
        service.foo();
        System.out.println(service.getClass());
    }
}
```

## 输出

```
    [1: BEFORE  METHOD]: service.Service.foo()
    service.Service.foo()
    class service.Service$$EnhancerBySpringCGLIB$$ed356710
```


