Equals 和 HashCode
☞ ==和equals的区别
==解读
- 基本数据类型：比较值是否相等
- 其他数据类型：比较内存中的地址是否相同
equals()解读
equals()本质是==，对于String和Integer等类，其内部重写了equals()方法，使之比较值是否相等；如果不重写，默认使用Object的equals()方法，比较是否指向同一个引用
参考：https://segmentfault.com/a/1190000039132885
☞ 两个对象的hashCode()相同，equals方法是否为ture
不对；“通话”和“重地”的hashCode相同，但是equals显然不同；
public static void main(String[] args) {
        String s4="通话", s5="重地";
        System.out.println("s4:"+s4.hashCode()+", s5:"+s5.hashCode()+", s4与s5的equals结果:"+s4.equals(s5));
}
结果：
s4:1179395, s5:1179395, s4与s5的equals结果:false
☞ 为什么要重写hashCode()和 equals() 方法
- 在使用hashMap时，如果使用自定义的对象做key，则需要重写hashCode方法和equals方法；因为在调用hashMap的put方法时，会调用key的hashCode方法，如果没有重写hashCode方法，那么调用的实际上是Object的hashCode方法，计算的是对象引用的hashCode值；这样一来，两个逻辑上相同的对象，有可能hashCode不同，导致在hashMap中有两个相同的entry
- 当重写了hashCode方法，在调用put方法时，相同的对象会得到相同的hashCode值，在hash表中如果计算出的hashCode为下标的位置存在元素，会调用equals方法比较两个元素是否相同，如果没有重写equals方法，调用的便是Object的equals方法，比较的是两个引用是否相同，即判断两个引用是否指向同一个对象；这样一来两个逻辑上相同的对象，equals返回可能为false
参考：https://www.cnblogs.com/JavaArchitect/p/10474448.html
☞ lomnok中是如何重写hashcode和equals的
参考：https://developer.51cto.com/art/202107/674924.htm
String
☞  说一下 String
- String 是 Java 中最常用的一个类，被设计成 final 的，即不可改变，如果对一个 String 变量 a 进行了赋值，然后将该变量 a 的赋值进行了改变，那么变量 a 实际上指向了新的字符串，原来的其实并没有改变；这样做有两个好处：一是可以进行字符串的复用，二是保证了系统的安全性
验证：可以通过观察对象的个数来验证 String 的不可变性和复用性
- String 底层是利用 StringTable 这样的数据结构进行存储，StringTable 是一个 Hash 表，最小长度是 1009；在 JDK 1.6 时，StringTable 存在于方法区（永久代），在 1.6 之后被挪至堆中，因为在方法区中的 StringTable 进行 GC 总是不方便，挪到堆中，可以更加方便的进行 GC；而且在堆中，可以给 StringTable 分配更大的空间，这样降低了 StringTable 的碰撞，提高了性能
- String 类中，Java 1.9 之前是利用 char 数组作为存储单元，一个 char 元素占用两个字节，经过统计发现 90% 以上的情况，每个 char 元素只需要占用一个字节即可，所以在 1.9 中将 String 的存储由 char 数组改为了 byte 数组
参考：Java基础1-String详解 - 掘金
☞ 说一下 StringTable
- StringTable 是 String 存储的底层的数据结构，是利用 hash 表来做存储的；最小长度是 1009；可以通过 -XX:StringTableSize=1009 去设置；长度过小时，发生碰撞的几率会增大
- 在 JDK 1.6 之后，StringTable 从方法区挪到了堆中存储，一是因为方法区空间较小，而是以为方法区 GC 不方便；在实际中，可以通过 -XX:+PrintStringTabIeStat1stIcs 参数来查看 StringTable 的情况
☞ String，StringBuilder，StringBuffer的区别？String为什么是不可变的？
String
Java中String类是final类型的，保存字符串的字符数组也是final修饰的（Java9 之后使用的是 byte 数组），所以String创建的是不可变的对象，每次操作都会创建新的对象；在Java中无论使用何种方式进行字符串连接，实际上都使用的是StringBuilder
StringBuilder 和 StringBuffer
StringBuilder和StringBuffer都在原对象的基础上操作；StringBuilder线程不安全，但是性能强于StringBuffer；StringBuffer的线程安全机制基于synchronized
适用场景
- String适用于数据量较少的操作
- StringBuilder适用于单线程环境下，数据量较多的操作
- StringBuffer适用于多线程环境下，数据量较多的操作
OOP
☞ 重写与重载的区别
区别点
重载
重写
发生范围
同一个类
子类
参数列表
必须不同
必须相同
返回类型
可以不同
必须相同
异常
可修改
可以减少或删除，但不能抛出新的异常或更大的异常
访问修饰符
可以修改
可以修改，但不能降低访问限制
发生阶段
编译期
运行期
☞ 说一下抽象类
- 普通类中不能包含抽象方法，抽象类中可以包含抽象方法
- 普通类可以实例化，抽象类不能实例化
- 抽象类是用来继承的，不能使用final修饰；final修饰的类不能被继承
☞ 接口与抽象类的区别
接口
1. 接口中的变量都是常量，默认由public static final修饰
2. 接口中的方法都是抽象方法
3. 接口可以多继承
4. 接口不可以实例化，只能实例化其实现类
5. 接口的作用是制定规范，降低代码的耦合性
抽象类
1. 抽象类中不一定有抽象方法，有抽象方法的类必定是抽象类（或接口）
2. 抽象类不可以被实例化，只能实例化其子类
3. 抽象类可以实现接口
4. 抽象类的作用是提取相同代码的公共部分，以提高代码的重用性
☞ 面向对象的三大特性六大原则
三大特性
- 封装
封装是指将一个对象的属性私有化，同时提供一些方法供外界访问属性
- 继承
继承是指在一个已经存在的类的基础上构建新的类，子类拥有父类的全部属性和方法，但是不能访问私有的属性和方法；同时子类也可以在父类的基础上进行扩展新的功能和属性；继承提高了代码的复用性
- 多态
五大原则
- 单一职责原则（Single Responsiblity Principle SRP）
- 接口隔离原则（Interface Segregation Principle，ISP）
- 依赖倒转（倒置）原则（Dependency Inversion Principle，DIP）
- 里氏代换原则（Liskov Substitution Principle，LSP）
- 开闭原则（Open Closed Principle，OCP）
- 迪米特法则（Law of Demeter）或最小知识原则（Principle of Least Knowledge，PLK）
- 合成/聚合复用原则（Composite/Aggregate Reuse Principle，CARP）
参考：面向对象三大特性五大原则 + 低耦合高内聚
注解
📌 
其他
☞ Java 中对象序列化
- 序列化：将Java对象转为字节序列的过程称为序列化
- 反序列化：将字节序列转为Java对象的过程称为反序列化
参考：
1. Java 之 Serializable 序列化和反序列化
2. java序列化
☞ Java中的装箱与拆箱
参考：深入剖析Java中的装箱和拆箱
☞ 为什么要使用克隆
- 克隆可以保持对象已经修改过后的属性，而使用new创建对象，所有属性都是初始值，需要一个个属性去手动赋值；而调用clone方法，则可以快速实现对象复制
参考： Java提高篇——对象克隆（复制）
☞ 如何实现对象克隆
1. 实现cloneable接口，重写Object中clone方法实现克隆
2. 实现serializable接口，利用对象的序列化与反序列化实现对象之间的深克隆
☞ 深拷贝和浅拷贝
1. 在浅拷贝中，如果原型对象的成员变量是基本类型，浅拷贝会复制一份到克隆对象；如果原型对象的成员变量是引用类型，会复制一份引用类型的内存地址到克隆对象，也就是说原型对象和克隆对象中引用类型的地址指向同一个对象
2. 在深拷贝中，会将原型对象的所有成员变量拷贝一份到克隆对象
☞ 说一下按值传递和引用传递
方法的参数类型一般为基本类型和引用类型，两种不同的类型在使用时，区别如下
- 基本类型和String：当方法的参数为基本类型时，在调用方法时，会拷贝一份数值给方法使用；在方法内部对数值进行修改不会影响原值
- 引用类型：当方法的参数为引用类型时，在调用方法时，会拷贝一份引用的值（即对象在内存中的地址）改方法使用；在方法内部对对象进行修改时，会影响源对象的值
public class Call_By_Value {

