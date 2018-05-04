# 名字查找

1. 当讨论特定的规则的时候，在当前上下文中，只要文法允许相关的名字，那么名字查找规则就会应用到相应的名字（包括`typedef`引入的名字，命名空间的名字和类类型的名字）上。名字查找将名字的用途与改名字的一系列声明进行关联。这些通过名字查找得到的某个特定名字的所有声明要么引用相同的程序实体，要么所有的声明都指示函数或者函数模板。在后面这种情况，所有该名字的声明形成了一个针对该名字的函数重载集合。函数重载的决议发生在名字查找成功完成之后。访问控制规则（`access rules`）在名字查找和函数重载决议（`unction overload resolution`），如果可应用的话，成功之后才进行检查。只有在名字查找，函数重载决议和访问控制检查都成功之后，被名字声明引入的属性才会被处理。
2. 一个名字的在一个表达式上下文中进行名字查找是指在表达式所在的作用域中进行该名字的非限定名字查找的过程。
3. 一个类注入名字（`injected-class-name`），是一个类为了实现名字隐藏和名字查找而引入的一种类成员名字。

## 非限定名字的查找

1. 在本节列举的所有情况，当为一个名字查找合适的声明的时候，按照各自的分类所列举的顺序对作用域进行查找。在查找的过程中，只要找到合适的声明，整个查找过程结束，如果没有找到合适的声明，那么程序是不符合规范的。
2. 通过`using-directive`语句引入的名字声明，在`using-directive`语句所在的命名空间可见。在进行本节描述的非限定名字查找的时候，由`using-directive`语句引入的名字声明，将按照其所在的命名空间名字声明参与查找。
3. 当一个非限定的名字用在一个函数调用的后缀表达式里面的时候，将使用下面章节的*实参依赖名字查找*规则进行名字查找。在某些情况下当进行名字查找的时候，一个名字后面跟一个`<`字符，就算名字查找没有发现一个`template-name`，该名字也会被当成一个`template-name`。
	
	例子说明：
	
	```cpp
	int h;
	void g();
	namespace N
	{
	   struct A {};
	   template <class T> int f(T);
	   template <class T> int g(T);
	   template <class T> int h(T);
	}
	int x = f<N::A>(N::A()); // f 名字查找没有找到任何声明，应用参数依赖查找，找到 f 函数模板
	int y = g<N::A>(N::A()); // g 名字查找找到了 void g() 不匹配，应用参数依赖查找，找到 g 函数模板
	int z = h<N::A>(N::A()); // 找到名字 h，h< 不开始一个 template-id，查找失败
	```
	在一个表达式语法解释时候，参数依赖名字查找没有效果。
	
	例子说明：
	
	```cpp
	typedef int f;
	namespace N
	{
	   struct A
	   {
	      friend void f(A &);
	      operator int();
	      void g(A a) {
	         int i = f(a); // 这里的 f 是 typedef 的别名，可能别名先应用，而不是友元函数，等价于 int(a)
	      }
	   };
	}
	```
   在这里这个表达式因为不是一个函数调用，所有参数依赖查找过程不会应用，所有友元函数没有被查找到。
4. 一个在任何函数和类和用户自定义命名空间之外的全局命名空间里面使用的名字，必须在全局命名空间中其使用点之前进行声明。
5. 在用户自定义的命名空间中，一个在任何函数和类之外使用的名字，必须在该命名空间中且名字使用点之前进行声明，或者该命名空间中且名字使用点之前的闭包命名空间中进行声明。
6. 在命名空间的一个函数定义时，在函数`declarator-id`之后出现的名字查找过程如下。在函数体中其所在的代码块中使用点之前应该有其声明，否在上层块中进行查找直到函数体的最上层代码块，否则在命名空间`N`中其使用点之前应该声明，如果`N`是一个嵌套命名空间，则在`N`的闭包命名空间，名字使用点之前应该声明，重复这个过程直到全局命名空间，如果还是没有发现该名字的声明，那么程序是不符合规范的。
	
	例子说明：
	
	```cpp
	namespace A
	{
	   namespace N
	   {
	      void f();
	   }
	}
	
	void A::N::f()
	{
	   i = 5;
	   // 名字 i 的名字查找过程如下：
	   // 函数 f 最外层块且在 i 使用之前的区域
	   // 命名空间 N 且在名字使用之前的区域
	   // 命名空间 A 且在名字使用之前的区域
	   // 全局命名空间，且在函数定义 A::N::f() 之前的区域
	}
	```
