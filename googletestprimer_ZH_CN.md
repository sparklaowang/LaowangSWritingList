# Googletest Primer

## 引言：为什么选择googletest？

因为*googletest*可以帮助你实现更好的C++测试。

googletest是由google的测试部门在充分考虑了google公司的一系列需求和限制约束条件的下开发一套测试框架。不管你在使用Linux，Windows，还是Mac，只要你在写C++代码，googletest就可以帮到你。而且googletest支持**任何**类型的测试，而非仅仅是云测试。

那么什么样的测试是好的呢？与此同时，googletest又是如何做到这些的呢？我们相信：

1. 测试应当是*独立*且*可重复*的。如果一个测试的成功与失败依赖于其他的测试，这无疑是一件十分痛苦的事情。googletest将各个测试作为独立的个体分别运行，当一项测试不通过的时候，googletest支持你单独运行它从而快速定位问题。

2. 测试应当具有良好的*组织结构*并能反应出被测试的代码结构。googletest将相关的测试组合成可以分享数据和子流程的测试套件（test suites）。
这种基本模式非常易读并且使得维护相对简单。
遵循这种一致性对于开发者需要切换不同的项目时显得尤其有帮助。

3. 测试应当是*可移植*且*可复用的*。Google有大量不依赖于具体平台的代码，这些代码的测试应当也是不依赖于具体平台的。googletest在不同的操作系统上、不同的编译器下、不同的配置环境下都能正常工作。

4. 当一个测试没有通过时，测试框架系统应当尽可能地提供关于问题地尽可能详细的信息。googletest并不会在第一个测试出错时停止运行整个测试，而是仅仅停止这一处测试并继续剩余的测试。当然你也可以选择让测试框架在当前测试结束时报告不致命的错误。因此在一轮测试中你可以发现并修改多处问题。

5. 测试框架应该把测试者从鸡毛蒜皮的琐事中解放出来并让他们专注于测试的*内容*。googletest不需要用户手工列出各个测试，而能自动的追踪所有被定义了的测试并执行。

6. 测试框架应该*快速*。googletest可以帮助你将各个测试共享的资源复用，从而你只需要初始化/析构一次，而不需要让测试相互依赖。

由于googletest基于非常流行的xUnit架构，因此如果你曾用过Junit或者PyUnit那么你将会感觉如同在自己家一样熟悉。如果没有，你大约需要花费十分钟时间来了解一下基础知识。那么我们开始吧

## 熟悉命名规则
注：这几个术语可能会引起一些误解：*Test*(翻译为测试), *Test Case* （翻译为测试用例），*Test Suite*（翻译为测试套件）

历史上，googletest将术语*测试用例 Test Case*定义为
一组相关的测试，而现在的出版物，例如国际软件测试资格委员会（ISTQB）的材料内容及许多软件方面的教科书，都是使用Test Suite来描述。

另一个相关的术语*测试Test*，在googletest中的含义等同于ISTQB定义的*测试用例 Test Case*。

术语*测试Test*通常指的是广义的测试，包括ISTQB所定义的*测试用例Test Case*，因此这个定义没有歧义。
但是术语*测试用例Test Case*在googleTest中的用法并不和常规意义一致，
因此可能会带来一些困惑。

googletest近来正在着手将术语*测试用例Test Case*替换为术语*测试套件Test Suite*。目前我们更支持的API是TestSuite。相对早期的TestCase API正在逐步被淘汰、重构

因此请注意术语定义上的区别：

| 含义 | googleTest术语 | ISTQB术语 |
|---------|-----------------|------------|
| 用一组特定的输入值检验程序并坚持结果| TEST() | Test Case |

## 基本概念
使用googletest的第一步是编写*assertion*（一下译为断言），
即一种确定某个条件是否为真的表述。
一个断言的结果可以是success(成功)，
nonfatal failure(非致命错误)，
或者fatal failure(致命错误)。
除了发生了致命错误时会停止运行当前函数，
否则程序将正常的继续运行。

*测试Test*使用断言来检验被测代码的行为。
如果一次测试崩溃了，或者有未通过的断言，那么这次测试*失败*，
否则成功。

