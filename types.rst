.. include:: glossaries.rst
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
===========

以下类型也称为值类型，因为这些类型的变量将始终按值来传递。
也就是说，当这些变量被用作函数参数或者被赋值时，总会进行值拷贝。

.. index:: ! bool, ! true, ! false

布尔类型
--------

``bool`` ：可能的取值为常量值 ``true`` 和 ``false`` 。

运算符：

*  ``!`` （逻辑非）
*  ``&&`` （逻辑与， "and" ）
*  ``||`` （逻辑或， "or " ）
*  ``==`` （等于）
*  ``!=`` （不等于）

运算符 ``||`` 和 ``&&`` 都遵循同样的短路（ short-circuiting ）规则。就是说在表达式 ``f(x) || g(y)`` 中，
如果 ``f(x)`` 的值为 ``true`` ，那么 ``g(y)`` 就不会被执行，即使会出现一些副作用。

.. index:: ! uint, ! int, ! integer

整型
---

``int`` / ``uint`` ：本别表示有符号和无符号的不同位数的整型变量。支持关键字 ``uint8`` 到 ``uint256`` （8位递增，从8位到256位）以及 ``int8`` 到 ``int256``。
``uint`` 和 ``int`` 分别代表 ``uint256`` 和 ``int256``。

运算符：

* 比较运算符： ``<=`` ， ``<`` ， ``==`` ， ``!=`` ， ``>=`` ， ``>`` （返回布尔值）
* 位运算符： ``&`` ， ``|`` ， ``^`` （异或）， ``~`` （位取反）
* 算数运算符： ``+`` ， ``-`` ， 一元运算 ``-`` ， 一元运算 ``+`` ， ``*`` ， ``/`` ， ``%`` （取余） ， ``**`` （幂）， ``<<`` （左移位） ， ``>>`` （右移位）

除法总是会截断的（仅仅编译到 EVM 中的 ``DIV`` 操作码），
但如果操作数都是 :ref:`常量（literals）<rational_literals>` （或者常量表达式），则不会截断。

除以零或者模零运算都会引发运行时异常。

移位运算的结果取决于运算符左边的类型。
表达式 ``x << y`` 与 ``x * 2**y`` 是等价的，
``x >> y`` 与 ``x / 2**y``是等价的。这意味着将负数符号转移。
按负数位移动会引发运行时异常。

·· warning::
   由有符号整数类型负值右移所产生的结果跟其它语言中所产生的结果是不同的。
   在 Solidity 中，右移和除是等价的，因此右移位一个负数向下取整时会为零（被截断）。
   而在其它语言中， 右移负数位的结果就像除以了负无穷。

.. index:: ! ufixed, ! fixed, ! fixed point number

定长浮点型
--------

.. warning::
    Solidity 还没有完全支持定长浮点型。可以声明定长浮点型的变量，但不能给它们赋值。

``fixed`` / ``ufixed``：表示各种大小的有符号和无符号的定长浮点型。
在关键词 ``ufixedMxN`` 和 ``fixedMxN`` 中，``M`` 表示该类型占用的位数，``N`` 表示可用的十进制长度。
``M`` 必须能整除 8，表示范围从 8 到 256 的位数。
``N`` 则可以是从 0 到 80 之间的任意数。
``ufixed`` 和 ``fixed`` 分别代表 ``ufixed128x19`` 和 ``fixed128x19``。

运算符：

* 比较运算符：``<=``， ``<``， ``==``， ``!=``， ``>=``， ``>`` （返回值是布尔型）
* 算术运算符：``+``， ``-``， 一元运算 ``-``， 一元运算 ``+``， ``*``， ``/``， ``%`` （取余数）

.. note::
    浮点型（在许多语言中的 ``float`` 和 ``double`` 类型，更准确地说是 IEEE 754 类型）和定长浮点型之间最大的不同点是，
    在前者中整数部分和分数部分（小数点后的部分）需要的位数是灵活可变的，而后者中这两部分的长度受到严格的规定。
    一般来说，在浮点型中，几乎整个空间都用来表示数字，但只有少数的位来表示小数点的位置。

.. index:: address, balance, send, call, callcode, delegatecall, transfer

.. _address:

地址型
-----

``address``：地址型存储一个 20 字节的值（以太坊地址的大小）。
地址型也有成员变量，并作为所有合约的基础。

运算符：

* ``<=``， ``<``， ``==``， ``!=``， ``>=`` 和 ``>``

.. note::
    从 0.5.0 版本开始，合约不会从地址型派生，但仍然可以显式地转换成地址型。

.. _members-of-addresses:

地址类型成员变量
^^^^^^^^^^^^^

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

``send`` 是 ``transfer`` 的低级版本。如果执行失败，当前的合约在终止时不会抛出异常，但 ``send`` 会返回 ``false``。

.. warning::
    在使用 ``send`` 的时候会有些风险：如果调用栈深度是 1024 会导致发送失败（这总是可以被调用者强制），如果接收者用光了 gas 也会导致发送失败。
    所以为了保证 |ether| 发送的安全，一定要检查 ``send`` 的返回值，使用 ``transfer`` 或者更好的办法：
    使用一种接收者可以取回资金的模式。

* ``call``， ``callcode`` 和 ``delegatecall``

