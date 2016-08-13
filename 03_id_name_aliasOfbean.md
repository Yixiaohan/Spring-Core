#ID, Name, Alias of Bean

- 每个 Bean 都有标志符，用这个标志符从 Spring IoC Container 里获取 Bean
- id, name, alias 都必须在 IoC 容器里唯一
- 每个 Bean 可以有一个 id 属性，并可以根据该 id 在 IoC 容器中查找该 Bean，该 id 属性值必须在 IoC 容器中唯一
- 如果不指定 id，只指定 name，那么 name 为 Bean 的标识符，并且需要在容器中唯一
- 同时指定 name 和 id，此时 id 为标识符，而 name 为 Bean 的别名，两者都可以找到目标 Bean
- 可以指定多个 name，之间可以用分号 ;、空格 或逗号 , 分隔开，如果没有指定 id，那么第一个 name 为标识符，其余的为别名；若指定了 id 属性，则 id 为标识符，所有的 name 均为别名
- 可以使用 <alias> 标签指定别名，别名也必须在 IoC 容器中唯一
- 如果 id 和 name 都不指定，那么 Spring 容器会为 Bean 生成一个 id，生成规则： 按 Bean 定义的顺序，在类的全路径名加上 #序号 (序号从 0 开始，第一个 bean 的别名是类的全路径名，所以可以使用类的全路径名来获取)，如：

```
com.jiangge.beans.User#0 (com.jiangge.beans.User)
com.jiangge.beans.User#1
com.jiangge.beans.User#2
com.jiangge.beans.User#3
```

##id 与 name 的区别

- id 与 name 属性在作用上基本没有区别，推荐使用 id
- id 命名必须满足 XML 的命名规范，因为 id 其实是 XML 中就做了限定的。总结起来就相当于一个Java 变量的命名：不能以数字，符号打头，不能有空格，如 123，?ad, "ab" 等都是不规范的，Spring 在初始化时就会报错
- name 属性则没有这些限定，你可以使用几乎任何的名称，就是一普通字符串
- 在 Spring 同一个上下文中 id，name 只能有一个，但是同一个 id 可以用多个别名
- 在 IDE 里，id 重复会实时报错，而 name 重复则不会在 IDE 里报错，从开发的角度来说，也是推荐使用 id

## 别名 <alias> 的好处

- 接口 UserDao 定义了多个实现: UserDaoMySqlImpl, UserDaoOracleImpl，而且可以在 XML 里同时定义他们的 Bean: userDaoMySqlImpl、userDaoOracleImpl。

- 使用别名 userDao 可以指向 userDaoMySqlImpl 或者 userDaoOracleImpl，在程序里获取 Bean 时使用别名来获取，而不是具体的 userDaoMySqlImpl 或者 userDaoOracleImpl，这样在切换接口的实现时只需要修改 XML 里的别名，而不是代码里 Bean 的 id。
下面将用例子演示给 Bean 指定 id, name, alias 以及不指定 id 和 name，Spring 自动生成 id。

## spring-beans.xml


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--给 bean 指定 id-->
    <bean id="alice" class="com.jiangge.beans.User">
        <property name="username" value="Alice"/>
    </bean>

    <!--bean 可以有 name，可以有多个 name-->
    <bean id="robert" name="hacker, developer" class="com.jiangge.beans.User">
        <property name="username" value="Robert"/>
    </bean>

    <!--可以给 bean 指定别名-->
    <alias alias="user-x" name="alice"/>

    <!--没有指定 id 和 name，则自动生成 id-->
    <!--id 为 com.jiangge.beans.User#0, 别名为 com.jiangge.beans.User-->
    <bean class="com.jiangge.beans.User">
        <property name="username" value="com.jiangge.beans.User or com.jiangge.beans.User#0"/>
    </bean>

    <!--id 为 com.jiangge.beans.User#1-->
    <bean class="com.jiangge.beans.User">
        <property name="username" value="com.jiangge.beans.User#1"/>
    </bean>

    <!--id 为 com.jiangge.beans.User#2-->
    <bean class="com.jiangge.beans.User">
        <property name="username" value="com.jiangge.beans.User#2"/>
    </bean>

</beans>
```

## com.jiangge.beans.User;

```
package com.jiangge.beans;

/**
 * Created by jiangge on 16/8/13.
 */
public class User {
    private int id;
    private String username;

    public User() {
    }

    public User(int id, String username) {
        this.id = id;
        this.username = username;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}

```

## IdentifierTest


```
import com.jiangge.beans.User;
import org.junit.AfterClass;
import org.junit.Assert;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Created by jiangge on 16/8/13.
 */
public class IdentifierTest {
    private static ApplicationContext context;
    private static ObjectMapper mapper = new ObjectMapper();

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @AfterClass
    public static void tearDown() {
    }

    /**
     * 测试使用 bean id 获取 user
     */
    @Test
    public void testBeanById() {
        User alice = context.getBean("alice", User.class);
        output(alice);
    }

    /**
     * 测试使用 bean name 获取 user
     * robert, hacker, developer 都是同一个人
     * robert 是 id
     * hacker 和 developer 是 name
     */
    @Test
    public void testBeanByName() {
        User robert = context.getBean("robert", User.class);
        output(robert);

        User hacker = context.getBean("hacker", User.class);
        output(hacker);

        User developer = context.getBean("developer", User.class);
        output(developer);

        Assert.assertEquals(robert, hacker);
        Assert.assertEquals(robert, developer);
        Assert.assertEquals(hacker, developer);
    }

    /**
     * 测试使用 bean alias 获取 user
     * alice 和 user-x 是同一个人
     * alice 是 id
     * user-x 是别名
     */
    @Test
    public void testBeanByAlias() {
        User alice = context.getBean("alice", User.class);
        output(alice);

        User userX = context.getBean("user-x", User.class);
        output(userX);

        Assert.assertEquals(alice, userX);
    }

    /**
     * 测试自动生成的 bean identifier
     */
    @Test
    public void testBeanByGeneratedIdentifier() {
        User user = context.getBean("com.jiangge.beans.User", User.class); // 等于 User.class.getName()
        output(user);

        User user0 = context.getBean("com.jiangge.beans.User#0", User.class);
        output(user0);

        User user1 = context.getBean("com.jiangge.beans.User#1", User.class);
        output(user1);

        User user2 = context.getBean("com.jiangge.beans.User#2", User.class);
        output(user2);

        Assert.assertEquals(user, user0);
        Assert.assertNotEquals(user0, user1);
        Assert.assertNotEquals(user0, user2);
        Assert.assertNotEquals(user1, user2);
    }

    /**
     * 以 Json 的格式输出对象
     * @param obj
     */
    private void output(Object obj) {
        try {
            System.out.println(mapper.writeValueAsString(obj));
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }


}

```


