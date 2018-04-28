# 基础词法单元

## 文件单独编译

## 编译过程

## 字符集

### 基本源码字符集

包含96个字符，主要包含如下几类：
1. 空格符
2. 控制字符(horizontal tab，vertical tab，form feed new-line)
3. 可显示图形字符

a b c d e f g h i j k l m n o p q r s t u v w x y z<br/>
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z<br/>
0 1 2 3 4 5 6 7 8 9<br/>
_ { } [ ] # ( ) < > % : ; . ? * + - / ^ & | ~ ! = , \ " ’<br/>

### Unicode 字符集 (code point)
我们可以通过全局字符名来引用其他字符，引用的格式如下：
```
hex-quad:
   hexadecimal-digit hexadecimal-digit hexadecimal-digit hexadecimal-digit

universal-character-name:
   \u hex-quad
   \U hex-quad hex-quad
```

## 预处理词法单元

## 普通词法单元

## 预处理数字

## 标识符
```
identifier:
    identifier-nondigit
    identifier identifier-nondigit
    identifier digit
    
identifier-nondigit:
    nondigit
    universal-character-name
    
nondigit: one of
    a b c d e f g h i j k l m
    n o p q r s t u v w x y z
    A B C D E F G H I J K L M
    N O P Q R S T U V W X Y Z _

digit: one of
    0 1 2 3 4 5 6 7 8 9

```

标识符是有任意长度的字母字符和数字字符组成。`code point` 遵循`ISO 10646`编码并且在表1的范围的`universal-character-name`可以出现在标识符里面，并且范围在表2
范围的不能出现在标识符的起始位置。标识符字母区分大小写，大小写不一样的字符序列代表不一样的标识符。标识符中所有的字符都是决定标识符最终是否一样的因素。

在表3出现的标识符在一定的上下文中具备特殊的含义，当在文法中引用的时候他们被当成一个整体显式的进行识别，而不使用`identifier`进行推导。除此之外，如果遇到标识符有特殊含义产生歧义的时候，除非有特殊的说明，否则统一都按照`identifier`产生式的方式进行解释。

补充说明，有一些标识符被极语言实现保留，不要在源代码中使用，如果使用了保留的标识符，编译器不需要输出诊断信息。
> 所有以双下划线 `__` 或者单下划线`_`并且紧跟一个大写字母的标识符由极语言编译器保留内部使用

> 所有以下划线`_`开头的标识符由极语言编译器保留, 用作由极语言编译器定义的全局变量

表1 — 允许的字符范围

|   column    |    column   |   column    |    column   |    column   |
|:-----------:|:-----------:|:-----------:|:-----------:|:-----------:|
|     00A8    |    00AA     |    00AD     |     00AF    |  00B2-00B5  |
|  00B7-00BA  |  00BC-00BE  |  00C0-00D6  |  00D8-00F6  |  00F8-00FF  |
|  0100-167F  |  202A-202E  |  180F-1FFF  |             |             |
|  200B-200D  |  1681-180D  |  203F-2040  |    2054     |  2060-206F  |
|  3004-3007  |  3021-302F  |  3031-D7FF  |             |             |
|  F900-FD3D  |  FD40-FDCF  |  FDF0-FE44  |  FE47-FFFD  |             |
| 10000-1FFFD | 20000-2FFFD | 30000-3FFFD | 40000-4FFFD | 50000-5FFFD |
| 60000-6FFFD | 70000-7FFFD | 80000-8FFFD | 90000-9FFFD | A0000-AFFFD |
| B0000-BFFFD | C0000-CFFFD | D0000-DFFFD | E0000-EFFFD |             |

表2 - 标识符首字符不允许出现的范围

|   column    |    column   |   column    |    column   |
|:-----------:|:-----------:|:-----------:|:-----------:|
|  0300-036F  |  1DC0-1DFF  |  20D0-20FF  |  FE20-FE2F  |

表3 - 特殊的意义的标识符

|   column    |    column   |
|:-----------:|:-----------:|
|   override  |    final    |

## 关键字

```
|------------------|------------------|------------------|------------------|------------------|
| alignas          | const_cast       | thread_local     | for              | public           |
| alignof          | continue         | throw            | decltype         | goto             |
| reinterpret_cast | true             | auto             | default          | try              |
| bool             | inline           | return           | break            | do               |
| int              | short            | case             | double           | long             |
| signed           | typename         | catch            | dynamic_cast     | mutable          |
| sizeof           | char             | else             | namespace        | static           |
| unsigned         | char16_t         | enum             | new              | static_assert    |
| using            | char32_t         | explicit         | noexcept         | static_cast      |
| virtual          | class            | export           | nullptr          | struct           |
| void             | while            | operator         | const            | false            |
| private          | template         | constexpr        | float            | protected        |
| this             |                  |                  |                  |                  |
|------------------|------------------|------------------|------------------|------------------|
```

## 字面量

## 代码注释

代码注释分为多行注释和单行注释
### 多行注释
多行注释以子串`/*`开头，以子串`*/`结束，多行注释不支持嵌套。

### 单行注释
单行注释由子串`//`开头，以紧跟的一个换行符结束。如果在注释中出现`form-feed`字符或者`vertical-tab`字符，则用一个空白符代替，在这种情况下不需要输出诊断信息。

### 注意事项
在单行注释中, 注释分隔符 `//`,`/*`和`*/`没有特殊意义，只是代表字符本身的意义，同样，在多行注释中注释分割符`//`和`/*`没有特殊意义，只是代表字符本身的意义。