此外，为了与不符合 |ABI| 的合约交互，于是就有了可以接受任意类型任意数量参数的 ``call`` 函数。
这些参数连接在一起填充在 32 字节的空间里。
其中一个例外是当第一个参数被编码成正好 4 个字节的情况。
在这种情况下，它被填充后不能使用函数签名。

::

    address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
    nameReg.call("register", "MyName");
    nameReg.call(bytes4(keccak256("fun(uint256)")), a);

``call`` 返回的布尔值表明了被调用的函数已经执行完毕（``true``）或者引发了一个 EVM 异常（``false``）。
无法访问返回的真实数据（为此我们需要事先知道编码和大小）。

可以使用 ``.gas()`` 修改器调整提供的 gas 数量 ::

    namReg.call.gas(1000000)("register", "MyName");

类似地，也能控制提供的 |ether| 的值 ::

   nameReg.call.value(1 ether)("register", "MyName"); 

最后一点，这些修改器可以联合使用。每个修改器出现的顺序不重要 ::

   nameReg.call.gas(1000000).value(1 ether)("register", "MyName"); 

.. note::
    It is not yet possible to use the gas or value modifiers on overloaded functions.

    A workaround is to introduce a special case for gas and value and just re-check
    whether they are present at the point of overload resolution.

.. note::
    目前还不能在重载函数中使用 gas 或者值修改器。

    一种解决方案是给 gas 和值引入一个特例，并重新检查它们是否在重载的地方出现。

类似地，也可以使用 ``delegatecall``：
区别在于只使用给定地址的代码，其它属性（存储，余额，……）都取自当前合约。
``delegatecall`` 的目的是使用存储在另外一个合约中的库代码。
用户必须确保两个合约中存储的分布适合使用 delegatecall。
在 homestead 版本之前，只有一个功能类似但有限的叫作 ``callcode`` 的函数可用，但使用它并不能访问 ``msg.sender`` 和 ``msg.value`` 的原始值。

这三个函数 ``call``， ``delegatecall`` 和 ``callcode`` 都是非常低级的函数，应该只把它们当作 *最后一招* 来使用，因为它们破坏了 Solitity 的类型安全性。

尽管 ``.value()`` 选项不支持 ``delegatecall``，但三种方法都有 ``.gas()`` 选项。

.. note::
    所有合约都集成地址类型的所有成员变量，因此可以使用 ``this.balance`` 查询当前合约的余额。

.. note::
    不鼓励使用 ``callcode``，在未来也会将其移除。

.. warning::
    这三个函数都属于低级函数，需要谨慎使用。
    具体来说，任何未知的合约都可能是恶意的。
    你在调用一个合约的同时就将控制权交给了它，它可以反过来调用你的合约，
    因此，当调用返回时要为你的状态变量的改变做好准备。

.. index:: byte array, bytes32

定长字节数组
----------

关键词有：``bytes1``， ``bytes2``， ``bytes3``， ...， ``bytes32``。``byte`` 代表 ``bytes1``。

运算符：

* Index access: If ``x`` is of type ``bytesI``, then ``x[k]`` for ``0 <= k < I`` returns the ``k`` th byte (read-only).

* 比较运算符：``<=``， ``<``， ``==``， ``!=``， ``>=``， ``>`` （返回布尔型）
* 位运算符： ``&``， ``|``， ``^`` （按位异或）， ``~`` （按位取反）， ``<<`` （左移位）， ``>>`` （右移位）
* 索引访问：如果 ``x`` 是 ``bytesI`` 类型，那么 ``x[k]``（其中 ``0 <= k < I``）返回第 ``k`` 个字节（只读）。

该类型可以和作为右操作数的任何整数类型进行移位运算（但返回结果的类型和左操作数类型相同），右操作数表示需要移动的位数。
进行负数位移运算会引发运行时异常。

成员变量：

* ``.length`` 表示这个字节数组的长度（只读）.

.. note::
    可以将 ``byte[]`` 当作字节数组使用，但这种方式非常浪费存储空间，准确来说，是在传入调用时，每个元素占 31 字节。
    更好地做法是使用 ``bytes``。

变长字节数组
----------

``bytes``:
    变长字节数组，参见 :ref:`arrays`。它并不是值类型。
``string``:
    变长 UTF-8 编码字符串类型，参见 :ref:`arrays`。并不是值类型。

根据经验，最好使用 ``bytes`` 存储任意长度的原始字节数据，使用 ``string`` 存储任意长度的字符串（UTF-8编码）数据。
如果长度可以确定，尽量使用 ``bytes1`` 到 ``bytes32`` 中的一个，因为它们的开销小得多。

.. index:: address, literal;address

.. _address_literals:

地址常量（Address Literals）
-------------------------

.. note::
    ``literal``也被译作“字面量”。下文统一译作“常量”

比如像 ``0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF`` 这样的通过了地址校验和测试的十六进制常量属于 ``address`` 类型。
长度在 39 到 41 个数字的，没有通过校验和测试而产生了一个警告的十六进制常量视为正常的有理数常量。

