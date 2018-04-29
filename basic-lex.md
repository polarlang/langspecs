# 基础词法单元

## 文件单独编译

在本文档中，程序文本被写在多个源文件中。
### 转换单元 (translation unit)

一个特定的源文件及其文件里面通过`import`指令进行加载的其他源文件(*Note:* 排除那些条件`import`且条件不满足的源文件)组成的一个整体叫做转换单元。

> *Note:* 转换单元 (translation unit) 和 实例化单元 (instantiation units) 可以分开保存或者在库里面 (libraries)。不同的转换单元 (translation unit) 通过使用具有导出属性的函数标识符或者使用具有导出属性的对象标识符或者操作数据文件
> 而产生相互联系。不同的转换单元可以被单独编译，然后再被连接器连接产生一个可执行的程序。

## 编译过程

转换过程中的各种语法规则的顺序由以下转换步骤进行指定：

1. 如果必要的话，物理源代码文件的内容字符被极语言编译器转换成基本源码字符集 (basic source character set) [*Note: 比如将行结束指示符编程行结束字符*]。物理源文件的字符集接受范围由极语言编译器的支持决定。对于物理源文件中出现的那些不在基本源码字符集的字符，我们使用等价的`universal-character-name`进行替换。
编译器可以使用任何自定义的内部编码，只要在源代码文件中遇见一个扩展字符我们就使用相应的`universal-charactername`进行描述(例如：使用`\uXXXX`记号)，除了那些出现在原生字符串字面量里面的扩展字符 [todo:有待明确]。
2. 删除所有`\`后面直接跟随一个换行符的字符序列，将所有的行拼接成一个逻辑行。物理源文件中只有最后的反斜杠才会在拼接的结果中保留。除了在原生字符串字面量中的，所有其他的如果拼接都到了一个`universal-character-name`的表示，这个时候编译器的行为是未定义。如果物理源文件为空，并且不以换行符结束，或者在拼接的地方以换行符结束，但是在换行符前面有一个`\`字符，那么编译器在处理的时候需要在文件末尾自动添加一个换行符。
3. 拼接后的源代码文本内容被分解成一个一个的预处理的词法单元 (preprocessing tokens)和空白字符序列 [*Note: 包括代码注释*]。源代码文本内容不能以不完整的预处理词法单元或者不完整的注释结束。每一个注释块都被替换成单个的空白字符。换行字符被保留。除了换行符号之外的非空空白字符序列被替换成单个的空白字符。
4. 预处理指令被执行，宏调用被展开，一元运算符`pragma`表达式被执行。如果一个预处理串联产生了一个符合`universal-character-name`语法的字符序列，编译器行为是未定义的 (todo: 是否可以确定，比如直接报错)。由`import`指令加载的源文件递归的应用步骤1到步骤4，然后删除所有的预处理指令。
5. 所有在字符字面量和字符串字面量里面的所有源代码字符集对应的字符和在字符字面量和非原生(noraw)字符串字面量里面的转义序列或者`universal-character-name`转换成执行字符集里面对应的字符，如果对应的执行字符不存在，就转换成由编译器自己定义的非`null`字符。
6. 相邻的字符串字面量的词法单元全部被串连在一起。
7. 用于分隔词法单元（token）的空白字符已经没有作用了。所有的预处理的词法单元转换成了普通的词法单元。得到的词法单元序列会被当做一个转换单元进行相应的语法分析和语义分析。[*Note: 在进行分析的过程中中可能会出现一个词法单元被等价的多个词法单元代替的情况*] [*Note 基本源代码字符集源码文本，转换单元，已经完成的转换单元不需要一定存储在文件中，也不一定要跟物理的源代码文件有一对一的关系*]
8. 已经完成转换的转换单元与实例化单元按照如下的对着进行合并。[*Note: 某些或者全部的词法单元可能来自库文件*]对所有的已完成转换的转换单元进行分析，产生一个需要模板实例化的列表。[*Note：这个列表中包含哪些显式需要实例化的词法单元*]，依赖的模板定义被定位，是否需要在转换单元的源代码中包含这些定义有极语言编译器决定。[*Note: 编译器可以提前把依赖的模板定义转换成已转换的转换单元，那么这里就可以不需要出现在源代码里面*] 执行所有的模板实例化产生模板实例化单元。[*Note: 实例化单元跟转换单元类似，但是不包含模板实例化请求单元和模板定义单元*] 如果任何模板实例化过程失败，那么程序是不规范的。
9. 所有的外部的实体引用都已解决 (resolved)，当前转换单元的所有外部符号引用都通过链接的库进行解决 (resolved)，所有的转换单元被输出到程序映像文件中，这个文件可以直接在执行环境中执行。


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

`ISO/IEC 10646`中的字符`NNNNNNNN`的`universal-character-name`表示形式是`\UNNNNNNNN`。`ISO/IEC 10646`中的字符`0000NNNN`的`universal-character-name`表示形式是`\uNNNN`。如果一个`universal-character-name`字符的16进制表示对应的是一个代理码点 (`surrogate code point`)，在闭区间`0xD800–0xDFFF`中的码点(code point)，那么当前的程序语法不符合规范。额外的如果一个`universal-character-name`的16进制的值在`c-char-sequence`, `s-char-sequence`和`r-char-sequence`表示的字符之外或者不是控制字符的字符串字面量(在闭区间`0x00–0x1F`或者闭区间`0x7F–0x9F`的范围之内)或者不对应一个基本源码字符集的字符，那么程序语法是不符合规范的。

基本运行字符集（basic execution character set）包含所有基本源代码字符集的成员，外加如下的控制字符集合：
1. 告警字符（alert）
2. 退格字符（backspace）
3. 回车字符（carriage return）
4. 空字符（null）值为0

在基本运行字符集和基本源代码字符集中那些在0字符之后的字符，排在后面的字符的数值比排在前面的字符数值要大。极语言的编译器支持的运行字符集可以是基本运行字符集合的超集。运行字符集的成员已经增加的字符成员是本地化敏感的字符。（locale-specific）

## 预处理词法单元

待定，可能不支持通用的预处理

## 普通词法单元

产生式如下：
```
token:
    identifier
    keyword
    literal
    operator
    punctuator
