# 表达式属性
## 值分类

1. 表达式按照图一所示的分类学进行分类。
	
	<pre>
	          expression
	          /        \
	      glvalue     rvalue
	       /    \     /   \
	    lvalue   xvalue  prvalue
	</pre>
	1. 一个`glvalue`分类是一个通过求值去决定一个对象，一个位域或者一个函数的身份的表达式（`identity`）。
	2. 一个`prvalue`分类是一个通过求值去初始化一个对象或者位域，或者计算运算符的操作数的值的一个表达式，具体是那种根据出现的上下文去决定。
	3. 一个`xvalue`分类是一个`glvalue`，但是这个代表一个对象或者位域等等其资源能够重复利用的程序实体（通常是因为快靠近它们的生命周期的结尾）。[*Note: 有些有`rvalue`引用参与的表达式会产生一个`xvalues`类型的值，比如：调用一个函数，其返回值是右值类型或者被转换成右值类型。*]
	4. 一个`lvalue`分类是由`glvalue`分类中那些除去`xvalue`分类的剩余分类。
	5. 一个`rvalue`分类是由`prvalue`分类和`xvalue`分类组成的。
2. 每个表达式属于并且只属于这个分类方法中的基础分类的一种：`lvalue`，`xvalue`和`prvalue`。表达式的这个属性叫做表达式的值分类。[*Note: 在本章中，我们将描述`8.5`章节中的内置运算符求值之后的值的分类和运算符的操作数期望的值的分类。举个例子，内置的赋值运算符期望它的左操作数的值分类是一个`lvalue`，它的右操作数的值类型是一个`prvalue`，并且求值之后的产生的结果的值类型是`lvalue`。用户自定义的重载操作符的各种值的分类由操作符重载函数的形参声明类型和返回值的声明类型来确定。*]
3. [*Note: 由于历史原因，左值（`lvalue`）和右值（`rvalue`）的叫法是因为它们一个出现在赋值运算符的左边，一个出现在赋值运算符的右边。（现在这个分类的方法现在已经不太合适了，不能描述所有的可能情况）；`glvalue`是通用意义上的左值，`prvalues`是纯净的右值。`xvalue`是声明周期即将结束（`eXpiring`）的左值。尽管它们的名字里面都是带有值（`value`），但是他们不是针对（`value`）的分类，而是针对表达式的分类。*]
4. 如果一个表达式满足一下条件之一，那么该表达式就属于`xvalue`分类：
	1. 不管是隐式或者显式的调用函数，如果其函数的返回值类型是对象的右值引用（`rvalue reference`）。
	2. 一个通过强制类型转换成对象类型的右值引用（`rvalue reference`）。
	3. 一个`xvalue`类型的对象的一个访问类非静态并且不是引用类型的数据成员的类成员访问表达式的分类是`xvalue`。
	4. 一个`.*`类型的指向类成员的表达式，其中运算符`.*`的左边操作数的值分类是`xvalue`，第二个操作数是指向数据成员的指针。

	简单来说，这条规则的效果是命名的右值（`rvalue`）引用被当成一个左值（`lvalue`），未命名的对象右值引用被当着`xvalue`；函数的右值（`rvalue`）引用，不管是否被命名都被当成`xvalue`。
	
	例子说明：
	
	```cpp
	class A
	{
	   int m;
	};
	A&& operator+(A, A);
	A&& f();
	
	A a;
	A&& ar = static_cast<A &&>(a);
	```
	在这个例子中表达式`f()`，`f().m`，`static_cast<A &&>(a)`和`a + a`的分类是`xvalue`。
