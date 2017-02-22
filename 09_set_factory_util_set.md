# SetFactoryBean 注入 Set -- `<util:set>`

```
<bean id="setExample" class="com.jiangge.beans.CollectionHolder">
   <property name="set">
       <set> <!--表示是 set-->
           <value>good</value>
           <value>better</value>
           <value>best</value>
       </set>
   </property>
</bean>
```
上面的方式注入 Set，Set 的类型不受我们控制，默认是 `java.util.LinkedHashSet`。

`SetFactoryBean` 可以创建一个特定类型的 Set。由于 `SetFactoryBean` 的配置太过麻烦，这里我们只结束和其等效的 `<util:set>`。

## spring-beans.xml


```<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util
       http://www.springframework.org/schema/util/spring-util.xsd">

    <bean id="setExample" class="com.jiangge.beans.CollectionHolder">
        <property name="set">
            <util:set set-class="java.util.HashSet">
                    <value>good</value>
                    <value>better</value>
                    <value>best</value>
            </util:set>
        </property>
    </bean>
</beans>
```

- 需要引入 util schema。
- 可以不指定 set-class，这样就可默认的 <set> 注入没有什么区别了。
	
## SetFactoryBeanTest

```
import com.jiangge.beans.CollectionHolder;
import com.jiangge.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SetFactoryBeanTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        CollectionHolder holder = context.getBean("setExample", CollectionHolder.class);
        CommonUtils.output(holder.getSet());
        CommonUtils.output(holder.getSet().getClass());
    }
}
```

## 输出

```
[
    "good",
    "better",
    "best"
]
"java.util.HashSet"
```


