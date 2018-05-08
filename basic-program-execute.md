# 程序执行
## 顺序执行
1. 具有自动存储周期的对象的实例跟其所在的代码块的各种出现的入口相关联。这种类型的对象存在并保持最后一次赋值的状态在当代码执行和或者所在代码块被挂起（通过一个新的函数调用或者接受到操作系统的一个信号）。
2. 一个`constituent`表达是的定义如下：
	1. 一个表达式的`constituent`表达式是其自身那个表达式。
	2. `braced-init-list`语句或者`expression-list`（可能有扩展）的`constituent`表达式是相应的列表中的元素成员的`constituent`表达式。
	3. `= initializer-clause`形式的`brace-or-equal-initializer`的`constituent`表达式是其`initializer-clause`的`constituent`表达式。
	
	例子说明：
	```cpp
	class A { int x; };
	class B { int y; class A a; };
	B b = { 5, { 1+1 }};
	```
	用于初始化对象`b`的`initializer`的`constituent`表达式`5`和`1+1`。
3. 表达式`e`的直接子表达式（`immediate subexpressions`）是：
	1. 表达式`e`的操作数的`constituent`表达式。
	2. 由表达式`e`隐式调用的函数调用。
	3. 如果表达式`e`是一个`lambda-expression`，那些通过复制类型捕获的实体的初始化和`init-captures`的`initializer`。
	4. 如果表达式`e`是一个函数调用或者隐式函数调用，在每个调用里面使用的默认参数的`constituent`表达式。
	5. 如果表达式`e`创建一个聚合对象，多于用于成员默认初始化的`constituent`表达式。
4. 一个表达式`e`的子表达式是`e`的直接子表达式或者`e`表达式的直接表达式的子表达式。[*Note: 出现在`lambda-expression`里面的`compound-statement`不是`lambda-expression`的子表达式。*]
5. 如果满足一下情况的表达式被称作完整表达式：
	1. 没有被求值的操作数。
	2. 一个常表达式。
	3. 一个`init-declarator`或者一个`mem-initializer`，包含`initializer`的`constituent`表达式。
	4. 除了临时对象变量之外的当生命周期结束之时的析构函数的调用表达式。
	5. 一个不是任何其他表达式的子表达式的表达式并且不是完整表达式的其他部分。
	如果一个语言结构是为了产生一个隐式的函数调用，那么这个语言构造按照其定义可以理解为一个表达式。为了满足语言构造语句的要求而在一个表达式的结果上进行的类型转换是完整表达式的一部分。对于`initializer`对实体进行初始化的`initialization`（包含给聚合类型的成员进行默认初始化的`initializers`表达式）也是完整表达式的一部分。
	
	例子说明：
	```cpp
	class S
	{
	   S(int i) : I(i) {} // 完整表达式是 I 的初始化语句
	   int &v() { return I; }
	   ~S() noexcept(false) {}
	private:
	   int I;
	};
	
	S s1(1); // 完整语句是 S::S(int) 构造函数调用
	void f()
	{
	   S s2 = 2; // 完整语句是 S::S(int) 构造函数调用
	   if (S(3).v()) {} // 完整的表示是 左值到右值的转换和整形到boolean类型的转换
	   					   // 在完整表达式结束之前进行
	   bool b = noexcept(S()); // exception specification of destructor of S considered for noexcept
	   // 完整表达式是 s2 在代码块结尾处的析构函数的调用
	}
	
	class B
	{
	   B(S = S(0));
	};
	
	B b[2] = { B(), B() }; // 完整表达式是整个初始化表达式，包含临时对象的析构表达式
	```
	6. [*Note: 完整表达式的执行可能包含那些不在文法范围里面的子表达式。比如给函数提供默认参数的表达式，就不在函数表达式的文法范围里面。*]
	7. 读取`volatile glvalue`的对象，修改一个对象，调用的一个标准库中的`IO`函数，或者调用一个具有额外作用的函数，比如修改执行环境的状态。对一个表达式或者子表达式进行求值包含值的计算（包括在`glvalue`求值的时候的对象的标识符的确定或者`prvalue`求值的时候获取对象的上一次赋值。）和额外作用的初始化。当调用库函数里面的IO函数返回或者访问一个已经求值`volatile glvalue`，代表额外作用已经完成。尽管函数调用隐含的一个额外的操作还没有完成，比如`IO`本身还没有完成，或者通过`volatile`访问可能还没有完成。[*todo: 这个地方理解不够清晰。*]
	8. `Sequenced before`是一个非对称的，传递性的，`pair-wise`的单线程环境中的两个求值的执行语句之间的关系。它在这些求值过程里面引入了一个半序关系。
	9. 所有的值计算和完整表达式关联的额外效果计算`sequenced before`下一个完整表达式的值计算和完整表达式关联的额外效果计算。
	10. 没有额外的说明的话，不同的运算符的操作符之间的求值或者不同表达式的子表达式之间的求值不是顺序的。[*Note：在一个多此被求值的表达式的子表达式的求值在两次求值的过程中，不是顺序的并且顺序不确定*]运算符的操作数求值`sequenced before`在运算符的结果求值之前。如果一个在内存位置的一个额外作用相对另一个在同样内存位置的额外作用不是顺序的话或者在计算的时候使用同一内存位置的任何对象并且他们不是并发执行的，程序的行为是未定义的。[*Note: 下一章节规定一个更复杂的并发计算情况的限制。*]
	
	例子说明：
	```cpp
	void g(int i)
	{
	   i = 7, i++, i++; // i 值是 9
	   i = i++ + 1; // i 的值增加
	   i = i++ + i; // 行为是未定义的
	   i = i + 1; // i 的值增加
	}
	```
	11. 当调用的一个函数的时候（不管这个函数是不是`inline`函数。）每个每个参数表达式值的计算或者关联的额外作用或者整个函数调用的后缀表达式`sequenced before`被调用的函数体里面的每个表达式或者语句。对于每个函数调用`F`，对于那些在函数体里面的求值`A`和那些不在函数体里面的求值`B`，但是他们在同一个线程的同一个信号处理器中求值。这个时候要么`A sequenced before B`或者`B sequenced before A`。[*Note：如果`A`跟`B`不是顺序求值，那么他们的顺序是不确定的。*]有些极语言的上下文中会对函数进行求值，即使没有对应的函数调用语句在转换单元里面。[*Example: 比如`new-expression`调用一个或者多个存储空间分配函数和构造函数。另一个例子就是在有些上下文中会自动调用类型转化函数。*]被调用函数的求值序列的限制是一种函数调用时的特性，而不管是什么样的形式来出发这样的函数调用。也就是说不是文法上面的特性，是运行时的限制。
	12. 如果调用`polar::raise`函数导致信号处理函数被执行。那么信号处理器执行是`sequenced after`在函数调用`polar::raise`并且在其返回之前。[*Note: 如果信号是通过其他原因获取的，信号的处理器的执行跟程序其他部分的执行顺序不是序列进行的。*]