7. 一个在类 `X` 定义之内且在任何成员函数体，默认参数定义，异常指示符（`noexcept-specifier`），嵌套类和默认成员初始化器之外的区域使用的名字，使用一下规则进行名字查找：
	1. 在其使用点之前的类`X`定义体重进行声明，或者是当前类`X`的基类中的一个类成员
	2. 如果类`X`是类`Y`的一个嵌套类，在类`Y`中，类`X`定义之前应该进行声明，或者该名字应该是类`Y`的基类的一个成员的声明。（这个过程依次应用与类`Y`的闭包类，从最里层的闭包类开始）
	3. 如果类`X`是一个局部类，或者是一个局部类的嵌套类，在类`X`所在程序块之内在类`X`的定义之前应该对名字进行声明。
	4. 如果`X`是命名空间`N`的一个成员或者是命名空间中的成员类中的一个嵌套类或者是一个局部类或者是一个命名空间`N`中的函数成员中的一个局部类的一个嵌套类，那么在命名空间`N`中，在`X`使用之前或者在`N`的闭包命名空间里面应该有该名字的声明。
	
	例子说明：
	
	```cpp
	namespace M
	{
	   class B {};
	}
	
	namespace N
	{
	   class Y extend M::B
	   {
	      class X
	      {
	         int a[i];
	      }
	   }
	}
	
	// 对于 名字 i 如下的作用域将被依次查找
	// 类作用域 N::Y::X 中，在 i 的使用之前
	// 类作用域 N::Y 在 N::Y::X 定义之前
	// 类作用域 N::B
	// 命名空间 N 中，类 N::Y 定义之前
	// 全局命名空间中，命名空间 N 定义之前
 	```
8. 对于类`X`的成员，一个用在在成员函数定义语句`declarator-id`之后的函数体，默认参数，异常指示符（`noexcept-specifier`），默认成员初始化器中的名字，需要按照下面的要求进行声明，条目之前是或的关系：
	1. 在名字所在的代码且在名字使用点之前进行声明，或者在其所在代码的闭包代码块且当前代码之前进行声明。
	2. 名字应该是类`X`的成员获取类`X`基类的成员。
	3. 如果`X`是类`Y`的嵌套类成员，那么名字应该是`Y`的成员，或者是类`Y`的基类的成员。（重复运用这个查询规则到类`Y`的闭包类，从最里层的闭包类开始）
	4. 如果`X`是局部类，或者是局部类的一个嵌套类，那么在类`X`定义的代码块且在类`X`定义之前进行声明。
	5. 如果`X`是命名空间的成员或者是一个命名空间`N`的成员类的嵌套类或者是一个局部类或者是命名空间`N`的成员函数的函数体代码块中的一个局部类的嵌套类，那么在命名空间`N`中且名字使用之前或者命名空间的闭包命名空间且在当前命名空间定义之前需要声明。

	例子说明：
	
	```cpp
	class B {};
	namespace M
	{
	   namespace N
	   {
	      class X extend B
	      {
	         void f();
	      }
	   }
	}
	
	void M::N::X::f()
	{
	   i = 16;
	}
	// 查找名字 i 时候，会依次搜索下面的命名空间
	// 搜索 函数 M::N::X::f 最外层代码块且在 i 使用之前
	// 搜索 类 M::N::X 的成员
	// 搜索 类 ::B 中的成员
	// 搜索 命名空间 M::N
	// 搜索 命名空间 M
	// 搜索 全局命名空间，在函数 M::N::X::f
	```

