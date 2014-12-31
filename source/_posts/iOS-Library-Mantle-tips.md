title: iOS Library Mantle tips
date: 2014-12-31 16:50:05
categories:
  - iOS
tags:
  - tips
---

iOS开发 Mantle库的使用心得
===

在iOS开发中，[Mantle库](https://github.com/Mantle/Mantle)可以很方便地将Objective-C中的模型与JSON格式相互转换。

这样就使得应用程序与服务器间的数据传输变得更加方便了。

以下记录一下我在使用过程中不断学习到的小技巧及心得，基本来自StackOverflow以及官方的README。
<!-- more -->

1. 在JSONKeyPathsByPropertyKey的方法中，我们需要返回变量与JSON中键的名字的匹配。</br>**如果这两个名字是大小写相同的话就可以省略**这样能省去大量重复性工作。

2. 对于一些辅助的变量，不希望它与JSON之间进行匹配可以指定为NSNULL.null.
{% code lang:objc %}
@"useless_variable" : NSNull.null
{% endcode %}

3. 一些变量可能并没有被赋值，这样在转换到JSON后可以移除这些空的变量。通过getter函数中进行判断，当然也可以移除一些特定的键。
{% code lang:objc %}
- (NSDictionary *)dictionaryValue {
    NSMutableDictionary *modifiedDictionaryValue = [[super dictionaryValue] mutableCopy];
    
    for (NSString *originalKey in [super dictionaryValue]) {
        if ([self valueForKey:originalKey] == nil) {
            [modifiedDictionaryValue removeObjectForKey:originalKey];
        }
    }
    
    return [modifiedDictionaryValue copy];
}
{% endcode %}

4. 对于模型之间的嵌套情况，也可以方便地解决。</br>
比如有个模型是班级，班级下面有很多学生。
那么学生就会以数组的形式存在于班级之中。
因此模型会像下面这样。
{% code lang:objc %}
...
@property (strong, nonatomic) NSArray *students;
{% endcode %}
那么NSArray其实并没有指定数组元素是什么类型的，这个会在Transformer里面实现。这样就可以完美地解决这个问题了。
{% code lang:objc %}
+ (NSValueTransformer *)studentsJSONTransformer {
    return [NSValueTransformer mtl_JSONArrayTransformerWithModelClass:Student.class];
}
{% endcode %}