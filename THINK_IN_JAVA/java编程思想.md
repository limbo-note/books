# 2. 一切都是对象
基本成员默认值，当变量作为类的成员使用时，java才确保给定其默认值。但却不会给局部变量（如某个方法里定义的变量）做初始化，方法变量未初始化在使用时会产生编译错误  

**javadoc**  

# 3. 操作符
不能操作符重载
# 5. 初始化与清理
方法重载，参数类型相同但顺序不同也将看做是重载；仅返回值不同不是重载  

带基本类型参数的方法重载，若传入的实际参数比声明的参数类型小，则可自动进行类型转换；若传入的参数较大，则必须强制进行窄化转换，否则编译报错  

构造器中，可以用this调用另一个构造器，但不能调用两个；必须将构造器置于最起始处，否则编译报错；禁止在非构造器方法中调用构造器  

终结处理和垃圾回收：finalize()，停止-复制，标记-清扫

变量初始化：类变量定义的先后顺序决定了初始化的顺序；即使变量定义散布在方法定义之间，它们仍然会在调用任何方法、构造器之前被初始化  

静态初始化：仅初始化一次，发生在首次生成这个类的对象，或首次访问属于那个类的静态数据成员时  

# 6. 访问权限控制
java可运行程序是一组可以打包并压缩为一个jar文件的class文件，java解释器负责这些文件的查找、装载和解释  

java解释器查找class过程：获得CLASSPATH；从根目录开始，获取包的名称并将句点替换成反斜杠，与CLASSPATH结合得到一些目录；在这些目录中查找class文件

### 6.4 类的访问权限
编译单元内没有public类也是可能的，这时可以随意对文件命名  

可以将所有的构造器指定为private来阻止直接创建此类的实例。但可以有两种方法使用此类：  
1. 构造一个public的static方法，它创建一个新的类对象实例并返回引用。2. 单例模式 

# 7. 复用类
组合( **has-a** )和继承( **is-a** )  

### 7.1 组合语法
将对象引用置于新类中

### 7.3 代理
基于组合和继承之间

与C++不同，Java的子类重载任何方法并不会屏蔽其父类原有的方法  

### 7.7 向上转型
形参为父类，实际传入的参数为子类

### 7.8 final
当对象的引用定义为final时，则无法把它改为指向另一个对象，但是对象本身是可以被修改的

final定义的变量必须在定义时初始化，或者在所有构造器中被初始化

final参数：无法被修改的参数

final方法：防止被重写  

private方法被隐式指定为final，因为private方法无法被子类重写  

final类：无法被继承

### 7.9 类的加载
class文件只在被用到时才会被加载，通常指的是创建第一个对象，或者首次访问static变量、方法时  

在加载子类时，父类也会自动被加载

# 8. 多态
动态绑定，使得只要遍历父类对象数组并进行相同的调用，就能自动正确地调用到其子类重写过的方法 

**构造器调用顺序(PDF192)：**  
1. 首先给将要创建的对象的存储空间全部初始化成0  
2. 按继承从上至下的顺序，调用所有基类构造器  
3. 按声明的顺序完成自身类的初始化动作，包括对象的初始化  
4. 调用自身类的构造器  

### 8.4 协变返回类型
在子类中被覆盖的方法可以返回父类中此方法返回类型的子类型

# 9. 接口
### 9.2 接口
接口可以包含变量，但是都隐式自动地是static和final的  
接口中不可以有static代码块  
接口中定义的方法必须为public（若没有显示声明为public，则自动成为public），否则将只有包访问权限，导致在继承的过程中，方法的可访问权限被降低了

### 9.3 完全解耦
接口提高复用性

### 9.4 java中的多重继承
即可以同时实现多个接口，但最多只能继承于一个父类

### 9.5 接口也可继承
接口的继承extends后可接多个接口，即`interface A extends B, C...`  

应尽量避免在继承多个接口时，多个接口中出现方法名相同而返回值不同的情况  

