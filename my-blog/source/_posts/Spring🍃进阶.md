---
title: Spring🍃进阶
date: 2022/10/2 01:54:28
updated: 2023/3/23 22:54:42
categories:
- BackEnd
tags:
- Spring
---

# Spring的特性🍃

非侵入式、控制反转、依赖注入、面向切面编程、容器、组件化

# Spring的组成

![image-20221002003328265](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20221002003328265.png)

> Data Access、Web、AOP、Aspects、Instrumentation、Messageing是Spring的模块，是对Spring的扩充。

> Spring的核心是Core Container

## CoreContainer（核心容器）

- **Beans**：提供对bean创建、配置、管理等功能支持。
- **Core**：核心工具类。
- **Context**：提供国际化、事件传播、资源加载等功能支持。
- **SpEL**：Spring-experssion，提供对于表达式语言的支持

## Data Access/Integration（数据访问/集成）

- **JDBC**：封装了数据库操作的api JDBC，提供jdbcTemplate，简化操作数据库。
- **ORM**：提供对HiBernate、JPA、MyBatis等框架支持。ORM是对象关系模型的意思。
- **OXM**：Object /XML，用于Java与XML之间交互。
- **JMS**：消息队列
- **Transactions**：对事务的支持

## Web

- **Web**：提供Web开发基础支持
- **Servlet**：提供SpringMVC Web框架事件。
- **WebSocket**：提供WebSocket支持，实现客户端服务器双向通讯。
- **Webflux**：Spring5新增的响应式Web框架，不需要Servlet API，完全异步。

## AOP、Aspects、Instrumentation、Messaging

- **AOP**：面向切面编程的实现。
- **Aspects**：提供与AspectJ的集成。AspectJ是一个AOP框架。
- **Instrumentation**：提供类工具和类加载器的实现。
- **Messaging**：消息队列
- **Jcl**：日志

>  Q1：Spring、SpringMVC、Springboot之间的关系
>
> Spring包含多个功能模块，其中核心是IOC和AOP，SpringMVC是Spring的一个模块，通过SpringMVC可以快速构建MVC架构的Web应用程序。MVC是modle、view、controller的缩写，其核心思想是将业务逻辑、数据保存、展示、分类开。SpringMVC在开发时需要进行很多的配置。SpringBoot简化了这些配置，可以快速的构建应用程序。

# Spring IOC

> Q2：什么是IOC
>
> Ioc是控制反转的意思，是一种设计思想。在原本的应用程序中，如果需要使用类的方法，就需要先创建对象。在开发时经常需要手动通过new关键字来创建对象。Ioc的思想让我们不用在自己手动去创建对象。有Spring框架来创建对象、管理对象之间的依赖关系。原本控制权在开发中手中，现在开发者将控制权交给Spring框架，就是控制反转。

1. 有用户管理Bean到由Spring框架管理Bean，叫控制反转。
2. Spring框架管理的Bean被放在Ioc容器中。
3. 程序从Ioc容器中获取Bean，在诸如到程序中，叫依赖注入。
4. 注入的方式，构造器方式，代码上体现在@Autowired、@Resource、@Qualifier等

## Ioc是什么

Ioc—Inversion of Control，控制反转。是一种设计思想，体现在Java中是指，将设计好的对象交给容器处理，而不是传统的在对象内部直接控制。

- **Control**：传统程序设计时，在一个对象内部，通过new关键字创建另一个对象，是由程序主动创建并注入对象。Ioc有一个容器专门控制对对象的创建。
- **Inversion**：有我们自己在对象中主动创建、注入，这是正转。由容器来创建注入对象，这是反转。具体时对象的获取被反转了，由原来的主动获取变成由容器先获取，再从容器获取。

对象之间的依赖注入例如，controller调用service，service调用mapper，如果没有Ioc，想要在controller调用service方法，需要先创建mapper，在将mapper注入到serivce，才能调用到service方法。如果有Ioc，直接从Ioc容器中获取service方法就行了，Ioc容器已经将mapper注入到serivce里了。

