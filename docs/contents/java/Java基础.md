# Java 基础

> 著作权归https://pdai.tech所有。 链接：https://www.pdai.tech/md/java/basic/java-basic-oop.html

## 面向过程和面向对象

**[面向过程](https://www.jianshu.com/p/7a5b0043b035)**

面向过程是具体化的，流程化的，解决一个问题，你需要一步一步的分析，一步一步的实现。代码从头写到尾，缺少业务封装，代码耦合度高。

优点：高效。
缺点：难维护、难复用、难扩展。

**面向对象**

面向对象是模型化的，你只需抽象出一个类，这是一个封闭的盒子，在这里你拥有数据也拥有解决问题的方法。需要什么功能直接使用就可以了，不必去一步一步的实现，至于这个功能是如何实现的，管我们什么事？我们会用就可以了。

优点：易维护、易复用、易扩展，低耦合。
缺点：性能比面向过程差。

## 面向对象(OOP)

### 三大特性

**封装**

利用抽象数据类型将数据和基于数据的操作封装在一起，使其构成一个不可分割的独立实体。数据被保护在抽象数据类型的内部，尽可能地隐藏内部的细节，只保留一些对外接口使之与外部发生联系。用户无需知道对象内部的细节，但可以通过对象对外提供的接口来访问该对象。

**继承**

继承实现了  **IS-A**  关系，例如 Cat 和 Animal 就是一种 IS-A 关系，因此 Cat 可以继承自 Animal，从而获得 Animal 非 private 的属性和方法。

继承应该遵循里氏替换原则，子类对象必须能够替换掉所有父类对象。

**多态**

多态分为编译时多态和运行时多态:

- 编译时多态主要指方法的重载

- 运行时多态指程序中定义的对象引用所指向的具体类型在运行期间才确定
  
  > 由于程序调用方法是在运行期才动态绑定的，那么引用变量所指向的具体实例对象在运行期才确定。所以这个对象的方法是运行期正在内存运行的这个对象的方法而不是引用变量的类型中定义的方法。

运行时多态有三个条件:

- 继承
- 重写
- 父类引用指向子类对象： `Parent p = new Child();` 

表现为当父类引用指向子类，调用父类引用的方法时会调用子类的重写的方法。

## 权限控制

| 修饰词       | 本类  | 同一个包的类 | 继承类 | 其他类 |
| --------- | --- | ------ | --- | --- |
| private   | √   | ×      | ×   | ×   |
| 无（默认）     | √   | √      | ×   | ×   |
| protected | √   | √      | √   | ×   |
| public    | √   | √      | √   | √   |

## 泛型

引入泛型的意义在于：

- **适用于多种数据类型执行相同的代码**（代码复用）

泛型的本质是为了参数化类型（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

### 类型擦除

Java泛型这个特性是从JDK 1.5才开始加入的，因此为了兼容之前的版本，Java泛型的实现采取了“**伪泛型**”的策略，即Java在语法上支持泛型，但是在编译阶段会进行所谓的“**类型擦除**”（Type Erasure），将所有的泛型表示（尖括号中的内容）都替换为具体的类型（其对应的原生态类型），就像完全没有泛型一样。

**擦除原则**

- 消除类型参数声明，即删除`<>`及其包围的部分。
- 根据类型参数的上下界推断并替换所有的类型参数为原生态类型：如果类型参数是无限制通配符或没有上下界限定则替换为Object，如果存在上下界限定则根据子类替换原则取类型参数的最左边限定类型（即父类）。
- 为了保证类型安全，必要时插入强制类型转换代码。
- 自动产生“桥接方法”以保证擦除类型后的代码仍然具有泛型的“多态性”。

**擦除步骤**

- 擦除类定义中的类型参数 - 无限制类型擦除
  
  当类定义中的类型参数没有任何限制时，在类型擦除中直接被替换为Object，即形如`<T>`和`<?>`的类型参数都被替换为Object。

- 擦除类定义中的类型参数 - 有限制类型擦除
  
  当类定义中的类型参数存在限制（上下界）时，在类型擦除中替换为类型参数的上界或者下界，比如形如`<T extends Number>`和`<? extends Number>`的类型参数被替换为`Number`，`<? super Number>`被替换为Object。

- 擦除方法定义中的类型参数
  
  擦除方法定义中的类型参数原则和擦除类定义中的类型参数是一样的

### 如何获取泛型的参数类型

既然类型被擦除了，那么如何获取泛型的参数类型呢？可以通过反射（`java.lang.reflect.Type`）获取泛型

`java.lang.reflect.Type`是Java中所有类型的公共高级接口, 代表了Java中的所有类型. Type体系中类型的包括：数组类型(GenericArrayType)、参数化类型(ParameterizedType)、类型变量(TypeVariable)、通配符类型(WildcardType)、原始类型(Class)、基本类型(Class), 以上这些类型都实现Type接口。

## 反射

**运行时获取类信息，动态调用对象方法。**

JAVA反射机制是在**运行**状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

### 反射基础

RRIT（Run-Time Type Identification）运行时类型识别。在《Thinking in Java》一书第十四章中有提到，其作用是在**运行时**识别一个对象的类型和类的信息。主要有两种方式：一种是“传统的”RTTI，它假定我们在编译时已经知道了所有的类型；另一种是“反射”机制，它允许我们在运行时发现和使用类的信息。

反射就是把java类中的各种成分映射成一个个的Java对象

例如：一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把个个组成部分映射成一个个对象。

### Class类

Class类，Class类也是一个实实在在的类，存在于JDK的java.lang包中。Class类的实例表示java应用运行时的类(class ans enum)或接口(interface and annotation)（每个java类运行时都在JVM里表现为一个class对象，可通过类名.class、类型.getClass()、Class.forName("类名")等方法获取class对象）

- **Class类的方法**

| 方法名                | 说明                                                                                                         |
| ------------------ | ---------------------------------------------------------------------------------------------------------- |
| forName()          | (1)获取Class对象的一个引用，但引用的类还没有加载(该类的第一个对象没有生成)就加载了这个类。<br />(2)为了产生Class引用，forName()立即就进行了初始化。                 |
| Object-getClass()  | 获取Class对象的一个引用，返回表示该对象的实际类型的Class引用。                                                                       |
| getName()          | 取全限定的类名(包括包名)，即类的完整名字。                                                                                     |
| getSimpleName()    | 获取类名(不包括包名)                                                                                                |
| getCanonicalName() | 获取全限定的类名(包括包名)                                                                                             |
| isInterface()      | 判断Class对象是否是表示一个接口                                                                                         |
| getInterfaces()    | 返回Class对象数组，表示Class对象所引用的类所实现的所有接口。                                                                        |
| getSupercalss()    | 返回Class对象，表示Class对象所引用的类所继承的直接基类。应用该方法可在运行时发现一个对象完整的继承结构。                                                  |
| newInstance()      | 返回一个Oject对象，是实现“虚拟构造器”的一种途径。使用该方法创建的类，必须带有无参的构造器。                                                          |
| getFields()        | 获得某个类的所有的公共（public）的字段，包括继承自父类的所有公共字段。 类似的还有getMethods和getConstructors。                                    |
| getDeclaredFields  | 获得某个类的自己声明的字段，即包括public、private和proteced，默认但是不包括父类声明的任何字段。类似的还有getDeclaredMethods和getDeclaredConstructors。 |

### 反射调用流程

- 反射类及反射方法的获取，都是通过从列表中搜寻查找匹配的方法，所以查找性能会随类的大小方法多少而变化；
- 当找到需要的方法，都会copy一份出来，而不是使用原来的实例，从而保证数据隔离；
- 每个类都会有一个与之对应的Class实例，从而每个类都可以获取method反射方法，并作用到其他实例身上；
- 反射也是考虑了线程安全的，放心使用；
- 反射使用软引用relectionData缓存class信息，避免每次重新从jvm获取带来的开销；
- 反射调用多次生成新代理Accessor, 而通过字节码生存的则考虑了卸载功能，所以会使用独立的类加载器；
- 调度反射方法，最终是由jvm执行invoke0()执行；

### 用途

- SpringMVC，你在方法上写对象，传入的参数就会帮你封装到对象上
- MyBatis，可以让我们只写接口不写具体的实现类，就可以执行SQL
- Spring，在类上写上@Component注解，Spring就会帮你创建对象

> 通过“约定”使用姿势，使用反射在运行时获取相应的信息，实现代码功能的[通用性]和[灵活性]。

## 引用类型

无论是通过引用计算算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关。

Java 具有四种强度不同的引用类型。

- **强引用**
  
  被强引用关联的对象不会被回收。使用 new 一个新对象的方式来创建强引用。
  
  ```java
  Object obj = new Object();
  ```

- **软引用**
  
  被软引用关联的对象只有在内存不够的情况下才会被回收。
  
  ```java
  Object obj = new Object();
  SoftReference<Object> sf = new SoftReference<Object>(obj);
  obj = null;  // 使对象只被软引用关联
  ```

- **弱引用**
  
  被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。
  
  ```java
  Object obj = new Object();
  WeakReference<Object> wf = new WeakReference<Object>(obj);
  obj = null;
  ```
  
  `ThreadLocal` 在 Java 中是一个非常有用的工具，它允许你创建线程局部变量。每个线程访问这个变量时，都会通过其内部的 `ThreadLocalMap` 来获取或设置自己的值，这样每个线程都可以独立地修改自己的变量副本，而不会影响到其他线程。
  
  `ThreadLocal` 使用**弱引用**（`WeakReference`）来存储线程和 `ThreadLocal` 变量之间的映射关系，主要是出于内存泄漏的考虑。在 Java 中，如果 `ThreadLocal` 被设置为 `null`，但是线程还在运行，那么理论上这个 `ThreadLocal` 对应的 `Entry`（在 `ThreadLocalMap` 中）应该被垃圾回收器回收，以避免内存泄漏。然而，如果 `ThreadLocalMap` 使用强引用来存储这些 `ThreadLocal` 引用，那么即使 `ThreadLocal` 被设置为 `null`，只要线程还在运行，这些 `ThreadLocal` 引用就不会被回收，因为它们仍然被 `ThreadLocalMap` 持有。
  
  使用弱引用可以解决这个问题。当 `ThreadLocal` 被设置为 `null` 后，如果没有其他强引用指向它，那么它就可以被垃圾回收器回收。这样，即使线程还在运行，`ThreadLocalMap` 中的 `Entry` 也可以因为 `ThreadLocal` 的回收而被清除，从而避免了内存泄漏。
  
   **例子**
  
  假设我们有一个长时间运行的线程（比如一个服务线程），它使用 `ThreadLocal` 来存储一些线程特定的数据。如果在某个时间点，我们不再需要这个 `ThreadLocal` 变量，并且将其设置为 `null`，但是线程还在运行，我们希望这个 `ThreadLocal` 变量能够被垃圾回收。
  
  ```java
  public class ThreadLocalExample {
  
      private static final ThreadLocal<String> threadLocal = new ThreadLocal<>();
  
      public static void main(String[] args) {
          Thread thread = new Thread(() -> {
              // 设置 ThreadLocal 变量
              threadLocal.set("Thread-specific data");
  
              // 假设这里有一些长时间运行的操作
              try {
                  Thread.sleep(10000); // 模拟长时间运行
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
  
              // 不再需要 ThreadLocal 变量，将其设置为 null
              threadLocal.remove(); // 或者直接设置为 null，但推荐使用 remove() 方法
  
              // 此时，如果 ThreadLocal 使用的是弱引用，并且没有其他强引用指向它，
              // 那么这个 ThreadLocal 就可以被垃圾回收器回收。
              // 对应的 ThreadLocalMap 中的 Entry 也会因为没有 ThreadLocal 的强引用而被清除（在下次清理时）。
  
              // 线程继续运行，但 ThreadLocal 已经被回收，避免了内存泄漏。
          });
  
          thread.start();
      }
  }
  ```
  
  在这个例子中，虽然线程还在运行，但是当我们不再需要 `ThreadLocal` 变量时，通过调用 `remove()` 方法（或者直接将 `ThreadLocal` 设置为 `null`，但推荐使用 `remove()` 以确保 `ThreadLocalMap` 中的 `Entry` 也能被清理）来释放它。由于 `ThreadLocalMap` 使用弱引用来引用 `ThreadLocal`，因此当 `ThreadLocal` 没有其他强引用时，它就可以被垃圾回收器回收，从而避免了内存泄漏。

- **虚引用**
  
  又称为幽灵引用或者幻影引用。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用取得一个对象。
  
  为一个对象设置虚引用关联的唯一目的就是能在这个对象被回收时收到一个系统通知。
  
  使用 PhantomReference 来实现虚引用。
  
  ```java
  Object obj = new Object();
  PhantomReference<Object> pf = new PhantomReference<Object>(obj);
  obj = null;
  ```

## 注解

注解是JDK1.5版本开始引入的一个特性，用于对代码进行说明，可以对包、类、接口、字段、方法参数、局部变量等进行注解。它主要的作用有以下四方面：

- 生成文档，通过代码里标识的元数据生成javadoc文档。
- 编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
- 编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
- 运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例。

注解的常见分类：

- **Java自带的标准注解**，包括`@Override`、`@Deprecated`和`@SuppressWarnings`，分别用于标明重写某个方法、标明某个类或方法过时、标明要忽略的警告，用这些注解标明后编译器就会进行检查。
- **元注解**，元注解是用于定义注解的注解，包括`@Retention`、`@Target`、`@Inherited`、`@Documented`，`@Retention`用于标明注解被保留的阶段，`@Target`用于标明注解使用的范围，`@Inherited`用于标明注解可继承，`@Documented`用于标明是否生成javadoc文档。
- **自定义注解**，可以根据自己的需求定义注解，并可用元注解对自定义注解进行注解。

### 元注解

**@Target**

Target注解的作用是：描述注解的使用范围（即：被修饰的注解可以用在什么地方） 。

```java
public enum ElementType {
    TYPE, // 类、接口、枚举类
    FIELD, // 成员变量（包括：枚举常量）
    METHOD, // 成员方法
    PARAMETER, // 方法参数
    CONSTRUCTOR, // 构造方法
    LOCAL_VARIABLE, // 局部变量
    ANNOTATION_TYPE, // 注解类
    PACKAGE, // 可用于修饰：包
    TYPE_PARAMETER, // 类型参数，JDK 1.8 新增
    TYPE_USE // 使用类型的任何地方，JDK 1.8 新增
}
```

**@Retention**

Reteniton注解的作用是：描述注解保留的时间范围（即：被描述的注解在它所修饰的类中可以被保留到何时） 。

```java
public enum RetentionPolicy {
    SOURCE,    // 源文件保留
    CLASS,       // 编译期保留，默认值
    RUNTIME   // 运行期保留，可通过反射去获取注解信息
}
```

## String

String 被声明为 final，因此它不可被继承。

内部使用 char 数组存储数据，该数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

### 不可变的好处

**1. 可以缓存 hash 值**

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

**2. String Pool 的需要**

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

**3. 安全性**

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

**4. 线程安全**

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

### String,StringBuffer,StringBuilder

**1. 可变性**

- String 不可变
- StringBuffer 和 StringBuilder 可变

**2. 线程安全**

- String 不可变，因此是线程安全的
- StringBuilder **不是线程安全**的
- StringBuffer 是**线程安全**的，内部使用 synchronized 进行同步

### String.intern()

使用 String.intern() 可以保证相同内容的字符串变量引用同一的内存对象。

## Java 的危与机

Java 并不是一个优秀的开发语言，这一点我是非常承认且确定的。但是 Java 有一个庞大的用户群体和异常丰富的生态，这是它的护城河。所以短时间内还倒不下来。

但是大风起于青萍之末。风雨欲来，而包括我在内的很多人都浑然不知。

在文章里面，周佬有这样的一段话：

> Java 支持提前编译最大的困难在于它是一门动态链接的语言，它假设程序的代码空间是开放的（Open World），允许在程序的任何时候通过类加载器去加载新的类，作为程序的一部分运行。要进行提前编译，就必须放弃这部分动态性，假设程序的代码空间是封闭的（Closed World），所有要运行的代码都必须在编译期全部可知。这一点不仅仅影响到了类加载器的正常运作，除了无法再动态加载外，反射（通过反射可以调用在编译期不可知的方法）、动态代理、字节码生成库（如 CGLib）等一切会运行时产生新代码的功能都不再可用，如果将这些基础能力直接抽离掉，Helloworld 还是能跑起来，但 Spring 肯定跑不起来，Hibernate 也跑不起来，大部分的生产力工具都跑不起来，整个 Java 生态中绝大多数上层建筑都会轰然崩塌。
