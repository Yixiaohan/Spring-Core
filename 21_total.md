# 环境搭建

从 Windows、MyEclipse 切换到 Mac OS X、IntelliJ IDEA 适应一下新的开发环境 :-)


------
* Mac OS X
* IntelliJ IDEA
* 使用 Maven 管理项目开发
* 使用 JUnit 来执行测试的内容，这样可以在一个类里有多个可执行的方法，方便测试不同的内容


```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.jiangge</groupId>
    <artifactId>SpringCore</artifactId>
    <version>1.0-SNAPSHOT</version>


    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.1.1.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.5.3</version>
        </dependency>
    </dependencies>

    <build>
        <finalName>SpringCore</finalName>
        <plugins>
            <plugin>
                <!--可选：配置编译器信息-->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source> <!-- 源代码使用的开发版本 -->
                    <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->
                    <encoding>UTF8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

---

# Spring Hello World

## Module 的结构图
![结构图](https://github.com/Yixiaohan/Spring-Core/blob/master/assets/helloWorld.png)


## spring-beans.xml

叫做 Bean configuration file，定义 bean 的属性和其他 bean 之间的依赖，Spring 根据 bean 的定义生成 bean。


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userDao" class="com.jiangge.dao.UserDaoMySqlImpl"/>

</beans>
```


## UserDao

```java
package com.jiangge.dao;

public interface UserDao {
    public void deleteUser(int id);
}
```

## UserDaoMySqlImpl


```java
package com.jiangge.dao;

public class UserDaoMySqlImpl implements UserDao {
    @Override
    public void deleteUser(int id) {
        System.out.println("UserDaoMySqlImpl.deleteUser()");
    }
}
```


## UserDaoOracleImpl


```java
package com.jiangge.dao;

public class UserDaoOracleImpl implements UserDao {
    @Override
    public void deleteUser(int id) {
        System.out.println("UserDaoOracleImpl.deleteUser()");
    }
}
```


## HelloWorld

```java
import com.jiangge.dao.UserDao;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class HelloWorld {
    @Test
    public void test() {
        // Spring 读取 bean 的定义文件并生成 bean
        ApplicationContext context = new ClassPathXmlApplicationContext("spring-beans.xml");
        
        // 从 Spring IoC 容器里获取 bean
        UserDao userDao = context.getBean("userDao", UserDao.class);
        userDao.deleteUser(0);
    }
}
```

## 输出

```
UserDaoMySqlImpl.deleteUser()
```

## 更换 UserDao 的实现

```xml
<bean id="userDao" class="com.jiangge.dao.UserDaoMySqlImpl"/> 
```

修改为

```xml
<bean id="userDao" class="com.jiangge.dao.UserDaoOracleImpl"/>，
```

输出
`UserDaoOracleImpl.deleteUser()`


把 UserDao 的实现从 UserDaoMySqlImpl 修改为 UserDaoOracleImpl，只需要修改 XML 配置文件，不需要修改代码。

**Spring IoC + 接口** ==> **容易实现非侵入式的开发，扩展已有代码。**

---

# Spring IoC Introduction

## IoC 是什么？

控制反转（Inversion of Control，英文缩写为 IoC）是一个重要的面向对象编程的法则来削减计算机程序的耦合问题，也是轻量级的 Spring 框架的核心。 控制反转一般分为两种类型，依赖注入（Dependency Injection，简称 DI）和依赖查找（Dependency Lookup）。依赖注入应用比较广泛。

应用控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体(Spring IoC Container)将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。所以，控制反转是，关于一个对象如何获取他所依赖的对象的引用，这个责任的反转。

对象的生成不再是在代码里用 new 创建的，而是在 XML 里定义对象之间的依赖关系，然后由 Spring 来生成对象。

## IoC 最大的好处是什么？

因为把对象生成放在了 XML 里定义，所以当我们需要换一个实现子类将会变成很简单（一般这样的对象都是实现于某种接口的），只要修改 XML 就可以了，这样我们甚至可以实现对象的热插拔（有点像 USB 接口和 SCSI 硬盘了）。

