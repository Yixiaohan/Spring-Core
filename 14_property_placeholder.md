# Property-Placeholder

在 `.properties` 文件里定义 `key | value`，然后在 Bean Configuration File 里用 `${key}` 引用 key 对应的值。

Bean Configuration File 里使用 `<context:property-placeholder>` 引入 `properties` 文件。

## 1. user.properties

```
username = Alice
password = Passw0rd
```
## 2. address.propertes

```
country = China
province = BeiJing
```
## 3. spring-beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:address.properties,classpath:user.properties" /> 


    <bean id="address" class="com.jiangge.beans.Address">
        <property name="country" value="${country}"/>
        <property name="province" value="${province}"/>
    </bean>

    <bean id="user" class="com.jiangge.beans.User">
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
        <property name="address" ref="address"/>
    </bean>
</beans>
```

## 4. PropertyPlaceholderTest

```java
import com.jiangge.beans.User;
import com.jiangge.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class PropertyPlaceholderTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        User user = context.getBean("user", User.class);
        CommonUtils.output(user);
    }
}
```

## 5. 输出

```json
{
    "id" : 0,
    "username" : "Alice",
    "password" : "Passw0rd",
    "address" : {
        "id" : 0,
        "country" : "China",
        "province" : "BeiJing",
        "street" : null
    }
}
```


