---
title: '初识Spring'
date: 2024-09-16T18:55:20+08:00
draft: false
layout: "post"
images: 
  - "/avatar.jpg"   # 设置封面图
description: ""     
categories: ["技术原理"]
tags: ["原理"]
---

# Spring

背景：现代软件开发中，各个模块依赖程度太高，高耦合度带来的是很致命的问题，需要将各个模块进行解耦。IOC是Inversion of Control的缩写，”控制反转“。将复杂系统分解成相互合作的对象，**将接口的实现类交给容器，由容器给予我们相对应的实现类。**高内聚，低耦合。

## IOC容器基础

有一个Spring IOC容器实例化，组装管理的对象就称作为**Bean**。实际上就是编写配置文件，让IOC容器代理通过反射机制创建一个新的对象，而不是直接new出一个新的对象。需要注意的是，在配置文件中的每一个Bean**都对应了一个单独的对象**，当多次调用同一个name的Bean时，得到的都是同一个对象。**默认为单例模式**。

可以通过修改成为从 **（原型模式）**。

```xml
<bean class="com.Student" scope="prototype"/>
```

当bean的作用域为单例模式**singleton**时，是在容器一开始加载的时候对象就已经被创建了，之后取对象时只取得这一个，可以通过设置懒加载开启：取消一开始就加载创建对象。

```xml
<bean class="com.Student" scope="prototype" lazy-init="true"/>
```

同时，需要维护Bean的加载顺序，对象的创建顺序：

```xml
<bean class="com.Student" scope="prototype" depend-on="teacher"/>
<bean class="com.Teacher name="teacher" scope="prototype"/>
```

### Core Container

Spring框架中最核心的一个模块。

### 依赖注入

当一个类内部需要依赖另一个类的创建时，Student中的Teacher成员变量，应将容器配置为在创建新的Student对象时，根据配置文件的需求，**将需要的成员变量类注入到新的Student对象中**。

向bean对象中添加相应的属性（对应的就是成员变量对象）。**要使用依赖注入，必须在该类中添加set成员变量方法**

```xml
<bean class="com.Teacher name="teacher" scope="prototype"/>
<bean class="com.Student"/>
	<property name="teacher" ref="artTeacher"/> // ref ： 对另外一个对象的引用
    <property name="sex" value=“female"> // value成员变量为常量时
</bean>
```

其实就是依靠容器，将具体实现类的信息全都交给配置文件，让容器动态的进行修改对象的各种属性,再对对象进行创建。

bean在创建对象时，默认的调用无参构造方法，当修改了默认的无参数构造方法时，需要在配置文件中手动的进行定义。

```xml
<bean class="com.Student"/>
	<constructor-arg name="teacher" ref="artTeacher"/> // ref ： 对另外一个对象的引用
	<constructor-arg name="sex" value=“female"/> // value成员变量为常量时
</bean>
```

通过这样设置构造函数的选择，将依赖注入的时间调整到了在构造对象调用构造方法的时候。

有时当name冲突时，需要特殊的具体的指定类型。

```xml
<bean class="com.Student"/>
	<property name="teacher" ref="artTeacher"/> 
    <property name="teacher" value=“female" type="String">
</bean>
```

同时如果需要注入集合类型对象，也可以加入相对应的初始值。

```xml
 <bean name="student" class="org.spring.entity.Student">
        <property name="list">
            <list>
                <value>1</value>
                <value>2</value>
                <value>3</value>
            </list>
        </property>
    </bean>
```

所以，一共有两种方式来对对象进行依赖注入：

- 使用Set方法
- 使用对象的构造方法

### 自动装配

通过自动装配，让容器自动去寻找需要进行装配的值，省去了编写**property**的步骤。还是需要设定Setter方法。开启自动装配：

```xml
<bean class="com.Student" autowire="byType"/>
<bean class="com.Teacher"/>
```

使用Byname属性。这里的name属性对应的是setter方法中的name。

同时设定setter方法的时候`setArt`,容器才能自动找到相对应的名字进行装配。

```xml
<bean class="com.Student" autowire="byName"   />
<bean name="art" class="com.Teacher"/>
```

构造方法的自动装配（这种情况默认也是byType寻找的）。`autowire="constructor"`

自动装配时，bean存在一个候选名单，当byType出现冲突的时候，也就是存在两个候选Bean的时候，可以设置`autowire-candidate=false`,将该对应的bean剔除候选名单。`primary="true"`,设置bean的优先级。

### Bean的生命周期与继承

