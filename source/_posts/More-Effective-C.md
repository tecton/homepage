title: More Effective C++
date: 2015-09-24 09:45:08
categories:
  - Study
tags:
  - Note
  - C++
---

《More Effective C++》笔记
=========================

读了2-3遍了，每遍都有看不懂的。</br>
看懂易忘的知识如下：

### Item 1
```
char *pc = 0;
char& rc = *pc;
```
行为不确定，很危险。
引用无法改变引用对象，且不能引用NULL。

### Item 2
static_cast是类型转换。</br>
C:</br>
```
int a;
double b = (double)a;
```

C++:</br>
```
double b = static_cast<double>(a);
```

reinterpret_cast是重新按照新的格式解读数据。

### Item 3
数组声明是存基类，结果存子类对象。
偏移计算会有问题。

### Item 4
没有默认构造函数会很麻烦。

```
class A {
public:
    A(int b){}
};
```
这样写了之后没有无参构造函数，会有三个问题：</br>
1. 数组 A a[10];
2. 模板类也无法new数组
3. 虚基类

### Item 6
operator++() 前自增
operator++(int) 后自增

后自增返回const，所以没法连用

```
i++++; // i.operator++(0).operator++(0); (error)
```

### Item 8
* new 分配内存+构造函数
* operator new 仅分配内存
* placement new 仅构造函数

### Item 10
构造函数防止资源泄漏

构造函数异常得自己处理，因为析构函数只析构构造完全的对象。
（析构函数里可以直接delete指针，因为C++确保删除空指针是安全的。）

对于初始化列表中出现异常，用auto_ptr。

### Item 12，13
函数调用，控制器会返回调用者；抛出异常不会。

异常对象总是要被拷贝。
用引用捕获异常。

### Item 21
每个重载的操作符必须有一个参数属于用户自定义类型。

```
const UPInt operator+(int lhs, int rhs);    // error
```

### Item 22
operator +调用operator +=进行返回。

### Item 25
构造函数里面也可以有虚函数行为！
```
vector<Base*> objs;
objs.push_back(new Derived());
```

流对象
```
virtual ostream& print(ostream& s, const NLComponent& c);
// 重载符都是将对象放在右边，()除外
```

### Item 26
被问过。
限制类的对象。

* 隐藏构造函数，把static对象放在方法里，通过方法返回。（别内联）
* 在构造和析构函数中进行计数

### Item 27
#####对象在堆上
```
class A() {
public:
    void destroy() { delete this; }
private:
    ~A() {}
};
```

#####判断对象在堆上
> 重载operator new来跟踪对象是不是在堆上。
这样是不行的，数组就会有问题。

> 根据内存地址高低来判断
这样也是有问题的，静态变量在堆的更低内存地址。

> 重载operator new，把获得的内存地址存入链表。
可行。

#####禁止创建在堆上
私有化operator new, operator delete, operator new[], operator delete[]

有一点需要注意，这些是static的。
[SO上的回答]
(http://stackoverflow.com/questions/2273187/why-is-the-delete-operator-required-to-be-static)

### Item 28
智能指针赋值会使原来的失效！

智能指针比较复杂，没有细看，等之后看C++11的内容。

### Item 30
区分读和写

```
const CharProxy operator[](int index) const;
CharProxy operator[](int index);
```

### Item 34
extern "C"

在C++源文件中函数声明前加的话可以使C++不会进行名字改编——能够链接到C编译的obj文件。


指定函数是定义在C的obj里面的：

```
extern "C" void foo(const char *);
//或者
extern "C" {
    int B(char *);
}
```

指定函数能够被C程序调用：

```
extern "C" double calc(double param) {
}
```

C没有重载。