```
极语言有五种词法单元类型：
1. 标识符 (identifiers)
2. 关键字 (keywords)
3. 字面量 (literals)
4. 运算符 (operators)
5. 其他分隔符

其中空白 (Blanks)，水平制表符 (horizontal tab)，垂直制表符 (vertical tabs)，换行符 (newlines)，formfeeds 和代码注释都会被忽略，除非他们用来分割词法单元。

> *Note:* 有的空白符被用来分隔相邻的标识符，关键字，数字字面量和其他包含字母表字符的词法单元

## 预处理数字

待定,可能不支持预处理

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

表1 - 允许的字符范围

```
|-------------|-------------|-------------|-------------|-------------|
|     00A8    |    00AA     |    00AD     |     00AF    |  00B2-00B5  |
|  00B7-00BA  |  00BC-00BE  |  00C0-00D6  |  00D8-00F6  |  00F8-00FF  |
|  0100-167F  |  202A-202E  |  180F-1FFF  |             |             |
|  200B-200D  |  1681-180D  |  203F-2040  |    2054     |  2060-206F  |
|  3004-3007  |  3021-302F  |  3031-D7FF  |             |             |
|  F900-FD3D  |  FD40-FDCF  |  FDF0-FE44  |  FE47-FFFD  |             |
| 10000-1FFFD | 20000-2FFFD | 30000-3FFFD | 40000-4FFFD | 50000-5FFFD |
| 60000-6FFFD | 70000-7FFFD | 80000-8FFFD | 90000-9FFFD | A0000-AFFFD |
| B0000-BFFFD | C0000-CFFFD | D0000-DFFFD | E0000-EFFFD |             |
|-------------|-------------|-------------|-------------|-------------|
```

表2 - 标识符首字符不允许出现的范围

```
|-------------|-------------|-------------|-------------|
|  0300-036F  |  1DC0-1DFF  |  20D0-20FF  |  FE20-FE2F  |
|-------------|-------------|-------------|-------------|
```

表3 - 特殊的意义的标识符

```
|-------------|-------------|
|   override  |    final    |
|-------------|-------------|
```

## 关键字

在表4跟表5中的标识符被极语言编译器保留，将被无条件识别成语言的关键字，除了出现在`attribute-token`。

表4 极语言关键字表
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
| this             | final            | override         | if               | import           |
| polar            | port             |                  |                  |                  |
|------------------|------------------|------------------|------------------|------------------|
```

