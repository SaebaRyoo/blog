> 最开始接触到var、let、const时，了解到的信息如下

var
- 会在当前执行环境中进行变量提升
- 可重复声明，可以修改
- 如果未使用var关键字声明，则会提升为全局  // 严格模式下回报错

let 
- let无法重复声明
- let有暂存死区，无法进行变量提升
- let声明的变量的作用域是块级的

const
- const声明一个常量，无法修改。但是在声明引用类型时，可以修改值。
- 其他同let

静下来想一想后，又会发下其他问题
1. 为什么var会有变量提升呢?
2. 为什么let和const有暂存死区？
3. let、const是否有变量提升？
4. 为什么var可以重复声明、修改，let不能重复声明，却可以修改，const可以修改引用类型的值？

接下来我们按照问题去解答

### 为什么var会有变量提升？
> 结论如下，这里具体可以查看[为什么js需要进行变量提升](https://segmentfault.com/q/1010000013591021)
1. 解析和预编译过程中的声明提升可以提高性能，让函数可以在执行时预先为变量分配栈空间
2. 声明提升还可以提高JS代码的容错性，使一些不规范的代码也可以正常执行


### 为什么let和const有暂存死区
首先可以看一段es6的标准
> let and const declarations define variables that are scoped to the running execution context’s LexicalEnvironment. The variables are created when their containing Lexical Environment is instantiated but may not be accessed in any way until the variable’s LexicalBinding is evaluated. A variable defined by a LexicalBinding with an Initializer is assigned the value of its Initializer’s AssignmentExpression when the LexicalBinding is evaluated, not when the variable is created. If a LexicalBinding in a let declaration does not have an Initializer the variable is assigned the value undefined when the LexicalBinding is evaluated.

_它的大概意思就是let和const会将当前执行的上下文栈形成一个封闭的作用域。在语法上，**从块顶部到该变量的初始化语句,称为 “暂时性死区”**（ temporal dead zone，简称 TDZ）。_

### let、const是否有变量提升？
> 出处[let和const有提升吗(hoist)](https://zhuanlan.zhihu.com/p/27558914)
- let 声明会提升到块顶部,
- 从块顶部到该变量的初始化语句，这块区域叫做 TDZ（临时死区）
- 如果你在 TDZ 内使用该变量，JS 就会报错（var 没有TDZ）

### 为什么var可以重复声明、修改，let不能重复声明，却可以修改，const可以修改引用类型的值
- var的话会直接在栈内存里预分配内存空间，然后等到实际语句执行的时候，再存储对应的变量，如果传的是引用类型，那么会在堆内存里开辟一个内存空间存储实际内容，栈内存会存储一个指向堆内存的指针

- let是不会在栈内存里预分配内存空间，而且在栈内存分配变量时，做一个检查，如果已经有相同变量名存在就会报错

- const也不会预分配内存空间，在栈内存分配变量时也会做同样的检查。不过const存储的变量是不可修改的，对于基本类型来说你无法修改定义的值，对于引用类型来说你无法修改栈内存里分配的指针，但是你可以修改指针指向的对象里面的属性


### 参考文档以及博客如下
[MDN var](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/var)
[MDN let](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let)
[MDN const](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/const)
[ECMA262 6.0](http://www.ecma-international.org/ecma-262/6.0/#sec-let-and-const-declarations)
