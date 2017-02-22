# Autowired 注入

`@Autowired` 是**基于类型的注入**，Spring 会在 IoC 容器里查找**类型匹配**的 Bean 注入。

## spring-beans.xml

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

    <context:component-scan base-package="com.jiangge.beans"/>

    <bean class="com.jiangge.beans.Door"/>
</beans>
```

## Door

```java
package com.jiangge.beans;

public class Door {
    private String description;

    public Door() {
        description = "Default Door";
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```
## House

使用 `@Autowired`注入 door。

```java
package com.jiangge.beans;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

// 生成的 Bean 的 ID 是 house
@Component
public class House {
    private String description;

    @Autowired
    private Door door;

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Door getDoor() {
        return door;
    }

    public void setDoor(Door door) {
        this.door = door;
    }
}
```

## 测试

```java
import com.jiangge.beans.House;
import com.jiangge.util.CommonUtils;
import org.junit.Assert;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ComponentScanTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        House house1 = context.getBean(House.class);
        CommonUtils.output(house1);
   
        //House house2 = context.getBean("house", House.class);
                 //CommonUtils.output(house2);   
    }
}
```

## 输出

```
{
    "description" : null,
    "door" : {
        "description" : "Default Door"
    }
}
```


