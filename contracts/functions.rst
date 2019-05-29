
.. index:: ! functions

.. _functions:

******
函数
******

.. _function-parameters-return-variables:

函数参数及返回值
========================================

与 Javascript 一样，函数可能需要参数作为输入;
而与 Javascript 和 C 不同的是，它们可能返回任意数量的参数作为输出。


函数参数（输入参数）
------------------------------


函数参数的声明方式与变量相同。不过未使用的参数可以省略参数名。

例如，如果我们希望合约接受有两个整数形参的函数的外部调用，可以像下面这样写::


    pragma solidity >=0.4.16 <0.7.0;

    contract Simple {
        uint sum;
        function taker(uint _a, uint _b) public {
            sum = _a + _b;
        }
    }

函数参数可以当作为本地变量，也可用在等号左边被赋值。


.. note::

   :ref:`外部函数<external-function-calls>` 不可以接受多维数组作为参数
   如果添加  ``pragma experimental ABIEncoderV2;`` 启用实验功能  ``ABIEncoderV2`` 则是可以的。

   :ref:`内部函数<external-function-calls>` 在不启用实验功能  ``ABIEncoderV2`` 的情况下也可以接受多维数组作为参数。

.. index:: return array, return string, array, string, array of strings, dynamic array, variably sized array, return struct, struct


返回变量
----------------

函数返回变量的声明方式在关键词 ``returns`` 之后，与参数的声明方式相同。

例如，如果我们需要返回两个结果：两个给定整数的和与积，我们应该写作
::

    pragma solidity >=0.4.16 <0.7.0;

    contract Simple {
        function arithmetic(uint _a, uint _b)
            public
            pure
            returns (uint o_sum, uint o_product)
        {
            o_sum = _a + _b;
            o_product = _a * _b;
        }
    }


返回变量名可以被省略。
返回变量可以当作为函数中的本地变量，没有显式设置的话，会使用 :ref:` 默认值 <default-value>` 
返回变量可以显式给它附一个值，也可以使用 ``return`` 语句指定，使用 ``return`` 语句可以一个或多个值，参阅 :ref:`multiple ones<multi-return>` 。

::

    pragma solidity >=0.4.16 <0.7.0;

    contract Simple {
        function arithmetic(uint _a, uint _b)
            public
            pure
            returns (uint o_sum, uint o_product)
        {
            return (_a + _b, _a * _b);
        }
    }

这个形式等同于赋值给返回参数，然后用 ``return;`` 退出。

.. note::
   非内部函数有些类型没法返回，比如限制的类型有：多维动态数组、结构体等。

   如果添加  ``pragma experimental ABIEncoderV2;`` 启用实验功能 ``ABIEncoderV2`` 则是可以的返回更多类型，不过 ``mapping``  仍然是受限的。

.. _multi-return:

返回多个值
-------------------------

当函数需要使用多个值，可以用语句 ``return (v0, v1, ..., vn)`` 。
参数的数量需要和声明时候一致。

.. index:: ! view function, function;view

.. _view-functions:

View 视图函数
==============

可以将函数声明为 ``view`` 类型，这种情况下要保证不修改状态。

.. note::
  
  如果编译器的 EVM 目标是拜占庭硬分叉（ 译者注：Byzantium 分叉发生在2017年10月，这次分叉进加入了4个操作符： REVERT 、RETURNDATASIZE、RETURNDATACOPY 、STATICCALL） 或更新的 (默认), 则操作码 ``STATICCALL`` 将用于视图函数, 这些函数强制在 EVM 执行过程中保持不修改状态。
  对于库视图函数, 使用 ``DELLEGATECALL``, 因为没有组合的 ``DELEGATECALL`` 和 ``STATICALL``。这意味着库视图函数不会在运行时检查进而阻止状态修改。
  这不会对安全性产生负面影响, 因为库代码通常在编译时知道, 并且静态检查器会执行编译时检查。


下面的语句被认为是修改状态：