9. 在类中定义的`inline`友元函数定义中使用的名字的查找规则跟普通的类中的成员函数的规则是一样的。不在类中定义的友元函数中使用的名字，查找规则跟普通的命名空间的函数成员的查找规则一样。
10. 当使用友元的方式授权一个类成员访问权限时候，在查找一个在函数声明符里面且不再模板形参列表中使用的名字时候，会首先查找被授权的成员函数所在的类的作用域，如果不在或者名字是在模板形参列表中使用的时候，使用非限定名字在友元声明时候的查找方式进行查找，具体见规则9。
	
	例子说明：
	
	```cpp
	class A
	{
	   typedef int AT;
	   void f1(AT);
	   void f2(float);
	   template <typename T> void f3();
	};
	
	class B
	{
	   typedef char AT;
	   typedef float BT;
	   friend void A::f1(AT); // 在所在的类A作用域中找，AT类型是 A::AT 
	   friend void A::f2(BT); // 先在类A作用找，没找到，然后再 B 中找，BT的类型是B::BT
	   friend void A::f3<AT>(); // 用在模板形参列表中，AT 的类型是 B::AT
	};
	```
11. 当进行函数的默认参数或者类的构造函数的初始化序列中的名字查找时候，函数形参的名字会隐藏当前代码库，类或者包含函数定义的命名空间里面的名字声明。
12. 在进行枚举项定义（`enumerator-definition`）的常量表达式（`constant-expression`）中的名字查找时，在之前定义的枚举项的名字会隐藏当前代码块，类或者包含`enum-specifier`的命名空间里面的名字。
13. 在静态数据成员定义中且在`qualified-id`之后的名字查找，跟在静态成员函数中出现的名字的查找规则一样。
14. 如果一个命名空间的成员的定义出现在命名空间外面的时候，任何在那个定义中使用的名字的查找规则跟该定义出现在命名空间里面时一样。
	
	例子说明：
	
	```cpp
	namespace N
	{
	   int i = 4;
	   extern int j;
	}
	
	int i = 2;
	int N::j = i; // N::j == 4
	```
15. 在函数的`function-try-block`的处理器块使用的名字的超早规则，跟这个名字在函数最外层块中使用一样，函数的形参的名字不能在`exception-declaration`和`function-try-block`的处理器的最外层块进行重新绑定。在`function-try-block`处理块作用域中的名字查找时，将会忽略函数定义的最外层块，但是函数的形参不忽略。[*todo: 需要好好研究，这个地方有点意思*]

## 实参依赖名字查找

1. 当一个非限定名字用在一个函数调用的后缀表达式中的时候，一些原本在正常的非限定名字查找是不会被搜索的命名空间在这种情况下会被搜索。这些搜索的命名空间，一些原本不会被发现的命名空间下的友元函数声明和函数模板的声明将可能被发现。至于那些命名空间被加入在搜索的命名空间跟参数的类型有关系。[*Note: 对于`template template arguments`，模板参数所在的命名空间将会被考虑*]
	
	例子说明：
	
	```cpp
	namespace N
	{
	   struct S{};
	   void f(S);
	}
	
	void g()
	{
	   N::S s;
	   f(s); // OK 使用 N::f
	   (f)(s); // 编译错误，()阻止了实参依赖名字查找规则的应用
	}
	```