.. note::
    混合大小写的地址校验和格式定义在 `EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_ 中。


.. index:: literal, literal;rational

.. _rational_literals:

有理数和整数常量
-------------

整数常量由范围在 0-9 的一串数字组成，表现成十进制。
例如，`69` 表示数字 69。
Solidity 中是没有八进制的，因此前置 0 是无效的。

十进制小数常量带有一个 ``.``，至少在其一边会有一个数字。
有效的事例如：``1.``，``.1``，和 ``1.3``。

科学符号也是支持的，尽管指数必须是整数，但基数可以是小数。
有效的事例如：``2e10``， ``-2e10``， ``2e-10``， ``2.5e1``。

数值常量表达式本身支持任意精度，除非它们被转换成了非常量类型（例如，当它们出现在非常量表达式中时就会发生转换）。
这意味着在数值常量表达式中计算时不会出现溢出或除法截断。

例如，``(2**800 + 1) - 2**800`` 的结果是常量 ``1``（属于 ``uint8`` 类型），尽管计算的中间结果已经超过了计算机字长。
此外， ``.5 * 8`` 的结果是整型 ``4``（尽管有非整型参与了计算）。

只要操作数是整型，任意整型支持的运算符都可以被运用在数值常量表达式中。
如果两个中的任一个数是小数，则不允许进行位运算。如果指数是小数的话，也不支持幂运算（因为这样可能会得到一个无理数）。

.. note::
    Solidity 对每个有理数都有对应的数值常量类型。
    整数常量和有理数常量都属于数值常量类型。
    除此之外，所有的数值常量表达式（即只包含数值常量和运算符的表达式）都属于数值常量类型。
    因此数值常量表达式 ``1 + 2`` 和 ``2 + 1`` 的结果跟有理数三的数值常量类型相同。

.. warning::
    在早期版本中，整数常量的除法也会截断，但在现在的版本中，会将结果转换成一个有理数。例如 ``5 / 2`` 并不等于 ``2``，而是等于 ``2.5``。

.. note::
    数值常量表达式只要在非常量表达式中使用就会转换成非常量类型。
    在下面的例子中，尽管我们知道 ``b`` 的值是一个整数，但 ``2.5 + a`` 这部分表达式并不进行类型检查，因此编译不能通过。

::

    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5;

.. index:: literal, literal;string, string

字符串常量
--------

字符串常量是指由双引号或单引号引起来的字符串（``"foo"`` 或者 ``'bar'``）。
不像在 C 语言中那样带有结束符；``"foo"`` 表示 3 个字节而不是 4 个。
和整数常量一样，字符串常量的类型也可以发生改变，但它们可以隐式地转换成 ``bytes1``，……，``bytes32``，如果合适的话，还可以转换成 ``bytes`` 以及 ``string``。

字符串常量支持转义字符，例如 ``\n``，``\xNN`` 和 ``\uNNNN``。``\xNN`` 表示一个 16 进制值，最终转换成合适的字节，
而 ``\uNNNN`` 表示 Unicode 编码值，最终会转换为 UTF-8 的序列。

.. index:: literal, bytes

十六进制常量
----------

十六进制常量以关键字 ``hex`` 打头，后面紧跟着用单引号或双引号引起来的字符串（例如，``hex"001122FF"``）。
字符串的内容必须是一个十六进制的字符串，它们的值将使用二进制表示。

十六进制常量跟字符串常量很类似，具有相同的转换规则。

.. index:: enum

.. _enums:

枚举类型
-------

在 Solidity 中，枚举类型可以用来创建自定义类型。
它可以显式地与整数类型进行转换，但不能隐式转换。
显式的转换会在运行时检查数值的范围，如果转换失败则会引发一个异常。
枚举类型至少需要一名成员。下面是枚举类型的一个例子：

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
-------

函数类型是一种表示函数的类型。可以将一个函数赋值给另一个函数类型的变量，也可以将一个函数作为参数进行传递，还能在函数调用中返回函数类型变量。
函数类型有两类：- *内部（internal）* 函数和 *外部（external）* 函数：

内部函数只能在当前合约内被调用（更具体来说，在当前代码块内，包括内部库函数和继承的函数中），因为它们不能在当前合约上下文的外部被执行。
调用一个内部函数是通过跳转到它的入口标签来实现的，就像在当前合约的内部调用一个函数。

外部函数由一个地址和一个函数签名组成，可以通过外部函数调用传递或者返回。

函数类型表示成如下的形式 ::

    function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]

与参数类型相比，返回类型不能为空 —— 如果函数类型不需要返回，则需要删除整个 ``returns (<return types>)`` 部分。

函数类型默认是内部函数，因此不需要声明 ``internal`` 关键词。
与此相反的是，合约中的函数本身默认是 public 的，只有当它被当做类型名称时，默认才是内部函数。

有两种方法可以访问当前合约中的函数：其中一种方法是直接使用它的名字，``f``，另一种方法是使用 ``this.f``。
前者适用于内部函数，后者适用于外部函数。

如果当函数类型的变量还没有初始化时就调用它的话会引发一个异常。
如果在一个函数被 ``delete`` 之后调用它也会发生相同的情况。

如果外部函数类型在 Solidity 的上下文环境以外的地方使用，它们会被视为 ``function`` 类型。
该类型将函数所在地址及其函数标识一起编码为一个 ``bytes24`` 类型。

请注意，当前合约的 public 函数既可以被当作内部函数也可以被当作外部函数使用。
如果想将一个函数当作内部函数使用，就用 ``f``调用，如果想将其当作外部函数，使用 ``this.f``。

除此之外，public（或外部）函数也有一个特殊的成员变量称作 ``selector``，可以返回 :ref:`ABI 函数选择器 <abi_function_selector>`::

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
      Oracle constant oracle = Oracle(0x1234567); // known contract 已知的合约
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

Reference Types 引用类型
==================

Complex types, i.e. types which do not always fit into 256 bits have to be handled
more carefully than the value-types we have already seen. Since copying
them can be quite expensive, we have to think about whether we want them to be
stored in **memory** (which is not persisting) or **storage** (where the state
variables are held).

在处理复杂的类型（例如这些类型占用的空间超过 256 位）时，需要比处理之前我们讨论过的值类型更加谨慎。
由于拷贝这些类型变量的开销相当大，我们不得不考虑它的存储位置，是将它们保存在 **memory** （并不是永久存储）中，
还是 **storage** （保存状态变量的地方）中。

Data location 数据位置
-------------

Every complex type, i.e. *arrays* and *structs*, has an additional
annotation, the "data location", about whether it is stored in memory or in storage. Depending on the
context, there is always a default, but it can be overridden by appending
either ``storage`` or ``memory`` to the type. The default for function parameters (including return parameters) is ``memory``, the default for local variables is ``storage`` and the location is forced
to ``storage`` for state variables (obviously).

所有的复杂类型，如 *数组* 和 *结构* 类型，都有一个额外属性，“数据位置”，说明数据是保存在 memory 中还是 storage 中。
根据上下文不同，大多数时候数据有默认的位置，但也可以通过在类型名后增加关键字 ``storage`` 或 ``memory`` 进行修改。
函数参数（包括返回的参数）的数据位置默认是 ``memory``，
局部变量的数据位置默认是 ``storage``，状态变量的数据位置强制是 ``storage`` （这是显而易见的）。

There is also a third data location, ``calldata``, which is a non-modifiable,
non-persistent area where function arguments are stored. Function parameters
(not return parameters) of external functions are forced to ``calldata`` and
behave mostly like ``memory``.

也存在第三种数据位置， ``calldata`` ，这是一块只读的，且不会永久存储的位置，用来存储函数参数。
外部函数的参数（非返回参数）的数据位置被强制指定为 ``calldata`` ，效果跟 ``memory`` 差不多。

Data locations are important because they change how assignments behave:
assignments between storage and memory and also to a state variable (even from other state variables)
always create an independent copy.
Assignments to local storage variables only assign a reference though, and
this reference always points to the state variable even if the latter is changed
in the meantime.
On the other hand, assignments from a memory stored reference type to another
memory-stored reference type do not create a copy.

数据位置的指定非常重要，因为它们影响着赋值行为：
在 storage 和 memory 之间两两赋值，或者 storage 向状态变量（甚至是从其它状态变量）赋值都会创建一份独立的拷贝。
然而状态变量向局部变量赋值时仅仅传递一个引用，而且这个引用总是指向状态变量，因此后者改变的同时前者也会发生改变。
另一方面，从一个 memory 存储的引用类型向另一个 memory 存储的引用类型赋值并不会创建拷贝。

::

    pragma solidity ^0.4.0;

    contract C {
        uint[] x; // x 的数据存储位置是 storage

        // the data location of memoryArray is memory
        // memoryArray 的数据存储位置是 memory
        function f(uint[] memoryArray) public {
            x = memoryArray; // works, copies the whole array to storage // 将整个数组拷贝到 storage 中，可行
            var y = x; // works, assigns a pointer, data location of y is storage // 分配一个指针（其中 y 的数据存储位置是 storage），可行
            y[7]; // fine, returns the 8th element // 返回第 8 个元素，可行
            y.length = 2; // fine, modifies x through y // 通过 y 修改 x，可行
            delete x; // fine, clears the array, also modifies y // 清除数组，同时修改 y，可行
            // The following does not work; it would need to create a new temporary /
            // unnamed array in storage, but storage is "statically" allocated:
            // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组， /
            // 但 storage 是“静态”分配的：
            // y = memoryArray;
            // This does not work either, since it would "reset" the pointer, but there
            // is no sensible location it could point to.
            // 下面这一行也不可行，因为这会“重置”指针，
            // 但并没有可以让它指向的合适的存储位置。
            // delete y;
            
            g(x); // calls g, handing over a reference to x 调用 g 函数，同时移交对 x 的引用
            h(x); // calls h and creates an independent, temporary copy in memory 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
        }

        function g(uint[] storage storageArray) internal {}
        function h(uint[] memoryArray) public {}
    }

Summary 总结
^^^^^^^

Forced data location:
 - parameters (not return) of external functions: calldata
 - state variables: storage

强制数据位置：
 - 外部函数的参数（不包括返回参数）： calldata
 - 状态变量： storage

Default data location:
 - parameters (also return) of functions: memory
 - all other local variables: storage

默认数据位置：
 - 函数参数（包括返回参数）： memory
 - 所有其它局部变量： storage

.. index:: ! array

.. _arrays:

Arrays 数组
------

Arrays can have a compile-time fixed size or they can be dynamic.
For storage arrays, the element type can be arbitrary (i.e. also other
arrays, mappings or structs). For memory arrays, it cannot be a mapping and
has to be an ABI type if it is an argument of a publicly-visible function.

数组可以在声明时指定长度，也可以动态调整大小。
对于 storage 的数组来说，元素类型可以是任意的（例如，元素也可以是数组类型，映射类型或者结构体）。
对于 memory 的数组来说，元素类型不能使映射类型，如果作为 public 函数的参数，它只能是 ABI 类型。

An array of fixed size ``k`` and element type ``T`` is written as ``T[k]``,
an array of dynamic size as ``T[]``. As an example, an array of 5 dynamic
arrays of ``uint`` is ``uint[][5]`` (note that the notation is reversed when
compared to some other languages). To access the second uint in the
third dynamic array, you use ``x[2][1]`` (indices are zero-based and
access works in the opposite way of the declaration, i.e. ``x[2]``
shaves off one level in the type from the right).

一个元素类型为 ``T``，固定长度为 ``k`` 的数组可以声明为 ``T[k]``，而动态数组声明为 ``T[]``。
举个例子，一个包含有 5 个元素类型为 ``uint`` 的动态数组的数组声明为 ``uint[][5]``（注意这里跟其它语言比，数组长度的声明位置是反的）。
使用 ``x[2][]`` 访问第三个动态数组的第二个元素（数组的序号从 0 开始，序号顺序与声明时的相反，例如，``x[2]``就是从右边削去了类型中的一级）。

Variables of type ``bytes`` and ``string`` are special arrays. A ``bytes`` is similar to ``byte[]``,
but it is packed tightly in calldata. ``string`` is equal to ``bytes`` but does not allow
length or index access (for now).

``bytes`` 和 ``string`` 类型的变量是特殊的数组。
``bytes`` 类似于 ``byte[]``，但它在 calldata 中会被压缩打包。
``string`` 与 ``bytes`` 在这点一样，但目前并不提供长度或序号的索引方式。

So ``bytes`` should always be preferred over ``byte[]`` because it is cheaper.

因此出于开销考虑，使用 ``bytes`` 比 ``byte[]`` 更好。

.. note::
    If you want to access the byte-representation of a string ``s``, use
    ``bytes(s).length`` / ``bytes(s)[7] = 'x';``. Keep in mind
    that you are accessing the low-level bytes of the UTF-8 representation,
    and not the individual characters!
    如果想要访问以字节型表示的字符串 ``s``，请使用 ``bytes(s).length`` / ``bytes(s)[7] = 'x';``。
    注意这时你访问的是 UTF-8 形式的低级 bytes 类型，而不是单个的字符。

It is possible to mark arrays ``public`` and have Solidity create a :ref:`getter <visibility-and-getters>`.
The numeric index will become a required parameter for the getter.

可以将数组标识为 ``public``，从而让 Solidity 创建一个 :ref:`getter <visibility-and-getters>`。
之后必须使用数字下标来访问 getter。

.. index:: ! array;allocating, new

Allocating Memory Arrays 创建内存数组
^^^^^^^^^^^^^^^^^^^^^^^^

Creating arrays with variable length in memory can be done using the ``new`` keyword.
As opposed to storage arrays, it is **not** possible to resize memory arrays by assigning to
the ``.length`` member.

可使用 ``new`` 关键词在内存中创建具有可变长度的数组。
与 storage 数组不同的是，你*不能*通过修改成员变量 ``.length`` 改变 memory 数组的大小。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);
            // Here we have a.length == 7 and b.length == len
            // 这里我们有 a.length == 7 以及 b.length == len
            a[6] = 8;
        }
    }

.. index:: ! array;literals, !inline;arrays

Array Literals / Inline Arrays 数组常量 / 内联数组
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Array literals are arrays that are written as an expression and are not
assigned to a variable right away.

数组常量是写作表达式形式的数组，并且不会立即赋值给变量。

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

The type of an array literal is a memory array of fixed size whose base
type is the common type of the given elements. The type of ``[1, 2, 3]`` is
``uint8[3] memory``, because the type of each of these constants is ``uint8``.
Because of that, it was necessary to convert the first element in the example
above to ``uint``. Note that currently, fixed size memory arrays cannot
be assigned to dynamically-sized memory arrays, i.e. the following is not
possible:

数组常量是一种定长的 memory 数组类型，它的基础类型由其中元素共同的类型决定。
例如，``[1, 2, 3]`` 的类型是 ``uint8[3] memory``，因为其中的每个常量的类型都是 ``uint8``。
正因为如此，有必要将上面这个例子中的第一个元素转换成 ``uint`` 类型。
目前需要注意的是，定长的 memory 数组并不能赋值给变长的 memory 数组，下面是个反例：

::

    // This will not compile.

    pragma solidity ^0.4.0;

    contract C {
        function f() public {
            // The next line creates a type error because uint[3] memory
            // cannot be converted to uint[] memory.
            // 这一行引发了一个类型错误，因为 unint[3] memory
            // 不能转换成 uint[] memory。
            uint[] x = [uint(1), 3, 4];
        }
    }

It is planned to remove this restriction in the future but currently creates
some complications because of how arrays are passed in the ABI.

已经计划在未来一处这样的限制，但目前数组在 ABI 中传递的问题造成了一些麻烦。

.. index:: ! array;length, length, push, !array;push

Members 成员
^^^^^^^

**length**:
    Arrays have a ``length`` member to hold their number of elements.
    Dynamic arrays can be resized in storage (not in memory) by changing the
    ``.length`` member. This does not happen automatically when attempting to access elements outside the current length.
    The size of memory arrays is fixed (but dynamic, i.e. it can depend on runtime parameters) once they are created.
    数组有 ``length`` 成员变量表示当前数组的长度。
    动态数组可以在 storage （而不是 memory）中通过改变成员变量 ``.length`` 改变数组大小。
    并不能通过访问超出当前数组长度的方式实现自动扩展数组的长度。
    memory 数组的大小一旦创建就是是固定的（但也是动态的，例如，可以通过运行时的参数改变数组大小）。

**push**:
    Dynamic storage arrays and ``bytes`` (not ``string``) have a member function called ``push`` that can be used to append an element at the end of the array.
    The function returns the new length.
    变长的 storage 数组以及 ``bytes`` 类型（而不是 ``string`` 类型）都有一个叫做 ``push`` 的成员函数，它用来附加新的元素到数组末尾。

.. warning::
    It is not yet possible to use arrays of arrays in external functions.
    在外部函数中目前还不能使用多维数组。

.. warning::
    Due to limitations of the EVM, it is not possible to return
    dynamic content from external function calls. The function ``f`` in
    ``contract C { function f() returns (uint[]) { ... } }`` will return
    something if called from web3.js, but not if called from Solidity.
    由于 |evm| 的限制，不能通过外部函数调用返回动态的内容。
    例如，如果通过 web3.js 调用 ``contract C { function f() returns (uint[]) { ... } }`` 中的 ``f`` 函数，它会返回一些内容，但通过 Solidity 不可以。

    The only workaround for now is to use large statically-sized arrays.
    目前唯一的变通方法是使用大型的静态数组。

::

    pragma solidity ^0.4.16;

    contract ArrayContract {
        uint[2**20] m_aLotOfIntegers;
        // Note that the following is not a pair of dynamic arrays but a
        // dynamic array of pairs (i.e. of fixed size arrays of length two).
        // 注意下面的代码并不是一对动态数组，
        // 而是一个元素为一对变量的数组（例如，一个元素为长度为二的定长数组的变长数组）。
        bool[2][] m_pairsOfFlags;
        // newPairs is stored in memory - the default for function arguments
        // newPairs 存储在 memory 中 —— 函数参数默认的存储位置

        function setAllFlagPairs(bool[2][] newPairs) public {
            // assignment to a storage array replaces the complete array
            // 向一个 storage 的数组赋值会替代整个数组
            m_pairsOfFlags = newPairs;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // access to a non-existing index will throw an exception
            // 访问一个不存在的数组下标会引发一个异常
            m_pairsOfFlags[index][0] = flagA;
            m_pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // if the new size is smaller, removed array elements will be cleared
            // 如果 newSize 更小，那么超出的元素会被清除
            m_pairsOfFlags.length = newSize;
        }

        function clear() public {
            // these clear the arrays completely 
            // 这些代码会将数组全部清空
            delete m_pairsOfFlags;
            delete m_aLotOfIntegers;
            // identical effect here
            // 这里也是实现同样地功能
            m_pairsOfFlags.length = 0;
        }

        bytes m_byteData;

        function byteArrays(bytes data) public {
            // byte arrays ("bytes") are different as they are stored without padding,
            // but can be treated identical to "uint8[]"
            // byte arrays ("bytes") 不一样，因为它们不是填充式存储的，
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
            // Dynamic memory arrays are created using `new`:
            // 使用 `new` 创建动态 memory 数组：
            uint[2][] memory arrayOfPairs = new uint[2][](size);
            // Create a dynamic byte array:
            // 创建一个动态字节数组：
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = byte(i);
            return b;
        }
    }


.. index:: ! struct, ! type;struct

.. _structs:

Structs 结构体
-------

Solidity provides a way to define new types in the form of structs, which is
shown in the following example:

Solidity 支持通过构造结构体的形式定义新的类型，以下是一个结构体使用的示例：

::

    pragma solidity ^0.4.11;

    contract CrowdFunding {
        // Defines a new type with two fields.
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
            // Creates new struct and saves in storage. We leave out the mapping type.
            // 创建新的结构体示例，存储在 storage 中。我们先不关注映射类型。
            campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // Creates a new temporary memory struct, initialised with the given values
            // and copies it over to storage.
            // Note that you can also use Funder(msg.sender, msg.value) to initialise.
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

The contract does not provide the full functionality of a crowdfunding
contract, but it contains the basic concepts necessary to understand structs.
Struct types can be used inside mappings and arrays and they can itself
contain mappings and arrays.

上面的合约只是一个简化版的众筹合约，但它已经足以让我们理解结构体的基础概念。
结构体类型可以作为元素用在映射和数组中，其自身也可以包含映射和数组作为成员变量。

It is not possible for a struct to contain a member of its own type,
although the struct itself can be the value type of a mapping member.
This restriction is necessary, as the size of the struct has to be finite.

尽管结构体本身可以作为映射的值类型成员，但它并不能包含自身。
这个限制是有必要的，因为结构体的大小必须是有限的。

Note how in all the functions, a struct type is assigned to a local variable
(of the default storage data location).
This does not copy the struct but only stores a reference so that assignments to
members of the local variable actually write to the state.

注意在函数中使用结构体时，一个结构体是如何赋值给一个局部变量（默认存储位置是 storage ）的。
在这个过程中并没有拷贝这个结构体，而是保存一个引用，因此对局部变量的赋值的同时实际上改变了原变量。

Of course, you can also directly access the members of the struct without
assigning it to a local variable, as in
``campaigns[campaignID].amount = 0``.

当然，你也可以直接访问结构体的属性而不用将其赋值给一个局部变量，例如，
``campaigns[campaignID].amount = 0``。

.. index:: !mapping

Mappings 映射
========

Mapping types are declared as ``mapping(_KeyType => _ValueType)``.
Here ``_KeyType`` can be almost any type except for a mapping, a dynamically sized array, a contract, an enum and a struct.
``_ValueType`` can actually be any type, including mappings.

映射类型在声明时的形式为 ``mapping(_KeyType => _ValueType)``。
其中 ``_KeyType`` 可以是可以是除了映射、变长数组、合约、枚举以及结构体以外的几乎所有类型。
``_ValueType`` 可以是包括映射类型在内的任何类型。

Mappings can be seen as `hash tables <https://en.wikipedia.org/wiki/Hash_table>`_ which are virtually initialized such that
every possible key exists and is mapped to a value whose byte-representation is
all zeros: a type's :ref:`default value <default-value>`. The similarity ends here, though: The key data is not actually stored
in a mapping, only its ``keccak256`` hash used to look up the value.

