title: Effective Modern C++
date: 2015-09-26 22:51:35
categories:
  - Study
tags:
  - Note
  - C++
  - C++11
---

《Effective Modern C++》 笔记（上)
============================

看过《Effective C++》，C++11有了很多新的特性。

这本书也是Scott Meyers的，可以看成《Effective C++》的新作吧。
对于C++11的使用总结了很多经验。

我觉得比直接看C++11特性更加有意思。

<!-- more -->

## Item 1
类型推断。

```
template<typename T>
void f(ParamType param);
f(expr);
```

T和ParamType都是根据expr推断的，但是有所不同。
ParamType有修饰，如const，reference。

#####ParamType是指针或引用

```
template<typename T>
void f(T& param);
f(x);
```

x是int：              T是int</br>
x是const int(&)：     T是const int

```
void f(const T& param);
```

上述情况T都是int。

#####ParamType是广义引用（Universal Reference）

```
template<typename T>
void f(T&& param);
f(x);
```

x是int：  T是int&</br>
x是const int(&)：    T是const int&</br>
x是27:   因为27是右值，所以T是int

#####ParamType非指针非引用

```
template<typename T>
void f(T param);
```

忽略引用和const、volatile修饰。</br>
x是int或const int(&):     T都是int</br>
x是const char * const：   T是const char *，因为传值只会去除指针的常量修饰。

#####数组的情况
```
template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) {
    return N;
}
```

数组如果是引用可以将T推断为数组参数，利用它可以通过编译期获得数组长度。

##Item 2
auto的类型推断和Item1类似。

只有一点不同！

```
auto x1 = 27;   // type is int
auto x2(27);    // ditto
auto x3 = {37}; // type is std::initializer_list<int>
auto x4{27};    // ditto
```

以下代码会出错

```
auto x5 = {1, 2, 3.0};
```

##Item 3
返回值auto会使用模板类型推断，但是引用会被消去，返回的的都是右值，如果是数组元素之类的左值就会出现问题，因此可以加上decltype。

decltype会保持变量的原有修饰。

C++14

```
template<typename Container, typename Index>
decltype(auto)
authAndAccess(Container&& c, Index i) {
    ...
    return std::forward<Container>(c)[i];
}
```

C++11

```
template<typename Container, typename Index>
decltype(auto)
authAndAccess(Container&& c, Index i)
-> decltype(std::forward<Container>(c)[i]) {
    ...
    return std::forward<Container>(c)[i];
}
```

使用通用引用会进行右值或者左值的判断。

------
decltype((x)) 变量加上括号会变成引用（左值）。

##Item 4
可以用以下代码进行类型测试。

```
template<typename T>
class TD;

TD<decltype(x)> xType;
```

运行时通过type_id输出有时不太准确，boost库有更加准确的判断。

##Item 5
auto有利有弊。

##Item 6
auto的局限：

```
vector<bool> features(const Widget& w);
auto highPriority = features(w)[5];
```

highPriority不是bool， 是vector<bool>::reference。
本来可以隐式转换为bool。auto形成了野指针。
auto遇到隐式类都会有问题，可以用static_cast转换。

##Item 7
只有{}初始化可以到处使用。
还有个优点是防止误声明了函数
```
Widget w();
```
其实没有那个构造函数，就变成了声明函数了= =

如果构造函数有参数initializer_list，那么{}会匹配到这个上面。

##Item 8
原来没有空指针，都是0和NULL的隐式转化。
用nullptr更加语义清晰。

##Item 9
对于模板用alias特别方便。

##Item 10

```
enum class Color { black, white };
```
加class进行范围限定，否则其他地方不能定义black之类变量了。

此外加class之后可以防止类型转换，如
{% codeblock %}
black < 12.5
{% endcodeblock %}

尽量前置声明enum，防止重新编译。如：

```
enum class Status;
enum class Status2: std::uint8_t;
```

##Item 11
使用 = delete来确保函数不会被调用。
如拷贝构造函数。

以前使用private声明来保护，而friend类仍然可以访问它。

此外可以防止某些函数重载。

##Item 12
函数重载要点：

* 基类中是virtual
* 名字相同
* 参数相同
* const相同
* 返回值和异常兼容

C++11中新增的：

