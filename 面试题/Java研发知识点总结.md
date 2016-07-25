## Java研发工程师知识点总结

----------

### 最近一次更新**2016年07月24日**

大纲

- [x] 一、Java基础
- [x] 二、Java高级
- [x] 三、多线程和并发
- [x] 四、Java虚拟机
- [x] 五、数据库
- [x] 六、算法与数据结构
- [ ] 七、计算机网络
- [ ] 八、操作系统

###　一、Java基础（语言、面向对象等）

**1. HashMap和Hashtable的区别**

- [x] Hashtable 是 JDK 1 遗留下来的类，而 HashMap 是后来增加的。
- [x] HashMap去掉了contains方法
- [x] HashMap线程不安全, 而
Hashtable 是同步的，比较慢，但 HashMap相对会更快
- [x] HashMap允许空键值
- [x] HashMap执行快速失败机制

注：`Fast-fail`机制:在使用迭代器的过程中有其它线程修改了集合对象结构或元素数量,都将抛出ConcurrentModifiedException，但是抛出这个异常是不保证的，我们不能编写依赖于此异常的程序。

**2. java的线程安全**

Vector、Stack、HashTable、ConcurrentHashMap、Properties

**3. java集合框架(常用)**

```
Collection - List - ArrayList
Collection - List - LinkedList
Collection - List - Vector
Collection - List - Vector - Stack
Collection - Set - HashSet
Collection - Set - TreeSet
Collection - List - LinkedHashSet
Map - HashMap
Map - TreeMap
Map - HashTable
Map - LinkedHashMap
Map - ConcurrentHashMap
```

**4. ArrayList**

- 无参构造 容量为10
- ArrayList(Collections<?extends E> c)构造包含指定collection的元素的列表
- ArrayList(int initialCapacity) 指定初始容量

**5. final关键字**

`final`修饰的变量是常量，必须进行初始化，可以显示初始化，也可以通过构造进行初始化，如果不初始化编译会报错。

**6. 接口与抽象类**

6.1 一个子类只能继承一个抽象类,但能实现多个接口
6.2 抽象类可以有构造方法,接口没有构造方法
6.3 抽象类可以有普通成员变量,接口没有普通成员变量
6.4 抽象类和接口都可有静态成员变量,抽象类中静态成员变量访问类型任意，接口只能public static final(默认)
6.5 抽象类可以没有抽象方法,抽象类可以有普通方法,接口中都是抽象方法
6.6 抽象类可以有静态方法，接口不能有静态方法
6.7 抽象类中的方法可以是public、protected;接口方法只有public abstract

**7. 抽象类和最终类**

抽象类可以没有抽象方法, 最终类可以没有最终方法

最终类不能被继承, 最终方法不能被重写(可以重载)

**8.异常**

相关的关键字 throw、throws、try...catch、finally

- throws用在方法签名上, 方法内部通过throw抛出异常
- try用于检测包住的语句块, 若有异常, catch子句捕获并执行catch块

**9. 关于finally**

- finally不管有没有异常都要处理
- finally比return先执行, 多个return执行一个后就不在执行
- 不管有木有异常抛出, finally在return返回前执行

finally不执行的几种情况：程序提前终止如调用了System.exit, 病毒，断电

