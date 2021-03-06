# 作用域
## 声明区域和作用域
1. 一个名字在程序文本中被引入的特定区域叫做这个名字的声明区域（*declarative region*）,这个区域是这个引入的名字的最大有效范围，在这个区域中，我们可以使用非限定的名字取访问同一个实体。一般来说，对于所有特定名字来说，程序文本中的那些名字有效的非连续的部分，叫做这个名字的作用域。去确定一个声明的作用域的时候，我们往往先找出当前声明的可能作用域（*potential scope*）。一个声明的作用域与可能作用域是相等的，除非在可能的作用域里面出现了相同名字的声明。这个时候，里面名字的声明区域的可能作用域被外面的声明区域的可能作用域排除在外。
2. 下面是个简单的例子：

	```cpp
   int j = 24;
   int main()
   {
      int i = j, j;
      j = 42;
   }
	```
	标识符`j`在这里被声明两个并且被使用两次。第一个`j`的声明区域是整个例子程序，他的可能的作用域从它的声明开始到程序的结束，它的使用作用域得除去字符`,`和字符`}`之间的程序区域。第二个`j`的声明区域（在分号前面的`j`）是字符`{`和字符`}`之间的程序区域，它的可能作用域是除去`i`声明之前的程序区域，它的实际作用域跟可能的作用域的范围一致。
3. 一个名字被一个声明引入当前作用域中，作用域由声明语句中名字出现后的地方开始，除了`elaborated-type-specifier`和`using-directives`语句，他们修改了这一行为。
4. 在一个声明区域里面，给定一个声明的集合，每一个声明都使用了相同的非限定名字，则应该是下面情况之一：
	1. 他们所有都应该引用相同的程序实体，或者所有都引用函数或者函数模板。
	2. 只能有一个名字声明类或者枚举类型并且不能是`typedef`类型的名字，其他剩余的声明中的名字必须引用相同的变量，非静态数据类成员，特定的枚举类型的成员（enumerator），或者所有声明都引用函数和函数模板；在这种情况下，类的名字和枚举类型的名字被隐藏了。[*Note: 一个命名空间的名字和类模板的名字必须在其声明区域保持唯一*]
[*Note: 这些限制作用于名字引用的声明区域，这个区域可能跟声明语句所在的区域不一样。特别的说 `elaborated-type-specifiers`语句将在当前区域的闭包的命名空间里面引入一个名字声明（可能不可见）。这些限制将作用于那个命名空间。局部的`extern`语句将在当前的声明区域和当前声明区域的闭包中引入一个名字 (闭包中的名字可能不可见)，所以这些限制将作用于这两个区域。*]
5. 对于一个给定的声明区域`R`和在`R`之外的一个点`P`，`P`与`R`之间的区域是由`R`闭包且不是`P`的闭包的部分组成。
6. 名字查找规则在下面名字查找章节进行总结。

## 声明点

1. 一个名字的声明点（Point of declaration）是从它自己的完整声明符（complete declarator）开始到它的初始化器结束，如果有初始化器的话。除了下面说明（Note）的情况：
	
	```cpp
	unsigned char x = 12;
	{
	   unsigned char x = x;
	}
	// 这里第二个 x 用它自己进行初始化
	```
2. [*Note: 一个来自外面的作用域的一个名字将保持可见，直到这个名字的另一个声明点隐藏它。*]
	```cpp
	const int i = 2;
	{
	   int i[i];
	}
	// 在块作用域中定义一个具有两个元素的数组变量
	```
3. 通过`class-specifier`产生式声明的类或者类模板的声明点是在其`class-head`产生式产生的`identifier`或者`simple-template-id`（如果有的话）后面。一个由`enum-specifier`产生式推导的枚举类型的声明点从其`identifier`开始，一个别名或者模板别名从别名引用的`type-id`开始。
4. 一个没有选择构造函数的`using-declarator`的声明点是用器声明符后面开始。
5. 一个枚举符的声明点从它的枚举项定义完成（enumerator-definition）后开始。
	
	```cpp
	const int x = 12;
	{
	   enum {
	      x = x;
	   };
	}
	// 这里枚举项（enumerator）x 使用常量 x 的值进行初始化
	```