2. 在函数调用中的每个实参的类型`T`，都有零个或者多个相关联的命名空间和零个或者多个类作用域被考虑，这个被考虑的命名空间和类作用域是根据函数调用的参数类型确定的（外加任何模板的模板参数`template template argument`的类型所在的命名空间）。通过typedef得到的名字和通过`using-declarations`得到的类型不加到这个集合中。这个集合中的命名空间和类通过下面的规则进行确定：
	1. 如果`T`是原生类型，那么关联的命名空间和类的集合为空。
	2. 如果`T`是一个类类型，那么他关联的类的集合如下：它自己，如果它是是一个嵌套类那么它的外层类加入集合，如果有的话。它的直接和间接的基类加入集合。关联的命名空间是，这些类集合中的类所在的最里层的闭包命名空间都是该集合的成员。更近一步，如果`T`是一个模板特殊化，类型`T`关联的命名空间和类的集合将包含，提供给模板形参的模板实参的类型所关联的类跟命名空间的集合也会被加入（排除`template template parameters`），任何`template template arguments`关联的命名空间是该集合的成员。那些被用做`template template arguments`的成员模板所在的类也是该集合的成员。[*Note: 非类型的模板实参不贡献任何关联的命名空间到这个集合中。*]
	3. 如果`T`是一个枚举类型，它所关联的命名空间是其定义所在的最里层的闭包命名空间。如果它定义在一个类型，那么它所关联的类是其成员定义所在的类，否则它关联的类的集合为空。
	4. 如果`T`是一个指向类型`U`的指针，或者是类型`U`的一个数组，那个`T`关联的命名空间和类的集合跟`U`是一样的。
	5. 如果`T`是一个函数类型，那么它所关联的命名空间和类的集合由它所有的参数类型和返回值类型所关联的命名空间和类集合组成。
	6. 如果`T`是一个指向类`X`函数成员的指针，那么它所关联的命名空间和类的集合由成员函数的所有参数类型和返回值类型和类型`X`所关联的命名空间和类的集合组成。
	7. 如果`T`是一个指向类`X`的数据成员指针，那么它所关联的命名空间和类的集合由指向的数据成员的类型和类`X`所关联的命名空间和类的集合组成。
如果所关联的命名空间是一个`inline`类型的命名空间，那么该命名空间的直接闭包命名空间也属于搜索集合。这些命名空间如果之间包含`inline`类型的命名空间，那么这些`inline`命名空间也属于搜索集合。如果一个参数是一个函数重载或者函数模板的名字或者地址，那么它所关联的命名空间和类的集合是这些实体所关联的命名空间和类的集合的并集。特别的如果，上述的重载函数集合中的函数有由`template-id`来命名的，那么这个函数类型所关联的类和命名空间的集合同时包含`type template-arguments`和`template template-arguments`所关联的命名空间和类的集合。
3. 让`X`代表由非限定名字查找得到的查找集合，`Y`代表有参数依赖查找得到的查找集合。如果`X`包含以下：
	1. 一个类成员的声明，
	2. 块作用域中的非`using-declaration`引入的函数原型声明，
	3. 既不是函数声明也不是函数模板声明
那么，`Y`集合为空。否则 `Y`集合是按照下面描述的在参数依赖查找到的关联命名空间中找到的声明的集合。这个时候名字查找考虑的集合是`X`与`Y`的并集。[*Note: 通过参数依赖找到的关联的命名空间或者类的集合可能包含通过非限定名字查找方式获取的命名空间或者类的集合。*]
	
	例子说明：
	
	```cpp
	namespace NS
	{
	   class T {};
	   void f(T);
	   void g(T, int);
	}
	
	NS::T param;
	void g(NS::T, float);
	int main()
	{
	   f(param); // OK， 调用 NS::f(T)
	   extern void g(NS::T, float); // 
	   g(param, 1); // 非限定查找不空，找到 g(NS::T, float)
	}
	```
4. 当前考虑一个相关联的命名空间的时候，名字的查找跟一个受限的查找过程一样，出去下面几种情况：
	1. 所有通过`using-directives`跟命名空间关联的声明被忽略
	2. 任何那些在类中声明的友元的声明，将在这个类所在的命名中被考虑，尽管他们在普通的名字查找时候不可见。
	3. 除了重载函数声明和函数模板之外的名字都被忽略。

## 限定名字查找

