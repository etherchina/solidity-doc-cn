.. index:: style, coding style

#############
编程风格指南
#############

************
概述
************

本指南旨在约定 Solidity 代码的编码规范。本指南是不断变化演进的，旧的、过时的编码规范会被淘汰，
而新的、有用的规范会被添加进来。

许多项目会实施他们自己的编码风格指南。如遇冲突，应优先使用具体项目的风格指南。

本风格指南中的结构和许多建议是取自 python 的 `pep8 style guide <https://www.python.org/dev/peps/pep-0008/>`_ 。



本指南并 *不是* 以指导正确或最佳的 Solidity 编码方式为目的。本指南的目的是保持代码的 *一致性* 。
来自 python 的参考文档 `pep8 <https://www.python.org/dev/peps/pep-0008/#a-foolish-consistency-is-the-hobgoblin-of-little-minds>`_ 。很好地阐述了这个概念。

.. note::

  风格指南是关于一致性的。重要的是与此风格指南保持一致。但项目中的一致性更重要。一个模块或功能内的一致性是最重要的。
  
  但最重要的是：知道什么时候不一致 —— 有时风格指南不适用。如有疑问，请自行判断。看看其他例子，并决定什么看起来最好。并应毫不犹豫地询问他人！

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

在 Solidity 源码中合约声明之间留出两个空行。


正确写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract A {
        ...
    }


    contract B {
        ...
    }


    contract C {
        ...
    }

错误写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

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

在相关联的各组单行语句之间可以省略空行。（例如抽象合约的 stub 函数）。

正确写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    abstract contract A {
        function spam() public virtual pure;
        function ham() public virtual pure;
    }


    contract B is A {
        function spam() public pure override {
            // ...
        }

        function ham() public pure override {
            // ...
        }
    }

错误写法:

.. code-block:: solidity

    pragma solidity >=0.4.0 <0.9.0;

    abstract contract A {
        function spam() virtual pure public;
        function ham() public virtual pure;
    }


    contract B is A {
        function spam() public pure override {
            // ...
        }
        function ham() public pure override {
            // ...
        }
    }


.. _maximum_line_length:

代码行的最大长度
===================

基于 `PEP 8 规范 <https://www.python.org/dev/peps/pep-0008/#maximum-line-length>`_ ，将代码行的字符长度控制在 79（或 99）字符来帮助读者阅读代码。

折行时应该遵从以下指引：

1. 第一个参数不应该紧跟在左括号后边
2. 用一个、且只用一个缩进
3. 每个函数应该单起一行
4. 结束符号 :code:`);` 应该单独放在最后一行

函数调用

正确写法:

.. code-block:: solidity

    thisFunctionCallIsReallyLong(
        longArgument1,
        longArgument2,
        longArgument3
    );

错误写法:

.. code-block:: solidity

    thisFunctionCallIsReallyLong(longArgument1,
                                  longArgument2,
                                  longArgument3
    );

    thisFunctionCallIsReallyLong(longArgument1,
        longArgument2,
        longArgument3
    );

    thisFunctionCallIsReallyLong(
        longArgument1, longArgument2,
        longArgument3
    );

    thisFunctionCallIsReallyLong(
    longArgument1,
    longArgument2,
    longArgument3
    );

    thisFunctionCallIsReallyLong(
        longArgument1,
        longArgument2,
        longArgument3);

赋值语句

正确写法:

.. code-block:: solidity

    thisIsALongNestedMapping[being][set][toSomeValue] = someFunction(
        argument1,
        argument2,
        argument3,
        argument4
    );

错误写法:

.. code-block:: solidity

    thisIsALongNestedMapping[being][set][toSomeValue] = someFunction(argument1,
                                                                       argument2,
                                                                       argument3,
                                                                       argument4);

定义事件和触发事件

正确写法:

.. code-block:: solidity

    event LongAndLotsOfArgs(
        adress sender,
        adress recipient,
        uint256 publicKey,
        uint256 amount,
        bytes32[] options
    );

    emit LongAndLotsOfArgs(
        sender,
        recipient,
        publicKey,
        amount,
        options
    );

