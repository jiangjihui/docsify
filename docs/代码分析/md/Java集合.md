# Java 集合易错点

> ArrayList 删除漏删、迭代时修改、subList 视图失效、HashMap 树化——集合是日常使用频率最高、也最容易踩坑的 API。每条陷阱都是「现象（实测输出）→ 原理 → 避坑」三层，输出全部来自本机 Java 11 实测。

## 速查表

| 陷阱 | 一句话原理 | 避坑 |
|------|-----------|------|
| 正序遍历 remove 漏删 | 删除后元素前移，`i++` 跳过下一个 | 倒序遍历 / `removeIf` / `iterator.remove` |
| `remove(int)` vs `remove(Object)` | List 里两个重载，传 int 走下标版 | 删元素用 `Integer.valueOf(x)` |
| 扩容是 1.5 倍不是 2 倍 | `grow()` 用 `oldCap + (oldCap >> 1)` | 已知规模用 `new ArrayList<>(n)` |
| 增强 for 中修改不保证抛 CME | modCount 检查只在 `next()` 执行 | 永远别赌「不抛」 |
| subList 是视图不是副本 | 内部共享 elementData 与 modCount | 要副本包一层 `new ArrayList<>()` |
| 不写 hashCode 的 key 丢数据 | 同桶碰撞树化后 equals 能找到，但退化为 O(log n) | key 必须同时重写 equals + hashCode |

## ArrayList

### 正序遍历删除：漏删

**现象**（实测）：

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 2, 3, 4, 5));
for (int i = 0; i < list.size(); i++) {
    if (list.get(i).equals(2)) {
        list.remove(i);
    }
}
System.out.println(list);
```

```
[1, 2, 3, 4, 5]   // 只删掉了一个 2！
```

**原理**：`remove(i)` 后面的元素整体前移一格。推演一遍（初始 `[1,2,2,3,4,5]`）：

| 步骤 | i | 操作 | 列表状态 | 说明 |
|-----|---|------|---------|------|
| 1 | 1 | `get(1)=2`，remove(1) | `[1,2,3,4,5]` | 第二个 2 前移到下标 1 |
| 2 | 2 | `get(2)=3` ≠ 2 | `[1,2,3,4,5]` | **i++ 跳过了前移上来的 2** |

删除本身没错，错的是「删除 + `i++`」的组合让每个被删元素后面的同值元素逃过检查。

**避坑**（四种写法，实测输出均 `[1, 3, 4, 5]`）：

```java
// 1. 倒序遍历：删除不影响未遍历的前段
for (int i = list.size() - 1; i >= 0; i--) {
    if (list.get(i).equals(2)) list.remove(i);
}

// 2. removeIf（首选，JDK 8+）
list.removeIf(x -> x.equals(2));

// 3. iterator.remove（唯一在迭代器层面合法的删除）
for (Iterator<Integer> it = list.iterator(); it.hasNext(); ) {
    if (it.next().equals(2)) it.remove();
}

// 4. 正序 + remove(Object) 必须手动 i-- 补偿
if (list.get(i).equals(2)) { list.remove(Integer.valueOf(2)); i--; }
```

### remove(int) vs remove(Object)

`List` 有两个删除重载：`remove(int index)` 按下标删，`remove(Object o)` 删第一个 equals 匹配的元素。`list.remove(2)` 走的是**下标版**——想删值为 2 的元素必须写 `list.remove(Integer.valueOf(2))` 强制走对象版。对 `List<Integer>` 这是经典事故源。

另外：`remove(Object)` 删除后会把末位槽显式置 null（`elementData[--size] = null`），这是为了让 GC 能回收——集合实现里「删引用必置 null」的范本。

### 扩容：1.5 倍，不是 2 倍

**现象**（反射读 `elementData` 实测，Java 11）：

```
new ArrayList<>() 初始:  size=0,  capacity=0
add 1 个后:              size=1,  capacity=10     // 首次 add 跳到 10
add 10 个后:             size=10, capacity=10
add 11 个后(触发扩容):   size=11, capacity=15     // 10 * 1.5
指定容量5装满:           size=5,  capacity=5
add 第6个后:             size=6,  capacity=7      // 5 * 1.5
```

**原理**：`grow()` 的新容量是 `oldCapacity + (oldCapacity >> 1)`（即 1.5 倍），不是 Vector 时代的 2 倍——1.5 倍在内存占用与复制频率之间取的折中。默认构造的 `elementData` 是共享空数组，首次 add 才分配 `Math.max(10, ...)`。

**避坑**：已知要装 N 个元素时 `new ArrayList<>(N)` 一次到位，省掉 log₁.₅(N/10) 次数组复制；反射示例还会触发 illegal access 警告，生产代码别这么读容量。

## 迭代与修改

### fail-fast：不保证抛，但别赌

**现象**（实测）：

```java
// 案例A：删倒数第二个位置的元素 —— 不抛！
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4));
for (Integer x : list) {
    if (x.equals(3)) list.remove(x);
}
// 结果：正常结束，list=[1, 2, 4]

