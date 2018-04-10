.. include :: glossaries.rst
.. index:: type

.. _types:

*****
类型
*****

Solidity is a statically typed language, which means that the type of each
variable (state and local) needs to be specified (or at least known -
see :ref:`type-deduction` below) at
compile-time. Solidity provides several elementary types which can be combined
to form complex types.

Solidity 是一种静态类型语言，这意味着每个变量（状态变量和局部变量）都需要在编译时指定变量的类型（或至少可以推导出——参考下面的 :ref:`type-deduction` ）。
Solidity 提供了几种基本类型，可以用来组合出复杂类型。

In addition, types can interact with each other in expressions containing
operators. For a quick reference of the various operators, see :ref:`order`.

除此之外，类型之间可以在包含运算符号的表达式中进行交互。
关于各种运算符号，可以参考 :ref:`order` 。

.. index:: ! value type, ! type;value

Value Types 值类型
===========

The following types are also called value types because variables of these
types will always be passed by value, i.e. they are always copied when they
are used as function arguments or in assignments.

一下类型也称为值类型，因为这些类型的变量将始终按值来传递。
也就是说，当这些变量被用作函数参数或者被赋值时，总是进行值拷贝。

.. index:: ! bool, ! true, ! false

Booleans 布尔类型
--------

``bool``: The possible values are constants ``true`` and ``false``.

``bool`` ：可能的取值为常量值 ``true`` 和 ``false`` 。

Operators:

运算符：

*  ``!`` (logical negation)
*  ``&&`` (logical conjunction, "and")
*  ``||`` (logical disjunction, "or")
*  ``==`` (equality)
*  ``!=`` (inequality)

*  ``!`` （逻辑非）
*  ``&&`` （逻辑与， "and" ）
*  ``||`` （逻辑或， "or " ）
*  ``==`` （等于）
*  ``!=`` （不等于）

The operators ``||`` and ``&&`` apply the common short-circuiting rules. This means that in the expression ``f(x) || g(y)``, if ``f(x)`` evaluates to ``true``, ``g(y)`` will not be evaluated even if it may have side-effects.

运算符 ``||`` 和 ``&&`` 都遵循短路（ short-circuiting ）规则。例如在表达式 ``f(x) || g(y)`` 中，
如果 ``f(x)`` 的值为 ``true`` ，那么 ``g(x)`` 就不会被执行，即使会出现一些副作用。

.. index:: ! uint, ! int, ! integer

Integers 整型
--------

``int`` / ``uint``: Signed and unsigned integers of various sizes. Keywords ``uint8`` to ``uint256`` in steps of ``8`` (unsigned of 8 up to 256 bits) and ``int8`` to ``int256``.
``uint`` and ``int`` are aliases for ``uint256`` and ``int256``, respectively.

``int`` / ``uint`` ：表示有符号和无符号不同位数整数。支持关键字 ``uint8`` 到 ``uint256`` （8位递增，从8位到256位）以及 ``int8`` 到 ``int256``。
``uint`` 和 ``int`` 分别是 ``uint256`` 和 ``int256`` 的别名。

Operators:

运算符：

* Comparisons: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (evaluate to ``bool``)
* Bit operators: ``&``, ``|``, ``^`` (bitwise exclusive or), ``~`` (bitwise negation)
* Arithmetic operators: ``+``, ``-``, unary ``-``, unary ``+``, ``*``, ``/``, ``%`` (remainder), ``**`` (exponentiation), ``<<`` (left shift), ``>>`` (right shift)

* 比较运算符： ``<=`` ， ``<`` ， ``==`` ， ``!=`` ， ``>=`` ， ``>`` （返回布尔值）
* 位运算符： ``&`` ， ``|`` ， ``^`` （异或）， ``~`` （位取反）
* 算数运算符： ``+`` ， ``-`` ， 一元运算 ``-`` ， 一元运算 ``+`` ， ``*`` ， ``/`` ， ``%`` （取余） ， ``**`` （幂）， ``<<`` （左移位） ， ``>>`` （右移位）

Division always truncates (it is just compiled to the ``DIV`` opcode of the EVM), but it does not truncate if both
operators are :ref:`literals<rational_literals>` (or literal expressions).

除法总是会截断的（仅仅编译到 EVM 中的 ``DIV`` 操作码），
但如果运算符都是 :ref:`字面量（literals）<rational_literals>` （或者字面表达式），则不会截断。

Division by zero and modulus with zero throws a runtime exception.

除以零或者模零运算都会引发运行时异常。

The result of a shift operation is the type of the left operand. The
expression ``x << y`` is equivalent to ``x * 2**y``, and ``x >> y`` is
equivalent to ``x / 2**y``. This means that shifting negative numbers
sign extends. Shifting by a negative amount throws a runtime exception.

移位运算的结果取决于运算符左边的类型。
表达式 ``x << y`` 与 ``x * 2**y`` 是等价的，
``x >> y`` 与 ``x / 2**y``是等价的。这意味着将负数符号转移。
按负数移动会引发运行时异常。


.. warning::
    The results produced by shift right of negative values of signed integer types is different from those produced
    by other programming languages. In Solidity, shift right maps to division so the shifted negative values
    are going to be rounded towards zero (truncated). In other programming languages the shift right of negative values
    works like division with rounding down (towards negative infinity).