5. 一个`prvalue`表达式的结果（`result`）是表达式在上下文中存入的值。一个`prvalue`分类的表达式的结果`V`，有时候被叫做有值`V`或者命名了值`V`。`pvalue`分类的表达式的结果对象（`result object`）是一个被`pvalue`分类表达式初始化的对象。一个被用来计算运算符的操作的值的`pvalue`分类的表达式或者计算具有`cv void`类型的值时没有结果对象（`result object`）。[*Note: 一个`prvalue`分类表达式除了用在`decltype-specifier`里面，一个类类型或者数组类型的`pvalue`永远具有结果对象。对于丢弃的`prvalue`表达式分类，一个临时对象被具体化了（todo: 这个是什么意思啊）；*]表达`glvalue`分类的结果是该表达式标识的程序对象实体。
6. 当一个`glvalue`分类的表达式出现在一个要求操作数的分类是`prvalue`的表达式中的时候，会对这个传入的`glvalue`分类的表达式的值进行，`lvalue-to-rvalue`，`array-to-pointer`或者`function-to-pointer`的标准转换，将一个`glvalue`分类的值转换成`prvalue`分类的值。[*Note：将一个`rvalue`分类的值绑定到一个`lvalue`分类的值上，不在这个转换上下文考虑之中。*][*Note：当一个表达式的值是一个非类类型的时候，表达式的值的`cv-qualifiers`在转换成`prvalue`之前就被删除掉了。举个例子来说，一个值类型是`const int`的`lvalue`表达式可以被值类型为`int`的`prvalue`分类的表达式所接受。*][*Note: `prvalue`分类的值类型不能为位域，当一个位域转换成一个`prvalue`分类的值类型时，`prvalue`分类的类型的位域创建之后马上就被做了整数提升转换。*]
7. 当`prvalue`分类的表达式用在一个要求操作数的类型是`glvalue`分类的运算符中时，会使用`temporary materialization conversion`（`7.4`）将`prvalue`分类类型的表达式的值转换成`xvalue`分类类型的表达式的值。
8. 在`11.6.3`中的引用初始化和`15.2`中的临时对象相关的指出了`lvalue`和`rvalue`在其他重要的上下文中的的行为。
9. 除非特别指出，一个`prvalue`类型的表达式应该是完整类型或者`void`类型。一个`glvalue`分类的表达式不能是`cv void`类型。[*Note: `glvalue`分类的表达式的类型可以是完整类型也可以是不完整的类型。`prvalue`分类的表达式的值类型如果是类类型或者数组类型，那么可以是`cv-qualified`类型，其他类型的`pvalue`分类的表达式的值类型必须是`cv-unqualified`类型。*]
10. 一个`lvalue`分类的表达式的值是可修改的（modifiable）除非值类型是`const-qualified`类型或者是函数类型。[*Note：如果程序通过一个不可修改的左值表达式或者右值表达式去修改一个对象，那么程序是不符合规范的。*]
11. 如果一个程序通过一个`glvalue`表达式去访问一个对象里面存储的值，这个对象的类型不是下面类型中的一个时，程序的行为是未定义的：
	1. 对象的动态类型。
	2. 对象的动态类型`cv-qualified`版本。
	3. 对象的动态类型的`type similar`（在`7.5`中定义）。
	4. 对象的动态类型对应的`signed`或者`unsigned`类型。
	5. 对象的动态类型`cv-qualified`版本类型对应的`signed`或者`unsigned`类型。
	6. 一个包含上述类型的元素或者非静态数据成员的聚合或者联合类型。(including, recursively, an element or non-static data member of a subaggregate orcontained union)。
	7. 对象的动态类型的基类类型（可能是`cv-qualified`）。
	8. `char`，`unsigned char`，或者`polar::byte`。

## 类型

1. 如果一个表达式的类型是类型`T`的引用类型，在进行其他分析之前表达式的类型会被调整成类型`T`。表达式指定对象或者函数的引用并且表达式的分类是`lvalue`或者`xvalue`由表达式本身决定。[*Note: 引用类型开始之前或者结束之后进行使用，程序的行为是未定义的。*]
2. 如果一个`prvalue`分类的表达式初始类型是`cv T`并且类型`T`是一个`cv-unqualified`的非类类型，非数组类型。表达式的类型在进行其他处理之前，会被调整为`T`。
3. 类型`T1`和类型`T2`的`cv-combined`类型`T3 similar to T1`并且`cv-qualification`原型是：
	1. 对于所有的`i > 0`，cv<sub>i</sub><sup>3</sup>是cv<sub>i</sub><sup>1</sup>和cv<sub>i</sub><sup>2</sup>的联合。
	2. 如果cv<sub>i</sub><sup>3</sup>跟cv<sub>i</sub><sup>1</sup>或者cv<sub>i</sub><sup>2</sup>不一样，那么`const`将添加到所有的cv<sub>k</sub><sup>3</sup>上面，其中`0 < k < i`。[ Note: Given similar types T1 and T2, this construction ensures that both can be converted to T3. —endnote ]
