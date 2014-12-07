title: Learn Objective-C coding style
date: 2014/11/27
tags:
  - coding
categories:
  - iOS
---

在编程中每门语言都有自己的代码规范利于多人合作编码。

自觉Objective-C代码规范不够好，所以在Github上看了一份不错的[规范指导书](https://github.com/raywenderlich/objective-c-style-guide)。在此准备梳理并记录一下需要注意的地方。（某些已知的就不列了）

<!-- more -->
1. 用2个空格进行缩进。

2. Block避免冒号对齐。
{% code lang:objc %}
// blocks are easily readable
[UIView animateWithDuration:1.0 animations:^{
  // something
} completion:^(BOOL finished) {
  // something
}];
{% endcode %}

3. 名字打全，不用缩写，便于理解。
{% code lang:objc %}
UIButton *settingsButton;
{% endcode %}
常量为首字母大写的骆驼式。

4. 变量之前的属性顺序：storage > atomicity

5. 用点操作成员变量而非set方法。

6. 用NS_ENUM()定义枚举。
{% code lang:objc %}
typedef NS_ENUM(NSInteger, RWTLeftMenuTopItemType) {
  RWTLeftMenuTopItemMain,
  RWTLeftMenuTopItemShows,
  RWTLeftMenuTopItemSchedule
};
{% endcode %}

7. switch时枚举不必加default。
8. 不必特意用== nil判断变量；YES定义为1、别那它直接比较。
	bool变量可以加isXXX到getter方法。
9. 单行if情况下也加大括号。
10. 用CGGeometry的函数获取CGRect的值。
{% code lang:objc %}
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
{% endcode %}

11. 避免在if条件中继续操作，if中多用于返回。
{% code lang:objc %}
- (void)someMethod {
  if (![someOther boolValue]) {
    return;
  }

  //Do something important
}
{% endcode %}

12. 依靠返回值判断error而非error变量。
{% code lang:objc %}
NSError *error;
if (![self trySomethingWithError:&error]) {
  // Handle Error
}
{% endcode %}