## IoC 最大的缺点是什么？
生成一个对象的步骤变复杂了（事实上操作上还是挺简单的），对于不习惯这种方式的人，会觉得有些别扭和不直观。
对象生成因为是使用反射编程，在效率上有些损耗。但相对于 IoC 提高的维护性和灵活性来说，这点损耗是微不足道的，除非某对象的生成对效率要求特别高。
缺少 IDE 重构操作的支持，如果在 Eclipse 要对类改名，那么你还需要去XML文件里手工去改了，这似乎是所有 XML 方式的缺憾所在。


---

# ID, Name, Alias of Bean

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

## id 与 name 的区别

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


```xml
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

```java
package com.jiangge.beans;


public class User {
    private int id;
    private String username;

    public User() {
    }

    public User(int id, String username) {
        this.id = id;
        this.username = username;
    }

    //省略 setter、getter
}

```

## IdentifierTest


```java
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

## 输出


```
{"id":0,"username":"com.jiangge.beans.User or com.jiangge.beans.User#0"}
{"id":0,"username":"com.jiangge.beans.User or com.jiangge.beans.User#0"}
{"id":0,"username":"com.jiangge.beans.User#1"}
{"id":0,"username":"com.jiangge.beans.User#2"}
{"id":0,"username":"Robert"}
{"id":0,"username":"Robert"}
{"id":0,"username":"Robert"}
{"id":0,"username":"Alice"}
{"id":0,"username":"Alice"}
{"id":0,"username":"Alice"}
```

---

# Setter 注入

## CommonUtils

```java
package com.jiangge.util;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.util.DefaultIndenter;
import com.fasterxml.jackson.core.util.DefaultPrettyPrinter;
import com.fasterxml.jackson.databind.ObjectMapper;

public class CommonUtils {
    private static DefaultPrettyPrinter printer;
    static {
        // Setup a pretty printer with an indenter (indenter has 4 spaces in this case)
        DefaultPrettyPrinter.Indenter indenter = new DefaultIndenter("    ", DefaultIndenter.SYS_LF);
        printer = new DefaultPrettyPrinter();
        printer.indentObjectsWith(indenter);
        printer.indentArraysWith(indenter);
    }

    /**
     * 以 Json 的格式输出对象
     * @param obj
     */
    public static void output(Object obj) {
        ObjectMapper mapper = new ObjectMapper();

        try {
            // 默认用 2 个空格缩进
            // System.out.println(mapper.writerWithDefaultPrettyPrinter().writeValueAsString(obj));

            // 自定义缩进，用 4 个空格
            System.out.println(mapper.writer(printer).writeValueAsString(obj));
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }
}
```

## User

```java
package com.jiangge.beans;

public class User {
    private int id;
    private String username;
    private String password;
    private Address address;
    
    // 省略 setter and getter
}
```

## Address


```java
package com.jiangge.beans;

public class Address {
    private int id;
    private String country;
    private String province;
    private String street;
    
    // 省略 setter and getter
}
```

## spring-beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

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
</beans>
```

## InjectionTest

```java
import com.jiangge.beans.User;
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
        User user = context.getBean("user", User.class);
        CommonUtils.output(user);
    }
}
```

## 输出

```
{
    "id" : 0,
    "username" : "Alice",
    "password" : "Passw0rd",
    "address" : {
        "id" : 0,
        "country" : "China",
        "province" : "BeiJing",
        "street" : "ChaoYang"
    }
}
```

## 说明

- `<property name="username" value="Alice"/> `调用 user.setUsername("Alice") 设置属性 username。

- `<property name="address" ref="address"/>`，找到标志为 address(可以是 id, name, alias) 的 Bean addressBean，然后调用 user.setAddress(addressBean) 设置属性 address。


## 注入方式: attribute 和 element

- value 和 ref 都可以使用 attribute 和 element 的方式注入，推荐尽量用 attribute 的方式，这样会简洁一些
- value 用于简单类型: primitive type(int, boolean, float 等), String, Date, 还有注册了 PropertyEditor 的类型(把字符串转换成对象)。
- ref 用于引用 Bean Configuration File 里定义的另一个 Bean


## value 用 element 的方式如下注入:

```xml
<bean id="user" class="com.jiangge.beans.User">
  <property name="username">
      <value>Alice</value>
  </property>
  <property name="password" value="Passw0rd"/>
  <property name="address" ref="address"/>
