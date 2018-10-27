# 一、 走近java
# 1. 走近Java
jdk: java语言、java虚拟机、java API类库  
jre: java SE API子集，java虚拟机  

4个平台：
- Java Card. java小程序运行在小内存设备的平台
- Java ME. java程序运行在移动终端的平台
- Java SE. 面向桌面级应用的平台
- Java EE. 使用多层架构的企业应用平台 

可考虑看jdk源码（C语言）

# 二、 自动内存管理机制
# 2. Java内存区域与内存溢出异常

### 2.2 运行时数据区域

- 程序计数器
	- 每个线程独自拥有一个（**线程私有**），可以看作是当前线程所执行的字节码的行号指示器。 字节码解释器通过改变此计数器的值，来选择下一条指令
	- 唯一一个没有规定OutOfMemoryError的内存区域
- Java虚拟机栈 
	- **线程私有**，每个方法在执行时创建一个栈帧，这些栈帧就在虚拟机栈中，所需内存大小在编译期已确定
	- 局部（本地）变量表：存放编译期可知的各种基本数据类型、对象引用、returnAddress, long/double型占两个slot，其它占一个
	- 栈帧局部变量表所需的内存空间在编译时已确定，分配时按照此值分配，运行期间不变
	- 线程请求的栈深度大于所允许的深度，则StackOverflowError
	- 虚拟机栈在动态拓展时没有足够内存，则OutOfMemoryError
- 本地方法栈
	- **线程私有**，与虚拟机栈类似，虚拟机栈为java方法服务，本地方法栈为Native方法服务
- Java堆
	- **线程公有**，在虚拟机启动时创建
	- 唯一目的：存放对象实例及数组
	- 垃圾收集器管理的主要区域，故又称'GC堆'
	- Java堆的内存是逻辑连续的
	- 内存不足以分配实例时，则OutOfMemoryError
	- 分新生代和老年代两个区域。老年代存放生命周期长的对象
- 方法区（又叫永久代）
	- **线程公有**，存储类信息、常量、静态变量等
	- 内存不足以分配时，则OutOfMemoryError
	- 运行时常量池：方法区的一部分，存放编译期生成的字面量和符号引用

**直接内存**：非运行时数据区域内存，java可用特殊方法直接分配堆外内存，不受Java堆大小的限制

### 2.3 虚拟机对象探秘

- 对象的创建
	- 类加载。首先在常量池中定位类的符号引用，检查此类是否被加载、解析和初始化；若没有，则先执行类的加载过程
	- 对象所需的内存在类加载后可确定。内存分配方式：指针碰撞（规整内存的java堆）、空闲列表（不规整内存的Java堆）
	- 分配内存的动作应该是线程安全的
	- 内存分配完成后，虚拟机将其初始化为0
	- 虚拟机对对象进行设置，存放在对象头中
- 对象的内存布局
	- 对象头。两部分：存储对象自身的运行时数据；类型指针（确定它是哪个类的实例）
	- 对象真正存储的有效信息（各字段内容等）
	- 对齐填充（无实际意义）
- 对象的访问定位
	- 通过栈上的reference（引用）数据来操作堆上的具体对象。两种访问方式：**使用句柄；直接指针**

### 2.4 OutOfMemoryError异常测试

- Java堆溢出
	- 虚拟机参数 -Xms20m -Xmx20m，将堆最小内存设为20M，最大Xmx与最小相等则代表不可拓展
	- 堆的内存不足，实例无法再获取到内存（java.lang.OutOfMemoryError: Java heap space）
	- 使用内存映像分析工具对堆快照进行分析。区分内存泄漏（对象不存活，但没有被垃圾回收）和内存溢出（对象都是存活的，只是内存不足）
- 虚拟机栈和本地方法栈溢出
	- 虚拟机参数 -Xss限制栈大小
	- 线程请求的栈深度大于虚拟机所允许的最大深度（即方法调用的嵌套深度），StackOverflowError异常（纵向，无法分配新的栈帧）
	- 虚拟机拓展栈时内存不足，OutOfMemoryError异常（横向，无法建立新的线程）
	- **单个线程**下，抛出的都是StackOverflowError异常；多线程下，每个线程分配到的栈容量越大，可建立的线程数目就越小，越容易出现OutOfMemoryError异常
