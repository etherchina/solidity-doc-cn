
.. index:: ! functions

.. _functions:

******
函数
******

可以在合约内部和外部定义函数。

.. note::
  译者注：函数可以在合约外部定义是从 0.7.0 之后才开始支持的。


合约之外的函数（也称为“自由函数”）始终具有隐式的 ``internal`` :ref:`可见性<visibility-and-getters>`。 它们的代码包含在所有调用它们合约中，类似于内部库函数。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.1 <0.9.0;

    function sum(uint[] memory arr) pure returns (uint s) {
        for (uint i = 0; i < arr.length; i++)
            s += arr[i];
    }

    contract ArrayExample {
        bool found;
        function f(uint[] memory arr) public {
            // This calls the free function internally.
            // The compiler will add its code to the contract.
            uint s = sum(arr);
            require(s >= 10);
            found = true;
        }
    }

.. note::
    在合约之外定义的函数仍然在合约的上下文内执行。 他们仍然可以访问变量 ``this`` ，也可以调用其他合约，将其发送以太币或销毁调用它们合约等其他事情。
    与在合约中定义的函数的主要区别为：自由函数不能直接访问存储变量和不在他们的作用域范围内函数。



.. _function-parameters-return-variables:

函数参数及返回值
========================================

与 Javascript 一样，函数可能需要参数作为输入;
而与 Javascript 和 C 不同的是，它们可能返回任意数量的参数作为输出。


函数参数（输入参数）
------------------------------


函数参数的声明方式与变量相同。不过未使用的参数可以省略参数名。

例如，如果我们希望合约接受有两个整数形参的函数的外部调用，可以像下面这样写：

.. code-block:: solidity

    pragma solidity >=0.4.16 <0.9.0;

    contract Simple {
        uint sum;
        function taker(uint a, uint b) public {
            sum = a + b;
        }
    }

函数参数可以当作为本地变量，也可用在等号左边被赋值。


.. note::

    :ref:`外部函数<external-function-calls>` 不可以接受多维数组作为参数
    如果原文件加入 `pragma abicoder v2;` 可以启用ABI v2版编码功能，这此功能可用。
    （注：在 0.7.0 之前是使用 ``pragma experimental ABIEncoderV2;`` ）

    :ref:`内部函数<external-function-calls>` 则不需要启用ABI v2 就接受多维数组作为参数。

.. index:: return array, return string, array, string, array of strings, dynamic array, variably sized array, return struct, struct


返回变量
----------------

函数返回变量的声明方式在关键词 ``returns`` 之后，与参数的声明方式相同。

例如，如果我们需要返回两个结果：两个给定整数的和与积，我们应该写作：

.. code-block:: solidity

    pragma solidity >=0.4.16 <0.9.0;

    contract Simple {
        function arithmetic(uint a, uint b)
            public
            pure
            returns (uint sum, uint product)
        {
            sum = a + b;
            product = a * b;
        }
    }


返回变量名可以被省略。
返回变量可以当作为函数中的本地变量，没有显式设置的话，会使用 :ref:` 默认值 <default-value>`
返回变量可以显式给它附一个值(像上面)，也可以使用 ``return`` 语句指定，使用 ``return`` 语句可以一个或多个值，参阅 :ref:`multiple ones<multi-return>` 。

.. code-block:: solidity

    pragma solidity >=0.4.16 <0.9.0;

    contract Simple {
        function arithmetic(uint a, uint b)
            public
            pure
            returns (uint sum, uint product)
        {
            return (a + b, a * b);
        }
    }

这个形式等同于赋值给返回参数，然后用 ``return;`` 退出。

如果使用 ``return`` 提前退出有返回值的函数， 必须在用 return 时提供返回值。


.. note::
   非内部函数有些类型没法返回，比如限制的类型有：多维动态数组、结构体等。

   如果添加  ``pragma abicoder v2;`` 启用 ABI V2 编码器，则是可以的返回更多类型，不过 ``mapping``  仍然是受限的。

.. _multi-return:

返回多个值
-------------------------

当函数需要使用多个值，可以用语句 ``return (v0, v1, ..., vn)`` 。
参数的数量需要和声明时候一致。


.. _state-mutability:

状态可变性
================

.. index:: ! view function, function;view

.. _view-functions:

View 视图函数
--------------

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

.. code-block:: solidity

    pragma solidity  >=0.5.0 <0.9.0;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + block.timestamp;
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
--------------

函数可以声明为 ``pure`` ，在这种情况下，承诺不读取也不修改状态变量。

特别是，应该可以在编译时确定一个 ``pure`` 函数，它仅处理输入参数和 ``msg.data`` ，对当前区块链状态没有任何了解。
这也意味着读取 ``immutable`` 变量也不是一个 ``pure`` 操作。