### 9.6 适配接口
接口的一种常见用法就是使用策略设计模式，适配器设计模式  

### 9.8 嵌套接口
(........）

### 9.9 工厂设计模式
(........)

# 10. 内部类
### 10.1 创建内部类
如果想在外部类非静态方法之外的地方创建内部类，需要OuterClass.InnerClass明确指明对象类型  

### 10.2 链接到外部类
“迭代器”设计模式：内部类拥有其外部类的所有元素的访问权  

.this：内部类中生成对外部类对象的引用，需要使用外部类名字后紧跟.this  
.new：在外部类非静态方法之外的地方 **直接** 创建内部类，必须使用外部类对象来创建内部类对象，并使用.new  

### 10.4 （......）
（......）

# 11. 持有对象
即容器
### 11.3 添加一组元素
`collection.addAll(Arrays.asList(array))`  

显式类型参数说明  

collection的默认toString()都可以生成可读性比较好的结果，即可以直接打印  

`List, LinkedList, Stack, Set, Map, Queue`  

`Colleciton和Iterator`  

java容器汇总简图：PDF279

# 12. 异常
### 12.7 java标准异常
对null进行调用，java会自动抛出NullPointerException异常  

RuntimeException“不受检查的异常”：会被java虚拟机自动抛出，将被自动捕获，如空指针异常;对这种异常类型，不需要throws声明;如果RuntimeException没有被捕获而直达main()，则在程序退出之前自动调用printStackTrace()  

### 12.8 finally
不管finally块前面有多少return点，finally块中的代码照常会执行  

finally不正确使用可能导致异常丢失PDF301

### 12.9 异常限制
重写方法时，最多只能抛出在父类方法中已经声明了的异常；构造器不受此限制  

子类构造器不能捕获父类构造器抛出的异常  

### 12.10 构造器
（..........）

### 12.11 异常匹配
异常处理系统按照代码的书写顺序找出最近的catch程序，执行此处理程序后，然后就不再查找；所以如果把父类的异常放在最前面，后面的子类异常会被全部屏蔽，编译器会报错

# 13. 字符串

### 13.1 不可变

每一个看起来会修改String值的方法，都是创建了一个全新的String对象

### 13.2 重载"+"与StringBuilder

重载符"+"使得编译器自动创建一个StringBuilder对象，每次"+"都调用一个StringBuilder的append()方法，而不是String的append()方法。因为StringBuilder更高效

	public class WhitherStringBuilder {
      	public String implicit(String[] fields) {
        	String result = "";
        	for(int i = 0; i < fields.length; i++)
         		result += fields[i];
        	return result;
      	}
	
    	public String explicit(String[] fields) {
        	StringBuilder result = new StringBuilder();
        	for(int i = 0; i < fields.length; i++)
          		result.append(fields[i]);
        	return result.toString();
      	}
    }
上述代码中，implicit()方法会在每次"+="或者"+"时都创建一个StringBuilder对象，而explicit()中只创建了一个StringBuilder。而且，显式创建StringBuilder还允许预先指定大小，避免多次重新分配缓冲区

### 13.3 无意识的递归

	public class InfiniteRecursion {  	
		public String toString() {
        	return " InfiniteRecursion address: " + this + "\n";
      	}

      	public static void main(String[] args) {
        	List<InfiniteRecursion> v =
          	new ArrayList<InfiniteRecursion>();
        	for(int i = 0; i < 10; i++)
          		v.add(new InfiniteRecursion());
        	System.out.println(v);
      	}
    }
toString()方法中的this会自动调用this.toString()方法，引起无限递归。若想得到对象的内存地址，应该调用Object.toString()方法，即super.toString()

### 13.5 格式化输出
printf(), System.out.format(), Formatter类, String.format()

- 格式化说明符: %[argument_index$][flags][width][.precision]converison
- Formatter类，类型转换

### 13.6 正则表达式

- split(), matches(), replace(), replaceAll()
- 查表

	![](https://github.com/limbo-china/books/blob/master/THINK_IN_JAVA/13-1.png)	
	![](https://github.com/limbo-china/books/blob/master/THINK_IN_JAVA/13-2.png)	
	![](https://github.com/limbo-china/books/blob/master/THINK_IN_JAVA/13-3.png)	
	![](https://github.com/limbo-china/books/blob/master/THINK_IN_JAVA/13-4.png)	
- 量词
	- 贪婪型，尽可能匹配最多
	- 勉强型，尽可能匹配最少
	- 占有型，只有在java中有，防止回溯，更高效
	
	![](https://github.com/limbo-china/books/blob/master/THINK_IN_JAVA/13-5.png)	
- Pattern和Matcher
	
		Pattern p = Pattern.compile(regular);
		Matcher m = p.matcher(string);
		print(m.group());
		print(m.end());
	详细见书
- 组group

### 13.7 扫描输入

BufferReader, Scanner, StringTokenizer(已废弃)

- Scanner定界符
	- Scanner的nextXXX()默认使用空格作为分界符，可以用useDelimiter(regular)方法修改
	- Scanner的next(regular)方法可以传入正则表达式，以正则表达式获取下一个符合条件的值

# 14. 类型信息

RTTI（运行时类型信息）和反射

### 14.1 RTTI作用
在程序运行时得到某个对象的确切类型，通过选择或者剔除对特定的类型，并进行特殊的操作

### 14.2 Class对象

Class对象仅在需要的时候才被加载，static初始化块是在类加载时进行的

Class.forName("XX")，获得xx类的Class对象的引用（参数必须是类的全限定名，即包含包名），每个类都有一个Class对象，会触发类加载；或者使用此类的对象的getClass()方法获得Class对象

- 类字面常量。除了forName()获得Class对象引用外，还可以用XX.class，更安全更高效
	- boolean.class == Boolean.TYPE
	- 仅使用.class不会引发类的初始化，而forName()会
	- 如果static final域是一个编译期常量，读取它不会引发初始化；否则会
- 泛化的Class引用 
	- Class<?>, Class< T >, Class<? extends XXX>
	- （........）见泛型
- cast()转型语法。使用Class引用来进行cast()转型。对于普通类的转型，等价于直接加括号转型；但对于特殊的泛型类时，则必须使用此方法

### 14.3 类型转换前先做检查

向下转型前，使用instanceof检查是非常重要的，否则可能得到ClassCastException异常

Class.isInstance()可动态测试，避免写出大量的instanceof语句

### 14.4 注册工厂

**工厂设计模式**（学习工厂设计模式后再看............）

### 14.5 instanceof和Class对象比较的差别

	class Base {}
    class Derived extends Base {}	

    public class FamilyVsExactType {
      	static void test(Object x) {
        	print("Testing x of type " + x.getClass());
        	print("x instanceof Base " + (x instanceof Base));
        	print("x instanceof Derived "+ (x instanceof Derived));
        	print("Base.isInstance(x) "+ Base.class.isInstance(x));
        	print("Derived.isInstance(x) " +
          Derived.class.isInstance(x));
        	print("x.getClass() == Base.class " +
          (x.getClass() == Base.class));
        	print("x.getClass() == Derived.class " +
          (x.getClass() == Derived.class));
        	print("x.getClass().equals(Base.class)) "+
          (x.getClass().equals(Base.class)));
        	print("x.getClass().equals(Derived.class)) " +
          (x.getClass().equals(Derived.class)));
      	}

      	public static void main(String[] args) {
        	test(new Base());
        	test(new Derived());
      	}	
    }

> Testing x of type class typeinfo.Base  
> x instanceof Base true  
> x instanceof Derived false  
> Base.isInstance(x) true  
> Derived.isInstance(x) false  
> x.getClass() == Base.class true  
> x.getClass() == Derived.class false  
> x.getClass().equals(Base.class)) true  
> x.getClass().equals(Derived.class)) false  
> Testing x of type class typeinfo.Derived  
> x instanceof Base true  
> x instanceof Derived true  
> Base.isInstance(x) true  
> Derived.isInstance(x) true  
> x.getClass() == Base.class false  
> x.getClass() == Derived.class true  
> x.getClass().equals(Base.class)) false  
> x.getClass().equals(Derived.class)) true  

instanceof 在检查类型时会考虑继承关系，而直接判断Class对象是否相等则不会考虑继承关系

### 14.6 反射：运行时的类信息

**RTTI相当于静态的类型信息，反射是动态的类型信息，编程相对更灵活**

对RTTI来说，编译器在编译时打开和检查.class文件。而对反射机制来说，.class文件在编译时是不可获取的，所以是在运行时打开和检查.class文件

- 类方法提取器
	
		public static void main(String[] args) {
        	if(args.length < 1) {
          		print(usage);
          		System.exit(0);
        	}

          	Class<?> c = Class.forName(args[0]);
          	Method[] methods = c.getMethods();
          	Constructor[] ctors = c.getConstructors();	
			...
		
	- Class类的getMethods()和getConstructors()方法
	- 其中Class.forName(args[0])生成的结果是编译时不可知的，方法的信息是在运行时提取出来的
	- IDE自动提示类方法功能的基础，也侧面证明了可以在编译期打开和检查.class文件

### 14.7 动态代理

**代理模式**：

	interface Interface {
        void doSomething();
        void somethingElse(String arg);
    }

    class RealObject implements Interface {
        public void doSomething() { print("doSomething"); }
        public void somethingElse(String arg) {
            print("somethingElse " + arg);
        }
    }	

    class SimpleProxy implements Interface {
        private Interface proxied;
        public SimpleProxy(Interface proxied) {
            this.proxied = proxied;
        }
        public void doSomething() {
            print("SimpleProxy doSomething");
            proxied.doSomething();
        }
        public void somethingElse(String arg) {
            print("SimpleProxy somethingElse " + arg);
            proxied.somethingElse(arg);
        }
    }	

    class SimpleProxyDemo {
        public static void consumer(Interface iface) {
            iface.doSomething();
            iface.somethingElse("bonobo");
        }
        public static void main(String[] args) {
            consumer(new SimpleProxy(new RealObject()));
        }
    }

SimpleProxy被插入到了consumer()和RealObject之间，相当于一个代理。如果希望跟踪对RealObject中方法的调用，或者希望度量这些调用的开销，就可以不修改RealObject中的业务代码，只修改代理类SimpleProxy中相应的方法

**动态代理**：

	class DynamicProxyHandler implements InvocationHandler {
		  private Object proxied;
		  public DynamicProxyHandler(Object proxied) {
		    this.proxied = proxied;
		  }
		  public Object
		  invoke(Object proxy, Method method, Object[] args)
		  throws Throwable {
		    System.out.println("**** proxy: " + proxy.getClass() +
		      ", method: " + method + ", args: " + args);
		    if(args != null)
		      for(Object arg : args)
		        System.out.println("  " + arg);
		    return method.invoke(proxied, args);
		  }
		}	

		class SimpleDynamicProxy {
		  public static void consumer(Interface iface) {
		    iface.doSomething();
		    iface.somethingElse("bonobo");
		  }
		  public static void main(String[] args) {
		    RealObject real = new RealObject();
		    consumer(real);
		    // Insert a proxy and call again:
		    Interface proxy = (Interface)Proxy.newProxyInstance(
		      Interface.class.getClassLoader(),
		      new Class[]{ Interface.class },
		      new DynamicProxyHandler(real));
		    consumer(proxy);
		  }
		}

Proxy.newProxyInstance()方法创建动态代理，参数：类加载器，该代理实现的接口列表及InvocationHandler的一个实现；可在invoke方法内过滤某些方法的执行

**动态代理实现事务：**

![](https://github.com/limbo-china/books/blob/master/THINK_IN_JAVA/14-1.png)	
	
### 14.8 空对象

用null表示缺少对象，则每次都需要检查是否为null，由此引入空对象

	public interface Null {}

	class Person {
      public final String first;
      public final String last;
      public final String address;
      // etc.
      public Person(String first, String last, String address){
        this.first = first;
        this.last = last;
        this.address = address;
      }	

      public static class NullPerson extends Person implements Null {
        private NullPerson() { super("None", "None", "None"); }
        public String toString() { return "NullPerson"; }
      }
      public static final Person NULL = new NullPerson();
    }
空对象通常为单例，可避免使用instanceof来检查，直接用equals()甚至是==来检查；如上述代码，NullPerson也有自己的toString()方法，这样在处理Person时，不用特殊地去检查是否为null而做另外的处理，直接统一调用toString()方法即可

在某些地方仍必须测试空对象，与测试null一样；但在其它很多地方，如上述的toString()，就不用执行额外的检查了

### 14.9 接口与类型信息

interface关键字的一个重要目标是隔离构件，降低耦合性。但是用类型信息会增加耦合性，如：
	
	public interface A {
		  void f();
	} 

	class B implements A {
      public void f() {}
      public void g() {}
    }

    public class InterfaceViolation {
      public static void main(String[] args) {
        A a = new B();
        a.f();
        // a.g(); // Compile error
        System.out.println(a.getClass().getName());
        if(a instanceof B) {
          B b = (B)a;
          b.g();
        }
      }
    }
这就使得可以通过强制转型，调用不在A中的方法

其次，如果通过：限制B的访问权限（使得无法直接通过引用B进行强制转型）、私有内部类、匿名类，都可以使用反射的特殊方法来调用那些非公共访问权限的方法和域，而且并不能阻止反射的这种特殊用法

# 15. 泛型
(.....重要，篇幅很大)

# 16. 数组
### 16.1 数组特殊性

与容器的区别主要有三方面：效率、类型和保存基本类型的能力
- 效率最高，但大小被固定
- 能直接用基本类型，容器需要包装类

### 16.2 数组是第一级对象
数组是一个对象，在堆中也是同对象一样分配了内存，保存的内容是其它对象的引用或者基本类型的值

多维数组的各维内数组的长度可以不一致，称为**粗糙数组**
### 16.5 数组与泛型

不能：
	
	Class<T>[] arr = new Class<T>[10];

(可以声明`Class<T>[] arr;`这么一个引用，不实例化)

但是可以：

	class MyClass<T>{
		private T[] = ...;
	}
### 16.6 创建测试数据

- Arrays.fill(). 只能用单一的数值来填充数组
- Generator数据生成器

(......重要，新知识，篇幅大)

### 16.7 Arrays实用功能

Arrays的一些static方法：
equals(), fill(), sort(), binarySearch()（用于在已排序的数组中查找元素）, toString(), hashCode(), asList() 

- 复制数组。 System.arraycopy()方法，不会自动装包拆包，在复制对象时是浅复制（即只复制引用）

# 18. Java I/O系统
###18.1 File类
**目录列表器**：获取目录下的文件列表  
`File f= new File("yourpath")`  
`String[] list = f.list([new FilenameFilter()])`  

**目录实用工具**：查找、遍历目录等方法  

File类还可以创建目录、创建文件、查看文件信息等文件操作  

###18.2 输入和输出
**InputStream类型**PDF567  
**OutputStream类型**PDF568  

**FilterInputStream类型**PDF569  
**FilterOutputStream类型**PDF570  
(面向字节的)

###18.4 Reader和Writer
提供兼容Unicode和面向字符的I/O，为了国际化考虑  

各类Reader和Writer PDF571  

RandomAccessFile类

###18.6 I/O流的典型使用方式
**缓冲输入文件**：  
`BufferedReader in = new BufferedReader(new FileReader("filename"));`  
`while((str = in.readline())!= null)`  
`in.close()`  

**从内存输入**  
**格式化的内存输入**  

**基本的文件输出**：  
`PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter("filename")));`  
`out.println("...");`  
`out.close();`  

**存储和恢复数据（跨平台，面向字节）**：  
`DataOutputStream out = new DataOutputStream(new BufferedOutputStream(new FileOutputStream("filename")));`  
`out.write...();`  
`out.close();`  

### 18.7 （.............）

（...........）

#19. 枚举 

（.........）

#20. 注解
###20.1 基本语法
定义注解和定义接口类似，在关键字interface前加@  
不包含元素（类似方法）的注解是标记注解  

三种标准注解：@Override, @Deprecated, @SuppressWarnings  
四种元注解：@Target, @Retention, @Documented, @Inherited  

###20.2 （......）

（........）

#21. 并发
从性能的角度看，如果没有任务会阻塞，那么在单处理器机器上使用并发就没有任何意义  

**协作多线程和抢占多线程(java是抢占式)**  
协作多线程：允许线程自己决定什么时候放弃控制，程序员可以精确地决定某个线程何时被挂机；缺点是可能会有某个线程长时间占用CPU，导致其他线程饥饿  
抢占式线程：操作系统决定何时打断线程，可以在任何时候。通常会在一个时间片后打断线程，防止出现线程饥饿的情况  

###21.2 基本的线程机制
静态方法Thread.yield()告诉线程调度器：“本线程已经执行完生命周期中最重要的部分了，现在正是切换给其它任务执行一段时间的大好时机。”此方法完全是可选的  

当从Runnable导出一个类时，必须具有run()方法，但是它不会具有任何内在的线程能力。要实现线程行为，必须显示地将一个任务附着到线程上  

**Thread类**  
start方法在调用后立即返回，线程启动  

**使用Executor**  
CachedThreadPool, FixedThreadPool(线程池中线程数量固定), SingleThreadExecutor(线程数量固定为1的线程池)  

**从任务中产生返回值**  
可以实现Callable<Type>接口代替Runnable接口,覆盖call()方法，并用ExecutorService.submit()代替ExecutorService.exec()  

**sleep**  
老方式：`Thread.sleep(time)`  
新方式：`TimeUnit.MILLISECONDS.sleep(time)`  

**优先级**  
**让步**：yield()方法  
**后台线程**：当最后一个非后台线程终止时，所有后台线程会突然终止  

**21.2.9 编码的变体**：  

1. 直接继承Thread类，并在类中实现run()方法，再直接调用start()启动
2. 定义一个类实现Runnable接口，但在此类中声明一个Thread引用，并通过new Thread(this)创建对象，再通过对此Thread调用strat()方法启动线程
3. 匿名内部类实现：`t= new Thread(){ public void run(){...} }; t.start();`  
`t= new Thread(new Runnable(){ public void run(){...} }); t.start();`  

线程不是任务：Thread类自身不执行任何操作，它只是驱动赋予它的任务  

**加入一个线程**：如果在某个线程上调用另一个线程t.join()，此线程将被挂起，直到线程t结束；可以在join()中传入超时参数，在给定时间内t未完成则强制返回；可用interrupt()中断  

**线程的异常**：使用一般方法执行Executor中的线程，run()方法内出现异常时，无法在run()方法外捕获到异常；可以使用Thread.UncaughtExceptionHandler进行改进

###21.3 共享受限资源
synchronized, lock(), unlock()  

**原子性与易变性**：除long和double之外的基本类型，在读取和写入时，可以看成是原子操作；但对于long和double，在读取和写入时，是非原子的，可使用volatile

volatile可保证可见性  

**21.3.7 线程本地存储**  
（...............）  

（........）