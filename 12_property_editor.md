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


