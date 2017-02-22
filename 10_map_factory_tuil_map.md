# MapFactoryBean 注入 Map -- 使用`<util:map>`

```
<bean id="mapExample" class="com.jiangge.beans.CollectionHolder">
   <property name="map">
       <map> <!--表示是 map-->
           <entry key="German" value="Gut"/>
           <entry key="English" value="Good"/>
           <entry key="Chinese" value="没问题"/>
       </map>
   </property>
</bean>
```

上面的方式注入 Map，Map 的**类型**不受我们控制，默认是 `java.util.LinkedHashMap`。MapFactoryBean 可以创建一个**特定类型**的 Map。由于 MapFactoryBean 的配置太过麻烦，这里我们只使用和其等效的 `<util:map>`。

## spring-beans.xml


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util
       http://www.springframework.org/schema/util/spring-util.xsd">

    <bean id="mapExample" class="com.jiangge.beans.CollectionHolder">
        <property name="map">
            <util:map map-class="java.util.HashMap">
                <entry key="German"  value="Gut"/>
                <entry key="English" value="Good"/>
                <entry key="Chinese" value="没问题"/>
            </util:map>
        </property>
    </bean>
</beans>
```

- 需要引入 `util schema`。
- 可以不指定 map-class，这样就可默认的 <map> 注入没有什么区别了。


## MapFactoryBeanTest


```
import com.jiangge.beans.CollectionHolder;
import com.jiangge.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MapFactoryBeanTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        CollectionHolder holder = context.getBean("mapExample", CollectionHolder.class);
        CommonUtils.output(holder.getMap());
        CommonUtils.output(holder.getMap().getClass());
    }
}
```

## 输出

```
{
    "English" : "Good",
    "Chinese" : "没问题",
    "German" : "Gut"
}
"java.util.HashMap"
```