## Ioc的优势

传统的程序由我们在类内部主动创建依赖对象，这样导致类与类之间耦合度很高，不符合高内聚低耦合的理念。如果将创建管理对象交给Ioc容器完成，由容器完成创建和注入，这样降低了耦合。简化了代码，方便代码复用。

## Ioc和DI

> 控制反转和依赖注入。控制反转是通过依赖注入实现的，Ioc是设计思想，DI是实现方式。

# 依赖注入的方式

### setter()

传统XML注入方式，通过`<property>`标签给类注入属性是通过`setter()`方法注入的

### 构造函数

传统XML注入方式，通过`<constructor-arg>`标签注入属性是通过调用被注入类的构造函数

### 注解

`@Autowired`，可选参数Constructor、byType、byName。默认使用byType，推荐使用构造器注入。

- **Constructor**：Spring匹配与构造方法参数类型一致的bean注入。
- **byName**：被注入的bean的名称与setter方法名相同。
- **byType**：找到符合参数类型的set方法注入。

> Q3：为什么推荐使用构造器注入
>
> 构造器注入能保证注入的组件不可变，保证所需要的依赖不为空，保证完全初始化状态。
>
> - **依赖不可变**：由final修饰
> - **依赖不为空**：由spring向对象中传入参数
> - **完全初始化状态**：向对象中传递参数时，保证所传的参数不为空。调用构造方法在类加载实例化中是最后一步，所以返回的是初始化状态

# 注解

> 将一个类申明为Bean的注解
>
> - @Component
> - @Repository
> - @Service
> - @Controller
> - @Resource
> - @Inject

## @Component

- 作用于类
- 通过类路径扫描来自动侦测并自动装配到Spring容器中

## @Bean

- 作用于方法
- 用于注入某个类的实例

## @Autowired

- spring默认注解
- 默认根据类型注入（byType），当一个接口有多个实现类时，无法根据类型注入，注入方式会换转换为（byName）。此时应该使用@Qualifier来指定按名称注入的名称。

## @Rsource

- @Resource是Java提供的注解，默认使用（byName）。如果（byName）无法使用，会尝试使用（byType），可以通过name指定按名称注入，或者使用type指定按类型注入

# Bean的作用域

- **singleton**：Ioc容器中唯一的Bean实例，单列模式。
- **prototype**：每次获取/使用都会创建一个Bean实例。
- **request**：每一次HTTP请求都会产生一个新的bean，仅在当前HTTP Request中生效。
- **session**：每次来自新的session的HTTP请求产生一个bean，仅在当前HTTP session中有效。
- **websocket**：每一次websocket会话产生一个bean。

配置bean的作用域

```java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}

```

# Bean的生命周期

> **什么是Bean：由IOC实例化，组装，管理的对象**
>
> Java 对象的生命周期就是两步：实例化、不再使用的Bean被GC回收♻️

![image-20221005200120957](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20221005200120957.png)

> 1. Bean定义
> 1. 实例化
> 2. 属性赋值
> 3. 初始化（初始化之前执行pre方法，初始化之后执行atfer方法）
> 3. 使用Bean
> 4. 销毁

1. Bean定义

    三种定义方式：XML文件（setter方法或构造函数），注解方式（7种注解）

    一个Bean需要定义9中属性

    定义的Bean会放到BeanFactory中的beanDefinitionMap中，之后执行BeanFactoryPostProcessor

2. 创建Bean实例，使用Java Reflection（反射）创建Bean实例

3. Bean属性赋值

4. 调用Bean实现的Aware接口中的方法

    实现`BeanNameAware`接口，调用`setBeanName()`方法，传入Bean名字。

    实现`BeanClassLoaderAware`接口，调用`setBeanClassLoader()`，传入`ClassLoader`对象实例。

    实现`BeanFactoryAware`接口，调用`setBeanFactory()`方法，传入`BeanFactory`对象实例。

    ······