映射可以视作 `哈希表 <https://en.wikipedia.org/wiki/Hash_table>`，它们在实际的初始化过程中创建每个可能的 key，
并将其映射到字节形式全是零的值：一个类型的 :ref:`默认值 <default-value>`。然而下面是映射与哈希表不同的地方：
在映射中，实际上并不存储 key，而是存储它的 ``keccak256`` 哈希值，从而便于查询实际的值。

Because of this, mappings do not have a length or a concept of a key or value being "set".

正因为如此，映射是没有长度的，也没有 key 或 value 的概念。

Mappings are only allowed for state variables (or as storage reference types
in internal functions).

映射只用来表示状态变量（或者在内部函数中作为存储位置为 storage 的引用类型）。

It is possible to mark mappings ``public`` and have Solidity create a :ref:`getter <visibility-and-getters>`.
The ``_KeyType`` will become a required parameter for the getter and it will
return ``_ValueType``.

可以将映射声明为 ``public``，然后使用 Solidity 创建一个 :ref:`getter <visibility-and-getters>`。
``_KeyType`` 是 getter 的参数，返回 ``_ValueType``。

The ``_ValueType`` can be a mapping too. The getter will have one parameter
for each ``_KeyType``, recursively.

``_ValueType`` 也可以是一个映射。这时在使用 getter 时将递归地传入每个 ``_KeyType`` 参数。

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
  Mappings are not iterable, but it is possible to implement a data structure on top of them.
  For an example, see `iterable mapping <https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol>`_.
  递归不支持迭代，但可以在此之上实现一个这样的数据结构。
  例子可以参考 `可迭代的映射 <https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol>`_。

