# AspectJ with Annotation

## 需要添加依赖
   
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

基于 Annotation 的 AOP 需要使用 `<aop:aspectj-autoproxy/>`（如果实现了接口，则使用 JDK 的动态代理生成代理对象，如果没有接口，则使用 `CGLib` 生成代理对象。如果都用 `CGLib` 生成代理对象，则用 `<aop:aspectj-autoproxy proxy-target-class="true"/>`）。


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

    <aop:aspectj-autoproxy/> <!--启用 Annotation 的 AOP-->

    <bean id="callAdvice" class="aop.CallAdvice"/> <!--Aspect-->
    <bean id="service" class="service.Service"/>
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
        throw new RuntimeException("Test exception");
    }
}
```

## CallAdvice

可以在 `Annotation` 里写 `pointcut`，如 `@Before("execution(* service.*.*(..))")`，也可以把 `pointcut` **提取出来以供复用**，如

```
@Pointcut("execution(* service.*.*(..))")
public void fooPointcut() {} 

@After("fooPointcut()") // 使用 pointcut
```


```
package aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;

@Aspect
public class CallAdvice {
    public static int steps = 1;

    @Pointcut("execution(* service.*.*(..))")
    public void fooPointcut() {}

    /**
     * 方法被执行前调用
     * @param jp
     */
    @Before("execution(* service.*.*(..))")
    public void doBefore(JoinPoint jp) {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: BEFORE  METHOD]: %s.%s()", steps++, className, methodName));
    }

    /**
     * 方法被执行后调用
     * @param jp
     */
    @After("fooPointcut()")
    public void doAfter(JoinPoint jp) {
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
    @Around("fooPointcut()")
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        String className = pjp.getTarget().getClass().getName();
        String methodName = pjp.getSignature().getName();
        int sn = steps++;

        System.out.println(String.format("[%d: METHOD BEFORE doAround]: %s.%s()", sn, className, methodName));

        long time = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        time = System.currentTimeMillis() - time;

        System.out.println(String.format("[%d: METHOD  AFTER doAround]: %s.%s() : %dms\n", sn, className, methodName, time));

        return retVal;
    }

    /**
     * 异常抛出后调用
     * @param jp
     * @param error
     */
    @AfterThrowing(pointcut="fooPointcut()", throwing="error")
    public void doThrowing(JoinPoint jp, Throwable error) {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: THROW EXCEPTION]: %s.%s()\n", steps++, className, methodName));
        System.out.println(error.getMessage());
    }
}
```

## AspectJAnnotationAOPTest

```
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import service.Service;

public class AspectJAnnotationAOPTest 
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
- 当 service 中没有抛出异常时：

```
public class Service 
{
    public void foo() 
    {
        System.out.println("service.Service.foo()");
    }
}
```

此时输出为：
    
    [1: METHOD BEFORE doAround]: service.Service.foo()
    [2: BEFORE  METHOD]: service.Service.foo()
    service.Service.foo()
    [1: METHOD  AFTER doAround]: service.Service.foo() : 36ms
    
    [3: AFTER   METHOD]: service.Service.foo()
    
    class service.Service$$EnhancerBySpringCGLIB$$3c33164f
   
- 当 service 中抛出异常时：

```
public class Service 
{
    public void foo() 
    {
        System.out.println("service.Service.foo()");
        throw new RuntimeException("Test exception");
    }
}
```
此时输出为：

    [1: METHOD BEFORE doAround]: service.Service.foo()
    [2: BEFORE  METHOD]: service.Service.foo()
    service.Service.foo()
    [3: AFTER   METHOD]: service.Service.foo()
    
    [4: THROW EXCEPTION]: service.Service.foo()
    
    Test exception
    
    java.lang.RuntimeException: Test exception
    
    	at service.Service.foo(Service.java:8)
    	at service.Service$$FastClassBySpringCGLIB$$c972b05c.invoke(<generated>)
    	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
    	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:717)
    	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveM 省略一些栈追溯

