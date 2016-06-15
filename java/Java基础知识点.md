## Java笔记连载

----------

### 最新一次更新**2015年11月26日**

#### 1.HashMap和HashTable的区别

* [x] HashMap去掉了contains方法
* [x] HashTable是同步的(线程安全)
* [x] HashMap允许空键值
* [x] HashMap执行快速失败机制
* [ ] `Fast-fail`机制:在使用迭代器的过程中有其它线程修改了集合对象结构或元素数量,都将抛出ConcurrentModifiedException

#### 2.java的线程安全类

Vector、Stack、HashTable、ConcurrentHashMap、Properties

#### 3.java集合框架

```
Collection - List - ArrayList
Collection - List - LinkedList
Collection - List - Vector
Collection - List - Vector - Stack
Collection - Set - HashSet
Collection - Set - TreeSet
Map - HashMap
Map - TreeMap
Map - HashTable
```

3.1 `ArrayList`的构造函数有三个

1. 无参构造 容量为10
2. ArrayList(Collections<?extends E> c)构造包含指定collection的元素的列表
3. ArrayList(int initialCapacity) 指定初始容量

3.2 `Iterator`支持从源集合安全地删除对象,防止并发修改异常(ConcurrentModifiedException)

#### 4.Java垃圾回收机制

4.1 调用system.gc() Runtime.getRuntime.gc()

4.2 垃圾回收:释放那些不在持有任何引用的对象的内存

4.3 怎样判断是否需要收集：
1. 引用计数法：对象没有任何引用与之关联(无法解决循环引用)
2. 对象引用遍历法：对象引用遍历从一组对象开始,沿着对象图的每条链接,递归确定可以到达的对象,如果某对象不能从这些根对象的一个(至少一个)到达,则将它作为垃圾收集。

4.4 垃圾回收方法

1. 标记清除法(Mark-Sweeping):易产生内存碎片
2. 复制回收法(Copying)：为了解决Mark-Sweep法而提出,内存空间减至一半
3. 标记压缩法(Mark-Compact):为了解决Copying法的缺陷,标记后移动到一端再清楚
4. 分代回收法(GenerationalCollection):新生代对象存活周期短,需要大量回收对象,需要复制的少,执行copying算法;老年代对象存活周期相对长,回收少量对象,执行mark-compact算法.
新生代划分：较大的eden区 和 2个survivor区

4.5 内存分配

* 新生代 |Eden Space|From Space|To Space|
* 对象主要分配在新生代的EdenSpace和FromSpace
* 如果EdenSapce和FromSpace空间不足,则发起一次GC
* 若进行GC后,EdenSpace和FromSpace能够容纳该对象,就放在Eden和FromSpace。在GC过程中会将EdenSpace和FromSpace存活的对象移动到ToSpace,然后清理Eden和From。若在清理过程中,ToSpace无法足够容纳该对象,则将该对象移入老年代中。在进行GC后,Eden和From为空,MinorGC完成。From和To标记互换。To区(逻辑上)始终为空。
* 新生代的回收成为MinorGC,对老年代的回收成为MajorGC又名FullGC

其他

- 优先在Eden上分配
- 大对象直接进入老年代
- 长期存活的对象进入老年代
- 动态对象年龄判定 suvivor区同年龄对象总和大于suvivor区空间的一半,MinorGC时复制至老年代
- 空间分配担保 新生代放不下借用老年代,虚拟机检测GC租借的老年代内存是否大于剩余的老年代内存。若大于,MinorGC变为一次FullGC。若小于,查看虚拟机是否允许担保失败,若允许则执行一次MinorGC,否则也要变为一次FullGC

#### 5.一些重要的关键字

