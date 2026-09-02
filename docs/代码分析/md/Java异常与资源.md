# Java 异常与资源易错点

> finally 与 return 的博弈、finally 异常吞噬主异常、close 抛异常的正确处理——异常处理代码平时跑得好好的，出事的那天你才发现异常被吞了，排查无从下手。每条都是「现象（实测输出）→ 原理 → 避坑」三层，输出全部来自本机 Java 11 实测。

## 速查表

| 陷阱 | 一句话原理 | 避坑 |
|------|-----------|------|
| finally 里 return 吞噬一切 | 编译期 finally return 覆盖 try/catch 的 return | finally 里永远别写 return |
| finally 改基本类型无效 | return 时返回值已快照入操作数栈 | 返回值只依赖 return 表达式 |
| finally 改引用生效 | 快照的是引用地址, 不是对象 | 语义易混淆, finally 只做清理 |
| finally 抛异常吞噬主异常 | 后抛的异常顶掉先抛的 | 用 try-with-resources |
| close 异常被静默丢弃 | 老 finally 写法 suppressed=0 | try-with-resources 把它放 getSuppressed() |
| 父类 catch 在前编译错 | 子类分支永不可达 | catch 顺序子类在前 |

## finally 与 return

### 三连案例：一个原理串到底

**原理先行**：JVM 执行 `return x` 时分两步——先把 x 的值（基本类型拷值 / 引用拷地址）**快照**进操作数栈，再跳 finally。所以 finally 里改「局部变量值」改不动已快照的返回值，但顺着「引用地址」改对象内容是生效的；finally 里若再写 return，则整个方法直接用它返回，try/catch 的 return 全部作废。

**现象**（三案例实测）：

```java
// 案例1: finally 里 return —— 吞噬 catch 的 return
public static int watch(int i) {
    int result = 0;
    try {
        result = result / i;          // i=0 抛 ArithmeticException
    } catch (Exception e) {
        result = result + 1;          // result = 1
        return result;                // 被下面覆盖
    } finally {
        result = result + 1;          // result = 2
        return result;                // 实际返回 2
    }
}
// 输出: 2
```

```java
// 案例2: finally 改基本类型 —— 无效
public static int foo(boolean ex) {
    int x;
    try {
        x = 1;
        return x;                     // 1 已快照
    } catch (Exception e) {
        return 2;
    } finally {
        x = 3;                        // 改不动快照
    }
}
// foo(false)=1, foo(true)=2（异常路径）——finally 的 x=3 从未生效
```

```java
// 案例3: finally 改引用指向的对象 —— 生效
public static List<Integer> fdd() {
    List<Integer> x = new ArrayList<>();
    try {
        x.add(1);
        int c = 1 / 0;
        return x;
    } catch (Exception e) {
        x.add(2);
        return x;
    } finally {
        x.add(3);                     // 引用已快照, 但顺着地址改内容生效
    }
}
// 输出: [1, 2, 3]
```

**避坑**：finally 里**永远不写 return、不写 throw**（编译器会对 finally return 给 warning，把它当错误）；finally 只做资源清理和日志。想表达「无论如何都以某值为准」的语义，用 try 表达式外的变量承接，别借 finally。

## 资源关闭

### close 抛异常：finally 吞噬 vs try-with-resources 抑制

**现象**（实测对照，close() 都会抛「关闭失败」）：

```java
// 写法A: 传统 try-finally
BadRes r = new BadRes("流");
try {
    throw new RuntimeException("业务异常");
} finally {
    r.close();      // close 抛新异常
}
// 捕获到: "关闭失败"
// suppressed 数量: 0        ← 业务异常彻底丢失！
```

```java
// 写法B: try-with-resources
try (BadRes r = new BadRes("流")) {
    throw new RuntimeException("业务异常");
}
// 捕获到: "业务异常"          ← 主异常保留
// getSuppressed(): [关闭失败] ← close 异常挂进 suppressed
```

**原理**：finally 里后抛的异常在异常表里**顶替**先抛的主异常（主异常引用直接丢失，suppressed 机制是 Java 7 才有的，老写法享受不到）；try-with-resources 编译生成的字节码会先捕获主异常、再调 close、把 close 的异常 `addSuppressed` 到主异常上。

**避坑**：一切 `AutoCloseable` 资源用 try-with-resources（多资源按声明**逆序**关闭）；排查线上异常时记得看 `getSuppressed()`——真实根因可能藏在被抑制的那个里。

### 多 catch 顺序：父类在前直接编译错

**现象**（javac 实测）：

```java
try {
    throw new RuntimeException("x");
} catch (Exception e) {          // 父类在前, 什么都接住了
} catch (RuntimeException e) {   // 编译错误
}
```

```
错误: 已捕获异常类RuntimeException
        } catch (RuntimeException e) {
          ^    // 已被上面的 catch 捕获
```

**原理**：JLS 要求 catch 子句按可达性排序——父类型在前会让后续子类型分支永不可达，编译期直接拒绝。这是少数「编译器救了你」的场景，反过来也提示：**catch Exception 接住一切**时你失去了分类处理的能力。

**避坑**：catch 从具体到宽泛排列；最宽泛的 `catch (Exception e)` 只放兜底日志与重抛；别在中间层吞异常后只打日志不抛（`catch (e) { log.error(...) }` 然后返回 null——下游拿到 null 更迷惑）。

## 相关文档

- 「迭代器中修改集合」抛 CME 的时机细节见 [集合](Java集合.md) fail-fast 一节
- try-with-resources 与 Spring 事务/连接释放的配合，见 [SpringBoot默认配置的隐藏陷阱](/实践/性能工程/SpringBoot默认配置的隐藏陷阱.md) 的连接池一节
