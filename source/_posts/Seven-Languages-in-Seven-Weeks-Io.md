title: Seven Languages in Seven Weeks - Io
categories:
  - Study
  - Programming Language
tags:
  - Io
  - Seven Languages in Seven Weeks
  - exercise
date: 2015-03-31 16:20:36
---

七周七语言 —— Io练习
===

[七周七语言-Ruby](http://tecton69.com/2015/03/30/Seven-Languages-in-Seven-Weeks-Ruby/)

Io语言是我第一次听到。它是一门基于原型的编程语言，语法糖很少。据说学完会更加透彻地理解JavaScript？

<!-- more -->

###第一天

`brew install io`

答

```
1. Io 我觉得应该是强类型的。1 + "one"无法执行，没有出错后继续执行的情况发生。
2. 0是true；空串也是true；nil是false；（0在ruby里面也是true）
3. xxx slotNames
4. =是对slot进行更新赋值,如果没有定义则会异常；
   :=是对slot进行定义并赋值；
   ::=是创建slot，设置setter并赋值；
```

做

```
1. io xxx.io
2. object slot
```

###第二天
做(拖了好久，感觉有难度)</br>
2015-4-12 update

```
1.
递归
fib := method(n,
    return (if (n < 3, 1, fib(n-1)+fib(n-2)));
);
非递归
fib2 := method(n,
    if (n == 1 or n == 2, return 1)
    a := 1
    b := 1
    for (i, 3, n,
        c := a + b
        a := b
        b := c
    )
    return b
);
```
```
2.
Number setSlot("originDivide", Number getSlot("/"))
Number / := method(denominator,
    if (denominator == 0, 0, originDivide(denominator))
)
```
```
3.
List sum2D ::= method(
    result := 0
    for (i, 0, size,
        duck := self at(i)
        if (duck type == "List",
            result := result + (duck sum)
        )
    )
    return result
)
```
```
4.
List myAverage ::= method(
    if (self size == 0, return nil)
    result := 0
    for (i, 0, size - 1,
        duck := self at(i)
        if (duck type() == "Number",
            result := result + duck,
            Exception raise("Not a number")
        )
    )
    return result / self size
)
```
```
5.
Matrix2D := Object clone
Matrix2D dim := method(x, y,
    self matrix := List clone
    line := List clone
    for (i, 1, x, line append(0));
    for (i, 1, y,
        self matrix append(line clone)
    );
);
Matrix2D get := method(x, y,
    return self matrix at(y) at(x);
);
Matrix2D set := method(x, y, value,
    self matrix at(y) atPut(x, value);
    return self;
);
```
``` Io Day2 Matrix2D https://gist.github.com/tecton/c2695bcb72e02813c8a0 gist
6.
Matrix2D := Object clone
Matrix2D dim := method(x, y,
    self transposed := false
    self matrix := List clone
    line := List clone
    for (i, 1, x, line append(0));
    for (i, 1, y,
        self matrix append(line clone)
    );
);
Matrix2D get := method(x, y,
    return if (self transposed, self matrix at(x) at(y), self matrix at(y) at(x));
);
Matrix2D set := method(x, y, value,
    if (self transposed, self matrix at(x) atPut(y, value), self matrix at(y) atPut(x, value));
    return self;
);
Matrix2D transpose := method(
    self transposed := self transposed not
)
```
```
7.
matrix := list(list(0, 1), list(2, 3))
f := File with("matrix")
f openForUpdating
f write(matrix serialized)
f close
matrix2 := doFile("matrix")
```
```
8.
n := Random value(100) floor
writeln(n)
for (i, 1, 10,
    writeln("Please input a number (1 - 100)")
    guess := File standardInput readLine asNumber;
    if (guess == n, writeln("Bingo!"); return)
    if (guess > n, writeln("Too big."), writeln("Too small."))
)
writeln("You failed to guess the number...")
```

###第三天
(做完第二天的练习再来看第三天的例子，很容易就看懂了)</br>
做
```
1.
Builder := Object clone
Builder depth := 0
Builder forward := method(
writeln("    " repeated(self depth), "<", call message name, ">")
    self depth := self depth + 1
    call message arguments foreach(
        arg,
        content := self doMessage(arg);
        if (content type == "Sequence", writeln("    " repeated(self depth), content))
    )
    self depth := self depth - 1
    writeln("    " repeated(self depth), "</", call message name, ">")
)
```
```
2.
curlyBrackets := method(
    r := List clone
    call message arguments foreach(arg,
        r append(arg)
    )
    r
)
```
结合了两个例子，输出XML格式很方便。</br>
```Io Day 3 xml attribute builder https://gist.github.com/tecton/2dc3d5501e723a374276 gist
3.
OperatorTable addAssignOperator(":" , "attribute" )

Builder := Object clone

Builder depth := 0

Builder forward := method(
    write("    " repeated(self depth), "<", call message name)
    self depth := self depth + 1
    
    arg0 := self doMessage(call message arguments at(0))
    if (arg0 type == "Map",
        arg0 keys foreach(key,
        write(" ", key, "=\"", arg0 at(key), "\"")
        )
    )
    write(">\n")
        
    call message arguments foreach(
        arg,
        content := self doMessage(arg);
        if (content type == "Sequence", writeln("    " repeated(self depth), content))
    )
    self depth := self depth - 1
    writeln("    " repeated(self depth), "</", call message name, ">")
)

curlyBrackets := method(
    r := Map clone
    call message arguments foreach(arg,
        r doMessage(arg)
    )
    r
)

Map attribute := method(
    self atPut(
        call evalArgAt(0) asMutable removePrefix("\"") removeSuffix("\""),
        call evalArgAt(1)
    )
)

doString("
Builder book({\"author\": \"Tate\", \"reader\": \"tecton\"},
    li({\"status\": \"learning\"}, \"Io\"),
    li(\"Lua\"),
    li(\"Javascript\")
)
")
```