// 案例B：forEach 里删 —— 抛
list3.forEach(x -> { if (x.equals(3)) list3.remove(x); });
// 结果：ConcurrentModificationException
```

**原理**：`ArrayList.Itr` 的 `next()` 先比对 `modCount != expectedModCount` 才抛 CME。案例 A 里删除发生在 `next()` 之后，下一次循环先走 `hasNext()`——`cursor != size` 判断因 size 已缩一位变成 false，**循环提前正常结束**，next() 的检查根本没机会执行。所以「增强 for 中修改必抛 CME」是错的，正确表述是「**不保证**抛」。

**避坑**：迭代中结构性修改只用 `iterator.remove()`（同步 modCount）或 `removeIf`；CME 没抛出时是**漏处理数据**的静默 bug，比抛异常更难发现。

## subList

### 视图不是副本

**现象**（实测三连）：

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
List<Integer> sub = list.subList(1, 4);   // [2, 3, 4]

sub.add(99);
// sub=[2, 3, 4, 99] 且 list=[1, 2, 3, 4, 99, 5]  ← 视图改，原列表跟着变

list.add(100);                             // 原列表结构修改
System.out.println(sub);                    // ConcurrentModificationException ← 视图失效

// 正确拿副本：
List<Integer> copy = new ArrayList<>(list.subList(1, 4));
```

**原理**：`subList` 返回 `SubList` 内部类视图——共享 `elementData`，`add/remove` 直写原数组；视图记录创建时的 `modCount`，原列表任何结构性修改后视图操作都先比对失败，直接抛 CME 自保。

**避坑**：subList 只用于**临时只读切片**；要独立副本必须 `new ArrayList<>(subList)`；别把 subList 存到字段/缓存里（生命周期超出原列表 неизбежно 出事）。

## HashMap

### 不写 hashCode：数据能存，取退化 + 语义丢失

**现象**（实测）：

```java
static class BadKey {
    int id;
    @Override public boolean equals(Object o) {
        return o instanceof BadKey && ((BadKey) o).id == id;
    }
    // hashCode 没重写
}

Map<BadKey, String> m = new HashMap<>();
m.put(new BadKey(1), "a");
m.get(new BadKey(1));   // 返回 "a"，能取到（equals 生效）
```

对比同桶 vs hash 分散的查询性能（2 万个 key，实测两轮）：

```
bad(同桶,已树化) get 20000 次: 4475 us / 7259 us
good(hash分散)   get 20000 次: 3079 us / 3448 us
倍数: 1.5 ~ 2.1
```

**原理**：没重写 hashCode → 全部 key 落进同一个桶。Java 8 后桶内链表长度 ≥ 8 且数组 ≥ 64 时转红黑树（树化），查询从 O(n) 止损到 O(log n)——但实测显示仍比正常分散慢 1.5~2 倍。树化是**止损机制不是性能方案**。真正的语义丢失在于「equals 相等的两个对象 hash 不等」违反了 hashCode 契约：跨 HashMap 实例、跨 contains 传递时行为不可预期。

**避坑**：自定义 key 必须 equals + hashCode 成对重写（IDE 生成或 Lombok `@EqualsAndHashCode`）；String/Integer 等内置类型直接放心用。

### 扩容与树化阈值（概念清单）

- 默认容量 16，负载因子 0.75：size 超过 `capacity × 0.75` 时 resize 翻倍
- 树化双条件：单桶节点 ≥ 8 **且** 数组长度 ≥ 64（不够先 resize 不树化）
- 树退化：resize 拆桶后节点 ≤ 6 时退回链表

## 相关文档

- 迭代中修改的正确姿势（`iterator.remove`）在 [异常与资源](Java异常与资源.md) 的 try-with-resources 一节有配套场景
- 集合框架的系统学习见知识体系 [集合](/contents/java/集合.md)