·· warning::
   由有符号整数类型负值右移所产生的结果跟其它语言中所产生的结果是不同的。
   在 Solidity 中，右移和除是等价的，因此右移位一个负数向下取整时会为零（被截断）。
   而在其它语言中， 右移负数位的结果就像除以了负无穷。

.. index:: ! ufixed, ! fixed, ! fixed point number

Fixed Point Numbers 定长浮点型
-------------------

.. warning::
    Fixed point numbers are not fully supported by Solidity yet. They can be declared, but
    cannot be assigned to or from.

.. warning::
    Solidity 还没有完全支持定长浮点型。可以声明定长浮点型的变量，但不能给它们赋值。

``fixed`` / ``ufixed``: Signed and unsigned fixed point number of various sizes. Keywords ``ufixedMxN`` and ``fixedMxN``, where ``M`` represents the number of bits taken by
the type and ``N`` represents how many decimal points are available. ``M`` must be divisible by 8 and goes from 8 to 256 bits. ``N`` must be between 0 and 80, inclusive.
``ufixed`` and ``fixed`` are aliases for ``ufixed128x19`` and ``fixed128x19``, respectively.

``fixed`` / ``ufixed``：表示各种大小的有符号和无符号的定长浮点型。
在关键词 ``ufixedMxN`` 和 ``fixedMxN`` 中，``M`` 表示该类型占用的位数，``N``表示可用的十进制长度。
``M`` 必须能整除 8，表示范围从 8 到 256 的位数。
``N`` 则可以是从 0 到 80 之间的任意数。
``ufixed`` 和 ``fixed`` 分别是 ``ufixed128x19`` 和 ``fixed128x19`` 的别名。

Operators:

运算符：

* Comparisons: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (evaluate to ``bool``)
* Arithmetic operators: ``+``, ``-``, unary ``-``, unary ``+``, ``*``, ``/``, ``%`` (remainder)

* 比较运算符：``<=``， ``<``， ``==``， ``!=``， ``>=``， ``>`` （返回值是布尔型）
* 算术运算符：``+``， ``-``， 一元运算 ``-``， 一元运算 ``+``， ``*``， ``/``， ``%`` （取余数）

.. note::
    The main difference between floating point (``float`` and ``double`` in many languages, more precisely IEEE 754 numbers) and fixed point numbers is
    that the number of bits used for the integer and the fractional part (the part after the decimal dot) is flexible in the former, while it is strictly
    defined in the latter. Generally, in floating point almost the entire space is used to represent the number, while only a small number of bits define
    where the decimal point is.

.. note::
    浮点型（在许多语言中的 ``float`` 和 ``double`` 类型，更准确地说是 IEEE 754 类型）和定长浮点型之间最大的不同点是，
    在前者中整数部分和分数部分（小数点后的部分）需要的位数是灵活可变的，而后者中这两部分的长度受到严格的规定。
    一般来说，在浮点型中，几乎整个空间都用来表示数字，但只有少数的位来表示小数点的位置。

.. index:: address, balance, send, call, callcode, delegatecall, transfer

.. _address:

Address 地址型
-------

``address``: Holds a 20 byte value (size of an Ethereum address). Address types also have members and serve as a base for all contracts.

``address``：地址型存储一个 20 字节的值（以太坊地址的大小）。
地址型也有成员变量，并作为所有合约的基础。

Operators:

运算符：

* ``<=``, ``<``, ``==``, ``!=``, ``>=`` and ``>``

* ``<=``， ``<``， ``==``， ``!=``， ``>=`` 和 ``>``

.. note::
    Starting with version 0.5.0 contracts do not derive from the address type, but can still be explicitly converted to address.

.. note::
    从0.5.0版本开始，合约不会从地址型派生，但仍然可以明确地转换成地址型。

.. _members-of-addresses:

Members of Addresses 地址成员变量
^^^^^^^^^^^^^^^^^^^^

* ``balance`` and ``transfer``

* ``balance`` 和 ``transfer``

For a quick reference, see :ref:`address_related`.

快速参考，请见 :ref:`address_related`。


It is possible to query the balance of an address using the property ``balance``
and to send Ether (in units of wei) to an address using the ``transfer`` function:

可以使用 ``balance`` 属性来查询一个地址的余额，
也可以使用 ``transfer`` 函数向一个地址发送 |ether| （以 wei 为单位）：

::

    address x = 0x123;
    address myAddress = this;
    if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);

.. note::
    If ``x`` is a contract address, its code (more specifically: its fallback function, if present) will be executed together with the ``transfer`` call (this is a feature of the EVM and cannot be prevented).
    If that execution runs out of gas or fails in any way, the Ether transfer will be reverted and the current contract will stop with an exception.

.. note::
    如果 ``x`` 是一个合约地址，它的代码（具体来说是它的 fallback 函数，如果有的话）会跟 ``transfer`` 函数调用一起执行（这是 EVM 的一个特性，无法改变）。
    如果在执行过程中用光了 gas 或者因为任何原因执行失败，|ether| 交易会被打回，当前的合约也会在终止的同时抛出异常。

* ``send``

Send is the low-level counterpart of ``transfer``. If the execution fails, the current contract will not stop with an exception, but ``send`` will return ``false``.

``send`` 是比 ``transfer`` 低级一点的函数。如果执行失败，当前的合约在终止时不会抛出异常，但 ``send`` 会返回 ``false``。

.. warning::
    There are some dangers in using ``send``: The transfer fails if the call stack depth is at 1024
    (this can always be forced by the caller) and it also fails if the recipient runs out of gas. So in order
    to make safe Ether transfers, always check the return value of ``send``, use ``transfer`` or even better:
    use a pattern where the recipient withdraws the money.

