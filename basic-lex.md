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

Table 2 — 允许的字符范围

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

## 关键字

## 字面量

## 代码注释

代码注释分为多行注释和单行注释
### 多行注释
多行注释以子串`/*`开头，以子串`*/`结束，多行注释不支持嵌套。

### 单行注释
单行注释由子串`//`开头，以紧跟的一个换行符结束。如果在注释中出现`form-feed`字符或者`vertical-tab`字符，则用一个空白符代替，在这种情况下不需要输出诊断信息。

### 注意事项
在单行注释中, 注释分隔符 `//`,`/*`和`*/`没有特殊意义，只是代表字符本身的意义，同样，在多行注释中注释分割符`//`和`/*`没有特殊意义，只是代表字符本身的意义。