6. 在一个类成员的声明点之后，在类作用域中都可以通过成员名进行查找。
	[*Note: 当类还没有看到完成的定义时，这条规则也成立*]
	```cpp
	struct X
	{
	   enum E
	   {
	      z = 16;
	   };
	   int b[x::E::z]; // OK
	};
	```
7. 对于`elaborated-type-specifier`产生式的类声明的声明点由以下规则指定：
	1. 对于声明形式为：
		```cpp
		class-key attribute-specifier-seqopt identifier;
		```
		会在声明出现的作用域中引入一个名字叫做`identifier `的类类型，否则参考下面一条规则。
	2. 对于声明形式为：
		```cpp
		class-key identifier;
		```
		如果`elaborated-type-specifier`被用于`decl-specifier-seq`产生式或者被用于一个在命名空间的函数定义的`parameter-declaration-clause`产生式中，那么在声明出现的命名空间里面引入名字为`identifier`的类类型；否则`identifier`将在包含声明的最小命名空间或者块作用域引入。[*Note: 这些规则在模板领域同样适用*] [*Note: 其他形式的`elaborated-type-specifier`没有声明一个新的名字，因此必须引用一个存在的类型名*]
8. `injected-class-name`的声明点是从类定义的左大括号开始。
9. 函数局部预定义的变量的声明点从函数体定义开始的地方开始。
10. 在`range-based` 类型的 `for`循环的`for-range-declaration`产生式中，变量和结构化绑定的声明点是从`for-range-initializer`产生式的后面开始。
11. 模板形参的声明点是从该模板形参的完整定义后面开始。
	例子：
	
	```cpp
	typedef unsinged char T;
	template <typename T
				= T, // 此时 T 被定义成 unsiged char
				, T, // 此时 T 为模板参数 T
				N = 0>
	class A
	{};
	```
12. [*Note: 函数友元声明和类友元声明引用的是最近的作用域的命名空间闭包里面的成员，他们不在当前作用域里面引入新的名字。在块作用域里面声明函数和`extern`类型的变量声明，他们引用的是命名空间闭包里面的成员，但是他们都不往当前作用域里面引入新的名字。*]
13. [*Note: 模板实例化的点。*]

## 块作用域

1. 在块（block）声明的名字，是块的局部变量，具有块级作用域。它的可能的作用域从名字的声明点开始当当前块结束。一个声明在块中的变量叫做局部变量。
2. 一个在`exception-declaration`声明的名字，是异常处理器的局部变量，所以不能在异常处理器的最外层块作用域里面重新声明。
3. 在`init-statement`，`for-range-declaration`和在条件判断上下文比如`if`，`while`，`for`和`switch`语句声明的名字是这些语言的局部变量，不能再后续的子条件序列中或者受控制的语句最外层的块中重新声明。[*Note: if 语句，就是任何最外层块*]

## 函数形参作用域

1. 一个函数的形参（包含`lambda-declarator`里面的形参）或者一个函数局部预定义变量具备函数形参作用域。函数形参和函数局部预定义变量的可能作用域从其声明点开始。如果最靠近的函数声明符不是一个函数定义，那么可能的作用域在函数声明符末尾结束。否则，如果函数具有一个`function-try-block`，那么可能的作用域结束于最后一个异常处理器的块末尾。否者可能的作用域结束于函数定义的最外层块的最末尾。函数形参的名字不能再函数定义的最外层块中重新声明或者不能在任何`function-try-block`异常处理器的最外层块中重新声明。

## 函数作用域

1. 标号（Lables）具备函数作用域并且能在他所定义的函数体的任何地方使用，只有标号能具有函数作用域。

## 命名空间作用域