5. 如果有和加载这个Bean的Spring容器相关的`BeanPostProcessor`对象，执行`postProcessBeforeInnitialization()`方法

6. 如果实现了`InitializingBean`接口，调用`afterPropertiesSet()`方法。

7. 如果定义了Init - method，执行Init - method

8. 如果有和加载这个Bean的Spring容器相关的`BeanPostProcessor`对象，执行`postProcessAfterInitialization()`方法。

9. 销毁：

    如果Bean实现了`DisposableBean`接口，执行`destory()`方法

    如果Bean定义了Destroy - method方法，执行Destroy - method

## Bean定义

一个Bean需要定义9中属性：

- Class：定义Bean的Class，也就是类型
- Name：定义Bean的名称
- Scope：定义Bean的作用域
- Constructer Arguments：定义Bean的构造方法
- Properties：Bean的属性
- Autowired Mode：注入模型
    - no：不注入，不加任何注解，当作普通字段
    - byName：在容器中寻找和属性同名的Bean注入
    - byType：在容器中寻找字段的class的bean注入
    - constructor：使用构造函数注入
- Lazy initialization mode：懒加载模型
- Initialization method：初始化方法
- Destruction method：销毁方法

# Spring AOP

## AOP是什么

AOP（Aspect Oriented Programming）面向切面编程。

通过面相切面编程，通过预编译和运行期间动态代理，可以封装一些事务处理、日志、权限控制的方法，减少重复代码，降低耦合度。

AOP基于动态代理实现。

- 如果要代理的对象实现了某个接口，Spring AOP会使用JDK Proxy创建代理对象。
- 如果要代理的对象没有实现接口，Spring AOP会使用Cglib生成一个被代理对象的子类最为代理。

## AOP中的概念

- Target（目标）：被代理的对象，被通知的对象，需要增强的方法，确定AOP的对象。
- Proxy（代理）：创建的目标对象的代理对象。
- JoinPoint（连接点）：目标对象所属的类中所有的对象，确定了可以进行AOP的位置。
- Pointcut（切点）：实际进行AOP的位置，需要增强的方法，需要通知的方法。
- Advice（通知）：获取AOP的位置之后需要执行的方法，在目标之前、之后等执行。
- Aspect（切面）：Advice通知 + Pointcut切点，切点和切点前后执行的方法组成切面。
- Weaving（织入）：动词，表示将切面连接到应用程序或对象上，并生成被通知的代理对象的过程。

## 通知类型

- 前置通知：连接点之前执行的通知。
- 后置通知：连接点正常执行完之后执行的通知。
- 异常通知：连接点执行抛出异常时候执行的通知。
- 最终通知：连接点退出时候执行的通知，方法执行完毕，方法返回值已经返回。
- 环绕通知：包围一个连接点的通知。

## Spring AOP与AspectJ

- AspectJ是实现AOP思想的框架
- Spring AOP是运行时增强，运行期间通过Java动态代理（底层是反射）或者CGLIB的动态代理(底层时继承)实现。对程序无侵入性。
- AspectJ是编译时增强，通过编译期间对于字节码操作实现。

# Spring事务

## 事务的特性（ACID）