.. warning::
    在使用 ``send`` 的时候会有些风险：如果调用栈深度是 1024 会导致发送失败（这总是可以被调用者强制），如果接收者用光了 gas 也会导致发送失败。
    所以为了保证 |ether| 发送的安全，一定要检查 ``send`` 的返回值，使用 ``transfer`` 或者更好地办法：
    使用一种接收者可以取回资金的模式。

* ``call``, ``callcode`` and ``delegatecall``

* ``call``， ``callcode`` 和 ``delegatecall``

Furthermore, to interface with contracts that do not adhere to the ABI,
the function ``call`` is provided which takes an arbitrary number of arguments of any type. These arguments are padded to 32 bytes and concatenated.
One exception is the case where the first argument is encoded to exactly four bytes. In this case, it is not padded to allow the use of function signatures here.

此外，为了与不符合 |ABI| 的合约交互，于是就有了可以接受任意类型任意数量参数的 ``call`` 函数。
这些参数连接在一起填充在 32 字节的空间里。
其中一个例外是当第一个参数被编码成正好 4 个字节的情况。
在这种情况下，它被填充后不能使用函数签名。

::

    address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
    nameReg.call("register", "MyName");
    nameReg.call(bytes4(keccak256("fun(uint256)")), a);

``call`` returns a boolean indicating whether the invoked function terminated (``true``) or caused an EVM exception (``false``).
It is not possible to access the actual data returned (for this we would need to know the encoding and size in advance).

``call`` 返回的布尔值表明了被调用的函数已经执行完毕（``true``）或者引发了一个 EVM 异常（``false``）规则。
无法访问返回的真实数据（为此我们需要事先知道编码和大小）。

It is possible to adjust the supplied gas with the ``.gas()`` modifier::

    namReg.call.gas(1000000)("register", "MyName");

Similarly, the supplied Ether value can be controlled too::

    nameReg.call.value(1 ether)("register", "MyName");

Lastly, these modifiers can be combined. Their order does not matter::

    nameReg.call.gas(1000000).value(1 ether)("register", "MyName");

可以使用 ``.gas()`` 修饰器调整提供的 gas 数量 ::

    namReg.call.gas(1000000)("register", "MyName");

类似地，也能控制提供的 |ether| 的值 ::

   nameReg.call.value(1 ether)("register", "MyName"); 

最后一点，这些修饰器可以联合使用。每个修改器出现的顺序不重要 ::

   nameReg.call.gas(1000000).value(1 ether)("register", "MyName"); 

.. note::
    It is not yet possible to use the gas or value modifiers on overloaded functions.

    A workaround is to introduce a special case for gas and value and just re-check
    whether they are present at the point of overload resolution.

.. note::
    目前还不能在重载函数中使用 gas 或者值修改器。

    一种解决方案是给 gas 和值引入一个特例，并重新检查它们是否在重载的地方出现。

In a similar way, the function ``delegatecall`` can be used:
the difference is that only the code of the given address is used, all other aspects (storage, balance, ...) are taken from the current contract.
The purpose of ``delegatecall`` is to use library code which is stored in another contract.
The user has to ensure that the layout of storage in both contracts is suitable for delegatecall to be used.
Prior to homestead, only a limited variant called ``callcode`` was available that did not provide access to the original ``msg.sender`` and ``msg.value`` values.

类似地，也可以使用 ``delegatecall``：
区别在于只使用给定地址的代码，其它属性（存储，余额，……）都取自当前合约。
``delegatecall`` 的目的是使用存储在另外一个合约中的库代码。
用户必须确保两个合约中存储的分布适合使用 delegatecall。
在 homestead 版本之前，只有一个功能类似但有限的叫作 ``callcode ``的函数可用，但使用它并不能访问 ``msg.sender`` 和 ``msg.value`` 的原始值。

All three functions ``call``, ``delegatecall`` and ``callcode`` are very low-level functions and should only be used as a *last resort* as they break the type-safety of Solidity.

这三个函数 ``call``， ``delegatecall`` 和 ``callcode`` 都是非常低级的函数，应该只把它们当作 *最后一招* 来使用，因为它们破坏了 Solitity 的类型安全性。

The ``.gas()`` option is available on all three methods, while the ``.value()`` option is not supported for ``delegatecall``.

尽管 ``.value()`` 选项不支持 ``delegatecall``，但三种方法都有 ``.gas()`` 选项。

.. note::
    All contracts inherit the members of address, so it is possible to query the balance of the
    current contract using ``this.balance``.
    所有合约都集成地址类型的所有成员变量，因此可以使用 ``this.balance`` 访问当前合约的余额。

.. note::
    The use of ``callcode`` is discouraged and will be removed in the future.
    不鼓励使用 ``callcode``，在未来也会将其移除。


.. warning::
    All these functions are low-level functions and should be used with care.
    Specifically, any unknown contract might be malicious and if you call it, you
    hand over control to that contract which could in turn call back into
    your contract, so be prepared for changes to your state variables
    when the call returns.
    这三个函数都属于低级函数，需要谨慎使用。
    具体来说，任何未知的合约都可能是恶意的。
    你在调用一个合约的同时就将控制权交给了它，它可以反过来调用你的合约，
    因此，当调用返回时准备好改变你的状态变量。

.. index:: byte array, bytes32

