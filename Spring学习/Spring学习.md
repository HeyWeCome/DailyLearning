### 第一章 引言

#### 1.EJB存在的问题

![1595397660129](../pic/1595397660129.png)

#### 2.什么是Spring

```
1.Spring是一个轻量级的JavaEE解决方案，整合众多优秀的设计模式
```

- 轻量级

  ~~~markdown
  1. 对于运行环境是没有额外要求的
     开源 tomcat resion jetty
     收费 weblogic websphere
  2. 代码移植性
     不需要实现额外接口
  ~~~

- JavaEE的解决方案

![1595398481160](../pic/1595398481160.png)

- 整合设计模式

```markdown
1. 工厂
2. 代理
3. 模板
4. 策略
```

#### 3.设计模式

~~~ markdown
1. 广义概念
面向对象设计中，解决特定问题的经典代码
2. 狭义概念
GOF4人帮所定义的23种设计模式：工厂、适配器、装饰器、门面、代理、模板...
~~~

#### 4.工厂模式

##### 4.1 什么是工厂设计模式

~~~markdown
1. 概念：通过工厂类，创建对象
2. 好处：解耦合
   耦合：指的是代码间的强关联关系，一方的改变会影响到另一方
   问题：不利于代码维护
   简单理解：把接口的实现类，硬编码再程序中
   		   UserService userService = new UserServiceImpl();
~~~

##### 4.2 简单工厂的设计

>对象的创建方式：
>
>1. 直接调用构造方法 创建对象 UserService userService = new UserServiceImpl();
>
>2. 通过反射的形式创建对象 解耦合
>
>   Class clazz = Class.forName("com.xxx.xxx.UserServiceImpl");
>
>   UserService userService  = (UserService)clazz.newInstance();

```java
package com.baizhiedu.basic;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class BeanFactory {
    private static Properties env = new Properties();
    
    static{
        try {
            //第一步 获得IO输入流
            InputStream inputStream = BeanFactory.class.getResourceAsStream("/applicationContext.properties");
            //第二步 文件内容 封装 Properties集合中 key = userService value = com.baizhixx.UserServiceImpl
            env.load(inputStream);

            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
    
    /*
	   对象的创建方式：
	       1. 直接调用构造方法 创建对象  UserService userService = new UserServiceImpl();
	       2. 通过反射的形式 创建对象 解耦合
	       Class clazz = Class.forName("com.baizhiedu.basic.UserServiceImpl");
	       UserService userService = (UserService)clazz.newInstance();
     */

    public static UserService getUserService() {
        UserService userService = null;
        try {
            //com.baizhiedu.basic.UserServiceImpl
            Class clazz = Class.forName(env.getProperty("userService"));
            userService = (UserService) clazz.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return userService;
    }

    public static UserDAO getUserDAO(){
        UserDAO userDAO = null;
        try {
            Class clazz = Class.forName(env.getProperty("userDAO"));
            userDAO = (UserDAO) clazz.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return userDAO;
    }
}
```

 配置文件 applicationContext.properties： 

```xml
# Properties 集合 存储 Properties文件的内容
# 特殊Map key=String value=String
# Properties [userService = com.baizhiedu.xxx.UserServiceImpl]
# Properties.getProperty("userService")

userService = com.baizhiedu.basic.UserServiceImpl
userDAO = com.baizhiedu.basic.UserDAOImpl
```

##### 4.3 通用工厂的设计

- 问题

```markdown
简单工厂会存在大量的代码冗余
```

![1595575901690](../pic/1595575901690.png)

- 通用工厂的代码

```java
package com.baizhiedu.basic;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class BeanFactory {
    private static Properties env = new Properties();
    static{
        try {
            //第一步 获得IO输入流
            InputStream inputStream = BeanFactory.class.getResourceAsStream("/applicationContext.properties");
            //第二步 文件内容 封装 Properties集合中 key = userService value = com.baizhixx.UserServiceImpl
            env.load(inputStream);

            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
     /*
      key 小配置文件中的key [userDAO,userService]
      */
     public static Object getBean(String key){
         Object ret = null;
         try {
             Class clazz = Class.forName(env.getProperty(key));
             ret = clazz.newInstance();
         } catch (Exception e) {
            e.printStackTrace();
         }
         return ret;
     }
}
```

