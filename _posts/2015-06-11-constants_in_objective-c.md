---
layout: post
title: Constants in Objective-C
date: June 11, 2015
category: ios
tags: ios objective-c constant const define
excerpt: 在刚开始学习编程的时候，我曾经被C里面的constant虐得死去活来。OC开发中之前也都只是模仿着用用，始终不曾认真研究过。这回，好好地探究一次。

---

关键字：`ios` `objective-c` `constant`

在刚开始学习编程的时候，我曾经被C里面的constant虐得死去活来。Objective-C作为一个C的超集，自然从C中继承了constant的概念，C里面constant的用法对于Objective-C同样适用。

### 来自C的一些用法
---

简单来说，一个变量如果用`const`修饰了，那它就变成了一个常量。在后续的程序中，再也无法被更改。但是这一简单的概念，实际用起来却并不简单。

##### 简单定义一个常量

constant最简单的用法自然就是用`const`修饰符定义一个常量，然后在代码中使用它。

```c
/* 定义一个整形的常量 */
const int myConstant = 21;
```

如果const的用法都像上面这么简单，当年我也就不会被虐得死去活来了。可惜当它和指针结合在一起的时候，就不是那么容易懂了。

```c
/* 定义一个指向整形常量的指针，指针可以重新赋值指向另一个整形常量 */
const int * myConstant1;

/* 如果把const往后移动一下，效果依然一样 */
int const * myConstant2;

/* 但是如果再移动一下，效果就不一样了。这里定义的是一个指向int的不可变指针，但是指向的int的值是可变的 */
int * const myConstant3;

/* 组合起来，就是一个指向整形常量的不可变指针 */
int const * const myConstant4;
// 或者 const int * const myConstant4;
```

根据我的理解，const会应用于紧贴在它左边的符号，当它左边没有任何符号的时候，就会应用到紧贴在它右边的符号。大家可以结合上面的四个例子自己理解一下，我感觉还是挺靠谱的。

##### 作为函数的参数

当指针和const结合在一起的时候，虽然代码会变得有点复杂，但是相应的，程序也有了更多的可能。

假如我们有下面这么一个函数，我们都知道函数里对a的改动并不会影响到b，因为b的值被拷贝了一份传给了a，它们两个在内存中的地址并不一样。

```c
void function(int a)
{
	a = 15;
}

int b = 10;
function(b);
```

那么如果有这么一个函数，又会发生什么呢？

```c
void function(BigStructureType a)
{
	// 利用a进行一些逻辑，但并不会改动a
}

BigStructureType b = SomeValue;
fuction(b);
```

尽管BigStructureType会很大，程序仍然会拷贝一份b的值，作为参数a传给函数，然后进行一系列的操作。但是，结构的内容在函数中并不会被改动，我们为什么不用一份数据呢，这样就可以少用一点内存了。于是，constant就派上用场了。

```c
void function(BigStructureType const *a)
{
	// 可以通过 *a 获取到数据，然后进行一系列只读操作
}

BigStructureType b = SomeValue;
fuction(&b);
```

##### 更多用途

我觉得一个东西的用途并不是固定的，只有了解一个东西的特性，你才能知道在你想做某件事情的时候，它是不是你的选择。用const定义了一个常量后，不可变的特性让它能够作为全局的识别符，通用的一些数值，或者便于修改的统一值。而当constant和指针结合在一起后，可能性就更多了，除了上面所说的函数参数，比如还可以用来修饰函数的返回值，避免改动一个不可改动的值。至于更多的内容，就让我们自己慢慢发掘吧。


### Objective-C中的常量
---

在OC开发中，所有来自C的const的用法同样适用。不过在平常的开发中，我们用的并不会很多，最常见的应该就是用来定义一些dictionary的key了。

##### 常见的用法

```objc
// 用法一，全局可能会用到的key
/* 在.h文件中声明 */
extern NSString * const kMyExampleKey;

/* 在.m文件中定义 */
NSString * const kMyExampleKey = @"xxxxxxxx";


// 用法二，单个文件中用到的key
static NSString * const kMyExampleKey = @"xxxxxxxx";
```

类似上面这样的用法，相信大家一定看到过不少，因为系统的key基本也是这么定义的，比如用来获取键盘信息的`UIKeyboardFrameEndUserInfoKey`等。那么，不知道有没有人在自己模仿着定义的时候，写成了这样呢？

```objc
/* 错误用法 */
const NSString * kMyWrongExampleKey1 = @"xxxxxxxx";
NSString const * kMyWrongExampleKey2 = @"xxxxxxxx";
```

