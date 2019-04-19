**条目均用大写数字标明，涉及到其它条目之处均在括号里指明**

# 1. 引言 # 

有效地使用java及基本类库:java.lang, java.util,  java.util.concurrent, java.io

最基本的几条原则：

>- 清晰性和简洁性最为重要：模块的用户永远也不应该被模块的行为所迷惑
>- 模块要尽可能小，但又不能太小
>- 代码应该被重用，而不是被拷贝
>- 模块之间的依赖性应该尽可能地降到最小
>- 错误应该尽早被检测出来，最好是在编译时刻

API：指类、接口、构造器、成员和序列化形式，程序员可以通过它们访问类、接口或者包；使用API编程的程序员称为API的用户，在类的实现中使用了API的类被称为API的客户

粗略来讲，一个包的API是由该包中的每个公有类或者接口中所有公有的或者受保护的成员和构造器组成

# 2. 创建和销毁对象 #

### 一：考虑用静态工厂方法代替构造器 ###

为了让别人获取自身的一个实例，可以使用公有的构造器。还有一种方法就是，提供一个公有的 **静态工厂方法**（一个返回类实例的静态方法）

此静态工厂方法不是指设计模式的工厂方法

相比于公有的构造器，优势在于：

>- 有名称。产生的代码更易于阅读
>- 不必在每次调用的时候都创建一个新的对象。能够为重复的调用返回相同的对象，有助于控制在某个时刻哪些实例应该存在。这种类称为 **实例受控的类**：可以确保它是一个Singleton（三）或者是不可实例化的（四）；还使得不可变类（十五）可以确保不会存在两个相等的实例，即当且仅当a==b时，才有a.equals(b)为true，就可以使用==操作符代替equals，提升性能(如： **枚举类型**)
>- 可以返回原返回类型的任何子类型对象。API可以返回对象，同时又不会使对象的类变成公有的，适用于基于接口的框架（十八）；返回的对象的类不仅可以是非公有的，而且该类还可以随着每次静态工厂方法调用的参数值不同而发生变化； **服务提供者框架(如JDBC)**：服务接口；提供者注册API；服务访问API
>- 在创建参数化类型实例的时候，使代码更简洁。编译器的 **类型推导**可以帮助自动找到类型参数。例如：`Map<String, List<String> m = new HashMap<String, List<String>>()`  
可以改为  
`public static <K, V> HashMap<K, V> newInstance(){ return new HashMap<K, V>(); }`  
`Map<String, List<String>> m = HashMap.newInstance();`

缺点在于：

>- 类如果不含公有的或者受保护的构造器，就不能被子类化
>- 与其他的静态方法实际上没有任何区别。不被javadoc识别，给查阅带来困难。

一些惯用的静态工厂方法名称：`valueOf, of, getInstance, newInstance, getType, newType`  
静态工厂方法和公有构造器各有长处，但通常静态工厂方法更优，优先考虑

### 二：遇到多个构造器参数时要考虑用构建器 ###

静态工厂方法和构造器公有的局限：不能很好地扩展到大量的可选参数。考虑一个类，有几个域变量是必需提供的，还有大量的可选域变量。  
对这样的类，传统的方法是 **重叠构造器(telescoping constructor)模式**：第一个构造器只有必要参数的，第二个构造器有一个可选参数，第三个有两个，以此类推，最后一个包含所有可选参数。这种方法通常需要传递很多本不想设置的参数，当可选参数增加时，代码会变得难以编写和阅读  
第二种方法就是 **JavaBeans模式**：调用一个无参构造器来创建对象，然后用set方法来设置每个必要的参数以及需要的可选参数。但这种方法有严重的缺点：构造过程被分到了几个调用中，在构造过程中可能处于不一致的状态；另一个缺点在于，这种模式阻止了把类做成不可变的可能（十五），需要付出额外的努力来确保线程安全

想要保证像重叠构造器模式的 **安全性**和javabeans模式的 **可读性**，可以使用 **构建器(Builder)模式**：不直接生成想要的对象，而是让客户端利用所有必要的参数调用构造器，得到一个builder对象，在builder对象上调用类似set方法来设置可选参数，客户端再调用无参的build方法来生成不可变对象  
	
	public class NutritionFacts {
		private final int servingSize;
		private final int servings;
		private final int calories;
		private final int fat;
		private final int sodium;
		private final int carbohydrate;

		public static class Builder {
			// Required parameters
			private final int servingSize;
			private final int servings;

			// Optional parameters - initialized to default values
			private int calories = 0;
			private int fat = 0;
			private int carbohydrate = 0;
			private int sodium = 0;

			public Builder(int servingSize, int servings) {
				this.servingSize = servingSize;
				this.servings = servings;
			}

			public Builder calories(int val) {
				calories = val;
				return this;
			}

			public Builder fat(int val) {
				fat = val;
				return this;
			}

			public Builder carbohydrate(int val) {
				carbohydrate = val;
				return this;
			}

			public Builder sodium(int val) {
				sodium = val;
				return this;
			}

			public NutritionFacts build() {
				return new NutritionFacts(this);
			}
		}

		private NutritionFacts(Builder builder) {
			servingSize = builder.servingSize;
			servings = builder.servings;
			calories = builder.calories;
			fat = builder.fat;
			sodium = builder.sodium;
			carbohydrate = builder.carbohydrate;
		}

		public static void main(String[] args) {
			NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
					.calories(100).sodium(35).carbohydrate(27).build();
		}
	}

### 三：用私有构造器或者枚举类型强化Singleton属性 ###

Singleton指仅仅被实例化一次的类  
三种方法实现：

- 把构造器保持为私有，导出公有的静态成员，公有静态成员为final域  
		
		public class Elvis {
			public static final Elvis INSTANCE = new Elvis();

			private Elvis() {
			}

			public void leaveTheBuilding() {
				System.out.println("Whoa baby, I'm outta here!");
			}

			// This code would normally appear outside the class!
			public static void main(String[] args) {
				Elvis elvis = Elvis.INSTANCE;
				elvis.leaveTheBuilding();
			}
		}