1. 命名空间定义（namespace-definition）的声明区域是命名空间体块（namespace-body）。声明在命名空间体里面的实体成为命名空间的成员。在命名空间的声明逾期由声明引入的名字叫做命名空间的成员名字（member names）。一个命名空间的成员名字具备命名空间作用域。命名空间成员名字的可能的作用域从各自的声明点开始到命名空间体尾部结束。通过`using-directive`指定的名称空间成员，这些成员名字的可能作用域包含`using-directive`语句中对应的成员名字声明的点之后的可能的作用域。

	下面是命名空间作用域的例子：

	```cpp
	namespace N
	{
	   int i;
	   int g(int a)
	   {
	      return a;
	   }
	   int j();
	   void q();
	}
	
	namespace
	{
	   int l = 1; // 可能的作用域从成员名字的声明点开始到当前转换单元结束
	}
	
	namespace N
	{
	   int g(char a) // 重载 N::g(int);
	   {
	      return l + a;
	   }
	   
	   int i; // 错误，重复定义
	   int j(); // Ok 重复函数声明
	   int j() // Ok， 定义函数 int N::j();
	   {
	      return g(i);
	   }
	   int q(); // 错误，仅仅返回值不一样，一个命名空间的成员名字不能用于不同的实体声明
	}
	```
2. 一个命名空间的成员名字或者通过`using-directive`语句引入的特定命名空间的成员名字可以通过作用域访问运算符`::`（scope resolution operator）由命名空间的名字进行访问。
3. 一个转换单元最外层的什么区域也是一个命名空间，叫做全局命名空间。全局命名空间里面的成员名字具有全局命名空间作用域（也称作全局作用域）。一个全局命名空间里面的名字的可能的作用域从它的声明点开始到转换单元（也是声明区域）的尾部结束。一个全局命名空间里面的名字叫做全局名字。

## 类作用域

1. 一个在类定义里面声明的名字的可能的作用域不仅包含从声明点开始往下的到类定义结束，而且还包含所有的方法定义体，默认参数，`noexcept-specifiers`异常说明语句和默认数据成员初始化语句 (default member initializers)。
2. 一个在类里面使用的名字，在其使用的上下文中指向的程序实体应该跟当被在完整的类的作用域中重新求值得到的程序实体一致，为同一个实体。违反这条规范编译器不需要输出相关的诊断信息。
3. 在成员函数中声明的名字会屏蔽那些作用域扩展成员函数所在类的定义体结束或者覆盖到的成员函数所在类的定义体结束之后相同的名字。
4. 作用域扩展到或者超过类定义体的名字的可能的作用域也会覆盖到由成员定义体所定义的区域，即使这些区域在文法上不在类的定义体里面（todo：这个地方可能有变动，极语言不区分类的声明和类的定义，我们将其合二为一），这些区域包含：静态数据成员的定义区域，成员函数定义区域（包含函数体，以及在声明符中出现在`declarator-id`之后的区域，比如形参声明语句`parameter-declaration-clause`和任何默认参数指定区域）
	
	下面是一个例子：
	
	```cpp
	typedef int c;
	enum {
	   i = 1
	};
	
	class X
	{
	   char v[i]; // 编译错误，i 这里引用的是 ::i，但是完整作用域下引用的是 X::i，同一个名字引用了不同的实体
	   int f()
	   {
	      return sizeof(c); // OK，这里引用了 X::c
	   }
	   
	   char c;
	   
	   enum {
	      i = 2
	   };
	};
	
	typedef char* T;
	struct Y
	{
	   T a; // 语法错误，这里引用的是 ::T，但是完整的类作用域下面引用的是 Y::T，同一个名字引用了不一样的实体
	   typedef long T;
	   T b;
	};
	
	typedef int I;
	class D
	{
	   typedef I I; // // 编译错误, even though no reordering involved todo：暂时不清楚错误原因
	};
	```
6. 类成员的名字应该按照以下方式进行使用：
	1. 可以在上面描述的名字所在的类的作用域中进行引用或者在所在类的派生类中进行引用。
	2. 当`.`运算符应用到名字所在的类或者所在类的派生类的类型时，`.`运算符后面进行使用。
	3. 当`->`运算符应用与名字所在类或者所在类的派生类的类型实例化的对象上面时，`->`运算符后面进行使用。
	4. 当`::`作用域访问运算符左右到名字所在类或者所在类的派生类的类型时，在`::`运算符后面进行使用。
	