如果你已经知道上面用法为什么是错误的，那么就可以跳过这一节了。之前在`简单定义一个常量`小节中，我提到了const的作用方式：

>根据我的理解，const会应用于紧贴在它左边的符号，当它左边没有任何符号的时候，就会应用到紧贴在它右边的符号。

所以在上面的错误用法中，const都作用在了NSString上，而我们都知道，在OC中，可修改和不可修改的字符串被分为了`NSMutableString`和`NSString`。NSString本身就是不可修改的，const并不会产生什么作用。而在定义key的时候，我们本身的目的是想让key指向的字符串不要发生变化，显然应该让const用于修饰指针。

除了用于定义key，const还可以定义一些固定的数值，比如tag，某个view固定的高度，固定的间距等。

```objc
static const int kTextFieldTag = 1111;
static const int kCellHeight = 100;
static const int kVerticalPadding = 5;
```

用const来定义这些数值，可以让代码中少一些magic number，增加代码的可读性。而且当需要更改某个数值的时候，仅需要轻松地改动一次。话说，你能够想象当你想把padding改成3的时候，却发现代码中各种不知道是不是padding的5的时候的感觉吗？

##### define VS. const

上面提到的const的用法，为我们日常工作提供了很多便利，但const并不是唯一的选择，有些时候，define可能会是更好的选择。

`#define`是用于宏定义的，定义的内容会在编译器工作前，就被批量替换掉。

```objc
#define M_E         2.71828182845904523536028747135266250
#define M_PI        3.14159265358979323846264338327950288
```

我无法准确地告诉你哪里要用const，哪里应该用define，我只能帮你分析一下各自的特性，具体要用哪个就要看你自己的取舍了。

`const`定义的常量是会占用内存空间的，如果是临时的还好，出了作用域就会被释放，但是如果是全局的，那么就会一直占用着内存空间。不幸的是，大多数你需要使用的场景，都会是全局的。幸运的是，以目前的设备状况来看，这么点内存的占用，应该是无关痛痒的。不过注意一点总是没错的，不要滥用。而`define`因为在编译前就会被批量替换掉，所以并没有这个问题。

说完了const的缺点，来说说const的优点。使用const定义的常量，是有确定的类型的，编译器会帮你做类型的检测，如果使用错误能够及时发现并改掉。而define虽然IDE也会帮着做一些检测，但是遇到问题的可能性会大上不少，比如下面这个场景：

```objc
/* define 版本 */
#define kTestValue 1
CGFloat testResulte = kTestValue / 2;

/* const 版本 */
const CGFloat kTestValue = 1;
CGFloat testResult = kTestValue / 2;

```

define的版本得到的结果是0，而const版本是0.5。上面只是一个简单的例子，实际使用过程中肯定不会这么简单，复杂的代码逻辑可能会把define导致的问题隐藏起来，增加寻找bug的难度。当然，如果你注意一点，定义的时候写成`#define kTestValue ((CGFloat)1)`或者`#define kTestValue 1.0`就没有问题了。

另外const还有一个比较大的好处就是，可以更快捷地进行比较。如果用const来定义key，就不需要对字符串的内容进行比较了，只需要比较两个key的地址就可以了，可以大大增加比较的速度。

在我平常的使用过程中，我一般用const来定义一些key，而用define来定义一些固定的数值。至于你怎么用，就看你的心情喽！

##### 被隐藏的常量

最后来说点题外话，OC里面有个`NSString`的子类叫`__NSCFConstantString`，大家可以结合下面这段代码自己尝试着研究一下。

```objc
/* 用于输出变量名、类名和地址的宏 */
#define TestInfo(obj) \
do{id _obj = ( obj );\
printf("%-61s %-20s %-16p\n", #obj" :", [NSStringFromClass([_obj class]) UTF8String], _obj);\
}while(0);

/* 定义了5个内容为“abcdef”的NSString指针 */
NSString * const test1 = @"abcdef";
NSString * const test2 = @"abcdef";
NSString * test3 = @"abcdef";
NSString * test4 = [@"abcdef" copy];
NSString * test5 = [NSString stringWithFormat:@"abcdef"];

/* 分别输出5个变量的信息 */
TestInfo(test1);
TestInfo(test2);
TestInfo(test3);
TestInfo(test4);
TestInfo(test5);

/* 你们觉得结果会是怎样的呢？又有几个变量的地址会是一样的呢？ */
```