</bean>

```

## ref 用 element 的方式如下注入:

```xml
<bean id="user" class="com.jiangge.beans.User">
  <property name="username" value="Alice"/>
  <property name="password" value="Passw0rd"/>
  <property name="address">
      <ref bean="address"/>
  </property>
</bean>
```

## 注入匿名对象

每创建一次 User，都会新创建一个 Address 的匿名对象。
使用 `<bean>`，不能设置 id, name 等。


```xml
<bean id="user" class="com.jiangge.beans.User">
   <property name="username" value="Alice"/>
   <property name="password" value="Passw0rd"/>
   <property name="address">
       <bean class="com.jiangge.beans.Address">
           <property name="country" value="Germany"/>
           <property name="province" value="Braunschweig"/>
           <property name="street" value="Wiesenstrasse"/>
       </bean>
   </property>
</bean>
```



---

# Constructor 注入


演示怎么给构造函数注入参数，使用指定的构造函数创建对象。

## Customer


```java
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

     //省略 setter、getter
}

```

## spring-beans.xml

```xml
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

```java
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

```xml
<bean id="customer" class="com.jiangge.beans.Customer">
   <constructor-arg type="java.lang.String" value="Alice"/>
   <constructor-arg type="int" value="40"/>
   <constructor-arg type="java.lang.String" value="1234567"/>
</bean>
```


---

# Array, List, Set, Map, Properties 注入


## CollectionHolder



```java
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
```java
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

```xml
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

```xml
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



---

# ListFactoryBean 注入 List -- 使用 `<util:list>`

```xml
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

```xml
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

```java
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

---

# SetFactoryBean 注入 Set -- `<util:set>`

```xml
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


```xml
<?xml version="1.0" encoding="UTF-8"?>
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

```java
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


---

# MapFactoryBean 注入 Map -- 使用`<util:map>`

```xml
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


```xml
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


```java
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

---

# FactoryBean 

FactoryBean 是用来创建 Bean 的工厂，实现 FactoryBean 接口的类就可以用来创建其他 Bean 了，在 Bean Configuration File 里的 <bean> 里定义要生成的 Bean。

FactoryBean **命名规范**：Bean 的类名后跟着 FactoryBean，例如创建 User 的 FactoryBean 的类名应该为 `UserFactoryBean`。

如果 Bean 的创建不只是简单的 setter, constructor 注入，而是有其他的逻辑，这个时候就可以用 FactoryBean 来创建 Bean(有点像同时用了 `Factory Pattern` 和 `Builder Pattern`)。

FactoryBean 是很常见的，例如 Spring 集成 MyBatis 时用 SqlSessionFactoryBean 来创建 SqlSession。

下面的示例展示怎么实现 UserFactoryBean，用来创建 User 的对象。

## spring-beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="user" class="com.jiangge.beans.factory.UserFactoryBean">
        <property name="username" value="Alice"/>
        <property name="password" value="Passw0rd"/>
    </bean>
</beans>
```
- <bean> 和普通 Bean 的定义完全一样
- 虽然最后是要生成 User 的对象，但是 class 不是 `com.jiangge.beans.User`，而是创建它的工厂 `com.jiangge.beans.factory.UserFactoryBean`
- username, password 等都是注入到 `UserFactoryBean`
- 当 Spring 看到 UserFactoryBean 实现了 FactoryBean 接口，就会调用 `UserFactoryBean.getObject()` 来创建 User 的对象

## User

```java
package com.jiangge.beans;

public class User {
    private int id;
    private String username;
    private String password;
    private Address address;
    
    // 省略 setter and getter
}
```
## UserFactoryBean

```java
package com.jiangge.beans.factory;

import com.jiangge.beans.User;
import org.springframework.beans.factory.FactoryBean;

public class UserFactoryBean implements FactoryBean {
    private String username;
    private String password;

