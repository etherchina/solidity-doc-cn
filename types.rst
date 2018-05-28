.. index:: type

.. _types:

*****
类型
*****

Solidity 是一种静态类型语言，这意味着每个变量（状态变量和局部变量）都需要在编译时指定变量的类型（或至少可以推导出变量类型——参考下文的 :ref:`type-deduction` ）。
Solidity 提供了几种基本类型，可以用来组合出复杂类型。

除此之外，类型之间可以在包含运算符号的表达式中进行交互。
关于各种运算符号，可以参考 :ref:`order` 。

.. index:: ! value type, ! type;value

值类型
======

以下类型也称为值类型，因为这些类型的变量将始终按值来传递。
也就是说，当这些变量被用作函数参数或者用在赋值语句中时，总会进行值拷贝。

.. index:: ! bool, ! true, ! false

布尔类型
--------

``bool`` ：可能的取值为字面常数值 ``true`` 和 ``false`` 。

运算符：

*  ``!`` （逻辑非）
*  ``&&`` （逻辑与， "and" ）
*  ``||`` （逻辑或， "or" ）
*  ``==`` （等于）
*  ``!=`` （不等于）

运算符 ``||`` 和 ``&&`` 都遵循同样的短路（ short-circuiting ）规则。就是说在表达式 ``f(x) || g(y)`` 中，
如果 ``f(x)`` 的值为 ``true`` ，那么 ``g(y)`` 就不会被执行，即使会出现一些副作用。

.. index:: ! uint, ! int, ! integer

整型
----

``int`` / ``uint`` ：分别表示有符号和无符号的不同位数的整型变量。
支持关键字 ``uint8`` 到 ``uint256`` （无符号，从 8 位到 256 位）以及 ``int8`` 到 ``int256``，以 ``8`` 位为步长递增。
``uint`` 和 ``int`` 分别是 ``uint256`` 和 ``int256`` 的别名。

运算符：

* 比较运算符： ``<=`` ， ``<`` ， ``==`` ， ``!=`` ， ``>=`` ， ``>`` （返回布尔值）
* 位运算符： ``&`` ， ``|`` ， ``^`` （异或）， ``~`` （位取反）
* 算数运算符： ``+`` ， ``-`` ， 一元运算 ``-`` ， 一元运算 ``+`` ， ``*`` ， ``/`` ， ``%`` （取余） ， ``**`` （幂）， ``<<`` （左移位） ， ``>>`` （右移位）

除法总是会截断的（仅被编译为 EVM 中的 ``DIV`` 操作码），
但如果操作数都是 :ref:`字面常数（literals）<rational_literals>` （或者字面常数表达式），则不会截断。

除以零或者模零运算都会引发运行时异常。

移位运算的结果取决于运算符左边的类型。
表达式 ``x << y`` 与 ``x * 2**y`` 是等价的，
``x >> y`` 与 ``x / 2**y`` 是等价的。这意味对一个负数进行移位会导致其符号消失。
按负数位移动会引发运行时异常。

.. warning::
   由有符号整数类型负值右移所产生的结果跟其它语言中所产生的结果是不同的。
   在 Solidity 中，右移和除是等价的，因此对一个负数进行右移操作会导致向 0 的取整（截断）。
   而在其它语言中， 对负数进行右移类似于（向负无穷）取整。

.. index:: ! ufixed, ! fixed, ! fixed point number

定长浮点型
----------

.. warning::
    Solidity 还没有完全支持定长浮点型。可以声明定长浮点型的变量，但不能给它们赋值或把它们赋值给其他变量。。

``fixed`` / ``ufixed``：表示各种大小的有符号和无符号的定长浮点型。
在关键字 ``ufixedMxN`` 和 ``fixedMxN`` 中，``M`` 表示该类型占用的位数，``N`` 表示可用的小数位数。
``M`` 必须能整除 8，即 8 到 256 位。
``N`` 则可以是从 0 到 80 之间的任意数。
``ufixed`` 和 ``fixed`` 分别是 ``ufixed128x19`` 和 ``fixed128x19`` 的别名。

运算符：

* 比较运算符：``<=``， ``<``， ``==``， ``!=``， ``>=``， ``>`` （返回值是布尔型）
* 算术运算符：``+``， ``-``， 一元运算 ``-``， 一元运算 ``+``， ``*``， ``/``， ``%`` （取余数）

.. note::
    浮点型（在许多语言中的 ``float`` 和 ``double`` 类型，更准确地说是 IEEE 754 类型）和定长浮点型之间最大的不同点是，
    在前者中整数部分和小数部分（小数点后的部分）需要的位数是灵活可变的，而后者中这两部分的长度受到严格的规定。
    一般来说，在浮点型中，几乎整个空间都用来表示数字，但只有少数的位来表示小数点的位置。

