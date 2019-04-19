**3.6.4 检测字符串是否相等(不用==)**  
**3.6.6 代码点与代码单元**  
**3.6.7 字符串常用api**  
**3.6.9 字符串构建器**  
**3.7 输入输出**  
**3.8.6 中断控制**: 带标签的break  

---
**4.3 自定义类**: 在一个源文件中，只能有一个共有类，可以有任意数量的非共有类  
**4.3.6**：访问器方法如果需要返回一个可变对象的引用，应该对它进行克隆`return ***.clone()`  
**4.3.7**：类的方法可以访问同一类的任何一个对象的私有域  
**4.3.9**：类中final声明的变量在构建对象时必须被构造器初始化，且之后不可变  
**4.4.5**：每个类可以有一个main方法，可以用于对类进行单元测试。想测试哪个类，只需运行那个类的编译文件  
**4.5**：java中参数均为值传递，传递对象时相当于传的是对象地址，虽然可以通过方法调用改变原对象的内容，但也属于值传递  
**4.6**：  
调用构造器的具体步骤——  

- 所有数据域被初始化为默认值
- 按照类声明中的次序，依次执行初始化语句和初始化块
- 如果构造器第一行调用了第二个构造器，执行第二个构造器主体
- 执行本构造器主体  

可以使用静态的初始化块对静态域进行初始化  
**4.8 类路径**  
**4.9 文档注释**：javadoc  

---
**5.1**：使用super调用构造器必须是子类构造器的第一条语句；如果子类没有显式调用父类构造器，则自动调用没有参数的构造器，如果父类没有不带参数的构造器，子类又没有显式调用其他构造器，则编译报错  
**5.1.4 阻止继承:final类和方法**：声明为final的类无法被继承，final方法无法被子类覆盖。final类中的方法自动成为final，但变量不会  
**5.1.5 强制类型转换**：只能在继承层次内进行类型转换；在将父类转换为子类之前，应该使用instanceof进行检查（类似c++中的dynamic_cast）  
**5.2 Object：所有类的父类**：  
equals方法  

- 参数为Object otherObject(可用@Override保证重写)
- 检测this和otherObject是否引用同一个对象
- 检测otherObject是否为null
- 比较this和otherObject是否属于同一个类(getClass/instanceof)
- 将otherObject转换为相应的类
- 对需要比较的变量进行比较  

hashCode()和toString()方法  
**5.3 泛型数组列表**：trimToSize(), toArray()  
**5.4 对象包装器**：Integer/Long/Double/...  
`static String toString(int i)`  
`static int parseInt(String s)`  
`static Integer valueOf(String s)`  
**5.7 反射**：能够分析类能力的程序  
**5.8 继承设计的技巧**：  

- 将公共操作和变量放在父类
- 不要使用protected变量，同包的所有类或者子类都能访问，破坏了封装性
- 不是is-a的关系，尽量不用继承
- 除非所有的继承方法都有意义，否则不要继承
- 重写时，不要改变预期的行为
- 使用多态，而不是instanceof

---
**6.1.2 接口与抽象类**：每个类只能拓展于一个类，但是可以实现多个接口，所以不使用抽象类而新引入了接口的概念  
**6.2 对象克隆**：默认的克隆操作是浅拷贝（只拷贝基本类型，不对内部对象进行拷贝）,可以实现Cloneable接口，使用public重新定义clone方法  
**6.3 接口和回调**  
**6.4 内部类**：可以访问该类定义所在的作用域的私有数据；可以对同一个包中的其他类隐藏；定义回调函数时，使用匿名内部类比较便捷  

- 局部内部类
- 匿名内部类
- 外部方法访问final变量
- 静态内部类

**6.5 代理（用得少）**

---
**7~9 用户界面**  

---
# 10.部署应用程序和applet
### 10.1 jar文件
`jar命令`  
清单文件MANIFEST.MF  
包密封  

### 10.2 Java Web Start

### 10.3 applet

### 10.4 配置
**10.4.1 属性映射(property map)**      
`Properties settings = new Properties();`  
`settings.put("...","...");`  
`settings.store(out,"...");`  
`settings.load(in);`  
`settings.getProperty("...");`  

**10.4.2 Preferences**  

# 11.异常、断言、日志、调试
### 11.1 处理错误
1. 用户输入错误
2. 设备错误
3. 物理限制
4. 代码错误

**11.1.1 异常分类**：Error类/RuntimeException类属于未检查异常，其他属于已检查异常  
**11.1.2 声明已检查异常(throws)**：方法应该在其首部声明所有可能的已检查异常，不应该声明未检查异常（如数组越界、系统错误等），未检查异常要么不可控制，要么应该避免发生  
如果子类中覆盖了父类的一个方法，子类方法中声明的已检查异常不能比父类方法声明的异常更通用。如果父类方法没有抛出任何已检查异常，则子类方法也不能抛出任何已检查异常  
**11.1.3 抛出异常(throw)**：`throw new Exception()`  

