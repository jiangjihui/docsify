# Java 语法语义易错点

> == 的缓存边界、常量池的折叠规则、finally 与 return 的博弈、泛型擦除的延迟爆炸——这些不是八股，是真实生产 bug 的出处。每条都是「现象（实测输出）→ 原理 → 避坑」三层，输出全部来自本机 Java 11 实测。

## 速查表

| 陷阱 | 一句话原理 | 避坑 |
|------|-----------|------|
| Integer == 在 128 处翻车 | IntegerCache 只缓存 -128~127 | 包装类型比较永远用 equals |
| 变量拼接的 String 不入池 | 编译期常量折叠只对字面量/final 生效 | 判断相等用 equals |
| BigDecimal equals 看「精度」 | equals 含 scale 比较 | 数值比较用 compareTo |
| 泛型擦除：写入不炸取值炸 | 类型检查在编译期，运行时只剩桥方法 | 别用原生类型/未经检查强转 |
| 三元运算符隐式拆箱 NPE | 一边基本类型一边包装类型 → 自动拆箱 | 三元两分支保持同类型 |
| 字段没有多态 | 字段按编译期（引用类型）绑定 | 字段访问走 getter |
| new Integer 过时且永不相等 | new 产生新实例绕过缓存 | Java 9+ 用 valueOf（自动装箱本来就走了） |

## 相等性比较

### Integer 缓存：== 在 128 处翻车

**现象**（实测）：

```java
Integer a = 1, b = 1;
System.out.println(a == b);                    // true
Integer c = 128, d = 128;
System.out.println(c == d);                    // false ！
Integer e = new Integer(1), f = new Integer(1);
System.out.println(e == f);                    // false
Integer g = 1; int h = 1;
System.out.println(g == h);                    // true（拆箱后比值）
```

**原理**：自动装箱走 `Integer.valueOf()`，其内部 `IntegerCache` 只缓存 **-128~127**——范围内的装箱值共享同一实例（`==` 为 true），范围外每次新建对象。`new Integer()` 无论传什么都绕过缓存直接造新实例。而包装类型与基本类型比较时，包装侧**自动拆箱**变数值比较，永不为 false（null 时抛 NPE）。

**避坑**：包装类型比较一律 `equals()`；业务上 Integer 做 key/去重时（如 `Map<Integer,...>`、`distinct()`）不受影响——它们内部用 equals。

### String 常量池：拼接方式决定入池与否

**现象**（实测）：

```java
String s1 = "ab";
String s2 = "a" + "b";            // true  ← 编译期常量折叠成 "ab"
System.out.println(s1 == s2);

String a = "a", b = "b";
String s3 = a + b;                 // false ← 运行期 StringBuilder 拼接
System.out.println(s1 == s3);

System.out.println(s1 == s3.intern());      // true  ← intern 归池
System.out.println(s1 == new String("ab")); // false ← new 出堆对象

final String fa = "a", fb = "b";
String s6 = fa + fb;               // true  ← final 是编译期常量, 折叠生效
System.out.println(s1 == s6);
```

**原理**：编译器只对**编译期常量表达式**做折叠（字面量拼接、final 常量拼接），折叠结果直接进 class 文件常量池；普通变量拼接在运行期用 StringBuilder 生成堆对象。`intern()` 把运行期字符串归池（池里有则复用）。

**避坑**：判断字符串相等永远 `equals()`；`==` 只在面经里有趣。顺带记住：循环里 `+=` 拼接每次都 new StringBuilder，量大时用显式 StringBuilder 或 String.join。

### BigDecimal：equals 不等于数值相等

**现象**（实测）：

```java
BigDecimal x = new BigDecimal("1.0");
BigDecimal y = new BigDecimal("1.00");
x.equals(y);                                  // false ！
x.compareTo(y) == 0;                          // true
new HashSet<>(Arrays.asList(x)).contains(y);  // false ！集合也中招
new BigDecimal(0.1);                          // 0.1000000000000000055511151231257827...
```

**原理**：`equals` 同时比较**数值与 scale（小数位数）**，1.0 与 1.00 scale 不同。HashSet/HashMap 内部用 equals，所以 contains 也翻车。而 `new BigDecimal(double)` 会把 double 的二进制误差完整暴露（0.1 在二进制里无限循环）。

**避坑**：数值相等用 `compareTo() == 0`；构造一律用字符串 `new BigDecimal("0.1")` 或 `BigDecimal.valueOf()`；金额场景用 `stripTrailingZeros()` 前先想清楚要不要统一 scale。

