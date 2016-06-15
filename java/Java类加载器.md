## 类装载器ClassLoader 

### 类装载器工作机制 

类装载器就是寻找类的节码文件并构造出类在JVM内部表示对象的组件。在Java中，类装载器把一个类装入JVM中，要经过以下步骤： 

[1.]装载：查找和导入Class文件； 
[2.]链接：执行校验、准备和解析步骤，其中解析步骤是可以选择的： 
    [2.1]校验：检查载入Class文件数据的正确性； 
    [2.2]准备：给类的静态变量分配存储空间； 
    [2.3]解析：将符号引用转成直接引用； 
[3.]初始化：对类的静态变量、静态代码块执行初始化工作。 

类装载工作由ClassLoader及其子类负责，ClassLoader是一个重要的Java运行时系统组件，它负责在运行时查找和装入Class字节码文件。JVM在运行时会产生三个ClassLoader：根装载器、ExtClassLoader（扩展类装载器）和AppClassLoader（系统类装载器）。其中，根装载器不是ClassLoader的子类，它使用C++编写，因此我们在Java中看不到它，根装载器负责装载JRE的核心类库，如JRE目标下的rt.jar、charsets.jar等。ExtClassLoader和AppClassLoader都是ClassLoader的子类。其中ExtClassLoader负责装载JRE扩展目录ext中的JAR类包；AppClassLoader负责装载Classpath路径下的类包。 

这三个类装载器之间存在父子层级关系，即根装载器是ExtClassLoader的父装载器，ExtClassLoader是AppClassLoader的父装载器。默认情况下，使用AppClassLoader装载应用程序的类，我们可以做一个实验： 

```java
public class ClassLoaderTest {
	public static void main(String[] args) {
		ClassLoader loader = Thread.currentThread().getContextClassLoader();
		System.out.println("current loader:"+loader);
		System.out.println("parent loader:"+loader.getParent());
		System.out.println("grandparent loader:"+loader.getParent(). getParent());
	}
}
```

运行以上代码，在控制台上将打出以下信息： 

```sh
current loader:sun.misc.Launcher$AppClassLoader@131f71a 
parent loader:sun.misc.Launcher$ExtClassLoader@15601ea 
//①根装载器在Java中访问不到，所以返回null 
grandparent loader:null
```

通过以上的输出信息，我们知道当前的ClassLoader是AppClassLoader，父ClassLoader是ExtClassLoader，祖父ClassLoader是根类装载器，因为在Java中无法获得它的句柄，所以仅返回null。 

JVM装载类时使用“全盘负责委托机制”，“全盘负责”是指当一个ClassLoader装载一个类的时，除非显式地使用另一个ClassLoader，该类所依赖及引用的类也由这个ClassLoader载入；“委托机制”是指先委托父装载器寻找目标类，只有在找不到的情况下才从自己的类路径中查找并装载目标类。这一点是从安全角度考虑的，试想如果有人编写了一个恶意的基础类（如java.lang.String）并装载到JVM中将会引起多么可怕的后果。但是由于有了“全盘负责委托机制”，java.lang.String永远是由根装载器来装载的，这样就避免了上述事件的发生。 

### ClassLoader重要方法 

在Java中，ClassLoader是一个抽象类，位于java.lang包中。下面对该类的一些重要接口方法进行介绍： 

- `Class loadClass(String name)`
    name参数指定类装载器需要装载类的名字，必须使用全限定类名，如com.baobaotao. beans.Car。该方法有一个重载方法loadClass(String name ,boolean resolve)，resolve参数告诉类装载器是否需要解析该类。在初始化类之前，应考虑进行类解析的工作，但并不是所有的类都需要解析，如果JVM只需要知道该类是否存在或找出该类的超类，那么就不需要进行解析。 
- `Class defineClass(String name, byte[] b, int off, int len)`
   将类文件的字节数组转换成JVM内部的java.lang.Class对象。字节数组可以从本地文件系统、远程网络获取。name为字节数组对应的全限定类名。 
- `Class findSystemClass(String name)`
   从本地文件系统载入Class文件，如果本地文件系统不存在该Class文件，将抛出ClassNotFoundException异常。该方法是JVM默认使用的装载机制。 
- `Class findLoadedClass(String name)`
  调用该方法来查看ClassLoader是否已装入某个类。如果已装入，那么返回java.lang.Class对象，否则返回null。如果强行装载已存在的类，将会抛出链接错误。 
- `ClassLoader getParent()`
   获取类装载器的父装载器，除根装载器外，所有的类装载器都有且仅有一个父装载器，ExtClassLoader的父装载器是根装载器，因为根装载器非Java编写，所以无法获得，将返回null。 

除JVM默认的三个ClassLoader以外，可以编写自己的第三方类装载器，以实现一些特殊的需求。类文件被装载并解析后，在JVM内将拥有一个对应的java.lang.Class类描述对象，该类的实例都拥有指向这个类描述对象的引用，而类描述对象又拥有指向关联ClassLoader的引用，如图所示。 

![](http://i.imgur.com/HuLuFXD.png)

每一个类在JVM中都拥有一个对应的java.lang.Class对象，它提供了类结构信息的描述。数组、枚举、注解以及基本Java类型（如int、double等），甚至void都拥有对应的Class对象。Class没有public的构造方法。Class对象是在装载类时由JVM通过调用类装载器中的defineClass()方法自动构造的。 

### 类的初始化
类什么时候才被初始化

1. 创建类的实例，也就是new一个对象
2. 访问某个类或接口的静态变量，或者对该静态变量赋值
3. 调用类的静态方法
4. 反射（Class.forName("com.lyj.load")）
5. 初始化一个类的子类（会首先初始化子类的父类）
6. JVM启动时标明的启动类，即文件名和类名相同的那个类
    
只有这6中情况才会导致类的类的初始化。

类的初始化步骤：
1. 如果这个类还没有被加载和链接，那先进行加载和链接
2. 假如这个类存在直接父类，并且这个类还没有被初始化（注意：在一个类加载器中，类只能初始化一次），那就初始化直接的父类（不适用于接口）
3. 加入类中存在初始化语句（如static变量和static块），那就依次执行这些初始化语句。