- 方法区和运行时常量池溢出
	- 虚拟机参数 -XX:PermSize=  -XX:MaxPermSize= 限制方法区大小
	- String的intern()方法。如果字符串常量池中已经包含等于此对象的字符串，则返回池中的对象；否则，将新的String对象字符串添加至常量池中，并返回此对象的引用
	- 关于intern()细节，见PDF80
	- java.lang.OutOfMemoryError: PermGen space
- 直接内存溢出

# 3. 垃圾收集器与内存分配策略

线程私有的内存大小几乎是在编译期已确定的，这部分（程序计数器、虚拟机栈、本地方法栈）的内存随着方法或线程的结束自然会自动被回收，故垃圾收集器关注的**仅仅是动态的内存**（java堆和方法区）

### 3.2 对象存活

- 引用计数算法
	- 有引用时加1，引用失效时减一，为0时判断为不存活
	- 但很少主流虚拟机选用它，难解决对象循环引用问题（两个对象相互引用）
- 可达性分析算法
	- ‘GC ROOTS’作为起始点，搜索引用链，当对象到'GC ROOTS'没有引用链相连时（不可达），即不存活
	- 可解决对象之间相互引用的情况（如下图）
	- ![](https://github.com/limbo-china/books/blob/master/JAVA_VM/3-1.png)
- 再谈引用
	- 强引用、软引用、弱引用、虚引用，引用强度依次递减
- 回收
	- 当与‘GC ROOTS’不可达时，进行第一次标记，并判断是否需要（覆盖了finalize方法且没有被虚拟机调用过）执行finalize()
	- 需要执行finalize()的对象放在F-Queue中等待执行
	- 对F-Queue中的对象进行第二次标记，如果对象在finalize()方法中重新建立引用链，那么此对象将不会被回收（对象可以在被GC时自我拯救；这种自救的机会只有一次，因为一个对象的finalize()方法最多只会被系统自动调用一次）
		
			public class FinalizeEscapeGC {
			public static FinalizeEscapeGC SAVE_HOOK = null;

			public void isAlive() {
				System.out.println("yes, i am still alive :)");
			}

			@Override
			protected void finalize() throws Throwable {
				super.finalize();
				System.out.println("finalize mehtod executed!");
				FinalizeEscapeGC.SAVE_HOOK = this;
			}

			public static void main(String[] args) throws Throwable {
				SAVE_HOOK = new FinalizeEscapeGC();

				// 对象第一次成功拯救自己
				SAVE_HOOK = null;
				System.gc();
				// 因为Finalizer方法优先级很低，暂停0.5秒，以等待它
				Thread.sleep(500);
				if (SAVE_HOOK != null) {
					SAVE_HOOK.isAlive();
				} else {
					System.out.println("no, i am dead :(");
				}

				// 下面这段代码与上面的完全相同，但是这次自救却失败了
				SAVE_HOOK = null;
				System.gc();
				// 因为Finalizer方法优先级很低，暂停0.5秒，以等待它
				Thread.sleep(500);
				if (SAVE_HOOK != null) {
					SAVE_HOOK.isAlive();
				} else {
					System.out.println("no, i am dead :(");
				}
			}
			}
	运行结果：
	finalize mehtod executed!
	yes, i am still alive :)
	no, i am dead :(
	- finalize()能做的所有工作，可以用try-finally完全代替，即完全可以忘记finalize()的存在
- 回收方法区
	- 废弃常量。没有地方引用此常量，可回收
	- 无用的类。 该类的所有实例都已被回收（java堆中不存在该类实例）；加载该类的ClassLoader已被回收；对应的java.lang.Class没有被引用，无法通过反射访问。满足上述三点，可回收

### 3.3 垃圾收集算法

- 标记-清除算法
	- 标记出所有需要回收的对象，之后统一回收所有被标记的对象
	- 两个不足：标记和清除的效率不高；产生大量不连续的内存碎片
- 复制算法（一般用在新生代）
	- 内存分为两块。每当其中一块内存耗尽，将存活的对象复制至另一块
	- 运行高效，无内存碎片。代价是内存减少一半
	- **新生代**内存区域中，8/1/1的Eden/Survivor/Survivor区，将Eden和一块survivor存活的对象复制到另一块survivor上
- 标记-整理算法
	- 当存活率较高时，**复制算法**需要进行较多的复制操作，效率降低
	- 与标记-清理不同，标记后不清理，而是将存活的对象整理到一起
- 分代收集算法（现代GC基本采用这种）
	- 将内存划分为多块，每块根据对象的特点灵活采用上述的不同算法

### 3.4 HotSpot虚拟机算法实现
### 3.5 各种垃圾收集器介绍
serial, parnew, parallel scavenge, serial old, parallel old, cms, g1

java堆分为若干个区域  

- GC日志
- 垃圾收集器参数

### 3.6 内存分配与回收策略

Minor GC: 新生代GC，非常频繁，回收速度快  
Major GC/Full GC: 老年代GC，一般伴随至少一次的Minor GC，速度比Minor GC慢10倍以上

- 对象优先在新生代的Eden区分配
- 大对象直接进入老年代。避免新生代中大量的内存复制
- 长期存活的对象将进入老年代。对象有一个年龄计数器，每进入一次Survivor空间，计数器加1，到一定数值将被移入老年代
- 动态对象年龄判定。进入老年代的年龄阈值更灵活
- 空间分配担保。发生Minor GC前，检查老年代最大可用连续空间是否大于新生代所有对象总空间

# 4. 虚拟机性能监控与故障处理

### 4.2 jdk命令行工具

- jps, 虚拟机进程状况工具
- jstat, 虚拟机统计信息监视工具。jstat [ option vmid [interval [s|ms]] [count]] (类装载、内存、垃圾收集、JIT编译)
- jinfo, Java配置信息工具。实时查看和调整虚拟机各项参数， jinfo [option] pid
- jmap, Java 内存映像工具，生成堆转储快照。jmap [option] vmid
- jhat, 虚拟机堆转储快照分析工具，一般不使用
- jstack, Java堆栈跟踪工具。生成当前时刻线程快照，jstack [option] vmid
- HSDIS

### 4.3 JDK可视化工具

- JConsole
- VisualVM

# 5. 调优案例分析与实战

### 5.2 案例

- 高性能硬件上的程序部署策略。使用64JDK利用大内存；若干个32位虚拟机（每个虚拟机相当于一个Java进程）建立逻辑集群
- 集群间同步导致的内存溢出。大量的失败同步请求，占用内存
- 堆外内存导致的溢出错误。案例中，由于Direct Memory的内存不足，而又无法（像java堆那样）触发虚拟机的垃圾回收，所以只能抛出内存溢出异常。**堆外内存**可能导致内存溢出：直接内存、线程栈、Socket缓存区（Too many open files）、JNI代码、虚拟机和GC自身占用的内存
- 外部命令导致系统缓慢
- 服务器JVM进程崩溃。可检查系统日志，hs_err_pid####.log
- 不恰当的数据结构导致内存占用过大。HashMap导致内存占用过大，Minor GC频繁，耗时增加，主要因为HashMap用来存储一些小数据，空间利用率很低，内存浪费
- Windows虚拟内存

### 5.3 Eclipse运行速度调优
（....）

# 三、 虚拟机执行子系统
# 6. 类文件结构

存储**字节码**的Class文件

### 6.3 Class类文件的结构
《Java虚拟机规范（Java SE 7）》中文版  

字节流，按顺序紧凑排列，没有分隔符，超过占用8位的数据按大端存储（高位在前）

类C语言结构体的伪结构：无符号数（相当于基本类型）和表（相当于结构体）

Class文件格式：相当于一张大表

- 可用javap -verbose XXXClass解析Class文件
- 魔数与Class文件的版本
	- 魔数为头4个字节，确定该文件是否是能被虚拟机接受的Class文件
	- 之后是2个字节的次版本号和2个字节的主版本号（向下兼容）
- 常量池
	- 2个字节的常量池容量（从1开始计数，实际数量为此数值减1）
	- 常量池两大类常量：字面量（字符串、数值常量）和符号引用（类和接口的全限定名、字段的名称和描述符、方法的名称和描述符），每个常量都是一个表结构（共14中常量类型，共同点是第一个字节都是类型标志位）
- 访问标志
	- 两个字节，16位其中的8个位可标志访问信息（public、final、abstract、类或接口等信息），其它8位均为0
- 类索引、父类索引与接口索引集合
	- 记录类、接口继承关系
- 字段表集合
	- 描述接口、类中声明的变量（不包括方法内部的局部变量），包括访问标志、简单名称、描述符（变量类型、方法的参数）、额外属性表
- 方法表集合
	- 同字段表集合，方法的代码放在属性表中
	- 与java语言里不同，Class文件里方法的特征签名包括返回值，若两个方法只有返回值不同，也能共存
- 属性表集合
	- 方法表、字段表、类文件带的额外属性
	- **Code属性**。操作数栈深度最大值；局部变量表所需存储空间值；编译生成的字节码指令（虚拟机规定了200多条字节码指令）。最重要的一个属性，整个Class文件中，Code属性用于描述**代码**，所有其它数据用于描述**元数据**
	- Exceptions属性。与Code属性平级，列出可能抛出的异常（throws关键字后面的异常）
	- LineNumberTable属性。描述java源码行号与字节码偏移量的对应
	- LocalVariableTable属性。栈帧局部变量表中的变量与java源码定义的变量的对应
	- SourceFile属性。记录生成这个Class文件的源码文件名称（有时候class文件名称与源码文件名称不对应，如内部类和包级类）
	- ConstantValue属性。 只有被static关键字修饰的变量才使用，自动为静态变量赋值
	- InnerClasses属性。 记录内部类与宿主类之间的关联
	- Deprecated, Synthetic, StackMapTable, Signature, BootstrapMethods属性

### 6.4 字节码指令简介

指令由一个字节的操作码和其后的零至多个字节的操作数构成，面向操作数栈。

放弃操作数长度对齐，损失一些性能，但使得编译后的代码短小，高效率传输

	do {
		自动计算PC寄存器的值加1；
		根据PC寄存器的指示位置，从字节码流中取出操作码；
		if(字节码存在操作数) 从字节码流中取出操作数；
		执行操作码的操作；
	}while(字节码流长度 > 0);

- 字节码与数据类型
	- 指令大多包含了其操作所对应的数据类型信息
	- Java虚拟机指令集对于特定的操作只提供了有限的类型相关指令
- 加载和存储指令。 将数据在栈帧中的局部变量表和操作数栈之间来回传输
	- 将一个局部变量加载到操作数栈，xload
	- 将一个数值从操作数栈存储到局部变量表, xstore
	- 将一个常量加载到操作数栈
	- 扩充局部变量表访问索引，wide
- 运算指令
	- 注意浮点数运算
- 类型转换指令
- 对象创建与访问指令
	- 对类实例和数组的创建与操作使用了不同的指令
- 操作数栈管理指令
	- 操作数出栈
	- 复制栈顶并压入
	- 栈顶两个数值互换
- 控制转移指令
- 方法调用和返回指令
- 异常处理指令
	- 虚拟机中处理异常不由字节码完成，由异常表完成
- 同步指令 

### 6.5 公有设计和私有实现

**公有设计**：虚拟机规范描绘java虚拟机的共同程序存储格式：Class文件格式和字节码指令集。这些与硬件、软件及虚拟机实现是完全独立的

**私有实现**：在满足规范约束下对具体的虚拟机实现做修改和优化

虚拟机实现：将字节码翻译成另一种虚拟机指令集；将字节码翻译成宿主机CPU的本地指令集（**JIT代码生产技术**）

# 7. 虚拟机类加载机制

实际情况中，每个Class文件都可能代表java语言中的一个类或接口

### 7.2 类加载的时机

类生命周期：  
- 加载、验证、准备、解析、初始化、使用、卸载
- 其中验证、准备、解析统称为连接
- 解析阶段顺序不一定，可以在初始化之后；而加载、验证、准备一定在初始化之前

![](https://github.com/limbo-china/books/blob/master/JAVA_VM/7-1.png)

（之前未初始化）必须立即对类进行**初始化**的情况（**有且仅有**）：
- 遇到new、getstatic、putstatic、invokestatic字节码，在java语言中，一般是：使用new关键字实例化对象；读取、设置类的静态字段；调用一个类的静态方法
- java.lang.reflect包对类进行反射调用时
- 初始化时，其父类必须先经过初始化（接口只在这点与类不同，接口不要求其父接口都完成了初始化）
- 虚拟机启动时，先初始化主类（main方法所在类）
- java.lang.invoke.MethodHandle

触发初始化的引用类的方式，为主动引用；否则为被动引用：
- 通过子类引用父类的静态字段，不会导致子类初始化
- 通过数组定义来引用类，不会触发类的初始化
- 常量(final）在编译阶段会存入调用类的常量池，直接引用这种静态常量本质并没有引用到类，不会触发类的初始化

### 7.3 类加载的过程

- 加载
	- 通过类的全限定名获取定义此类的二进制字节流（可控性最强，可重写loadClass()方法）
	- 将字节流的静态存储结构转化为方法区的运行时数据结构
	- 在内存中生成代表此类的java.lang.Class对象，作为方法区访问入口
	- 数组类的加载与普通类有所不同(。。。。。。。。)
- 验证 
	- 确保字节流中包含的信息符合虚拟机的要求，因为字节流不一定由编译器产生，可以人工手动产生有害字节流
	- 文件格式验证：魔数、版本号、常量池常量类型、常量索引值等。此步验证通过后，字节流进入方法区，后面的验证步骤**基于方法区**
	- 元数据验证：语义分析，检查是否符合java语言规范
	- 字节码验证：分析数据流和控制流，对**方法体**进行验证，保证安全性
	- 符号引用验证：保证解析动作正常执行
- 准备
	- 正式为类变量（非实例变量）分配内存，


(........)

# 8. 虚拟机字节码执行引擎

### 8.2 运行时栈帧结构

每个方法从调用开始到完成，都对应着栈帧在虚拟机栈中从入栈到出栈的过程

对执行引擎来说，只有位于栈顶的栈帧才是有效的，为**当前栈帧**，其方法称为**当前方法**。执行引擎的所有字节码指令只对当前栈帧进行操作

- 局部表量表
	- 在编译时期，就确定了需要分配的局部变量表的最大容量
	- Slot为单位，long/double占两个，其它占一个
	- reference类型，虚拟机能通过这个引用找到java堆中数据存放的起始地址；并且也能通过此引用找到对象所属类在方法区中的类型信息
	- 实例（非static）方法局部变量表顺序：第0位是this参数；然后是显式参数；最后是方法体内部定义的变量，顺序均和定义的顺序相同
	- 实例方法的局部变量必须被手动初始化
	- 关于slot复用对垃圾收集的影响：
		
			public static void main(String[] args)() {
	            byte[] placeholder = new byte[64 * 1024 * 1024];
	            System.gc();
	        }
		上述代码中，placeholder没有被回收。因为placeholder还在作用域中，虚拟机自然不敢回收
			
			public static void main(String[] args)() {
                {
                    byte[] placeholder = new byte[64 * 1024 * 1024];
                }
                System.gc();
            }
		而这段代码中，placeholder依然没有被回收，是因为虽然已经离开了作用域，但是局部变量表的slot中仍然存在placeholder数组的引用，且**没有被其它变量复用**，所以依然判断为不回收
		
			public static void main(String[] args)() {
                {
                    byte[] placeholder = new byte[64 * 1024 * 1024];
                }
                int a = 0;
                System.gc();
            }
		而这段代码在离开作用域后，有一个其它变量a复用了placeholder的slot，使得其引用被解除，才得以回收。这就解释了为什么有的代码规范里，规定“作用域过期的对象应手动赋值为null”；但是，不应过多依赖于赋值null，以恰当的变量作用域来控制变量回收时间才是最优雅的解决方法
- 操作数栈（操作栈）
	- 最大深度在编译时已确定
	- 如加法指令，将栈顶两个元素出栈并相加，再将结果入栈
	- java虚拟机的基于栈的执行引擎，其中栈指的就是操作栈
- 动态连接
- 方法返回地址
	- 方法正常退出时，栈帧中保存调用者的PC计数器的值作为返回地址
	- 方法异常退出时，返回地址通过异常处理表来确定，栈帧中一般不保存
- 附加信息

### 8.3 方法调用 

Class文件编译过程中不包含传统编译的连接步骤，方法调用在Class文件中只是符号引用，不是实际内存布局中的入口地址（直接引用），直接引用需要在类加载期间甚至运行期间才能确定

- 解析
	- 解析期间，一部分符号引用转化为直接引用，此类方法称为**非虚方法**（包括静态方法、私有方法、实例构造器、父类方法四类，还有个特殊是final方法，即不可能通过继承或别的方式重写其它版本），其它则为**虚方法**
- 分派
	- 分派调用过程是多态性的最基本体现
	- 静态分派（重载）
			
			public class StaticDispatch {

                static abstract class Human {
                }

                static class Man extends Human {
                }

                static class Woman extends Human {
                }

                public void sayHello(Human guy) {
                    System.out.println("hello,guy!");
                }

                public void sayHello(Man guy) {
                    System.out.println("hello,gentleman!");
                }

                public void sayHello(Woman guy) {
                    System.out.println("hello,lady!");
                }

                public static void main(String[] args) {
                    Human man = new Man();
                    Human woman = new Woman();
                    StaticDispatch sr = new StaticDispatch();
                    sr.sayHello(man);
                    sr.sayHello(woman);
                }
            }
		上面代码将输出:  
		hello,guy!  
		hello,guy!  
		
		使用哪个重载版本，完全取决于传入参数的数量和静态数据类型，这个是编译器确定的。编译器在重载时通过参数的静态类型而不是实际类型作为判断依据，静态类型是编译器可知的。称为**静态分派**

			public class Overload {

                public static void sayHello(Object arg) {
                    System.out.println("hello Object");
                }

                public static void sayHello(int arg) {
                    System.out.println("hello int");
                }

                public static void sayHello(long arg) {
                    System.out.println("hello long");
                }

                public static void sayHello(Character arg) {
                    System.out.println("hello Character");
                }

                public static void sayHello(char arg) {
                    System.out.println("hello char");
                }

                public static void sayHello(char... arg) {
                    System.out.println("hello char ...");
                }

                public static void sayHello(Serializable arg) {
                    System.out.println("hello Serializable");
                }

                public static void main(String[] args) {
                    sayHello('a');
                }
            }
		上述代码输出: hello char  
		上述例子这种字面量参数的优先级为char->int->long->Character->Serializable->Object->char ...
	- 动态分派（重写）
		
			public class DynamicDispatch {

                static abstract class Human {
                    protected abstract void sayHello();
                }

                static class Man extends Human {
                    @Override
                    protected void sayHello() {
                        System.out.println("man say hello");
                    }
                }

                static class Woman extends Human {
                    @Override
                    protected void sayHello() {
                        System.out.println("woman say hello");
                    }
                }

                public static void main(String[] args) {
                    Human man = new Man();
                    Human woman = new Woman();
                    man.sayHello();
                    woman.sayHello();
                    man = new Woman();
                    man.sayHello();
                }
            }
		输出结果：  
		man say hello  
		woman say hello  
		woman say hello  

		动态分派，把常量池中的类方法符号引用解析到了不同的直接引用上，根据实际类型确定方法执行版本。invokevirtual指令的多态查找过程：一，找到操作栈顶的元素所指向的对象的实际类型C；二，在类型C中找到描述符和简单名称都相符的方法，进行权限校验，通过则返回此方法的直接引用，查找结束，否则java.lang.IllegalAccessError；三，否则，按继承关系从下至上对C的父类进行查找；四，若始终没有合适的方法，则报java.lang.AbstractMethodError异常
	- 单分派与多分派
		
			public class Dispatch {

                static class QQ {}

                static class _360 {}

                public static class Father {
                    public void hardChoice(QQ arg) {
                        System.out.println("father choose qq");
                    }

                    public void hardChoice(_360 arg) {
                        System.out.println("father choose 360");
                    }
                }

                public static class Son extends Father {
                    public void hardChoice(QQ arg) {
                        System.out.println("son choose qq");
                    }

                    public void hardChoice(_360 arg) {
                        System.out.println("son choose 360");
                    }
                }

                public static void main(String[] args) {
                    Father father = new Father();
                    Father son = new Son();
                    father.hardChoice(new _360());
                    son.hardChoice(new QQ());
                }
            }
		输出结果：  
		father choose 360  
		son choose qq  
		java是静态多分派，动态单分派  

	- 虚拟机动态分派实现

		一般建立一个虚方法表，提高查找性能。上节代码所对应的虚方法表如下：  
		![](https://github.com/limbo-china/books/blob/master/JAVA_VM/8-1.png)

		Son重写了来自Father的全部方法，因此Son的方法表没有指向Father类型数据的箭头。但Son和Father都没有重写来自Object的方法，所以都指向了Object的数据类型
- 动态类型语言支持

	（..........）

### 8.4 基于栈的字节码解释执行引擎

java虚拟机执行引擎分为**解释执行**（通过解释器执行）和**编译执行**（通过即时编译器产生本地代码）。刚开始java都是解释执行（解释Class文件）的，但是后来发展出可以直接产生本地代码的编译器（或者编译Class文件）

- 解释执行
	- 编译执行和解释执行的过程如图：
		
		![](https://github.com/limbo-china/books/blob/master/JAVA_VM/8-2.png)
- 基于栈的指令集和基于寄存器的指令集
	- 基于栈的指令集的优点是可移植、代码相对更紧凑。寄存器由硬件直接提供，程序不可避免地受到硬件的约束
	- 基于栈的指令集的缺点是执行速度相对稍慢，所需的指令数量较多
- 基于栈的解释器执行过程
	
		public int calc(){
			int a = 100;
			int b = 200;
			int c = 300;
			return (a + b) * c;
		}

	字节码指令如下：
		
		public int calc();
		Code:
			Stack=2, Locals=4, Args_size=1
			0: bipush 100
			2: istore_1
			3: sipush 200
			6: istore_2
			7: sipush 300
			10: istore_3
			11: iload_1
			12: iload_2
			13: iadd
			14: iload_3
			15: imul
			16: ireturn
	执行过程如下：

	![](https://github.com/limbo-china/books/blob/master/JAVA_VM/8-3.png)  
	![](https://github.com/limbo-china/books/blob/master/JAVA_VM/8-4.png)  
	![](https://github.com/limbo-china/books/blob/master/JAVA_VM/8-5.png)  
	![](https://github.com/limbo-china/books/blob/master/JAVA_VM/8-6.png)  
	![](https://github.com/limbo-china/books/blob/master/JAVA_VM/8-7.png)  
	![](https://github.com/limbo-china/books/blob/master/JAVA_VM/8-8.png)  
	![](https://github.com/limbo-china/books/blob/master/JAVA_VM/8-9.png)

# 9. 类加载及执行子系统的案例与实战

### 9.2 案例分析

- Tomcat：正统的类加载器架构
	- （.............）
- OSGi：灵活的类加载器架构 
	-  (.............)
- 字节码生成技术与动态代理的实现

（。。。。。。。。。看完反射再来看这章）