##### 4.4 通用工厂的使用方式

```markdown
1. 定义类型（类）
2. 通过配置文件的配置来告知工厂(applicationContext.properties)
   key = value
3. 通过工厂来获得类的对象
   Object ret = BeanFactory.getBean("key")
```

#### 5.总结

```markdown
Spring的本质：工厂 ApplicationContext 配置文件 applicationContext.xml
```

### 第二章 第一个Spring程序

#### 1.软件版本

```markdown
1. JDK1.8+
2. Maven3.5+
3. IDEA 2018+
4. SpringFramework 5.1.4
```

#### 2.环境搭建

- Spring的jar包

  ```markdown
  <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.14.RELEASE</version>
  </dependency>
  ```

- Spring的配置文件

```markdown
1. 配置文件的放置位置：任意位置 没有硬性要求
2. 配置文件的命名： 没有硬性要求 建议：applicationContext.xml

思考：日后应用Spring框架时，需要进行配置文件路径的设置。
```

![1595577457612](../pic/1595577457612.png)

#### 3.Spring的核心API

- ApplicationContext

  ```markdown
  作用：Spring提供的ApplicationContext这个工厂，用于对象的创建
  好处：解耦合
  ```

  - ApplicationContext接口类型

    ```markdown
    接⼝：屏蔽实现的差异
    ⾮web环境 ：ClassPathXmlApplicationContext(main junit 不启动服务器) 
    web环境 ：XmlWebApplicationContext（启动服务器）
    ```

    ![1595578294770](../pic/1595578294770.png)

  - 重量级资源

    ```markdown
    ApplicationContext工厂的对象占用大量内存
    不会频繁的创建对象： 一个应用只会创建一个工厂对象
    ApplicationContext工厂：一定是线程安全的（多线程并发访问）
    ```
```
    



#### 4.程序开发

​```markdown
1. 创建类型
2. 配置文件的配置 applicationContext.xml
3. 通过工厂类，获得对象
   ApplicationContext
   		Junit中 |- ClassPathXmlApplicationContext
   ApplicationContext ctx = new ClassPathXmlApplicationContent("applicationContext.xml");
   Person person = (Person)ctx.getBean("person"); // 键值对的方式来获取
```



#### 5.细节分析

- 名词解释

  ```markdown
  Spring ⼯⼚创建的对象，叫做 bean 或者 组件(componet)；
  ```

- Spring工厂的一些方法

```java
// getBean：传入 id值 和 类名 获取对象，不需要强制类型转换。
// 通过这种⽅式获得对象，就不需要强制类型转换
Person person = ctx.getBean("person", Person.class);
System.out.println("person = " + person);

// getBean：只指定类名，Spring 的配置文件中只能有一个 bean 是这个类型。
// 使用这种方式的话, 当前Spring的配置文件中 只能有一个bean class是Person类型
Person person = ctx.getBean(Person.class);
System.out.println("person = " + person);

// getBeanDefinitionNames：获取 Spring 配置文件中所有的 bean 标签的 id 值。
// 获取的是Spring工厂配置文件中所有bean标签的id值  person person1
String[] beanDefinitionNames = ctx.getBeanDefinitionNames();
for (String beanDefinitionName : beanDefinitionNames) {
	System.out.println("beanDefinitionName = " + beanDefinitionName);
}

// getBeanNamesForType：根据类型获得 Spring 配置文件中对应的 id 值。
// 根据类型获得Spring配置文件中对应的id值
String[] beanNamesForType = ctx.getBeanNamesForType(Person.class);
for (String id : beanNamesForType) {
	System.out.println("id = " + id);
}

// containsBeanDefinition：用于判断是否存在指定 id 值的 bean，不能判断 name 值
// 用于判断是否存在指定id值的bean,不能判断name值
if (ctx.containsBeanDefinition("person")) {
	System.out.println(true);
} else {
	System.out.println(false);
}