一个*测试套件 Test Suite*包含一个或多个测试。
各个测试应当按照被测代码的结构编组称一个个测试套件，
当一个测试套件内的各个测试有共享的资源或者流程时，
你可以将这些资源组织成*test fixture class*
(译者注：fixture意为固定装备，固定设施，这个术语暂不做翻译)

一个*测试程序 Test Program*可以包含多个测试套件。

我们接下来从如何编写一个断言的程度开始解释如何写一个测试程序，
并逐步解释如何写测试和测试套件。

## 断言 Assertion
googletest的断言时一组看起来就像函数调用的宏。
如果你需要测试一个类或者一个函数时，
你可以通过对他的行为下断言来进行测试。
当一个断言不通过时，googletest将打印出断言定位的源代码文件和行数，
以及一些错误信息。
你可能也在googletest的默认错误信息后自行添加一些信息。

所有的断言都是成对的：一对断言在测试相同条件、相同功能时会有不同的效果：
ASSERT_\*版本的断言在不通过时会抛出致命错误并**终止当前函数**。
EXPECT_\*版本的断言在这种情况下抛出非致命错误，并且不会终止当前函数。
通常我们更推荐使用EXPECT_\*版本的函数，因为他们允许一次测试报出多余一个错误,
然而如果某一个断言不通过时继续测试就完全没有意义的时候，
你还是应该选择ASSERT_\*版本的函数。

由于ASSERT_*类型的断言在不通过时会立即返回当前函数，
因此有可能会略过一些跟在后面的清理现场的代码，
所以有可能造成泄露。
考虑到泄露的自然本质，
修复这种问题可能有价值，也有可能没有任何价值。
因此如果你在断言所提示的错误之外收到了一个堆错误，
请记住这种可能性。

你可以通过将额外信息通过```<<```操作符来打印出来，
比如下面这个例子：

```c++
ASSERT_EQ(x.size(), y.size()) << "Vectors x and y are of unequal length";

for (int i = 0; i < x.size(); ++i) {
  EXPECT_EQ(x[i], y[i]) << "Vectors x and y differ at index " << i;
}
```

任何可以流输出到ostream中的东西都可以被输出到断言宏当中，
比如C的字符串以及string对象。如果一个widestring(在windows中表现为UNICODE模式的wchar_t*, THCAR *
，或者std::wstring)被流输出到了一个断言宏当中，
他将被以UTF-8编码打印出来。

## 几种基本的断言

下面所列出的这几种断言可以做简单的是/否类型的判断：

|致命断言| 非致命断言| 判定条件|
|--------|-----------|---------|
|```ASSERT_TRUE(condition);```|```EXPECT_TRUE(condition);```|```condition```为真|
|```ASSERT_FALSE(condition);```|```EXPECT_FALSE(condition);```|```condition```为假|

请允许我再提醒一下，如果一个断言没有通过，
ASSERT_\*断言将抛出一个致命错误并返回当前函数，
而EXPECT_\*断言将抛出一个非致命错误并允当前函数继续运行。
不管是上述那种情况，如果出现了一个错误就意味着一个测试没有通过。

**可用性**：Linux，Windows，Mac
(译者注：Availability，在这里和以下出现的地方均以为可用性，指的是上文所提到的内容再特定平台上是否可用)

## 二值化比较（这个翻译很蠢，实际的意思就是产生结果是是/否的比较）

接下来这一节主要描述进行比较两个值的断言：
|致命断言| 非致命断言| 判定条件|
|--------|-----------|---------|
|```ASSERT_EQ(val1, val2);```|```EXPECT_EQ(val1, val2);	```|```val1 == val2```|
|```ASSERT_NE(val1, val2);```|```EXPECT_NE(val1, val2);	```|```val1 != val2```|
|```ASSERT_LT(val1, val2);```|```EXPECT_LT(val1, val2);	```|```val1 < val2```|
|```ASSERT_LE(val1, val2);```|```EXPECT_LE(val1, val2);	```|```val1 <= val2```|
|```ASSERT_GT(val1, val2);```|```EXPECT_GT(val1, val2);	```|```val1 > val2```|
|```ASSERT_GE(val1, val2);```|```EXPECT_GE(val1, val2);	```|```val1 >= val2```|