- 把构造器保持为私有，导出公有的静态成员，公有成员是个静态工厂方法  

		public class Elvis {
			private static final Elvis INSTANCE = new Elvis();

			private Elvis() {
			}

			public static Elvis getInstance() {
				return INSTANCE;
			}

			public void leaveTheBuilding() {
				System.out.println("Whoa baby, I'm outta here!");
			}

			// This code would normally appear outside the class!
			public static void main(String[] args) {
				Elvis elvis = Elvis.getInstance();
				elvis.leaveTheBuilding();
			}
		}

	这种Singleton实现可序列化仅仅加上implements Serializable是不够的，应提供一个readResolve方法

- 编写一个包含单个元素的枚举类型

		public enum Elvis {
			INSTANCE;

			public void leaveTheBuilding() {
				System.out.println("Whoa baby, I'm outta here!");
			}

			// This code would normally appear outside the class!
			public static void main(String[] args) {
				Elvis elvis = Elvis.INSTANCE;
				elvis.leaveTheBuilding();
			}
		}

### 四：通过私有构造器强化不可实例化的能力 ###

有时可能需要编写只包含静态方法和静态域变量的类，这种工具类不希望被实例化；企图通过将类声明为抽象类来强制该类不可被实例化是行不通的，因为抽象类可以被子类化，子类可以被实例化  

只要让这个类包含私有构造器，就不能被实例化了  

	public class UtilityClass {
		// Suppress default constructor for noninstantiability
		private UtilityClass() {
			throw new AssertionError();
		}
	}

因为子类的构造器都必须显示或隐式地调用父类的构造器，所以只包含私有构造器的类不能被子类化

### 五：避免创建不必要的对象 ###

最好能重用对象，而不是每次都创建一个新对象，如`String s = new String("aaa");` 能改成 `String s= "aaa";` 其中`"aaa"`就是一个String实例，虚拟机能保证只要包含相同的字符串常量，该字符串对象就会被重用，避免每次创建新的对象  

能用静态工厂方法的尽量不用构造器来创建对象，因为构造器肯定在每次调用时会创建一个新的对象  

可以用静态的初始化器来避免每次函数调用都创建一些新的实例对象，如：
	
	public class Person {
		private final Date birthDate;

		public Person(Date birthDate) {
			// Defensive copy - see Item 39
			this.birthDate = new Date(birthDate.getTime());
		}

		// Other fields, methods omitted

		// DON'T DO THIS!
		public boolean isBabyBoomer() {
			// Unnecessary allocation of expensive object
			Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
			gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
			Date boomStart = gmtCal.getTime();
			gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
			Date boomEnd = gmtCal.getTime();
			return birthDate.compareTo(boomStart) >= 0
					&& birthDate.compareTo(boomEnd) < 0;
		}
	}
可改为：

	class Person {
		private final Date birthDate;

		public Person(Date birthDate) {
			// Defensive copy - see Item 39
			this.birthDate = new Date(birthDate.getTime());
		}

		// Other fields, methods

		/**
		 * The starting and ending dates of the baby boom.
		 */
		private static final Date BOOM_START;
		private static final Date BOOM_END;

		static {
			Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
			gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
			BOOM_START = gmtCal.getTime();
			gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
			BOOM_END = gmtCal.getTime();
		}

		public boolean isBabyBoomer() {
			return birthDate.compareTo(BOOM_START) >= 0
					&& birthDate.compareTo(BOOM_END) < 0;
		}
	}

---

优先使用基本类型而不是装箱基本类型，当心无意识的自动装箱，如：

	public class Sum {
		// Hideously slow program! Can you spot the object creation?
		public static void main(String[] args) {
			Long sum = 0L;
			for (long i = 0; i < Integer.MAX_VALUE; i++) {
				sum += i;
			}
			System.out.println(sum);
		}
	}
其中，变量sum被声明成Long而不是long，这就使程序构造了大量的多余Long实例，效率下降

**小对象的创建和回收是非常廉价的，通过创建必要的对象，提升程序的清晰性、简洁性和功能性，通常是好事**
 
### 六：消除过期的对象引用 ###

如，栈内部维护着对象的过期引用：

	public class Stack {
		private Object[] elements;
		private int size = 0;
		private static final int DEFAULT_INITIAL_CAPACITY = 16;

		public Stack() {
			elements = new Object[DEFAULT_INITIAL_CAPACITY];
		}

		public void push(Object e) {
			ensureCapacity();
			elements[size++] = e;
		}

		public Object pop() {
			if (size == 0)
				throw new EmptyStackException();
			return elements[--size];
		}

		/**
		 * Ensure space for at least one more element, roughly doubling the capacity
		 * each time the array needs to grow.
		 */
		private void ensureCapacity() {
			if (elements.length == size)
				elements = Arrays.copyOf(elements, 2 * size + 1);
		}
	}
Stack类自己管理内存，数组活动区域中的元素是已分配的，而数组其余部分的元素则是自由，但是垃圾回收机制并不知道这一点，造成内存泄露。可优化为：
	
	public Object pop() {
		if (size == 0)
			throw new EmptyStackException();
		Object result = elements[--size];
		elements[size] = null; //Eliminate obsolete reference
		return result;
	}
	
内存泄漏的另外两个可能来源是 **缓存**和 **监听器、其他回调**

### 七：避免使用终结方法 ###

终结方法(finalizer)通常是不可预测的，也是很危险的，一般情况下是不必要的。它的缺点在于不能保证会被及时地执行，甚至不会被执行：从一个对象变得不可到达时，到它的终结方法被执行，这段时间是任意长的。所以不能用终结方法来关闭已经打开的文件等注重时间的任务

通常的做法是提供一个 **显示的终止方法**，要求类在实例不再有用时调用这个方法。显示终止方法必须在一个私有域中记录下“该对象已经不再有效”，其他方法调用时检查此私有域，如果在对象被终止后调用了其他方法，就抛出异常。典型的例子就是`close()`和`cancel()`方法，并与`try-finally`块结合使用

终结方法(finalizer)的两个用途：当忘记调用显示的终止方法时，充当“安全网”；本地对等体(native peer)拥有的非关键资源的回收。除了这两个用途，应尽量避免使用它；在子类使用终结方法的时候，应记得调用`super.finalize()`

# 3. 对于所有对象都通用的方法 #

### 八：覆盖equals时请遵守通用约定 ###

