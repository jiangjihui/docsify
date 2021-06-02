# 易错代码分析



## 集合

### ArrayList

```java
public static void main(String[] args) {
    ArrayList<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    list.add(2);
    list.add(3);
    list.add(4);
    list.add(5);
    for (int i = 0; i < list.size(); i++) {
        if (list.get(i).equals(2)) {
            list.remove(i);
        }
    }
    System.out.println(list);
}
```

知识点：

`remove()`方法也两个版本，一个是`remove(int index)`删除指定位置的元素，另一个是`remove(Object o)`删除第一个满足`o.equals(elementData[index])`的元素。删除操作是`add()`操作的逆过程，需要将删除点之后的元素向前移动一个位置。需要注意的是为了让GC起作用，必须显式的为最后一个位置赋`null`值。

输出结果：

```
[1, 2, 3, 4, 5]
```



## 代码块

```java
public class ClassA {
    public ClassA() {
        System.out.println("Hello Class A");
    }

    { System.out.println("I`am Class A"); }

    static {
        System.out.println("Class A");
    }

    static class ClazzB extends ClassA {
        public ClazzB() {
            System.out.println("Hello Class B");
        }

        { System.out.println("I`am Class B"); }

        static {
            System.out.println("Class B");
        }

        public static void main(String[] args) {
            new ClazzB();
        }
    }
}
```

知识点：

**构造块**：用`{}`裹起来的代码片段,构造块在创建对象时会被调用,每次创建对象时都会被调用,并且优先于类构造函数执行。

输出结果：

```
Class A
Class B
I`am Class A
Hello Class A
I`am Class B
Hello Class B
```



## finally

```java
public class FinallyTest {
    public static void main(String[] args) {
        System.out.println(watch(0));
    }

    public static int watch(int i) {
        int result = 0;
        try {
            result = result / i;
        } catch (Exception e) {
            result = result + 1;
            return result;
        }
        finally {
            result = result + 1;
            return result;
        }
    }
}
```

finally里面如果有return的话，将直接使用finally的return返回。

输出结果：

```
2
```