.. index:: address, balance, send, call, callcode, delegatecall, transfer

.. _address:

地址类型
------

``address``：地址类型存储一个 20 字节的值（以太坊地址的大小）。
地址类型也有成员变量，并作为所有合约的基础。

运算符：

* ``<=``， ``<``， ``==``， ``!=``， ``>=`` 和 ``>``

.. note::
    从 0.5.0 版本开始，合约不会从地址类型派生，但仍然可以显式地转换成地址类型。

.. _members-of-addresses:

地址类型成员变量
^^^^^^^^^^^^^^^^

* ``balance`` 和 ``transfer``

快速参考，请见 :ref:`address_related`。

可以使用 ``balance`` 属性来查询一个地址的余额，
也可以使用 ``transfer`` 函数向一个地址发送 |ether| （以 wei 为单位）：

::

    address x = 0x123;
    address myAddress = this;
    if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);

.. note::
    如果 ``x`` 是一个合约地址，它的代码（更具体来说是它的 fallback 函数，如果有的话）会跟 ``transfer`` 函数调用一起执行（这是 EVM 的一个特性，无法阻止）。
    如果在执行过程中用光了 gas 或者因为任何原因执行失败，|ether| 交易会被打回，当前的合约也会在终止的同时抛出异常。

* ``send``

``send`` 是 ``transfer`` 的低级版本。如果执行失败，当前的合约不会因为异常而终止，但 ``send`` 会返回 ``false``。

.. warning::
    在使用 ``send`` 的时候会有些风险：如果调用栈深度是 1024 会导致发送失败（这总是可以被调用者强制），如果接收者用光了 gas 也会导致发送失败。
    所以为了保证 |ether| 发送的安全，一定要检查 ``send`` 的返回值，使用 ``transfer`` 或者更好的办法：
    使用一种接收者可以取回资金的模式。

* ``call``， ``callcode`` 和 ``delegatecall``

此外，为了与不符合 |ABI| 的合约交互，于是就有了可以接受任意类型任意数量参数的 ``call`` 函数。
这些参数会被打包到以 32 字节为单位的连续区域中存放。
其中一个例外是当第一个参数被编码成正好 4 个字节的情况。
在这种情况下，这个参数后边不会填充后续参数编码，以允许使用函数签名。

::

    address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
    nameReg.call("register", "MyName");
    nameReg.call(bytes4(keccak256("fun(uint256)")), a);

``call`` 返回的布尔值表明了被调用的函数已经执行完毕（``true``）或者引发了一个 EVM 异常（``false``）。
无法访问返回的真实数据（为此我们需要事先知道编码和大小）。

可以使用 ``.gas()`` |modifier| 调整提供的 gas 数量 ::

    namReg.call.gas(1000000)("register", "MyName");

类似地，也能控制提供的 |ether| 的值 ::

   nameReg.call.value(1 ether)("register", "MyName"); 

最后一点，这些 |modifier| 可以联合使用。每个修改器出现的顺序不重要 ::

   nameReg.call.gas(1000000).value(1 ether)("register", "MyName"); 

.. note::
    目前还不能在重载函数中使用 gas 或者 value |modifier| 。

    一种解决方案是给 gas 和值引入一个特例，并重新检查它们是否在重载的地方出现。

类似地，也可以使用 ``delegatecall``：
区别在于只使用给定地址的代码，其它属性（存储，余额，……）都取自当前合约。
``delegatecall`` 的目的是使用存储在另外一个合约中的库代码。
用户必须确保两个合约中的存储结构都适用于 delegatecall。
在 homestead 版本之前，只有一个功能类似但作用有限的 ``callcode`` 的函数可用，但它不能获取委托方的 ``msg.sender`` 和 ``msg.value``。

这三个函数 ``call``， ``delegatecall`` 和 ``callcode`` 都是非常低级的函数，应该只把它们当作 *最后一招* 来使用，因为它们破坏了 Solidity 的类型安全性。

.. note::
    所有合约都继承了地址（address）的成员变量，因此可以使用 ``this.balance`` 查询当前合约的余额。

.. note::
    不鼓励使用 ``callcode``，在未来也会将其移除。

.. warning::
    这三个函数都属于低级函数，需要谨慎使用。
    具体来说，任何未知的合约都可能是恶意的。
    你在调用一个合约的同时就将控制权交给了它，它可以反过来调用你的合约，
    因此，当调用返回时要为你的状态变量的改变做好准备。

.. index:: byte array, bytes32