具有自己特有的“逻辑相等”的类需要覆盖equals方法。但有一种“值类”比较特殊，不需要覆盖equals方法，即实例受控的类（一），确保了“每个值最多只存在于一个对象”，可直接使用==代替equals，如枚举类型  

覆盖equals方法时，需要 **遵守约定**：  

>- 自反性。x.equals(x)必须返回true
>- 对称性。当且仅当x.equals(y)为true，y.equals(x)也为true
>- 传递性。x.equals(y), y.equals(z) -> x.equals(z)
>- 一致性。在x,y不变的情况下，多次调用x.equals(y)，返回的结果是确定的
>- x.equals(null)必须为false。约定equals方法不允许抛出NullPointerException异常。一般都会在equals方法开头对参数类型进行检查`if(!(object instanceof MyType))  return false;` 这就直接包括了对null的检查

实现equals方法的 **一般步骤**：

>1. 首先，== 操作符检查是否为“引用”相等，是一种性能优化
>2. instanceof 操作符检查参数是否为正确的类型，不是则返回false（此步骤包含了null检查）
>3. 把参数转换为正确的类型，强制转换`(MyType)object`
>4. 比较类中需要检测的域，判断是否逻辑相等（对float,double类型的判断，可使用Float/Double.compare方法）
>5. 编写单元测试，检查 **自反性、对称性、传递性、一致性**

覆盖equals需要注意：

>- 覆盖equals时总要覆盖hashCode（九）
>- 不要企图让equals过于智能。过度追求等价关系，则很难遵守equals约定
>- equals方法的参数为Object类型，不要替换为其他类型，否则是重载而不是覆盖

### 九：覆盖equals时总要覆盖hashCode ###

关于hashCode的约定：

>- 只要equals方法没有被修改，对同一个对象调用多次hashCode方法都必须返回同一个整数
>- 如果两个对象`x.equals(y)`为true，则对x,y调用hashCode方法得到的整数必须相等
>- 如果两个对象`x.equals(y)`不为true，则对x,y调用hashCode方法得到的结果不一定要不同。但如果能给出不同的结果，能提高散列表的性能

根据第二条约定：相等的对象必须具有相等的散列码。故，在覆盖equals方法时，必须也覆盖hashCode方法

	public final class PhoneNumber {
		private final short areaCode;
		private final short prefix;
		private final short lineNumber;

		public PhoneNumber(int areaCode, int prefix, int lineNumber) {
			rangeCheck(areaCode, 999, "area code");
			rangeCheck(prefix, 999, "prefix");
			rangeCheck(lineNumber, 9999, "line number");
			this.areaCode = (short) areaCode;
			this.prefix = (short) prefix;
			this.lineNumber = (short) lineNumber;
		}

		private static void rangeCheck(int arg, int max, String name) {
			if (arg < 0 || arg > max)
				throw new IllegalArgumentException(name + ": " + arg);
		}

		@Override
		public boolean equals(Object o) {
			if (o == this)
				return true;
			if (!(o instanceof PhoneNumber))
				return false;
			PhoneNumber pn = (PhoneNumber) o;
			return pn.lineNumber == lineNumber && pn.prefix == prefix
					&& pn.areaCode == areaCode;
		}

		public static void main(String[] args) {
			Map<PhoneNumber, String> m = new HashMap<PhoneNumber, String>();
			m.put(new PhoneNumber(707, 867, 5309), "Jenny");
			System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
		}
	}
上面的例子中，打印出的结果是null，因为没有覆盖hashCode方法

--- 

好的散列函数一般使得不相等的对象产生不相等的散列码（提高散列性能），构造散列函数的 **步骤**如下：  

>1.构造非零值 `int result = xx;`  
>2.对每个关键域(equals涉及到的域)`f`，执行：  	

>>1. 计算int类型的散列码`c`  
	- 域是boolean类型，`c = (f ? 1 : 0)`  
	- 域是byte、char、short、int类型, `c = (int)f`  
	- 域是long类型，`c = (int)(f ^ (f >>> 32))`  
	- 域是float类型，`c = Float.floatToIntBits(f)`  
	- 域是double类型，`lf = Double.doubleToLongBits(f)` 再有 `c = (int)(lf ^ (lf >>> 32))`  
	- 域是一个对象引用，按照equals递归调用equals的方式，来递归调用hashCode计算c. 如果域为null，则 c = 0  
	- 域是一个数组，要把每个元素当作单独的域来处理。然后按照步骤2.2计算总的结果c  
>>2. 按照`result = 31 * result + c`，把步骤2.1计算得到的各个域的散列码c合并到result中  

>3.返回result  
>4.编写单元测试，检查“相等的实例是否具有相等的散列码”

冗余域（可以根据参与计算的其它域的值计算出来的域）可以不参与hashCode计算。而且hashCode计算中必须排除equals方法比较中没有用到的域

### 十：始终要覆盖toString ###
覆盖toString并不像遵守equals和hashCode约定那么重要，但是提供好的toString方法可以使类用起来更加舒适。当对象被传递给 **println, print, 字符串连接+, assert及其它打印方法**时，toString被自动调用

### 十一：谨慎地覆盖clone ###
（..........）

### 十二：考虑实现Comparable接口 ###
给实现了Comparable接口的对象数组a进行排序只需：`Arrays.sort(a);`  

如果正在编写一个值类，它具有明显的内在排序关系，如按字母顺序、数值顺序或年代顺序，就应该考虑实现Comparable接口

	public interface Comparable<T>{
		int compareTo(T t);
	}

实现Comparable接口的 **约定**（其中，当表达式分别为负、0、正值时，sgn函数分别返回-1, 0, 1）:  
>- 对称性。必须满足，sgn(x.compareTo(y)) == -sgn(y.compareTo(x))。当且仅当y.compareTo(x)抛出异常时，x.compareTo(y)才必须抛出异常
>- 传递性。必须满足，(x.compareTo(y) > 0 && y.compareTo(z) > 0) => x.compareTo(z) > 0
>- 必须满足，x.compareTo(y) == 0，则对所有的z，都满足sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 
>- 与equals一致。建议，(x.compareTo(y) == 0) == (x.equals(y)), 但并不是绝对必要。如果违反了这个条件，就应该在注释中给以说明：“注意：该类具有内在的排序功能，但是与equals不一致。” 