- volatile
Java 语言提供了一种稍弱的同步机制,即`volatile`变量.用来确保将变量的更新操作通知到其他线程,保证了新值能立即同步到主内存,以及每次使用前立即从主内存刷新。 当把变量声明为volatile类型后,编译器与运行时都会注意到这个变量是共享的。`volatile`修饰变量,每次被线程访问时强迫其从主内存重读该值,修改后再写回共享内存。保证读取的可见性,对其他线程立即可见。由于不保证原子性,也就不能保证线程安全。由于及时更新，很可能导致另一线程访问最新变量值，无法跳出循环的情况。同时,volatile屏蔽了VM中必要的代码优化,效率上较低。另一个优点：禁止指令重排序

- final
`final`修饰的变量是常量，必须进行初始化，可以显示初始化，也可以通过构造进行初始化，如果不初始化编译会报错。

#### 6.多线程 & 并发 & 同步 & 锁

6.1 线程的run方法和start方法

* `start方法`
用start方法来启动线程,是真正实现了多线程。调用thread类的start方法来启动一个线程,此时线程处于就绪状态,一旦得到cpu时间片,就开始执行run方法。注：此时无需等待run方法执行完毕,即可执行下面的代码,所以run方法并没有实现多线程。
* `run方法`
只是thread类的一个普通方法,若直接调用程序中依然只有主线程这一个线程,还要顺序执行,依然要等待run方法体执行完毕才可执行下面的代码。

6.2 ReadWriteLock(读写锁)

写写互斥 读写互斥 读读并发
在读多写少的情况下可以提高效率 

6.3 resume(继续挂起的线程)和suspend(挂起线程)一起用

6.4 wait与notify、notifyall一起用

6.5 sleep与wait的异同点

1. sleep是Thread类的静态方法,wait来自object类
2. sleep不释放锁,wait释放锁
3. wait,notify,notifyall必须在同步代码块中使用,sleep可以在任何地方使用
4. 都可以抛出InterruptedException

6.6 让一个线程停止执行

异常 - 停止执行
休眠 - 停止执行
阻塞 - 停止执行

6.7 ThreadLocal相关

* ThreadLocal解决了变量并发访问的冲突问题

当使用ThreadLocal维护变量时,ThreadLocal为每个使用该变量的线程提供独立的变量副本,每个线程都可以独立地改变自己的副本,而不会影响其它线程所对应的副本,是线程隔离的。线程隔离的秘密在于ThreadLocalMap类(ThreadLocal的静态内部类)

* 与synchronized同步机制的比较

首先,它们都是为了解决多线程中相同变量访问冲突问题。不过,在同步机制中,要通过对象的锁机制保证同一时间只有一个线程访问该变量。该变量是线程共享的,使用同步机制要求程序缜密地分析什么时候对该变量读写,什么时候需要锁定某个对象,什么时候释放对象锁等复杂的问题,程序设计编写难度较大,是一种“以时间换空间”的方式。
而ThreadLocal采用了以“以空间换时间”的方式。

#### 7.接口与抽象类

1. 一个子类只能继承一个抽象类,但能实现多个接口
2. 抽象类可以有构造方法,接口没有构造方法
3. 抽象类可以有普通成员变量,接口没有普通成员变量
4. 抽象类和接口都可有静态成员变量,抽象类中静态成员变量访问类型任意，接口只能public static final(默认)
5. 抽象类可以没有抽象方法,抽象类可以有普通方法,接口中都是抽象方法
6. 抽象类可以有静态方法，接口不能有静态方法
7. 抽象类中的方法可以是public、protected和默认;接口方法只有public

#### 8.Statement接口

8.1

- Statement是最基本的用法，不传参，采用字符串拼接，存在注入漏洞
- PreparedStatement传入参数化的sql语句,同时检查合法性，效率高，可以重用,防止sql注入
- CallableStatement接口扩展PreparedStatement，用来调用存储过程
- public interface CallableStatement extends PreparedStatement 
- public interface PreparedStatement extends Statement 
- BatchedStatement用于批量操作数据库，BatchedStatement不是标准的Statement类

8.2 Statement与PrepareStatement的区别