* 引用类型相同

为了方便编译器检查，用**override**。

##Item 13
其实我不懂为何C++98里面向容器插入会有问题。

在非容器序列上使用begin，end等。

```
using std::cbegin;
using std::cend;
auto it = std::find(cbegin(container), cend(container), value);
container.insert(it, newValue);
```

C++11中cbegin还没法用。替代方案:

```
template<class C>
auto cbegin(const C& container)->decltype(std::begin(container)) {
    return std::begin(container);
}
```

利用begin作用在const对象上返回const_iterator的特性。

##Item 14
noexcept会舍弃运行时出错的栈回退资源维护，所以效率有提升。

* noexcept is particularly valuable for the move operations, swap, memorydeallocation functions, and destructors.* Most functions are exception-neutral rather than noexcept.

##Item 15
constexpr 可以表示变量编译期间是常量，这样的话可以派很多用处了！
比如数组大小。

```
constexpr int constSize() {
    return 10;
}
int a[constSize];
```

##Item 16
const 函数需要保证线程安全。
如果是单个变量的话可以用atomic，效率更高。
而如果涉及多个变量，需用lock_guard保护起来。

##Item 17
除了默认的构造函数，析构函数，赋值函数，复制构造函数，C++11加了2个：

* 移动构造函数
* 移动赋值

```
class Widget {
public:
    Widget(Widget&& rhs);
    Widget& operator=(Widget&& rhs);
};
```

不同于复制和赋值，移动的两个函数只要定义了一个，另一个就不会隐式添加了。
有了拷贝函数，move函数也不会被编译器添加；反之亦然。

##Item 18
使用unique_ptr代替auto_ptr。
待补充

##Item 19
shared_ptr内存大小是两倍，因为有两个指针。

自定义的deleter与unique_ptr有区别：

```
unique_ptr<Widget, decltype(loggingDel)> upw(new Widget, loggingDel);
shared_ptr<Widget> spw(new Widget, loggingDel);
```

![内存布局](http://7sbnmq.com1.z0.glb.clouddn.com/shared_ptr-memory.png)

##Item 20
用shared_ptr来测试weak_ptr是不是为空了。

```
std::shared_ptr<Widget> spw1 = wpw.lock();
```

用途在缓存系统判断第一次载入的场景。

##Item 21
new 在多线程的情况下会存在顺序问题，导致资源泄漏。
要用:

```
processWidget(std::make_shared<Widget>(), computePriority());
```

A special feature of std::make\_shared (compared to direct use of new) is improved efficiency. Using std::make\_shared allows compilers to generate smaller, faster code that employs leaner data structures.

有2次分配内存变为了一次，所以尽量避免直接new而是用make_*

例外：

* 对unique_ptr来说，初始化时用new可以用{ }，而make_unique只会调用()。
* 以及有自定义的deleter时只能用new。

对shared_ptr来说，还有两个情况：

控制块中记录了引用次数，还有多少个weak_ptr指向它(weak count)。

* 用make生成的会等到weak_ptr也释放了才一起释放资源，而new出来的会立马释放。
根源是等control block释放，而weak_ptr在control block中有记录，所以才会在较大内存占用的对象有区别。
* 使用以下方式来进行异常安全的操作：

```
std::shared_ptr<Widget> spw(new Widget, cusDel);
processWidget(std::move(spw), // both efficient and computePriority()); // exception safe
```

原因是move不会增加计数。

##Item 22
Pimpl思想，减少编译时间。
可以使用unique_ptr(基本就是替换raw pointer的选择）。

需要定义一个析构函数，否则编译器会因为无法释放指针而报错。又由于有了析构函数编译器就不会生成move操作了，所以还要定义move函数。

```
class Widget {public:  Widget();  ~Widget();  Widget(Widget&& rhs);  Widget& operator=(Widget&& rhs);...private:  struct Impl;  std::unique_ptr<Impl> pImpl;};#include <string>...struct Widget::Impl { ... };Widget::Widget(): pImpl(std::make_unique<Impl>()){}Widget::~Widget() = default;Widget::Widget(Widget&& rhs) = default;Widget& Widget::operator=(Widget&& rhs) = default;
```

也可以用shared_ptr，可以省去上述步骤。