定长字节数组
------------

关键字有：``bytes1``， ``bytes2``， ``bytes3``， ...， ``bytes32``。``byte`` 是 ``bytes1`` 的别名。

运算符：

* 比较运算符：``<=``， ``<``， ``==``， ``!=``， ``>=``， ``>`` （返回布尔型）
* 位运算符： ``&``， ``|``， ``^`` （按位异或）， ``~`` （按位取反）， ``<<`` （左移位）， ``>>`` （右移位）
* 索引访问：如果 ``x`` 是 ``bytesI`` 类型，那么 ``x[k]`` （其中 ``0 <= k < I``）返回第 ``k`` 个字节（只读）。

该类型可以和作为右操作数的任何整数类型进行移位运算（但返回结果的类型和左操作数类型相同），右操作数表示需要移动的位数。
进行负数位移运算会引发运行时异常。

成员变量：

* ``.length`` 表示这个字节数组的长度（只读）.

.. note::
    可以将 ``byte[]`` 当作字节数组使用，但这种方式非常浪费存储空间，准确来说，是在传入调用时，每个元素会浪费 31 字节。
    更好地做法是使用 ``bytes``。

变长字节数组
------------

``bytes``:
    变长字节数组，参见 :ref:`arrays`。它并不是值类型。
``string``:
    变长 UTF-8 编码字符串类型，参见 :ref:`arrays`。并不是值类型。

.. index:: address, literal;address

.. _address_literals:

地址字面常数（Address Literals）
----------------------------

比如像 ``0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF`` 这样的通过了地址校验和测试的十六进制字面常数属于 ``address`` 类型。
长度在 39 到 41 个数字的，没有通过校验和测试而产生了一个警告的十六进制字面常数视为正常的有理数字面常数。