.. index:: assignment, ! delete, lvalue

Operators Involving LValues 涉及 LValues 的运算符
===========================

If ``a`` is an LValue (i.e. a variable or something that can be assigned to), the following operators are available as shorthands:

如果 ``a`` 是一个 LValue（例如，一个可以被赋值的变量之类的），一下运算符都可以使用简写：

``a += e`` is equivalent to ``a = a + e``. The operators ``-=``, ``*=``, ``/=``, ``%=``, ``|=``, ``&=`` and ``^=`` are defined accordingly.
``a++`` and ``a--`` are equivalent to ``a += 1`` / ``a -= 1`` but the expression itself still has the previous value of ``a``.
In contrast, ``--a`` and ``++a`` have the same effect on ``a`` but return the value after the change.

``a += e`` 等同于 ``a = a + e``。 其它运算符 ``-=``， ``*=``， ``/=``， ``%=``， ``|=``， ``&=`` 以及 ``^=`` 都是如此定义的。
``a++`` 和 ``a--`` 分别等同于 ``a += 1`` 和 ``a -= 1``，但表达式本身的值等于 ``a`` 在计算之前的值。
与之相反，``--a`` 和 ``++a`` 虽然最终 ``a`` 的结果与之前的表达式相同，但表达式的返回值是计算之后的值。