## 泛型与类型

### 泛型擦除：写入不炸，取值炸

**现象**（实测）：

```java
List<String> ls = new ArrayList<>();
List<Integer> li = new ArrayList<>();
ls.getClass() == li.getClass();      // true —— 运行时同一个 Class

ls.add("abc");
Object o = ls;
List<Integer> li2 = (List<Integer>) o;   // 未经检查强转, 编译只给 warning
li2.get(0);                            // ClassCastException: String cannot be cast to Integer
```

**原理**：泛型是编译期类型检查，编译后类型参数被擦除（`List<String>` 与 `List<Integer>` 字节码里都是 `List`）。所以写入时检查拦不住「未经检查的强转」绕路——运行时真正的类型错误在**取值赋给具体类型时**才由强转触发。

**避坑**：别写原生类型 `List` 和未经检查强转（有 warning 就当错误处理）；跨类型边界传集合用 `List<?>` + 内部 equals/instanceof；JSON 反序列化的 `Map<String, Object>` 里数字默认可能不是你以为的类型（Integer vs Long 边界 21 亿）。

### 三元运算符的隐式拆箱 NPE

**现象**（实测）：

```java
Boolean cond = null;
Integer r = cond ? 1 : 2;       // NullPointerException ！（cond 拆箱）
Integer r2 = false ? 1 : null; // null（false 分支不触发拆箱, 恰好没事）
Integer r3 = true ? Integer.valueOf(1) : Integer.valueOf(2); // 1（两分支全包装类型, 安全）
```

**原理**：JLS 规定三元表达式的一个分支是基本类型、另一个是包装类型时，**包装侧自动拆箱**以统一类型。`cond` 为 null 的拆箱直接 NPE；第二例的 `null` 赋给 Integer 无拆箱需求。这是 `Map.get` 返回 null + 三元的组合杀，Stack Overflow 上的经典题。

**避坑**：三元两分支保持同一类型（都写包装类型或都写字面量同型）；条件来源可能为 null 时先判空。Kotlin/新风格里用 `Optional.map(...).orElse(...)` 从根上规避。

## 初始化与多态

### 初始化顺序：静态一次，父类先行

**现象**（实测）：

```java
class ClassA {
    static { System.out.println("1. ClassA static"); }
    { System.out.println("2. ClassA instance block"); }
    ClassA() { System.out.println("3. ClassA constructor"); }
}
class ClazzB extends ClassA {
    static { System.out.println("4. ClazzB static"); }
    { System.out.println("5. ClazzB instance block"); }
    ClazzB() { System.out.println("6. ClazzB constructor"); }
}
new ClazzB();  // 第二次再 new
```

```
== 第一次 new ClazzB() ==
1. ClassA static
4. ClazzB static
2. ClassA instance block
3. ClassA constructor
5. ClazzB instance block
6. ClazzB constructor
== 第二次 new ClazzB() ==
2. ClassA instance block
3. ClassA constructor
5. ClazzB instance block
6. ClazzB constructor      ← 静态块不再执行
```

**原理**：类加载阶段执行静态块（父类先于子类，整个 JVM 生命周期只一次）；实例创建时先跑父类实例块+构造器，再子类。实例块在每个构造器前执行（编译器把它插到每个构造器 super() 之后）。

**避坑**：静态块里别依赖其它类的初始化顺序（循环依赖会死锁在类加载上）；实例块与字段初始化按源码顺序执行，声明在实例块之后的字段在块里还是默认值。

### 方法有多态，字段没有

**现象**（实测）：

```java
Instrument i = new Wind();
i.play();                       // Wind is playing...   ← 方法运行期分派
System.out.println(i.name);     // instrument-field     ← 字段按引用类型绑定！
System.out.println(((Wind) i).name); // wind-field      ← 强转后才拿到子类字段
```

**原理**：方法调用按**运行期实际对象**分派（虚方法表）；字段访问按**编译期引用类型**绑定。父子类同名字段是两个独立存储，没有覆盖关系。

**避坑**：字段全部 private 封装、外部访问走 getter——getter 是方法，享受多态；「父类引用读字段拿到父类值」造成的配置覆盖失灵很隐蔽。

## 相关文档

- `==` 的字节码视角（IntegerCache、常量池）在 [集合](Java集合.md) 的 HashMap key 一节有配合场景
- finally 与 return 的博弈见 [异常与资源](Java异常与资源.md)
- JVM 类加载机制系统学习见 [JVM](/contents/java/JVM.md)
