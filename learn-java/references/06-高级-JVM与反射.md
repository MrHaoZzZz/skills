# Java 高级 · JVM、垃圾回收与反射

理解 JVM 是 Java 工程师的「内功」，也是面试高频考点。

## 1. JVM 内存模型（运行时数据区）

| 区域 | 内容 | 线程 | 生命周期 |
|------|------|------|----------|
| **堆（Heap）** | 所有对象、数组 | 共享 | GC 管理 |
| **栈（Stack）** | 局部变量、方法调用帧 | 私有 | 方法结束释放 |
| **程序计数器** | 当前执行指令地址 | 私有 | 线程结束 |
| **方法区/元空间** | 类元信息、常量、静态变量 | 共享 | 类加载后 |

```java
Person p = new Person();   // p 在栈上（引用），Person 对象在堆上
```

## 2. 垃圾回收（Garbage Collection, GC）

Java 自动回收不再使用的堆内存，程序员**无需手动释放**（与 C++ 的 `delete` 对比）。

### 对象存活判断
**可达性分析**：从 GC Roots（栈引用、静态变量等）出发，不可达的对象判定为垃圾。

### 分代回收

| 代 | 特点 | 回收 |
|----|------|------|
| 新生代（Young） | 新对象，大多朝生夕死 | Minor GC，频繁 |
| 老年代（Old） | 存活久的大对象 | Major/Full GC，慢 |

### 常见 GC 算法
- 标记-清除（Mark-Sweep）：有碎片。
- 复制（Copying）：新生代常用。
- 标记-整理（Mark-Compact）：老年代常用。

### 常见垃圾回收器
G1（默认主流）、ZGC（低延迟）、Parallel、CMS（已废弃）。

### 内存泄漏（memory leak）在 Java 中
Java 有 GC，但仍会泄漏——**对象被无意识强引用持有，无法回收**。典型：静态集合不断 `add`、未关闭的资源、监听器未注销。

## 3. 类加载（Class Loading）

类生命周期：加载 → 连接（验证/准备/解析）→ 初始化 → 使用 → 卸载。

```java
// 类加载器层级
ClassLoader loader = ClassName.class.getClassLoader();
```

**双亲委派**：加载请求先交给父加载器，父能加载则父加载，避免核心类被篡改。

## 4. 反射（Reflection）

在运行时检查/操作类、方法、字段——**框架（Spring 等）的基石**。

```java
import java.lang.reflect.*;

Class<?> clazz = Person.class;              // 获取 Class 对象
Field[] fields = clazz.getDeclaredFields(); // 所有字段
Method m = clazz.getMethod("getName");      // 获取方法
Object result = m.invoke(personInstance);   // 调用方法

// 访问私有成员
Field f = clazz.getDeclaredField("age");
f.setAccessible(true);                       // 突破访问控制
f.set(personInstance, 30);
```

**代价**：反射慢、绕过了类型检查和封装，**仅在有明确需求时使用**（框架、工具类），业务代码少用。

## 5. 注解（Annotation）

元数据，编译期或运行时处理。

```java
// 自定义注解
@Retention(RetentionPolicy.RUNTIME)   // 运行时可见
@Target(ElementType.METHOD)           // 用于方法
public @interface MyAnnotation {
    String value() default "";
}

// 使用 + 反射读取
@MyAnnotation("hello")
public void foo() {}

Method m = clazz.getMethod("foo");
MyAnnotation ann = m.getAnnotation(MyAnnotation.class);
```

## 6. 常见 OOM 与调优

```bash
# 设置堆大小
java -Xms512m -Xmx2g MyApp

# 打印 GC 日志
java -Xlog:gc MyApp

# 查看默认 GC
java -XX:+PrintCommandLineFlags -version
```

**常见异常**：
- `OutOfMemoryError: Java heap space`（堆不够）
- `StackOverflowError`（递归过深）

## 本阶段练习

1. 写一个会「内存泄漏」的程序（静态集合不断 add），分析原因。
2. 用反射遍历一个类的所有方法并打印签名。
3. 自定义一个注解 + 用反射读取它。

## 掌握度检查清单

- [ ] 能画出 JVM 内存区域，说出堆和栈存什么
- [ ] 理解可达性分析、分代回收、常见 GC 算法
- [ ] 能解释「Java 有 GC 为什么还会内存泄漏」
- [ ] 会用反射获取类/方法/字段并调用
- [ ] 会自定义注解并理解 `@Retention`/`@Target`

> 下一步：`references/07-工程实践-构建调试.md`
