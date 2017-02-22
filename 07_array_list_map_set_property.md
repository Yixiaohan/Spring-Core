# Array, List, Set, Map, Properties 注入


## CollectionHolder



```
package com.jiangge.beans;

import java.util.*;

public class CollectionHolder {
    private List<String> list;
    private List<User> listUers;
    private Set<String> set;
    private Map<String, String> map;
    private Properties props;
    private int[] array;

   // 省略 setter and getter
}
```

## User
```
package com.jiangge.beans;

public class User {
    private int id;
    private String username;
    private String password;
    private Address address;

    public User() {
    }

    public User(int id, String username) {
        this.id = id;
        this.username = username;
    }

    // 省略 setter and getter
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
<!--
    <bean id="address" class="com.jiangge.beans.Address">
        <property name="country" value="China"/>
        <property name="province" value="BeiJing"/>
        <property name="street" value="ChaoYang"/>
    </bean>

    <bean id="user" class="com.jiangge.beans.User">
        <property name="username" value="Alice"/>
        <property name="password" value="Passw0rd"/>
        <property name="address" ref="address"/>
    </bean>

    <bean id="customer" class="com.jiangge.beans.Customer">
        <constructor-arg value="Alice" />
        <constructor-arg value="40" />
        <constructor-arg value="1234567"/>
    </bean>

-->
    <bean id="address" class="com.jiangge.beans.Address">
        <property name="country" value="China"/>
        <property name="province" value="BeiJing"/>
        <property name="street" value="ChaoYang"/>
    </bean>


    <bean id="user" class="com.jiangge.beans.User" >
        <property name="id"  value="01"/>
        <property name="username"  value="xiaohan"/>
        <property name="password"  value="123456"/>
        <property name="address"  ref="address"/>
    </bean>

    <!-- 注入数组 -->
    <bean id="arrayExample" class="com.jiangge.beans.CollectionHolder">
        <property name="array">
            <list>
                <value>11</value>
                <value>22</value>
                <value>33</value>
            </list>
        </property>
    </bean>

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

    <!-- 注入list, 使用 ref -->
    <bean id="listExample2" class="com.jiangge.beans.CollectionHolder">
        <property name="listUers">
            <list>
                <ref bean="user" />
                <bean class="com.jiangge.beans.User" />
            </list>
        </property>
    </bean>

    <!-- 注入set -->
    <bean id="setExample" class="com.jiangge.beans.CollectionHolder">
        <property name="set">
            <set>
                <value>长江</value>
                <value>黄河</value>
                <value>大山</value>
            </set>
        </property>
    </bean>

    <!-- 注入map -->
    <bean id="mapExample" class="com.jiangge.beans.CollectionHolder">
        <property name="map">
            <map>
                <entry key="German" value="OK"/>
                <entry key="English" value="Good"/>
                <entry key="Chinese" value="没问题"/>
            </map>
        </property>
    </bean>

    <!-- 注入property -->
    <bean id="propsExample" class="com.jiangge.beans.CollectionHolder">
        <property name="props">
            <props>
                <prop key="German">你好</prop>
                <prop key="English">他好</prop>
                <prop key="Chinese">大家好</prop>
            </props>
        </property>
    </bean>

</beans>
```

## InjectionTest

```
import com.jiangge.beans.CollectionHolder;
import com.jiangge.beans.Customer;
import com.jiangge.beans.User;
import com.jiangge.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;


public class InjectionTest {
    private static ApplicationContext context;

//    @BeforeClass
//    public static void setup() {
//        context = new ClassPathXmlApplicationContext("spring-beans.xml");
//    }
//
//    @Test
//    public void test() {
//        Customer customer = context.getBean("customer", Customer.class);
//        CommonUtils.output(customer);
//    }

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void testArray() {
        CollectionHolder holder = context.getBean("arrayExample", CollectionHolder.class);
        CommonUtils.output(holder.getArray());
        CommonUtils.output(holder.getArray().getClass());
    }

    @Test
    public void testList() {
        CollectionHolder holder = context.getBean("listExample", CollectionHolder.class);
        CommonUtils.output(holder.getList());
        CommonUtils.output(holder.getList().getClass());
    }

    @Test
    public void testList2() {
        CollectionHolder holder = context.getBean("listExample2", CollectionHolder.class);
        CommonUtils.output(holder.getListUers());
        CommonUtils.output(holder.getListUers().getClass());
    }

    @Test
    public void testSet() {
        CollectionHolder holder = context.getBean("setExample", CollectionHolder.class);
        CommonUtils.output(holder.getSet());
        CommonUtils.output(holder.getSet().getClass());
    }

    @Test
    public void testMap() {
        CollectionHolder holder = context.getBean("mapExample", CollectionHolder.class);
        CommonUtils.output(holder.getMap());
        CommonUtils.output(holder.getMap().getClass());
    }

    @Test
    public void testProps() {
        CollectionHolder holder = context.getBean("propsExample", CollectionHolder.class);
        CommonUtils.output(holder.getProps());
        CommonUtils.output(holder.getProps().getClass());
    }
}
```