## 枚举作用域

1. 枚举成员名字的作用域叫做枚举作用域，枚举成员的名字可能的作用域从名字的声明点开始到枚举定义`enum-specifier`指示符的末尾结束。

## 模板形参作用域

1. 一个由`template template-parameter`表达式引入的模板形参名字的声明区域是名字所在的`template-parameter-list`中的最小区域。
2. 一个模板的模板形参名字的声明区域是从名字所在的`template-declaration`最小区域。仅仅是模板形参的名字属于这一区域。模板声明`template-declaration`引入的其他名字不会加入到这个声明区域，这些名字会加入到跟采用非模板声明时相同的名字所属的声明区域中去。
	
	例子说明：
	
	```cpp
	namespace N
	{
	   template <typename T> struct A {}; // #1
	   template <typename U> void f(U) {} // #2
	   struct B {
	      template <typename V> friend int g(struct C*); // #3
	   };
	};
	```
	名字 `T`，`U`和`V`的声明区域是 #1，#2 和 #3所代表的模板声明表达式区域（template-declarations）。名字`A`，`f`，`g`和 `C`全部属于同一个声明区域，这个区域是命名空间体。（namespace-body）名字`g`属于这个声明区域，尽管在限定或者非限定名字查找时他被隐藏。
3. 模板形参可能的作用域从名字的声明点开始到所在的声明区域结束。[*Note: 这条规则蕴含着，当前的模板形参可以参与到后续的新参声明表达式中，或者作为其默认参数，但是不能用于当前形参前面的形参相关的声明语句和用作其默认参数*]
	
	例子说明：
	
	```cpp
	template<class T, T* p, class U = T> class X { /* ... */ };
	template<class T> void f(T* p = new T);
	```
	这条规则同样蕴含着，模板形参可以用来定义基类：
	
	```cpp
	template<class T> class X : public Array<T> { /* ... */ };
	template<class T> class Y : public T { /* ... */ };
	```
	把模板形参用于指定基类，那么在模板实例化的时候，对应的模板实参必须看到完成的定义，不能仅仅只看到声明。
4. 模板形参的名字在声明点之后马上嵌入直接包含其声明区域的闭包作用域。[*Note: 这条规则暗示着，在其声明点之后马上隐藏这个名字的闭包作用域中相同的名字*]
	
	例子说明：
	
	```cpp
	typedef int N;
	template<N X, typename N, template<N Y> class T> struct A;
	```
	在这里，`X` 的类型是非模板参数类型`int`，但是`Y`的类型是模板参数`N`所代表的类型
5. [*Note: 因为模板形参的名字不能在其可能的作用域中重新声明，所以一个形参的作用域与其可能的作用是相同的。尽管如此，模板的形参的名字依旧可能被隐藏。todo: 怎么隐藏？*]

### 名字隐藏

1. 如果一个名字在其所在的作用域的内嵌的声明区域或者其所在的类的派生类类中被显示的声明，那么这个名字将被新声明的名字隐藏。
2. 当一个类的名字和一个枚举类型的名字会被在同一作用域的变量的名字，数据成员名字，函数名字或者枚举项的名字所隐藏。当类名字和枚举类型名字与变量，数据成员，函数或者枚举项在同一作用域使用同一名字以任何顺序声明时，只要变量，数据成员，函数或者枚举项名字一旦可见，那么类和枚举类型的名字声明将被隐藏。
3. 在成员函数体的块作用域中声明一个名字，那么这个名字将会隐藏相同名字的类成员。在派生类的成员名字，将会隐藏父类中相同名字的成员声明。
4. 在名字查找的时候，使用命名空间限定的方式与访问一个名字，比如 `A::a`，其中`A`命名空间，`a`是在命名空间中使用`using-directive`方式进行引入的，如果在该命名空间中存在相同名字的声明，通过`using-directive`方式引入的名字将被隐藏。
5. 在一个作用域中，如果一个名字声明没有被其他相同名字的声明所隐藏，那么我们说该名字的这个声明在作用域中可见（visible）。