Fixed-size byte arrays 定长字节数组
----------------------

``bytes1``, ``bytes2``, ``bytes3``, ..., ``bytes32``. ``byte`` is an alias for ``bytes1``.

关键词有：``bytes1``， ``bytes2``， ``bytes3``， ...， ``bytes32``。``byte`` 是 ``bytes1`` 的别名。

Operators:

运算符：

* Comparisons: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (evaluate to ``bool``)
* Bit operators: ``&``, ``|``, ``^`` (bitwise exclusive or), ``~`` (bitwise negation), ``<<`` (left shift), ``>>`` (right shift)
* Index access: If ``x`` is of type ``bytesI``, then ``x[k]`` for ``0 <= k < I`` returns the ``k`` th byte (read-only).

* 比较运算符：``<=``， ``<``， ``==``， ``!=``， ``>=``， ``>`` （返回布尔型）
* 位运算符： ``&``， ``|``， ``^`` （按位异或）， ``~`` （按位取反）， ``<<`` （左移位）， ``>>`` （右移位）
* 索引访问：如果 ``x`` 是 ``bytesI`` 类型，那么 ``x[k]`` 且 ``0 <= k < I`` 返回第 ``k`` 个字节（只读）。

The shifting operator works with any integer type as right operand (but will
return the type of the left operand), which denotes the number of bits to shift by.
Shifting by a negative amount will cause a runtime exception.

该类型可以和作为右操作数的任何整数类型进行移位运算（但返回结果的类型和左操作数类型相同），右操作数表示需要移动的位数。
进行负数位移运算会引发运行时异常。

Members:

成员变量：

* ``.length`` yields the fixed length of the byte array (read-only).

* ``.length`` 表示这个字节数组的长度（只读）.

.. note::
    It is possible to use an array of bytes as ``byte[]``, but it is wasting a lot of space, 31 bytes every element,
    to be exact, when passing in calls. It is better to use ``bytes``.
    可以将 ``byte[]`` 当作字节数组使用，但这种方式非常浪费存储空间，准确来说，是在传入调用时，每个元素占 31 字节。
    更好地做法是使用 ``bytes``。

Dynamically-sized byte array 变长字节数组
----------------------------

``bytes``:
    Dynamically-sized byte array, see :ref:`arrays`. Not a value-type!
    变长字节数组，参见 :ref:`arrays`，并不是值类型。
``string``:
    Dynamically-sized UTF-8-encoded string, see :ref:`arrays`. Not a value-type!
    变长 UTF-8 编码字符串类型，参见 :ref:`arrays`，并不是值类型。

As a rule of thumb, use ``bytes`` for arbitrary-length raw byte data and ``string``
for arbitrary-length string (UTF-8) data. If you can limit the length to a certain
number of bytes, always use one of ``bytes1`` to ``bytes32`` because they are much cheaper.

根据经验，最好使用 ``bytes`` 存储任意长度的原始字节数据，使用 ``string`` 存储任意长度的字符串（UTF-8编码）数据。
如果长度可以确定，尽量使用 ``bytes1`` 到 ``bytes32`` 中的一个，因为它们的开销小得多。

.. index:: address, literal;address

.. _address_literals:

Address Literals 地址常量（Address Literals）
----------------

.. note::
    ``literal``也被译作“字面量”。下文统一译作“常量”

Hexadecimal literals that pass the address checksum test, for example
``0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF`` are of ``address`` type.
Hexadecimal literals that are between 39 and 41 digits
long and do not pass the checksum test produce
a warning and are treated as regular rational number literals.

比如像 ``0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF`` 这样的通过了地址校验和测试的十六进制常量属于 ``address`` 类型。
长度在 39 到 41 个数字的，没有通过校验和测试而产生了一个警告的十六进制常量被当做正常的有理数常量。

.. note::
    The mixed-case address checksum format is defined in `EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_.
    混合大小写的地址校验和格式定义在 `EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_ 中。


.. index:: literal, literal;rational

.. _rational_literals:

Rational and Integer Literals 有理数和整数常量
-----------------------------

Integer literals are formed from a sequence of numbers in the range 0-9.
They are interpreted as decimals. For example, ``69`` means sixty nine.
Octal literals do not exist in Solidity and leading zeros are invalid.

整数常量由范围在 0-9 的一串数字组成，表现成十进制。
例如，`69` 表示数字六十九。
Solidity 中是没有八进制的，因此前置 0 是无效的。

Decimal fraction literals are formed by a ``.`` with at least one number on
one side.  Examples include ``1.``, ``.1`` and ``1.3``.

十进制小数常量带有一个 ``.``，至少在其一边会有一个数字。
有效的事例如：``1.``，``.1``，和 ``1.3``。

Scientific notation is also supported, where the base can have fractions, while the exponent cannot.
Examples include ``2e10``, ``-2e10``, ``2e-10``, ``2.5e1``.

科学符号也是支持的，尽管指数必须是整数，但基数可以是小数。
有效的事例如：``2e10``， ``-2e10``， ``2e-10``， ``2.5e1``。

Number literal expressions retain arbitrary precision until they are converted to a non-literal type (i.e. by
using them together with a non-literal expression).
This means that computations do not overflow and divisions do not truncate
in number literal expressions.

数值常量表达式本身支持任意精度，除非它们被转换成了非常量类型（例如，当它们出现在非常量表达式中时就会发生转换）。
这意味着在数值常量表达式中计算时不会出现溢出或除法截断。