错误写法:

.. code-block:: solidity

    event LongAndLotsOfArgs(adress sender,
                            adress recipient,
                            uint256 publicKey,
                            uint256 amount,
                            bytes32[] options);

    LongAndLotsOfArgs(sender,
                      recipient,
                      publicKey,
                      amount,
                      options);

源文件编码格式
====================

首选 UTF-8 或 ASCII 编码。

导入文件规范
====================

Import 语句应始终放在文件的顶部。

正确写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    import "./Owned.sol";

    contract A {
        ...
    }


    contract B is owned {
        ...
    }

错误写法:

.. code-block:: solidity

    contract A {
        ...
    }


    import "./Owned.sol";


    contract B is owned {
        ...
    }

函数顺序
==================

排序有助于读者识别他们可以调用哪些函数，并更容易地找到构造函数和 fallback 函数的定义。

函数应根据其可见性和顺序进行分组：

- 构造函数
- receive 函数（如果存在）
- fallback 函数（如果存在）
- 外部函数(external)
- 公共函数(public)
- 内部函数(internal)
- 私有函数(private)

在一个分组中，把 ``view`` 和 ``pure`` 函数放在最后。

正确写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    contract A {
        function A() public {
            ...
        }

        receive() external payable {
            // ...
        }

        fallback() external {
            // ...
        }

        // External functions
        // ...

        // External functions that are view
        // ...

        // External functions that are pure
        // ...

        // Public functions
        // ...

        // Internal functions
        // ...

        // Private functions
        // ...
    }

错误写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    contract A {

        // External functions
        // ...


        fallback() external {
            // ...
        }
        receive() external payable {
            // ...
        }

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

在以下情况下避免无不必要的空格：

除单行函数声明外，紧接着小括号，中括号或者大括号的内容应该避免使用空格。

正确写法:

.. code-block:: solidity

    spam(ham[1], Coin({name: "ham"}));

错误写法:

.. code-block:: solidity
    
    spam( ham[ 1 ], Coin( { name: "ham" } ) );

例外情况:

.. code-block:: solidity

    function singleLine() public { spam(); }


紧接在逗号，分号之前不需要有空格：

正确写法:

.. code-block:: solidity

    function spam(uint i, Coin coin) public;

错误写法:

.. code-block:: solidity

    function spam(uint i , Coin coin) public ;

赋值或其他操作符两边一个的空格：

正确写法:

.. code-block:: solidity

    x = 1;
    y = 2;
    long_variable = 3;

错误写法:

.. code-block:: solidity

    x             = 1;
    y             = 2;
    long_variable = 3;

fallback 和 receive 函数中不要包含空格：

正确写法:

.. code-block:: solidity

    receive() external payable {
        ...
    }

    function() public {
        ...
    }

错误写法:

.. code-block:: solidity

    receive () external payable {
        ...
    }

    function () public {
        ...
    }

控制结构
==================

用大括号表示一个合约，库、函数和结构。
应该：

* 开括号与声明应在同一行。
* 闭括号在与之前函数声明对应的开括号保持同一缩进级别上另起一行。
* 开括号前应该有一个空格。

正确写法:

.. code-block:: solidity

    contract Coin {
        struct Bank {
            address owner;
            uint balance;
        }
    }

错误写法:

.. code-block:: solidity

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

正确写法:

.. code-block:: solidity

    if (...) {
        ...
    }

    for (...) {
        ...
    }

错误写法:

.. code-block:: solidity

    if (...)
    {
        ...
    }

    while(...){
    }

    for (...) {
        ...;}

对于控制结构， *如果* 其主体内容只包含一行，则可以省略括号。

正确写法:

.. code-block:: solidity

    if (x < 10)
        x += 1;

错误写法:

.. code-block:: solidity

    if (x < 10)
        someArray.push(Coin({
            name: 'spam',
            value: 42
        }));

对于具有 ``else`` 或 ``else if`` 子句的 ``if`` 块， ``else`` 应该是与 ``if`` 的闭大括号放在同一行上。 这一规则区别于
其他块状结构。

正确写法:

.. code-block:: solidity

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

错误写法:

.. code-block:: solidity

    if (x < 3) {
        x += 1;
    }
    else {
        x -= 1;
    }

函数声明
====================

对于简短的函数声明，建议函数体的开括号与函数声明保持在同一行。

闭大括号应该与函数声明的缩进级别相同。

开大括号之前应该有一个空格。

正确写法:

.. code-block:: solidity

    function increment(uint x) public pure returns (uint) {
        return x + 1;
    }

    function increment(uint x) public pure onlyowner returns (uint) {
        return x + 1;
    }

错误写法:

.. code-block:: solidity

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

你应该严格地标示所有函数的可见性，包括构造函数。



正确写法:

.. code-block:: solidity

    function explicitlyPublic(uint val) public {
        doSomething();
    }

错误写法:

.. code-block:: solidity

    function implicitlyPublic(uint val) {
        doSomething();
    }


函数修改器的顺序应该是:

1. 可见性（Visibility）
2. 可变性（Mutability）
3. 虚拟（Virtual）
4. 重载（Override）
5. 自定义修改器（Custom modifiers）

正确写法:

.. code-block:: solidity

    function balance(uint from) public view override returns (uint)  {
        return balanceOf[from];
    }

    function shutdown() public onlyowner {
        selfdestruct(owner);
    }

错误写法:

.. code-block:: solidity

    function balance(uint from) public override view returns (uint)  {
        return balanceOf[from];
    }

    function shutdown() onlyowner public {
        selfdestruct(owner);
    }


对于长函数声明，建议将每个参数独立一行并与函数体保持相同的缩进级别。闭括号和开括号也应该
独立一行并保持与函数声明相同的缩进级别。

正确写法:

.. code-block:: solidity

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

错误写法:

.. code-block:: solidity

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

正确写法:

.. code-block:: solidity

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
        address z
    )
        public
        onlyowner
        priced
        returns (address)
    {
        doSomething();
    }