4. 类型`T1`的操作数`p1`和类型`T2`的操作数`p2`的`composite pointer type`，至少有一个是指针类型或者成员指针类型或者`polar::nullptr_t`：
	1. 如果`p1`跟`p2`都是空指针常量的话组合结果是`polar::nullptr_t`;
	2. 如果`p1`或者`p2`是空指针常量，组合结果相应的是类型`T2`或者类型`T1`;
	3. 如果类型`T1`或者类型`T2`，其中一个是指向`cv1 void`的指针类型并且另一个是指向`cv2 T`的指针类型，这里的`T`是对象类型或者`void`类型。那么他们组合指针类型是`pointer to cv12 void`，这里的`cv12`是`cv1`和`cv2`的联合。
	4. 如果类型`T1`或者类型`T2`，其中一个是指向`noexcept`函数类型的指针类型并且另一个是指向函数类型的指针类型，那么组合类型是指向函数类型的指针类型。
	5. 如果`T1`是指向`cv1 C1`的指针并且`T2`是指向`cv2 C2`的指针，这里`C1`是`C2`的引用相关的类型，或者`C2`是`C1`的引用相关的类型。那么组合指针类型是`T1`和`T2`的`cv-combined`类型或者是`T2`和`T1`的`cv-combined`类型。
	6. 如果`T1`是指向类`C1`的类型为`cv1 U1`的成员的成员指针类型并且`T2`是指向类`C2`的类型为`cv2 U2`的成员的成员指针类型，这里的`C1`是`C2`引用相关的类型或者`C2`是`C1`引用相关的类型。组合指针类型是`T2`和`T1`的`cv-combined`类型，或者`T1`和`T2`的`cv-combined`类型。
	7. 如果`T1`跟`T2`是`similar`类型，那么组合指针类型是`T1`和`T2`的`cv-combined`类型。
	8. 如果其他任何形式的组合出现，那么程序是不符合规范的。
	
	例子说明：
	
	```cpp
	typedef void *p;
	typedef const int *q;
	typedef int **pi;
	typedef const int **pci;
	```
	指针类型`p`和指针类型`q`的组合指针类型是`const void *`，指针类型`pi`和指针类型`pci`的组合指针类型是`const int * const *`。

## 上下文依赖

1. 在有些上下文中，能够出现不被求值的操作数（8.4.7, 8.5.1.8, 8.5.2.3, 8.5.2.7, 10.1.7.2, Clause 17）。一个不被求值的操作数是没有没求值的。[*Note: 在一个没有求值的环境中，一个类的非静态成员可能被命名，但是对象或者函数没有命名，所以要求其定义一个要提供。一个未求值的操作数可以看成一个完整的表达式。*]
2. 在一些上下文中，表达式的存在仅仅是为了其副作用。这样的表达式叫做值丢弃表达式（`discarded-value expression`），标准准换`array-to-pointer`和`function-to-pointer`不会被应用。当且仅当表达式的类型是被`volatile-qualified`修饰`glvalue`的分类的值类型并且是下面的情况之一时，表达式的值类型会被进行标准的`lvalue-to-rvalue`转换：
	1. `(expression)`，这里的`expression`列出的几种情况之一的表达式。
	2. 标识符表达式（`id-expression`）。
	3. 下标访问表达式（`subscripting`）。
	4. 类成员访问表达式（`class member access`）。
	5. 间接访问表达式（`indirection`）。
	6. 指向成员的操作表达式（`pointer-to-member`）。
	7. 三元条件表达式（`conditional expression`），其中其二个和第三个操作数是这个列表中出现的表达式类型之一。
	8. 逗号表达式（`comma expression`），其中右边的表达式是这里支持的表达式类型之一。
	[*Note: 使用重载的运算符会导致一个函数调用；上面的情况只包含内置的操作运算符的情况。*]如果表达式（或者转换后的）的分类是`prvalue`，那么`temporary materialization conversion`将被应用。[*Note: 如果表达式的类型是类类型的左值分类，那么它必须具有一个易变的构造函数去初始化由应用`lvalue-to-rvalue`转换获取的临时对象。*]`glvalue`分类的表达式将会被求值，但是结果被丢弃。