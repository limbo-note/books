---
 typora-root-url: ./
---

[TOC]

# 1. Spring之旅

### 1.1 简化开发

根本使命就是简化开发：

- POJO轻量级和最小侵入性编程

- 依赖注入、面向接口

  - 紧耦合的构造模式

  ![](/1-1.jpg)
  
  - 构造器依赖注入

  ![](/1-2.jpg)

  - ApplicationContext 创建和组装bean，xml文件配置的方式则用ClassPathXmlApplicationContext ，该类加载位于应用程序类路径下的一个或多个XML配置文件 

- 切面和惯例，声明式编程

  - 若无Spring-aop，则需要在切点方法上下手动加上通知类，使代码紧耦合
  - 而Spring-aop只需要在配置文件中配置切点即通知即可，代码和原来无变化，消除耦合

- 切面和模板，减少代码

  - 如JdbcTemplate 

### 1.2 Spring 容器

自带多个容器：BeanFactory和ApplicationContext （一般采用这个）

- 使用ApplicationContext 

  - 多种加载方式，如从文件系统加载、从类路径下加载等，对应不同的

- Bean的生命周期

  ![](/1-3.jpg)

### 1.3 Spring架构

![](/1-4.jpg)

# 2. 装配Bean

### 2.1 配置方式

- 隐式的bean发现机制和自动装配
- Java中进行显式配置
- XML中进行显式配置

尽可能地使用自动配置的机制，显式配置越少越好 ；且Java显式方式优先于XML

### 2.2 自动装配

- 组件扫描
  - `@Component`标记需要扫描的类，之后Spring会为其自动创建Bean。扫描功能默认不开启，需要`@ComponentScan`
  - `@ComponentScan`默认会扫描与配置类相同的包及子包，查找带有`@Component`注解的类 ；或者使用xml配置：`<context:component-scan base-package="package" />`
  - `@Component("name")`为 bean命名
  - `@ComponentScan(basePackages={"package1","package2"})`设置扫描包，默认为当前类所在包
- 自动装配（`@Autowired `注解）
  
  - 注解在构造器上
  - 注解在set方法上
    - 对上面两种方法，Spring都会尝试满足方法参数上所声明的依赖 ，`required`属性指定必要性
  - 直接注解在域上（属性注入）

### 2.3 Java显式配置

- 创建配置类

  - `@Configuration`表明该类是配置类

- 配置创建Bean的方式

  - `@Bean`注解

    ![](/2-1.jpg)

### 2.4 XML显式配置

- 构造器注入（`<constructor-arg> `）

- 属性注入（`<property> `）

    - `<util:list> `创建集合类型的bean

        ![](/2-2.jpg)

### 2.5 导入和混合配置

- `@Configuration`类引入另一个`@Configuration`类
    - `@Import(C.class)`注解
- `@Configuration`类引入另一个XML配置文件
    - `@ImportResource("path.xml")`注解
- XML文件引入`@Configuration`类
    - `<bean class="soundsystem.CDConfig" />`
- XML文件引入另一个XML文件
    - `<import resource = "path.xml"/>`

# 3. 高级装配

### 3.1 环境与profile 

(........)

### 3.2 条件化的bean

`@Conditional `注解，bean只有在应用的类路径下包含特定的库时、当另外某个特定的bean也声明了之后 、或某个特定的环境变量设置之后才创建

(........)

### 3.3 自动装配的歧义性

- 当自动装配的可选bean不是唯一时，spring就无法做出选择而抛出异常，如：
  
  ![](/3-1.jpg)

- 在`@Component`注解或`@Bean`注解上使用`@Primary `注解指定最优先或XML中的`primary="true"`属性
- 在`@Autowired`注解上使用`@Qualifier("beanId")` 注解指定注入bean，在`@Component`注解或`@Bean`注解上使用`@Qualifier("beanId")`注解为类分配beanID
- 当需要多个限定符来定位一个Bean时，就不能仅仅用`@Qualifier`，可用自定义的注解共同定位一个bean

### 3.4 bean的作用域

在`@Component`注解或`@Bean`注解上使用`@Scope`注解吗，或XML文件中使用`scope`属性

- 单例
- 原型：每次注入或者通过Spring应用上下文获取的时候，都会有一个新的实例
- 会话：每个Session一个新的实例
  - 作用域代理：`@Scope`注解的`proxyMode`属性（XML中`<aop:scoped-proxy>`） (解决会话、请求作用域的bean注入到单例作用域的bean中的问题(.......))
- 请求：每个Request一个新的实例

### 3.5 运行时值注入 

- 注入外部的值

  - 声明属性源并通过Spring的Environment来检索属性（`@PropertySource`注解和`Environment` ），如

    ![](/3-2.jpg)

    - `Environment`还提供了一些方法来检查哪些profile处于激活状态 

  - 占位符装配属性(`${property.value}`) 

    - 需配置`PropertySourcesPlaceholderConfigurer` bean或XML中的`<context:propertyplaceholder>` 
    - 自动装配中使用`@Value`注解

- Spring表达式语言进行装配 （SpEL）

  - 使用bean的ID来引用bean `#{beanId.property}` `#{beanId.method()}`
  - 调用方法和访问对象的属性 `#{T(System).currentTimeMillis()}` `#{systemProperties['property.value']}`
  - 对值进行算术、 关系和逻辑运算 `#{基本类型数据、String、运算表达式}` `#{2 * T(java.lang.Math).PI * circle.radius}`
  - 正则表达式匹配 
  - 集合、数组操作 

# 4. 面向切面

### 4.1 AOP

- AOP术语

  - 连接点：所有可以插入切面的点

  - 切点：定义在“何处”插入切面，会匹配通知要织入的一个或多个连接点 

  - 切面：通知+切点

  - 引入 ：向现有的类添加新方法或属性 

  - 织入 ：把切面应用到目标对象并创建新的代理对象的过程

    - 编译期织入：AspectJ 的织入编译器
    - 类加载期
    - 运行期：Spring AOP 的织入方式（在代理类中包裹切面，调用目标方法前，会执行切面逻辑 ）

- Spring的AOP

  - 创建切点来定义切面所织入的连接点是AOP框架的基本功能 

  - Spring4种类型AOP（前三种都是在动态代理的基础上，局限于方法拦截 ）
      - 基于代理的经典AOP（手动编写代码，繁杂，不使用）
      - 纯POJO切面（将纯POJO转换为切面 ，需要XML配置）
      - `@AspectJ` 注解驱动的切面（注解式）
      - 注入式AspectJ切面 （不局限于方法拦截）
  - 应用需要被代理的bean时， Spring才创建代理对象 

### 4.2 通过切点选择连接点 

- AspectJ的切点表达式规范见书

- 编写切点

  ![](/4-1.jpg)

- bean()指示器，限制切点只匹配特定的bean 

  ![](/4-2.jpg)

### 4.3 使用注解创建切面 

- 定义切面

  ![](/4-3.jpg)

  配置：

  ![](/4-4.jpg)

  或：

  ![](/4-5.jpg)

- 环绕通知

  ![](/4-6.jpg)

  - 若不调用proceed()方法，将会阻塞

- 处理切点方法中的参数 

  ![](/4-7.jpg)

- 通过注解引入新功能 （切面可以为Spring bean添加新方法 ），`@DeclareParents` 注解

### 4.4 在XML中声明切面

基本与注解方式能一一对应，详细见书

### 4.5 注入AspectJ切面 

AspectJ提供了Spring AOP所不能支持的许多类型的切点 ，如构造器切点、属性切点

(........)

