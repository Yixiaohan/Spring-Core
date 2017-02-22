# 多个 Bean 配置文件

把 Spring 的 Bean Configuration File 根据模块分散到不同的文件里，便于管理，然后使用 `<import>` 把它们组织在一起。例如：

Bean | Bean Configuration File
------- | -------
user | spring-beans.xml
address | spring-beans-1.xml
customer | spring-beans-2.xml

在 `spring-beans.xml` 里 import `spring-beans-1.xml` and `spring-beans-2.xml`，然后 `Spring Context` 加载 `spring-beans.xml`。

Bean 在配置文件中定义的顺序任意，其实最后就是把所有的配置文件**合成一个大的配置文件**，然后在解析创建对象，所以就像在单一的配置文件里定义 Bean 一样顺序不重要。

## spring-beans-1.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="address" class="com.jiangge.beans.Address">
        <property name="country" value="China"/>
    </bean>
</beans>
```
## spring-beans-2.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="customer" class="com.jiangge.beans.Customer">
        <constructor-arg value="Alice"/>
        <constructor-arg value="40"/>
        <constructor-arg value="1234567"/>
    </bean>
</beans>
```
## spring-beans.xml


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="classpath:spring-beans-1.xml"/>
    <import resource="classpath:spring-beans-2.xml"/>

    <bean id="user" class="com.jiangge.beans.User">
        <property name="username" value="Alice"/>
        <property name="address" ref="address"/>
    </bean>
</beans>
```

- import 用于引入另外一个配置文件
- resource 为配置文件的路径
- classpath: 指在 classpath 下搜索配置文件
- classpath*: 指在 classpath 下 和 jar 包中搜索配置文件


## 测试

```java
import com.jiangge.beans.Address;
import com.jiangge.beans.Customer;
import com.jiangge.beans.User;
import com.jiangge.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MultipleBeanConfigurationFilesTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void testGetCustomer() {
        Customer customer = context.getBean("customer", Customer.class);
        CommonUtils.output(customer);
    }

    @Test
    public void testGetUser() {
        User user = context.getBean("user", User.class);
        CommonUtils.output(user);
    }

    @Test
    public void testGetAddress() {
        Address address = context.getBean("address", Address.class);
        CommonUtils.output(address);
    }
}
```

## 输出

```json
{
    "id" : 0,
    "country" : "China",
    "province" : null,
    "street" : null
}
{
    "id" : 0,
    "username" : "Alice",
    "password" : null,
    "address" : {
        "id" : 0,
        "country" : "China",
        "province" : null,
        "street" : null
    }
}
{
    "name" : "Alice",
    "age" : 40,
    "telephone" : "1234567"
}
```