## 多线程执行和数据的竞态条件

1. 执行线程（简称线程）是程序中的一个控制流程。包含调用指定的顶层函数，和所有的随后被线程执行的所有的函数调用。[*Note：当一个线程创建另一个线程的时候，顶层函数的的初始化调用在新创建的线程中进行调用，而不是在线程创建者中调用。*]一个程序中的所有线程都可能去访问程序中的每一个对象和函数，在特定的宿主环境中，一个极语言可以有多个并发执行的线程。每个线程的执行由下面的文档进行定义。一个程序的执行包含了程序中所有线程的执行。[*Note: 通常执行可以看着所有线程交叉执行的情景。然后，一些原子操作，举个例子，可以在一些简单的交叉执行中保持不一致，比如后面的例子。*] 在一个没有宿主环境（freestanding）的执行环境中，由编译器决定是否执行多个执行线程。
2. 对于一个不是由`polar::raise`函数引起的事件处理器来说，由那个线程来执行时间处理器函数是未指定的。

### 数据的竞态条件

1. 一个对象的值对一个线程`T`在特定的点可见是这个对象的初始值，线程`T`给对象赋一个新的值，或者这个对象被其他线程赋一个新的值，需要根据下面的规则进行。[*Note: 在某些情况下，可以不用出现未定义的情况。在本章中，我们倾向通过显式和详尽可见的限制来支持原子操作，然后，对于严格限制的程序来说有隐式的简单的视图来支持原子操作。*]
2. 如果一个表达式正在修改一个内存位置，另一个表达式在读取或者修改一个内存位置，那么这两个表达式是冲突的。
3. 标准库定义了一系列的原子操作和在互斥量上的操作，这两个操作叫做同步操作。这些操作发挥了重要的作用，让一个线程的修改让另一个线程可见。在一个或者多个内存位置的同步操作要么是一个`consume`操作，一个`acquire`操作，一个`release`操作或者既是`acquire`操作又是`release`操作。一个不与内存位置的同步操作叫做`fence`，要么是`acquire fence`，`release fence`或者同时是`acquire fence`和`release fence`。进一步，有不严格的原子操作（`relaxed atomic operations`），他不是一个同步操作，还有一些具备特殊特性的原子`read-modify-write`操作。[*Note: 举个例子，一个获取互斥量的操作，将会在组成互斥量的位置进行一个`acquire`操作，对应在对一个互斥量进行释放操作，活在组成互斥量的位置进行一个`release`操作。非形式化的说，在`A`处进行一次`release`操作，将会让在`A`点之前对内存位置的修改让一个对同一个互斥量进行`consume`或者`acquire`操作的线程可见。`Relaxed`原子操作尽管看着很像同步操作，但是其不是同步操作，对数据竞态条件的解决没有任何帮助。*]
4. 对一个原子对象`M`所有的操作，组成了一个特定的全局操作顺序。叫做原子对象`M`的修改顺序。[*Note: 不同的原子对象有不同的修改顺序。是否需要对所有的修改顺序合成一个没有做任何规定。通常来说，这么多是不可能的，不同的线程可能观察到的不同对象的修改顺序是不一致的。*]
5. 一个释放序列（`release sequence`）是在原子对象`M`上以一个释放操作`A`开头（`release operation`）后跟一个最大长度的额外作用的修改原子对象的顺序，第一个操作是`A`，后续的每个操作应该满足下面的条件之一：
	1. 必须跟操作`A`在同一个线程。
	2. 是一个原子的`read-modify-write`操作。
