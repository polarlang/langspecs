# 声明和定义

## 什么是声明

1. 一个声明可以向当前的转换单元中引入一个或者多个名字，或者重新声明当前声明前面的已经声明过的名字。声明指定了名字的解释方式和名字的相关属性，声明具有以下的效果：
	1. 产生一个静态断言
	2. 控制模板实例化（template instantiation）
	3. 辅助构造函数的模板参数的推导
	4. 指定一个名字的属性
	5. 空声明（empty-declaration）
2. 除了以下几种情况外，一个声明同时也是一个定义：
	1. 声明了函数的原型，但是没有函数体的定义
	2. 包含了`extern`指示符或者程序链接规格符（inkage-specification），但是没有任何的初始化序列（initializer）和函数体定义序列（function-body）
	3. 在类定义体里面声明一个非 inline 的静态数据成员
	4. 由`elaborated-type-specifier`产生式推导出的声明
	5. 一个由`opaque-enum-declaration`产生式推导出的声明
	6. 一个由`template-parameter`产生式推导出的函数形参的声明
	7. 是一个非函数定义的函数原型定义（function declarator）的形参声明（parameter-declaration）
	8. 一个`typedef`语句引入的声明
	9. 一个由别名产生式（alias-declaration）推导出的声明
	10. 由推导辅助产生式（deduction-guide）推导出的模板辅助推导原型声明
	11. 一个静态断言声明产生式（static_assert-declaration）推导出的声明
	12. 一个属性产生式（attribute-declaration）推导出的声明
	13. 一个空声明生成式推导出的声明（empty-declaration）
	14. 一个由使用指令产生式（using-directive）推导出的声明
	15. 一个显式模板实例化声明
	16. 一个显示式的并且不是定义的模板特殊化
	
	```
	// 一个当有程序定义的声明叫做定义，下面例子中除了一个是程序实体声明，其余的都是程序定义：
	
	int a; // 定义整形变量 a
	extern const int c = 1; // 定义一个整形变量 c
	// 定义一个函数 f
	int f(int x)
	{
	   return x + a;
	}
	
	struct S // 定义一个结构体 S 跟成员 S::a 和 S::b
	{
	   int a;
	   int b;
	};
	
	struct X // 定义一个结构体 X
	{
	   int x; // 定义一个非静态成员 X::x
	   static int y; // 声明一个静态成员 X::y
	}
	
	// todo: 极语言在这里可能有调整
	int X::y = 1; // 定义一个静态成员 X::y
	
	enum Direction // 定义一个枚举类型 `Direction`
	{
	   Up,
	   Down
	}
	
	// 定义一个命名空间 N 和一个命名空间的变量 N::d
	namespce N
	{
	   int d;
	}
	
	namespace N1 = N; // 定义一个命名空间 N1
	
	X anX; // 定义一个 X 类型的变量 anX
	
	// 以下的全部都只是程序声明：
	extern int a; // 声明 a
	extern const int c; // 声明 c
	int f(int); // 声明 f
	struct S; // 声明 S
	typedef int Integer; // 声明 Integer
	extern X anotherX; // 声明 anotherX
	using N::d; // 声明 d
	```
3. 在一定的情况下编译器会自动为类型定义默认构造函数，复制构造函数，移动构造函数，复制运算符成员函数和移动运算符成员函数。