.. note::
    混合大小写的地址校验和格式定义在 `EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_ 中。

.. index:: literal, literal;rational

.. _rational_literals:

有理数和整数字面常数
----------------

整数字面常数由范围在 0-9 的一串数字组成，表现成十进制。
例如，`69` 表示数字 69。
Solidity 中是没有八进制的，因此前置 0 是无效的。

十进制小数字面常数带有一个 ``.``，至少在其一边会有一个数字。
比如：``1.``，``.1``，和 ``1.3``。

科学符号也是支持的，尽管指数必须是整数，但底数可以是小数。
比如：``2e10``， ``-2e10``， ``2e-10``， ``2.5e1``。

数值字面常数表达式本身支持任意精度，除非它们被转换成了非字面常数类型（也就是说，当它们出现在非字面常数表达式中时就会发生转换）。
这意味着在数值常量表达式中, 计算不会溢出而除法也不会截断。

例如， ``(2**800 + 1) - 2**800`` 的结果是字面常数 ``1`` （属于 ``uint8`` 类型），尽管计算的中间结果已经超过了 |evm| 的机器字长度。
此外， ``.5 * 8`` 的结果是整型 ``4`` （尽管有非整型参与了计算）。

只要操作数是整型，任意整型支持的运算符都可以被运用在数值字面常数表达式中。
如果两个中的任一个数是小数，则不允许进行位运算。如果指数是小数的话，也不支持幂运算（因为这样可能会得到一个无理数）。

.. note::
    Solidity 对每个有理数都有对应的数值字面常数类型。
    整数字面常数和有理数字面常数都属于数值字面常数类型。
    除此之外，所有的数值字面常数表达式（即只包含数值字面常数和运算符的表达式）都属于数值字面常数类型。
    因此数值字面常数表达式 ``1 + 2`` 和 ``2 + 1`` 的结果跟有理数三的数值字面常数类型相同。

.. warning::
    在早期版本中，整数字面常数的除法也会截断，但在现在的版本中，会将结果转换成一个有理数。即 ``5 / 2`` 并不等于 ``2``，而是等于 ``2.5``。

.. note::
    数值字面常数表达式只要在非字面常数表达式中使用就会转换成非字面常数类型。
    在下面的例子中，尽管我们知道 ``b`` 的值是一个整数，但 ``2.5 + a`` 这部分表达式并不进行类型检查，因此编译不能通过。

::

    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5;

.. index:: literal, literal;string, string

字符串字面常数
----------

字符串字面常数是指由双引号或单引号引起来的字符串（``"foo"`` 或者 ``'bar'``）。
不像在 C 语言中那样带有结束符；``"foo"`` 相当于 3 个字节而不是 4 个。
和整数字面常数一样，字符串字面常数的类型也可以发生改变，但它们可以隐式地转换成 ``bytes1``，……，``bytes32``，如果合适的话，还可以转换成 ``bytes`` 以及 ``string``。

字符串字面常数支持转义字符，例如 ``\n``，``\xNN`` 和 ``\uNNNN``。``\xNN`` 表示一个 16 进制值，最终转换成合适的字节，
而 ``\uNNNN`` 表示 Unicode 编码值，最终会转换为 UTF-8 的序列。

.. index:: literal, bytes

十六进制字面常数
------------

十六进制字面常数以关键字 ``hex`` 打头，后面紧跟着用单引号或双引号引起来的字符串（例如，``hex"001122FF"``）。
字符串的内容必须是一个十六进制的字符串，它们的值将使用二进制表示。

十六进制字面常数跟字符串字面常数很类似，具有相同的转换规则。

.. index:: enum

.. _enums:

枚举类型
--------

::

    pragma solidity ^0.4.16;

    contract test {
        enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
        ActionChoices choice;
        ActionChoices constant defaultChoice = ActionChoices.GoStraight;

        function setGoStraight() public {
            choice = ActionChoices.GoStraight;
        }

        // 由于枚举类型不属于 |ABI| 的一部分，因此对于所有来自 Solidity 外部的调用，
        // "getChoice" 的签名会自动被改成 "getChoice() returns (uint8)"。
        // 整数类型的大小已经足够存储所有枚举类型的值，随着值的个数增加，
        // 可以逐渐使用 `uint16` 或更大的整数类型。
        function getChoice() public view returns (ActionChoices) {
            return choice;
        }

        function getDefaultChoice() public pure returns (uint) {
            return uint(defaultChoice);
        }
    }

.. index:: ! function type, ! type; function

.. _function_types:

函数类型
---------

函数类型是一种表示函数的类型。可以将一个函数赋值给另一个函数类型的变量，也可以将一个函数作为参数进行传递，还能在函数调用中返回函数类型变量。
函数类型有两类：- *内部（internal）* 函数和 *外部（external）* 函数：

内部函数只能在当前合约内被调用（更具体来说，在当前代码块内，包括内部库函数和继承的函数中），因为它们不能在当前合约上下文的外部被执行。
调用一个内部函数是通过跳转到它的入口标签来实现的，就像在当前合约的内部调用一个函数。

外部函数由一个地址和一个函数签名组成，可以通过外部函数调用传递或者返回。

函数类型表示成如下的形式 ::

    function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]

与参数类型相反，返回类型不能为空 —— 如果函数类型不需要返回，则需要删除整个 ``returns (<return types>)`` 部分。

函数类型默认是内部函数，因此不需要声明 ``internal`` 关键字。
与此相反的是，合约中的函数本身默认是 public 的，只有当它被当做类型名称时，默认才是内部函数。

有两种方法可以访问当前合约中的函数：一种是直接使用它的名字，``f`` ，另一种是使用 ``this.f`` 。
前者适用于内部函数，后者适用于外部函数。

如果当函数类型的变量还没有初始化时就调用它的话会引发一个异常。
如果在一个函数被 ``delete`` 之后调用它也会发生相同的情况。

如果外部函数类型在 Solidity 的上下文环境以外的地方使用，它们会被视为 ``function`` 类型。
该类型将函数地址紧跟其函数标识一起编码为一个 ``bytes24`` 类型。。

请注意，当前合约的 public 函数既可以被当作内部函数也可以被当作外部函数使用。
如果想将一个函数当作内部函数使用，就用 ``f`` 调用，如果想将其当作外部函数，使用 ``this.f`` 。

除此之外，public（或 external）函数也有一个特殊的成员变量称作 ``selector``，可以返回 :ref:`ABI 函数选择器 <abi_function_selector>`::

    pragma solidity ^0.4.16;

    contract Selector {
      function f() public view returns (bytes4) {
        return this.f.selector;
      }
    }

如果使用内部函数类型的例子::

    pragma solidity ^0.4.16;

    library ArrayUtils {
      // 内部函数可以在内部库函数中使用，
      // 因为它们会成为同一代码上下文的一部分
      function map(uint[] memory self, function (uint) pure returns (uint) f)
        internal
        pure
        returns (uint[] memory r)
      {
        r = new uint[](self.length);
        for (uint i = 0; i < self.length; i++) {
          r[i] = f(self[i]);
        }
      }
      function reduce(
        uint[] memory self,
        function (uint, uint) pure returns (uint) f
      )
        internal
        pure
        returns (uint r)
      {
        r = self[0];
        for (uint i = 1; i < self.length; i++) {
          r = f(r, self[i]);
        }
      }
      function range(uint length) internal pure returns (uint[] memory r) {
        r = new uint[](length);
        for (uint i = 0; i < r.length; i++) {
          r[i] = i;
        }
      }
    }

    contract Pyramid {
      using ArrayUtils for *;
      function pyramid(uint l) public pure returns (uint) {
        return ArrayUtils.range(l).map(square).reduce(sum);
      }
      function square(uint x) internal pure returns (uint) {
        return x * x;
      }
      function sum(uint x, uint y) internal pure returns (uint) {
        return x + y;
      }
    }

另外一个使用外部函数类型的例子::

    pragma solidity ^0.4.11;

    contract Oracle {
      struct Request {
        bytes data;
        function(bytes memory) external callback;
      }
      Request[] requests;
      event NewRequest(uint);
      function query(bytes data, function(bytes memory) external callback) public {
        requests.push(Request(data, callback));
        NewRequest(requests.length - 1);
      }
      function reply(uint requestID, bytes response) public {
        // 这里要验证 reply 来自可信的源
        requests[requestID].callback(response);
      }
    }

    contract OracleUser {
      Oracle constant oracle = Oracle(0x1234567); // 已知的合约
      function buySomething() {
        oracle.query("USD", this.oracleResponse);
      }
      function oracleResponse(bytes response) public {
        require(msg.sender == address(oracle));
        // 使用数据
      }
    }

.. note::
    Lambda 表达式或者内联函数的引入在计划内，但目前还没支持。

.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

引用类型
========

比起之前讨论过的值类型，在处理复杂的类型（即占用的空间超过 256 位的类型）时，我们需要更加谨慎。
由于拷贝这些类型变量的开销相当大，我们不得不考虑它的存储位置，是将它们保存在 ** |memory| ** （并不是永久存储）中，
还是 ** |storage| ** （保存状态变量的地方）中。

.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

数据位置
---------

所有的复杂类型，即 *数组* 和 *结构* 类型，都有一个额外属性，“数据位置”，说明数据是保存在 |memory| 中还是 |storage| 中。
根据上下文不同，大多数时候数据有默认的位置，但也可以通过在类型名后增加关键字 ``storage`` 或 ``memory`` 进行修改。
函数参数（包括返回的参数）的数据位置默认是 ``memory``，
局部变量的数据位置默认是 ``storage``，状态变量的数据位置强制是 ``storage`` （这是显而易见的）。

也存在第三种数据位置， ``calldata`` ，这是一块只读的，且不会永久存储的位置，用来存储函数参数。
外部函数的参数（非返回参数）的数据位置被强制指定为 ``calldata`` ，效果跟 ``memory`` 差不多。

数据位置的指定非常重要，因为它们影响着赋值行为：
在 |storage| 和 |memory| 之间两两赋值，或者 |storage| 向状态变量（甚至是从其它状态变量）赋值都会创建一份独立的拷贝。
然而状态变量向局部变量赋值时仅仅传递一个引用，而且这个引用总是指向状态变量，因此后者改变的同时前者也会发生改变。
另一方面，从一个 |memory| 存储的引用类型向另一个 |memory| 存储的引用类型赋值并不会创建拷贝。

::

    pragma solidity ^0.4.0;

    contract C {
        uint[] x; // x 的数据存储位置是 storage

        // memoryArray 的数据存储位置是 memory
        function f(uint[] memoryArray) public {
            x = memoryArray; // 将整个数组拷贝到 storage 中，可行
            var y = x;  // 分配一个指针（其中 y 的数据存储位置是 storage），可行
            y[7]; // 返回第 8 个元素，可行
            y.length = 2; // 通过 y 修改 x，可行
            delete x; // 清除数组，同时修改 y，可行
            // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组， /
            // 但 storage 是“静态”分配的：
            // y = memoryArray;
            // 下面这一行也不可行，因为这会“重置”指针，
            // 但并没有可以让它指向的合适的存储位置。
            // delete y;
            
            g(x); // 调用 g 函数，同时移交对 x 的引用
            h(x); // 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
        }

        function g(uint[] storage storageArray) internal {}
        function h(uint[] memoryArray) public {}
    }

总结
^^^^^

强制指定的数据位置：
 - 外部函数的参数（不包括返回参数）： calldata
 - 状态变量： storage

默认数据位置：
 - 函数参数（包括返回参数）： memory
 - 所有其它局部变量： storage

.. index:: ! array

.. _arrays:

数组
-----

数组可以在声明时指定长度，也可以动态调整大小。
对于 |storage| 的数组来说，元素类型可以是任意的（即元素也可以是数组类型，映射类型或者结构体）。
对于 |memory| 的数组来说，元素类型不能是映射类型，如果作为 public 函数的参数，它只能是 ABI 类型。

一个元素类型为 ``T``，固定长度为 ``k`` 的数组可以声明为 ``T[k]``，而动态数组声明为 ``T[]``。
举个例子，一个长度为 5，元素类型为 ``uint`` 的动态数组的数组，应声明为 ``uint[][5]`` （注意这里跟其它语言比，数组长度的声明位置是反的）。
要访问第三个动态数组的第二个元素，你应该使用 x[2][1]（数组下标是从 0 开始的，且访问数组时的下标顺序与声明时相反，也就是说，x[2] 是从右边减少了一级）。。

``bytes`` 和 ``string`` 类型的变量是特殊的数组。
``bytes`` 类似于 ``byte[]``，但它在 calldata 中会被“紧打包”（译者注：将元素连续地存在一起，不会按每 32 字节一单元的方式来存放）。
``string`` 与 ``bytes`` 相同，但（暂时）不允许用长度或索引来访问。

.. note::
    如果想要访问以字节表示的字符串 ``s``，请使用 ``bytes(s).length`` / ``bytes(s)[7] = 'x';``。
    注意这时你访问的是 UTF-8 形式的低级 bytes 类型，而不是单个的字符。

可以将数组标识为 ``public``，从而让 Solidity 创建一个 :ref:`getter <visibility-and-getters>`。
之后必须使用数字下标作为参数来访问 getter。

.. index:: ! array;allocating, new

创建内存数组
^^^^^^^^^^^^^

可使用 ``new`` 关键字在内存中创建变长数组。
与 |storage| 数组相反的是，你 *不能* 通过修改成员变量 ``.length`` 改变 |memory| 数组的大小。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);
            // 这里我们有 a.length == 7 以及 b.length == len
            a[6] = 8;
        }
    }

.. index:: ! array;literals, !inline;arrays

数组字面常数 / 内联数组
^^^^^^^^^^^^^^^^^^^

数组字面常数是写作表达式形式的数组，并且不会立即赋值给变量。

::

    pragma solidity ^0.4.16;

    contract C {
        function f() public pure {
            g([uint(1), 2, 3]);
        }
        function g(uint[3] _data) public pure {
            // ...
        }
    }

数组字面常数是一种定长的 |memory| 数组类型，它的基础类型由其中元素的普通类型决定。
例如，``[1, 2, 3]`` 的类型是 ``uint8[3] memory``，因为其中的每个字面常数的类型都是 ``uint8``。
正因为如此，有必要将上面这个例子中的第一个元素转换成 ``uint`` 类型。
目前需要注意的是，定长的 |memory| 数组并不能赋值给变长的 |memory| 数组，下面是个反例：

::

    // 这段代码并不能编译。

    pragma solidity ^0.4.0;

    contract C {
        function f() public {
            // 这一行引发了一个类型错误，因为 unint[3] memory
            // 不能转换成 uint[] memory。
            uint[] x = [uint(1), 3, 4];
        }
    }

已经计划在未来移除这样的限制，但目前数组在 ABI 中传递的问题造成了一些麻烦。

.. index:: ! array;length, length, push, !array;push

成员
^^^^^^

**length**:
    数组有 ``length`` 成员变量表示当前数组的长度。
    动态数组可以在 |storage| （而不是 |memory| ）中通过改变成员变量 ``.length`` 改变数组大小。
    并不能通过访问超出当前数组长度的方式实现自动扩展数组的长度。
    一经创建，|memory| 数组的大小就是固定的（但却是动态的，也就是说，它依赖于运行时的参数）。

**push**:
    变长的 |storage| 数组以及 ``bytes`` 类型（而不是 ``string`` 类型）都有一个叫做 ``push`` 的成员函数，它用来附加新的元素到数组末尾。
    这个函数将返回新的数组长度。

.. warning::
    在外部函数中目前还不能使用多维数组。

.. warning::
    由于 |evm| 的限制，不能通过外部函数调用返回动态的内容。
    例如，如果通过 web3.js 调用 ``contract C { function f() returns (uint[]) { ... } }`` 中的 ``f`` 函数，它会返回一些内容，但通过 Solidity 不可以。

    目前唯一的变通方法是使用大型的静态数组。

::

    pragma solidity ^0.4.16;

    contract ArrayContract {
        uint[2**20] m_aLotOfIntegers;
        // 注意下面的代码并不是一对动态数组，
        // 而是一个数组元素为一对变量的动态数组（也就是数组元素为长度为 2 的定长数组的动态数组）。
        bool[2][] m_pairsOfFlags;
        // newPairs 存储在 memory 中 —— 函数参数默认的存储位置

        function setAllFlagPairs(bool[2][] newPairs) public {
            // 向一个 storage 的数组赋值会替代整个数组
            m_pairsOfFlags = newPairs;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // 访问一个不存在的数组下标会引发一个异常
            m_pairsOfFlags[index][0] = flagA;
            m_pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // 如果 newSize 更小，那么超出的元素会被清除
            m_pairsOfFlags.length = newSize;
        }

        function clear() public {
            // 这些代码会将数组全部清空
            delete m_pairsOfFlags;
            delete m_aLotOfIntegers;
            // 这里也是实现同样的功能
            m_pairsOfFlags.length = 0;
        }

        bytes m_byteData;

        function byteArrays(bytes data) public {
            // 字节的数组（语言意义中的 byte 的复数 ``bytes``）不一样，因为它们不是填充式存储的，
            // 但可以当作和 "uint8[]" 一样对待
            m_byteData = data;
            m_byteData.length += 7;
            m_byteData[3] = byte(8);
            delete m_byteData[2];
        }

        function addFlag(bool[2] flag) public returns (uint) {
            return m_pairsOfFlags.push(flag);
        }

        function createMemoryArray(uint size) public pure returns (bytes) {
            // 使用 `new` 创建动态 memory 数组：
            uint[2][] memory arrayOfPairs = new uint[2][](size);
            // 创建一个动态字节数组：
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = byte(i);
            return b;
        }
    }


.. index:: ! struct, ! type;struct

.. _structs:

结构体
-------

Solidity 支持通过构造结构体的形式定义新的类型，以下是一个结构体使用的示例：

::

    pragma solidity ^0.4.11;

    contract CrowdFunding {
        // 定义的新类型包含两个属性。
        struct Funder {
            address addr;
            uint amount;
        }

        struct Campaign {
            address beneficiary;
            uint fundingGoal;
            uint numFunders;
            uint amount;
            mapping (uint => Funder) funders;
        }

        uint numCampaigns;
        mapping (uint => Campaign) campaigns;

        function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
            campaignID = numCampaigns++; // campaignID 作为一个变量返回
            // 创建新的结构体示例，存储在 storage 中。我们先不关注映射类型。
            campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // 以给定的值初始化，创建一个新的临时 memory 结构体，
            // 并将其拷贝到 storage 中。
            // 注意你也可以使用 Funder(msg.sender, msg.value) 来初始化。
            c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
            c.amount += msg.value;
        }

        function checkGoalReached(uint campaignID) public returns (bool reached) {
            Campaign storage c = campaigns[campaignID];
            if (c.amount < c.fundingGoal)
                return false;
            uint amount = c.amount;
            c.amount = 0;
            c.beneficiary.transfer(amount);
            return true;
        }
    }

上面的合约只是一个简化版的众筹合约，但它已经足以让我们理解结构体的基础概念。
结构体类型可以作为元素用在映射和数组中，其自身也可以包含映射和数组作为成员变量。

尽管结构体本身可以作为映射的值类型成员，但它并不能包含自身。
这个限制是有必要的，因为结构体的大小必须是有限的。

注意在函数中使用结构体时，一个结构体是如何赋值给一个局部变量（默认存储位置是 |storage| ）的。
在这个过程中并没有拷贝这个结构体，而是保存一个引用，所以对局部变量成员的赋值实际上会被写入状态。

当然，你也可以直接访问结构体的成员而不用将其赋值给一个局部变量，就像这样，
``campaigns[campaignID].amount = 0``。

.. index:: !mapping

映射
=====

映射类型在声明时的形式为 ``mapping(_KeyType => _ValueType)``。
其中 ``_KeyType`` 可以是除了映射、变长数组、合约、枚举以及结构体以外的几乎所有类型。
``_ValueType`` 可以是包括映射类型在内的任何类型。

映射可以视作 `哈希表 <https://en.wikipedia.org/wiki/Hash_table>`，它们在实际的初始化过程中创建每个可能的 key，
并将其映射到字节形式全是零的值：一个类型的 :ref:`默认值 <default-value>`。然而下面是映射与哈希表不同的地方：
在映射中，实际上并不存储 key，而是存储它的 ``keccak256`` 哈希值，从而便于查询实际的值。