delete 删除
------

``delete a`` assigns the initial value for the type to ``a``. I.e. for integers it is equivalent to ``a = 0``,
but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset.
For structs, it assigns a struct with all members reset.

``delete a`` 的结果是将 ``a`` 的类型在初始化时的值赋值给 ``a``。例如，对于整型变量来说，相当于 ``a = 0``，
但 delete 也适用于数组，对于动态数组来说，是将数组的长度设为 0，而对于静态数组来说，是将数组中的所有元素重置。
如果对象是结构体，则将结构体中的所有属性重置。

``delete`` has no effect on whole mappings (as the keys of mappings may be arbitrary and are generally unknown).
So if you delete a struct, it will reset all members that are not mappings and also recurse into the members unless they are mappings.
However, individual keys and what they map to can be deleted.

``delete`` 对整个映射是无效的（因为映射的键可以是任意的，通常也是未知的）。
因此在你删除一个结构体时，结果将重置所有的非映射属性，这个过程是递归进行的，除非它们是映射。
然而，单个的键及其映射的值是可以被删除的。

It is important to note that ``delete a`` really behaves like an assignment to ``a``, i.e. it stores a new object in ``a``.
理解 ``delete a`` 的效果就像是给 ``a``赋值很重要，例如，这相当于在 ``a`` 中存储了一个新的对象。