[已经写过了📖](https://www.beyondhorizon.top/article/vIJY813l)

## 编程式事务管理

使用`TransactionTemplate`和`TransactionManager`手动管理事务。

## 声明式事务管理

基于AOP实现，侵入性小，基于`@Transactional`注解

通过以下三个接口管理事务

- `PlatformTransactionManager`：平台事务管理器。
- `TransactionDefinition`：事务定义信息（事务隔离级别、传播行为、超时、制度、回滚）。
- `TransactionStatus`：事务运行状态。

### PlatformTransactionManager_事务管理接口

> Spring不直接管理事务，而是提供事务管理器。通过事务管理器（PlatformTransactionManager），Spring为其他框架提供事务管理器，如JDBC（DataSourceTranscationManager），JPA，Hibernate等。

**PlatformTransactionManager**接口提供了三个方法用于事务管理

- `    TransactionStatus getTransaction()`：获取事务。
- `void commit()`：提交事务
- `void rollback()`：回滚事务

### TransactionDefinition_事务属性

> 事务属性包含5个部分：
>
> 1. 隔离级别
> 2. 传播行为
> 3. 回滚规则
> 4. 是否只读
> 5. 事务超时

`TransactionDefinition` 接口中定义了 5 个方法以及一些表示事务属性的常量。

```java
public interface TransactionDefinition {
    // 需要传播
    int PROPAGATION_REQUIRED = 0;
    // 传播支持
    int PROPAGATION_SUPPORTS = 1;
    // 强制传播
    int PROPAGATION_MANDATORY = 2;
    // 创建一个新事务
    int PROPAGATION_REQUIRES_NEW = 3;
    // 不需要支持
    int PROPAGATION_NOT_SUPPORTED = 4;
    // 不传播
    int PROPAGATION_NEVER = 5;
    // 嵌套
    int PROPAGATION_NESTED = 6;
    int ISOLATION_DEFAULT = -1;
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
    int TIMEOUT_DEFAULT = -1;
    // 返回事务的传播行为，默认值为 REQUIRED。
    int getPropagationBehavior();
    // 返回事务的隔离级别，默认值是 DEFAULT
    int getIsolationLevel();
    // 返回事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。
    int getTimeout();
    // 返回是否为只读事务，默认值为 false
    boolean isReadOnly();

    @Nullable
    String getName();
}
```



### TransactionStatus_事务状态

`TransactionStatus`接口用来记录事务状态，接口中方法用于获取或判断事务状态

```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事务
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
}
```

# 事务传播行为

事务传播为了解决业务层方法之间相互调用的问题。

当一个有事务的方法被一个无事务的方法调用时，应该考虑该事务是否需要传播，及是否需要两个方法都开启事务。

如果A方法调用了B方法，此时A方法出现错误需要回滚，那么B方法要不要回滚呢。

## TransactionDefinition.PROPAGATION_REQUIRED

事务属性.需要传播，@Transaction默认设置的传播行为。需要传播。

- 如过外部方法没有开启事务，被PROPAGATION_REQUIRED修饰的方法会新开启自己的事务（事务之间相互独立，互不干扰）。
- 如果外部方法开启事务并且被PROPAGATION_REQUIRED修饰的方法，外部方法和内部方法同属于一个事务，只要有一个发生回滚，另一个也会回滚。

## TransactionDefinition.PROPAGATION_REQUIRES_NEW

创建一个新的事务。

- 如果当前方法存在事务，则把当前事务挂起，重新开启一个事务，与原来的事务互相独立。
- 如果当前方法不存在事务，则开启新的事务。

## TransactionDefinition.PROPAGATION_NESTED

- 如果当前方法存在事务，就在嵌套事务内进行。
- 如果当前方法没有事务，单独开启一个事务。

## TransactionDefinition.PROPAGATION_MANDATORY

- 如果当前方法有事务，则加入该事务。
- 如果当前方法没事事务，抛出异常。

# 事务隔离级别

## TransactionDefinition.ISLATION_DEFAULT

数据库默认隔离级别，MySQL中默认隔离级别时可重复读（REPEATABLE_READ）

## TransactionDefinition.ISOLATION_READ_UNCOMMITTED

读未提交

## TransactionDefinition.ISOLATION_READ_COMMITTED

读已提交

## TransactionDefinition.ISLOATION_REPEATABLE_READ

可重复读

## TransactionDefinition.ISOLATION_SERIALIZABLE

串行化

# @Transactional(rollbackFor = Exception.class)

- 如果不在`@Transactional`中表明回滚行为，只有在发生RunTImeException，事务才会回滚
- 如果表明`rollbackFor = Exception.class`在遇到任何异常都会回滚