    @Override
    public Object getObject() throws Exception {
        User user = new User();

        // 可以有很复杂的逻辑
        user.setUsername(username.toUpperCase());
        user.setPassword(password);

        return user;
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
## FactoryBeanTest

```java
import com.jiangge.beans.User;
import com.jiangge.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class FactoryBeanTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        // 这里得到的不是 UserFactoryBean，而是它的 getObject() 方法返回的 User
        User user = context.getBean("user", User.class);
        CommonUtils.output(user);
        CommonUtils.output(user.getClass());
    }
}
```
## 输出

```
{
    "id" : 0,
    "username" : "ALICE",
    "password" : "Passw0rd",
    "address" : null
}
"com.jiangge.beans.User"
```


---

#Property Editor

### 在设置一个对象类型的属性时：

1. 使用 ref 引用已经存在的对象
2.	 使用 bean 标签创建一个匿名对象
3.	 还有一种方式是给 value 赋值一个字符串，Spring 自动的把这个字符串转换为对象

### 第三种方式的实现需要以下几步：

1.	 需要提供一个把 String 转换为对象的类，称之为`自定义编辑器（CustomEditor）`
2.	 然后把这个`自定义编辑器`注册到 Spring
3.	 使用字符串配置对象属性

下面就以为 Address 实现一个`自定义编辑器 AddressEditor` 为例，介绍具体的实现细节。

## 1. User

```
package com.jiangge.beans;

public class User {
    private int id;
    private String username;
    private String password;
    private Address address;
    
    // 省略 setter and getter
}
```
## 2. Address

```
package com.jiangge.beans;

public class Address {
    private int id;
    private String country;
    private String province;
    private String street;
    
    // 省略 setter and getter
}
```

##3. AddressEditor

自定义编辑器需要实现 `java.beans.PropertyEditor` 接口，`java.beans.PropertyEditorSupport` 提供了 `PropertyEditor` 的默认实现，我们只需要继承 `PropertyEditorSupport` 并实现`setAsText() `方法就可以了。


```java
package com.jiangge.beans.editor;

import com.jiangge.beans.Address;
import java.beans.PropertyEditorSupport;

public class AddressEditor extends PropertyEditorSupport {
    /**
     * 把字符串转换为 Address 的对象。
     *
     * @param text 格式为 id|country|province|street
     * @throws IllegalArgumentException
     */
    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        String[] tokens = text.split("\\|");

        Address address = new Address();
        address.setId(Integer.parseInt(tokens[0]));
        address.setCountry(tokens[1]);
        address.setProvince(tokens[2]);
        address.setStreet(tokens[3]);

        setValue(address);
    }
}
```
## 4. spring-beans.xml 里注册自定义编辑器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--注册自定义的编辑器-->
    <bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
        <property name="customEditors">
            <map>
                <!--Key 是要生成对象的类名，Value 是 Editor 的类名-->
                <entry key="com.jiangge.beans.Address" value="com.jiangge.beans.editor.AddressEditor"/>
            </map>
        </property>
    </bean>
    ...
</beans>
```
## 5. 使用字符串配置 Address 对象属性


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--注册自定义的 PropertyEditor-->
    <bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
        <property name="customEditors">
            <map>
                <!--Key 是要生成对象的类名，Value 是 Editor 的类名-->
                <entry key="com.jiangge.beans.Address" value="com.jiangge.beans.editor.AddressEditor"/>
            </map>
        </property>
    </bean>

    <bean id="user" class="com.jiangge.beans.User">
        <property name="username" value="Alice"/>
        <property name="password" value="Passw0rd"/>
        
        <!--Spring 会把字符串 "2|China|BeiJing|WangJin" 自动的转换为 Address 的对象-->
        <property name="address" value="2|China|BeiJing|WangJin"/>
    </bean>
</beans>
```
## 6. 测试


```java
import com.jiangge.beans.User;
import com.jiangge.util.CommonUtils;
import com.jiangge.beans.editor.AddressEditor;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AddressEditorTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        AddressEditor editor = new AddressEditor();
        editor.setAsText("2|China|BeiJing|WangJin");
        CommonUtils.output(editor.getValue());
    }

    @Test
    public void testAddressEditor() {
        User user = context.getBean("user", User.class);
        CommonUtils.output(user);
    }
}
```

## 输出

```
{
    "id" : 2,
    "country" : "China",
    "province" : "BeiJing",
    "street" : "WangJin"
}
{
    "id" : 0,
    "username" : "Alice",
    "password" : "Passw0rd",
    "address" : {
        "id" : 2,
        "country" : "China",
        "province" : "BeiJing",
        "street" : "WangJin"
    }
}
```


---

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

```xml
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


```xml
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

---

# Property-Placeholder

在 `.properties` 文件里定义 `key | value`，然后在 Bean Configuration File 里用 `${key}` 引用 key 对应的值。

Bean Configuration File 里使用 `<context:property-placeholder>` 引入 `properties` 文件。

## 1. user.properties

```
username = Alice
password = Passw0rd
```
## 2. address.propertes

```
country = China
province = BeiJing
```
## 3. spring-beans.xml

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

    <context:property-placeholder location="classpath:address.properties,classpath:user.properties" /> 


    <bean id="address" class="com.jiangge.beans.Address">
        <property name="country" value="${country}"/>
        <property name="province" value="${province}"/>
    </bean>

    <bean id="user" class="com.jiangge.beans.User">
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
        <property name="address" ref="address"/>
    </bean>
</beans>
```

## 4. PropertyPlaceholderTest

```java
import com.jiangge.beans.User;
import com.jiangge.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class PropertyPlaceholderTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        User user = context.getBean("user", User.class);
        CommonUtils.output(user);
    }
}
```

## 5. 输出

```json
{
    "id" : 0,
    "username" : "Alice",
    "password" : "Passw0rd",
    "address" : {
        "id" : 0,
        "country" : "China",
        "province" : "BeiJing",
        "street" : null
    }
}
```
---

# Component-Scan

使用 `<context:component-scan>` 特性，可以自动扫描 `base-package `下类名有注解 `@Component`、`@Service` 或者 `@Controller` 的类，为其在 `Spring 容器`里创建一个对象。

`@Service`、`@Controller` 和 `@Component` 在语法上作用是一样的，区别是各自有自己的语义，例如 SpringMVC 里用 `@Controller` 表明类是 `Controller`。

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
</beans>
```