equals方法的参数是Object类型，所以在函数内部需要进行类型检查；而compareTo方法的参数类型是静态的，不符合类型的参数无法通过编译，所以不需要在函数内进行类型检查

基本类型使用< > 操作符进行比较。浮点类型用Double.compare或Float.compare。数组比较，需要应用到每个元素上

# 4. 类和接口 #
### 十三：使类和成员的可访问性最小化 ###

**信息隐藏、封装**是软件设计的基本原则之一

- 尽可能地使每个类或成员不被外界访问。
	
	如果类或接口能够做成包级私有的，就应该被做成包级私有。  
	如果一个包级私有类或接口只是在某一个类的内部被用到，就应该考虑改成那个类的私有嵌套类（二十二）。

如果一个方法覆盖了父类中的方法，子类中方法的访问级别就不能低于父类方法的访问级别，这样保证了使用父类实例的地方都可以使用子类实例（为多态性考虑）。所以，实现了接口的方法也必须都是public的（接口中定义的所有方法都默认是public的）

不能为了测试，而将类、接口或者成员变成public的。变成包级私有，还可以接受

**实例域绝不能是公有的**：否则，等于放弃了这个域不可变的能力；当这个域被修改时，也失去了对它采取任何行动的能力。 所以，**包含公有可变域的类并不是线程安全的**  

长度非零的数组总是可变的，返回公有的static final数组域总是错误的。是漏洞的一个常见根源，如：
	
	public static final Thing[] VALUES = {...};

可改成：
	
	private static final Thing[] PRIVATE_VALUES = {...};
	public static final List<Thing> VALUES = 
		Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
或者改成：

	private static final Thing[] PRIVATE_VALUES = {...};
	public static final Thing[] values(){
		return PRIVATE_VALUES.clone();
	}

总之，除了公有static final域的特殊情况，公有类都不应该包含公有域，并且要确保公有static final域所引用的对象都是不可变的

### 十四：在公有类中使用访问方法而非公有域 ###
用包含私有域和公有访问方法(get)、设值方法(set)来代替类中的公有域，如：

	class Point {
		private double x;
		private double y;

		public Point(double x, double y) {
			this.x = x;
			this.y = y;
		}

		public double getX() {
			return x;
		}

		public double getY() {
			return y;
		}

		public void setX(double x) {
			this.x = x;
		}

		public void setY(double y) {
			this.y = y;
		}
	}
公有类永远都不应该暴露可变的域（暴露不可变的域危害比较小）。有时候会用包级私有或者私有的嵌套类来暴露域，无论域是可变还是不可变的

### 十五：使可变性最小化（不可变类） ###
**不可变类**：其实例不能被修改的类，实例包含的所有信息必须在创建时就提供，并在整个生命周期中固定不变

类要成为不可变，应该 **遵循五条规则**：
>1. 不要提供任何会修改对象状态的方法（相对外部可见而言）
>2. 保证类不会被子类化。一般的做法是使其成为final的，或将构造器指定为私有的（用静态工厂方法代替构造器）
>3. 所有的域都是final的
>4. 所有的域都是私有的（可以允许只包含基本类型或不可变对象引用的公有final域，但不建议这么做）
>5. 确保对于任何可变组件的互斥访问。如果类具有指向可变对象的域，则必须确保它不被暴露

- 不可变对象本质上是线程安全的，不要求同步。不可变对象可以被自由地共享（永远也不需要进行保护性拷贝）
- 不仅可以共享不可变对象，甚至可以共享它们的内部信息
- 不可变对象为其他对象提供了大量的构件
- **唯一的缺点**：对于每一个不同的值都需要一个单独的对象

类应该尽量做成不可变的，除非类有可变的需求；如果类不能被做成不可变的，也应该尽量限制它的可变性

### 十六：复合（组合）优先于继承 ###
此条目讨论的是具体类的继承（类继承类），不包括接口继承（类实现接口）和接口扩展（接口扩展另一个接口）

- 继承打破了封装性。子类依赖于其父类的实现细节，子类需要跟着其父类更新而演变。如：


	public class InstrumentedHashSet<E> extends HashSet<E> {
		// The number of attempted element insertions
		private int addCount = 0;

		public InstrumentedHashSet() {
		}

		public InstrumentedHashSet(int initCap, float loadFactor) {
			super(initCap, loadFactor);
		}

		@Override
		public boolean add(E e) {
			addCount++;
			return super.add(e);
		}

		@Override
		public boolean addAll(Collection<? extends E> c) {
			addCount += c.size();
			return super.addAll(c);
		}

		public int getAddCount() {
			return addCount;
		}

		public static void main(String[] args) {
			InstrumentedHashSet<String> s = new InstrumentedHashSet<String>();
			s.addAll(Arrays.asList("Snap", "Crackle", "Pop"));
			System.out.println(s.getAddCount());
		}
	}

我们期望上面的程序输出InstrumentedHashSet中的元素个数为3，但实际上会输出6. 因为在其父类HashSet中，addAll方法是调用add方法来实现的，所以在子类中，会出现重复计数的情况

导致子类脆弱的另一个原因是：它们的父类在后续的发行版本中可以获得新的方法，很可能导致子类无法正常工作

- 可以使用 **复合（组合）**和 **转发**的方法来代替继承，实现类本身和转发类（forwarding class），如：

	
	public class ForwardingSet<E> implements Set<E> {
		private final Set<E> s;

		public ForwardingSet(Set<E> s) {
			this.s = s;
		}

		public boolean add(E e) {
			return s.add(e);
		}
		
		public boolean addAll(Collection<? extends E> c) {
			return s.addAll(c);
		}
	}
	
	public class InstrumentedSet<E> extends ForwardingSet<E> {
		private int addCount = 0;

		public InstrumentedSet(Set<E> s) {
			super(s);
		}

		@Override
		public boolean add(E e) {
			addCount++;
			return super.add(e);
		}

		@Override
		public boolean addAll(Collection<? extends E> c) {
			addCount += c.size();
			return super.addAll(c);
		}

		public int getAddCount() {
			return addCount;
		}

		public static void main(String[] args) {
			InstrumentedSet<String> s = new InstrumentedSet<String>(
					new HashSet<String>());
			s.addAll(Arrays.asList("Snap", "Crackle", "Pop"));
			System.out.println(s.getAddCount());
		}
	}
