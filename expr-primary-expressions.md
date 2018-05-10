# 主表达式

<pre>
primary-expression:
	literal
	this
	(expression)
	id-expression
	lambda-expression
	fold-expression
	requires-expression
</pre>

## 字面量

1. 一个`literal`是一个主表达式，它的类型依赖它的形式。除了字符串字面量的分类是`lvalue`；所有其他的字面量的分类都是`prvalue`。

## 关键字 this

1. 关键字`this`是一个命名的指向一个对象的指针，用与在对象的非静态函数成员的调用或者对象的非静态数据成员的初始化器的求值。
2. 如果在一个类类型`X`的成员函数的或者成员模板函数的声明中，`this`表达式在可选的`cv-qualifier-seq`和在`function-definition`，`member-declarator`或者`declarator`的结尾这个范围里面的类型是指向`cv-qualifier-seq X`的指针。它不应该出现在可选`cv-qualifier-seq`之前，也不应该出现在静态成员函数声明中。[*Note: 即使`this`表达式的类型和分类在静态函数成员中跟非静态函数成员中一样。*][*Note: 在一个`trailing-return-type`，对于类类型成员的访问不要求该类类型是完整的类型。类类型成员的声明稍后不可见。*]
	
	例子说明：
	
	```cpp
	class A
	{
	   char g();
	   template <typename T> auto f(T t) -> decltype(t + g())
	   {
	      return t + g();
	   }
	};
	
	template auto A::f(int t) -> decltype(t + g());
	```
3. 否则，如果`member-declarator`声明一个类类型`X`的非静态数据成员，那么在可选的默认成员初始化器中，`this`表达式分类是`prvalue`并且分类的类型是`X *`类型。它不能出现在`member-declarator`的其他地方。
4. `this`表达式不应该出现在上面说的之外的其他地方：
	
	例子说明：
	
	```cpp
	class Outer
	{
	   int a[sizeof(*this)]; // 错误，没有出现在成员的默认初始化器中
	   unsigned int sz = sizeof(*this); // 可以，出现在成员的初始化器中
	   
	   void f()
	   {
	      int b[sizeof(*this)]; // OK
	      
	      struct Inner
	      {
	         int c[sizeof(*this)]; // 错误，没有在局部类型的成员默认初始化器中
	      };
	   }
	}
	```

## 括号

1. 被加上括号的表达式`(E)`是一个主表达式，这个表达式的类型，值和值的分类跟表达式`E`保持一致。表达式`(E)`能够使用的上下文环境跟表达式`E`一样，同时表示的意义也是一样，除非另有说明。

## 名字

<pre>
id-expression:
	unqualified-id
	qualified-id
</pre>

1. 一个`id-expression`是一个受限形式的`primary-expression`。[*Note: 一个`id-expression`可以出现在`.`和`->`运算符的后面。*]
2. 如如一个`id-expression`被用作指示一个类类型的非静态数据成员或者非静态函数成员，那么它只能按照下面的方式进行使用：
	1. 作为类类型成员访问表达式的一部分，其中对象表达式表示成员所在的类类型或者一个从其派生得到的类类型，或者
	2. 组成一个指向成员的指针类型表达式，或者
	3. 如果`id-expression`指示了一个非静态的数据成员并且用在了非求值的操作数表达式中。
	
	例子说明：
	
	```cpp
	class S
	{
	   int m;
	};
	
	int i = sizeof(S::m); // OK
	int j = sizeof(S::m + 42); // OK
	```
3. 如果一个`id-expression`表达式指示的是一个概念类型（`concept`）的特殊化，并且那个概念类型的结果是一个分类的值类型是`bool`类型的`prvalue`分类。那么表达式为`true`，如果指定的模板类型实参满足了概念的`constraint-expression`，否则表达式的结果是`false`。
	
	例子说明：
	
	```cpp
	template<typename T> concept C = true;
	static_assert(C<int>); // OK
	```
	[*Note: 概念限制在使用一个模板名字，函数重载的选择并且这些他们在一个限制的半序排序中被比较。（`17.4.4`）*]
4. 如果一个程序显式或者隐式的引用一个尾部具有`requires-clause`分句的函数，并且其限制没有被满足，那么除了声明这个函数，其他任何使用，程序都是不符合规范的。
	
	例子说明：
	
	```cpp
	void f(int) requires false;
	void g() {
	   f(0); // error: cannot call f
	   void (*p1)(int) = f; // error: cannot take the address of f
	   decltype(f)* p2 = nullptr; // error: the type decltype(f) is invalid
	}
	```
	在每一种情况中函数`f`的限制都没有被满足，在声明`p2`中，尽管是用在非求值的操作数中，函数`f`的这些限制也是会被考虑。
	
