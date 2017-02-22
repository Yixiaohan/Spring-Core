# FactoryBean 

FactoryBean 是用来创建 Bean 的工厂，实现 FactoryBean 接口的类就可以用来创建其他 Bean 了，在 Bean Configuration File 里的 <bean> 里定义要生成的 Bean。

FactoryBean **命名规范**：Bean 的类名后跟着 FactoryBean，例如创建 User 的 FactoryBean 的类名应该为 `UserFactoryBean`。

如果 Bean 的创建不只是简单的 setter, constructor 注入，而是有其他的逻辑，这个时候就可以用 FactoryBean 来创建 Bean(有点像同时用了 `Factory Pattern` 和 `Builder Pattern`)。

FactoryBean 是很常见的，例如 Spring 集成 MyBatis 时用 SqlSessionFactoryBean 来创建 SqlSession。

下面的示例展示怎么实现 UserFactoryBean，用来创建 User 的对象。

## spring-beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="user" class="com.jiangge.beans.factory.UserFactoryBean">
        <property name="username" value="Alice"/>
        <property name="password" value="Passw0rd"/>
    </bean>
</beans>
```
- <bean> 和普通 Bean 的定义完全一样
- 虽然最后是要生成 User 的对象，但是 class 不是 `com.jiangge.beans.User`，而是创建它的工厂 `com.jiangge.beans.factory.UserFactoryBean`
- username, password 等都是注入到 `UserFactoryBean`
- 当 Spring 看到 UserFactoryBean 实现了 FactoryBean 接口，就会调用 `UserFactoryBean.getObject()` 来创建 User 的对象

## User

```java
package com.jiangge.beans;

public class User {
    private int id;
    private String username;
    private String password;
    private Address address;
    
    // 省略 setter and getter
}
```
## UserFactoryBean

```java
package com.jiangge.beans.factory;

import com.jiangge.beans.User;
import org.springframework.beans.factory.FactoryBean;

public class UserFactoryBean implements FactoryBean {
    private String username;
    private String password;

    @Override
    public Object getObject() throws Exception {
        User user = new User();

        // 可以有很复杂的逻辑
        user.setUsername(username.toUpperCase());
        user.setPassword(password);

        return user;
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
## FactoryBeanTest

```java
import com.jiangge.beans.User;
import com.jiangge.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class FactoryBeanTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        // 这里得到的不是 UserFactoryBean，而是它的 getObject() 方法返回的 User
        User user = context.getBean("user", User.class);
        CommonUtils.output(user);
        CommonUtils.output(user.getClass());
    }
}
```
## 输出

```
{
    "id" : 0,
    "username" : "ALICE",
    "password" : "Passw0rd",
    "address" : null
}
"com.jiangge.beans.User"
```


