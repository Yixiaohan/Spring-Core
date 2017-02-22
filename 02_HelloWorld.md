# Spring Hello World

## Module 的结构图

![结构图](https://github.com/Yixiaohan/Spring-Core/blob/master/assets/helloWorld.png)


## spring-beans.xml

叫做 Bean configuration file，定义 bean 的属性和其他 bean 之间的依赖，Spring 根据 bean 的定义生成 bean。


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userDao" class="com.jiangge.dao.UserDaoMySqlImpl"/>

</beans>
```




## UserDao

```
package com.jiangge.dao;

public interface UserDao {
    public void deleteUser(int id);
}
```

## UserDaoMySqlImpl


```
package com.jiangge.dao;

public class UserDaoMySqlImpl implements UserDao {
    @Override
    public void deleteUser(int id) {
        System.out.println("UserDaoMySqlImpl.deleteUser()");
    }
}
```


##UserDaoOracleImpl


```
package com.jiangge.dao;

public class UserDaoOracleImpl implements UserDao {
    @Override
    public void deleteUser(int id) {
        System.out.println("UserDaoOracleImpl.deleteUser()");
    }
}
```


## HelloWorld

```
import com.jiangge.dao.UserDao;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class HelloWorld {
    @Test
    public void test() {
        // Spring 读取 bean 的定义文件并生成 bean
        ApplicationContext context = new ClassPathXmlApplicationContext("spring-beans.xml");
        
        // 从 Spring IoC 容器里获取 bean
        UserDao userDao = context.getBean("userDao", UserDao.class);
        userDao.deleteUser(0);
    }
}
```

## 输出
UserDaoMySqlImpl.deleteUser()

## 更换 UserDao 的实现

```
<bean id="userDao" class="com.jiangge.dao.UserDaoMySqlImpl"/> 
```

修改为

```
<bean id="userDao" class="com.jiangge.dao.UserDaoOracleImpl"/>，
```
输出
`UserDaoOracleImpl.deleteUser()`


把 UserDao 的实现从 UserDaoMySqlImpl 修改为 UserDaoOracleImpl，只需要修改 XML 配置文件，不需要修改代码。

**Spring IoC + 接口** ==> **容易实现非侵入式的开发，扩展已有代码。**