这些断言所比较的值必须得是可比较的，否则将会编译报错。
在旧一些的版本中我们要求被比较的值支持```<<```运算符，
这样就可以将他流输出到ostream中了，但是在现在的版本中我们不再做这一要求。
如果被比较的值支持```<<```运算符，那么断言不通过时会调用```<<```来输出这个参数，
否则googletest会尽可能合理的打印出这个值。
关于这个问题的更多细节，以及如何自己设定输出的参数，可以参考文档。

对于用户自定的类型，只有当他们支持比较运算符
（例如```==```或者```<```）
时这一类断言才能正常运行。
实际上由于google的C++ Style Guide并不推荐这种做法，
因此对于用户自定类型来说最好使用ASSERT_TRUE()或者EXPECT_TURE()这种类型的断言
来做比较。

然而，我们推荐尽可能使用ASSERT_EQ(actualm, expected)而不是ASSERT_TRUE(actuial == expected)
因为前者可以在不通过时告诉你actual和expected的实际值。

参数总是只被求值一次。
因此如果参数被求值会产生副作用并不是一件不能被接受的事情。
然而，由于对于任何传统的C/C++函数来说，
参数被求值的顺序是未定义的
（例如，编译器可以选择任何可能的顺序），
因此你的代码不应该依赖于某种特定的求值顺序。

```ASSERT_EQ()```对指针做判断时实际上是判断指针是否相等。
如果用这个断言来判断两个C风格字符串，
他将判断这两个字符串是否是在相同的地址，
而不是这两个字符串是否相同。
因此如果你想要比较两个c风格的字符串（例如const char *）
的值（译者注：也就是字符串里面的内容）是否相同，
应该用后文提到的```ASSSERT_STREQ()```。
尤其值得注意的一种情况是，当你断言一个C风格字符串```c_string```是```NULL```时，
你应该写成```ASSERT_STREQ(c_string, NULL)```。
如果你的环境支持C++11，你可以考虑替换为```ASSERT_EQ(c_string, nullptr)```。
当你要比较两个string对象的时候，应该使用```ASSERT_EQ```。