## House

这里，我们使用@Component

```java
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

        House house2 = context.getBean("house", House.class);
        CommonUtils.output(house2);

        Assert.assertEquals(house1, house2);
    }
}
```

## 输出


```json
{
    "description" : null
}
{
    "description" : null
}
```


----

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

```json
{
    "description" : null,
    "door" : {
        "description" : "Default Door"
    }
}
```


---
# Qualifier

当找到**多个同类型的 Bean** 的定义时，`@Autowired` 注入的时候不知道要用哪一个，会报异常，可以使用 **@Qualifier 指定**要注入的 Bean。

## spring-beans.xml

door1、door2的类型都为 Door

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

    <bean id="door1" class="com.jiangge.beans.Door"/>
    <bean id="door2" class="com.jiangge.beans.Door"/>
</beans>
```
## House

使用 `@Qualifier("door1")` ，注入标志为 door1 的 Bean

```xml
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



---

# AspectJ with XML -- 使用 XML 配置 AOP

## 添加 aop 相关依赖

```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aop</artifactId>
       <version>4.1.1.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.8.1</version>
   </dependency>
```
## spring-beans.xml

pointcut 表示需要织入切面的地方

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="service" class="service.Service"/>
    <bean id="callAdvice" class="aop.CallAdvice"/>

    <!--使用 XML 配置 AOP-->
    <aop:config>
        <aop:aspect ref="callAdvice">
            <aop:pointcut id="methodPointCut" expression="execution(* service.*.*(..))"/>
            <aop:before pointcut-ref="methodPointCut" method="doBefore"/>
            
            <!--<aop:after  pointcut-ref="methodPointCut" method="doAfter"/>-->
            <!--<aop:around pointcut-ref="methodPointCut" method="doAround"/>-->
            <!--<aop:after-throwing pointcut-ref="methodPointCut"
                     method="doThrowing" throwing="ex"/>-->
        </aop:aspect>
    </aop:config>
