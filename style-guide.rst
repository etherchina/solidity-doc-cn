.. index:: style, coding style

#############
风格指南
#############

************
概述
************

本指南旨在约定solidity代码的编码规范。本指南是不断变化演进的，旧的、过时的编码规范会被淘汰，
而新的、有用的规范会被添加进来。

许多项目会实施他们自己的编码风格指南。如遇冲突，应优先使用具体项目的风格指南。

本风格指南中的结构和许多建议是取自python的 `pep8 style guide <https://www.python.org/dev/peps/pep-0008/>`_。

本指南并 *不是* 以指导正确或最佳的solidity编码方式为目的。本指南的目的是保持代码的 *一致性* 。
来自python的参考文档`pep8 <https://www.python.org/dev/peps/pep-0008/#a-foolish-consistency-is-the-hobgoblin-of-little-minds>`_。
很好的阐述了这个概念。

    风格指南是关于一致性的。与此风格指南的一致性非常重要。项目中的一致性更重要。一个模块或功能内的一致性是最重要的。
    但最重要的是：知道什么时候不一致 ——有时风格指南不适用。如有疑问，请自行判断。看看其他例子，并决定什么看起来最好。并应毫不犹豫的询问他人！

***********
代码结构
***********


缩进
===========

每个缩进级别使用4个空格。

制表符或空格
==============

空格是首选的缩进方法。

应该避免混合使用制表符和空格。

空行
===========

在solidity源码中合约声明之间留出两个空行。


正确写法::

    contract A {
        ...
    }


    contract B {
        ...
    }


    contract C {
        ...
    }

错误写法::

    contract A {
        ...
    }
    contract B {
        ...
    }

    contract C {
        ...
    }

在一个合约中的函数声明之间留有一个空行。

在同一合约中的一组函数声明之间可以省略空行（例如抽象合约的存根函数）

正确写法::

    contract A {
        function spam() public;
        function ham() public;
    }


    contract B is A {
        function spam() public {
            ...
        }

        function ham() public {
            ...
        }
    }

错误写法::

    contract A {
        function spam() public {
            ...
        }
        function ham() public {
            ...
        }
    }

源文件编码格式
====================

首选UTF-8或ASCII编码。

Imports规范
=======

导入语句应始终放在文件的顶部。

正确写法::

    import "owned";


    contract A {
        ...
    }


    contract B is owned {
        ...
    }

错误写法::

    contract A {
        ...
    }


    import "owned";


    contract B is owned {
        ...
    }

函数顺序
==================

排序有助于读者识别他们可以调用哪些函数，并更容易地找到构造函数和回退函数的定义。

功能应根据其可见性和顺序进行分组：

- 构造函数
- 回退函数定义（如果存在）
- 外部函数
- 公共函数
- 内部函数和变量
- 私有函数和变量

在一个分组中，最后放置``常量``函数。

正确写法::

    contract A {
        function A() public {
            ...
        }

        function() public {
            ...
        }

        // External functions
        // ...

        // External functions that are constant
        // ...

        // Public functions
        // ...

        // Internal functions
        // ...

        // Private functions
        // ...
    }

错误写法::

    contract A {

        // External functions
        // ...

        // Private functions
        // ...

        // Public functions
        // ...

        function A() public {
            ...
        }

        function() public {
            ...
        }

        // Internal functions
        // ...
    }

表达式中的空格
=========================

在以下情况下避免无关的空格：

除单行函数声明外，中括号或者大括号中的立即数应该避免空格。

正确写法::

    spam(ham[1], Coin({name: "ham"}));

错误写法::

    spam( ham[ 1 ], Coin( { name: "ham" } ) );

除外::

    function singleLine() public { spam(); }

紧接在逗号，分号之前：

正确写法::

    function spam(uint i, Coin coin) public;

错误写法::

    function spam(uint i , Coin coin) public ;

赋值或者其他操作符两边用于对齐的多个空格：

正确写法::

    x = 1;
    y = 2;
    long_variable = 3;

错误写法::

    x             = 1;
    y             = 2;
    long_variable = 3;

回退函数中不要包含空格：

正确写法::

    function() public {
        ...
    }

错误写法::

    function () public {
        ...
    }

控制结构

==================

用大括号表示一个合约，库、函数和结构
应该：

* 开括号与声明应在同一行
* 闭括号在与之前函数声明对应的开括号保持同一缩进级别上另起一行.
* 开括号前应该有一个空格。

正确写法::

    contract Coin {
        struct Bank {
            address owner;
            uint balance;
        }
    }

错误写法::

    contract Coin
    {
        struct Bank {
            address owner;
            uint balance;
        }
    }

对于控制结构 ``if``， ``else``， ``while``， ``for`` 的实施建议与以上相同。

另外，诸如 ``if``， ``else``， ``while``， ``for`` 这类的控制结构和条件表达式的块之间应该有一个单独的空格，
同样的，条件表达式的块和开括号之间也应该有一个空格。