For example, ``(2**800 + 1) - 2**800`` results in the constant ``1`` (of type ``uint8``)
although intermediate results would not even fit the machine word size. Furthermore, ``.5 * 8`` results
in the integer ``4`` (although non-integers were used in between).

例如，``(2**800 + 1) - 2**800`` 的结果是常量 ``1``（属于 ``uint8`` 类型），尽管计算的中间结果已经超过了计算机字长。
此外， ``.5 * 8`` 的结果是整型 ``4``（尽管有非整型参与了计算）。

Any operator that can be applied to integers can also be applied to number literal expressions as
long as the operands are integers. If any of the two is fractional, bit operations are disallowed
and exponentiation is disallowed if the exponent is fractional (because that might result in
a non-rational number).

只要操作数是整型，任意整型支持的运算符都可以被运用在数值常量表达式中。
如果两个中的任一个数是小数，则不允许进行位运算。如果指数是小数的话，也不支持幂运算（因为这样可能会得到一个无理数）。

.. note::
    Solidity has a number literal type for each rational number.
    Integer literals and rational number literals belong to number literal types.
    Moreover, all number literal expressions (i.e. the expressions that
    contain only number literals and operators) belong to number literal
    types.  So the number literal expressions ``1 + 2`` and ``2 + 1`` both
    belong to the same number literal type for the rational number three.
    Solidity 对每个有理数都有对应的数值常量类型。
    整数常量和有理数常量都属于数值常量类型。
    除此之外，所有的数值常量表达式（即只包含数值常量和运算符的表达式）都属于数值常量类型。
    因此数值常量表达式 ``1 + 2`` 和 ``2 + 1`` 的结果跟有理数三的数值常量类型相同。

.. warning::
    Division on integer literals used to truncate in earlier versions, but it will now convert into a rational number, i.e. ``5 / 2`` is not equal to ``2``, but to ``2.5``.
    在早期版本中，整数常量的除法也会截断，但在现在的版本中，会将结果转换成一个有理数。例如 ``5 / 2`` 并不等于 ``2``，而是等于 ``2.5``。

.. note::
    Number literal expressions are converted into a non-literal type as soon as they are used with non-literal
    expressions. Even though we know that the value of the
    expression assigned to ``b`` in the following example evaluates to
    an integer, but the partial expression ``2.5 + a`` does not type check so the code
    does not compile
    数值常量表达式只要在非常量表达式中使用就会转换成非常量类型。
    在下面的例子中，尽管我们知道 ``b`` 的值是一个整数，但 ``2.5 + a`` 这部分表达式并不进行类型检查，因此编译不能通过。

::

    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5;

.. index:: literal, literal;string, string

String Literals 字符串常量
---------------

String literals are written with either double or single-quotes (``"foo"`` or ``'bar'``).
They do not imply trailing zeroes as in C; ``"foo"`` represents three bytes not four.
As with integer literals, their type can vary, but they are implicitly convertible to ``bytes1``, ..., ``bytes32``, if they fit, to ``bytes`` and to ``string``.

字符串常量是指由双引号或单引号引起来的字符串（``"foo"`` 或者 ``'bar'``）。
不像在 C 语言中那样带有结束符；``"foo"`` 表示 3 个字节而不是 4 个。

String literals support escape characters, such as ``\n``, ``\xNN`` and ``\uNNNN``.
``\xNN`` takes a hex value and inserts the appropriate byte,
while ``\uNNNN`` takes a Unicode codepoint and inserts an UTF-8 sequence.

字符串常量支持转义字符，例如 ``\n``，``\xNN`` 和 ``\uNNNN``。``\xNN`` 表示一个 16 进制值，最终转换成合适的字节，
而 ``\uNNNN`` 表示 Unicode 编码值，最终会转换为 UTF-8 的序列。

.. index:: literal, bytes

Hexadecimal Literals 十六进制常量
--------------------

Hexademical Literals are prefixed with the keyword ``hex`` and are enclosed in double or single-quotes (``hex"001122FF"``).
Their content must be a hexadecimal string and their value will be the binary representation of those values.

十六进制常量以关键字 ``hex`` 打头，后面紧跟着用单引号或双引号引起来的字符串（例如，``hex"001122FF"``）。
字符串的内容必须是一个十六进制的字符串，它的值将使用二进制表示。

Hexademical Literals behave like String Literals and have the same convertibility restrictions.

十六进制常量跟字符串常量很类似，具有相同的转换规则。

.. index:: enum

.. _enums:

Enums 枚举类型
-----

Enums are one way to create a user-defined type in Solidity. They are explicitly convertible
to and from all integer types but implicit conversion is not allowed.  The explicit conversions
check the value ranges at runtime and a failure causes an exception.  Enums needs at least one member.

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

        // Since enum types are not part of the ABI, the signature of "getChoice"
        // will automatically be changed to "getChoice() returns (uint8)"
        // for all matters external to Solidity. The integer type used is just
        // large enough to hold all enum values, i.e. if you have more values,
        // `uint16` will be used and so on.
        function getChoice() public view returns (ActionChoices) {
            return choice;
        }

        function getDefaultChoice() public pure returns (uint) {
            return uint(defaultChoice);
        }
    }

.. index:: ! function type, ! type; function

.. _function_types:

Function Types 函数类型
--------------

Function types are the types of functions. Variables of function type
can be assigned from functions and function parameters of function type
can be used to pass functions to and return functions from function calls.
Function types come in two flavours - *internal* and *external* functions:

函数类型是一种表示函数的类型。可以将一个函数赋值给另一个函数类型的变量，也可以将一个函数作为参数进行传递，还能在函数调用中返回函数类型变量。
函数类型有两类：- *内部（internal）* 函数和 *外部（external）* 函数：

Internal functions can only be called inside the current contract (more specifically,
inside the current code unit, which also includes internal library functions
and inherited functions) because they cannot be executed outside of the
context of the current contract. Calling an internal function is realized
by jumping to its entry label, just like when calling a function of the current
contract internally.

内部函数只能在当前合约内被调用（更具体来说，在当前代码块内，包括内部库函数和继承的函数中），因为它们不能在当前合约上下文的外部被执行。
调用一个内部函数是通过跳转到它的入口标签来实现的，就像在当前合约的内部调用一个函数。

External functions consist of an address and a function signature and they can
be passed via and returned from external function calls.

外部函数由一个地址和一个函数签名组成，可以通过外部函数调用传递或者返回。

Function types are notated as follows::

函数类型表示成如下的形式 ::

    function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]

In contrast to the parameter types, the return types cannot be empty - if the
function type should not return anything, the whole ``returns (<return types>)``
part has to be omitted.

与参数类型相比，返回类型不能为空 —— 如果函数类型不需要返回，则需要删除整个 ``returns (<return types>)`` 部分。

By default, function types are internal, so the ``internal`` keyword can be
omitted. In contrast, contract functions themselves are public by default,
only when used as the name of a type, the default is internal.

函数类型默认是内部函数，因此不需要声明 ``internal`` 关键词。
与此相反的是，合约中的函数本身默认是 public 的，只有当它被当做类型名称时，默认才是内部函数。

There are two ways to access a function in the current contract: Either directly
by its name, ``f``, or using ``this.f``. The former will result in an internal
function, the latter in an external function.

有两种方法可以访问当前合约中的函数：其中一种方法是直接使用它的名字，``f``，另一种方法是使用 ``this.f``。
前者适用于内部函数，后者适用于外部函数。

If a function type variable is not initialized, calling it will result
in an exception. The same happens if you call a function after using ``delete``
on it.

如果当函数类型的变量还没有初始化时就调用它的话会引发一个异常。
如果在一个函数被 ``delete`` 之后调用它也会发生相同的情况。

If external function types are used outside of the context of Solidity,
they are treated as the ``function`` type, which encodes the address
followed by the function identifier together in a single ``bytes24`` type.

如果外部函数类型在 Solidity 的上下文环境意外的地方使用，它们会被视为 ``function`` 类型。
该类型将函数所在地址及其函数标识一起编码为一个 ``bytes24`` 类型。

Note that public functions of the current contract can be used both as an
internal and as an external function. To use ``f`` as an internal function,
just use ``f``, if you want to use its external form, use ``this.f``.

请注意，当前合约的 public 函数既可以被当作内部函数也可以被当作外部函数使用。
如果想将一个函数当作内部函数使用，就用 ``f``调用，如果想将其当作外部函数，使用 ``this.f``。

Additionally, public (or external) functions also have a special member called ``selector``,
which returns the :ref:`ABI function selector <abi_function_selector>`::

除此之外，public（或外部）函数也有一个特殊的成员变量称作 ``selector``，可以返回 :ref:`ABI 函数选择器 <abi_function_selector>`::

    pragma solidity ^0.4.16;

    contract Selector {
      function f() public view returns (bytes4) {
        return this.f.selector;
      }
    }

Example that shows how to use internal function types::

如果使用内部函数类型的例子::

    pragma solidity ^0.4.16;

    library ArrayUtils {
      // internal functions can be used in internal library functions because
      // they will be part of the same code context
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

Another example that uses external function types::

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
        // Here goes the check that the reply comes from a trusted source
        requests[requestID].callback(response);
      }
    }

    contract OracleUser {
      Oracle constant oracle = Oracle(0x1234567); // known contract
      function buySomething() {
        oracle.query("USD", this.oracleResponse);
      }
      function oracleResponse(bytes response) public {
        require(msg.sender == address(oracle));
        // Use the data
      }
    }

.. note::
    Lambda or inline functions are planned but not yet supported.
    Lambda 表达式或者内联函数的引入在计划内，但目前还没支持。

.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

Reference Types
==================

Complex types, i.e. types which do not always fit into 256 bits have to be handled
more carefully than the value-types we have already seen. Since copying
them can be quite expensive, we have to think about whether we want them to be
stored in **memory** (which is not persisting) or **storage** (where the state
variables are held).

Data location
-------------

Every complex type, i.e. *arrays* and *structs*, has an additional
annotation, the "data location", about whether it is stored in memory or in storage. Depending on the
context, there is always a default, but it can be overridden by appending
either ``storage`` or ``memory`` to the type. The default for function parameters (including return parameters) is ``memory``, the default for local variables is ``storage`` and the location is forced
to ``storage`` for state variables (obviously).

There is also a third data location, ``calldata``, which is a non-modifiable,
non-persistent area where function arguments are stored. Function parameters
(not return parameters) of external functions are forced to ``calldata`` and
behave mostly like ``memory``.

Data locations are important because they change how assignments behave:
assignments between storage and memory and also to a state variable (even from other state variables)
always create an independent copy.
Assignments to local storage variables only assign a reference though, and
this reference always points to the state variable even if the latter is changed
in the meantime.
On the other hand, assignments from a memory stored reference type to another
memory-stored reference type do not create a copy.