错误写法:

.. code-block:: solidity

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

多行输出参数和返回值语句应该遵从 :ref:`代码行的最大长度 <maximum_line_length>` 一节的说明。

正确写法:

.. code-block:: solidity

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

错误写法:

.. code-block:: solidity

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

对于继承合约中需要参数的构造函数，如果函数声明很长或难以阅读，建议将基础构造函数像多个修饰符的风格那样
每个下沉到一个新行上书写。

正确写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.7.0;

    // Base contracts just to make this compile
    contract B {
        constructor(uint) {
        }
    }
    contract C {
        constructor(uint, uint) {
        }
    }
    contract D {
        constructor(uint) {
        }
    }

    contract A is B, C, D {
        uint x;

        constructor(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4)
        {
            // do something with param5
            x = param5;
        }
    }

错误写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;


    // Base contracts just to make this compile
    contract B {
        constructor(uint) {
        }
    }


    contract C {
        constructor(uint, uint) {
        }
    }


    contract D {
        constructor(uint) {
        }
    }


    contract A is B, C, D {
        uint x;

        constructor(uint param1, uint param2, uint param3, uint param4, uint param5)
        B(param1)
        C(param2, param3)
        D(param4)
        public {
            x = param5;
        }
    }


    contract X is B, C, D {
        uint x;

        constructor(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4)
            public {
                x = param5;
            }
    }


当用单个语句声明简短函数时，允许在一行中完成。

允许的写法:

.. code-block:: solidity

    function shortFunction() public { doSomething(); }

这些函数声明的准则旨在提高可读性。
因为本指南不会涵盖所有内容，作者应该自行作出最佳判断。

映射
========

在变量声明中，不要用空格将关键字 ``mapping`` 和其类型分开。
类型之间用一个空格隔开。不要用空格分隔任何嵌套的 ``mapping`` 关键词和它的类型。