以上的程序输出的结果为所期望的值3，不会发生重复计数的现象

每一个InstrumentedSet实例都把另一个Set实例包装起来了，所以InstrumentedSet又称为 **包装类**，上面的程序正是 **Decorator（包装类）模式**，对一个集合进行了装饰，并增加了计数特性。包装类几乎没有缺点，但不适合用在回调框架中

只有当 **A is-a B**时，A才应该继承于B；否则，就应该用复合代替继承

###  十七：要么为继承而设计，并提供文档，要么就禁止继承 ###
为继承而专门设计的类，必须有文档精确地描述 **覆盖每个方法**所带来的影响，即说明 **可覆盖的方法的自用性**（例子见十六）；且对于这种类，唯一的测试方法就是编写子类，必须在发布类之前先编写子类对类进行测试

为了允许继承，**构造器决不能调用可被覆盖的方法**，如：

	public class Super {
		// Broken - constructor invokes an overridable method
		public Super() {
			overrideMe();
		}

		public void overrideMe() {
		}
	}

	public final class Sub extends Super {
		private final Date date; // Blank final, set by constructor

		Sub() {
			date = new Date();
		}

		// Overriding method invoked by superclass constructor
		@Override
		public void overrideMe() {
			System.out.println(date);
		}

		public static void main(String[] args) {
			Sub sub = new Sub();
			sub.overrideMe();
		}
	}
上面这段程序，期待的是会有两次输出，但实际上第一次输出的是Null，因为overrideMe方法被Super调用的，Sub的构造器还没有开始初始化date

同理，实现Cloneable和Serializable接口时，clone和readObject方法都不可以调用可覆盖的方法

---

所以，对于那些并非为了安全地进行子类化而设计和编写文档的类，要禁止子类化（声明为final或私有化构造器）。可以用包装和转发的复合方法来代替。或者，不禁止子类化，但要确保这个类永远不会调用它的任何可覆盖的方法（完全消除可覆盖方法的自用特性）

### 十八：接口优于抽象类 ###
- 现有的类可以很容易被更新，以实现新的接口。扩展抽象类时，必须把抽象类放到类型层次的高处，这可能伤害到类层次；而实现接口，则不管这个类处于类层次的哪个位置
- 接口是定义mixin（混合类型）的理想选择。即，接口可以多“继承”，抽象类只能单继承
- 接口允许构造非层次结构的类型框架。(....)

(........)

### 十九：接口只用于定义类型 ###
有一种接口称为 **常量接口**，没有任何方法，只包含静态的final域。常量接口模式是对接口的不良使用。

要导出常量，可以：
- 如果常量与某个现有的类或接口相关，就应该把这些常量加入到这些类或接口中
- 枚举类型
- 不可实例化的工具类，如：
		
	
	public class PhysicalConstants {
		private PhysicalConstants() {
		} // Prevents instantiation

		// Avogadro's number (1/mol)
		public static final double AVOGADROS_NUMBER = 6.02214199e23;

		// Boltzmann constant (J/K)
		public static final double BOLTZMANN_CONSTANT = 1.3806503e-23;

		// Mass of the electron (kg)
		public static final double ELECTRON_MASS = 9.10938188e-31;
	}（可以用静态导入机制来避免必须用类名来修饰这些常量）

### 二十：类层次优于标签类 ###
标签类，如下：

	class Figure {
		enum Shape {
			RECTANGLE, CIRCLE
		};

		// Tag field - the shape of this figure
		final Shape shape;

		// These fields are used only if shape is RECTANGLE
		double length;
		double width;

		// This field is used only if shape is CIRCLE
		double radius;

		// Constructor for circle
		Figure(double radius) {
			shape = Shape.CIRCLE;
			this.radius = radius;
		}

		// Constructor for rectangle
		Figure(double length, double width) {
			shape = Shape.RECTANGLE;
			this.length = length;
			this.width = width;
		}

		double area() {
			switch (shape) {
			case RECTANGLE:
				return length * width;
			case CIRCLE:
				return Math.PI * (radius * radius);
			default:
				throw new AssertionError();
			}
		}
	}
这种类充斥着枚举声明、标签域以及条件语句，过于冗长、容易出错、效率低下

实际上，标签类正是类层次的一种简单的仿效。可以将标签类转变成类层次，将标签类中的方法、公共变量都放到一个抽象类；然后为每种原始标签都定义一个子类，如：

	abstract class Figure {
		abstract double area();
	}

	class Circle extends Figure {
		final double radius;

		Circle(double radius) {
			this.radius = radius;
		}

		double area() {
			return Math.PI * (radius * radius);
		}
	}

	class Rectangle extends Figure {
		final double length;
		final double width;

		Rectangle(double length, double width) {
			this.length = length;
			this.width = width;
		}

		double area() {
			return length * width;
		}
	}
标签类很少有适用的时候，当想要编写一个包含显式标签域的类时，都应该考虑是否可以用类层次来代替

### 二十一：用函数对象表示策略
此条主要讲解 **策略模式**，可类比C语言里的 **函数指针**

策略接口：

	public interface Comparator<T> {
		public int compare(T t1, T t2);
	}

具体策略类往往使用匿名类(二十二)声明：

	Arrays.sort(array, new Comparator<String>() {
		public int compare(String s1, String s2) {
			return s1.length() - s2.length();
		}
	});

但是使用匿名类会在每次调用时都创建一个新的实例，可考虑用静态工厂方法，存储到“宿主类”的一个私有的static final域中：

	class Host {
		private static class StrLenCmp
			implments Comparator<String> {
				public int compare(String s1, String s2) {
					return s1.length() - s2.length();
				}
			}
		
		public static final Comparator<String> 
			STRING_LENGTH_COMPARATOR = new StrLenCmp();
	}

函数指针的主要用途就是实现策略模式。在Java中，要声明一个接口来表示该策略，并为每个具体的策略声明一个实现接口的类。当具体的策略类只会被用到一次时，通常使用匿名类；当会被重复使用时，一般声明为另一个宿主类的一个静态成员类，并用public static final域导出实例