::

    pragma solidity ^0.4.0;

    contract C {
        uint[] x; // the data location of x is storage

        // the data location of memoryArray is memory
        function f(uint[] memoryArray) public {
            x = memoryArray; // works, copies the whole array to storage
            var y = x; // works, assigns a pointer, data location of y is storage
            y[7]; // fine, returns the 8th element
            y.length = 2; // fine, modifies x through y
            delete x; // fine, clears the array, also modifies y
            // The following does not work; it would need to create a new temporary /
            // unnamed array in storage, but storage is "statically" allocated:
            // y = memoryArray;
            // This does not work either, since it would "reset" the pointer, but there
            // is no sensible location it could point to.
            // delete y;
            g(x); // calls g, handing over a reference to x
            h(x); // calls h and creates an independent, temporary copy in memory
        }

        function g(uint[] storage storageArray) internal {}
        function h(uint[] memoryArray) public {}
    }

Summary
^^^^^^^

Forced data location:
 - parameters (not return) of external functions: calldata
 - state variables: storage

Default data location:
 - parameters (also return) of functions: memory
 - all other local variables: storage

.. index:: ! array

.. _arrays:

Arrays
------

Arrays can have a compile-time fixed size or they can be dynamic.
For storage arrays, the element type can be arbitrary (i.e. also other
arrays, mappings or structs). For memory arrays, it cannot be a mapping and
has to be an ABI type if it is an argument of a publicly-visible function.

An array of fixed size ``k`` and element type ``T`` is written as ``T[k]``,
an array of dynamic size as ``T[]``. As an example, an array of 5 dynamic
arrays of ``uint`` is ``uint[][5]`` (note that the notation is reversed when
compared to some other languages). To access the second uint in the
third dynamic array, you use ``x[2][1]`` (indices are zero-based and
access works in the opposite way of the declaration, i.e. ``x[2]``
shaves off one level in the type from the right).

Variables of type ``bytes`` and ``string`` are special arrays. A ``bytes`` is similar to ``byte[]``,
but it is packed tightly in calldata. ``string`` is equal to ``bytes`` but does not allow
length or index access (for now).

So ``bytes`` should always be preferred over ``byte[]`` because it is cheaper.

.. note::
    If you want to access the byte-representation of a string ``s``, use
    ``bytes(s).length`` / ``bytes(s)[7] = 'x';``. Keep in mind
    that you are accessing the low-level bytes of the UTF-8 representation,
    and not the individual characters!

It is possible to mark arrays ``public`` and have Solidity create a :ref:`getter <visibility-and-getters>`.
The numeric index will become a required parameter for the getter.

.. index:: ! array;allocating, new

Allocating Memory Arrays
^^^^^^^^^^^^^^^^^^^^^^^^

Creating arrays with variable length in memory can be done using the ``new`` keyword.
As opposed to storage arrays, it is **not** possible to resize memory arrays by assigning to
the ``.length`` member.

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);
            // Here we have a.length == 7 and b.length == len
            a[6] = 8;
        }
    }

.. index:: ! array;literals, !inline;arrays

Array Literals / Inline Arrays
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Array literals are arrays that are written as an expression and are not
assigned to a variable right away.

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

::

    // This will not compile.

    pragma solidity ^0.4.0;

    contract C {
        function f() public {
            // The next line creates a type error because uint[3] memory
            // cannot be converted to uint[] memory.
            uint[] x = [uint(1), 3, 4];
        }
    }

It is planned to remove this restriction in the future but currently creates
some complications because of how arrays are passed in the ABI.

.. index:: ! array;length, length, push, !array;push

Members
^^^^^^^

**length**:
    Arrays have a ``length`` member to hold their number of elements.
    Dynamic arrays can be resized in storage (not in memory) by changing the
    ``.length`` member. This does not happen automatically when attempting to access elements outside the current length. The size of memory arrays is fixed (but dynamic, i.e. it can depend on runtime parameters) once they are created.
**push**:
     Dynamic storage arrays and ``bytes`` (not ``string``) have a member function called ``push`` that can be used to append an element at the end of the array. The function returns the new length.

.. warning::
    It is not yet possible to use arrays of arrays in external functions.

.. warning::
    Due to limitations of the EVM, it is not possible to return
    dynamic content from external function calls. The function ``f`` in
    ``contract C { function f() returns (uint[]) { ... } }`` will return
    something if called from web3.js, but not if called from Solidity.

    The only workaround for now is to use large statically-sized arrays.


::

    pragma solidity ^0.4.16;

    contract ArrayContract {
        uint[2**20] m_aLotOfIntegers;
        // Note that the following is not a pair of dynamic arrays but a
        // dynamic array of pairs (i.e. of fixed size arrays of length two).
        bool[2][] m_pairsOfFlags;
        // newPairs is stored in memory - the default for function arguments

        function setAllFlagPairs(bool[2][] newPairs) public {
            // assignment to a storage array replaces the complete array
            m_pairsOfFlags = newPairs;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // access to a non-existing index will throw an exception
            m_pairsOfFlags[index][0] = flagA;
            m_pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // if the new size is smaller, removed array elements will be cleared
            m_pairsOfFlags.length = newSize;
        }

        function clear() public {
            // these clear the arrays completely
            delete m_pairsOfFlags;
            delete m_aLotOfIntegers;
            // identical effect here
            m_pairsOfFlags.length = 0;
        }

        bytes m_byteData;

        function byteArrays(bytes data) public {
            // byte arrays ("bytes") are different as they are stored without padding,
            // but can be treated identical to "uint8[]"
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
            uint[2][] memory arrayOfPairs = new uint[2][](size);
            // Create a dynamic byte array:
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = byte(i);
            return b;
        }
    }