#. 修改状态变量。
#. :ref:`产生事件 <events>`。
#. :ref:`创建其它合约 <creating-contracts>`。
#. 使用 ``selfdestruct``。
#. 通过调用发送以太币。
#. 调用任何没有标记为 ``view`` 或者 ``pure`` 的函数。
#. 使用低级调用。
#. 使用包含特定操作码的内联汇编。

::

    pragma solidity  >=0.5.0 <0.7.0;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + now;
        }
    }

.. note::
  ``constant`` 之前是 ``view`` 的别名，不过在0.5.0之后移除了。

.. note::
  Getter 方法自动被标记为 ``view``。

.. note::
  
  在0.5.0 版本之前, 编译器没有对 ``view`` 函数使用 ``STATICCALL`` 操作码。
  这样通过使用无效的显式类型转换会启用视图函数中的状态修改。
  通过对 ``view`` 函数使用 ``STATICCALL`` , 可以防止在 EVM 级别上对状态进行修改。


.. index:: ! pure function, function;pure

.. _pure-functions:

Pure 纯函数
==============

函数可以声明为 ``pure`` ，在这种情况下，承诺不读取也不修改状态。


.. note::
  如果编译器的 EVM 目标是 Byzantium 或更新的 (默认), 则使用操作码 ``STATICCALL`` , 这并不保证状态未被读取, 但至少不被修改。


除了上面解释的状态修改语句列表之外，以下被认为是读取状态：

#. 读取状态变量。
#. 访问 ``address(this).balance`` 或者 ``<address>.balance``。
#. 访问 ``block``，``tx``， ``msg`` 中任意成员 （除 ``msg.sig`` 和 ``msg.data`` 之外）。
#. 调用任何未标记为 ``pure`` 的函数。
#. 使用包含某些操作码的内联汇编。

::

    pragma solidity >=0.5.0 <0.7.0;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }

纯函数能够使用 `revert()` 和 `require()` 在 :ref:`发生错误 <assert-and-require>` 时去还原潜在状态更改。

还原状态更改不被视为 "状态修改", 因为它只还原以前在没有``view`` 或 ``pure`` 限制的代码中所做的状态更改, 并且代码可以选择捕获 ``revert`` 并不传递还原。

这种行为也符合 ``STATICCALL`` 操作码。


.. warning::
  不可能在 EVM 级别阻止函数读取状态, 只能阻止它们写入状态 (即只能在 EVM 级别强制执行 ``view`` , 而 ``pure`` 不能强制)。

.. note::
  在0.5.0 版本之前, 编译器没有对 ``pure`` 函数使用 ``STATICCALL`` 操作码。这样通过使用无效的显式类型转换启用 ``pure`` 函数中的状态修改。
  通过对 ``pure`` 函数使用 ``STATICCALL`` , 可以防止在 EVM 级别上对状态进行修改。


.. note::
  
  在0.4.17版本之前，编译器不会强制 ``pure`` 函数不读取状态。它是一个编译时类型检查, 可以避免在合约类型之间进行无效的显式转换, 因为编译器可以验证合约类型没有状态更改操作, 但它不会在运行时能检查调用实际的类型。


.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fallback 回退函数
=================

合约可以有一个未命名的函数。这个函数不能有参数也不能有返回值。
如果在一个到合约的调用中，没有其他函数与给定的函数标识符匹配（或没有提供调用数据），那么这个函数（fallback 函数）会被执行。

除此之外，每当合约收到以太币（没有任何数据），这个函数就会执行。此外，为了接收以太币，fallback 函数必须标记为 ``payable`` 。
如果不存在这样的函数，则合约不能通过普通转账交易接收以太币。

在最坏的情况下，回退函数只有 2300 gas 可以使用（如，当使用 `send` 或 `transfer` 时）， 除了基础的日志输出之外，进行其他操作的余地很小。下面的操作消耗会操作 2300  gas :

- 写入存储
- 创建合约
- 调用消耗大量 gas 的外部函数
- 发送以太币

与任何其他函数一样，只要有足够的 gas 传递给它，回退函数就可以执行复杂的操作。

.. note::
    即使 fallback 函数不能有参数，仍然可以使用 ``msg.data`` 来获取随调用提供的任何有效数据。