### 二十二：优先考虑静态成员类

嵌套类四种：静态成员类、非静态成员类、匿名类、局部类，除静态成员类的其它三种合称为**内部类**。嵌套类应只为外围类服务

- 静态成员类
	- 与其它静态成员一样，遵守同样的可访问性规则
	- 常见用法是作为公有的辅助类，仅当与外部类一起使用才有意义。如Calculator.Operation.PLUS
	- 私有静态成员类常见用法是用来代表外围类的对象组件
	- 实例与外围类的实例独立，没有引用指向外围实例来进行关联

- 非静态成员类
	- 语法上只是少一个static，但用法区别很大
	- 非静态成员类每个实例都隐含着一个外围实例，在没有外围实例的情况下，不可能创建非静态成员类的实例（只可能是静态成员类的实例）
	- 常见用法是定义Adapter
	- 如果不要求访问外围实例，就**优先定义成静态成员类**。因为非静态成员类的实例包含一个额外的指向外围实例的引用，消耗时间和空间，而且会导致外围实例在垃圾回收时仍有引用

- 匿名类
	- 不是外围类的一个成员，不与外围类的其它成员一起声明，而是在使用的同时被声明和实例化
	- 不可能拥有任何静态成员
	- 当且仅当出现在非静态环境中，才有外围实例
	- 常见用法是动态创建函数对象（二十一）和创建过程对象（如Runnable, Thread）和在静态工厂方法内部使用

- 局部类
	- 用的最少的嵌套类
	- 在任何可以声明局部变量的地方都可以由局部类，遵守作用域规则
	- 与成员类一样，有名字，可重复被使用；与匿名类一样，当且仅当出现在非静态环境中，才有外围实例，且不可能拥有任何静态成员

当一个类需要在方法外可见或者太长不适合放在方法内部，就应该声明为成员类。如果不需要每个实例都有指向外围实例的引用，则应该声明为静态成员类。如果方法内部的类只需要在一个地方创建实例，则应该用匿名类；否则用局部类

# 5. (....)

# 7. 方法
### 三十八： 检查参数的有效性

- 一般方法若接收到了无效的参数值，应该抛出异常，并且应在Javadoc的@throws标签中说明
- 未被导出的方法因为只有自己使用，可以使用断言来检查参数
- 有些参数被保存起来供以后使用，如构造器的参数。检查这种参数的有效性是很重要的
- 方法应该设计得对参数的限制越少越好，这些限制应该写在文档中，并在方法开头进行检查

### 三十九：必要时进行保护性拷贝

	public final class Period {
		private final Date start;
		private final Date end;

		/**
		 * @param start
		 *            the beginning of the period
		 * @param end
		 *            the end of the period; must not precede start
		 * @throws IllegalArgumentException
		 *             if start is after end
		 * @throws NullPointerException
		 *             if start or end is null
		 */
		public Period(Date start, Date end) {
			if (start.compareTo(end) > 0)
				throw new IllegalArgumentException(start + " after " + end);
			this.start = start;
			this.end = end;
		}

		public Date start() {
			return start;
		}

		public Date end() {
			return end;
		}
		
		public String toString() {
			return start + " - " + end;
		}
	}
如上，该类看上去可以表示不可变的一段时间周期，但是Date对象本身是可变的，可在外部对传入的Date进行修改，会导致Period内部还是能被修改的。如：

	Date start = new Date();
	Date end = new Date();
	Period p = new Period(start, end);
	end.setYear(xx);

所以，为了避免这种情况，应该**对构造器的每个可变参数进行保护性拷贝**。即改成：↓

	public Period(Date start, Date end) {
		this.start = new Date(start.getTime());
		this.end = new Date(end.getTime());

		if (start.compareTo(end) > 0)
			throw new IllegalArgumentException(start + " after " + end);	
	}
保护性拷贝是在参数有效性检查之前的，检查针对的是拷贝后的对象。对于参数类型可以被子类化的参数，不要使用clone方法进行保护性拷贝

但是，公有访问方法还是会有漏洞，如：

	Period p = new Period(start, end);
	p.end().setYear(xx);

防止这种情况，可以将返回值也改为保护性拷贝后的对象：

	public Date end() {
		return new Date(end.getTime());
	}
而这种非构造器中的拷贝可以使用end.clone()方法，因为在之前已经确保了end对象是一个Date类型，而不是其子类型

若上述类的域是不可变的，则不会出现上述的这些问题，比如说String类型和基本类型。若有可变的域，除非规定并且信任其不会被修改，否则就必须进行保护性拷贝

### 四十：谨慎设计方法签名

- 方法命名。 遵守（五十六）命名习惯
- 不要过于追求提供便利的方法
- 避免过长的参数列表。 四个或者更少，三种方法可以缩短参数列表：
	- 方法分解成多个方法。 而且可以通过提升**正交性**减少方法数目
	- 创建辅助类。 将多个参数合并到一个类中再传递
	- Builder模式（见第二条）
- 参数类型应该尽量选用接口（父类）而不是类（子类），这样可以扩大传入参数的类型范围
- boolean类型的参数都应该用两元素的枚举类型来代替，可读性更强

### 四十一：慎用重载

	public class CollectionClassifier {
		public static String classify(Set<?> s) {
			return "Set";
		}

		public static String classify(List<?> lst) {
			return "List";
		}

		public static String classify(Collection<?> c) {
			return "Unknown Collection";
		}

		public static void main(String[] args) {
			Collection<?>[] collections = { new HashSet<String>(),
					new ArrayList<BigInteger>(),
					new HashMap<String, String>().values() };

			for (Collection<?> c : collections)
				System.out.println(classify(c));
		}
	}
上述程序，由于重载的**静态分派**，不会打印"Set, List, Unknown Collection", 而是打印三次"Unknown Collection"。 调用哪个重载方法是在编译时就确定的

	class Wine {
		String name() {
			return "wine";
		}
	}

	class SparklingWine extends Wine {
		@Override
		String name() {
			return "sparkling wine";
		}
	}

	class Champagne extends SparklingWine {
		@Override
		String name() {
			return "champagne";
		}
	}

	public class Overriding {
		public static void main(String[] args) {
			Wine[] wines = { new Wine(), new SparklingWine(), new Champagne() };
			for (Wine wine : wines)
				System.out.println(wine.name());
		}
	}