.. note::
    如果编译器的 EVM 编译目标设置为 Byzantium 或之后的版本 (默认), 则使用操作码 ``STATICCALL`` , 这并不保证状态未被读取, 但至少不被修改。


除了上面解释的状态修改语句列表之外，以下被认为是读取状态：

#. 读取状态变量。
#. 访问 ``address(this).balance`` 或者 ``<address>.balance``。
#. 访问 ``block``，``tx``， ``msg`` 中任意成员 （除 ``msg.sig`` 和 ``msg.data`` 之外）。
#. 调用任何未标记为 ``pure`` 的函数。
#. 使用包含某些操作码的内联汇编。

.. code-block:: solidity

    pragma solidity >=0.5.0 <0.9.0;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }

纯函数能够使用 ``revert()`` 和 ``require()`` 在 :ref:`发生错误 <assert-and-require>` 时去还原潜在状态更改。

还原状态更改不被视为 "状态修改", 因为它只还原以前在没有``view`` 或 ``pure`` 限制的代码中所做的状态更改, 并且代码可以选择捕获 ``revert`` 并不传递还原。

这种行为也符合 ``STATICCALL`` 操作码。


.. warning::
  不可能在 EVM 级别阻止函数读取状态, 只能阻止它们写入状态 (即只能在 EVM 级别强制执行 ``view`` , 而 ``pure`` 不能强制)。

.. note::
  在0.5.0 版本之前, 编译器没有对 ``pure`` 函数使用 ``STATICCALL`` 操作码。这样通过使用无效的显式类型转换启用 ``pure`` 函数中的状态修改。
  通过对 ``pure`` 函数使用 ``STATICCALL`` , 可以防止在 EVM 级别上对状态进行修改。


.. note::

  在0.4.17版本之前，编译器不会强制 ``pure`` 函数不读取状态。它是一个编译时类型检查, 可以避免在合约类型之间进行无效的显式转换, 因为编译器可以验证合约类型没有状态更改操作, 但它不会在运行时能检查调用实际的类型。

.. _special-functions:

特别的函数
=================

.. index:: ! receive ether function, function;receive ! receive

.. _receive-ether-function:

receive 接收以太函数
----------------------

一个合约最多有一个 ``receive`` 函数, 声明函数为：
``receive() external payable { ... }``

不需要 ``function`` 关键字，也没有参数和返回值并且必须是　``external``　可见性和　``payable`` 修饰．
它可以是 ``virtual`` 的，可以被重载也可以有 |modifier| 。

在对合约没有任何附加数据调用（通常是对合约转账）是会执行 ``receive`` 函数．　例如　通过 ``.send()`` or ``.transfer()``
如果 ``receive`` 函数不存在，　但是有payable　的 :ref:`fallback 回退函数 <fallback-function>`　
那么在进行纯以太转账时，fallback 函数会调用．　
　
如果两个函数都没有，这个合约就没法通过常规的转账交易接收以太（会抛出异常）．


更糟的是，``receive`` 函数可能只有 2300 gas 可以使用（如，当使用 ``send`` 或 ``transfer`` 时）， 除了基础的日志输出之外，进行其他操作的余地很小。下面的操作消耗会操作 2300  gas :

- 写入存储
- 创建合约
- 调用消耗大量 gas 的外部函数
- 发送以太币


.. warning::
    一个没有定义 fallback 函数或　 receive 函数的合约，直接接收以太币（没有函数调用，即使用 ``send`` 或 ``transfer``）会抛出一个异常，
    并返还以太币（在 Solidity v0.4.0 之前行为会有所不同）。
    所以如果你想让你的合约接收以太币，必须实现receive函数（使用 payable　fallback 函数不再推荐，因为它会让借口混淆）。

.. warning::
    一个没有receive函数的合约，可以作为 *coinbase 交易* （又名 *矿工区块回报* ）的接收者或者作为 ``selfdestruct`` 的目标来接收以太币。

    一个合约不能对这种以太币转移做出反应，因此也不能拒绝它们。这是 EVM 在设计时就决定好的，而且 Solidity 无法绕过这个问题。

    这也意味着 ``address(this).balance`` 可以高于合约中实现的一些手工记帐的总和（例如在receive　函数中更新的累加器记帐）。

下面是一个例子：

.. code-block:: solidity

    pragma solidity ^0.6.0;

    // 这个合约会保留所有发送给它的以太币，没有办法取回。　
    contract Sink {
        event Received(address, uint);
        receive() external payable {
            emit Received(msg.sender, msg.value);
        }
    }


.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fallback 回退函数
-----------------

合约可以最多有一个回退函数。函数声明为： ``fallback () external [payable]`` 或 ``fallback (bytes calldata input) external [payable] returns (bytes memory output)`` 

没有　``function``　关键字。　必须是　``external``　可见性，它可以是 ``virtual`` 的，可以被重载也可以有 |modifier| 。