- 创建时的区别
```
Statement statement = conn.createStatement();
PreparedStatement preStatement = conn.prepareStatement(sql);
```
- 执行的时候
```
ResultSet rSet = statement.executeQuery(sql);
ResultSet pSet = preStatement.executeQuery();
```
由上可以看出，PreparedStatement有预编译的过程，已经绑定sql，之后无论执行多少遍，都不会再去进行编译，
而 statement 不同，如果执行多遍，则相应的就要编译多少遍sql，所以从这点看，preStatement 的效率会比 Statement要高一些

- 安全性

preStatement是预编译的，所以可以有效的防止SQL注入等问题

- 代码的可读性和可维护性

PreparedStatement更胜一筹

#### 9.抽象类和最终类

抽象类可以没有抽象方法,最终类可以,没有最终方法

最终类不能被继承,最终方法不能被重写(可以重载)

#### 10.异常

10.1 throw、throws、try...catch、finally

1. throws用在方法上,方法内部通过throw抛出异常
2. try用于检测包住的语句块,若有异常,抛出并执行catch子句
3. catch捕获try块中抛出的异常并处理

10.2 关于`finally`

1. finally不管有没有异常都要处理
2. finally{}比return先执行,多个return执行一个后就不在执行
3. 不管有木有异常抛出,finally在return返回前执行