而如上程序，重写（覆盖），因为是**动态分派**，会打印出不同的结果。若想在第一个程序中打印不同结果，就必须用instanceof来显示检测类型做不同处理

所以，在重载时要谨慎：**永远不要导出两个具有相同参数数目的重载方法**。在普通方法中，若有相同参数数目的重载方法，则可以通过修改方法名来避免；而对构造器来说，可以通过使用静态工厂方法来避免

在引入自动装箱和泛型后，重载得更为谨慎。如List<Integer>.add(i)方法，搞不清楚i是在自动装箱后作为元素加入，还是作为在指定位置处加入，即容易混淆add(Integer)和add(int)方法

### 四十二：慎用可变参数
(.....)

### 四十三：返回零长度的数组或者集合，而不是null

	private final List<Cheese> cheesesInStock = ...;
	
	public Cheese[] getCheeses(){
		if(cheesesInStock.size() == 0)
			return null;
		...
	}
如上代码，当长度为0时返回了null值。 这就会让客户端必须对此null返回值进行额外的检查和处理。对集合也是如此，应该返回零长度的数组或集合，而不是null

- 虽然返回数组和集合有一点开销，但是在这个级别上考虑性能问题是不明智的
- 零长度数组和集合是不可变的，可以被自由地共享

### 四十四：为所有导出的API元素编写文档注释

- 在每个被导出的类、接口、构造器、方法和域声明之前增加文档注释。可序列化的类则对它的序列化形式编写文档
- 方法的注释应简洁地描述它和客户端之间的约定
	- 做了什么，而不是怎么做的
	- 前提条件（调用此方法之前需满足的条件）和后置条件（调用此方法之后需满足的条件）
	- 副作用
	- 线程安全性
	- 让方法的每个参数都有@param，让方法有一个@return(除void)，该方法抛出的每个异常都有@throws（Exception if..）。描述均没有结束句点
	- {@code}标签插入代码
	- {@literal}标签处理特殊会产生歧义的字符
- 文档注释的第一句话是概要描述
- 包含泛型处，应该说明所有的类型参数
- 枚举类型的文档注释，确保说明常量、类型和公有方法
- 注解类型的文档，说明所有成员以及类型本身
- javadoc有继承注释的能力，接口优先于父类

# 8. 通用程序设计
### 四十五：将局部变量的作用域最小化

java允许在任何可以出现语句的地方声明变量，不应该全部挤在代码块的开头处声明

- 使局部变量作用域最小化，应在第一次使用它的地方声明，而不应该过早声明
- 每个局部变量的声明都应该包含一个有意义的初始化表达式，如果还没有得到有意义的初始化值，就应该推迟声明
- for循环可使得循环变量作用域最小化，且代码更简短、可读性更高，所以优先于while循环
- 作用域最小化可以避免因“复制-粘贴”带来的隐藏的BUG，第一个代码块中的变量在粘贴得来的第二个代码块中不起作用，编译报错

### 四十六：for-each风格循环优先于传统for循环

	for (Iterator<Suit> i = suits.iterator(); i.hasNext();)
			for (Iterator<Rank> j = ranks.iterator(); j.hasNext();)
				deck.add(new Card(i.next(), j.next()));
如上述代码，本意是在每个内循环中使用同一个Suit，但明显i.next()写在了内循环里面不会得到期望的结果。应该把i.next()移到内层循环外面，这样就会多一个变量声明。而若改成：
	
	for(Suit suit: suits)
		for(Rank rank: ranks)
			deck.add(new Card(suit, rank));
即简洁又不容易出错，并且没有性能损失

但，有三种情况不能使用for-each风格的循环：

- 过滤。 需要删除集合中的元素，得调用Iterator.remove()
- 转换。 需要更新集合中的元素，得通过迭代器拿到索引值
- 平行迭代。 平行遍历多个集合，需要显式控制迭代器、索引变量，使其同步前移

### 四十七：了解和使用类库

- 通过使用标准类库，可以充分利用这些编写标准类库的专家的知识，以及在之前的其他人的使用经验
- 在开发时避免过多花时间在细节上（其实还是要懂原理）
- 标准类库的性能一般都很高
- 使自己的代码易被别人接受，因为大家都用一套标准类库

### 四十八：如果需要高精确度，避免使用float,double

追求精确度，使用BigDecimal（代替float, double）或者 int, long（自己控制小数点，即整数和小数部分都用整型代替）

### 四十九：基本类型优先于装箱类型

基本类型和装箱基本类型三个区别：

- 基本类型只有值，值相等则相等；装箱类型不仅仅有值，值相等但同一性不一定相等
- 基本类型只有值，装箱类型还有null
- 基本类型比装箱类型更节省时间和空间

三个常见问题：

- 对装箱基本类型运用==操作符几乎总是错误的，它检测同一性而不是值是否相等 
- 当在一项操作中混合使用基本类型和装箱类型时，装箱类型会自动拆箱，而未手动初始化的装箱类型的默认值是null,自动拆箱会异常
- 另外，注意（第五条）中潜在的自动装箱情况，导致性能下降

使用装箱类型的场景：

- 作为集合中的元素、键和值
- 类型参数只能为装箱类型。 如`ThreadLocal<Integer>`，而不能是`ThreadLocal<int>`
- 反射的方法调用时，必须使用装箱类型

基本类型优先于装箱类型

### 五十：如果其他类型更适合，则尽量避免使用字符串

字符串不适合代替值类型(如int,float)、枚举类型、聚集类型(如数据类，最好不用字符串加上分隔符表示)、能力表

### 五十一：字符串连接性能

+号在每次连接时都会自动产生一个StringBuilder类，消耗巨大；应该手动创建一个StringBuilder进行连接，使用append()方法

### 五十二：通过接口引用对象

- 如果有合适的接口，应该用接口类型而不是类来声明参数、返回值、变量和域，即：

		List<Object> o = new Vector<Object>();

	而不是：

		Vector<Object> o = new Vector<Object>();
