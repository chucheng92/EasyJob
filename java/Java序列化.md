## Java序列化

对于一个存在Java虚拟机中的对象来说，其内部的状态只是保存在内存中。JVM退出之后，内存资源也就被释放，Java对象的内部状态也就丢失了。而在很多情况下，对象内部状态是需要被持久化的，将运行中的对象状态保存下来(最直接的方式就是保存到文件系统中)，在需要的时候可以还原，即使是在Java虚拟机退出的情况下。 

对象序列化机制是Java内建的一种对象持久化方式，可以很容易实现在JVM中的活动对象与字节数组(流)之间进行转换，使用得Java对象可以被存储，可以被网络传输，在网络的一端将对象序列化成字节流，经过网络传输到网络的另一端，可以从字节流重新还原为Java虚拟机中的运行状态中的对象。 

### 1.相关的接口 
Java类中对象的序列化工作是通过ObjectOutputStream和ObjectInputStream来完成的。          
 
Java代码
```java
ObjectOutputStream(OutputStream out);  
    void writeObject(Object obj);//将指定的对象的非transient，非static属性，写入ObjectOutputStream  
     
    ObjectInputStream(InputStream in);  
    Object readObject();//从指定的流中读取还原对象信息  
```
只能使用readObject()|writeObject()方法对对象进行读写操作。除对象之外，Java中的基本类型和数组也可以被序列化,对于基本类型，可以使用readInt(),writeInt(), 
readDouble(),writeDouble()等类似的接口进行读写。 


### 2.Serializable接口  
对于任何需要被序列化的对象，都必须要实现接口Serializable,它只是一个标识接口，本身没有任何成员，只是用来标识说明当前的实现类的对象可以被序列化. 


### 3.transient关键字 
如果在类中的一些属性，希望在对象序列化过程中不被序列化，使用关键字transient标注修饰就可以.当对象被序列化时，标注为transient的成员属性将会自动跳过。 

  
### 4.Java序列化中需要注意: 
(1).当一个对象被序列化时，只保存对象的非静态成员变量，不能保存任何的成员方法,静态的成员变量和transient标注的成员变量。 
(2).如果一个对象的成员变量是一个对象，那么这个对象的数据成员也会被保存还原，而且会是递归的方式。 
(3).如果一个可序列化的对象包含对某个不可序列化的对象的引用，那么整个序列化操作将会失败，并且会抛出一个NotSerializableException。可以将这个引用标记transient,那么对象仍然可以序列化。 


### 5.一个综合实例: 
```java
class Student implements Serializable{  
      
    private String name;  
    private transient int age;  
    private Course course;  
      
    public void setCourse(Course course){  
        this.course = course;  
    }  
      
    public Course getCourse(){  
        return course;  
    }  
      
    public Student(String name, int age){  
        this.name = name;  
        this.age = age;  
    }  
  
    public String  toString(){  
        return "Student Object name:"+this.name+" age:"+this.age;  
    }  
}  
  
class Course implements Serializable{  
      
    private static String courseName;  
    private int credit;  
      
    public Course(String courseName, int credit){  
        this.courseName  = courseName;  
        this.credit = credit;  
    }  
      
    public String toString(){  
          
        return "Course Object courseName:"+courseName  
               +" credit:"+credit;  
    }  
}  
```

**将对象写入文件，序列化** 
```java
public class TestWriteObject{  
  
    public static void main(String[] args) {  
  
        String filePath = "C://obj.txt";  
        ObjectOutputStream objOutput = null;  
        Course c1 = new Course("C language", 3);  
        Course c2 = new Course("OS", 4);  
          
        Student s1 = new Student("king", 25);  
        s1.setCourse(c1);  
          
        Student s2 = new Student("jason", 23);  
        s2.setCourse(c2);  
  
        try {  
            objOutput = new ObjectOutputStream(new FileOutputStream(filePath));  
            objOutput.writeObject(s1);  
            objOutput.writeObject(s2);  
            objOutput.writeInt(123);  
        } catch (FileNotFoundException e) {  
            e.printStackTrace();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }finally{  
            try {  
                objOutput.close();  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
        }  
          
        System.out.println("Info:对象被写入"+filePath);  
    }  
```

**从文件中读取对象，反序列化**
```java 
public class TestReadObject  {  
      
    public static void main(String[] args) {  
          
        String filePath = "C://obj.txt";  
        ObjectInputStream objInput = null;  
        Student s1 = null,s2 = null;  
        int intVal = 0;  
      
        try {  
            objInput = new ObjectInputStream(new FileInputStream(filePath));  
            s1 = (Student)objInput.readObject();  
            s2 = (Student)objInput.readObject();  
            intVal = objInput.readInt();  
              
        } catch (FileNotFoundException e) {  
            e.printStackTrace();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }catch(ClassNotFoundException cnfe){  
            cnfe.printStackTrace();  
        }finally{  
              
            try {  
                objInput.close();  
            } catch (IOException e) {  
                e.printStackTrace();  
            }  
        }  
          
        System.out.println("Info:文件"+filePath+"中读取对象");  
        System.out.println(s1);  
        System.out.println(s1.getCourse());  
        System.out.println(s2);  
        System.out.println(s2.getCourse());  
        System.out.println(intVal);  
    }  
}  
```

输出: 

[TestWriteObject] 
Info:对象被写入C://obj.txt 

[TestReadObjec] 
Info:文件C://obj.txt中读取对象 
Info:文件C://obj.txt中读取对象 
Student Object name:king age:0 
Course Object courseName:null credit:3 
Student Object name:jason age:0 
Course Object courseName:null credit:4 
123 

可知Person中的age属性被标注为transient后，在序列化对象时，age属性就没有被序列化了; Course中的name属性被static后，Course的name静态属性就没有被序列化;虽然是序列化Person对象，但是Person所引用的Course对象也被初始化了。 

----------

### 全文下载

+ [Github托管](https://github.com/Lemonjing/TinyMood/blob/master/技术文章/Java序列化.md)

### 我的其他项目

+ [TinyMooc](https://github.com/Lemonjing/tinymooc)
+ [TinyCoding](https://github.com/Lemonjing/tinycoding)
+ [JavaSE工程](https://github.com/Lemonjing/myjavase)
+ [Leetcode题解](https://github.com/Lemonjing/leetcode)

### 最近更新

+ [最近更新](https://github.com/Lemonjing/TinyMood/blob/master/UPDATE_LOG.md)

### 联系我

- WebSite:[http://www.tinymood.com][1]
- Mail: 932191671@qq.com
- Linux交流群: [257265338][2]
- 邂逅厦大交流群:[103949230][3]

作者 [微博@Campanulaceae][4]       
2015 年 11月 15日


[1]: http://www.tinymood.com   
[2]: http://jq.qq.com/?_wv=1027&k=ZKsbKb
[3]: http://jq.qq.com/?_wv=1027&k=Xxno3g
[4]: http://weibo.com/u/1662536394
