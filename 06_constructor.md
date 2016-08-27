# Constructor 注入


演示怎么给构造函数注入参数，使用指定的构造函数创建对象。

## Customer


```
package com.jiangge.beans;


public class Customer {
    private String name;
    private int age;
    private String telephone;

    public Customer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Customer(String name, String telephone, int age) {
        this.name = name;
        this.telephone = telephone;
        this.age = age;
    }

    public Customer(String name, int age, String telephone) {
        this.name = name;
        this.age = age;
        this.telephone = telephone;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getTelephone() {
        return telephone;
    }

    public void setTelephone(String telephone) {
        this.telephone = telephone;
    }
}

```

## spring-beans.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
       
    <bean id="customer" class="com.jiangge.beans.Customer">
        <constructor-arg value="Alice" />
        <constructor-arg value="40" />
        <constructor-arg value="1234567"/>
    </bean>
```

## InjectionTest

```
import com.jiangge.beans.Customer;
import com.jiangge.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class InjectionTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        Customer user = context.getBean("customer", Customer.class);
        CommonUtils.output(user);
    }
}

```

## 构造函数注入时，可以增加 type 字段

```
<bean id="customer" class="com.jiangge.beans.Customer">
   <constructor-arg type="java.lang.String" value="Alice"/>
   <constructor-arg type="int" value="40"/>
   <constructor-arg type="java.lang.String" value="1234567"/>
</bean>
```

