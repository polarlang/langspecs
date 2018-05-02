# 基础概念

1. 本章节主要是定义极语言的基本概念。在这里我们将解释名字与对象之间的差异与联系，他们跟表达式的值分类之间的关系。这里我们将规范声明与定义的概念，以及极语言的各种类型表示，作用域，程序实体间的连接，和实体的生命周期的概念。我们将讨论程序的初始化和程序终止的基础机制。最后我们将描述语言内置的基本数据类型，以及怎么根据这些基础数据类型来构建复合的数据类型。
2. 本章节不包含那些只影响语言一部分的特性，这种单一的语言特性我们将在专门的章节进行讨论和规范。
3. 程序实体(entity)是如下的类型的总称：值(`value`)，对象(`object`)，引用(`reference`)，结构化绑定(`structured binding`)，函数(`function`)，枚举(`enumerator`)，数据类型(`type`)，类成员(`class member`)，模板(`template`)，模板特殊化(`template specialization`)，命名空间(`namespace`)或者形参包装(`parameter pack`)。
4. 名字是使用`identifier`，`operator-function-id`，`literal-operator-id`，`conversionfunction-id`或者`template-id`去表示程序实体(`entity`)或者标号(`label`)的一种符号。