正确写法::

    if (...) {
        ...
    }

    for (...) {
        ...
    }

错误写法::

    if (...)
    {
        ...
    }

    while(...){
    }

    for (...) {
        ...;}

对于控制结构，如果其主体内容只包含一行，则可以省略括号。

正确写法::

    if (x < 10)
        x += 1;

错误写法::

    if (x < 10)
        someArray.push(Coin({
            name: 'spam',
            value: 42
        }));

对于具有 ``else`` 或 ``else if`` 子句的 ``if`` 块， ``else`` 应该是与 ``if`` 的闭大括号放在同一行上。 这一规则区别于
其他块状结构。

正确写法::

    if (x < 3) {
        x += 1;
    } else if (x > 7) {
        x -= 1;
    } else {
        x = 5;
    }


    if (x < 3)
        x += 1;
    else
        x -= 1;

错误写法::

    if (x < 3) {
        x += 1;
    }
    else {
        x -= 1;
    }

函数声明
====================

对于简短的函数声明，建议函数体与函数声明保持在同一行。

闭大括号应该与函数声明的缩进级别相同。

开大括号之前应该有一个空格。

正确写法::

    function increment(uint x) public pure returns (uint) {
        return x + 1;
    }

    function increment(uint x) public pure onlyowner returns (uint) {
        return x + 1;
    }

错误写法::

    function increment(uint x) public pure returns (uint)
    {
        return x + 1;
    }

    function increment(uint x) public pure returns (uint){
        return x + 1;
    }

    function increment(uint x) public pure returns (uint) {
        return x + 1;
        }

    function increment(uint x) public pure returns (uint) {
        return x + 1;}

函数的可见性修饰符应该出现在任何自定义修饰符之前。

正确写法::

    function kill() public onlyowner {
        selfdestruct(owner);
    }

错误写法::

    function kill() onlyowner public {
        selfdestruct(owner);
    }

对于长函数声明，建议将每个参数独立一行并与函数体保持相同的缩进级别。闭括号和开括号也应该
独立一行并保持与函数声明相同的缩进级别。

正确写法::

    function thisFunctionHasLotsOfArguments(
        address a,
        address b,
        address c,
        address d,
        address e,
        address f
    )
        public
    {
        doSomething();
    }

错误写法::

    function thisFunctionHasLotsOfArguments(address a, address b, address c,
        address d, address e, address f) public {
        doSomething();
    }

    function thisFunctionHasLotsOfArguments(address a,
                                            address b,
                                            address c,
                                            address d,
                                            address e,
                                            address f) public {
        doSomething();
    }

    function thisFunctionHasLotsOfArguments(
        address a,
        address b,
        address c,
        address d,
        address e,
        address f) public {
        doSomething();
    }


如果一个长函数声明有修饰符，那么每个修饰符应该下沉到独立的一行。

