# Qualifier

当找到**多个同类型的 Bean** 的定义时，`@Autowired` 注入的时候不知道要用哪一个，会报异常，可以使用 **@Qualifier 指定**要注入的 Bean。

## spring-beans.xml

door1、door2的类型都为 Door

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

    <bean id="door1" class="com.jiangge.beans.Door"/>
    <bean id="door2" class="com.jiangge.beans.Door"/>
</beans>
```
## House

使用 `@Qualifier("door1")` ，注入标志为 door1 的 Bean
```
package com.jiangge.beans;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

// 生成的 Bean 的 ID 是 house
@Component
public class House {
    private String description;

    @Autowired
    @Qualifier("door1") // 注入标志为 door1 的 Bean
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