- 找到一个合适的异常类
- 创建这个类的一个对象
- 将对象抛出  

**11.1.4 创建异常类**：  

- 派生于一个异常类，包含两个构造器
- 默认的无参构造器
- 带String参数的能描述详细信息的构造器  

### 11.2 捕获异常
通常应该捕获那些知道如何处理的异常，而将那些不知道处理的异常继续进行传递  
可以捕获多个异常  
**11.2.3 finally子句**：当异常不是由catch子句捕获的（或者没有catch语句），将跳过try中剩余的语句，执行finally中的语句，并将异常重新抛给方法的调用者。并且强烈建议独立使用try/catch和try/finally语句，如：  
```
try
{
	try
	{
		...
	}
	finally
	{
		...
	}
}
catch(Exception e)
{
	...
}  
```  
**11.2.4 带资源的try语句**  
**11.2.5 分析堆栈跟踪元素**：`printStackTrace(), getStackTrace(), getAllStackTraces()`  

### 11.3 使用异常机制的技巧
1. 异常处理不能代替简单的检测判断(如，退栈时应该用isEmpty()检测栈是否为空，而不应该使用异常机制)
2. 不应该过分细化异常：把任务包装在一个try语句块中
3. 利用异常层次结构
4. 不压制异常：所有声明(throws)了异常的方法，在被调用时都应该被捕获，否则会产生编译错误
5. 检测错误时，尽量苛刻，在出错的地方抛出异常
6. 有时传递异常比捕获异常更好（早抛出，晚捕获）

### 11.4 断言
断言允许在测试期间向代码中插入一些检查语句。当代码发布时，这些插入的检测语句会被自动地移走  
**11.4.1 启动和禁用**：`-enableassertions -ea -disableassertions -da`  
**11.4.2 使用断言完成参数检查**：断言只用于开发和测试阶段  

### 11.5 记录日志
全局日志记录器：`Logger.getGlobal().info();`  

`private static final Logger logger = Logger.getLogger("...");`  

### 11.6 调试技巧
1. 日志输出
2. 类中放main方法，单元测试
3. JUnit单元测试框架
4. 异常对象printStackTrace()
5. 标准输出/标准错误重定向
6. 观察类的加载过程`-verbose`
7. -Xlint, 代码检查
8. jconsole, 图形界面监控
9. jmap, 显示堆的对象
10. -Xprof, 跟踪经常被调用的方法

### 11.7 GUI调试

### 11.8 使用调试器(IDE中)

# 12. 泛型程序设计
...

# 13. 集合
### 13.1 集合接口
Collection接口和Iterator接口等

### 13.2 具体的集合
Collection接口：  

- ArrayList, 数组列表，删除插入一个元素代价很大
- LinkedList, 链表，java中的都是双向链表
- HashSet, 散列集
- TreeSet, 树集，有序集合
- 队列和双端队列
- PriorityQueue, 优先队列

Map接口：

- HashMap(非排序), TreeMap(排序), 映射表

### 13.3 集合框架
实现用于多种集合类型的泛型，或增加新的集合类型

### 13.4 算法
- 排序和混排, `sort()`, `shuffle()`
- 二分查找, `binarySearch()`
- 基本算法操作(min,max等)

### 13.5 一直存在的集合类
Hashtable, Enumeration, Properties, Stack, BitSet

# 14. 多线程
1. `class MyRunnable implements Runnable{ public void run(){} }`
2. `Runnable r = new MyRunnable()`
3. `Thread t = new Thread(r)`
4. `t.start()`

### 14.2 中断线程
return返回，或者出现了没有捕获的异常时，线程终止  
调用`interrupt()`方法  

### 14.3 线程状态
`getState()`  

- New（新创建）
- Runnable（可运行）
- Blocked（被阻塞）
- Waiting（等待）
- Timed waiting（计时等待）
- Terminated（被终止）

### 14.4 线程属性
**线程优先级**：`setPriority()`设置优先级1-10   
**守护线程**  
**未捕获异常处理器**  

### 14.5 同步
**锁对象**：ReentrantLock类, lock(), unlock()  
**条件对象**；newCondition类, await(), signalAll(), signal()  

- 锁用来保护代码片段
- 锁可以管理试图进入被保护代码段的线程
- 锁可以拥有一个或多个条件对象
- 每个条件对象管理那些已经进入被保护的代码段但还不能运行的线程  

**synchronized关键字**：`public synchronized void method(){}`, wait(), notifyAll()  

**建议**：

- 最好不用Lock/Condition，也不用synchronized，许多情况下可以使用java.util.concurrent的机制
- 无其他选择的情况下，synchronized比Lock/Condition更简洁
- 除非特别需要Lock/Condition，才使用它  

(...........)