1. 在类和命名空间的成员的名字或者枚举项的名字可以通过应用在`nested-name-specifier`上的作用域访问运算符`::`的方式进行访问，这里的`nested-name-specifier`代表类名或者命名空间或者枚举的名字。如果应用在`nested-name-specifier`的作用域访问运算符`::`之前没有`decltype-specifier`修饰，那么在作用域运算符之前的名字只考虑命名空间，类型[*todo: 原文是`types`，这里包含原生类型吗？还是只是类类型。*]或者被特殊化成类型的模板。如果找到的名字不指示一个命名空间，类，枚举或者依赖的类型[*todo: 何为依赖的类型`dependent type`。*]那么程序是不符合规范的。
	
	例子说明：
	
	```cpp
	class A
	{
	public:
	   static int n;
	};
	
	int main()
	{
	   int A;
	   A::n = 42; // :: 之前的只考虑命名空间类类型
	   A b; // 编译错误 A 不是一个类型
	}
	```
2. [*Note: 多重限定名字，比如：`N1::N2::N3::n`，可以用来引用嵌套类的成员或者嵌套命名空间的成员。*]
3. 在一个声明语句中，如果`declarator-id`是一个`qualified-id`，用在`qualified-id`之前的名字在定义所在的命名空间中查找，在`qualified-id`之后的名字在成员所在的类或者命名空间的作用域中查找。
	
	例子说明：
	
	```cpp
	class X{};
	class C
	{
	   class X{};
	   static const int number = 50;
	   static X arr[number];
	};
	
	X C::arr[number]; // 语法错误
	                  // 等价于 ::X C::arr[C::number]
	                  // 而不是 C::X C::arr[C::number]
	```
4. 一个被一元命名空间访问运算符修饰的名字，将在全局命名空间中进行查找，在全局命名空间中其使用之前应该被声明或者其声明被`using-directive`引入到全局命名空间。通过一元命名空间作用域访问法，可以在名字在当前作用域被隐藏的情况下访问全局命名空间下的声明。
5. 一个被枚举类型的`nested-name-specifier`前缀修饰的名字，必须是该枚举类型下的一个枚举项的名字。
6. 如果`pseudo-destructor-name`中包含有`nested-name-specifier`，那么其中的`type-names`名字就在`nested-name-specifier`指示的命名空间中查找。类似的如果一个`qualified-id`具备如下的形式：

	<pre>
	nested-name-specifier<sub>opt</sub> class-name::~class-name
	</pre>
	第二个`class-name`查找的作用域跟第一个`class-name`一样。
	
	举个例子：
	
	```cpp
	class C
	{
	   typedef int I;
	};
	
	typedef int I1, I2;
	extern int *p;
	extern int *q;
	p->C::I::~I(); // I 在类 C 中进行查找
	q->I1::~I2(); // I2 在后缀表达式所在的作用域中进行查找
	
	class A
	{
	   ~A();
	};
	
	typedef A AB;
	
	int main()
	{
	   AB *p;
	   p->AB::~AB(); // 显式的调用A的析构函数
	}
	
	```
	
### 类成员名字查找

1. 如果一个`qualified-id`的`nested-name-specifier`指示一个类，在`nested-name-specifier`之后的名字将在这个类中进行查找，除了下面指出的这些情况。名字应该代表这个类或者其基类的成员名字。[*Note：一个类的成员可以使用`qualified-id`的方式在该类的可能的作用域范围里进行引用。*]，上面名字查找规则的意外的情况如下：
	1. 析构函数查找规则特殊，在上一节中已经指出。
	2. `conversion-function-id`的`conversion-type-id`的查找方式跟把`conversion-type-id`看成类成员名字，跟类成员的访问方式一样。
	3. `template-id`的`template-argument`里面的名字，在整个后缀表达式所在的作用域中进行查找。
	4. `using-declaration`中的名字查找会考虑在当前作用域中被隐藏的类和枚举的类型声明。
2. 在一个函数的名字不能忽略并且`nested-name-specifier`指示的是一个类`C`的查找过程中：
	1. 如果一个名字出现在`nested-name-specifier`的后面，当在类`C`中进行名字查找并找到相关名字且名字是类`C`的`injected-class-name`成员。
	2. 在一个`using-declaration`的`using-declarator`是一个`member-declaration`。如果在`nested-name-specifier`后面指定的名字跟`identifier`一样的时候或者`simple-template-id`的`template-name`出现在`nested-name-specifier`最后一部分的时候，