::

    pragma solidity ^0.4.0;

    contract DeleteExample {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // sets x to 0, does not affect data
            delete x; // 将 x 设为 0，并不影响数据
            delete data; // sets data to 0, does not affect x which still holds a copy
            delete data; // 将 data 设为 0，并不影响 x，因为它仍然有个副本
            uint[] storage y = dataArray;
            delete dataArray; // this sets dataArray.length to zero, but as uint[] is a complex object, also
            // y is affected which is an alias to the storage object
            // On the other hand: "delete y" is not valid, as assignments to local variables
            // referencing storage objects can only be made from existing storage objects.
            // 将 dataArray.length 设为 0，但由于 uint[] 是一个复杂的对象，y 也将受到影响，
            // 因为它是一个存储位置是 storage 的对象的别名
            // 另一方面："delete y" 是非法的，引用了 storage 对象的局部变量只能由已有的 storage 对象赋值。
        }
    }

.. index:: ! type;conversion, ! cast

Conversions between Elementary Type
基本类型之间的转换
====================================

Implicit Conversions
隐式转换
--------------------

If an operator is applied to different types, the compiler tries to
implicitly convert one of the operands to the type of the other (the same is
true for assignments). In general, an implicit conversion between value-types
is possible if it
makes sense semantically and no information is lost: ``uint8`` is convertible to
``uint16`` and ``int128`` to ``int256``, but ``int8`` is not convertible to ``uint256``
(because ``uint256`` cannot hold e.g. ``-1``).
Furthermore, unsigned integers can be converted to bytes of the same or larger
size, but not vice-versa. Any type that can be converted to ``uint160`` can also
be converted to ``address``.

