# ListFactoryBean 注入 List -- 使用 `<util:list>`

```
    <!-- 注入list,使用 value -->
    <bean id="listExample" class="com.jiangge.beans.CollectionHolder">
        <property name="list">
            <list>
                <value>good</value>
                <value>better</value>
                <value>没太棒了</value>
            </list>
        </property>
    </bean>
```

上面的方式注入 List，List 的类型不受我们控制，默认是 `java.util.ArrayList`。

`ListFactoryBean` 可以创建一个**特定类型**的 List。由于 `ListFactoryBean` 的配置太过麻烦，这里我们只介绍和其等效的 `<util:list>`。


## spring-beans.xml

```
<?xml version=“1.0” encoding=“UTF-8”?>

 <beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns:util="http://www.springframework.org/schema/util"
 xsi:schemaLocation="
 http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/util
 http://www.springframework.org/schema/util/spring-util.xsd">

<!-- 注入list,使用 value -->
<!--
<bean id="listExample" class="com.jiangge.beans.CollectionHolder">
    <property name="list">
        <list>
            <value>good</value>
            <value>better</value>
            <value>best</value>
        </list>
    </property>
</bean>
-->

<!-- 注入 有类型的 list，util:list -->
<bean id="listExample" class="com.jiangge.beans.CollectionHolder">
    <property name="list">
        <util:list list-class="java.util.LinkedList">
            <value>good</value>
            <value>better</value>
            <value>best</value>
        </util:list>
    </property>
</bean>
</beans>

```
注：需要引入 `util schema`。

## 测试 ListFactoryBeanTest

```
import com.xtuer.beans.CollectionHolder;
import com.xtuer.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ListFactoryBeanTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        CollectionHolder holder = context.getBean("listExample", CollectionHolder.class);
        CommonUtils.output(holder.getList());
        CommonUtils.output(holder.getList().getClass());
    }
}
```

## 输出：

```
[
    "good",
    "better",
    "best"
]
"java.util.LinkedList"
```

