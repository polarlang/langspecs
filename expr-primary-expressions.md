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