`init-method destory-method`两种属性设置Bean默认的初始化方法，和销毁方法。后者方法，当调用`context.close()`时，会调用`destory-method`,并且其中为单例模式的bean也会被销毁，且执行销毁方法。出于原型模式的bean，并不会执行其销毁方法。

Bean之间的继承关系是Bean之间的属性继承。`parent`同时可以在最外层的beans标签中添加全局的默认属性。

### Bean工厂模式

```xml	
<bean factory-bean="studentFactory" factory-method="getStudent"/>
```

使用工厂bean之后不需要在提供指定的class了。如果想要获取工厂bean提供给我们的bean也就是学生只需要：

```java
Student bean = (Student) context.getBen("studentFactory");
```

如果想要获取到工厂类的实例：

```java
Student bean = (Student) context.getBen("&studentFactory");
```

### 使用注解开发

使用注解完成对配置文件的配置。使用注解开发，现在就只需要创建一个配置类就行了（等同于配置文件的开头的配置文件模板）。

```java
@Configuration
@Import(LBWconfiguration.class) // 通过import引入其他的配置类
public class MainConfiguration() {
    @Bean("test") // name 
    @Lazy
    @Scope()
    public Student Student() {
        return new Student;
    }
}
```

- `@autowire`采用默认的byType的方式来进行匹配。`@Qualifier`用来使用byName模式来进行自动装配。

- `@PostConsturct` `@PreDestroy` === `init-method` `destroy-method`

- `@Resource` 默认的是使用`Byname`的形式来进行匹配 ，找不到则使用`byType``@AutoWire`则是只能使用byType方式匹配。同时后者不建议加在字段之上，前者可以加在字段上。
- `@Component`直接加在响应的类之上，将类变成Bean`@ComponentScan`手动告诉配置类需要进行扫描的包名。
- 注意：注解中的数组表示{}

加上注解之后，默认的会赋予Bean名字（首字母小写）

## Spring高级特性

### Bean Aware

一种接口，Aware，就是可以在创建对象之前，先“aware"到加载阶段的东西，可以提前获得类的各种属性。

### 任务调度

spring所提供的能够简化类似于多线程工作环境下的任务处理。

- `EnableSync` `EnableAsync`开启异步同步特性。 Example：异步和同步的实现：`Sync` `Async`
  - `@Scheduled`开启定时任务 

## SpringEL表达式

###  外部属性的注入

从外部的properties配置文件中得到属性。

```java
 ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression("'Hello World'.length");
        System.out.println(exp.getValue());
```

其实就是将java表达式作为字符串进入表达式进行执行，同时也可以接受对象，并对对象的属性进行设置或者获取 ，其中也可以写类似与python，C++等语言的对集合元素的控制方式，增大了对集合元素代码的掌控.

## AOP(Aspect Oriented Programming)

不修改源代码的情况下，给添加更多的内容。将整个类给切开，在需要的地方作为切入点，从切入点加入需要加入的东西。AOP是基于动态代理来实现的，所以经过切入之后，现在的类已经不是之前的类了。 

### 使用接口实现AOP

与配置文件大同小异，需要切入类需要实现对应的接口，来调整切入的时间。 

**通过给切片方法添加JoinPoint的参数获取原来类的方法的参数**

- `@Before` `@After` `@Around` 
- around方式，将方法给整体给进行代理。

## 数据框架的整合

将Spring和`Mybatis`框架进行整合。

### 数据源

`Mybatis`并不是通过`DriverManager`的方法来获取连接。而是使用`DataSource`的实现类来进行获取的。和数据库连接是一个极其耗费资源的操作，`DataSource`中有很多个`Connection`对象，也就形成了一个连接池，其中包含了很多个`Connection`对象，想要进行连接的时候，直接返回现成的连接对象。**连接池**，池化的数据源，内部是根据栈的数据结构来进行实现的

### 整合Mybatis框架

需要配置文件的方法，省去了自己配写工具类。

```java
 public SqlSessionTemplate sqlSessionTemplate() throws IOException {
       	SqlSessionFactory factory = new 				
       	SqlSessionFactoryBuilder()
            .build(Resources.getResourceAsStream("mybatis-config.xml"));
        return new SqlSessionTemplate(factory); // 返回一个Sqlsesion模板交给Spring进行管理
    }
```

在配置中自己创建一`DataSource`并且创建`Bean`进行管理，提供`SqlSessionFactoryBean`,此时数据库就可以正常根据`mapper`来运行了。

