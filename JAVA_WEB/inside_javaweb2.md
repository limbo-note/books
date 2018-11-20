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