// containsBean：用于判断是否存在指定 id 值的 bean，也可以判断 name 值。
// 用于判断是否存在指定id值的bean,也可以判断name值
if (ctx.containsBean("person")) {
	System.out.println(true);
} else {
	System.out.println(false);
}
```

- 配置文件中需要注意的细节

  ```markdown
  1. 只配置class属性
  <bean class="Person">
a) 上述这种配置 有没有id值？ 答案：有，并且自动生成为Person#0
  b) 应用场景：如果这个bean只需要使用一次，那么就可以省略id值
  		   如果这个bean会使用多次，或者被其他 bean 引⽤则需要设置 id 值； 
  		   
  2. name属性
  作⽤：⽤于在 Spring 的配置⽂件中，为 bean 对象定义别名（小名）
  相同：
   	1.ctx.getBean("id") 或 ctx.getBean("name") 都可以获取对象；
      2.<bean id="person" class="Person"/>
        在定义方面等效于
        <bean name="person" class="Person"/>；
  区别：
  	1. 别名可以定义多个，但是id属性只能有一个值；（别名：<bean name="person,p1,p2"/>）
  	2. XML的id属性的值，命名要求：必须要以字母开头，可以包含字母、数字、下划线、连字符；不能以特殊字符开头/person；
  	   name属性的值，命名没有要求，可以设置成 /person的格式；
  	   name属性会应用在特殊命名的场景下：/person；
  	   
  	   XML发展到了今天：ID属性的限制已经不存在，/person也可以。
  	3. 代码
  	   containsBeanDefinition不能通过别名name来判断
  	   containsBean可以通过别名name来判断
  	   
  	   // 用于判断是否存在指定id值的bean,不能判断name值
          if (ctx.containsBeanDefinition("person")) {
              System.out.println(true);
          } else {
              System.out.println(false);
          }
          // 用于判断是否存在指定id值的bean,也可以判断name值
          if (ctx.containsBean("person")) {
              System.out.println(true);
          } else {
              System.out.println(false);
          }
  	   
  ```
  



#### 6.Spring工厂的底层实现原理（简易版）

**Spring工厂是可以调用对象私有的构造方法来创建对象，因为底层都是通过反射实现的**

![20200521002624493](../pic/20200521002624493.png)



#### 7.思考

```markdown
问题：未来在开发过程中，是不是所有的对象，都会交给 Spring ⼯⼚来创建呢？ 
回答：理论上是的，但是有特例 ：实体对象(entity) 是不会交给Spring创建，它由持久层框架进⾏创建。
```



### 第三章、Spring5.x与日志框架的整合

```markdown
Spring与日志框架进行整合，日志框架就可以在控制台中，输出Spring框架运行过程中的一些重要信息。
好处:便于了解Spring框架的运行过程，利于程序的调试。
```

- Spring如何整合日志框架

  ```markdown
  默认
  	Spring1.2.3早期都是与commons-logging.jar
  	Spring5.x默认整合的日志框架是 logback或者log4j2
  
  Spring5.x整合log4j
  	1. 引入log4j jar包
  	2. 引入log4.properties配置文件
  ```

  - pom

    ```xml
    // 日志门面，取消spring默认的日志，让Spring来采用log4j
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.25</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    ```

  - log4j.properties

    ```markdown
    # resources 文件夹根目录下
    ### 配置根
    log4j.rootLogger = debug,console
    
    ### 日志输出到控制台显示
    
    log4j.appender.console=org.apache.log4j.ConsoleAppender
    log4j.appender.console.Target=System.out
    log4j.appender.console.layout=org.apache.log4j.PatternLayout
    log4j.appender.console.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
    ```

    ![1595856819981](../pic/1595856819981.png)



### 第四章、注入（injection）

#### 1.什么是注入

```markdown
通过Spring工厂及配置文件，为所创建对象的成员变量赋值
```

##### 1.1 为什么需要注入

**通过编码的方式，为成员变量进行赋值，存在耦合**

![1596074770731](../pic/1596074770731.png)

##### 1.2 如何进行注入[开发步骤]

- 类的成员变量提供set get方法；

- 配置Spring的配置文件；

  ```xml
  <bean id="Person" class="com.Person">
  	<property name="id">
          <value>10</value>
      </property>
      <property name="name">
          <value>heywecome</value>
      </property>
  </bean>
  ```

##### 1.3 注入好处

```markdown
解耦合
```

#### 2. Spring注入的原理分析(简易版)

```markdown
<bean>相当于新建一个对象；
<property>相当于set方法赋值。
```

**Spring会通过底层调用对象属性对应的set方法，完成成员变量的赋值，这种方法我们也称之为set注入**

![1596179811562](../pic/1596179811562.png)



### 第五章、set方法详解

**set注入的类型**

![1596180652022](../pic/1596180652022.png)

~~~markdown
针对于不同类型的成员变量，在<property标签中，需要嵌套其他标签:
<property>
...
</property>
~~~



#### 1. JDK内置类型

##### 1.1 String+8种基本类型

~~~markdown
<value>hwk</value>
~~~

##### 1.2 数组

~~~markdown
<list>
	<value>kangkang</value>
	<value>heywecome</value>
	<value>hehe</value>
</list>
~~~

##### 1.3 Set集合

~~~xml
<set>
	<value>111111</value>
    <value>132111</value>
    <value>144111</value>
</set>

如果set里面没有指定类型，那么里面的value可以存储多个类型的数据
<set>
	<ref bean=""></ref>
    <set></set>
    <value>313213</value>
</set>
~~~

##### 1.4 List集合

~~~xml
<list>
	<value>eran</value>
    <value>nantong</value>
</list>

<list>
	<ref bean=""></ref>
    <set></set>
    <value>313213</value>
</list>
~~~

##### 1.5 Map

~~~xml
注意： map -- 一个键值对包含在一个entry里面 -- key有特定的标签 <key></key>
					  					 值根据对应类型选择类型的标签
<map>
    <entry>
        <key><value>kangkang</value></key>
        <value>21</value>
    </entry>
    <entry>
        <key><value>xiaohe</value></key>
        <ref bean=""></ref>
    </entry>
</map>
~~~

##### 1.6 Properties

~~~markdown
Properties类型，它是一个特殊的Map，它的Key是String类型，value也是String类型的。
每一个pro都是一个键值对
~~~

~~~xml
<pros>
	<pro key="key1">value1</pro>
    <pro key="key2">value2</pro>
</pros>
~~~

##### 1.7 复杂的JDK类型(Date)

~~~markdown
需要程序员自定义类型转化器处理。
~~~

#### 2. 用户自定义类型

##### 2.1 第一种方式

- 为成员变量提供set get方法

- 配置文件中进行注入(赋值)

  ~~~xml
  <bean id="userService" class="xxx.UserServiceImpl">
  	<property name="userDao">
      	<bean class="xxx.UserDaoImpl"></bean>
      </property>
  </bean>
  ~~~

##### 2.2 第二种方式

- 第一种方式存在的问题

  ~~~markdown
  1. 配置文件代码冗余
  2. 被注入的对象(UserDao)，多次创建，浪费(JVM)内存资源
  ~~~

- 为成员变量提供set get方法

- 配置文件中进行配置

  ~~~xml
  <bean id="userDAO" class="xxx.UserDAOImpl">
  </bean>
  
  <bean id="userService" class="xxx.UserServiceImpl">
      <!-- property中的name为UserServiceImpl中的成员变量名 -->
  	<property name="userDAO">
          <!-- ref的userDao则是上面定义的bean -->
      	<ref bean="userDAO"/>
      </property>
  </bean>
  
  # Spring4.x废除了 <ref local=""/> 基本等效于 <ref bean=""/>
  ~~~

  

#### 3. Set注入的简化写法

##### 3.1 基于属性简化

~~~xml
JDK类型注入
<property name="name">
	<value>kangkang</value>
</property>

简化之后
<property name="name" value="kangkang"></property>
注意：
	value属性 只能简化 8种基本类型+String 注入标签

用户自定义类型
<property name="userDAO">
	<ref bean="userDAO"/>
</property>
简化：
<property name="userDAO" ref="userDAO"/>
~~~

##### 3.2 基于p命名空间简化

~~~xml
JDK类型注入
<bean id="person" class="xxx.Person">
    <property name="name">
        <value>kangkang</value>
    </property>
</bean>

简化之后
<bean id="" name="" p:name="suns"/>
注意：
	只能简化 8种基本类型+String 注入标签

用户自定义类型
<bean id="userService" class="xxx.UserServiceImpl">
    <property name="userDAO">
        <ref bean="userDAO"/>
    </property>
</bean>

简化：
<bean id="userService" class="xxx.UserServiceImpl" p:userDAO-ref="userDAO"/>
~~~

### 第六章、构造注入

~~~markdown
注入：通过Spring的配置文件，为成员变量赋值
Set注入：Spring调用Set方法，通过配置文件，为成员变量赋值
构造注入：Spring调用构造方法，通过配置文件，为成员变量赋值
~~~

#### 1.开发步骤

- 类提供有参构造方法；

  ~~~java
  public class Customer {
      private String name;
      private int age;
  
      public Customer(String name, int age) {
          this.name = name;
          this.age = age;
      }
  
      @Override
      public String toString() {
          return "Customer{" +
                  "name='" + name + '\'' +
                  ", age=" + age +
                  '}';
      }
  }
  ~~~

  

- Spring的配置文件

  ~~~xml
  # constructor-arg的个数和顺序都要和构造方法中的参数一致
  <bean id="customer" class="com.yusael.constructor.Customer">
      <constructor-arg>
          <value>kangkang</value>
      </constructor-arg>
      <constructor-arg>
          <value>21</value>
      </constructor-arg>
  </bean>
  ~~~

#### 2.构造方法重载

重载：个数不同、类型不同、顺序不同。不存在类型一致的相同构造方法，会报错的

##### 2.1 参数个数不同时

~~~markdown
通过控制<constructor-arg>标签的数量来进行区分

如果只有一个参数的话，只需要一对  <constructor-arg> 标签： 
<bean id="customer" class="com.constructor.Customer">
    <constructor-arg>
        <value>kangkang</value>
    </constructor-arg>
</bean>

如果有两个参数的话，用两对 <constructor-arg> 标签，以此类推。
<bean id="customer" class="com.constructor.Customer">
    <constructor-arg>
        <value>kangkang</value>
    </constructor-arg>
    <constructor-arg>
        <value>22</value>
    </constructor-arg>
</bean>
~~~

##### 2.2构造参数个数相同时

~~~markdown
通过在标签中引入 type属性 进行类型的区分 <constructor-arg type="">

<bean id="customer" class="com.constructor.Customer">
	<constructor-arg type="int">
	    <value>20</value>
	</constructor-arg>
</bean>
~~~

#### 3.注入的总结

~~~markdown
未来的实战中，应⽤ set注入 还是 构造注入？

答：set 注入更多。

1. 构造注入麻烦（重载）
2. Spring 框架底层⼤量应⽤了 set注入。
~~~

 ![注入](../pic/32133123232) 



### 第七章、反转控制与依赖注入

#### 1.反转(转移)控制(IOC Inverse of control)

~~~markdown
控制：对于成员变量的控制权
不用Spring：直接在代码中，完成对于成员变量的赋值，对于成员变量赋值的控制权是由代码控制的，存在着耦合；
使用Spring：对于成员变量复制的控制权 = Spring配置文件 + Spring工厂，解耦合；

面试重点：
所谓反转控制，即：对于成员变量赋值的控制权 由代码反转(转移)到Spring工厂和配置文件中完成。
好处：解耦合
底层实现：工厂设计模式
~~~

#### 2.依赖注入(DI Dependency injection)

~~~markdown
注入：通过Spring的工厂及配置文件，为对象（bean，组件）的成员变量赋值
依赖通俗点来讲，就是我需要用到你，那我就是依赖你

依赖注入：当一个类需要另一个类时，就意味着依赖，一旦出现依赖，就可以把另一个类作为本类的成员变量，最终通过Spring配置文件进行注入(赋值)。

好处：解耦合
~~~

![1596764428104](../pic/1596764428104.png)



### 第八章、Spring工厂创建复杂对象

![1596959602727](../pic/1596959602727.png)

#### 1.什么是复杂对象

复杂对象：指的就是不能直接通过new方法创建的对象

#### 2.Spring工厂创建复杂对象的3种方式

##### 2.1 FactoryBean接口

- 开发步骤

  - 实现FactoryBean接口

    ![1596960300778](../pic/1596960300778.png)

    ~~~java
    public class ConnectionFactoryBean implements FactoryBean<Connection> {
        // 用于书写创建复杂对象时的代码
        @Override
        public Connection getObject() throws Exception {
            Class.forName("com.mysql.jdbc.Driver");
            Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/spring", "root", "1234");
            return conn;
        }
    
        // 返回创建的复杂对象的类型
        @Override
        public Class<Connection> getObjectType() {
            return Connection.class;
        }
    
        // 是否单例
        @Override
        public boolean isSingleton() {
            return false; // 每一次都创建新的复杂对象
            // return true; // 只创建一次这种类型的复杂对象
        }
    }
    
    ~~~

  - Spring配置文件的配置

    ~~~xml
    如果class中指定的类型 是FactoryBean接口的实现类，那么通过ID值获得的是这个类所创建的复杂对象 Connection，而不是ConnectionFactoryBean
    比如下面 class 指定的是 ConnectionFactoryBean，获得的是 Connection 对象。
    <bean id="conn" class="com.factorybean ConnectionFactoryBean"/>
    ~~~

- 细节

  - 如果就想获得FactoryBean类型的对象，使用ctx.getBean("&conn")，这样获得的就是ConnectionFactoryBean

    ![1596961188494](../pic/1596961188494.png)

  - isSingleton方法

    返回 true 的时候 只会创建一个复杂对象

    返回 false 的时候 每一次都会创建新的对象

    我们需要根据这个对象的特点，决定是返回 true(SqlSessionFactory) 还是 false (Connection)

    比如连接对象Connection，每次都需要新的连接，所有返回false
    
  - mysql高版本连接创建时，需要制定SSL证书，解决问题的方式
  
    ~~~markdown
    url = "jdbc:mysql://localhost:3306/suns?userSSL=false"
    ~~~
  
  - 依赖注入的体会(DI)
  
    ~~~markdown
    把 ConnectionFactoryBean 中依赖的 4 个字符串信息 ，通过配置⽂件进行注⼊。
    ~~~
  
    ~~~java
    @Getter@Setter // 提供 get set 方法
    public class ConnectionFactoryBean implements FactoryBean<Connection> {
    	// 将依赖的字符串信息变为成员变量, 利用配置文件进行注入。
        private String driverClassName;
        private String url;
        private String username;
        private String password;
        @Override
        public Connection getObject() throws Exception {
            Class.forName(driverClassName);
            Connection conn = DriverManager.getConnection(url, username, password);
            return conn;
        }
        @Override
        public Class<Connection> getObjectType() {
            return Connection.class;
        }
        @Override
        public boolean isSingleton() {
      		return false;
        }
    }
    ~~~
  
    ~~~xml
    <!--体会依赖注入, 好处: 解耦合, 今后要修改连接数据库的信息只需要修改配置文件, 无需改动代码-->
    <bean id="conn" class="com.yusael.factorybean.ConnectionFactoryBean">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring?useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="1234"/>
    </bean>
    ~~~

- FactoryBean实现原理[简易版]

   原理：**接口回调**。 

  ~~~xml
  问题：
  1. 为什么 Spring 规定 FactoryBean 接⼝实现 getObject()？
  2. 为什么 ctx.getBean("conn") 获得的是复杂对象 Connection ⽽非 ConnectionFactoryBean？
  
  Spring 内部运行流程：
  1. 配置文件中通过 id conn 获得 ConnectionFactoryBean 类的对象 ，进而通过 instanceof 判断出是 FactoryBean 接⼝的实现类；
  2. Spring 按照规定 getObject() —> Connection；
  3. 返回 Connection；
  ~~~

   ![在这里插入图片描述](../pic/3231221321) 

- FactoryBean总结

  Spring 中⽤于创建复杂对象的⼀种方式，也是 Spring **原⽣提供的**，后续 Spring 整合其他框架时会⼤量应⽤ FactoryBean 方式。 

##### 2.1 实例工厂

~~~markdown
1. 避免Spring框架的侵入
2. 整合遗留系统 
~~~



##### 2.2 静态工厂

- 开发步骤

  - StaticConnectionFactory类

    ~~~java
    public class StaticFactoryBean {
    	// 静态方法
        public static Connection getConnection() {
            Connection conn = null;
            try {
                Class.forName("com.mysql.jdbc.Driver");
                conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/spring?useSSL=false", "root", "1234");
            } catch (ClassNotFoundException | SQLException e) {
                e.printStackTrace();
            }
            return conn;
        }
    }
    ~~~

  - 配置文件

    ~~~xml
    <!--静态工厂-->
    <bean id="conn" class="com.yusael.factorybean.StaticFactoryBean" factory-method="getConnection"/>
    ~~~

#### 3. Spring工厂创建对象的次数

![7213987](E:\Workspace\学习日常\DailyLearning\pic\7213987.png)





### 第九章、控制Spring工厂创建对象的次数

#### 1. 如何控制简单对象的创建次数

配置文件中进行配置：

~~~xml
<!--控制简单对象创建次数-->
<bean id="scope" scope="singleton" class="com.scope.Scope"/>

singleton：只会创建一次简单对象，为默认值；（默认单例模式）
prototype：每一次都会创建新的对象；
~~~

#### 2.如何控制复杂对象的创建次数

~~~java
public class xxxFactoryBean implements FactoryBean {
	public boolean isSingleton() {
		return true; // 只会创建⼀次
		// return false; // 每⼀次都会创建新的
	}
	// 省略其余实现方法......
}

如果是实例工厂或者静态工厂，没有 isSingleton ⽅法，还是通过 scope 控制对象的创建次数控制。
~~~

#### 3.为什么要控制对象的创建次数

~~~markdown
最大的好处：节省不必要的内存浪费
~~~

- 什么样的对象只创建一次？

  ~~~markdown
  1. SqlSessionFactory：很占内存，创建一次就够了
  2. DAO
  3. Service
  ~~~

- 什么样的对象 每一次都要创建新的？

  ~~~markdown
  1. Connection：连接事务，不会被共用；
  2. SqlSession：封装了连接，不能被共用；
  3. Session
  4. Strusts2中的Action
  ~~~

  **不能被共用，线程不安全，就需要每一次都创建新的。**

### 第十章、对象的生命周期

#### 1. 什么是对象的生命周期

~~~markdown
指的是一个对象创建、存活、消亡的一个完整过程
~~~

#### 2. 为什么要学习对象的生命周期

~~~markdown
由Spring来负责对象的创建、存活、销毁，了结生命周期，有利于我们使用好Spring为我们创建的对象
~~~

#### 3. 生命周期的3个阶段

- 创建阶段

  ~~~markdown
  Spring工厂何时创建对象
  ~~~

  - scope=“singleton”

    ~~~markdown
    Spring工厂创建的同时，对象的创建
    
    注意：通过配置 <bean lazy-init="true"/> 也可以实现获取对象的同时，创建对象。
    
    也就是饿汉模式变成了懒汉模式
    ~~~

  - scope="prototype"

    ~~~markdown
    Spring工厂会在活区对象的同时，创建对象
    ctx.getBean("")：这就是活区对象
    ~~~

  **由此可见，饿汉和懒汉都是单例模式**

- 初始化阶段

  ~~~markdown
  Spring工厂在创建完对象后，调用对象的初始化方法，完成对应的初始化操作
  
  1. 初始化方法提供：程序员根据需求，提供初始化方法，最终完成初始化操作。
  2. 初始化方法调用：Spring工厂进行调用
  ~~~

  定义初始化方法，有两种途径

  - initializingBean接口

    ~~~markdown
    afterProperitesSet()
    ~~~

    ~~~java
    public class Product implements InitializingBean {
        public Product() {
            System.out.println("Product.Product");
        }
        
        // 这个就是初始化方法：做一些初始化操作
        // Spring会进行调用
        @Override
        public void afterPropertiesSet() throws Exception{
            sout("");
        }
    }
    ~~~

  - 对象中提供一个普通的方法

    ~~~Java
    public void myInit(){
        
    }
    
    如何让spring知道我提供了这个方法呢？
        <bean id="product" class="xxx.Product" init-method="myInit"/>
    ~~~

    代码演示：

    ~~~java
    public class Product implements InitializingBean {
        public Product() {
            System.out.println("Product.Product");
        }
        
        public void myInit(){
        	sout("product.myInit");
    	}
    }
    ~~~

    配置文件：

    ~~~xml
    <bean id="product"> class="com.life.Product" init-method="myInit" />
    ~~~

  - 细节分析

    1. 如果⼀个对象既实现 `InitializingBean` 同时⼜提供的 普通的初始化方法，他们的执行顺序?

       回答：先执行 `InitializingBean`，再执行普通初始化方法。 两者都执行。

    2. 注入一定发生在初始化操作的前面

       **Spring在创建完对象之后，首先进行注入(DI)，然后再初始化(init)**

    3. 什么叫做初始化操作

       ~~~markdown
       资源的初始化：数据库 IO 网络 ...
       ~~~

- 销毁阶段

  ~~~markdown
  Spring销毁对象钱，会调用对象的销毁方法，完成销毁操作
  
  1. Spring什么时候销毁所创建的对象？
  	ctx.close();
  2. 销毁方法：程序员根据自己的需求，定义销毁方法，完成销毁操作
  	调用：Spring工厂完成调用
  ~~~

  - DisposableBean

    ~~~java
    public class Product implements DisposableBean {
        // 程序员根据⾃⼰的需求, 定义销毁方法, 完成销毁操作
        @Override
        public void destroy() throws Exception {
            System.out.println("Product.destroy");
        }
    }
    
    ~~~

  - 定义一个普通的销毁方法 ，配置文件种配置 `destroy-method`

    ~~~java
    public class Product {
    	// 程序员根据⾃⼰的需求, 定义销毁方法, 完成销毁操作
        public void myDestory() {
            System.out.println("Product.myDestory");
        }
    }
    ~~~

    ~~~xml
    <bean id="product" class="com.life.Product" destroy-method="myDestory"/>
    ~~~

  - 细节分析

    ~~~markdown
    1. 销毁方法的操作只适用于 scope="singleton"
    2. 什么叫做销毁操作
       主要指的就是 资源的释放操作 io.close() connection.close();
    ~~~

    

#### 4. 对象的生命周期总结

~~~java
public class Product implements InitializingBean, DisposableBean {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("Product.setName");
        this.name = name;
    }

    Product() { // 创建
        System.out.println("Product.Product");
    }

    // 程序员根据需求实现的方法, 完成初始化操作
    public void myInit() {
        System.out.println("Product.myInit");
    }

    // 程序员根据需求实现的方法, 完成初始化操作
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Product.afterPropertiesSet");
    }

    public void myDestory() {
        System.out.println("Product.myDestory");
    }

    // 程序员根据⾃⼰的需求, 定义销毁方法, 完成销毁操作
    @Override
    public void destroy() throws Exception {
        System.out.println("Product.destroy");
    }
}
~~~

~~~xml
<bean id="product" class="com.life.Product" init-method="myInit" destroy-method="myDestory">
	<property name="name" value="kangkang"/>
</bean>

~~~

![1600089479799](../pic/1600089479799.png)