正因为如此，映射是没有长度的，也没有 key 的集合或 value 的集合的概念。

只有状态变量（或者在 internal 函数中的对于存储变量的引用）可以使用映射类型。。

可以将映射声明为 ``public``，然后来让 Solidity 创建一个 :ref:`getter <visibility-and-getters>`。
``_KeyType`` 将成为 getter 的必须参数，并且 getter 会返回 ``_ValueType``。

``_ValueType`` 也可以是一个映射。这时在使用 getter 时将将需要递归地传入每个 ``_KeyType`` 参数。

::

    pragma solidity ^0.4.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingUser {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(this);
        }
    }


.. note::
  递归不支持迭代，但可以在此之上实现一个这样的数据结构。
  例子可以参考 `可迭代的映射 <https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol>`_。

.. index:: assignment, ! delete, lvalue

涉及 LValues 的运算符
=====================

如果 ``a`` 是一个 LValue（即一个变量或者其它可以被复制的东西），以下运算符都可以使用简写：

``a += e`` 等同于 ``a = a + e``。 其它运算符 ``-=``， ``*=``， ``/=``， ``%=``， ``|=``， ``&=`` 以及 ``^=`` 都是如此定义的。
``a++`` 和 ``a--`` 分别等同于 ``a += 1`` 和 ``a -= 1``，但表达式本身的值等于 ``a`` 在计算之前的值。
与之相反，``--a`` 和 ``++a`` 虽然最终 ``a`` 的结果与之前的表达式相同，但表达式的返回值是计算之后的值。