正确写法:

.. code-block:: solidity

    mapping(uint => uint) map;
    mapping(address => bool) registeredAddresses;
    mapping(uint => mapping(bool => Data[])) public data;
    mapping(uint => mapping(uint => s)) data;

错误写法:

.. code-block:: solidity

    mapping (uint => uint) map;
    mapping( address => bool ) registeredAddresses;
    mapping (uint => mapping (bool => Data[])) public data;
    mapping(uint => mapping (uint => s)) data;


变量声明
=====================

数组变量的声明在变量类型和括号之间不应该有空格。

正确写法:

.. code-block:: solidity

    uint[] x;

错误写法:

.. code-block:: solidity

    uint [] x;


其他建议
=====================

* 字符串应该用双引号而不是单引号。

正确写法:

.. code-block:: solidity

      str = "foo";
      str = "Hamlet says, 'To be or not to be...'";

错误写法:

.. code-block:: solidity

      str = 'bar';
      str = '"Be yourself; everyone else is already taken." -Oscar Wilde';

* 操作符两边应该各有一个空格。

正确写法:

.. code-block:: solidity

    x = 3;
    x = 100 / 10;
    x += 3 + 4;
    x |= y && z;

错误写法:

.. code-block:: solidity

    x=3;
    x = 100/10;
    x += 3+4;
    x |= y&&z;

* 为了表示优先级，高优先级操作符两边可以省略空格。这样可以提高复杂语句的可读性。你应该在操作符两边总是使用相同的空格数：

正确写法:

.. code-block:: solidity

    x = 2**3 + 5;
    x = 2*y + 3*z;
    x = (a+b) * (a-b);

错误写法:

.. code-block:: solidity

    x = 2** 3 + 5;
    x = y+z;
    x +=1;

***************
布局顺序
***************

函数的各元素建议布局的顺序如下：

1. Pragma 语句
2. Import 语句
3. 接口
4. 库
5. 合约

在每个合约、库或接口内，使用如下顺序：

1. 类型声明
2. 状态声明
3. 事件
4. 修改器
5. 函数

.. note::

    在声明类型时，挨着其使用的时间或状态时，会更清晰。

******************
命名规范
******************

当完全采纳和使用命名规范时会产生强大的作用。 当使用不同的规范时，则不会立即获取代码中传达的重要 *元* 信息。

这里给出的命名建议旨在提高可读性，因此它们不是规则，而是透过名称来尝试和帮助传达最多的信息。

最后，基于代码库中的一致性，本文档中的任何规范总是可以被（代码库中的规范）取代。


命名风格
=============

为了避免混淆，下面的名字用来指明不同的命名方式。

* ``b`` (单个小写字母)
* ``B`` (单个大写字母)
* ``lowercase`` （小写）
* ``UPPERCASE`` （大写）
* ``UPPER_CASE_WITH_UNDERSCORES`` （大写和下划线）
* ``CapitalizedWords`` (驼峰式，首字母大写）
* ``mixedCase`` (混合式，与驼峰式的区别在于首字母小写！)

..note:: 
    
    当在驼峰式命名中使用缩写时，应该将缩写中的所有字母都大写。 因此 HTTPServerError 比 HttpServerError 好。
 当在混合式命名中使用缩写时，除了第一个缩写中的字母小写（如果它是整个名称的开头的话）以外，其他缩写中的字母均大写。
 因此 xmlHTTPRequest 比 XMLHTTPRequest 更好。


应避免的名称
==============

* ``l`` - el的小写方式
* ``O`` - oh的大写方式
* ``I`` - eye的大写方式

切勿将任何这些用于单个字母的变量名称。 他们经常难以与数字 1 和 0 区分开。

合约和库名称
==========================

合约和库名称应该使用驼峰式风格。比如： ``SimpleToken`` ， ``SmartBank`` ， ``CertificateHashRepository`` ， ``Player`` ， ``Congress``, ``Owned``。
* 合约和库的名称应该和他们的文件名一致。
* 如果合约文件包含多个合约或库，则文件名应该匹配 *核心合约* ，但我们应该尽量避免这个情况。

如下面的例子所示，如果合约名称是 ``Congress`` ，库名称是 ``Owned``，那么它们的相关文件名应该是 ``Congress.sol`` 和 ``Owned.sol`` 。

正确写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.7.0;


    // Owned.sol
    contract Owned {
        address public owner;

        constructor() {
            owner = msg.sender;
        }

        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }

        function transferOwnership(address newOwner) public onlyOwner {
            owner = newOwner;
        }
    }