    public static void main(String[] args){
            int num = 10;
            M1(num);
            // 输出: M1 num: 15        Main num: 10
            System.out.println("\tMain num: " + num);

            int[] arr = {1, 2, 3};
            M2(arr);
            // 输出: M2 arr: [6, 2, 3]        Main arr: [6, 2, 3]
        System.out.println("\tMain arr: " + Arrays.toString(arr));

        Student student = new Student("lt", 12);
        M3(student);
        // 输出: M3 student: Student(name=lt, age=25)        Main student: Student(name=lt, age=25)
        System.out.println("\tMain student: " + student);

        Person p1 = new Person("lt", 12);
        Person p2 = new Person("yr", 15);
        M4(p1, p2);
        // 输出: M4 p1: Person(name=yr, age=15), p2: Person(name=lt, age=12)        Main p1: Person(name=lt, age=12), p2: Person(name=yr, age=15)
        System.out.println("\tMain p1: " + p1 +  ", p2: " + p2);

        String s = "啦啦啦";
        M5(s);
        // 输出: M5 s: 啦啦啦, 德玛西亚万岁!        Main s: 啦啦啦
        System.out.println("\tMain s: " + s);

    }

    public static void M1(int num) {
        num += 5;
        System.out.print("M1 num: " + num);
    }

    public static void M2(int[] arr) {
        arr[0] += 5;
        System.out.print("M2 arr: " + Arrays.toString(arr));
    }

    public static void M3(Student student) {
        student.setAge(25);
        System.out.print("M3 student: " + student);
    }

    public static void M4(Person p1, Person p2) {
        Person p = p1;
        p1 = p2;
        p2 = p;
        p1.setAge(100);
        System.out.print("M4 p1: " + p1 +  ", p2: " + p2);
    }

    public static void M5(String s) {
        s += ", 德玛西亚万岁!";
        System.out.print("M5 s: " + s);
    }
}

@Data
@AllArgsConstructor
class Student {
    private String name;
    private int age;
}

@Data
@AllArgsConstructor
class Person {
    private String name;
    private int age;
}
☞  System.nanoTime() 与System.currentTimeMillis()
- 参考：一次用System.nanoTime()填坑System.currentTimeMills()的实例记录 - 博客园
☞ 内部类、外部类使用和调用
使用方式：<外部类>.<内部类>的方式创建实例
Test1.Static_Inner inner = new Test1.Static_Inner();
方法调用

外部类
其他类
内部类中 private 方法
利用实例对象直接调用
不能调用
内部类中 public 方法
利用实例对象直接调用
利用实例对象直接调用
- 如果外部类中定义了 private 方法，内部类中 public 方法调用了该 private 方法，那么通过内部类可以访问该外部类的 private 方法

参考
- Java 最常见的 208 道面试题：第一模块答案