这个名字被解释为类`C`的构造函数。[*Note: 举个例子：在`elaborated-type-specifier`中，构造函数不是可接受的情况，所以构造函数不能用在`injected-class-name`出现的地方。*]这样的一个构造函数的名字只能用在那是命名一个构造函数的`declarator-id`里面，或者在一个`using-declaration`里面。[*todo: 这条规则有点晕*]
	
	例子说明：
	
	```cpp
	class A
	{
       A();
	};
	
	class B extend A
	{
	   B();
	};
	
	A::A() {}
	B::B() {}
	
	B::A ba; // 对象的类型是 A
	A::A a; // A::A 不是一个类型
	class A::A a2; // 对象的类型是 A
	```
3. 一个被嵌套声明区域或者派生类隐藏的类成员的声明，可以通过使用类名后跟作用域访问运算符加成员名字进行访问。

### 命名空间名字查找

1. 如果一个`qualified-id`的`nested-name-specifier`指示一个命名空间（包含`nestedname-specifier`是`::`的情况，指定为全局命名空间），在`nested-name-specifier`之后指定的名字在`::`前面的命名空间作用域进行查找。在`template-id`的`template-argument`出现的名字，将在`postfix-expression`完成出现的上下文中作用域进行查找。
2. 对于一个命名空间`X`和一个名字`m`，命名空间限定的查找集合`S(X,m)`定义如下：让`S'(X,m)`代表名字`m`在命名空间`X`和`X`里面`inline`命名空间中所有的声明集合。如果`S'(X,m)`不为空，那么`S(X,m)`就是`S'(X,m)`；否则，`S(X,m)`是所有 S(N<sub>i</sub>,m) 的并集，其中 N<sub>i</sub> 是在命名空间`X`及其`inline`命名空间通过`using-directives`语句引入的命名空间的集合。
3. 给定一个`X::m`（`X`是用户自定义的命名空间）或者`::m`（`X`是一个全局命名空间），如果`S(X,m)`是一个空集，那么程序是不符合规范的。如果`S(X,m)`只有一个成员，或者引用的上下文是一个`using-declaration`，`S(X,m)`是名字的必要声明集合。否则如果`m`的使用没有被集合中`S(X,m)`唯一一个声明表示，那么程序不符合规范的。
	
	例子说明：
	
	```cpp
	int x;
	namespace Y
	{
	   void f(float);
	   void h(int);
	}
	
	namespace Z
	{
	   void h(double);
	}
	
	namespace A
	{
	   using namespace Y;
	   void f(int);
	   void g(int);
	   int i;
	}
	
	namespace B
	{
	   using namespace Z;
	   void f(char);
	   int i;
	}
	
	namespace AB
	{
	   using namespace A;
	   using namespace B;
	   void g();
	}
	
	void h()
	{
	   AB::g(); // g 直接定义在命名空间 AB 中，S 是 {AB::g()} 选择 AB::g()
	   AB::f(1); // f 没有在 A 中发现，所以 规则应用到 A 和 B 上
	             // 这个时候 S = {A::f(int), B::f(char)} 不为空 选择 A::f(int)
	   AB::f('c'); // 跟上面一样，但是选择 A::f(char)
	   AB::x++;// x 没有在 AB 中定义，也没有在 A 和 B定义，同时也没在 Y, Z中定义，所以S = {} 编译错误
	   AB::h(16.8); // h 没有在AB，A，B 下定义，同样的规则应用于 Y 和 Z，所以S = {Y::h(int), Z::h(double)}
	                // 通过重载规则，选择 Z::h(double)
	}
	```
4. [*Note: 同样的声明找到多次不会造成歧义。*]
	
	例子说明：
	
	```cpp
	namespace A
	{
	   int a;
	}
	
	namespace B
	{
	   using namespace A;
	}
	
	namespace C
	{
	   using namespace A;
	}
	
	namespace BC
	{
	   using namespace B;
	   using namespace C;
	}
	
	void f()
	{
	   BC::a++; // S = {A::a, A::a}
	}
	
	namespace D
	{
	   using A::a;
	}
	
	namespace BD
	{
	   using namespace B;
	   using namespace D;
	}
	
	void g()
	{
	   BD::a++; // OK: S = {A::a, A::a}
	}
	```