在 ``Congress.sol`` 文件中:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    import "./Owned.sol";


    contract Congress is Owned, TokenRecipient {
        //...
    }

错误写法:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.7.0;


    // owned.sol
    contract owned {
        address public owner;

        constructor() {
            owner = msg.sender;
        }

        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }

        function transferOwnership(address newOwner) public onlyOwner {
            owner = newOwner;
        }
    }

在 ``Congress.sol`` 中:

.. code-block:: solidity

    import "./owned.sol";


    contract Congress is owned, tokenRecipient {
        //...
    }


结构体名称
==========================

结构体名称应该使用驼峰式风格。比如：``MyCoin``，``Position``，``PositionXY``。

事件名称
===========

事件名称应该使用驼峰式风格。比如：``Deposit``，``Transfer``，``Approval``，``BeforeTransfer``，``AfterTransfer``。

函数名称
==============
函数应该使用混合式命名风格。比如：``getBalance``，``transfer``，``verifyOwner``，``addMember``，``changeOwner``。

函数参数命名
=======================

函数参数命名应该使用混合式命名风格。比如：``initialSupply``，``account``，``recipientAddress``，``senderAddress``，``newOwner``。
在编写操作自定义结构的库函数时，这个结构体应该作为函数的第一个参数，并且应该始终命名为 ``self``。

局部变量和状态变量名称
==============================

使用混合式命名风格。比如：``totalSupply``，``remainingSupply``，``balancesOf``，``creatorAddress``，``isPreSale``，``tokenExchangeRate``。

常量命名
=========

常量应该全都使用大写字母书写，并用下划线分割单词。比如：``MAX_BLOCKS``，``TOKEN_NAME``，``TOKEN_TICKER``，``CONTRACT_VERSION``。

修饰符命名
==============

使用混合式命名风格。比如：``onlyBy``，``onlyAfter``，``onlyDuringThePreSale``。

枚举命名
====================

在声明简单类型时，枚举应该使用驼峰式风格。比如：``TokenGroup``，``Frame``，``HashStyle``，``CharacterLocation``。


避免命名冲突
==========================

* ``singleTrailingUnderscore_``

当所起名称与内建或保留关键字相冲突时，建议照此惯例在名称后边添加下划线。

.. _style_guide_natspec:

************************
描述注释 NatSpec
************************

Solidity 智能合约包含了NatSpec注释形式。
单行使用  ``///`` 开始，多行使用 ``/**`` 开头以 ``*/`` 结尾。

例如, 以来自 :ref:`简单合约 <simple-smart-contract>` 加上注释为例，看上去是这样：

.. code-block:: solidity

    pragma solidity >=0.4.16 <0.9.0;

    /// @author The Solidity Team
    /// @title A simple storage example
    contract TinyStorage {
        uint storedData;

        /// Store `x`.
        /// @param x the new value to store
        /// @dev stores the number in the state variable `storedData`
        function set(uint x) public {
            storedData = x;
        }

        /// Return the stored value.
        /// @dev retrieves the value of the state variable `storedData`
        /// @return the stored value
        function get() public view returns (uint) {
            return storedData;
        }
    }


推荐使用 :ref:`NatSpec <natspec>` 为所有的开放接口（在 ABI 里呈现的内容）进行完整的注释。

参考 :ref:`NatSpec <natspec>` 部分了解更多。

