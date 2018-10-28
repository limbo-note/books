# 1. 企业开发
瀑布模型、极限开发、敏捷开发

数据库选型：SQL, NoSQL  
SQL: MySQL, SQL Server, Oracle  
NoSQL: redis, memcached, MongDB等

SSH: Spring, Struts2, Hibernate  
SSM: Spring, SpringMVC, MyBaties  （Spring都是指第二个S）  

# 2. Spring架构设计
以IoC（控制反转）和AOP（面向切面编程）为核心

非侵入式设计：无需继承框架提供的类，重用性高  
轻量级：非入侵的、所依赖的东西少、资源占用少、部署简单  
POJO：Plain Old Java Objects 简单的java对象，可以包含业务、持久化逻辑，但不担当特殊角色，不继承、不实现类或接口  
容器：装对象的对象  
控制反转：IoC, 或者叫依赖注入，由容器控制程序之间的关系  
Bean：容器管理对象

优点，可以使我们只关注业务逻辑：
>1. 根据配置文件创建及组装对象之间的依赖关系，修改时无需该代码，只改配置文件
>2. 日志记录、权限控制、性能统计可以从业务逻辑中分离出来
>3. 帮助管理数据库事务
>4. 能与很多其它框架结合

应用场景：
>1. 典型的Spring Web应用，三层架构：数据模型层实现域对象，数据访问层实现数据访问，逻辑层实现业务逻辑
>2. 前端使用第三方Web框架
>3. 远程调用
>4. SSH/SSM

子项目：Spring Framework, Spring Security, Spring Web Flow, Spring Boot, Spring Data, Spring Batch, Spring Integration等

### Spring整体架构
Core Container 模块  
AOP, Aspects 模块  
Data Access/Integration 模块  
Web/Remoting 模块  
Test 模块

# 3. 核心概念Ioc
### 3.1 耦合  
一个类中包含调用着另一个类，当被调用的类做了修改时，调用类也需要做出相应的代码修改  

解耦：  
解耦的设计模式可以使用 **工厂模式**

耦合的代码产生的原因是在类中直接使用new创建对象，而创建的对象由可能发生改变。使用工厂模式创建对象，可以在创建的对象发生改变时通过修改工厂类完成，如：

	public interface IStudentBiz{}

	public class StudentBiz implements IStudentBiz{}

	public class StudentBizFactory {	
		public static IStudentBiz getInstance(){
			return new StudentBiz();
		}
	}

	//task
	public class StudentAction {
		IStudentBiz studentBiz = StudentBizFactory.getInstance();
	}

工厂类作为调用者和被调用者的中间环节，使StudentAction不再靠自身的代码去获得StudentBiz对象实例，而把这一工作交给了工厂类
	
### 3.2 Spring IoC/DI
管理所有的Java类，类对象的创建和依赖关系都通过IoC进行控制。例如，当在A类中new一个B时，控制权由A掌握，为控制正转；当A类使用的B类实例由Spring创建时，控制权由Spring掌握，为控制反转。

**工作原理：**  
像是一个容器，预先放入了很多类，程序需要什么类就从中取出相应的实例

在使用MVC模式进行三层架构设计时，通常Java类的调用顺序是Web层->业务层->持久层  
**步骤:**
>1. 为业务层和持久层设计接口，声明所需方法
>2. 编写持久层接口和具体类
>3. 编写业务层接口和具体类，并声明持久层的接口类型的域
>4. 编写Web层具体类，并声明业务层接口类型的域
>5. 在Spring配置文件中对Web层、业务层、持久层的具体类对象进行定义，并赋值
>6. 在代码中获取Web层对象