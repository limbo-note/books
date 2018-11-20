# 9. Servlet工作原理解析

### 9.1 从Servlet容器说起

如Jetty和Tomcat，都是Servlet容器。Tomcat容器模型如图：
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-1.jpg)

真正管理Servlet的容器是Context容器，一个Context对应一个web工程。Tomcat配置中的`<Context path="..." docBase="..." reloadable="true" />`可以体现这一点

- Servlet容器启动过程
	- 创建实例并新增web应用						
		![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-2.jpg)
		![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-3.jpg)
	- 调用Tomcat的start方法，启动逻辑基于Listener设计。启动时序图：
		![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-4.jpg)
		- 当Context容器状态为init时，其中的Listener将会被触发，包括ContextConfig，其init方法完成对应用的配置文件的解析
		- 而后，Context容器的startInternal方法完成对应用的初始化工作
- Web应用的初始化工作	
	- 解析web.xml是在ContextConfig的configureStart方法中实现的
		- 按照globalWebXml->hostWebXml->WebXml三个级别的顺序去寻找配置文件，web.xml会被解析保存到WebXml对象中
		- WebXml的configureContext()方法将WebXml对象中的属性设置到Context容器中，包括创建Servlet, filter, listener
		- configureContext()方法中是将Servlet包装成Context容器中的StandardWrapper, 因为Servlet是一个独立的Web开发标准，不应该强耦合在Tomcat中，所以应该先被包装再放入Context容器

### 9.2 创建Servlet实例

- 创建Servlet对象
	- 若Servlet的`load-on-startup`配置项大于0，就会跟随Context容器启动而实例化，如默认配置中定义的DefaultServlet和JspServlet
	- 从Wrapper.loadServlet()方法开始，获取servletClass，然后交给InstanceManager创建对象，涉及的类结构如图：	
		![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-5.jpg)
- 初始化Servlet
	- 若是普通的Servlet，则通过StandardWrapper.initServlet()方法，进一步调用了Servlet的init()方法进行初始化
	- 若是JspServlet，则在init()后会模拟一次简单的请求，请求调用jsp文件
- Servlet从解析到初始化的完整时序图如下：			
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-6.jpg)
