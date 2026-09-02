# 源码分析

> 记录开发中容易踩坑的代码语义与行为，每条都是「现象（实测输出）→ 原理 → 避坑」三层结构，帮助理解底层逻辑、避开平时开发中的 BUG。

## 内容导航

- **Java**
  - [集合](md/Java集合.md) —— ArrayList 删除漏删、扩容 1.5 倍、fail-fast 不保证抛、subList 视图、HashMap 树化
  - [语法语义](md/Java语法语义.md) —— Integer 缓存、String 常量池、BigDecimal equals、泛型擦除、三元拆箱 NPE、初始化顺序、字段无多态
  - [异常与资源](md/Java异常与资源.md) —— finally 三连、close 异常吞噬与抑制、catch 顺序
- **SQL**
  - [SQL易错](md/SQL易错.md) —— left join on/where 之别、NULL 三值逻辑、隐式转换索引失效、高危操作清单

## 与知识体系的分工

- 本专题收「**案例与避坑**」：面向踩过/即将踩坑的读者，以实测现象开篇
- `contents/` 下的知识体系收「**系统学习**」：面向需要完整理解的读者（如 [集合](/contents/java/集合.md)、[MySQL](/contents/storage/MySQL.md)）

## 阅读约定

- 代码块标注「实测」的输出全部来自本机 Java 11 编译运行，非手写
- SQL 篇无本机环境，按 MySQL 8.0 语义推写并已在篇首标注