```java	
@Configuration
@MapperScan("org.spring.mapper")
public class MainConfiguration {

    @Bean // 新建一个Bean，方便后续替换 这个默认的数据源是由JDBC官方默认提供的，还有着很多的优秀的数据源
    public DataSource dataSource() {
        return new PooledDataSource("com.mysql.cj.jdbc.Driver", "jdbc:mysql://localhost:3306/study", "root", " ");
    }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean;
    }

}
```

###  使用HikariCP连接池

还有着很多的不同的数据源实现。使用方法原理都是相通的

```java
 @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/study");
        dataSource.setUsername("root");
        dataSource.setPassword(" ");
        return dataSource;
    }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean;
    }
```

### Mybatis事务管理

- (Atomicity):事务是一个原子操作，应该保证动作全部完成或者完全不起作用。
- (Consistency):一致性不能一个部分成功或者一个部分失败。
- (Isolation)：在很多个事务处理相同的数据的时候，每个事务应该和其他事务隔离开了，每个人看到的结果应该都是独立的。
- (Durability):持久性

并发执行下，没有并发控制，会出现的三种情况：

- 脏读：读取到了一个事务提交过后的数据，但最后该事务又进行了回滚的操作，读取到了没有意义的数据。
- 虚读（不可重复读）：是在某一个事务对数据进行修改之前读取一次，修改完毕之后再次读取一次，导致两次读取的数据不一样的情况。
- 幻读：由于在该事务执行过程中，无法感知到其他事务强行插入删除数据记录，导致数据整个表的数据记录数量前后不一致的情况。

**虚读和幻读：虚读是对某个单个数据前后读取不一致，幻读则是对整个表的记录数量前后读取不一致**。

![nHfV8R1ZUybTSd2.png (342×202)](https://image.itbaima.cn/markdown/2022/12/17/nHfV8R1ZUybTSd2.png)

事务的操作分为：创建，提交，回滚，关闭。而这些四个操作则被Mybatis抽象为一个接口 `Transaction`。Mybatis对事务的管理有着两种实现方式，也就是两种种不同的实现这个接口的方式：

- `JDBC`使用JDBC的事务管理机制，利用对应的数据库（数据源）驱动生成的对象`connection`
- `Manage`管理机制，将事务管理交给类似于Spring的框架。

### Spring事务管理

```java
@Component
public class TestServiceImpl implements TestService {

    @Autowired
    TestMapper mapper;

    @Transactional // 将该操作作为事务进行执行
    @Override
    public void test() {
        mapper.insertStudent("小红", "男");
        mapper.insertStudent("小君", "女");
    }
}
```

## J Unit集合测试

Spring框架提供了一个Test模块，可以自动继承JUnit进行测试。依赖：

```xml
  <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.11.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>6.1.12</version>
        </dependency>
```

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = MainConfiguration.class)
public class MainTest {

    @Autowired // 
    TestService testService;

    @Test
    public void test() {
        testService.test();
    }
}

```

## `@Resource` and `@autowired`对比

`@Resource`是`Java EE`中的一个注解，**根据名字自动进行装配依赖**，首先根据类名字寻找有无管理的依赖来进行注入，如果没有再根据类型来选择依赖进行注入。

`@autowired`则是Spring中提供的根据类型，进行注入依赖的注解。显得更加灵活

`Interceptor`中依赖注入问题，GPT原文如下：

> - **`HandlerInterceptor` 的生命周期问题：** `HandlerInterceptor` 是用于拦截器链的对象。Spring Web 配置中通常会手动注册拦截器，而拦截器实例的管理方式和普通的 Spring Bean 不完全相同。**具体来说，`HandlerInterceptor` 的实例是由 Spring 在请求处理时创建的，而不是由 Spring 容器管理的普通 Bean**。这样，当你尝试在 `Interceptor` 中使用 `@Resource` 注入时，Spring 可能不会正确地完成依赖注入，尤其是在没有显式注册拦截器 Bean 的情况下。
> - **构造函数注入 vs 字段注入：** 如果你用 `@Component` 注册了 `LoginInterceptor`，Spring 会创建一个默认的无参数构造函数实例，但它并不总是正确地将 `@Resource` 或 `@Autowired` 注解的依赖注入到拦截器中，特别是当拦截器是在 Web 配置中手动注册时**。通常，Spring 无法通过字段注入来处理 `Interceptor`，因为它没有机会管理依赖的生命周期。**

所以，因为`interceptor`应该算是一个特殊的管理的Bean，通常不使用字段注入，而是使用构造函数注入，且在配置类中进行注入所需依赖之后，传递给`interceptor`。