如果在一个对合约调用中，没有其他函数与给定的函数标识符匹配fallback会被调用．
或者在没有 :ref:`receive 函数 <receive-ether-function>`　时，而没有提供附加数据对合约调用，那么fallback 函数会被执行。

fallback　函数始终会接收数据，但为了同时接收以太时，必须标记为　``payable`` 。

如果使用了带参数的版本， ``input`` 将包含发送到合约的完整数据（等于 ``msg.data`` ），并且通过 ``output`` 返回数据。
返回数据不是 ABI 编码过的数据，相反，它返回不经过修改的数据。


更糟的是，如果回退函数在接收以太时调用，可能只有 2300 gas 可以使用，参考　:ref:`receive接收函数 <receive-ether-function>`

与任何其他函数一样，只要有足够的 gas 传递给它，回退函数就可以执行复杂的操作。

.. warning::
    ``payable`` 的fallback函数也可以在纯以太转账的时候执行， 如果没有　:ref:`receive 以太函数 <receive-ether-function>`
    推荐总是定义一个receive函数，而不是定义一个``payable`` 的fallback函数，

.. note::
    如果想要解码输入数据，那么前四个字节用作函数选择器，然后用 ``abi.decode`` 与数组切片语法一起使用来解码ABI编码的数据：
     ``(c, d) = abi.decode(_input[4:], (uint256, uint256));``

     请注意，这仅应作为最后的手段，而应使用对应的函数。



.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.2 <0.9.0;

    contract Test {
        // 发送到这个合约的所有消息都会调用此函数（因为该合约没有其它函数）。
        // 向这个合约发送以太币会导致异常，因为 fallback 函数没有 `payable` 修饰符
        fallback() external { x = 1; }
        uint x;
    }


    // 这个合约会保留所有发送给它的以太币，没有办法返还。
    contract TestPayable {
        uint x;
        uint y;

        // 除了纯转账外，所有的调用都会调用这个函数．
        // (因为除了 receive 函数外，没有其他的函数).
        // 任何对合约非空calldata 调用会执行回退函数(即使是调用函数附加以太).
        fallback() external payable { x = 1; y = msg.value; }

        // 纯转账调用这个函数，例如对每个空empty calldata的调用
        receive() external payable { x = 2; y = msg.value; }
    }

    contract Caller {
        function callTest(Test test) public returns (bool) {
            (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            //  test.x 结果变成 == 1。

            // address(test) 不允许直接调用 ``send`` ,  因为 ``test`` 没有 payable 回退函数
            //  转化为 ``address payable`` 类型 , 然后才可以调用 ``send``
            address payable testPayable = payable(address(test));


            // 以下将不会编译，但如果有人向该合约发送以太币，交易将失败并拒绝以太币。
            // test.send(2 ether）;
        }

        function callTestPayable(TestPayable test) public returns (bool) {
            (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            // 结果 test.x 为 1  test.y 为 0.
            (success,) = address(test).call{value: 1}(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            // 结果test.x 为1 而 test.y 为 1.

            // 发送以太币, TestPayable 的 receive　函数被调用．
            
            // 因为函数有存储写入, 会比简单的使用 ``send`` or ``transfer``消耗更多的 gas。
            // 因此使用底层的call调用
            (success,) = address(test).call{value: 2 ether}("");
            require(success);

            // 结果 test.x 为 2 而 test.y 为 2 ether.

            return true;
        }

    }

.. index:: ! overload

.. _overload-function:

函数重载
====================

合约可以具有多个不同参数的同名函数，称为“重载”（overloading），这也适用于继承函数。以下示例展示了合约 ``A`` 中的重载函数 ``f``。

.. code-block:: solidity

    pragma solidity >=0.4.16 <0.9.0;

    contract A {
        function f(uint value) public pure returns (uint out) {
            out = value;
        }

        function f(uint value, bool really) public pure returns (uint out) {
            if (really)
                out = value;
        }
    }

重载函数也存在于外部接口中。如果两个外部可见函数仅区别于 Solidity 内的类型而不是它们的外部类型则会导致错误。

.. code-block:: solidity

    // 以下代码无法编译
    pragma solidity >=0.4.16 <0.9.0;

    contract A {
        function f(B value) public pure returns (B out) {
            out = value;
        }

        function f(address value) public pure returns (address out) {
            out = value;
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

.. code-block:: solidity

    pragma solidity >=0.4.16 <0.9.0;

    contract A {
        function f(uint8 val) public pure returns (uint8 out) {
            out = val;
        }

        function f(uint256 val) public pure returns (uint256 out) {
            out = val;
        }
    }

调用  ``f(50)`` 会导致类型错误，因为 ``50`` 既可以被隐式转换为 ``uint8`` 也可以被隐式转换为 ``uint256``。
另一方面，调用 ``f(256)`` 则会解析为 ``f(uint256)`` 重载，因为 ``256`` 不能隐式转换为 ``uint8``。
