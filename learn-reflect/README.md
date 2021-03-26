# Java的反射
## Java反射机制的概述

Reflection（反射）是Java被视为动语言的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。 

可以使用代码```Class c = Class.forName("java.lang.String");```来获取Class对象

加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象（一个类只有一个Class对象，类似于单例模式），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。这个对象就像一面镜子，透过这个镜子看到类的结构，所以，我们形象的称之为反射。

![20200921211112.png](https://i.loli.net/2020/09/22/SDbijtaNqZnCWw7.png)

反射的优点：可以实现动态创建对象和编译，体现出很大的灵活性

反射的缺点：对性有影响，使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么它满足我们的要求。这类操作总是慢于直接执行相同的操作。

我们先创建一个如下的User类：
```Java
package Reflection;

public class User {
    private int age;
    private String name;
    private int id;

    public User() {
    }

    public User(int age, String name, int id) {
        this.age = age;
        this.name = name;
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    @Override
    public String toString() {
        return "User{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", id=" + id +
                '}';
    }
}
```
测试代码：
```java
@Test
public void test01() throws ClassNotFoundException {
    Class clazz = Class.forName("Reflection.User");
    System.out.println(clazz);//输出信息为：class Reflection.User
    Class clazz1 = Class.forName("Reflection.User");//创建另一个Class
    Assert.assertEquals(clazz, clazz1);//返回true，相当于是单例模式，打印出的hashCode是同一个
}
```

## 创建Class对象的几种方式
另外提一个：Java创建对象的几种方式（网易互娱面试题）
1. 通过new关键字创建
2. 通过反射进行创建
3. 通过clone()方法进行创建
4. 通过反序列化的readObject()方法进行创建

创建Class对象有如下几种方式：
1. Class.forName()静态方法
2. 类名.class获得
3. 对象.getClass()获得
4. Java内置的包装类型（如Integer等）都有一个Type属性
   
测试代码如下：
```Java
@Test
public void testGenerateClass() throws ClassNotFoundException {
    Class clazz1 = Class.forName("Reflection.User");
    Class clazz2 = User.class;
    User user = new User();
    Class clazz3 = user.getClass();
    Assert.assertEquals(clazz1, clazz2);//返回true
    Assert.assertEquals(clazz2, clazz3);//返回true

    Class clazz4 = Integer.TYPE;//包装类型都有一个Type属性
    System.out.println(clazz4);//输出：int
}
```
```clazz1==clazz2==clazz3```也说明了每个类的Class对象在JVM中只存在一份。

## 所有类型的Class对象
拥有Class对象的类型有如下几种：
1. class，所有的类
2. interface，所有的接口
3. []，所有的数组，包括二维数组
4. enum，所有的枚举
5. annotation，所有的注解
6. primitive type，所有的基本数据类型
7. void，空
   
测试代码如下：

```Java
@Test
public void testAllKindsOfClass() {
    Class clazz1 = User.class;//class

    Class clazz2 = Comparable.class;//interface

    int[] values = new int[10];
    Class clazz3 = values.getClass();//[]

    String[][] strings = new String[10][10];
    Class clazz4 = strings.getClass();//[][]

    Class clazz5 = Override.class;//annotation

    Class clazz6 = Proxy.Type.class;//enum

    Class clazz7 = int.class;//primitive type

    Class clazz8 = void.class;//void
}
```
输出的信息如下：
```
class Reflection.User
interface java.lang.Comparable
class [I
class [[Ljava.lang.String;
interface java.lang.Override
class java.net.Proxy$Type
int
void
```
值得注意的是任何同类型的数组返回的Class对象是同一个，如下代码返回的是true：
```Java
int []values1 = new int[10];
Class clazz1 = values1.getClass();
int []values2 = new int[100];
Class clazz2 = values2.getClass();
Assert.assertEquals(clazz1, clazz2);//返回true
```
## Java的内存分析
![20200921222810.png](https://i.loli.net/2020/09/22/D8YliSg6NPETB47.png)

**类加载器加载class的过程：**

1. 加载：将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后生成一个代表这个类的```java.langClass```对象． 

2. 链接：将Java类的二进制代码合并到JVM的运行状态之中的过程。 
   >验证：确保加载的类信息符合JVM规范，没有安全方面的问题 
   
   >准备：正式为类变量(static)分配内存并设置类变量默认初始值的阶段，这些内存都将在方法区中进行分配。 
   
   >解析：虚拟机常量池内的符号引用（常量名）替换为直接引用（地址）的过程。 
3. 初始化：执行类构造器```<clinit>()```方法的过程。
   >类构造器```<clinit>()```方法是由编译期自动收集类中所有类变量的賦值动作和静态代码块中的语句合并产生的。（类构造器是构造类信息的，不是构造该类对象的构造器）。
   
   >当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化。 
   
   >虚拟机会保证一个类的```<clinit>()```方法在多线程环境中被正确加锁和同步。
   
![image.png](https://i.loli.net/2020/09/21/bmZtCXdcO1APqFu.png)

## 类的初始化的时期
1. 类的主动引用（一定会发生类的初始化） 
   >当虚拟机启动，先初始化main方法所在的类 

   >new一个类的对象
   
   >调用类的静态成员（除了final常量）和静态方法

   >使用java.lang，reflect包的方法对类进行反射调用 
   
   >当初始化一个类，如果其父类没有被初始化，则先会初始化它的父类 
   
2. 类的被动引用（不会发生类的初始化） 
   >当访问一个静态域时，只有真正声明这个域的类才会被初始化。如：当通过子类引用父类的静态变量，不会导致子类初始化
   
   >通过数组定义类引用，不会触发此类的初始化
   
   >引用常量不会触发此类的初始化（常量在链接阶段就存入调用类的常量池中了）

我们的测试代码如下：
```Java
package Reflection;

public class Main {
    static {
        System.out.println("main类被加载");
    }

    public static void main(String[] args) throws ClassNotFoundException {
        //Person person = new Student();//main，子类，父类均被加载

        //Class.forName("Reflection.Person");//main，父类被加载

        //System.out.println(Student.value);//不会产生子类的加载

        //Person[] people = new Person[10];//不会进行person类的加载

        //System.out.println(Person.b);//不会进行person类的加载
    }
}

class Person {
    static {
        System.out.println("父类被加载");
    }

    static int value = 100;
    static final int b = 10;
}

class Student extends Person {
    static {
        System.out.println("子类被加载");
    }
}
```
## 类加载器
**类加载器的作用**
1. 将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时的数据结构，然后在堆中生成一个代表这个类的```java.lang.Class```对象，作为方法区中类数据的访问入口。 
2. 类缓存：标准的JavaSE类加载器可以按要求查找类，但一旦某个类被加载到类加载器中，它将维持加载（缓存）一段时间。不过JVM垃圾回收机制可以回收这些Class对象。

![img.png](https://i.niupic.com/images/2020/09/22/8I7e.png)

测试代码如下：
```Java
@Test
public void testClassLoader() {
    ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
    System.out.println(systemClassLoader);//系统类加载器

    ClassLoader extClassLoader = systemClassLoader.getParent();
    System.out.println(extClassLoader);//拓展类加载器

    ClassLoader rootClassLoader = extClassLoader.getParent();
    System.out.println(rootClassLoader);//根类加载器，无法直接获取，为null
}
```
输出结果如下：
```Java 
jdk.internal.loader.ClassLoaders$AppClassLoader@1f89ab83
jdk.internal.loader.ClassLoaders$PlatformClassLoader@1a3869f4
null
```
测试类由那些类加载器加载以及可以加载的路径代码如下：
```Java
//系统类加载器
ClassLoader classLoader = Class.forName("Reflection.Person").getClassLoader();
System.out.println(classLoader);
//根加载器，为null
ClassLoader classLoader1 = String.class.getClassLoader();
System.out.println(classLoader1);

//获取系统的类加载的路径
System.out.println(System.getProperty("java.class.path"));
```

**类加载的双亲委派机制**
![img.png](https://i.niupic.com/images/2020/09/22/8I7h.png)

一个类加载器加载一个类时，首先会把加载动作委派给他的父加载器，如果父加载器无法完成这个加载动作时才由该类加载器进行加载。由于类加载器会向上传递加载请求，所以一个类加载时，首先尝试加载它的肯定是启动类加载器（逐级向上传递请求，直到启动类加载器，它没有父加载器），之后根据是否能加载的结果逐级让子类加载器尝试加载，直到加载成功。

比如我们自己写了一个```com.test.T1```类，假设我们没有自定义类加载器，那么这个类会由应用程序类加载器加载。应用程序类加载器加载时先把加载任务委派给扩展类加载器（父加载器），然后扩展类加载器再把加载任务委托给启动类加载器（父加载器），启动类加载器没有父加载器。于是它自己尝试加载，结果发现T1类并不在自己的记载类路径之中，于是告诉扩展类加载器（子加载器）自己无法加载该类。这时扩展类只好自己加载这个类，结果发现也无法加载该类，于是它也告诉应用程序类加载器这个消息，这时应用程序类加载器自己进行T1类的加载动作，加载成功。

再比如我们自己写了一个```java.lang.String```类，假设我们没有自定义类加载器，那么这个类会由应用程序类加载器加载。应用程序类加载器加载时先把加载任务委派给扩展类加载器（父加载器），然后扩展类加载器再把加载任务委托给启动类加载器（父加载器），启动类加载器没有父加载器。于是它自己尝试加载，发现```java.lang.String```类在自己的记载类路径之中(或者是已经加载)，则加载成功，而不会加载用户编写的```java.lang.String```类。

由上面的第二个例子我们可以看出双亲委派机制的好处：

1. 特定的类由特定的类加载器加载，每次加载都委托父类的过程让类对象在内存中的数量保持为一个，让同类名的类无法被替换。
2. 如果没有双亲委派机制，只有一个类加载器，我们自己写一个java.lang.Object类，也可以被加载，结果就是内存中有两个Object类，引用的时候会造成安全的问题。而且一些核心的类可能会被替换，导致重大的安全问题。
3. 有了双亲委派机制，我们自己写的java.lang.Object类在加载时会被加载器委托给父加载器，在某一个父加载器中发现内存中已经存在了java.lang.Object类，那么将不会进行加载，这样就保证了特定的类只能有一个在内存中。

## 获取运行时类的完整结构
反射可以在运行时获取类的完整结构，包括如下：
1. 实现的全部接口
2. 所继承的父类
3. 全部的构造器
4. 全部的方法
5. 全部的Field（字段）
6. 注解
7. ...
   
测试代码如下：
```Java
@Test
public void testClassInfo() throws ClassNotFoundException {
    Class clazz = Class.forName("Reflection.User");

    System.out.println(clazz.getName());//包名+类名
    System.out.println(clazz.getSimpleName());//类名

    //获得所有的属性
    Field[] fields = clazz.getFields();//只能找到public的属性
    for (Field field : fields) {
        System.out.println(field);//这里输出为空
    }

    fields = clazz.getDeclaredFields();//这里能找到所有的属性

    for (Field field : fields) {
        System.out.println(field);//这里可以输出信息
    }

    //获取对象的方法
    Method[] methods = clazz.getMethods();//只能找到public的方法
    for (Method method : methods) {
        System.out.println(method);//这里输出为空
    }

    methods = clazz.getDeclaredMethods();//这里可以找到所有的方法
    for (Method method : methods) {
        System.out.println(method);//这里输出有信息
    }

    Constructor[] constructors = clazz.getConstructors();
    for (Constructor constructor : constructors) {
        System.out.println(constructor);
    }

    //上述三种方法也可通过指定方法明进行获取对应的一个方法或者属性或者构造器
}
```

## 通过反射动态获取对象
测试代码如下：
```Java
@Test
public void testGetInstance() throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {
    Class clazz = Class.forName("Reflection.User");
    //newInstance()方法
    User user = (User) clazz.getDeclaredConstructor().newInstance();
    System.out.println(user);
    //使用Method.invoke方法来运行对应的方法
    Method method = clazz.getMethod("setName", String.class);
    method.invoke(user, "shuqy");
    System.out.println(user);

    //操作对应的属性
    Field field = clazz.getDeclaredField("age");
    //私有属性设置为可接触才能继续操作
    field.setAccessible(true);
    field.setInt(user, 100);
    System.out.println(user);
}
```
## 反射操作泛型
由于Java采用泛型擦除的机制来引入泛型，Java中的泛型仅仅是给编译器javac使用的，确保数据的全性和免去强制类型转换问题，但是一旦编译完成所有和泛型有关的类型全部擦除。

为了过反射操作这些类型，Java新增了ParameterizedType，GenericArrayType TypeVariable和WildcardType几种类型来代表不能被归一到Class类中的类型但是又和原始类型齐名的类型。

ParametenzedType：表示一一种参数化类型比如Collection< String >

GenericArrayType·表示一种元素类型是参数化类型或者类型变量的数组类型 

TypeVariable：是各种类型变量的公共父接口

WildcardType代表一整配符类型表达式

测试代码如下：

```Java
public static void testMethod(Map<String, User> userMap) {

}

@Test
public void testGeneric() throws NoSuchMethodException {
    Method method = User.class.getMethod("testMethod", Map.class);
    Type[] genericParameterTypes = method.getGenericParameterTypes();
    for (Type genericParameterType : genericParameterTypes) {
        System.out.println("#" + genericParameterType);
        if (genericParameterType instanceof ParameterizedType) {
            Type[] actualTypeArguments = ((ParameterizedType) genericParameterType).getActualTypeArguments();
            for (Type actualTypeArgument : actualTypeArguments) {
                System.out.println(actualTypeArgument);
            }
        }
    }
}
```