.. index:: ! struct, ! type;struct

.. _structs:

Structs
-------

Solidity provides a way to define new types in the form of structs, which is
shown in the following example:

::

    pragma solidity ^0.4.11;

    contract CrowdFunding {
        // Defines a new type with two fields.
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
            campaignID = numCampaigns++; // campaignID is return variable
            // Creates new struct and saves in storage. We leave out the mapping type.
            campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // Creates a new temporary memory struct, initialised with the given values
            // and copies it over to storage.
            // Note that you can also use Funder(msg.sender, msg.value) to initialise.
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

It is not possible for a struct to contain a member of its own type,
although the struct itself can be the value type of a mapping member.
This restriction is necessary, as the size of the struct has to be finite.

Note how in all the functions, a struct type is assigned to a local variable
(of the default storage data location).
This does not copy the struct but only stores a reference so that assignments to
members of the local variable actually write to the state.

Of course, you can also directly access the members of the struct without
assigning it to a local variable, as in
``campaigns[campaignID].amount = 0``.

.. index:: !mapping

Mappings
========

Mapping types are declared as ``mapping(_KeyType => _ValueType)``.
Here ``_KeyType`` can be almost any type except for a mapping, a dynamically sized array, a contract, an enum and a struct.
``_ValueType`` can actually be any type, including mappings.

Mappings can be seen as `hash tables <https://en.wikipedia.org/wiki/Hash_table>`_ which are virtually initialized such that
every possible key exists and is mapped to a value whose byte-representation is
all zeros: a type's :ref:`default value <default-value>`. The similarity ends here, though: The key data is not actually stored
in a mapping, only its ``keccak256`` hash used to look up the value.

Because of this, mappings do not have a length or a concept of a key or value being "set".

Mappings are only allowed for state variables (or as storage reference types
in internal functions).

It is possible to mark mappings ``public`` and have Solidity create a :ref:`getter <visibility-and-getters>`.
The ``_KeyType`` will become a required parameter for the getter and it will
return ``_ValueType``.

The ``_ValueType`` can be a mapping too. The getter will have one parameter
for each ``_KeyType``, recursively.

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

.. index:: assignment, ! delete, lvalue

Operators Involving LValues
===========================

If ``a`` is an LValue (i.e. a variable or something that can be assigned to), the following operators are available as shorthands:

``a += e`` is equivalent to ``a = a + e``. The operators ``-=``, ``*=``, ``/=``, ``%=``, ``|=``, ``&=`` and ``^=`` are defined accordingly. ``a++`` and ``a--`` are equivalent to ``a += 1`` / ``a -= 1`` but the expression itself still has the previous value of ``a``. In contrast, ``--a`` and ``++a`` have the same effect on ``a`` but return the value after the change.

delete
------

``delete a`` assigns the initial value for the type to ``a``. I.e. for integers it is equivalent to ``a = 0``, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset.

``delete`` has no effect on whole mappings (as the keys of mappings may be arbitrary and are generally unknown). So if you delete a struct, it will reset all members that are not mappings and also recurse into the members unless they are mappings. However, individual keys and what they map to can be deleted.

It is important to note that ``delete a`` really behaves like an assignment to ``a``, i.e. it stores a new object in ``a``.

::

    pragma solidity ^0.4.0;

    contract DeleteExample {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // sets x to 0, does not affect data
            delete data; // sets data to 0, does not affect x which still holds a copy
            uint[] storage y = dataArray;
            delete dataArray; // this sets dataArray.length to zero, but as uint[] is a complex object, also
            // y is affected which is an alias to the storage object
            // On the other hand: "delete y" is not valid, as assignments to local variables
            // referencing storage objects can only be made from existing storage objects.
        }
    }

.. index:: ! type;conversion, ! cast

Conversions between Elementary Types
====================================

Implicit Conversions
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

Explicit Conversions
--------------------

If the compiler does not allow implicit conversion but you know what you are
doing, an explicit type conversion is sometimes possible. Note that this may
give you some unexpected behaviour so be sure to test to ensure that the
result is what you want! Take the following example where you are converting
a negative ``int8`` to a ``uint``:

::

    int8 y = -3;
    uint x = uint(y);

At the end of this code snippet, ``x`` will have the value ``0xfffff..fd`` (64 hex
characters), which is -3 in the two's complement representation of 256 bits.

If a type is explicitly converted to a smaller type, higher-order bits are
cut off::

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // b will be 0x5678 now

.. index:: ! type;deduction, ! var

.. _type-deduction:

Type Deduction
==============

For convenience, it is not always necessary to explicitly specify the type of a
variable, the compiler automatically infers it from the type of the first
expression that is assigned to the variable::

    uint24 x = 0x123;
    var y = x;

Here, the type of ``y`` will be ``uint24``. Using ``var`` is not possible for function
parameters or return parameters.

.. warning::
    The type is only deduced from the first assignment, so
    the loop in the following snippet is infinite, as ``i`` will have the type
    ``uint8`` and the highest value of this type is smaller than ``2000``.
    ``for (var i = 0; i < 2000; i++) { ... }``

