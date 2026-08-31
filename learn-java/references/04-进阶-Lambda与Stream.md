# Java 进阶 · Lambda、函数式接口与 Stream

## 1. Lambda 表达式

Java 8 引入，本质是「匿名函数的简写」。语法：`(参数) -> { 函数体 }`。

```java
// 传统匿名内部类（冗长）
Runnable r1 = new Runnable() {
    @Override
    public void run() { System.out.println("hi"); }
};

// Lambda（简洁）
Runnable r2 = () -> System.out.println("hi");

// 带参数
Comparator<String> byLen = (a, b) -> a.length() - b.length();
```

**前提**：Lambda 只能赋给**函数式接口**——只有一个抽象方法的接口。

## 2. 函数式接口（@FunctionalInterface）

```java
@FunctionalInterface
interface MyFunc {
    int apply(int x);
}

MyFunc square = x -> x * x;
System.out.println(square.apply(5));   // 25
```

### 常用内置函数式接口

| 接口 | 签名 | 用途 |
|------|------|------|
| `Function<T,R>` | `R apply(T t)` | 转换 |
| `Predicate<T>` | `boolean test(T t)` | 判断 |
| `Consumer<T>` | `void accept(T t)` | 消费 |
| `Supplier<T>` | `T get()` | 提供 |
| `UnaryOperator<T>` | `T apply(T t)` | 一元运算 |
| `BiFunction<T,U,R>` | `R apply(T t, U u)` | 二元 |

```java
Function<String, Integer> len = s -> s.length();
Predicate<Integer> isEven = n -> n % 2 == 0;
```

## 3. 方法引用（method reference）

Lambda 的更简写法 `类::方法`。

```java
// 静态方法引用
Function<String, Integer> len = String::length;   // 等价 s -> s.length()

// 实例方法引用
List<String> names = ...;
names.forEach(System.out::println);   // 等价 s -> System.out.println(s)

// 构造方法引用
Supplier<Person> creator = Person::new;
```

## 4. Stream API

Stream 是「数据流上的函数式管道」，做**声明式**的数据处理：过滤、映射、聚合。特点：惰性求值、不改变原集合。

```java
List<String> names = List.of("alice", "bob", "charlie", "dave");

List<String> result = names.stream()          // 创建流
    .filter(n -> n.length() > 3)              // 过滤（中间操作）
    .map(String::toUpperCase)                 // 映射（中间操作）
    .sorted()                                 // 排序（中间操作）
    .toList();                                // 收集（终止操作）

System.out.println(result);   // [ALICE, CHARLIE, DAVE]
```

### 关键概念
- **中间操作**（返回流，惰性）：`filter`、`map`、`sorted`、`distinct`、`limit`、`skip`、`flatMap`。
- **终止操作**（触发执行）：`collect`、`forEach`、`count`、`reduce`、`toList`、`anyMatch`。

### 常用终止操作

```java
long count = stream.count();
boolean any = stream.anyMatch(n -> n > 10);
Optional<String> first = stream.findFirst();

// 聚合 reduce
int sum = IntStream.range(1, 101).reduce(0, (a, b) -> a + b);

// 收集 collect
Map<Integer, List<String>> byLen = names.stream()
    .collect(Collectors.groupingBy(String::length));
```

### 数值流与并行流

```java
IntStream.range(0, 10).sum();               // 数值流
names.parallelStream().filter(...);         // 并行流（注意线程安全）
```

## 5. Optional

解决 `null` 的优雅方式，避免 `NullPointerException`。

```java
Optional<String> opt = Optional.ofNullable(mayBeNull);

String value = opt.orElse("default");              // 空则给默认值
String value2 = opt.orElseGet(() -> compute());    // 惰性默认
opt.ifPresent(v -> System.out.println(v));          // 存在则执行
String value3 = opt.orElseThrow(() -> new IllegalStateException());

// 链式
opt.map(String::toUpperCase).filter(s -> s.length() > 3).ifPresent(System.out::println);
```

**惯用法**：方法可能返回空值时返回 `Optional`，而不是 `null`。

## 6. 现代集合工厂方法

```java
List<String> l = List.of("a", "b", "c");     // 不可变，Java 9+
Map<String, Integer> m = Map.of("a", 1, "b", 2);
```

## 本阶段练习

1. 用 Stream 找出一个整数列表中的偶数、平方、去重、降序排列。
2. 用 `Collectors.groupingBy` 按首字母给单词列表分组。
3. 写一个方法返回 `Optional<String>`，练习链式处理和 `orElse`。

## 掌握度检查清单

- [ ] 会写 Lambda 并理解函数式接口
- [ ] 认识 Function/Predicate/Consumer/Supplier 及方法引用
- [ ] 理解「中间操作惰性、终止操作触发」
- [ ] 会用 filter/map/sorted/collect/reduce
- [ ] 会用 Optional 优雅处理空值

> 下一步：`references/05-高级-并发.md`
