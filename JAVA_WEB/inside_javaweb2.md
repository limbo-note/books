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

### 9.3 Servlet体系结构


- Servlet顶层类关联如图：				
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-7.jpg)
	- ServletConfig在Servlet初始化时就传给Servlet了，保存着一些配置；而ServletResponse和ServletRequest是在请求达到时才传递的
	- ServletContext是一个“握手型交互模式”：交易场景由ServletContext描述，配置由ServletConfig描述，ServletResponse和ServletRequest就是要交易的对象
- ServletConfig和ServletContext
	- StandardWrapper和StandardWrapperFacade都实现了ServletConfig接口。实际传给Servlet的是StandardWrapperFacade，是门面类，可以保证从StandardWrapper中拿到必要的数据，而又不暴露不必要的数据
	- 同样，ApplicationContextFacade实现了ServletContext接口，并传给Servlet，也是门面类。此为**门面设计模式**
- Request和Response	
	- 一次请求对应的类转化图如图：						
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-8.jpg)
	- Tomcat接到请求时首先初步解析创建两个轻量级类org.apache.coyote.Request和org.apache.coyote.Response
	- 当交给用户线程去处理请求时又创建org.apache.catalina.connector.Request和org.apache.catalina.connector.Response, 传给Servlet的是这两个类的门面类RequestFacade和ResponseFacade

### 9.4 Servlet工作原理

- Mapper类保存了Container容器中所有子容器的信息，Request类在进入Container容器之前，Mapper会将hostname和contextpath设置到Request的mapppingData属性中，访问哪个子容器就已经确定了
- MapperListener类作为监听器会加到整个Container容器中的每个子容器中，任何一个容器发生变化，MapperListener都会被通知到，其mapper属性就会被更新
- Request在容器中的路由图：					
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-9.jpg)
- Request在到达最终的Servlet之前，还必须执行filter链和通知在web.xml中定义的Listener

### 9.5 Servlet中的Listener

基于**观察者模式**设计，主要是六大接口，包括对属性修改的监听和对生命周期状态变化的监听。例如spring的ContextLoaderListener监听器

### 9.6 Filter工作原理

核心是传递的FilterChain对象，这对象保存了链路上所有的Filter对象，保存在一个filters数组中。在链路上每执行一个Filter对象，计数加1，并可以通过doFilter方法进行传递，当链路上的所有Filter对象执行完成后，就会执行最终的Servlet

### 9.7 Servlet中的url-pattern

- 精确匹配. /foo.htm
- 路径匹配. /foo/*
- 后缀匹配. *.htm

Servlet（由Mapper类进行匹配）三种匹配方式优先级从高到低，但Filter只要能匹配上，都会被加入filters数组中，就都会在请求链上被调用

# 10. 深入理解Session和Cookie