如果一个运算符用在两个不同类型的变量之间，那么编译器将隐式地将其中一个类型转换为另一个类型（不同类型之间的赋值也是一样）。
总是，只要值类型之间的转换在语义上行得通，而且转换的过程中没有信息丢失，那么隐式转换基本都是可以实现的：
``uint8`` 可以转换成 ``uint16``，``int128`` 转换成 ``int256``，但 ``int8`` 不能转换成 ``uint256``
（因为 ``uint256`` 不能涵盖某些值，例如，``-1``）。
更进一步来说，无符号整型可以转换成跟它大小相等或更大的字节类型，但反之不能。
任何可以转换成 ``uint160`` 的类型都可以转换成 ``address`` 类型。

Explicit Conversions
显式转换
--------------------

If the compiler does not allow implicit conversion but you know what you are
doing, an explicit type conversion is sometimes possible. Note that this may
give you some unexpected behaviour so be sure to test to ensure that the
result is what you want! Take the following example where you are converting
a negative ``int8`` to a ``uint``:

如果某些情况下编译器不支持隐式转换，但是你很清楚你要做什么，这种情况可以考虑显式转换。
注意这可能会发生一些无法预料的后果，因此一定要进行测试，确保结果是你想要的！
下面的示例是将一个 ``int8`` 类型的负数转换成 ``uint``：

::

    int8 y = -3;
    uint x = uint(y);

At the end of this code snippet, ``x`` will have the value ``0xfffff..fd`` (64 hex
characters), which is -3 in the two's complement representation of 256 bits.

这段代码的最后，``x`` 的值将是 ``0xfffff..fd``（64 个 16 禁止字符），因为这是 -3 的 256 位补码形式。

If a type is explicitly converted to a smaller type, higher-order bits are
cut off::

如果一个类型显式转换成与其相似的类型，排序更高的位将被舍弃 ::

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // b will be 0x5678 now 此时 b 的值是 0x5678

.. index:: ! type;deduction, ! var

.. _type-deduction:

Type Deduction
类型推断
==============

For convenience, it is not always necessary to explicitly specify the type of a
variable, the compiler automatically infers it from the type of the first
expression that is assigned to the variable::

为了方便起见，没有必要每次都精确指定一个变量的类型，编译器会根据分配该变量的第一个表达式的类型自动推断该变量的类型 ::

    uint24 x = 0x123;
    var y = x;

Here, the type of ``y`` will be ``uint24``. Using ``var`` is not possible for function
parameters or return parameters.

这里 ``y`` 的类型将是 ``uint24``。不能对函数参数或者返回参数使用 ``var``。

.. warning::
    The type is only deduced from the first assignment, so
    the loop in the following snippet is infinite, as ``i`` will have the type
    ``uint8`` and the highest value of this type is smaller than ``2000``.
    ``for (var i = 0; i < 2000; i++) { ... }``
    类型只能从第一次赋值中推断出来，因此以下代码中的循环是无限的，
    原因是``i`` 的类型是 ``uint8``，而这个类型变量的最大值比 ``2000`` 小。
    ``for (var i = 0; i < 2000; i++) { ... }``