6. 特定的标准库中的函数调用跟另一个线程的标准库的函数调用具有`synchronize with`的关系。举个例子：一个原子的`store-release`操作跟一个原子的`load-acquire`在同一个原子对象想具备`synchronize with`的关系。[*Note: 同步操作的规范定义了读取一个由其他线程写的值。对于原子操作来说，这个定义很清晰。对于一个在互斥量上的所有操作具备一个全局的顺序。每一个互斥量的获取是读取最后一个互斥量被写入的值。*]
7. 一个求值`A`跟`B`具有`carries a dependency`如果下面条件之一满足：
	1. `A`的值作为`B`的操作数，除非下面的情况之一满足：
		1. `B`是一个特殊的调用`polar::kill_dependency`。
		2. `A`是操作符`&&`和`||`的左边操作数。
		3. `A`是三元操作符最左边操作数。
		4. `A`是内置的逗号操作符的左边操作数。
	2. `A`在一个标量或者位域`M`中写入一个值，`B`从`M`中读取`A`写入的值并且`A sequenced before B`。
	3. 对于有些求值`X`，`A`跟`X`具有`carries a dependency`的关系，`X`跟`B`具有`carries a dependency`的关系。
8. 一个求值`A`跟一个求值`B`具有`dependency-ordered before`的关系，如果下面任意条件满足：
	1. `A`在原子对象上进行一个`release`操作，其他的线程的一个操作`B`在同一个原子对象上进行一个`consume`操作并且读取任何在`A`之前对原子对象`M`进行的任何额外操作。
	2. 对于一些求值`X`，`A dependency-ordered before X`并且`X carries a dependency to B`。
9. 一个求值操作`A`跟求值操作`B`具有`inter-thread happens before`的关系，只要满足下面的任意一个条件：
	1. 求值操作`A`跟求值操作`B`具有`synchronizes with`的关系。
	2. 求值操作`A`跟求值操作`B`具有`dependency-ordered before`的关系。
	3. 对于某些求值操作`X`：
		1. A synchronizes with X 并且 X is sequenced before B。
		2. A is sequenced before X 并且 X inter-thread happens before B。
		3. A inter-thread happens before X 并且 X inter-thread happens before B。
	[*Note: `inter-thread happens before`关系描述了任意的`sequenced before`，`synchronizes with`，`dependency-ordered before`的串连关系，除了下面的两种情况。第一个排除情况是，`dependency-ordered before`后面跟着`sequenced before`的串连是不允许的。这个限制的原因是参与`dependency-ordered before`关系的`consume`操作仅仅保证参与这个关系的对象以及依赖的对象提供顺序保证。第二个意外情况是，串连不能全部由`sequenced before`关系组成。这个限制的原因是：（1）允许`inter-thread happens before`关系能够关闭，（2）下面定义的`happens before`已经囊括了串连中全部是`sequenced before`的情况。*]