**10. 受检查异常和运行时异常**
![](http://uploadfiles.nowcoder.com/images/20151010/214250_1444467985224_6A144C1382BBEF1BE30E9B91BC2973C8)

10.1 粉红色的是受检查的异常(checked exceptions),其必须被try...catch语句块所捕获, 或者在方法签名里通过throws子句声明。受检查的异常必须在编译时被捕捉处理,命名为Checked Exception是因为Java编译器要进行检查, Java虚拟机也要进行检查, 以确保这个规则得到遵守。 

常见的checked exception：ClassNotFoundException IOException FileNotFoundException EOFException

10.2 绿色的异常是运行时异常(runtime exceptions), 需要程序员自己分析代码决定是否捕获和处理,比如空指针,被0除... 

常见的runtime exception：NullPointerException ArithmeticException ClassCastException IllegalArgumentException IllegalStateException IndexOutOfBoundsException NoSuchElementException 

10.3 而声明为Error的，则属于严重错误，如系统崩溃、虚拟机错误、动态链接失败等，这些错误无法恢复或者不可能捕捉，将导致应用程序中断，Error不需要捕获。 

**11. this & super**

11.1 super出现在父类的子类中。有三种存在方式

1. super.xxx(xxx为变量名或对象名)意思是获取父类中xxx的变量或引用
2. super.xxx(); (xxx为方法名)意思是直接访问并调用父类中的方法
3. super() 调用父类构造

注：super只能指代其直接父类

11.2 this() & super()在构造方法中的区别

1. 调用super()必须写在子类构造方法的第一行, 否则编译不通过
2. super从子类调用父类构造, this在同一类中调用其他构造
3. 均需要放在第一行
4. 尽管可以用this调用一个构造器, 却不能调用2个
5. this和super不能出现在同一个构造器中, 否则编译不通过
6. this()、super()都指的对象,不可以在static环境中使用
7. 本质this指向本对象的指针。super是一个关键字

**12. 修饰符一览**

```
修饰符 			类内部  	同一个包		子类 		任何地方
private 		yes
default         yes			yes
protected		yes			yes				yes
public			yes			yes				yes			yes
```

**13. 构造内部类和静态内部类对象**

```java
public class Enclosingone {
	public class Insideone {}
	public static class Insideone{}
}

public class Test {
	public static void main(String[] args) {
	// 构造内部类对象需要外部类的引用
	Enclosingone.Insideone obj1 = new Enclosingone().new Insideone();
	// 构造静态内部类的对象
	Enclosingone.Insideone obj2 = new Enclosingone.Insideone();
	}
}
```
**14. 序列化**

声明为static和transient类型的数据不能被序列化， 反序列化需要一个无参构造函数

序列化参见我的笔记[Java-note-序列化.md](https://github.com/it-interview/easy-job/blob/master/java/Java%E5%BA%8F%E5%88%97%E5%8C%96.md)

**15.正则表达式**

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
举例:北京市(海淀区)(朝阳区)(西城区)

Regex: .*(?=\\()

**模式和匹配器的典型调用次序**

1. 把正则表达式编译到模式中
Pattern p = Pattern.compile("a*b");
2. 创建给定输入与此模式的匹配器
Matcher m = p.matcher("aaab");
3. 尝试将整个区域与此模式匹配
boolean b = m.matches();

**16. 面向对象的五大基本原则(solid)**

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

**17. 面向对象设计其他原则**

1. 封装变化
2. 少用继承 多用组合
3. 针对接口编程 不针对实现编程
4. 为交互对象之间的松耦合设计而努力
5. 类应该对扩展开发 对修改封闭（开闭OCP原则）
6. 依赖抽象，不要依赖于具体类（依赖倒置DIP原则）
7. 密友原则：只和朋友交谈（最少知识原则，迪米特法则）
	
	说明：一个对象应当对其他对象有尽可能少的了解，将方法调用保持在界限内，只调用属于以下范围的方法：
	该对象本身（本地方法）对象的组件 被当作方法参数传进来的对象 此方法创建或实例化的任何对象

8. 别找我（调用我） 我会找你（调用你）（好莱坞原则）
9. 一个类只有一个引起它变化的原因（单一职责SRP原则）

**18. null可以被强制转型为任意类型的对象**

**19.代码执行次序**

1. 多个静态成员变量, 静态代码块按顺序执行
2. 单个类中: 静态代码 -> main方法 -> 构造块 -> 构造方法
3. 构造块在每一次创建对象时执行
4. 涉及父类和子类的初始化过程
	a.初始化父类中的静态成员变量和静态代码块  
	b.初始化子类中的静态成员变量和静态代码块
	c.初始化父类的普通成员变量和构造代码块(按次序)，再执行父类的构造方法(注意父类构造方法中的子类方法覆盖)
	d.初始化子类的普通成员变量和构造代码块(按次序)，再执行子类的构造方法

**20. 数组复制方法**

1. for逐一复制
2. System.arraycopy() -> 效率最高native方法
3. Arrays.copyOf() -> 本质调用arraycopy
4. clone方法 -> 返回Object[],需要强制类型转换

**21. 多态**

1. Java通过方法重写和方法重载实现多态 
2. 方法重写是指子类重写了父类的同名方法 
3. 方法重载是指在同一个类中，方法的名字相同，但是参数列表不同 

**22. Java文件**

.java文件可以包含多个类，唯一的限制就是：一个文件中只能有一个public类， 并且此public类必须与
文件名相同。而且这些类和写在多个文件中没有区别。

**23. Java移位运算符**

java中有三种移位运算符

1. << :左移运算符,x << 1,相当于x乘以2(不溢出的情况下),低位补0
2. >> :带符号右移,x >> 1,相当于x除以2,正数高位补0,负数高位补1
3. >>> :无符号右移,忽略符号位,空位都以0补齐

参见我的GitHub，[Java移位符](https://github.com/it-interview/easy-job/blob/master/java/Java%E7%A7%BB%E4%BD%8D%E7%AC%A6.md)

**24. 形参&实参**

1. 形式参数可被视为local variable.形参和局部变量一样都不能离开方法。只有在方法中使用，不会在方法外可见。
2. 形式参数只能用final修饰符，其它任何修饰符都会引起编译器错误。但是用这个修饰符也有一定的限制，就是在方法中不能对参数做任何修改。不过一般情况下，一个方法的形参不用final修饰。只有在特殊情况下，那就是：方法内部类。一个方法内的内部类如果使用了这个方法的参数或者局部变量的话，这个参数或局部变量应该是final。
3. 形参的值在调用时根据调用者更改，实参则用自身的值更改形参的值（指针、引用皆在此列），也就是说真正被传递的是实参。

**25. IO流一览**

![](http://uploadfiles.nowcoder.com/images/20150328/138512_1427527478646_1.png)

**26. 局部变量为什么要初始化**

局部变量是指类方法中的变量，必须初始化。局部变量运行时被分配在栈中，量大，生命周期短，如果虚拟机给每个局部变量都初始化一下，是一笔很大的开销，但变量不初始化为默认值就使用是不安全的。出于速度和安全性两个方面的综合考虑，解决方案就是虚拟机不初始化，但要求编写者一定要在使用前给变量赋值。

**27. Java语言的鲁棒性**

Java在编译和运行程序时，都要对可能出现的问题进行检查，以消除错误的产生。它提供自动垃圾收集来进行内存管理，防止程序员在管理内存时容易产生的错误。通过集成的面向对象的例外处理机制，在编译时，Java揭示出可能出现但未被处理的异常，帮助程序员正确地进行选择以防止系统的崩溃。另外，Java在编译时还可捕获类型声明中的许多常见错误，防止动态运行时不匹配问题的出现。

**28. Java语言特性**

1. Java致力于检查程序在编译和运行时的错误
2. Java虚拟机实现了跨平台接口
3. 类型检查帮助检查出许多开发早期出现的错误
4. Java自己操纵内存减少了内存出错的可能性
5. Java还实现了真数组，避免了覆盖数据的可能

**29. 包装类的equals()方法不处理数据转型，必须类型和值都一样才相等。**

**30. 子类可以继承父类的静态方法！但是不能覆盖。因为静态方法是在编译时确定了，不能多态，也就是不能运行时绑定。**

**31. Java语法糖**

1. Java7的switch用字符串 - hashcode方法 switch用于enum枚举
2. 伪泛型 - List<E>原始类型
3. 自动装箱拆箱 - Integer.valueOf和Integer.intValue
4. foreach遍历 - Iterator迭代器实现
5. 条件编译
6. enum枚举类、内部类
7. 可变参数 - 数组
8. 断言语言
9. try语句中定义和关闭资源

**32. Java 中应该使用什么数据类型来代表价格？**

如果不是特别关心内存和性能的话，使用BigDecimal，否则使用预定义精度的 double 类型。

**33. 怎么将 byte 转换为 String？**

可以使用 String 接收 byte[] 参数的构造器来进行转换，需要注意的点是要使用的正确的编码，否则会使用平台默认编码，这个编码可能跟原来的编码相同，也可能不同。

**34. Java 中怎样将 bytes 转换为 long 类型？**

String接收bytes的构造器转成String，再Long.parseLong

**35. 我们能将 int 强制转换为 byte 类型的变量吗？如果该值大于 byte 类型的范围，将会出现什么现象？**

是的，我们可以做强制转换，但是 Java 中 int 是 32 位的，而 byte 是 8 位的，所以，如果强制转化是，int 类型的高 24 位将会被丢弃，byte 类型的范围是从 -128 到 127。

**36. 存在两个类，B 继承 A，C 继承 B，我们能将 B 转换为 C 么？如 C = (C) B；**

可以，向下转型。但是不建议使用，容易出现类型转型异常.

**37. 哪个类包含 clone 方法？是 Cloneable 还是 Object？**

java.lang.Cloneable 是一个标示性接口，不包含任何方法，clone 方法在 object 类中定义。并且需要知道 clone() 方法是一个本地方法，这意味着它是由 c 或 c++ 或 其他本地语言实现的。

**38. Java 中 ++ 操作符是线程安全的吗？**

不是线程安全的操作。它涉及到多个指令，如读取变量值，增加，然后存储回内存，这个过程可能会出现多个线程交差。还会存在竞态条件（读取-修改-写入）。

**39. a = a + b 与 a += b 的区别**

+= 隐式的将加操作的结果类型强制转换为持有结果的类型。如果两这个整型相加，如 byte、short 或者 int，首先会将它们提升到 int 类型，然后在执行加法操作。

```java

byte a = 127;
byte b = 127;
b = a + b; // error : cannot convert from int to byte
b += a; // ok

```

（因为 a+b 操作会将 a、b 提升为 int 类型，所以将 int 类型赋值给 byte 就会编译出错）

**40. 我能在不进行强制转换的情况下将一个 double 值赋值给 long 类型的变量吗？**

不行，你不能在没有强制类型转换的前提下将一个 double 值赋值给 long 类型的变量，因为 double 类型的范围比 long 类型更广，所以必须要进行强制转换。

**41. 3*0.1 == 0.3 将会返回什么？true 还是 false？**

false，因为有些浮点数不能完全精确的表示出来。

**42. int 和 Integer 哪个会占用更多的内存？**

Integer 对象会占用更多的内存。Integer 是一个对象，需要存储对象的元数据。但是 int 是一个原始类型的数据，所以占用的空间更少。

**43. 为什么 Java 中的 String 是不可变的（Immutable）？**

Java 中的 String 不可变是因为 Java 的设计者认为字符串使用非常频繁，将字符串设置为不可变可以允许多个客户端之间共享相同的字符串。更详细的内容参见答案。

**44. 我们能在 Switch 中使用 String 吗？**

从 Java 7 开始，我们可以在 switch case 中使用字符串，但这仅仅是一个语法糖。内部实现在 switch 中使用字符串的 hash code。

**45. Java 中的构造器链是什么？**

当你从一个构造器中调用另一个构造器，就是Java 中的构造器链。这种情况只在重载了类的构造器的时候才会出现。

**46. 枚举类**

JDK1.5出现 每个枚举值都需要调用一次构造函数

**48. 什么是不可变对象（immutable object）？Java 中怎么创建一个不可变对象？**

不可变对象指对象一旦被创建，状态就不能再改变。任何修改都会创建一个新的对象，如 String、Integer及其它包装类。

如何在Java中写出Immutable的类？

要写出这样的类，需要遵循以下几个原则：

1）immutable对象的状态在创建之后就不能发生改变，任何对它的改变都应该产生一个新的对象。

2）Immutable类的所有的属性都应该是final的。

3）对象必须被正确的创建，比如：对象引用在对象创建过程中不能泄露(leak)。

4）对象应该是final的，以此来限制子类继承父类，以避免子类改变了父类的immutable特性。

5）如果类中包含mutable类对象，那么返回给客户端的时候，返回该对象的一个拷贝，而不是该对象本身（该条可以归为第一条中的一个特例）

**49. 我们能创建一个包含可变对象的不可变对象吗？**

是的，我们是可以创建一个包含可变对象的不可变对象的，你只需要谨慎一点，不要共享可变对象的引用就可以了，如果需要变化时，就返回原对象的一个拷贝。最常见的例子就是对象中包含一个日期对象的引用。

**50. List和Set**

List 是一个有序集合，允许元素重复。它的某些实现可以提供基于下标值的常量访问时间，但是这不是 List 接口保证的。Set 是一个无序集合。

**51. poll() 方法和 remove() 方法的区别？**

poll() 和 remove() 都是从队列中取出一个元素，但是 poll() 在获取元素失败的时候会返回空，但是 remove() 失败的时候会抛出异常。

**52. Java 中 LinkedHashMap 和 PriorityQueue 的区别是什么？**

PriorityQueue 保证最高或者最低优先级的的元素总是在队列头部，但是 LinkedHashMap 维持的顺序是元素插入的顺序。当遍历一个 PriorityQueue 时，没有任何顺序保证，但是 LinkedHashMap 课保证遍历顺序是元素插入的顺序。

**53. ArrayList 与 LinkedList 的区别？**

最明显的区别是 ArrrayList 底层的数据结构是数组，支持随机访问，而 LinkedList 的底层数据结构书链表，不支持随机访问。使用下标访问一个元素，ArrayList 的时间复杂度是 O(1)，而 LinkedList 是 O(n)。

**54. 用哪两种方式来实现集合的排序？**

你可以使用有序集合，如 TreeSet 或 TreeMap，你也可以使用有顺序的的集合，如 list，然后通过 Collections.sort() 来排序。

**55. Java 中怎么打印数组？**

你可以使用 Arrays.toString() 和 Arrays.deepToString() 方法来打印数组。由于数组没有实现 toString() 方法，所以如果将数组传递给 System.out.println() 方法，将无法打印出数组的内容，但是 Arrays.toString() 可以打印每个元素。

**56. Java 中的 LinkedList 是单向链表还是双向链表？**

是双向链表，你可以检查 JDK 的源码。在 Eclipse，你可以使用快捷键 Ctrl + T，直接在编辑器中打开该类。

**57. Java 中的 TreeMap 是采用什么树实现的？**

Java 中的 TreeMap 是使用红黑树实现的。

**58. Java 中的 HashSet，内部是如何工作的？**

HashSet 的内部采用 HashMap来实现。由于 Map 需要 key 和 value，所以所有 key 的都有一个默认 value。类似于 HashMap，HashSet 不允许重复的 key，只允许有一个null key，意思就是 HashSet 中只允许存储一个 null 对象。

**59. 写一段代码在遍历 ArrayList 时移除一个元素？**

该问题的关键在于面试者使用的是 ArrayList 的 remove() 还是 Iterator 的 remove()方法。这有一段示例代码，是使用正确的方式来实现在遍历的过程中移除元素，而不会出现 ConcurrentModificationException 异常的示例代码。

**60. 我们能自己写一个容器类，然后使用 for-each 循环吗？**

可以，你可以写一个自己的容器类。如果你想使用 Java 中增强的循环来遍历，你只需要实现 Iterable 接口。如果你实现 Collection 接口，默认就具有该属性。

**61. ArrayList 和 HashMap 的默认大小是多数？**

在 Java 7 中，ArrayList 的默认大小是 10 个元素，HashMap 的默认大小是16个元素（必须是2的幂）。这就是 Java 7 中 ArrayList 和 HashMap 类的代码片段：

```java
// from ArrayList.java JDK 1.7
private static final int DEFAULT_CAPACITY = 10;
 
//from HashMap.java JDK 7
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

**62. 有没有可能两个不相等的对象有有相同的 hashcode？**

有可能，两个不相等的对象可能会有相同的 hashcode 值，这就是为什么在 hashmap 中会有冲突。相等 hashcode 值的规定只是说如果两个对象相等，必须有相同的hashcode 值，但是没有关于不相等对象的任何规定。

**63. 两个相同的对象会有不同的的 hash code 吗？**

不能，根据 hash code 的规定，这是不可能的。

**64. 我们可以在 hashcode() 中使用随机数字吗？**

不行，因为对象的 hashcode 值必须是相同的。

**65. Java 中，Comparator 与 Comparable 有什么不同？**

Comparable 接口用于定义对象的自然顺序，而 comparator 通常用于定义用户定制的顺序。Comparable 总是只有一个，但是可以有多个 comparator 来定义对象的顺序。

**66. 为什么在重写 equals 方法的时候需要重写 hashCode 方法？**

因为有强制的规范指定需要同时重写 hashcode 与 equal 是方法，许多容器类，如 HashMap、HashSet 都依赖于 hashcode 与 equals 的规定。

**67. “a==b”和”a.equals(b)”有什么区别？**

如果 a 和 b 都是对象，则 a==b 是比较两个对象的引用，只有当 a 和 b 指向的是堆中的同一个对象才会返回 true，而 a.equals(b) 是进行逻辑比较，所以通常需要重写该方法来提供逻辑一致性的比较。例如，String 类重写 equals() 方法，所以可以用于两个不同对象，但是包含的字母相同的比较。

**68. a.hashCode() 有什么用？与 a.equals(b) 有什么关系？**

简介：hashCode() 方法是相应对象整型的 hash 值。它常用于基于 hash 的集合类，如 Hashtable、HashMap、LinkedHashMap等等。它与 equals() 方法关系特别紧密。根据 Java 规范，两个使用 equal() 方法来判断相等的对象，必须具有相同的 hash code。

1、hashcode的作用

List和Set，如何保证Set不重复呢？通过迭代使用equals方法来判断，数据量小还可以接受，数据量大怎么解决？引入hashcode，实际上hashcode扮演的角色就是寻址，大大减少查询匹配次数。

2、hashcode重要吗

对于数组、List集合就是一个累赘。而对于hashmap, hashset, hashtable就异常重要了。

3、equals方法遵循的原则

- 对称性 若x.equals(y)true，则y.equals(x)true
- 自反性 x.equals(x)必须true
- 传递性 若x.equals(y)true,y.equals(z)true,则x.equals(z)必为true
- 一致性 只要x,y内容不变，无论调用多少次结果不变
- 其他 x.equals(null) 永远false，x.equals(和x数据类型不同)始终false

两者的关系

![](http://mmbiz.qpic.cn/mmbiz/eZzl4LXykQzNwBLMbG9xO2dKOTibLqDS3X5Udic6TtRBNWs66SicbAHticSe3HBRtWKZb8lXnf1b5IHnxKznoaQ2GQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

**69. final、finalize 和 finally 的不同之处？**

final 是一个修饰符，可以修饰变量、方法和类。如果 final 修饰变量，意味着该变量的值在初始化后不能被改变。Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的，但是什么时候调用 finalize 没有保证。finally 是一个关键字，与 try 和 catch 一起用于异常的处理。finally 块一定会被执行，无论在 try 块中是否有发生异常。

**70. Java 中的编译期常量是什么？使用它又什么风险？**

变量也就是我们所说的编译期常量，这里的 public 可选的。实际上这些变量在编译时会被替换掉，因为编译器知道这些变量的值，并且知道这些变量在运行时不能改变。这种方式存在的一个问题是你使用了一个内部的或第三方库中的公有编译时常量，但是这个值后面被其他人改变了，但是你的客户端仍然在使用老的值，甚至你已经部署了一个新的jar。为了避免这种情况，当你在更新依赖 JAR 文件时，确保重新编译你的程序。

**71. 说出几点 Java 中使用 Collections 的最佳实践**

这是我在使用 Java 中 Collectionc 类的一些最佳实践：

a）使用正确的集合类，例如，如果不需要同步列表，使用 ArrayList 而不是 Vector。

b）优先使用并发集合，而不是对集合进行同步。并发集合提供更好的可扩展性。

c）使用接口代表和访问集合，如使用List存储 ArrayList，使用 Map 存储 HashMap 等等。

d）使用迭代器来循环集合。

e）使用集合的时候使用泛型。

------

### 二、Java高级（JavaEE、框架等）

**1. Servlet**

1.1 Servlet继承实现结构

```sh
Servlet (接口) 			-->      init|service|destroy方法
GenericServlet(抽象类)  -->      与协议无关的Servlet
HttpServlet(抽象类)		-->		 实现了http协议
自定义Servlet			-->		 重写doGet/doPost
```

1.2 编写Servlet的步骤

1. 继承HttpServlet
2. 重写doGet/doPost方法
3. 在web.xml中注册servlet

1.3 Servlet生命周期

1. `init`:仅执行一次,负责装载servlet时初始化servlet对象
2. `service`:核心方法,一般get/post两种方式
3. `destroy`:停止并卸载servlet,释放资源

1.4 过程

1. 客户端request请求 -> 服务器检查Servlet实例是否存在 -> 若存在调用相应service方法
2. 客户端request请求 -> 服务器检查Servlet实例是否存在 -> 若不存在装载Servlet类并创建实例 -> 调用init初始化 -> 调用service
3. 加载和实例化、初始化、处理请求、服务结束

1.5 doPost方法要抛出的异常:ServletExcception、IOException

1.6 Servlet容器装载Servlet

1. web.xml中配置load-on-startup启动时装载
2. 客户首次向Servlet发送请求
3. Servlet类文件被更新后, 重新装载Servlet

1.7 HttpServlet容器响应web客户请求流程

1. Web客户向servlet容器发出http请求
2. servlet容器解析Web客户的http请求
3. servlet容器创建一个HttpRequest对象, 封装http请求信息
4. servlet容器创建一个HttpResponse对象
5. servlet容器调用HttpServlet的service方法, 把HttpRequest和HttpResponse对象作为service方法的参数传给HttpServlet对象
6. HttpServlet调用httprequest的有关方法, 获取http请求信息
7. httpservlet调用httpresponse的有关方法, 生成响应数据
8. Servlet容器把HttpServlet的响应结果传给web客户

1.8 HttpServletRequest完成的一些功能

1. request.getCookie()
2. request.getHeader(String s)
3. request.getContextPath()
4. request.getSession()

```
HttpSession session = request.getSession(boolean create)
返回当前请求的会话
```

1.9 HttpServletResponse完成一些的功能

1. 设http响应头
2. 设置Cookie
3. 输出返回数据

1.10 Servlet与JSP九大内置对象的关系

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

**2. JSP**

JSP的前身就是Servlet

**3. Tomcat**

3.1 Tomcat容器的等级

Tomcat - **Container** - **Engine** - **Host** - **Servlet** - 多个Context(一个Context对应一个web工程)-Wrapper

**4. struts**

1. struts可进行文件上传
2. struts基于MVC模式
3. struts让流程结构更清晰
4. struts有许多action类, 会增加类文件数目

**5. Hibernate的7大鼓励措施**

1. 尽量使用many-to-one, 避免使用单项one-to-many
2. 灵活使用单项one-to-many
3. 不用一对一, 使用多对一代替一对一
4. 配置对象缓存, 不使用集合对象
5. 一对多使用bag, 多对一使用set
6. 继承使用显示多态
7. 消除大表, 使用二级缓存

**6. Hibernate延迟加载**

1. Hibernate2延迟加载实现：a)实体对象 b)集合（Collection） 
2. Hibernate3 提供了属性的延迟加载功能 
当Hibernate在查询数据的时候，数据并没有存在与内存中，当程序真正对数据的操作时，对象才存在与内存中，就实现了延迟加载，他节省了服务器的内存开销，从而提高了服务器的性能。 
3. hibernate使用Java反射机制，而不是字节码增强程序来实现透明性。 
4. hibernate的性能非常好，因为它是个轻量级框架。映射的灵活性很出色。它支持各种关系数据库，从一对一到多对多的各种复杂关系。

------

### 三、多线程和并发

**1. volatile**

Java 语言提供了一种稍弱的同步机制,即`volatile`变量.用来确保将变量的更新操作通知到其他线程,保证了新值能立即同步到主内存,以及每次使用前立即从主内存刷新。 当把变量声明为volatile类型后,编译器与运行时都会注意到这个变量是共享的。`volatile`修饰变量,每次被线程访问时强迫其从主内存重读该值,修改后再写回。保证读取的可见性,对其他线程立即可见。`volatile`的另一个语义是禁止指令重排序优化。但是`volatile`并不保证原子性,也就不能保证线程安全。

**2. Java 中能创建 volatile 数组吗？**

能，Java 中可以创建 volatile 类型数组，不过只是一个指向数组的引用，而不是整个数组。我的意思是，如果改变引用指向的数组，将会受到 volatile 的保护，但是如果多个线程同时改变数组的元素，volatile 就不能起到之前的保护作用了。

**3. volatile 能使得一个非原子操作变成原子操作吗？**

一个典型的例子是在类中有一个 long 类型的成员变量。如果你知道该成员变量会被多个线程访问，如计数器、价格等，你最好是将其设置为 volatile。为什么？因为 Java 中读取 long 类型变量不是原子的，需要分成两步，如果一个线程正在修改该 long 变量的值，另一个线程可能只能看到该值的一半（前 32 位）。但是对一个 volatile 型的 long 或 double 变量的读写是原子。

**4. volatile 禁止指令重排序的底层原理**
指令重排序，是指CPU允许多条指令不按程序规定的顺序分开发送给相应电路单元处理。但并不是说任意重排，CPU需要能正确处理指令依赖情况以正确的执行结果。`volatile`禁止指令重排序是通过内存屏障实现的，指令重排序不能把后面的指令重排序到内存屏障之前。由内存屏障保证一致性。注：该条语义在JDK1.5才得以修复，这点也是JDK1.5之前无法通过双重检查加锁来实现单例模式的原因。

**5. volatile 类型变量提供什么保证？**

volatile 变量提供有序性和可见性保证，例如，JVM 或者 JIT为了获得更好的性能会对语句重排序，但是 volatile 类型变量即使在没有同步块的情况下赋值也不会与其他语句重排序。 volatile 提供 happens-before 的保证，确保一个线程的修改能对其他线程是可见的。某些情况下，volatile 还能提供原子性，如读 64 位数据类型，像 long 和 double 都不是原子的，但 volatile 类型的 double 和 long 就是原子的。

**6. volatile的性能**

volatile变量的读操作性能消耗和普通变量差不多，但是写操作可能相对慢一些，因为它需要在本地代码中插入许多内存屏障指令以确保处理器不发生乱序执行。大多数情况下，volatile总开销比锁低，但我们要注意volatile的语义能否满足使用场景。

**7. 10 个线程和 2 个线程的同步代码，哪个更容易写？**

从写代码的角度来说，两者的复杂度是相同的，因为同步代码与线程数量是相互独立的。但是同步策略的选择依赖于线程的数量，因为越多的线程意味着更大的竞争，所以你需要利用同步技术，如锁分离，这要求更复杂的代码和专业知识。

**8. 你是如何调用 wait（）方法的？使用 if 块还是循环？为什么？**

wait() 方法应该在循环调用，因为当线程获取到 CPU 开始执行的时候，其他条件可能还没有满足，所以在处理前，循环检测条件是否满足会更好。下面是一段标准的使用 wait 和 notify 方法的代码：

```java

// The standard idiom for using the wait method
synchronized (obj) {
while (condition does not hold)
obj.wait(); // (Releases lock, and reacquires on wakeup)
... // Perform action appropriate to condition
}

```

参见 Effective Java 第 69 条，获取更多关于为什么应该在循环中来调用 wait 方法的内容。

**9. 什么是多线程环境下的伪共享（false sharing）？**

伪共享是多线程系统（每个处理器有自己的局部缓存）中一个众所周知的性能问题。伪共享发生在不同处理器的上的线程对变量的修改依赖于相同的缓存行，如下图所示：

![False Sharing in Multi-threaded application](http://jbcdn2.b0.upaiyun.com/2015/11/2bccd7f52a70db95aa72524ef3a55164.gif)

伪共享问题很难被发现，因为线程可能访问完全不同的全局变量，内存中却碰巧在很相近的位置上。如其他诸多的并发问题，避免伪共享的最基本方式是仔细审查代码，根据缓存行来调整你的数据结构。

**10. 线程的run方法和start方法**

- `run方法`

只是thread类的一个普通方法,若直接调用程序中依然只有主线程这一个线程,还要顺序执行,依然要等待run方法体执行完毕才可执行下面的代码。

- `start方法`

用start方法来启动线程,是真正实现了多线程。调用thread类的start方法来启动一个线程,此时线程处于就绪状态,一旦得到cpu时间片,就开始执行run方法。

**11. ReadWriteLock(读写锁)**

写写互斥 读写互斥 读读并发, 在读多写少的情况下可以提高效率 

**12. resume(继续挂起的线程)和suspend(挂起线程)一起用**

**13. wait与notify、notifyall一起用**

**14. sleep与wait的异同点**

- sleep是Thread类的静态方法, wait来自object类
- sleep方法短暂停顿不释放锁, wait方法条件等待要释放锁，因为只有这样，其他等待的线程才能在满足条件时获取到该锁。
- wait, notify, notifyall必须在同步代码块中使用, sleep可以在任何地方使用
- 都可以抛出InterruptedException

**15. 让一个线程停止执行**

异常 - 停止执行
休眠 - 停止执行
阻塞 - 停止执行

**16. ThreadLocal简介**

16.1 ThreadLocal解决了变量并发访问的冲突问题

当使用ThreadLocal维护变量时,ThreadLocal为每个使用该变量的线程提供独立的变量副本,每个线程都可以独立地改变自己的副本,而不会影响其它线程所对应的副本,是线程隔离的。线程隔离的秘密在于ThreadLocalMap类(ThreadLocal的静态内部类)

16.2 与synchronized同步机制的比较

首先,它们都是为了解决多线程中相同变量访问冲突问题。不过,在同步机制中,要通过对象的锁机制保证同一时间只有一个线程访问该变量。该变量是线程共享的, 使用同步机制要求程序缜密地分析什么时候对该变量读写, 什么时候需要锁定某个对象, 什么时候释放对象锁等复杂的问题,程序设计编写难度较大, 是一种“以时间换空间”的方式。
而ThreadLocal采用了以“以空间换时间”的方式。

**17. 线程局部变量原理**

当使用ThreadLocal维护变量时,ThreadLocal为每个使用该变量的线程提供独立的变量副本,每个线程都可以独立地改变自己的副本,而不会影响其它线程所对应的副本,是线程隔离的。线程隔离的秘密在于ThreadLocalMap类(ThreadLocal的静态内部类)

线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程间共享。Java 提供 ThreadLocal 类来支持线程局部变量，是一种实现线程安全的方式。但是在管理环境下（如 web 服务器）使用线程局部变量的时候要特别小心，在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。任何线程局部变量一旦在工作完成后没有释放，Java 应用就存在内存泄露的风险。

ThreadLocal的方法：void set(T value)、T get()以及T initialValue()。

ThreadLocal是如何为每个线程创建变量的副本的：

首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

总结：

1. 实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的

2. 为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象，因为每个线程中可有多个threadLocal变量，就像上面代码中的longLocal和stringLocal；

3. 在进行get之前，必须先set，否则会报空指针异常；如果想在get之前不需要调用set就能正常访问的话，必须重写initialValue()方法

**17. JDK提供的用于并发编程的同步器**

1. `Semaphore` Java并发库的Semaphore可以很轻松完成信号量控制，Semaphore可以控制某个资源可被同时访问的个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。
2. `CyclicBarrier` 主要的方法就是一个：await()。await()方法每被调用一次，计数便会减少1，并阻塞住当前线程。当计数减至0时，阻塞解除，所有在此CyclicBarrier上面阻塞的线程开始运行。
3. `CountDownLatch` 直译过来就是倒计数(CountDown)门闩(Latch)。倒计数不用说，门闩的意思顾名思义就是阻止前进。在这里就是指 CountDownLatch.await() 方法在倒计数为0之前会阻塞当前线程。

**18. 什么是 Busy spin？我们为什么要使用它？**

Busy spin 是一种在不释放 CPU 的基础上等待事件的技术。它经常用于避免丢失 CPU 缓存中的数据（如果线程先暂停，之后在其他CPU上运行就会丢失）。所以，如果你的工作要求低延迟，并且你的线程目前没有任何顺序，这样你就可以通过循环检测队列中的新消息来代替调用 sleep() 或 wait() 方法。它唯一的好处就是你只需等待很短的时间，如几微秒或几纳秒。LMAX 分布式框架是一个高性能线程间通信的库，该库有一个 BusySpinWaitStrategy 类就是基于这个概念实现的，使用 busy spin 循环 EventProcessors 等待屏障。

**19. Java 中怎么获取一份线程 dump 文件？**

在 Linux 下，你可以通过命令 kill -3 PID （Java 进程的进程 ID）来获取 Java 应用的 dump 文件。在 Windows 下，你可以按下 Ctrl + Break 来获取。这样 JVM 就会将线程的 dump 文件打印到标准输出或错误文件中，它可能打印在控制台或者日志文件中，具体位置依赖应用的配置。

**20. Swing 是线程安全的？**

不是，Swing 不是线程安全的。你不能通过任何线程来更新 Swing 组件，如 JTable、JList 或 JPanel，事实上，它们只能通过 GUI 或 AWT 线程来更新。这就是为什么 Swing 提供 invokeAndWait() 和 invokeLater() 方法来获取其他线程的 GUI 更新请求。这些方法将更新请求放入 AWT 的线程队列中，可以一直等待，也可以通过异步更新直接返回结果。


**21. 用 wait-notify 写一段代码来解决生产者-消费者问题？**

记住在同步块中调用 wait() 和 notify()方法，如果阻塞，通过循环来测试等待条件。

**22. 用 Java 写一个线程安全的单例模式（Singleton）？**

当我们说线程安全时，意思是即使初始化是在多线程环境中，仍然能保证单个实例。Java 中，使用枚举作为单例类是最简单的方式来创建线程安全单例模式的方式。参见我整理的单例的文章[6种单例模式的实现以及double check的剖析](http://tinymood.com/2016/02/06/singleton-comparasion.html)

**23. Java 中，编写多线程程序的时候你会遵循哪些最佳实践？**

这是我在写Java 并发程序的时候遵循的一些最佳实践：

a）给线程命名，这样可以帮助调试。

b）最小化同步的范围，而不是将整个方法同步，只对关键部分做同步。

c）如果可以，更偏向于使用 volatile 而不是 synchronized。

d）使用更高层次的并发工具，而不是使用 wait() 和 notify() 来实现线程间通信，如 BlockingQueue，CountDownLatch 及 Semeaphore。

e）优先使用并发集合，而不是对集合进行同步。并发集合提供更好的可扩展性。

**24. 说出至少 5 点在 Java 中使用线程的最佳实践。**

这个问题与之前的问题类似，你可以使用上面的答案。对线程来说，你应该：

a）对线程命名

b）将线程和任务分离，使用线程池执行器来执行 Runnable 或 Callable。

c）使用线程池

------

### 四、Java虚拟机

**1. 主动GC** 

调用system.gc() Runtime.getRuntime.gc()

**2. 垃圾回收**

释放那些不在持有任何引用的对象的内存

**3. 怎样判断是否需要收集**

- 引用计数法：对象没有任何引用与之关联(无法解决循环引用)

ext：Python使用引用计数法

- 可达性分析法：通过一组称为GC Root的对象为起点,从这些节点向下搜索，如果某对象不能从这些根对象的一个(至少一个)所到达,则判定该对象应当回收。

ext：可作为GCRoot的对象：虚拟机栈中引用的对象。方法区中类静态属性引用的对象，方法区中类常量引用的对象，本地方法栈中JNI引用的对象

**4. 垃圾回收方法**

- 标记清除法(Mark-Sweeping):易产生内存碎片
- 复制回收法(Copying)：为了解决Mark-Sweep法而提出,内存空间减至一半
- 标记压缩法(Mark-Compact):为了解决Copying法的缺陷,标记后移动到一端再清除
- 分代回收法(GenerationalCollection):新生代对象存活周期短,需要大量回收对象,需要复制的少,执行copy算法;老年代对象存活周期相对长,回收少量对象,执行mark-compact算法.新生代划分：较大的eden区 和 2个survivor区

**5. 内存分配**

- 新生代 Survivor区的三部分 |Eden Space|From Space|To Space|，对象主要分配在新生代的Eden区

- 大对象直接进入老年代
大对象比如大数组直接进入老年代，可通过虚拟机参数-XX：PretenureSizeThreshold参数设置

- 长期存活的对象进入老年代
ext：虚拟机为每个对象定义一个年龄计数器，如果对象在Eden区出生并经过一次MinorGC仍然存活，将其移入Survivor的To区，GC完成标记互换后，相当于存活的对象进入From区，对象年龄加1，当增加到默认15岁时，晋升老年代。可通过-XX：MaxTenuringThreshold设置

- GC的过程：GC开始前，对象只存在于Eden区和From区，To区逻辑上始终为空。对象分配在Eden区，Eden区空间不足，发起MinorGC，将Eden区所有存活的对象复制到To区，From区存活的对象根据年龄判断去向，若到达年龄阈值移入老年代，否则也移入To区，GC完成后Eden区和From区被清空，From区和To区标记互换。对象每在Survivor区躲过一次MinorGC年龄加一。MinorGC将重复这样的过程，直到To区被填满，To区满了以后，将把所有对象移入老年代。

- 动态对象年龄判定 suvivor区相同年龄对象总和大于suvivor区空间的一半,年龄大于等于该值的对象直接进入老年代

- 空间分配担保 在MinorGC开始前，虚拟机检查老年代最大可用连续空间是否大于新生代所有对象总空间，如果成立，MinorGC可以确保是安全的。否则，虚拟机会查看HandlePromotionFailure设置值是否允许担保失败，如果允许，继续查看老年代最大可用连续空间是否大于历次晋升到老年代对象的平均大小，如果大于则尝试MinorGC，尽管这次MinorGC是有风险的。如果小于，或者HandlerPromotionFailure设置不允许，则要改为FullGC。

- 新生代的回收称为MinorGC,对老年代的回收成为MajorGC又名FullGC

**6. 关于GC的虚拟机参数**

GC相关

-XX:NewSize和-XX:MaxNewSize 新生代大小
-XX:SurvivorRatio Eden和其中一个survivor的比值
-XX：PretenureSizeThreshold 大对象进入老年代的阈值
-XX:MaxTenuringThreshold 晋升老年代的对象年龄

收集器设置
-XX:+UseSerialGC:设置串行收集器
-XX:+UseParallelGC:设置并行收集器
-XX:+UseParalledlOldGC:设置并行年老代收集器
-XX:+UseConcMarkSweepGC:设置并发收集器

堆大小设置

-Xmx:最大堆大小
-Xms:初始堆大小(最小内存值)
-Xmn:年轻代大小
-XXSurvivorRatio:3 意思是Eden:Survivor=3:2
-Xss栈容量

垃圾回收统计信息

-XX:+PrintGC 输出GC日志
-XX:+PrintGCDetails 输出GC的详细日志

**7. 方法区的回收**
回收内容：废弃常量和无用的类

无用的类的判定

1. 该类的所有实例都已被回收
2. 加载该类的ClassLoader已被回收
3. 该类对应的Class对象没有任何地方引用，无法通过反射访问该类的方法

**8. JVM工具**

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

**9. JVM内存结构**

1. 堆:新生代和年老代
2. 方法区(非堆):持久代, 代码缓存, 线程共享
3. JVM栈:中间结果,局部变量,线程隔离
4. 本地栈:本地方法(非Java代码)
5. 程序计数器 ：线程私有，每个线程都有自己独立的程序计数器，用来指示下一条指令的地址
6. 注：持久代Java8消失, 取代的称为元空间(本地堆内存的一部分)

**10. JVM的方法区**

与堆一样，是线程共享的区域。方法区中存储：被虚拟机加载的类信息，常量，静态变量，JIT编译后的代码等数据。参见我是一个Java Class。

**11. Java类加载器**

一个jvm中默认的classloader有Bootstrap ClassLoader、Extension ClassLoader、App ClassLoader，分别各司其职： 

1. Bootstrap ClassLoader(引导类加载器) 负责加载java基础类，主要是 %JRE_HOME/lib/目录下的rt.jar、resources.jar、charsets.jar等
2. Extension ClassLoader(扩展类加载器) 负责加载java扩展类，主要是 %JRE_HOME/lib/ext目录下的jar等
3. App ClassLoader(系统类加载器) 负责加载当前java应用的classpath中的所有类。 
classloader 加载类用的是全盘负责委托机制。 所谓全盘负责，即是当一个classloader加载一个Class的时候，这个Class所依赖的和引用的所有 Class也由这个classloader负责载入，除非是显式的使用另外一个classloader载入。 
所以，当我们自定义的classloader加载成功了com.company.MyClass以后，MyClass里所有依赖的class都由这个classLoader来加载完成。

**12. 64 位 JVM 中，int 的长度是多大？**

Java 中，int 类型变量的长度是一个固定值，与平台无关，都是 32 位。意思就是说，在 32 位 和 64 位 的Java 虚拟机中，int 类型的长度是相同的。

**13. Serial 与 Parallel GC之间的不同之处？**

Serial 与 Parallel 在GC执行的时候都会引起 stop-the-world。它们之间主要不同 serial 收集器是默认的复制收集器，执行 GC 的时候只有一个线程，而 parallel 收集器使用多个 GC 线程来执行。

**14.Java 中 WeakReference 与 SoftReference的区别？**

Java中一共有四种类型的引用。StrongReference、 SoftReference、 WeakReference 以及 PhantomReference。

StrongReference 是 Java 的默认引用实现, 它会尽可能长时间的存活于 JVM 内，当没有任何对象指向它时将会被GC回收

WeakReference，顾名思义, 是一个弱引用, 当所引用的对象在 JVM 内不再有强引用时, 将被GC回收

虽然 WeakReference 与 SoftReference 都有利于提高 GC 和 内存的效率，但是 WeakReference ，一旦失去最后一个强引用，就会被 GC 回收，而 SoftReference 会尽可能长的保留引用直到 JVM 内存不足时才会被回收(虚拟机保证), 这一特性使得 SoftReference 非常适合缓存应用

**15. WeakHashMap 是怎么工作的？**

WeakHashMap 的工作与正常的 HashMap 类似，但是使用弱引用作为 key，意思就是当 key 对象没有任何引用时，key/value 将会被回收。

**16. JVM 选项 -XX:+UseCompressedOops 有什么作用？为什么要使用？**

当你将你的应用从 32 位的 JVM 迁移到 64 位的 JVM 时，由于对象的指针从 32 位增加到了 64 位，因此堆内存会突然增加，差不多要翻倍。这也会对 CPU 缓存（容量比内存小很多）的数据产生不利的影响。因为，迁移到 64 位的 JVM 主要动机在于可以指定最大堆大小，通过压缩 OOP 可以节省一定的内存。通过 -XX:+UseCompressedOops 选项，JVM 会使用 32 位的 OOP，而不是 64 位的 OOP。

**17. 怎样通过 Java 程序来判断 JVM 是 32 位 还是 64 位？**

你可以检查某些系统属性如 sun.arch.data.model 或 os.arch 来获取该信息。

**18. 32 位 JVM 和 64 位 JVM 的最大堆内存分别是多数？**

理论上说上 32 位的 JVM 堆内存可以到达 2^32，即 4GB，但实际上会比这个小很多。不同操作系统之间不同，如 Windows 系统大约 1.5 GB，Solaris 大约 3GB。64 位 JVM允许指定最大的堆内存，理论上可以达到 2^64，这是一个非常大的数字，实际上你可以指定堆内存大小到 100GB。甚至有的 JVM，如 Azul，堆内存到 1000G 都是可能的。

** 19. JRE、JDK、JVM 及 JIT 之间有什么不同？**

JRE 代表 Java 运行时（Java run-time），是运行 Java 应用所必须的。JDK 代表 Java 开发工具（Java development kit），是 Java 程序的开发工具，如 Java 编译器，它也包含 JRE。JVM 代表 Java 虚拟机（Java virtual machine），它的责任是运行 Java 应用。JIT 代表即时编译（Just In Time compilation），当代码执行的次数超过一定的阈值时，会将 Java 字节码转换为本地代码，如，主要的热点代码会被准换为本地代码，这样有利大幅度提高 Java 应用的性能。

![JVM JRE JDK](http://jbcdn2.b0.upaiyun.com/2015/11/4468a2440b48658c08acc50f15c3985b.jpg)

**20. 解释 Java 堆空间及 GC？**

当通过 Java 命令启动 Java 进程的时候，会为它分配内存。内存的一部分用于创建堆空间，当程序中创建对象的时候，就从对空间中分配内存。GC 是 JVM 内部的一个后台进程，回收无效对象的内存用于将来的分配。

**21. 你能保证 GC 执行吗？**

不能，虽然你可以调用 System.gc() 或者 Runtime.getRuntime().gc()，但是没有办法保证 GC 的执行。

**22. 怎么获取 Java 程序使用的内存？堆使用的百分比？**

可以通过 java.lang.Runtime 类中与内存相关方法来获取剩余的内存，总内存及最大堆内存。通过这些方法你也可以获取到堆使用的百分比及堆内存的剩余空间。Runtime.freeMemory() 方法返回剩余空间的字节数，Runtime.totalMemory() 方法总内存的字节数，Runtime.maxMemory() 返回最大内存的字节数。

**23. Java 中堆和栈有什么区别？**

JVM 中堆和栈属于不同的内存区域，使用目的也不同。栈常用于保存方法帧和局部变量，而对象总是在堆上分配。栈通常都比堆小，也不会在多个线程之间共享，而堆被整个 JVM 的所有线程共享。

------

### 五、数据库（Sql、MySQL、Redis等）

**1. Statement**

1.1 基本内容

- Statement是最基本的用法, 不传参, 采用字符串拼接，存在注入漏洞
- PreparedStatement传入参数化的sql语句, 同时检查合法性, 效率高可以重用, 防止sql注入
- CallableStatement接口扩展PreparedStatement，用来调用存储过程
- BatchedStatement用于批量操作数据库，BatchedStatement不是标准的Statement类

```java
public interface CallableStatement extends PreparedStatement 
public interface PreparedStatement extends Statement 
```

1.2 Statement与PrepareStatement的区别

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
由上可以看出，PreparedStatement有预编译的过程，已经绑定sql，之后无论执行多少遍，都不会再去进行编译，而 statement 不同，如果执行多遍，则相应的就要编译多少遍sql，所以从这点看，preStatement 的效率会比 Statement要高一些

- 安全性

PreparedStatement是预编译的，所以可以有效的防止SQL注入等问题

- 代码的可读性和可维护性

PreparedStatement更胜一筹

**2. 游标**

### 六、算法与数据结构

**1. 二叉搜索树**:(Binary Search Tree又名：二叉查找树,二叉排序树)它或者是一棵空树,或者是具有下列性质的二叉树： 若它的左子树不空,则左子树上所有结点的值均小于它的根结点的值；若它的右子树不空,则右子树上所有结点的值均大于它的根结点的值；它的左、右子树也分别为二叉搜索树。

**2. RBT红黑树**

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

**3. 排序算法**

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

- 树排序[不稳定]
应用：TreeSet的add方法、TreeMap的put方法
不支持相同元素,没有稳定性问题
复杂度：平均最差O(nlogn)

- 堆排序(就地排序)[不稳定]
复杂度：O(nlogn) - O(nlgn) - O(nlgn) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

- 快速排序[不稳定]
复杂度：O(nlgn) - O(nlgn) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]
栈空间0(lgn) - O(n)

- 选择排序[不稳定]
复杂度：O(n^2) - O(n^2) - O(n^2) - O(1)[平均 - 最好 - 最坏 - 空间复杂度]

- 希尔排序[不稳定]
复杂度 小于O(n^2) 平均 O(nlgn) 最差O(n^s)[1<s<2] 空间O(1)

九大内部排序算法代码及性能分析参见我的[GitHub](https://github.com/it-interview/easy-job/tree/master/algorithm)

**4. 查找与散列**

4.1 散列函数设计

* 直接定址法:```f(key) = a*key+b```

简单、均匀,不易产生冲突。但需事先知道关键字的分布情况,适合查找表较小且连续的情况,故现实中并不常用

* 除留余数法:```f(key) = key mod p (p<=m) p取小于表长的最大质数 m为表长```

* DJBX33A算法(time33哈希算法```hash = hash*33+(unsigned int)str[i];```

平方取中法 折叠法 更多....

4.2 冲突处理

闭散列(开放地址方法):要求装填因子a较小，闭散列方法把所有记录直接存储在散列表中

- 线性探测:易产生堆积现象(基地址不同堆积在一起)
- 二次探测:f(key) = (f(key)+di) % m di=1^2,-1^2,2^2,-2^2...可以消除基本聚集
- 随机探测:f(key) = (f(key)+di),di采用随机函数得到,可以消除基本聚集
- 双散列:避免二次聚集

开散列(链地址法):原地处理

- 同义词记录存储在一个单链表中,散列表中子存储单链表的头指针。
- 优点:无堆积 事先无需确定表长 删除结点易于实现 装载因子a>=1,缺点:需要额外空间

**5. 跳表**

----------

### 致谢和参考文献

深入理解Java虚拟机

最近5年133个Java面试问题列表

### 联系我

- WebSite:[http://www.tinymood.com][1]
- Mail: 932191671@qq.com

作者 [_NoThankYou][2]       

[1]: http://www.tinymood.com   
[2]: http://weibo.com/u/1662536394