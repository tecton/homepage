title: Effective Modern C++
date: 2015-09-26 22:51:35
categories:
  - Study
tags:
  - Note
  - C++
  - C++11
---

《Effective Modern C++》 笔记
============================

看过《Effective C++》，C++11有了很多新的特性。

这本书也是Scott Meyers的，可以看成《Effective C++》的新作吧。
对于C++11的使用总结了很多经验。

我觉得比直接看C++11特性更加有意思。

### Item 1
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

###Item 2
auto的类型推断和Item1类似。

只有一点不同！

```
auto x1 = 27;   // type is int
auto x2(27);    // ditto
auto x3 = {37}; // type is std::initializer_list<int>
auto x4{27};    // ditto
```

以下代码会出错```auto x5 = {1, 2, 3.0};```

###Item 3
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

###Item 4
可以用以下代码进行类型测试。

```
template<typename T>
class TD;

TD<decltype(x)> xType;
```

运行时通过type_id输出有时不太准确，boost库有更加准确的判断。

###Item 5
auto有利有弊。

###Item 6
auto的局限：

```
vector<bool> features(const Widget& w);
auto highPriority = features(w)[5];
```

highPriority不是bool， 是vector<bool>::reference。
本来可以隐式转换为bool。auto形成了野指针。
auto遇到隐式类都会有问题，可以用static_cast转换。

###Item 7
只有{}初始化可以到处使用。
还有个优点是防止误声明了函数
```
Widget w();
```
其实没有那个构造函数，就变成了声明函数了= =

如果构造函数有参数initializer_list，那么{}会匹配到这个上面。

###Item 8
原来没有空指针，都是0和NULL的隐式转化。
用nullptr更加语义清晰。

###Item 9
对于模板用alias特别方便。

##Item 10
```
enum class Color { black, white };
```
加class进行范围限定，否则其他地方不能定义black之类变量了。

此外加class之后可以防止类型转换，如```black < 12.5```

尽量前置声明enum，防止重新编译。如：

```
enum class Status;
enum class Status2: std::uint8_t;
```