5. [*Note: 因为被引用的命名空间只多被搜索一次，下面的代码片段是符合标准的。*]
	
	例子说明：
	
	```cpp
	namespace B
	{
	   int b;
	}
	
	namespace A
	{
	   using namespace B;
	   int a;
	}
	
	namespace B
	{
	   using namespace A;
	}
	
	void f()
	{
	   A::a++; // a 在 A 中定义 S = {A::a}
	   B::a++; // A 和 B 只被搜索一次 S = {A::a}
	   A::b++; // A 和 B 只被搜索一次 S = {B::b}
	   B::b++; // 直接定义在命名空间中 S = {B::b}
	}
	```
6. 在查找受限命名空间成员名字的时候，如果发现多个关于这个成员的声明，如果其中一个声明引入了一个类名字或者枚举类型的名字，其他的声明要么引用同一个变量，或者同一个枚举项或者重载函数集合，当且仅当非类型名字出现在跟类声明和枚举类型声明同一作用域时，会隐藏其名字的声明，否则程序是不符合规范的。
	
	例子说明：
	
	```cpp
	namespace A
	{
	   class x {};
	   int x;
	   int y;
	}
	
	namespace B
	{
	   class y {};
	}
	
	namespace C
	{
	   using namespace A;
	   using namespace B;
	   int i = C::x; // Ok A::x 类型 int
	   int j = C::y; // 歧义 A::y 或者 B::y 因为不在命名空间中
	}
	```
7. 在声明命名空间成员的时候，`declarator-id`是一个`qualified-id`并且具有以下的格式：
	<pre>
	nested-name-specifier unqualified-id
	</pre>
	那么`unqualified-id`将表示命名空间`nested-name-specifier`的是一个成员名字或者是该命名空间下的`inline`命名空间里面的成员。
	
	例子说明：
	
	```cpp
	namespace A
	{
	   namespace B
	   {
	      void f1(int);
	   }
	   using namespace B;
	}
	
	void A::f1(int) {} // 在定义上下文，f1不是命名空间A的成员，编译错误
	```
	然后再这种命名空间定义的时候`nested-name-specifier`依赖`using-directives`隐式提供的`nested-name-specifier`初始化部分。
	
	例子说明：
	
	```cpp
	namespace A
	{
	   namespace B
	   {
	      void f1(int);
	   }
	}
	
	namespace C
	{
	   namespace D
	   {
	      void f1(int);
	   }
	}
	
	using namespace A;
	using namespace C::D;
	
	void B::f1(int) {} // OK，定义 A::B::f1(int)
	```

## Elaborated 类型指示符

1. `elaborated-type-specifier`可能被用作去引用前面被非类型名字隐藏的类或者枚举类型的声明。
2. 如果`elaborated-type-specifier`没有`nested-name-specifier`并且除非`elaborated-type-specifier`在一个声明中按照如下形式出现
	<pre>
	class-key attribute-specifier-seq<sub>opt</sub> identifier;
	</pre>
	这个时候`identifier`按照非限定名字查找规则进行查找，但是查找的时候忽略掉非类型名字。如果`elaborated-type-specifier`通过`class-key`引入并且在前面的定义之中没有声明`type-name`或者如果`elaborated-type-specifier`在一个声明中按照如下的形式出现:
	<pre>
	class-key attribute-specifier-seq<sub>opt</sub> identifier;
	</pre>
	这个时候`elaborated-type-specifier`会在当前作用域中引入一个`class-name`的声明。