表5 极语言保留字，暂时识别成关键字，但是目前不使用，可能将来会被启用

```
|------------------|------------------|------------------|------------------|------------------|
| asm              | friend           | register         | typeid           | union            |
| extern           | concept          | delete           | wchar_t          | include          |
| require          |                  |                  |                  |                  |
|------------------|------------------|------------------|------------------|------------------|
```
## 运算符与标点符号 (Operators and punctuators)

在极语言词法表示里面有很多预处理词法单元，他们会在预处理阶段使用或者转换成运算符词法单元和标点符号词法单元。

产生式如下：
```
preprocessing-op-or-punc:
    { } [ ] # ## ( )
    ; : ... new ? :: . .* ->
     ->* ~ ! + - * / % ^ & |
    = += -= *= /= %= ^= &= |=
    == != < > <= >= <=> && ||
    << >> <<= >>= ++ -- ,
```
所有的`preprocessing-op-or-punc`在编译过程步骤7转换成单个的词法单元(token)

## 字面量

### 字面量种类

极语言支持如下几种字面量：

```
literal:	integer-literal	character-literal	floating-literal	string-literal	boolean-literal	pointer-literal	user-defined-literal
```

### 整形字面量
<pre>
integer-literal:	binary-literal integer-suffix<sub>opt</sub>	octal-literal integer-suffix<sub>opt</sub>	decimal-literal integer-suffix<sub>opt</sub>	hexadecimal-literal integer-suffix<sub>opt</sub>
	
binary-literal:	0b binary-digit	0B binary-digit	binary-literal ’<sub>opt</sub> binary-digit
	
octal-literal:	0	octal-literal ’<sub>opt</sub> octal-digit
	
decimal-literal:	nonzero-digit	decimal-literal ’<sub>opt</sub> digit
	
hexadecimal-literal:	hexadecimal-prefix hexadecimal-digit-sequence

binary-digit: one of	0 1
	octal-digit: one of	0 1 2 3 4 5 6 7
	nonzero-digit: one of	1 2 3 4 5 6 7 8 9
	hexadecimal-prefix: one of	0x 0X
	hexadecimal-digit-sequence:	hexadecimal-digit	hexadecimal-digit-sequence ’<sub>opt</sub> hexadecimal-digit
	
hexadecimal-digit: one of	0 1 2 3 4 5 6 7 8 9	a b c d e f	A B C D E F
	
integer-suffix:	unsigned-suffix long-suffix<sub>opt</sub>	unsigned-suffix long-long-suffix<sub>opt</sub>	long-suffix unsigned-suffix<sub>opt</sub>	long-long-suffix unsigned-suffix<sub>opt</sub>

unsigned-suffix: one of	u U

long-suffix: one of
	l Llong-long-suffix: one of	ll LL

</pre>

整数字面量是一个不带小数点和指数符号的数字的序列，在这个序列中可以有单引号进行分隔，但是在计算值的时候会被忽略。一个整数字面量可以有一个指示进制的基数和一个指示其类型的后缀。从文法上来讲，序列中第一个数字的权重最大（most significant）。一个二进制字面量（base two）以`0b`或者`0B`开头，后面跟二进制的数字序列。一个八进制的数是以`0`数字开头后面跟随一个八进制的字符序列。一个16进制字面量是以`0x`或者`0X`开头，后面跟一个16进制数字的序列，十六进制字符是由`a-f`和`A-F`和`0-9`数组组成。[*例子: 数字20可以写成： 12， 014， 0XC或者0b1100，数字字面量 1048576, 1’048’576, 0X100000, 0x10’0000, and 0’004’000’000 描述的是同一个数值*]

