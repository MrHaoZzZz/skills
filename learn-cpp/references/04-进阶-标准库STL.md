# C++ 进阶 · STL 标准库与智能指针

STL（Standard Template Library）是 C++ 最强大的武器，由三大件组成：**容器（container）、算法（algorithm）、迭代器（iterator）**，通过模板（下一章）实现通用。

## 1. 常用容器

| 容器 | 底层结构 | 特点 | 适用场景 |
|------|----------|------|----------|
| `std::vector` | 动态数组 | 连续内存，随机访问 O(1)，尾部增删快 | **默认首选** |
| `std::string` | 动态字符数组 | 字符串 | 文本 |
| `std::array` | 固定数组 | 大小编译期确定 | 定长 |
| `std::list` | 双向链表 | 任意位置插入 O(1)，无随机访问 | 频繁中间插入 |
| `std::deque` | 双端队列 | 头尾增删都快 | 队列场景 |
| `std::map` | 红黑树（有序） | 按键有序，O(log n) | 需要有序遍历 |
| `std::unordered_map` | 哈希表 | 平均 O(1) 查找 | 快速查找（无序） |
| `std::set` | 有序集合 | 元素唯一且有序 | 去重+有序 |
| `std::unordered_set` | 哈希集合 | 元素唯一，O(1) | 快速去重 |

### vector（最常用）

```cpp
#include <vector>
std::vector<int> v = {1, 2, 3};

v.push_back(4);          // 尾部添加
v.emplace_back(5);       // 尾部原地构造（推荐，避免临时对象）
int first = v[0];        // 下标访问（不检查越界，快）
int last  = v.back();    // 最后一个
size_t n  = v.size();    // 元素个数
v.reserve(100);          // 预留容量，避免反复扩容

for (int x : v) { ... }  // 范围 for 遍历
```

**性能要点**：`vector` 满了会「扩容」——重新分配更大内存并拷贝全部元素，代价高。用 `reserve` 提前预留，可避免频繁扩容。

### map 与 unordered_map

```cpp
#include <map>
#include <unordered_map>
std::map<std::string, int> m;
m["alice"] = 30;                       // 直接下标（不存在则插入）

std::unordered_map<std::string, int> um;
um["bob"] = 25;

// 查找
auto it = um.find("bob");
if (it != um.end()) {
    std::cout << it->first << "=" << it->second << '\n';
}
```

## 2. 迭代器（iterator）

迭代器是「泛化的指针」，用于遍历容器，是容器与算法之间的桥梁。

```cpp
std::vector<int> v = {1, 2, 3, 4};

// 三种典型用法
for (auto it = v.begin(); it != v.end(); ++it) {  // 显式迭代器
    std::cout << *it << ' ';                      // *it 解引用
}
for (int x : v) { ... }                           // 范围 for（底层就是迭代器）
for (auto it = v.rbegin(); it != v.rend(); ++it) {} // 反向
```

**易错点**：容器增删元素可能使迭代器失效（invalidation），不要在遍历中 `push_back` 到 vector。

## 3. 算法（algorithm）

算法通过迭代器操作容器，与容器类型解耦。

```cpp
#include <algorithm>

std::vector<int> v = {3, 1, 4, 1, 5};

std::sort(v.begin(), v.end());               // 排序
auto it = std::find(v.begin(), v.end(), 4);  // 查找
int cnt = std::count(v.begin(), v.end(), 1); // 计数
std::reverse(v.begin(), v.end());            // 反转

// 带条件的：用 lambda
auto it2 = std::find_if(v.begin(), v.end(),
                        [](int x) { return x > 3; });
```

## 4. Lambda 表达式

匿名函数，语法：`[捕获](参数) -> 返回类型 { 函数体 }`。

```cpp
// [=] 按值捕获外部变量，[&] 按引用捕获
int threshold = 3;
auto big = std::count_if(v.begin(), v.end(),
    [=](int x) { return x > threshold; });
```

## 5. 智能指针（smart pointer）

自动管理堆内存的类，是 RAII 的实践，**取代手动 new/delete**。

```cpp
#include <memory>

// unique_ptr：独占所有权，不可拷贝（可移动），最常用
std::unique_ptr<Person> p = std::make_unique<Person>("Alice", 30);
p->introduce();   // 当指针用

// shared_ptr：共享所有权，引用计数，最后一份释放时销毁
std::shared_ptr<Person> s1 = std::make_shared<Person>("Bob", 25);
std::shared_ptr<Person> s2 = s1;   // 引用计数 +1

// weak_ptr：不增加引用计数，用于打破 shared_ptr 循环引用
std::weak_ptr<Person> w = s1;
```

### 选择规则
1. 默认用 `unique_ptr`（独占，零开销）。
2. 确实需要多处共享时才用 `shared_ptr`（有引用计数开销）。
3. `weak_ptr` 解决循环引用（如双向链表、父子互相引用）。

## 6. 移动语义（move semantics）

C++11 引入，核心是「**转移资源而非拷贝**」，大幅提升性能。

```cpp
std::vector<int> makeBig() {
    std::vector<int> v(1000000);
    return v;            // 现代 C++ 会自动「移动」而非拷贝（返回值优化 RVO）
}

std::vector<int> a = makeBig();   // 不拷贝，直接转移
std::vector<int> b = std::move(a); // 显式移动：a 的内容被转移走，a 变空
```

关键点：
- **左值（lvalue）**：有名字、可取地址的对象（`a`）。
- **右值（rvalue）**：临时对象、即将销毁的值（`makeBig()` 的返回值）。
- **`&&` 右值引用**：绑定到右值，实现移动。
- **`std::move`**：把左值「转成」右值引用，请求移动（本身不移动，只做类型转换）。

```cpp
void sink(std::vector<int>&& v) {  // 右值引用参数
    // 接管 v 的资源
}
```

> 移动语义是「深度学习 C++」的核心课题之一，建议结合 Rule of Five 反复体会。

## 本阶段练习

1. 用 `vector` + `map` 统计一段文本中每个单词出现次数。
2. 用 `std::sort` + lambda 对一个结构体数组按多个字段排序。
3. 写一个返回 `std::unique_ptr<Person>` 的工厂函数，观察所有权转移。

## 掌握度检查清单

- [ ] 知道 vector/map/unordered_map 的底层结构与选用场景
- [ ] 会写迭代器遍历（含反向）和范围 for
- [ ] 会用 sort/find/count/find_if 等常用算法
- [ ] 理解 unique_ptr/shared_ptr/weak_ptr 的区别与选择
- [ ] 能解释左值/右值、`std::move`、移动构造

> 下一步：`references/05-高级-模板与泛型.md`