3. 如果`elaborated-type-specifier`具有一个`nested-name-specifier`，那么将按照限定名字查找规则进行查找，但是忽略非类型的名字，如果在前面没有查找到`type-name`的声明，那么`elaborated-type-specifier`是不符合规范的。
	
	例子说明：
	
	```cpp
	class Node
	{
	   class Node *Next;// OK 引用全局作用域的 Node
	   class Data *Data; // 在全局命名空间声明 Data 类型 同时定义成员 Data, todo 为什么？
	}
	
	class Data
	{
	   class Node *Node; // 引用全局 Node 类型
	   friend class ::Glob; // Glob 类型不存在，不能引入受限类型
	   friend class Glob; // OK 友元方式引用会在全局命名空间声明类型 Glob
	}
	
	class Base
	{
	   class Data; // 声明嵌套类型 Data
	   class ::Data *thatData; // 引用全局类型 ::Data
	   class Base::Data *thisData; // 引用嵌套类型 Data
	   friend class ::Data; // 引用全局类型 Data, 全局Data是个友元
	   friend class Data; // 引用嵌套类型，嵌套类型是个友元
	   class Data {}; // 定义嵌套类型 Data
	}
	
	class Data; // 重新声明全局命名空间 Data
	class ::Data; // 不能引入受限名字
	class Base::Data; // 不能引入受限类型
	class Base::Datum; // Datum 未定义
	class Base::Data *pBase; // 引用嵌套类型 Data
	```
	
## 类成员访问

1. 在类成员访问表达式中，如果`.`或者`->`后面直接跟一个后跟`<`符号的`identifier`，那么在进行名字查找的时候必须确定`<`是开始一个参数列表还是小于运算符。`identifier`首先在对象的类的名称空间寻找成员名字，如果没有找到，那么就在整个`postfix-expression`所在的上下文中寻找，这个时候`identifier`应该是类模板的名字。
2. 如果在类成员访问表达式中的`id-expression`是一个非限定名字`unqualified-id`，并且对象表达式的类型是类类型`C`，那么这个`unqualified-id`将在类类型`C`的作用域中进行查找。对于一个`pseudo-destructor`调用，`unqualified-id`将在完整的`postfix-expression`的上下文中进行查找。
3. 如果`unqualified-id`是`~type-name`，那么`type-name`在完整的后缀表达式`postfix-expression`所在的上下文中进行查找。<br/>
	如果对象表达式的类型为类型`T`是类类型`C`，`type-name`，也将在类类型`C`中进行查找。至少能查到到一个名字，它引用`cv T`。
	
	例子说明：
	
	```cpp
	class A {};
	class B
	{
	   class A {};
	   void f(::A *a);
	}
	
	void B::f(::A *a)
	{
	   a->~A(); // 通过 *a 的类型找到了`injected-class-name`
	}
	```
4. 在类成员访问表达式中的`id-expression`是一个`qualified-id`是如下的形式：
	<pre>
	class-name-or-namespace-name::...
	</pre>
	跟在`.`或者`->`后面的`class-name-or-namespace-name`，首先会在对象表达式的类类型去找，如果找到就使用，否则就在完整的后缀表达式`postfix-expression`的上下文中去寻找。
5. 如果`qualified-id`具有如下的形式：
	`::class-name-or-namespace-name::...`
	那么`class-name-or-namespace-name`会在全局命名空间中当做类名字或者命名空间名字取查找。
6. 如果`nested-name-specifier`包含了一个`simple-template-id`，那么在`template-arguments`中的名字会在完整的后缀表达式`postfix-expression`所在的上下文中去查找。
7. 如果`id-expression`是一个`conversion-function-id`，它的`conversion-type-id`在对象表达式的类型中进行查找，如果找到就使用。否则就在完整的后缀表达式所在的上下文中进行查找。在所有的这些查找中，只有类型名，或者模板特殊化得到的类型名字会被考虑。
	
	例子说明：
	
	```cpp
	class A {};
	namespace N
	{
	   class A {
	      void g() {}
	      template <typename T> operator T();
	   }
	}
	
	int main()
	{
	   N::A a;
	   a.operator A(); // 调用 N::A::operator N::A
	}
	```

## using 指令和命名空间别名

在一个`using-directive`和`namespace-alias-definition`，在查找`namespace-name`或者在`nested-name-specifier`中的名字时候，只有命名空间名字会被考虑。