当需要做指针的比较时，可以用```*_ER(ptr, nullptr)```和```*_NE(ptr, nullptr)```
而不是```*_ER(ptr, NULL)```和```*_NE(ptr, NULL)```,
因为nullptr时有类型的而NULL没有类型（译者注：请参考[cppreference](https://en.cppreference.com/w/cpp/language/nullptr)）。
关于这一点可以查看[FAQ](https://github.com/google/googletest/blob/master/googletest/docs/faq.md)来了解更多细节。

如果你在比较浮点型的数，你可能需要用这些断言针对浮点型的变形
来避免取整带来的问题。
你也可以参考[Advanced googletst Topics](https://github.com/google/googletest/blob/master/googletest/docs/advanced.md)
来获取更多信息。

这一节所描述的宏对于narrow和wide的string对象都是适用的
（也就是```string```和```wstring```）

**可用性**：Linux，Windows，Mac。

**历史**：在2016年二月以前```*_EQ```被习惯性的写作```ASSERT_EQ(expected, actual)```
所以很多现存的代码使用这种习惯。
现如今```*_EQ```把这种两种参数用相同方式处理。

## 字符串比较

这一组断言比较的是两个**C风格字符串**。如果你想要比较两个```string```
|致命断言| 非致命断言| 判定条件|
|--------|-----------|---------|
|```ASSERT_STREQ(str1,str2);	    ```|```EXPECT_STREQ(str1,str2);	    ```| 比较的两个字符串有相同的内容|
|```ASSERT_STRNE(str1,str2);	    ```|```EXPECT_STRNE(str1,str2);	    ```|  比较的两个字符串有相同的内容|
|```ASSERT_STRCASEEQ(str1,str2);	```|```EXPECT_STRCASEEQ(str1,str2);	```|  比较的两个字符串有相同的内容, 忽略大小写|
|```ASSERT_STRCASENE(str1,str2);	```|```EXPECT_STRCASENE(str1,str2);	```| 比较的两个字符串内容不同，忽略大小写|

注意如果一个断言的名字中包含"CASE"意味着忽略大小写。
一个```NULL```指针和一个空字符串被认为是*不同的*。

```*STREQ*```和```*STRNE*```同样可以传入wide的C风格字符串（wchar_t *）.
如果一个对两个wide的字符串的比较不通过，
那么他们的内容将被以UTF-8编码的narrow字符串的形式打印出来。


**可用性**：Linux，Windows，Mac。

**另见**：关于更多字符串比较的技巧
（例如子串，前缀，后缀，正则表达式匹配）
可以参考[这份](https://github.com/google/googletest/blob/master/googletest/docs/advanced.md)关于googletest的进阶指南。

## 基础测试

三步创建一个测试：

1. 用```TEST()```宏来定义、命名一个测试函数。他们本质上是一个没有返回值的C++函数

2. 在这个函数中，你可以使用任何合法的C++语句以及一系列googletest断言来检查特定的值。

3. 测试的结果由断言决定。如果任意一个断言不通过（不管是致命还是非致命的）,或者是测试崩溃了，那么整个测试就不通过。否则通过

```C++
TEST(TestSuiteName, TestName) {
  ... test body ...
}
```

```TEST()```的参数由普遍到具体，第一个参数是测试套件的名字，
第二个参数是测试套件这这个测试的名字。
这两个名字必须是合法的C++标识符，
并且命名中不能包含下划线（_).
一个测试的全名包含组成他的测试套件的名字和他自己的名字。
来自不同的测试套件的测试可以拥有相同的自己的名字。

我们以一个简单的函数为例子：
```c++
int Factorial(int n);  // Returns the factorial of n
```
这个函数的测试套件看起来会是这样的：
```c++
// Tests factorial of 0.
TEST(FactorialTest, HandlesZeroInput) {
  EXPECT_EQ(Factorial(0), 1);
}

// Tests factorial of positive numbers.
TEST(FactorialTest, HandlesPositiveInput) {
  EXPECT_EQ(Factorial(1), 1);
  EXPECT_EQ(Factorial(2), 2);
  EXPECT_EQ(Factorial(3), 6);
  EXPECT_EQ(Factorial(8), 40320);
}
```

googletest会将测试结果按照测试套件分组，
因此逻辑上相关的测试应当被分入相同的测试套件。
换句话说，他们的```TEST()```的第一个参数应该是一样的。
再上面的例子中，我们有两个测试：
```HandlesZeroInput```和```HandlesPositiveInput```，
他们都属于相同的测试套件```FactorialTest```。

记得给你的测试、测试套件命名时应当遵循一定的
[命名规范](https://google.github.io/styleguide/cppguide.html#Function_Names)

**可用性**：Linux，Windows，Mac。

## 测试设施：让多个测试共享数据和配置
(译者注：Test Fixture，这里译为测试设施)

如果你发现你所写的多个测试在操作相似的数据，
你可以使用*测试设施*。
它允许你在多个测试间复用同一个对象的配置。

你可以按如下步骤创建一个设施：
1. 从```::testing::Test```派生一个类。在他的声明中以```protected:```开始，因为我们需要再派生类中访问设施类的成员
2. 在类中，声明你需要使用的对象
3. 如果必要的话，编写一个默认构造函数或者```SetUp()```函数来准备各个对象。一个常见的错误是吧```SetUp()```拼成小写U的```Setup()```--如果你使用C++11标准的话使用override关键字可以保证你正确拼写了（译者注：否则编译器会提示你override了并不是成员方法的函数）。
4. 如果必要的话，编写一个析构函数或者```TearDown()```函数来释放各种你在```SetUp()```中分配的资源。你可以参考[常见问题](https://github.com/google/googletest/blob/master/googletest/docs/faq.md#CtorVsSetUp)来了解什么时候应该使用构造/析构函数而什么时候应该使用```SetUp()/TearDown()```。
5. 如果必要的话，可以定义一些你的测试需要使用的子函数

当你使用一个设施的时候，你应该使用允许你访问成员以及子函数的版本```TEST_F()```而不是```TEST()```：
```C++
TEST_F(TestFixtureName, TestName) {
  ... test body ...
}
```
就如同```TEST()```，第一个参数是测试套件的名字，
而对于```TEST_F()```来说第一个参数必须是测试设施类的名字。
你可能也猜到了```_F```后缀就代表着设施(译者注：fixture)。

不幸的是，C++的宏系统不允许我们定义一个能同时处理两种类型测试的宏。
所以使用错误的宏会导致一个编译错误。

同样，你首先应当在使用```TEST_F()```之前定义一个测试设施类，
否则你将会使用得到一个编译器报错："```virtual outside class declaration```"

对于```TEST_F()```定义的每一个函数，
googletest会在运行时复制一份*全新的*测试设施实例，
立即通过```SetUp()```初始化他，
运行测试，
然后用 ```TearDown()```清理现场，
最后删除这个测试设施实例。
有一点一定要记住：
对于一个测试套件中的不同测试，
他们面对的是不同的测试设施实力，
googletest总是在建立新的测试设施实例前删除前一个。
googletest不会在多个测试间复用同一个测试设施的实例。
某个测试对设施造成的任何影响不会影响到其他的测试。

举个例子，如果我们写一个FIFO 队列的测试，名为```Queue```，
他(译者注：指的是FIFO类)的接口如下所示：
```C++
template <typename E>  // E is the element type.
class Queue {
 public:
  Queue();
  void Enqueue(const E& element);
  E* Dequeue();  // Returns NULL if the queue is empty.
  size_t size() const;
  ...
};
```
首先，定义一个设施类。根据习惯，
对于```XXX```类的测试应该被命名为```XXXTest```:
```C++
class QueueTest : public ::testing::Test {
 protected:
  void SetUp() override {
     q1_.Enqueue(1);
     q2_.Enqueue(2);
     q2_.Enqueue(3);
  }

  // void TearDown() override {}

  Queue<int> q0_;
  Queue<int> q1_;
  Queue<int> q2_;
};
```
在这个例子中，
析构函数已经做了清理现场的工作，
因此我们并不需要```TearDown()```来为每个实例清理现场。

接下里，我们来用```TEST_F()```来编写测试以及其设施：
```C++
TEST_F(QueueTest, IsEmptyInitially) {
  EXPECT_EQ(q0_.size(), 0);
}

TEST_F(QueueTest, DequeueWorks) {
  int* n = q0_.Dequeue();
  EXPECT_EQ(n, nullptr);

  n = q1_.Dequeue();
  ASSERT_NE(n, nullptr);
  EXPECT_EQ(*n, 1);
  EXPECT_EQ(q1_.size(), 0);
  delete n;

  n = q2_.Dequeue();
  ASSERT_NE(n, nullptr);
  EXPECT_EQ(*n, 2);
  EXPECT_EQ(q2_.size(), 1);
  delete n;
}
```
上面的代码同时用了```ASSERT_*```和```EXPECT_*```断言。
基本的规则是当你希望测试在某个断言不通过时可以报出更多的错误时
使用```EXPECTE_*```；而当某个断言不通过时继续测试根本没有意义
则使用```ASSERT_*```。
例如，上面的```Dequeue```中的第二个测试是```ASSERT_NE(nullptr, n)```，
因为我们之后需要释放指针```n```，
因此如果```n```是```NULL```的话会导致段错误。

当测试开始运行时，会发生下列事件：

1. googletest构造一个```QueueTest```对象（我们称其为t1）
2. t1.SetUp()将初始化t1
3. 对t1执行第一个测试（IsEmptyInitially）
4. ```t1.TearDown()```在测试结束时清理现场
5. ```t1```被析构
6. 上述流程对另一个```QueueTest```对象执行，这次执行```DqueueWorks```测试。


**可用性**：Linux，Windows，Mac。

## 触发测试

```TEST()```和```TEST_F()```会向googletest显式地注册他们的测试，
因此，不像很多其他的C++测试框架，
你不需要为了执行测试而重新列出所有测试。

在定义你的测试之后，
你可以通过```RUN_ALL_TESTS()```来执行他们，
如果所有的测试都通过了，这个宏将会返回0，否则是1.
注意```RUN_ALL_TESTS()```会执行相关模块中的*所有测试*，
即使他们在不同的测试套件，甚至不同的源文件之中。

当你调用触发了```RUN_ALL_TESTS()```的时候，他会做以下事情：
1. 存储所有Googletest Flag（译者注：指的是一系列用于标注状态的变量，以下该术语不做翻译）的状态
2. 为第一项测试创建一个测试套件对象
3. 通过```SetUp()```方法初始化他
4. 对测试套件对象执行测试
5. 通过```TearDown()```方式来清理测试套件
6. 删除测试套件
7. 恢复所有googltest flag的状态
8. 对每一个测试重复以上步骤

如果发生了致命错误，那么之后的步骤就会被跳过。

> 重要：千万不要忽略RUN_ALL_TESTS()的返回值，否则你会得到一个编译报错。
这种设计的理由在于很多自动化测试服务是通过程序的返回值来界定测试是否通过的，
而不是他的标准输出/标准错误上的输出。因此你的主函数```main()```
必须要返回```RUN_ALL_TESTS()```的返回值。

> 同时，你应该只调用一次```RUN_ALL_TESTS()```。
如果调用它多于一次将会和很多进阶的googletest特性向冲突
（例如线程安全的死亡测试），
因此我们不建议这么做。

**可用性**：Linux，Windows，Mac。

##  编写```main()```函数

绝大多数用户不会需要手动写```main()```函数，
而是连接到```gtest_main```,
他定了一个合理的入口函数。
这一节所说的内容仅仅适用于
当你需要实现一些在开始测试前客制化的流程，
且这种流程无法通过框架本身，或者测试设施、测试套件来表达的情况。

如果你需要写自己的```main```函数，他应该返回```RUN_ALL_TESTS()```的返回值。

你可以从下面的样例开始：
```C++
#include "this/package/foo.h"

#include "gtest/gtest.h"

namespace my {
namespace project {
namespace {

// The fixture for testing class Foo.
class FooTest : public ::testing::Test {
 protected:
  // You can remove any or all of the following functions if their bodies would
  // be empty.

  FooTest() {
     // You can do set-up work for each test here.
  }

  ~FooTest() override {
     // You can do clean-up work that doesn't throw exceptions here.
  }

  // If the constructor and destructor are not enough for setting up
  // and cleaning up each test, you can define the following methods:

  void SetUp() override {
     // Code here will be called immediately after the constructor (right
     // before each test).
  }

  void TearDown() override {
     // Code here will be called immediately after each test (right
     // before the destructor).
  }

  // Class members declared here can be used by all tests in the test suite
  // for Foo.
};

// Tests that the Foo::Bar() method does Abc.
TEST_F(FooTest, MethodBarDoesAbc) {
  const std::string input_filepath = "this/package/testdata/myinputfile.dat";
  const std::string output_filepath = "this/package/testdata/myoutputfile.dat";
  Foo f;
  EXPECT_EQ(f.Bar(input_filepath, output_filepath), 0);
}

// Tests that Foo does Xyz.
TEST_F(FooTest, DoesXyz) {
  // Exercises the Xyz feature of Foo.
}

}  // namespace
}  // namespace project
}  // namespace my

int main(int argc, char **argv) {
  ::testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}
```

```::testing::InitGoogleTest()```函数会解析命令行参数，
并设置googletest flag，并去除所有已经识别到的Flags
（译者注：这一句的翻译存疑）。
这允许用户通过一系列的flag来定制测试程序的行为，
具体的内容我们会在
[进阶指南](https://github.com/google/googletest/blob/master/googletest/docs/advanced.md)
中说明。
在调用```RUN_ALL_TESTS()```之前必须调用这个函数，
否则各个Flag将不会被正确的初始化。

在Windows平台上，
```InitGoogleTest()```对于所有```wide stirng```都是可用的，
因此它也可以用于以```UNICODE```模式编译的程序。

但是你可能认为编写```main()```非常麻烦，我们完全同意这一点，
这就是为什么Google Test会提供一个```main()```的基本实现。
如果他能满足你的需求，
那就直接把你的程序连接到```gtest_main```库就可以啦。

注意：```ParseGUnitFlags()```已经废弃，现在用的是```InitGoogleTest()```