整形数字字面量的支持的类型，及其类型所对应的整形数据类型对应关系罗列在表6中

表6 整形字面量的类型
```
|------------------|------------------------|---------------------------------------|
| Suffix           | Decimal literal        | Binary, octal, or hexadecimal literal |
|------------------|------------------------|---------------------------------------|
| none             | int                    | int                                   |
|                  | long int               | unsigned int                          |
|                  | long long int          | long int                              |
|                  |                        | unsigned long int                     |
|                  |                        | long long int                         |
|                  |                        | unsigned long long int                |
|------------------|------------------------|---------------------------------------|
| u or U           | unsigned int           | unsigned int                          |
|                  | unsigned long int      | unsigned long int                     |
|                  | unsigned long long int | unsigned long long int                |
|------------------|------------------------|---------------------------------------|
| l or L           | long int               | long int                              |
|                  | long long int          | unsigned long int                     |
|                  |                        | long long int                         |
|                  |                        | unsigned long long int                |
|------------------|------------------------|---------------------------------------|
| Both u or U      | unsigned long int      | unsigned long int                     |
| and l or L       | unsigned long long int | unsigned long long int                |
|------------------|------------------------|---------------------------------------|
| ll or LL         | long long int          | long long int                         |
|                  |                        | unsigned long long int                |
|------------------|------------------------|---------------------------------------|
| Both u or U      | unsigned long long int | unsigned long long int                |
| and ll or LL     |                        |                                       |
|------------------|------------------------|---------------------------------------|
```
如果一个整形字面量不能用上面的表中所列举的类型进行表示，那么他如果能被扩展的整形类型表示，那么可以被转换成一个扩展的整形类型进行表示。如果上表中的类型是有符号的类型，那么对应的扩展类型也应为有符号类型。如果上表的中的类型是无符号，那么对应的扩展类型也应该是无符号类型。如果列表中的类型是既包含有符号类型又包含无符号类型，那么对应的扩展类型也应该既包含有符号类型又包含无符号类型。如果在一个转换单元中整形字面量不能使用任何允许的整形类型进行表示，那么程序的语法是不符合规范的。

### 字符字面量

<pre>

character-literal:	encoding-prefixopt 'c-char-sequence'
	
encoding-prefix: one of	u8 u U

c-char-sequence:	c-char	c-char-sequence c-char