删除
-----

``delete a`` 的结果是将 ``a`` 的类型在初始化时的值赋值给 ``a``。即对于整型变量来说，相当于 ``a = 0``，
但 delete 也适用于数组，对于动态数组来说，是将数组的长度设为 0，而对于静态数组来说，是将数组中的所有元素重置。
如果对象是结构体，则将结构体中的所有属性重置。

``delete`` 对整个映射是无效的（因为映射的键可以是任意的，通常也是未知的）。
因此在你删除一个结构体时，结果将重置所有的非映射属性，这个过程是递归进行的，除非它们是映射。
然而，单个的键及其映射的值是可以被删除的。

理解 ``delete a`` 的效果就像是给 ``a`` 赋值很重要，换句话说，这相当于在 ``a`` 中存储了一个新的对象。

::

    pragma solidity ^0.4.0;

    contract DeleteExample {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // 将 x 设为 0，并不影响数据
            delete data; // 将 data 设为 0，并不影响 x，因为它仍然有个副本
            uint[] storage y = dataArray;
            delete dataArray; 
            // 将 dataArray.length 设为 0，但由于 uint[] 是一个复杂的对象，y 也将受到影响，
            // 因为它是一个存储位置是 storage 的对象的别名。
            // 另一方面："delete y" 是非法的，引用了 storage 对象的局部变量只能由已有的 storage 对象赋值。
        }
    }