</beans>
```
## Service

```java
package service;

public class Service 
{
    public void foo() 
    {
        System.out.println("service.Service.foo()");
    }
}
```
## CallAdvide

```java
package aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;

public class CallAdvice 
{
    public static int steps = 1;

    /**
     * 方法被执行前调用
     * @param jp
     */
    public void doBefore(JoinPoint jp) 
    {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: BEFORE  METHOD]: %s.%s()", steps++, className, methodName));
    }

    /**
     * 方法被执行后调用
     * @param jp
     */
    public void doAfter(JoinPoint jp) 
    {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: AFTER   METHOD]: %s.%s()\n", steps++, className, methodName));
    }

    /**
     * 方法被执行时调用
     * @param pjp
     * @return
     * @throws Throwable
     */
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable 
    {
        String className = pjp.getTarget().getClass().getName();
        String methodName = pjp.getSignature().getName();
        int sn = steps++;

        System.out.println(String.format("[%d: METHOD BEFORE]: %s.%s()", sn, className, methodName));

        long time = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        time = System.currentTimeMillis() - time;

        System.out.println(String.format("[%d: METHOD  AFTER]: %s.%s() : %dms\n", sn, className, methodName, time));

        return retVal;
    }

    /**
     * 异常抛出后调用
     * @param jp
     * @param ex
     */
    public void doThrowing(JoinPoint jp, Throwable ex) 
    {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: THROW EXCEPTION]: %s.%s()\n", steps++, className, methodName));
        System.out.println(ex.getMessage());
    }
}
```

## 测试 AspectJXmlAOPTest

```java
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import service.Service;

public class AspectJXmlAOPTest 
{
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() 
    {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() 
    {
        // service 是 Spring 生成的代理类的对象
        Service service = context.getBean("service", Service.class);
        service.foo();
        System.out.println(service.getClass());
    }
}
```

## 输出

```
    [1: BEFORE  METHOD]: service.Service.foo()
    service.Service.foo()
    class service.Service$$EnhancerBySpringCGLIB$$ed356710
```


---

# AspectJ with Annotation

## 需要添加依赖
   
```xml
<dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aop</artifactId>
       <version>4.1.1.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.8.1</version>
   </dependency>
```

## spring-beans.xml

基于 Annotation 的 AOP 需要使用 `<aop:aspectj-autoproxy/>`（如果实现了接口，则使用 JDK 的动态代理生成代理对象，如果没有接口，则使用 `CGLib` 生成代理对象。如果都用 `CGLib` 生成代理对象，则用 `<aop:aspectj-autoproxy proxy-target-class="true"/>`）。


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

    <aop:aspectj-autoproxy/> <!--启用 Annotation 的 AOP-->

    <bean id="callAdvice" class="aop.CallAdvice"/> <!--Aspect-->
    <bean id="service" class="service.Service"/>
</beans>
```

## Service

```
package service;

public class Service 
{
    public void foo() 
    {
        System.out.println("service.Service.foo()");
        throw new RuntimeException("Test exception");
    }
}
```

## CallAdvice

可以在 `Annotation` 里写 `pointcut`，如 `@Before("execution(* service.*.*(..))")`，也可以把 `pointcut` **提取出来以供复用**，如

```
@Pointcut("execution(* service.*.*(..))")
public void fooPointcut() {} 

@After("fooPointcut()") // 使用 pointcut
```


```
package aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;

@Aspect
public class CallAdvice {
    public static int steps = 1;

    @Pointcut("execution(* service.*.*(..))")
    public void fooPointcut() {}

    /**
     * 方法被执行前调用
     * @param jp
     */
    @Before("execution(* service.*.*(..))")
    public void doBefore(JoinPoint jp) {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: BEFORE  METHOD]: %s.%s()", steps++, className, methodName));
    }

    /**
     * 方法被执行后调用
     * @param jp
     */
    @After("fooPointcut()")
    public void doAfter(JoinPoint jp) {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: AFTER   METHOD]: %s.%s()\n", steps++, className, methodName));
    }

    /**
     * 方法被执行时调用
     * @param pjp
     * @return
     * @throws Throwable
     */
    @Around("fooPointcut()")
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        String className = pjp.getTarget().getClass().getName();
        String methodName = pjp.getSignature().getName();
        int sn = steps++;

        System.out.println(String.format("[%d: METHOD BEFORE doAround]: %s.%s()", sn, className, methodName));

        long time = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        time = System.currentTimeMillis() - time;

        System.out.println(String.format("[%d: METHOD  AFTER doAround]: %s.%s() : %dms\n", sn, className, methodName, time));

        return retVal;
    }

    /**
     * 异常抛出后调用
     * @param jp
     * @param error
     */
    @AfterThrowing(pointcut="fooPointcut()", throwing="error")
    public void doThrowing(JoinPoint jp, Throwable error) {
        String className = jp.getTarget().getClass().getName();
        String methodName = jp.getSignature().getName();
        System.out.println(String.format("[%d: THROW EXCEPTION]: %s.%s()\n", steps++, className, methodName));
        System.out.println(error.getMessage());
    }
}
```

## AspectJAnnotationAOPTest

```
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import service.Service;

public class AspectJAnnotationAOPTest 
{
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() 
    {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() 
    {
        // service 是 Spring 生成的代理类的对象
        Service service = context.getBean("service", Service.class);
        service.foo();
        System.out.println(service.getClass());
    }
}
```

## 输出
- 当 service 中没有抛出异常时：

```
public class Service 
{
    public void foo() 
    {
        System.out.println("service.Service.foo()");
    }
}
```

此时输出为：
    