- 用接口作为类型，程序会更灵活，如想把Vector改成ArrayList，只需：

		List<Object> o = new ArrayList<Object>();
	
	而程序的其它部分不用改变
- 不适合用接口来引用对象的情况：
	- 没有合适的接口存在
	- 对象属于一个框架，框架的基本类型是类。这时也应该用基类来引用，而不是实现类
	- 类实现了接口，但提供了接口中不存在的额外方法。但这种类很少用作参数类型

### 五十三：接口优先于反射机制

通过反射机制可以构造底层类的实例、调用底层类的方法，并访问底层类的域。而且，反射可以使用一个还未被编译的类

缺点：
- 丧失了编译时类型检查的好处
- 执行反射访问所需要的代码笨拙又冗长
- 性能损失

反射最初是为了基于组件的应用工具设计的。这类工具通过装载类，利用反射功能找出它们支持的方法和构造器。普通应用程序在运行时不应该以反射方式访问对象

反射使用场景：
- 在编译时存在无法获取的类，但是存在它的接口或者父类。这时就可以以反射方式创建实例，而用接口或者父类访问这些实例。创建实例可用java.lang.reflect包或者Class.newInstance()

System.exit会终止整个虚拟机

### 五十四：谨慎使用本地方法

本地方法：用本地程序设计语言（如c/c++）来编写特殊的方法

本地方法三种用途：
- 访问特定于平台的机制。如注册表、文件锁
- 访问遗留代码库的能力
- 编写程序中注重性能的部分，提高系统的性能（不值得提倡，因为JVM已经越来越快）

本地方法缺点：
- 不安全。可能损坏内存
- 使得程序不再可自由移植
- 更难调试
- 进入和退出本地代码都需要开销，有可能反而降低性能
- 使得代码难以阅读

### 五十五：谨慎地进行优化

- 不要因为性能而牺牲合理的结构，努力编写好的程序而不是快的程序
- 主要的性能瓶颈问题应该在设计过程中考虑到，并在编写时避免
- 考虑API设计的性能问题。API的设计对于性能的影响是非常实际的
- 利用性能剖析器去找性能瓶颈所在

### 五十六：普遍接受的命名惯例

- 包名尽量用缩写
- 类名、接口和方法、域名都是驼峰命名，类、接口名首字母大写，方法和域首字母不大写
- 常量域全大写，单词间用_分隔
- 局部变量与域类似，但是允许缩写，取决于上下文环境（一般作用域越大，名字越长）
- 类型参数通常为单个字母（五种之一）：
	- T. 任意类型
	- E. 集合的元素类型
	- K. 键类型
	- V. 值类型
	- X. 异常类型
	- T1,T2,T3(T,U,V). 类型序列
		
# 9. 异常
### 五十七：只针对异常的情况才使用异常
		
	try{
		int i = 0;
		while(true)
			range[i++] = xx;
	}catch(ArrayIndexOutOfBoundsException e){
	}

如上述代码，试图用异常的捕获来终止正常的循环。所以：

- 异常应该只用于异常的情况下，永远不应该用于正常的控制流
- 设计良好的API不应该强迫客户端为了正常的控制流而使用异常
	
	一种方法是“状态测试方法”，迭代器中的hasNext()方法设计：  
		
		for(Iterator i = ... ; i.hasNext();)
			i.next();
	
	若缺少hasNext()方法，则会强制客户端使用异常：

		try{
			Iterato<T> i = x.iterator();
			while(true)
				i.next();
		}catch(Exception e){
		}
	
	另一种方法是“可识别的返回值”，如在调用next()方法时，若达到尾部，可返回一个特殊值标识已到达尾部，客户端可以对此返回值进行识别

### 五十八：对可恢复的情况使用受检异常，对编程错误使用运行时异常

- 如果期望调用者能够适当地恢复，则应该使用受检异常
	- 强迫调用者在catch语句中处理异常，或者上抛
	- 调用者理论上可以只捕获却不进行处理，但是这种做法不推荐（六十五）
- 对不可恢复的情形（继续执行下去有害无益），则使用未受检的异常（运行时异常）
	- 如果程序没有捕获这种异常，程序会停止运行（对于不确定有没有可能恢复的情况，最好使其停止运行）
	- 用运行时异常来表明编程错误
	- 不要再实现Error子类，未受检异常都应该使用RuntimeException子类
- 异常也是对象，可以在其上面定义任意方法，主要是为了提供额外的异常信息
- 也可以在异常上定义一些辅助方法，如：一用户因余额不足而产生拨号异常，这时可以在此异常上提供一个查询余额的方法

### 五十九：避免不必要地受检异常

过分使用受检的异常会使API使用起来非常不方便

(...........)

### 六十：优先使用标准的异常

重用现有的标准异常：
- API易于使用，与其他人习惯一致
- 可读性更好，不会出现不熟悉的异常
- 异常类越少，装载这些类的时间消耗就越小

常见的异常：
- IllegalArgumentException. 非法参数的异常
- IllegalStateException. 非法状态的异常，比如在使用某个对象前必须初始化某个状态
- NullPointerException
- IndexOutOfBoundsException
- ConcurrentModificationException. 一个要求线程安全性的对象正在被未同步的并发修改，就抛出这个异常
- UnsupportedOperationException. 对象不支持某个操作

只要有某个异常满足需要，就应该重用上述的现有异常（如果希望提供更多的异常信息，则可以子类化这些异常）

### 六十一：抛出与抽象相对应的异常

- 如果能阻止低层异常，尽量避免低层方法抛出异常
- 异常转译(更高层的实现应该捕获低层的异常，同时抛出可以按照高层抽象进行解释的异常)，如：

		try{
			... // lower exception
		}catch(LowerLevelException e){
			throw new HigherLevelException(...);
		}
- 异常链与支持链（使得可以用getCause访问异常原因）

### 六十二：每个方法抛出的异常都要有文档

- 方法中单独声明受检的异常，并用@throws标记，记录每个异常的条件
	- 不要声明throws Exception, Throwable这种极端异常，毫无作用
- 同样，@throws在方法文档中记录每个未受检异常，但是不要声明在方法代码中 
- 如果一个类的许多方法由于同一个原因抛出同一个异常，需要在类的文档中说明该异常，而不是在每个方法文档中说明