10.3 受检查异常和运行时异常
![](http://uploadfiles.nowcoder.com/images/20151010/214250_1444467985224_6A144C1382BBEF1BE30E9B91BC2973C8)

1. 粉红色的是受检查的异常(checked exceptions),其必须被try...catch语句块所捕获,或者在方法签名里通过throws子句声明。受检查的异常必须在编译时被捕捉处理,命名为Checked Exception是因为Java编译器要进行检查,Java虚拟机也要进行检查,以确保这个规则得到遵守。 

2. 绿色的异常是运行时异常(runtime exceptions),需要程序员自己分析代码决定是否捕获和处理,比如空指针,被0除... 

3. 而声明为Error的，则属于严重错误，如系统崩溃、虚拟机错误、动态链接失败等，这些错误无法恢复或者不可能捕捉，将导致应用程序中断，Error不需要捕捉。 

#### 11.this & super

11.1 super出现在父类的子类中。有三种存在方式

1. super.xxx(xxx为变量名或对象名)意思是获取父类中xxx的变量或引用
2. super.xxx(); (xxx为方法名)意思是直接访问并调用父类中的方法
3. super() 调用父类构造
- super只能指代其直接父类

11.2 this() & super()在构造方法中的区别

1. 调用super()必须写在子类构造方法的第一行,否则编译不通过
2. super从子类调用父类构造,this在同一类中调用其他构造
3. 均需要放在第一行
4. 尽管可以用this调用一个构造器,却不能调用2个
5. this和super不能出现在同一个构造器中,否则编译不通过
6. this()、super()都指的对象,不可以在static环境中使用
7. 本质this指向本对象的指针。super是一个关键字

#### 12.修饰符一览

```
修饰符 			类内部  	同一个包		子类 		任何地方
private 		yes
default         yes			yes
protected		yes			yes				yes
public			yes			yes				yes			yes
```

#### 13.构造内部类对象

```java
public class Enclosingone {
	public class Insideone {}
	public static class Insideone{}
}

public class Test {
	public static void main(String[] args) {
	Enclosingone.Insideone obj1 = new Enclosingone().new Insideone();
	Enclosingone.Insideone obj2 = new Enclosingone.Insideone();
	}
}
```
#### 14.序列化

声明为static和transient类型的数据不能被序列化,序列化的笔记参见[Java-note-序列化.md][5]

#### 15.Java的方法区

与堆一样,是线程共享的区域。方法区中存储：被虚拟机加载的类信息，常量，静态变量，编译器编译后的代码等数据。这个区域的内存回收目标主要是针对常量池的对象的回收和对类型的卸载。

#### 16.正则表达式

**次数符号**
```
* 0或多次
+ 1或多次
？0或1次
{n} 恰n次
{n,m} 从n到m次
```

**其他符号**

符号	等价形式
```
\d		[0-9]
\D      [^0-9]  
\w 		[a-zA-Z_0-9]
\W 		[^a-zA-Z_0-9]
\s 		[\t\n\r\f]
\S 		[^\t\n\r\f]
. 		任何字符
```

**边界匹配器**

行开头	^
行结尾  $
单词边界 \b

**贪婪模式**:最大长度匹配 非贪婪模式:匹配到结果就好,最短匹配

**环视**

```
字符 				描述 					匹配对象
.					单个任意字符			
[...] 				字符组 					列出的任意字符
[^...] 										未列出的任意字符
^ 					caret 					行的起始位置
$     				dollar 					行的结束位置
\<   										单词的起始位置
\> 											单词的结束位置
\b   				单词边界
\B 					非单词边界
(?=Expression)		顺序肯定环视			成功,如果右边能够匹配
(?!Expression)		顺序否定环视			成功,如果右边不能够匹配
(?<=Expression)		逆序肯定环视			成功,如果左边能够匹配
(?<!Expression) 	逆序否定环视			成功,如果左边不能够匹配
```
举例:北京市(海定区)(朝阳区)(西城区)

Regex: .*(?=\\()

**模式和匹配器的典型调用次序**

1. 把正则表达式编译到模式中
Pattern p = Pattern.compile("a*b");
2. 创建给定输入与此模式的匹配器
Matcher m = p.matcher("aaab");
3. 尝试将整个区域与此模式匹配
boolean b = m.matches();

#### 17.Servlet & JSP & Tomcat

17.1 Servlet继承实现结构
```
Servlet (接口) 			-->      init|service|destroy方法
GenericServlet(抽象类)  -->      与协议无关的Servlet
HttpServlet(抽象类)		-->		 实现了http协议
自定义Servlet			-->		 重写doGet/doPost
```

17.2 编写Servlet的步骤

1. 继承HttpServlet
2. 重写doGet/doPost方法
3. 在web.xml中注册servlet

17.3 Servlet生命周期

1. `init`:仅执行一次,负责装载servlet时初始化servlet对象
2. `service`:核心方法,一般get/post两种方式
3. `destroy`:停止并卸载servlet,释放资源

17.4 过程

1. 客户端request请求 -> 服务器检查Servlet实例是否存在 -> 若存在调用相应service方法
2. 客户端request请求 -> 服务器检查Servlet实例是否存在 -> 若不存在装载Servlet类并创建实例 -> 调用init初始化 -> 调用service
3. 加载和实例化、初始化、处理请求、服务结束

17.5 doPost方法要抛出的异常:ServletExcception、IOException

17.6 Servlet容器装载Servlet

1. web.xml中配置load-on-startup启动时装载
2. 客户首次向Servlet发送请求
3. Servlet类文件被更新后,重新装载Servlet

17.7 HttpServlet容器响应web客户请求流程

1. Web客户向servlet容器发出http请求
2. servlet容器解析Web客户的http请求
3. servlet容器创建一个HttpRequest对象,封装http请求信息
4. servlet容器创建一个HttpResponse对象
5. servlet容器调用HttpServlet的service方法,把HttpRequest和HttpResponse对象作为service方法的参数传给HttpServlet对象
6. HttpServlet调用httprequest的有关方法,获取http请求信息
7. httpservlet调用httpresponse的有关方法,生成响应数据
8. Servlet容器把HttpServlet的响应结果传给web客户

17.8 HttpServletRequest完成的功能
1. request.getCookie()
2. request.getHeader(String s)
3. request.getContextPath()

17.9 HttpServletResponse完成的功能

1. 设http头
2. 设置Cookie
3. 输出返回数据

17.10 `session`
```
HttpSession session = request.getSession(boolean create)
返回当前请求的会话
```

17.11 JSP的前身就是Servlet

17.12 Tomcat容器的等级

Tomcat - **Container** - **Engine** - **Host** - **Servlet** - 多个Context(一个Context对应一个web工程)-Wrapper

17.13 Servlet与JSP九大内置对象的关系

JSP对象 				怎样获得
```
1. out				->		response.getWriter
2. request 		->		Service方法中的req参数
3. response 		->		Service方法中的resp参数
4. session 		->		request.getSession
5. application 	->		getServletContext
6. exception 		->		Throwable
7. page  			->		this
8. pageContext  	->		PageContext
9. Config 			->		getServletConfig
```

exception是JSP九大内置对象之一，其实例代表其他页面的异常和错误。只有当页面是错误处理页面时，即isErroePage为 true时，该对象才可以使用。

#### 18.struts

1. struts可进行文件上传
2. struts基于MVC模式
3. struts让流程结构更清晰
4. struts有许多action类,会增加类文件数目

#### 19.Hibernate的7大鼓励措施

1. 尽量使用many-to-one,避免使用单项one-to-many
2. 灵活使用单项one-to-many
3. 不用一对一,使用多对一代替一对一
4. 配置对象缓存,不使用集合对象
5. 一对多使用bag,多对一使用set
6. 继承使用显示多态
7. 消除大表,使用二级缓存

#### 20.JVM

20.1 JVM内存配置参数

1. -Xmx:最大堆大小
2. -Xms:初始堆大小(最小内存值)
3. -Xmn:年轻代大小
4. -XXSurvivorRatio:3 意思是Eden:Survivor=3:2
5. -Xss栈容量
6. -XX:+PrintGC 输出GC日志
7. -XX:+PrintGCDetails 输出GC的详细日志

20.2 JVM内存结构

1. 堆:Eden、Survivor、old 线程共享
2. 方法区(非堆):持久代,代码缓存,线程共享
3. JVM栈:中间结果,局部变量,线程隔离
4. 本地栈:本地方法(非Java代码)
5. 程序计数器 ：线程私有，每个线程都有自己独立的程序计数器，用来指示下一条指令的地址
6. 注：持久代Java8消失,取代的称为元空间(本地堆内存的一部分)

#### 21.面向对象的五大基本原则(solid)

1. S单一职责`SRP`:Single-Responsibility Principle
一个类,最好只做一件事,只有一个引起它的变化。单一职责原则可以看做是低耦合,高内聚在面向对象原则的引申,将职责定义为引起变化的原因,以提高内聚性减少引起变化的原因。

2. O开放封闭原则`OCP`:Open-Closed Principle
软件实体应该是可扩展的,而不是可修改的。对扩展开放,对修改封闭

3. L里氏替换原则`LSP`:Liskov-Substitution Principle
子类必须能够替换其基类。这一思想表现为对继承机制的约束规范,只有子类能够替换其基类时,才能够保证系统在运行期内识别子类,这是保证继承复用的基础。

4. I接口隔离原则`ISP`:Interface-Segregation Principle
使用多个小的接口,而不是一个大的总接口

5. D依赖倒置原则`DIP`:Dependency-Inversion Principle
依赖于抽象。具体而言就是高层模块不依赖于底层模块,二者共同依赖于抽象。抽象不依赖于具体,具体依赖于抽象。

## 面向对象设计原则

1. 封装变化
2. 少用继承 多用组合
3. 针对接口编程 不针对实现编程
4. 为交互对象之间的松耦合设计而努力
5. 类应该对扩展开发 对修改封闭（开闭OCP原则）
6. 依赖抽象，不要依赖于具体类（依赖倒置DIP原则）
7. 密友原则：只和朋友交谈（最少知识原则）
	
	说明：将方法调用保持在界限内，只调用属于以下范围的方法：
	该对象本身（本地方法）对象的组件 被当作方法参数传进来的对象 此方法创建或实例化的任何对象

8. 别找我（调用我） 我会找你（调用你）（好莱坞原则）
9. 一个类只有一个引起它变化的原因（单一职责SRP原则）


#### 22.null可以被强制转型为任意类型的对象。

#### 23.代码执行次序

1. 多个静态成员变量,静态代码块按顺序执行
2. 单个类中: 静态代码 -> main方法 -> 构造块 -> 构造方法
3. 构造块在每一次创建对象时执行
4. 涉及父类和子类的初始化过程
	a.初始化父类中的静态成员变量和静态代码块  
	b.初始化子类中的静态成员变量和静态代码块
	c.初始化父类的普通成员变量和构造代码块(按次序)，再执行父类的构造方法(注意父类构造方法中的子类方法覆盖)
	d.初始化子类的普通成员变量和构造代码块(按次序)，再执行子类的构造方法

#### 24.红黑树

**二叉搜索树**:(Binary Search Tree又名：二叉查找树,二叉排序树)它或者是一棵空树,或者是具有下列性质的二叉树： 若它的左子树不空,则左子树上所有结点的值均小于它的根结点的值；若它的右子树不空,则右子树上所有结点的值均大于它的根结点的值；它的左、右子树也分别为二叉搜索树。

**红黑树**的定义:满足以下五个性质的二叉搜索树

1. 每个结点或是红色的或是黑色的
2. 根结点是黑色的
3. 每个叶结点是黑色的
4. 如果一个结点是红色的,则它的两个子结点是黑色的
5. 对于每个结点,从该结点到其后代叶结点的简单路径上,均包含相同数目的黑色结点

黑高

从某个结点x出发(不含x)到达一个叶结点的任意一条简单路径上的黑色结点个数称为该结点的黑高。
红黑树的黑高为其根结点的黑高。

其他

- 一个具有n个内部结点的红黑树的高度h<=2lg(n+1)
- 结点的属性(五元组):color key left right p
- 动态集合操作最坏时间复杂度为O(lgn)

#### 25.排序

- 稳定排序:插入排序、冒泡排序、归并排序、基数排序

- 插入排序[稳定]
适用于小数组,数组已排好序或接近于排好序速度将会非常快
复杂度：O(n^2) - O(n) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

- 归并排序[稳定]
采用分治法
复杂度：O(nlogn) - O(nlgn) - O(nlgn) - O(n)[平均 - 最好 - 最坏 - 空间复杂度]

- 冒泡排序[稳定]
复杂度：O(n^2) - O(n) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

- 基数排序 分配+收集[稳定]
复杂度： O(d(n+r)) r为基数d为位数 空间复杂度O(n+r)

- 树排序
应用：TreeSet的add方法、TreeMap的put方法
不支持相同元素,没有稳定性问题
复杂度：平均最差O(nlogn)

- 堆排序(就地排序)
复杂度：O(nlogn) - O(nlgn) - O(nlgn) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

- 快速排序
复杂度：O(nlgn) - O(nlgn) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]
栈空间0(lgn) - O(n)

- 选择排序
复杂度：O(n^2) - O(n^2) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

- 希尔排序
复杂度 小于O(n^2) 平均 O(nlgn) 最差O(n^s)[1<s<2] 空间O(1)

#### 26.查找与散列

26.1 散列函数设计

* 直接定址法:```f(key) = a*key+b```

简单、均匀,不易产生冲突。但需事先知道关键字的分布情况,适合查找表较小且连续的情况,故现实中并不常用

* 除留余数法:```f(key) = key mod p (p<=m) p取小于表长的最大质数 m为表长```

* DJBX33A算法(time33哈希算法```hash = hash*33+(unsigned int)str[i];```

平方取中法 折叠法 更多....

26.2 冲突处理

闭散列(开放地址方法):要求装填因子a较小，闭散列方法把所有记录直接存储在散列表中

- 线性探测:易产生堆积现象(基地址不同堆积在一起)
- 二次探测:f(key) = (f(key)+di) % m di=1^2,-1^2,2^2,-2^2...可以消除基本聚集
- 随机探测:f(key) = (f(key)+di),di采用随机函数得到,可以消除基本聚集
- 双散列:避免二次聚集

开散列(链地址法):原地处理

- 同义词记录存储在一个单链表中,散列表中子存储单链表的头指针。
- 优点:无堆积 事先无需确定表长 删除结点易于实现 装载因子a>=1,缺点:需要额外空间

#### 27.枚举类

JDK1.5出现 每个枚举值都需要调用一次构造函数

#### 28.数组复制方法

1. for逐一复制
2. System.arraycopy() -> 效率最高native方法
3. Arrays.arrayOf() -> 本质调用arraycopy
4. clone方法 -> 返回Object[],需要强制类型转换

#### 29.多态

1. Java通过方法重写和方法重载实现多态 
2. 方法重写是指子类重写了父类的同名方法 
3. 方法重载是指在同一个类中，方法的名字相同，但是参数列表不同 

#### 30.Java文件

.java文件可以包含多个类，唯一的限制就是：一个文件中只能有一个public类， 并且此public类必须与
文件名相同。而且这些类和写在多个文件中没有区别。

#### 31.Java移位运算符

java中有三种移位运算符

1. << :左移运算符,x << 1,相当于x乘以2(不溢出的情况下),低位补0
2. >> :带符号右移,x >> 1,相当于x除以2,正数高位补0,负数高位补1
3. >>> :无符号右移,忽略符号位,空位都以0补齐

#### 32.形参&实参
1. 形式参数可被视为local variable.形参和局部变量一样都不能离开方法。只有在方法中使用，不会在方法外可见。
2. 形式参数只能用final修饰符，其它任何修饰符都会引起编译器错误。但是用这个修饰符也有一定的限制，就是在方法中不能对参数做任何修改。不过一般情况下，一个方法的形参不用final修饰。只有在特殊情况下，那就是：方法内部类。一个方法内的内部类如果使用了这个方法的参数或者局部变量的话，这个参数或局部变量应该是final。
3. 形参的值在调用时根据调用者更改，实参则用自身的值更改形参的值（指针、引用皆在此列），也就是说真正被传递的是实参。

#### 33.IO
![](http://uploadfiles.nowcoder.com/images/20150328/138512_1427527478646_1.png)

#### 34.局部变量为什么要初始化

局部变量是指类方法中的变量，必须初始化。局部变量运行时被分配在栈中，量大，生命周期短，如果虚拟机给每个局部变量都初始化一下，是一笔很大的开销，但变量不初始化为默认值就使用是不安全的。出于速度和安全性两个方面的综合考虑，解决方案就是虚拟机不初始化，但要求编写者一定要在使用前给变量赋值。

#### 35.JDK提供的用于并发编程的同步器

1. Semaphore Java并发库的Semaphore可以很轻松完成信号量控制，Semaphore可以控制某个资源可被同时访问的个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。
2. CyclicBarrier 主要的方法就是一个：await()。await()方法每被调用一次，计数便会减少1，并阻塞住当前线程。当计数减至0时，阻塞解除，所有在此CyclicBarrier上面阻塞的线程开始运行。
3. 直译过来就是倒计数(CountDown)门闩(Latch)。倒计数不用说，门闩的意思顾名思义就是阻止前进。在这里就是指 CountDownLatch.await() 方法在倒计数为0之前会阻塞当前线程。

#### 36.Java类加载器

一个jvm中默认的classloader有Bootstrap ClassLoader、Extension ClassLoader、App ClassLoader，分别各司其职： 

1. Bootstrap ClassLoader(引导类加载器) 负责加载java基础类，主要是 %JRE_HOME/lib/目录下的rt.jar、resources.jar、charsets.jar等
2. Extension ClassLoader(扩展类加载器) 负责加载java扩展类，主要是 %JRE_HOME/lib/ext目录下的jar等
3. App ClassLoader(系统类加载器) 负责加载当前java应用的classpath中的所有类。 
classloader 加载类用的是全盘负责委托机制。 所谓全盘负责，即是当一个classloader加载一个Class的时候，这个Class所依赖的和引用的所有 Class也由这个classloader负责载入，除非是显式的使用另外一个classloader载入。 
所以，当我们自定义的classloader加载成功了com.company.MyClass以后，MyClass里所有依赖的class都由这个classLoader来加载完成。

#### 37.Java语言的鲁棒性

Java在编译和运行程序时，都要对可能出现的问题进行检查，以消除错误的产生。它提供自动垃圾收集来进行内存管理，防止程序员在管理内存时容易产生的错误。通过集成的面向对象的例外处理机制，在编译时，Java揭示出可能出现但未被处理的例外，帮助程序员正确地进行选择以防止系统的崩溃。另外，Java在编译时还可捕获类型声明中的许多常见错误，防止动态运行时不匹配问题的出现。

#### 38.Java语言特性

1. Java致力于检查程序在编译和运行时的错误
2. Java虚拟机实现了跨平台接口
3. 类型检查帮助检查出许多开发早期出现的错误
4. Java自己操纵内存减少了内存出错的可能性
5. Java还实现了真数组，避免了覆盖数据的可能

#### 39.Hibernate延迟加载

1. Hibernate2延迟加载实现：a)实体对象 b)集合（Collection） 
2. Hibernate3 提供了属性的延迟加载功能 
当Hibernate在查询数据的时候，数据并没有存在与内存中，当程序真正对数据的操作时，对象才存在与内存中，就实现了延迟加载，他节省了服务器的内存开销，从而提高了服务器的性能。 
3. hibernate使用Java反射机制，而不是字节码增强程序来实现透明性。 
4. hibernate的性能非常好，因为它是个轻量级框架。映射的灵活性很出色。它支持各种关系数据库，从一对一到多对多的各种复杂关系。

#### 40.包装类的equals()方法不处理数据转型，必须类型和值都一样才相等。

#### 41.子类可以继承父类的静态方法！但是不能覆盖。因为静态方法是在编译时确定了，不能多态，也就是不能运行时绑定。

#### 42.Java语法糖

1. Java7的switch用字符串 - hashcode方法 switch用于enum枚举
2. 伪泛型 - List<E>原始类型
3. 自动装箱拆箱 - Integer.valueOf和Integer.intValue
4. foreach遍历 - Iterator迭代器实现
5. 条件编译
6. enum枚举类、内部类
7. 可变参数 - 数组
8. 断言语言
9. try语句中定义和关闭资源

#### 43.JVM工具

命令行

1. jps(jvm processor status)虚拟机进程状况工具
2. jstat(jvm statistics monitoring)统计信息监视
3. jinfo(configuration info for java)配置信息工具
4. jmap(memory map for java)Java内存映射工具
5. jhat(JVM Heap Analysis Tool)虚拟机堆转储快照分析工具
6. jstack(Stack Trace for Java)Java堆栈跟踪工具
7. HSDIS：JIT生成代码反汇编

可视化
1. JConsole(Java Monitoring and Management Console):Java监视与管理控制台
2. VisualVM(All-in-one Java Troubleshooting Tool):多合一故障处理工具

----------

### 全文下载
+ [Github托管](https://github.com/Lemonjing/TinyMood/blob/master/技术笔记/Java-note-basic.md)

### 我的其他项目

+ [TinyMooc](https://github.com/Lemonjing/tinymooc)
+ [TinyCoding](https://github.com/Lemonjing/tinycoding)
+ [JavaSE工程](https://github.com/Lemonjing/myjavase)
+ [Leetcode题解](https://github.com/Lemonjing/leetcode)

### 最近更新

+ [文章列表](https://github.com/Lemonjing/TinyMood/blob/master/UPDATE_LOG.md)

### 联系我

- WebSite:[http://www.tinymood.com][1]
- Mail: 932191671@qq.com
- Linux交流群: [257265338][2]
- 邂逅厦大交流群:[103949230][3]

作者 [微博@Campanulaceae][4]       

[1]: http://www.tinymood.com   
[2]: http://jq.qq.com/?_wv=1027&k=ZKsbKb
[3]: http://jq.qq.com/?_wv=1027&k=Xxno3g
[4]: http://weibo.com/u/1662536394
[5]: https://github.com/Lemonjing/TinyMood/blob/master/技术文章/Java序列化.md