c-char:
	除了单引号`'`，反斜杠`\`和换行符之外的任何源代码字符集里面的字符	escape-sequence	universal-character-name
	
escape-sequence:	simple-escape-sequence	octal-escape-sequence	hexadecimal-escape-sequence
	
simple-escape-sequence: one of	\’ \" \? \\	\a \b \f \n \r \t \v

octal-escape-sequence:	\octal-digit	\octal-digit octal-digit	\octal-digit octal-digit octal-digit

</pre>

1. 一个字符字面量是一个或者多个包含在单引号里面的字符组成的序列，比如'a'，在字符字面量之前可以有后面这几个前缀进行修饰`u8`，`u`或者`U`，比如 `u8'w'，u'x'或者U'y'。
2. 一个字符字面量没有以前缀`u`，`U`，`u8`开头的话，就是属于普通字符字面量。一个字符字面量函数一个由产生式`c-char`推导得到的字符，并且在运行字符集的类型为`char`。它的值跟`c-char`类型的字符在运行字符集中的整形值相等。如果一个普通字符字面量含有多个`c-char`，那么就叫做多字符字面量。如果多字节字面量和普通字符字面量中包含有不在运行字符集里面的字符，由编译器自行决定是否支持，这个时候类型为`int`，值由编译器自行决定。
3. 如果一个字符字面量由`u8`开头，比如`u8'a'`，这样的字符字面量的类型是`char`，称作 `UTF8`类型的字符字面量。`UTF8`类型的字符字面量的值等于`ISO 10646`中的码点的值，这个值由单个UTF8的编码单元（code unit）进行表示（提供 C0 Controls 和 Basic Latin Unicode block）。如果值不能由单个 UTF8 的编码单元进行表示，程序就不符合规范。UTF8 字符字面量如果含有多个`c-chars`，程序是不符合规范的。
4. 如果一个字符字面量有字母`u`开头，比如`u'a'`，这样的字符字面量的类型是`char16_t`，这样的字符的值是一个由`c-char`表示的`ISO 10646`编码点的值。他提供的编码点的值由一个16位的编码单元进行表示（范围是基本多语言平面 basic multi-lingual plane）。如果`char16_t`字符字面量的值不能有一个16位的编码单元进行表示，或者包含多个`c-chars`产生的字符，那么程序是不符合规范的。
5. 如果一个字符字面量有字母`U`开头，比如`U'a'`，这样的字符字面量的类型是`char32_t`，这样的字符的值是一个由`c-char`表示的`ISO 10646`编码点的值。他提供的编码点的值由一个16位的编码单元进行表示（范围是基本多语言平面 basic multi-lingual plane）。如果`char16_t`字符字面量的值不能有一个16位的编码单元进行表示，或者包含多个`c-chars`产生的字符，那么程序是不符合规范的。
6. 一个特定的非图形字符，单引号，双引号，问号和反斜杠可以通过表7中的展现方式进行显示。双引号和问号可以直接用自身来表示或者加上反斜杠，比如`\"`，`\?`，但是单引号和反斜杠之后在前面加上反斜杠进行表示`\'`和`\\`。没有出现在表7中的转义字符由极语言编译器自行决定是否支持，注意转义描述方式只代表单个字符。
<pre>
表7 支持的转义序列

|------------------|--------|-------|
| new-line         | NL(LF) | \n    |
| horizontal tab   | HT     | \t    |
| vertical tab     | VT     | \v    |
| backspace        | BS     | \b    |
| carriage return  | CR     | \r    |
| form feed        | FF     | \f    |
| alert            | BEL    | \a    |
| backslash        | \      | \\    |
| question mark    | ?      | \?    |
| single quote     | ’      | \’    |
| double quote     | "      | \"    |
| octal number     | ooo    | \ooo  |
| hex number       | hhh    | \xhhh |
|------------------|--------|-------|
</pre>
7. 转义序列`\ooo`由一个反斜杠开始后面跟一个，两个或者三个八进制的数如表示特定的字符。转义序列`\xhhh`由一个反斜杠开始后跟一个`x`然后紧跟一个或者多个16进制的数字来表示特定的字符，16进制的数字序列的长度没有限制。一个八进制或者16进制的序列由一个不是8进制或者16进制的数字终止。一个无前缀修饰字符字面量的值超过了`char`数据类型所能描述的最大范围，那么这个时候的值由编译器自行指定。[*Node: 如果一个由`u`,`U`或者`u8`前缀修饰的字符字面量超过了对应的数据类型的范围，那么程序是不符合规范的*]
8. `universal-character-name`描述的字符字面量转换到运行字符集对应的字符，如果不存在这个对应的编码字符，那么久转换到由编译器使用的一个内部字符集所对应的字符上。[*Note: 早编译转换步骤1的时候，当在源代码中遇到一个扩展的字符时候，我们将其转换成`universal-character-name`进行表示，因此所有的扩展字符都是用对应的`universal-character-name`进行表示。但是编译器可以使用自己本地的字符集，只要能保证同样的结果*]

## 代码注释

代码注释分为多行注释和单行注释
### 多行注释
多行注释以子串`/*`开头，以子串`*/`结束，多行注释不支持嵌套。

### 单行注释
单行注释由子串`//`开头，以紧跟的一个换行符结束。如果在注释中出现`form-feed`字符或者`vertical-tab`字符，则用一个空白符代替，在这种情况下不需要输出诊断信息。

### 注意事项
在单行注释中, 注释分隔符 `//`,`/*`和`*/`没有特殊意义，只是代表字符本身的意义，同样，在多行注释中注释分割符`//`和`/*`没有特殊意义，只是代表字符本身的意义。
