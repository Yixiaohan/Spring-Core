# Component-Scan

使用 `<context:component-scan>` 特性，可以自动扫描 `base-package `下类名有注解 `@Component`、`@Service` 或者 `@Controller` 的类，为其在 `Spring 容器`里创建一个对象。

`@Service`、`@Controller` 和 `@Component` 在语法上作用是一样的，区别是各自有自己的语义，例如 SpringMVC 里用 `@Controller` 表明类是 `Controller`。

## spring-beans.xml


```
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
</beans>
```

## House

这里，我们使用@Component

```
package com.jiangge.beans;

import org.springframework.stereotype.Component;

// 生成的 Bean 的 ID 是 house
@Component
public class House {
    private String description;

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```

## 测试 ComponentScanTest

```
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

        House house2 = context.getBean("house", House.class);
        CommonUtils.output(house2);

        Assert.assertEquals(house1, house2);
    }
}
```

## 输出


```
{
    "description" : null
}
{
    "description" : null
}
```