.. warning::
    
    如果调用方打算调用不可用的函数, 也会执行回退函数。如果要实现回退函数仅用于接收以太, 则应添加类似 ``require(msg.data.length == 0)`` 检查以防止哪些无效的调用。

.. warning::
    一个没有定义 fallback 函数的合约，直接接收以太币（没有函数调用，即使用 ``send`` 或 ``transfer``）会抛出一个异常，
    并返还以太币（在 Solidity v0.4.0 之前行为会有所不同）。所以如果你想让你的合约接收以太币，必须实现 fallback 函数。

.. warning::
    一个没有 payable fallback 函数的合约，可以作为 `coinbase 交易` （又名 `矿工区块回报` ）的接收者或者作为 ``selfdestruct`` 的目标来接收以太币。

    一个合约不能对这种以太币转移做出反应，因此也不能拒绝它们。这是 EVM 在设计时就决定好的，而且 Solidity 无法绕过这个问题。

    这也意味着 ``address(this).balance`` 可以高于合约中实现的一些手工记帐的总和（例如在回退函数中更新的累加器记帐）。

::

    pragma solidity >=0.5.0 <0.7.0;

    contract Test {
        // 发送到这个合约的所有消息都会调用此函数（因为该合约没有其它函数）。
        // 向这个合约发送以太币会导致异常，因为 fallback 函数没有 `payable` 修饰符
        function() external { x = 1; }
        uint x;
    }


    // 这个合约会保留所有发送给它的以太币，没有办法返还。
    contract Sink {
        function() external payable { }
    }

    contract Caller {
        function callTest(Test test) public returns (bool) {
            (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            //  test.x 结果变成 == 1。

            // address(test) 不允许直接调用 ``send`` ,  因为 ``test`` 没有 payable 回退函数
            // 需要通过 uint160 转化为 ``address payable`` 类型 , 然后才可以调用 ``send``
            address payable testPayable = address(uint160(address(test)));


            // 以下将不会编译，但如果有人向该合约发送以太币，交易将失败并拒绝以太币。
            // test.send(2 ether）;
        }
    }

.. index:: ! overload

.. _overload-function:

函数重载
====================

合约可以具有多个不同参数的同名函数，称为“重载”（overloading），这也适用于继承函数。以下示例展示了合约 ``A`` 中的重载函数 ``f``。

::

    pragma solidity >=0.4.16 <0.7.0;

    contract A {
        function f(uint _in) public pure returns (uint out) {
            out = _in;
        }

        function f(uint _in, bool _really) public pure returns (uint out) {
            if (_really)
                out = _in;
        }
    }

重载函数也存在于外部接口中。如果两个外部可见函数仅区别于 Solidity 内的类型而不是它们的外部类型则会导致错误。

::

    // 以下代码无法编译
    pragma solidity >=0.4.16 <0.7.0;

    contract A {
        function f(B _in) public pure returns (B out) {
            out = _in;
        }

        function f(address _in) public pure returns (address out) {
            out = _in;
        }
    }

    contract B {
    }


以上两个 ``f`` 函数重载都接受了 ABI 的地址类型，虽然它们在 Solidity 中被认为是不同的。

重载解析和参数匹配
-----------------------------------------

通过将当前范围内的函数声明与函数调用中提供的参数相匹配，可以选择重载函数。
如果所有参数都可以隐式地转换为预期类型，则选择函数作为重载候选项。如果一个候选都没有，解析失败。

.. note::
    返回参数不作为重载解析的依据。

::

    pragma solidity >=0.4.16 <0.7.0;

    contract A {
        function f(uint8 _in) public pure returns (uint8 out) {
            out = _in;
        }

        function f(uint256 _in) public pure returns (uint256 out) {
            out = _in;
        }
    }

调用  ``f(50)`` 会导致类型错误，因为 ``50`` 既可以被隐式转换为 ``uint8`` 也可以被隐式转换为 ``uint256``。
另一方面，调用 ``f(256)`` 则会解析为 ``f(uint256)`` 重载，因为 ``256`` 不能隐式转换为 ``uint8``。