    [1: METHOD BEFORE doAround]: service.Service.foo()
    [2: BEFORE  METHOD]: service.Service.foo()
    service.Service.foo()
    [1: METHOD  AFTER doAround]: service.Service.foo() : 36ms
    
    [3: AFTER   METHOD]: service.Service.foo()
    
    class service.Service$$EnhancerBySpringCGLIB$$3c33164f
   
- 当 service 中抛出异常时：

```
public class Service 
{
    public void foo() 
    {
        System.out.println("service.Service.foo()");
        throw new RuntimeException("Test exception");
    }
}
```
此时输出为：

    [1: METHOD BEFORE doAround]: service.Service.foo()
    [2: BEFORE  METHOD]: service.Service.foo()
    service.Service.foo()
    [3: AFTER   METHOD]: service.Service.foo()
    
    [4: THROW EXCEPTION]: service.Service.foo()
    
    Test exception
    
    java.lang.RuntimeException: Test exception
    
    	at service.Service.foo(Service.java:8)
    	at service.Service$$FastClassBySpringCGLIB$$c972b05c.invoke(<generated>)
    	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
    	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:717)
    	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveM 省略一些栈追溯

---
# Quartz 实现定时任务

定义一个定时任务，需要 `Task`, `Job`, `Trigger`, `Scheduler` 4 个类.

其中 Task 是我们**自定义的类**，是任务的实现逻辑，另外 3 个类则是由 Spring 和 Quartz 提供的，在 Bean 的配置文件里配置即可。

## 1. pom.xml

Quartz 需要 spring-context-support，spring-tx 和 quartz 的依赖。

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>4.1.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.1.1.RELEASE</version>
</dependency>

<!-- Quartz framework -->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.2.1</version>
</dependency>
```

## 2. spring-beans.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--1. Task: 我们自定义的 Task 类-->
    <bean id="fooTask" class="com.xtuer.scheduler.FooTask"/>

    <!--2. Job-->
    <bean id="jobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
        <property name="targetObject" ref="fooTask"/> <!--自定义的 Task 类-->
        <property name="targetMethod" value="execute"/> <!--Task 的方法名-->
        
        <!--false表示等上一个任务执行完后再开启新的任务-->
        <property name="concurrent" value="false"/>
    </bean>

    <!--3. Trigger: run every 5 seconds-->
    <bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="jobDetail"/>
        <property name="cronExpression" value="0/5 * * * * ?"/>
    </bean>

    <!--4. Scheduler-->
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
            <list>
                <ref bean="cronTrigger"/>
            </list>
        </property>
    </bean>
</beans>
```
## 3. Task 类

