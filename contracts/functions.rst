
.. index:: ! functions

.. _functions:

******
函数
******


.. _function-parameters-return-variables:

Function Parameters and Return Variables
========================================

As in JavaScript, functions may take parameters as input. Unlike in JavaScript
and C, functions may also return an arbitrary number of values as output.

Function Parameters
-------------------

Function parameters are declared the same way as variables, and the name of
unused parameters can be omitted.

For example, if you want your contract to accept one kind of external call
with two integers, you would use something like::

    pragma solidity >=0.4.16 <0.7.0;

    contract Simple {
        uint sum;
        function taker(uint _a, uint _b) public {
            sum = _a + _b;
        }
    }

Function parameters can be used as any other local variable and they can also be assigned to.

.. note::

  An :ref:`external function<external-function-calls>` cannot accept a
  multi-dimensional array as an input
  parameter. This functionality is possible if you enable the new
  experimental ``ABIEncoderV2`` feature by adding ``pragma experimental ABIEncoderV2;`` to your source file.

  An :ref:`internal function<external-function-calls>` can accept a
  multi-dimensional array without enabling the feature.

.. index:: return array, return string, array, string, array of strings, dynamic array, variably sized array, return struct, struct

Return Variables
----------------

Function return variables are declared with the same syntax after the
``returns`` keyword.

For example, suppose you want to return two results: the sum and the product of
two integers passed as function parameters, then you use something like::

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

The names of return variables can be omitted.
Return variables can be used as any other local variable and they
are initialized with their :ref:`default value <default-value>` and have that value unless explicitly set.

You can either explicitly assign to return variables and
then leave the function using ``return;``,
or you can provide return values
(either a single or :ref:`multiple ones<multi-return>`) directly with the ``return``
statement::

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

This form is equivalent to first assigning values to the
return variables and then using ``return;`` to leave the function.

.. note::
    You cannot return some types from non-internal functions, notably
    multi-dimensional dynamic arrays and structs. If you enable the
    new experimental ``ABIEncoderV2`` feature by adding ``pragma experimental
    ABIEncoderV2;`` to your source file then more types are available, but
    ``mapping`` types are still limited to inside a single contract and you
    cannot transfer them.

.. _multi-return:

Returning Multiple Values
-------------------------

When a function has multiple return types, the statement ``return (v0, v1, ..., vn)`` can be used to return multiple values.
The number of components must be the same as the number of return types.


.. index:: ! view function, function;view

.. _view-functions:

View 函数
==============

可以将函数声明为 ``view`` 类型，这种情况下要保证不修改状态。

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
  ``constant`` 是 ``view`` 的别名。

.. note::
  Getter 方法被标记为 ``view``。

.. warning::
  编译器没有强制 ``view`` 方法不能修改状态。

.. index:: ! pure function, function;pure

.. _pure-functions:

Pure 函数
==============

函数可以声明为 ``pure`` ，在这种情况下，承诺不读取或修改状态。

除了上面解释的状态修改语句列表之外，以下被认为是从状态中读取：

#. 读取状态变量。
#. 访问 ``this.balance`` 或者 ``<address>.balance``。
#. 访问 ``block``，``tx``， ``msg`` 中任意成员 （除 ``msg.sig`` 和 ``msg.data`` 之外）。
#. 调用任何未标记为 ``pure`` 的函数。
#. 使用包含某些操作码的内联汇编。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }

.. warning::
  编译器没有强制 ``pure`` 方法不能读取状态。

.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fallback 函数
=================

合约可以有一个未命名的函数。这个函数不能有参数也不能有返回值。
如果在一个到合约的调用中，没有其他函数与给定的函数标识符匹配（或没有提供调用数据），那么这个函数（fallback 函数）会被执行。

除此之外，每当合约收到以太币（没有任何数据），这个函数就会执行。此外，为了接收以太币，fallback 函数必须标记为 ``payable``。
如果不存在这样的函数，则合约不能通过常规交易接收以太币。

在这样的上下文中，通常只有很少的 gas 可以用来完成这个函数调用（准确地说，是 2300 gas），所以使 fallback 函数的调用尽量廉价很重要。
请注意，调用 fallback 函数的交易（而不是内部调用）所需的 gas 要高得多，因为每次交易都会额外收取 21000 gas 或更多的费用，用于签名检查等操作。

具体来说，以下操作会消耗比 fallback 函数更多的 gas：

- 写入存储
- 创建合约
- 调用消耗大量 gas 的外部函数
- 发送以太币

请确保您在部署合约之前彻底测试您的 fallback 函数，以确保执行成本低于 2300 个 gas。

.. note::
    即使 fallback 函数不能有参数，仍然可以使用 ``msg.data`` 来获取随调用提供的任何有效数据。

.. warning::
    一个没有定义 fallback 函数的合约，直接接收以太币（没有函数调用，即使用 ``send`` 或 ``transfer``）会抛出一个异常，
    并返还以太币（在 Solidity v0.4.0 之前行为会有所不同）。所以如果你想让你的合约接收以太币，必须实现 fallback 函数。

.. warning::
    一个没有 payable fallback 函数的合约，可以作为 `coinbase transaction` （又名 `miner block reward` ）的接收者或者作为 ``selfdestruct`` 的目标来接收以太币。

    一个合约不能对这种以太币转移做出反应，因此也不能拒绝它们。这是 EVM 在设计时就决定好的，而且 Solidity 无法绕过这个问题。

    这也意味着 ``this.balance`` 可以高于合约中实现的一些手工记帐的总和（即在 fallback 函数中更新的累加器）。

::

    pragma solidity >=0.5.0 <0.7.0;

    contract Test {
        // 发送到这个合约的所有消息都会调用此函数（因为该合约没有其它函数）。
        // 向这个合约发送以太币会导致异常，因为 fallback 函数没有 `payable` 修饰符
        function() public { x = 1; }
        uint x;
    }


    // 这个合约会保留所有发送给它的以太币，没有办法返还。
    contract Sink {
        function() public payable { }
    }

    contract Caller {
        function callTest(Test test) public {
            test.call(0xabcdef01); // 不存在的哈希
            // 导致 test.x 变成 == 1。
            // 以下将不会编译，但如果有人向该合约发送以太币，交易将失败并拒绝以太币。
            // test.send(2 ether）;
        }
    }

.. index:: ! overload

.. _overload-function:

函数重载
====================

合约可以具有多个不同参数的同名函数。这也适用于继承函数。以下示例展示了合约 ``A`` 中的重载函数 ``f``。

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