.. index:: ! type;conversion, ! cast

基本类型之间的转换
==================

隐式转换
---------

如果一个运算符用在两个不同类型的变量之间，那么编译器将隐式地将其中一个类型转换为另一个类型（不同类型之间的赋值也是一样）。
一般来说，只要值类型之间的转换在语义上行得通，而且转换的过程中没有信息丢失，那么隐式转换基本都是可以实现的：
``uint8`` 可以转换成 ``uint16``，``int128`` 转换成 ``int256``，但 ``int8`` 不能转换成 ``uint256``
（因为 ``uint256`` 不能涵盖某些值，例如，``-1``）。
更进一步来说，无符号整型可以转换成跟它大小相等或更大的字节类型，但反之不能。
任何可以转换成 ``uint160`` 的类型都可以转换成 ``address`` 类型。

显式转换
---------

如果某些情况下编译器不支持隐式转换，但是你很清楚你要做什么，这种情况可以考虑显式转换。
注意这可能会发生一些无法预料的后果，因此一定要进行测试，确保结果是你想要的！
下面的示例是将一个 ``int8`` 类型的负数转换成 ``uint``：

::

    int8 y = -3;
    uint x = uint(y);

这段代码的最后，``x`` 的值将是 ``0xfffff..fd`` （64 个 16 进制字符），因为这是 -3 的 256 位补码形式。

如果一个类型显式转换成更小的类型，相应的高位将被舍弃 ::

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // 此时 b 的值是 0x5678

.. index:: ! type;deduction, ! var

.. _type-deduction:

类型推断
=========

为了方便起见，没有必要每次都精确指定一个变量的类型，编译器会根据分配该变量的第一个表达式的类型自动推断该变量的类型 ::

    uint24 x = 0x123;
    var y = x;

这里 ``y`` 的类型将是 ``uint24``。不能对函数参数或者返回参数使用 ``var``。

.. warning::
    类型只能从第一次赋值中推断出来，因此以下代码中的循环是无限的，
    原因是``i`` 的类型是 ``uint8``，而这个类型变量的最大值比 ``2000`` 小。
    ``for (var i = 0; i < 2000; i++) { ... }``