正确写法::

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public
        onlyowner
        priced
        returns (address)
    {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(
        address x,
        address y,
        address z,
    )
        public
        onlyowner
        priced
        returns (address)
    {
        doSomething();
    }

错误写法::

    function thisFunctionNameIsReallyLong(address x, address y, address z)
                                          public
                                          onlyowner
                                          priced
                                          returns (address) {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public onlyowner priced returns (address)
    {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public
        onlyowner
        priced
        returns (address) {
        doSomething();
    }

多路输出参数和返回语句应遵循推荐统一风格，可参考 :ref:`Maximum Line Length <maximum_line_length>` 章节。
正确写法::

    function thisFunctionNameIsReallyLong(
        address a,
        address b,
        address c
    )
        public
        returns (
            address someAddressName,
            uint256 LongArgument,
            uint256 Argument
        )
    {
        doSomething()

        return (
            veryLongReturnArg1,
            veryLongReturnArg2,
            veryLongReturnArg3
        );
    }

错误写法::

    function thisFunctionNameIsReallyLong(
        address a,
        address b,
        address c
    )
        public
        returns (address someAddressName,
                 uint256 LongArgument,
                 uint256 Argument)
    {
        doSomething()

        return (veryLongReturnArg1,
                veryLongReturnArg1,
                veryLongReturnArg1);
    }

对于继承合约中需要参数构造函数，如果函数声明很长或难以阅读，则建议将基构造函数和修饰符下沉放在
新的一行上。

正确写法::

    contract A is B, C, D {
        function A(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4)
            public
        {
            // do something with param5
        }
    }

错误写法::

    contract A is B, C, D {
        function A(uint param1, uint param2, uint param3, uint param4, uint param5)
        B(param1)
        C(param2, param3)
        D(param4)
        public
        {
            // do something with param5
        }
    }

    contract A is B, C, D {
        function A(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4)
            public {
            // do something with param5
        }
    }

当用单个语句声明简短函数时，允许在一行中完成。

允许：：

  function shortFunction() public { doSomething(); }

这些函数声明的准则旨在提高可读性。
因为本指南不会涵盖所有内容，作者应该自行判断函数声明的可能排列方式。

映射
========

待定

变量声明
=====================

数组变量的声明在变量类型和括号之间不应该有空格。

正确写法::

    uint[] x;

错误写法::

    uint [] x;


其他建议
=====================

* 字符串应该用双引号而不是单引号。

正确写法::

      str = "foo";
      str = "Hamlet says, 'To be or not to be...'";

错误写法::

      str = 'bar';
      str = '"Be yourself; everyone else is already taken." -Oscar Wilde';

* 操作符两边应该各有一个空格。

正确写法::

    x = 3;
    x = 100 / 10;
    x += 3 + 4;
    x |= y && z;

错误写法::

    x=3;
    x = 100/10;
    x += 3+4;
    x |= y&&z;

* 为了表示优先级，高优先级操作符两边可以省略空格。这样可以提高复杂语句的可读性。 你应该在操作符两边
总是使用相同的空白量：

正确写法::

    x = 2**3 + 5;
    x = 2*y + 3*z;
    x = (a+b) * (a-b);

错误写法::

    x = 2** 3 + 5;
    x = y+z;
    x +=1;


******************
命名规范
******************

当完全采纳和使用命名规范时会产生强大的作用。 当使用不同的规范时，则不会立即获取代码中传达的重要 *元* 信息。

这里给出的命名建议旨在提高可读性，因此它们不是规则，而是透过名称来尝试和帮助传达最大的信息量。

最后，基于代码库中的一致性，本文档中的任何规范总是可以被（代码库中的规范）取代。 


命名方式
=============

为了避免混淆，下面的名字用来指不同的命名方式。

* ``b`` (单个小写字母)
* ``B`` (单个大写字母)
* ``lowercase`` （小写）
* ``lower_case_with_underscores`` （小写和下划线）
* ``UPPERCASE`` （大写）
* ``UPPER_CASE_WITH_UNDERSCORES`` （大写和下划线）
* ``CapitalizedWords`` (驼峰式，首字母大写）
* ``mixedCase`` (混合命名法，区别于首字母大写的初始字母小写!)
* ``Capitalized_Words_With_Underscores`` (首字母大写和下划线)

..注意:: 当使用驼峰式进行首字母缩写时，大写缩写中的所有字母。 因此HTTPServerError比HttpServerError好。
 当使用混合名称命名时，除了保留第一个缩写小写（如果它是名称的开头），大写缩写中的所有字母。
 因此xmlHTTPRequest比XMLHTTPRequest更好。


应避免的名称
==============

* ``l`` - el的小写方式
* ``O`` - oh的大写方式
* ``I`` - eye的大写方式

切勿将任何这些用于单个字母的变量名称。 他们经常与数字1和零不可区分。

合约和库名称
==========================

合约和库名称应该使用驼峰式风格。比如：``SimpleToken``, ``SmartBank``, ``CertificateHashRepository``, ``Player``.

结构体名称
==========================

结构体名称应该使用驼峰式风格。比如：``MyCoin``, ``Position``, ``PositionXY``.

事件名称
===========

事件名称应该使用驼峰式风格。比如：``Deposit``, ``Transfer``, ``Approval``, ``BeforeTransfer``, ``AfterTransfer``.

函数名称
==============
函数名称不同于结构，应该使用混合命名法风格。比如：``getBalance``, ``transfer``, ``verifyOwner``, ``addMember``, ``changeOwner``.

函数参数命名
=======================

函数参数命名应该使用混合命名法风格。比如：``initialSupply``, ``account``, ``recipientAddress``, ``senderAddress``, ``newOwner``.
在编写操作自定义结构的库函数时，第一个参数应该是结构体，并且应该始终命名为“self”。

本地变量和状态变量名称
==============================

使用混合命名法风格。比如：``totalSupply``, ``remainingSupply``, ``balancesOf``, ``creatorAddress``, ``isPreSale``, ``tokenExchangeRate``.

常量命名
=========

常量应该全都使用大写字母书写，并用下划线分割单词。比如：``MAX_BLOCKS``, `TOKEN_NAME`, ``TOKEN_TICKER``, ``CONTRACT_VERSION``.

修饰符命名
==============

使用混合命名法风格。比如：``onlyBy``, ``onlyAfter``, ``onlyDuringThePreSale``.

枚举变量命名
=====

枚举，在简单类型声明时，应该使用驼峰式风格。比如：``TokenGroup``, ``Frame``, ``HashStyle``, ``CharacterLocation``.

避免命名冲突
==========================

* ``single_trailing_underscore_``

当所起名称与内建或保留关键字相冲突时，建议使用此惯例内置或其他保留名称。


常规建议
=======================

待定