10. 一个求值操作`A`跟求值操作`B`具有`happens before`的关系，或者说一个求值操作`B`跟求值操作`A`具有`happens after`的关系，如果下面的条件之一满足：
	1. A is sequenced before B。
	2. A inter-thread happens before B。
	编译器在这里应该确保程序中的`happens before`不出现循环的情况。[*Note: 这种循环只能出现在`consume`操作中。*]
11. 一个求值操作`A`跟求值操作`B`具有`strongly happens before`的关系，如果下面条件之一满足：
	1. A is sequenced before B。
	2. A synchronizes with B。
	3. A strongly happens before X`并且`X strongly happens before B。
	[*Note: 去掉了`consume`操作，`happens before`就跟`strongly happens before`是一致的。`strongly happens before`排除了`consume`操作。*]
12. 一个在标量对象或者位域`M`上的可见的额外效果`A`跟在`M`上的`B`操作有关系必须满足以下所有条件：
	1. A happens before B。
	2. 没有一个在`M`上的额外效果，满足 A happens before X 并且 X happens before B。
	非原子的标量和位域`M`的值在求值操作`B`的时候确定的值必须是可见额外效果`A`存入的值。[*Note：如果对非原子的对象或者位域存在一个歧义的额外效果，程序行为是为指定的或者是未定义的。*][*Note: 这个指出了在普通对象上的操作被不可见的重新安排顺序了。这个通常不能检测到是否有数据竞态条件。但是有时必须要确定相关的数据竞态条件，通过合理的使用原子变量，以应对简单交叉执行中的数据竞态问题。*]
13. 对于一个原子对象`M`，如果`B does not happen before A`，那么数据操作`B`读到的数据应该是数据操作`A`的额外效果存入`M`的值。[*Note: 这里是的额外操作的集合需要做相关的限制，特别的有一些下面描述的连贯性限制。*]
14. 如果一个数据操作`A`修改一个原子对象`M`跟数据操作`B`修改一个同一个原子对象`M`具有`happens before`的关系，那么数据操作`A`的修改顺序在数据操作`B`的修改顺序之前发生。[*这个限制叫做写-写连贯性操作（`write-write coherence`）*]
15. 如果在原子对象上的数据读操作`A`跟数据读操作`B`具有`happens before`的关系，数据操作`A`从原子对象`M`上获取某次数据操作`X`存入的值，然后数据操作`B`在原子对象`M`获取的值，要么是数据操作存入的值，或者是某次额外效果`Y`在原子对象`M`存入的值，在`M`的修改顺序中，数据操作`Y`跟在数据操作`X`的后面。[*Note: 这个限制称做读-读连贯性（`read-read coherence`）。*]
16. 如果一个在原子对象`M`上的数据读操作`A`跟数据写操作`B`具有`happens before`的关系，那么数据操作`A`应该从原子对象上的某次额外效果`X`存入的值，并且在`M`的修改顺序中`X`在`B`之前。[*Note: 这个限制称作读-写连贯性（`read-write coherence`）。*]
17. 如果一个在原子对象`M`上的数据写操作`A`跟数据读操作`B`具有`happens before`的关系，那么在数据读操作`B`中获取的值要么是在数据操作`A`中写入原子对象中的值，要么是一个在原子对象的修改顺序中在数据操作`A`后面的数据写入操作`X`写入的值。[*Note: 这个限制称作写-读连贯性（`write-readcoherence`）。*]
18. [*Note：上面四条连贯性的限制，禁止了编译器改变单个原子对象的原子操作顺序。即使两个操作都是`relaxed`读取。这个也让底层硬件一共的原子操作缓存的连贯性得到了有效的利用。*]
19. [ Note: The value observed by a load of an atomic depends on the “happens before” relation, which depends on the values observed by loads of atomics. The intended reading is that there must exist an association of atomic loads with modifications they observe that, together with suitably chosen modification orders and the "happens before" relation derived as described above, satisfy the resulting constraints as imposed here.—end note ]

### 前置处理


## 开始执行和终止

### main 函数

### 静态初始化

### 非局部变量的动态初始化

### 程序终止


