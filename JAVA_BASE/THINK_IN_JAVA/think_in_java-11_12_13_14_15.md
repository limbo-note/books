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