Task 类就是一个普通的类，不需要继承或者实现其它任何的类或者接口。

```
package com.xtuer.scheduler;

import java.text.SimpleDateFormat;
import java.util.Date;

public class FooTask {
    private SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    public void execute() {
        System.out.println(format.format(new Date()));
    }
}
```

## 4. 测试


```
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        new ClassPathXmlApplicationContext("spring-beans.xml");
    }
}
```

## 输出

```
2015-06-19 08:16:10
2015-06-19 08:16:15
2015-06-19 08:16:20
2015-06-19 08:16:25
2015-06-19 08:16:30
...省略其他输出
```

## Cron 的语法
   
   
``` 
<!--3. Trigger: run every 5 seconds-->
    <bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="jobDetail"/>
        <property name="cronExpression" value="0/5 * * * * ?"/>
    </bean>
```
`Quartz-Cron` 的语法类似于 Linux 中的 Cron 的语法，但有些的区别，例如 ? 表达式中同时有日和周时，它们不能相同，所以用 ? 来区别开，不能同时为 * 等。 
```
秒 分 时 日 月 周 年
```

表达式 | 意义
------- | -------
0 0 12 * * ? | 每天中午12点触发
0 15 10 ? * * | 每天上午10:15触发
0 15 10 * * ? | 每天上午10:15触发
0 15 10 * * ? * | 每天上午10:15触发
0 15 10 * * ? 2005 | 2005年的每天上午10:15触发
0 * 14 * * ? | 在每天下午2点到下午2:59期间的每1分钟触发
0 0/5 14 * * ? | 在每天下午2点到下午2:55期间的每5分钟触发
0 0/5 14,18 * * ? | 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
0 0-5 14 * * ? | 在每天下午2点到下午2:05期间的每1分钟触发
0 10,44 14 ? 3 WED | 每年三月的星期三的下午2:10和2:44触发
0 15 10 ? * MON-FRI | 周一至周五的上午10:15触发
0 15 10 15 * ? | 每月15日上午10:15触发
0 15 10 L * ? | 每月最后一日的上午10:15触发
0 15 10 ? * 6L | 每月的最后一个星期五上午10:15触发
0 15 10 ? * 6L 2002-2005 | 2002年至2005年的每月的最后一个星期五上午10:15触发
0 15 10 ? * 6#3 | 每月的第三个星期五上午10:15触发
0 6 * * * | 每天早上6点
0 */2 * * * | 每两个小时
0 23-7/2，8 * * * | 晚上11点到早上8点之间每两个小时，早上八点
0 11 4 * 1-3 | 每个月的4号和每个礼拜的礼拜一到礼拜三的早上11点
0 4 1 1 * | 1月1日早上4点



## cronExpression中问号(?)的解释

http://blog.csdn.net/chh_jiang/article/details/4603529

1. 如官方文档解释的那样，`问号(?)`的作用是指明该字段‘**没有特定的值**’；

2. 星号(*)和其它值，比如数字，都是给该字段指明特定的值，只不过用星号(*)代表所有可能值；

3. `cronExpression` 对日期和星期字段的处理规则是它们必须互斥，即只能且必须有一个字段有特定的值，另一个字段必须是‘没有特定的值’；

4. `问号(?)`就是用来对日期和星期字段做**互斥**的。
 
#### 基于以上结论就可以解释下列情况：


- 当星期和日期都为`*`或数字时，报错
 `Support for specifying both a day-of-week AND a day-of-month parameter is not implemented.`

即两个字段不能都指明的特定的值，必须互斥。这里的`*`和数字是一样的，如果都指明特定的数字，也是报一样的错。

- 当星期和日期都为`?`时，报错 `'?' can only be specfied for Day-of-Month -OR- Day-of-Week.`

即两个字段不能都‘**没有特定的值**’。
 
这个是 Quartz 的实现，没有什么道理，Quartz 就是**规定**这两个字段必须这样互斥的设置。

**这与UNIX的crontab设置不一样**，crontab的规则是日期和星期中只要满足一个就触发，所以不存在互斥的问题。

---

