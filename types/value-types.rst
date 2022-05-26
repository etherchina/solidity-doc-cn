.. include:: glossaries.rst

.. index:: ! value type, ! type;value

.. _value-types:

值类型
======

以下类型也称为值类型，因为这些类型的变量将始终按值来传递。
也就是说，当这些变量被用作函数参数或者用在赋值语句中时，总会进行值拷贝。

.. index:: ! bool, ! true, ! false

布尔类型
--------

``bool`` ：可能的取值为字面常量值 ``true`` 和 ``false`` 。

运算符：

*  ``!`` （逻辑非）
*  ``&&`` （逻辑与， "and" ）
*  ``||`` （逻辑或， "or" ）
*  ``==`` （等于）
*  ``!=`` （不等于）

运算符 ``||`` 和 ``&&`` 都遵循同样的短路（ short-circuiting ）规则。就是说在表达式 ``f(x) || g(y)`` 中，
如果 ``f(x)`` 的值为 ``true`` ，那么 ``g(y)`` 就不会被执行，即使会出现一些副作用。

.. index:: ! uint, ! int, ! integer
.. _integers:

整型
----

``int`` / ``uint`` ：分别表示有符号和无符号的不同位数的整型变量。
支持关键字 ``uint8`` 到 ``uint256`` （无符号，从 8 位到 256 位）以及 ``int8`` 到 ``int256``，以 ``8`` 位为步长递增。
``uint`` 和 ``int`` 分别是 ``uint256`` 和 ``int256`` 的别名。

运算符：

* 比较运算符： ``<=`` ， ``<`` ， ``==`` ， ``!=`` ， ``>=`` ， ``>`` （返回布尔值）
* 位运算符： ``&`` ， ``|`` ， ``^`` （异或）， ``~`` （位取反）
* 移位运算符： ``<<`` （左移位） ， ``>>`` （右移位）
* 算数运算符： ``+`` ， ``-`` ， 一元运算负 ``-`` （仅针对有符号整型）， ``*`` ， ``/`` ， ``%`` （取余或叫模运算） ， ``**`` （幂）


对于整形 ``X``，可以使用 ``type(X).min`` 和 ``type(X).max`` 去获取这个类型的最小值与最大值。


.. warning::
  Solidity中的整数是有取值范围的。 例如 ``uint32`` 类型的取值范围是 ``0`` 到 ``2 ** 32-1`` 。
  0.8.0 开始，算术运算有两个计算模式：一个是 "wrapping"（截断）模式或称 "unchecked"（不检查）模式，一个是"checked" （检查）模式。
  默认情况下，算术运算在 "checked" 模式下，即都会进行溢出检查，如果结果落在取值范围之外，调用会通过 :ref:`失败异常<assert-and-require>` 回退。
  你也可以通过 ``unchecked { ... }`` 切换到 "unchecked"模式，更多可参考 :ref:`unchecked <unchecked>` .


比较运算
^^^^^^^^^^^

比较整型的值

位运算
^^^^^^^^^^^^^^

位运算在数字的二进制补码表示上执行。
这意味着： ``~int256（0）== int256（-1）``。

移位
^^^^^^

移位操作的结果具有左操作数的类型，同时会截断结果以匹配类型。
右操作数必须是无符号类型。 尝试按带符号的类型移动将产生编译错误。

移位可以通过用2的幂的乘法来 "模拟"(方法如下)。请注意，左操作数的截断总是在最后发生，但是不会明确提醒（译者注：应该是指编译器不是提示）。

 - ``x << y`` 等于数学表达式 ``x * 2 ** y``。
 - ``x >> y`` 等于数学表达式 ``x / 2 ** y`` ， 四舍五入到负无穷。


.. warning::
  在版本 ``0.5.0`` 之前，对于负 ``x`` 的右移 ``x >> y`` 相当于 ``x / 2 ** y`` ，会四舍五入到零，而不是向负无穷。

.. note::
    对于移位操作不会像算术运算那样执行溢出检查，其结果总是被截断。


加、减、乘法运算
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

加法，减法和乘法和通常理解的语义一样，不过有两种模式来应对溢出（上溢及下溢）：

默认情况下，算术运算都会进行溢出检查，但是也可以禁用检查，可以通过 :ref:`unchecked block<unchecked>` 来禁用检查，此时会返回截断的结果，更多的详情可以前往链接查看。

.. note::
  溢出的检查功能是在 0.8.0 版本加入的，在此版本之前，请使用 OpenZepplin SafeMath 库。



表达式 ``-x`` 相当于 ``(T(0) - x)`` 这里 ``T`` 是指 ``x`` 的类型。 ``-x`` 只能应用在有符号型的整数上。
如果 ``x`` 为负数， ``-x`` 为正数。 由于使用两进制补码表示数据，你还需要小心:

如果有 ``int x = type(int).min;``， 那 ``-x`` 将不在正数取值的范围内。
这意味着这个检测 ``unchecked { assert(-x == x); }``  是可以通过的（即这种情况下，不能假设它的负数会是正数），如果是 checked 模式，则会触发异常。



除法运算
^^^^^^^^^^^^

除法运算结果的类型始终是其中一个操作数的类型，整数除法总是产生整数。
在Solidity中，分数会取零。 这意味着 ``int256(-5) / int256(2) == int256(-2)`` 。

注意在智能合约中，在 :ref:`字面常量<rational_literals>` 上进行除法会保留精度（保留小数位）。

.. note::
  除以0 会发生 :ref:`Panic 错误<assert-and-require>` ， 而且这个检查，不可以通过 ``unchecked { ... }`` 禁用掉。

.. note::
  表达式 ``type(int).min / (-1)`` 是仅有的整除会发生向上溢出的情况。
  在算术检查模式下，这会触发一个失败异常，在截断模式下，表达式的值将是 ``type(int).min`` 。

模运算（取余）
^^^^^^^^^^^^^^^

模运算 ``a％n`` 是在操作数 ``a`` 的除以 ``n`` 之后产生余数 ``r`` ，其中 ``q = int(a / n)`` 和 ``r = a - (n * q)`` 。 这意味着模运算结果与左操作数相同的符号相同（或零）。
对于 负数的a : ``a % n == -(-a % n)``， 几个例子：

* ``int256(5) % int256(2) == int256(1)``
* ``int256(5) % int256(-2) == int256(1)``
* ``int256(-5) % int256(2) == int256(-1)``
* ``int256(-5) % int256(-2) == int256(-1)``

.. note::
  对0取模会发生错误 :ref:`Panic 错误<assert-and-require>`，该检查不能通过``unchecked { ... }`` 。

幂运算
^^^^^^^^^^^^^^

幂运算仅适用于无符号类型。 结果的类型总是等于基数的类型.
请注意类型足够大以能够容纳幂运算的结果，要么发生潜在的assert异常或者使用截断模式。



.. note::
  在“checked” 模式下，幂运算仅会为小基数使用相对便宜的 ``exp`` 操作码。 
  例如 ``x**3`` 的例子，表达式 ``x*x*x`` 也许更便宜。
  在任何情况下，都建议进行 gas 消耗测试和使用优化器。
  

.. note::
  注意 ``0**0`` 在EVM中定义为 ``1`` 。


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

.. index:: address, balance, send, call, delegatecall, transfer

.. _address:

地址类型 Address
--------------------

地址类型有两种形式，他们大致相同：

 - ``address``：保存一个20字节的值（以太坊地址的大小）。
 - ``address payable`` ：可支付地址，与 ``address`` 相同，不过有成员函数 ``transfer`` 和 ``send`` 。

这种区别背后的思想是 ``address payable`` 可以向其发送以太币，而不能先一个普通的 ``address`` 发送以太币，例如，它可能是一个智能合约地址，并且不支持接收以太币。


类型转换:

允许从 ``address payable`` 到 ``address`` 的隐式转换，而从 ``address`` 到 ``address payable`` 必须显示的转换, 通过 ``payable(<address>)`` 进行转换。
.. note::
  
    在0.5版本,执行这种转换的唯一方法是使用中间类型，先转换为 ``uint160`` 如,  address payable ap = address(uint160(addr)); 

``address`` 允许和 ``uint160``、 整型字面常量、``bytes20`` 及合约类型相互转换。


只能通过 ``payable(...)`` 表达式把 ``address`` 类型和合约类型转换为 ``address payable``。
只有能接收以太币的合约类型，才能够进行此转换。例如合约要么有  :ref:`receive <receive-ether-function>` 或可支付的回退函数。
注意 ``payable(0)`` 是有效的，这是此规则的例外。


.. note::
    如果你需要 ``address`` 类型的变量，并计划发送以太币给这个地址，那么声明类型为 ``address payable`` 可以明确表达出你的需求。
    同样，尽量更早对他们进行区分或转换。

运算符:

* ``<=``, ``<``, ``==``, ``!=``, ``>=`` and ``>``

.. warning::
    如果将使用较大字节数组类型转换为 ``address`` ，例如 ``bytes32`` ，那么 ``address`` 将被截断。
    为了减少转换歧义，0.4.24及更高编译器版本要求我们在转换中显式截断处理。
    以32bytes值 ``0x111122223333444455556666777788889999AAAABBBBCCCCDDDDEEEEFFFFCCCC`` 为例， 如果使用 ``address(uint160(bytes20(b)))`` 结果是 ``0x111122223333444455556666777788889999aAaa``， 而使用 ``address(uint160(uint256(b)))`` 结果是 ``0x777788889999AaAAbBbbCcccddDdeeeEfFFfCcCc`` 。


.. note::
    ``address`` 和 ``address payable`` 的区别是在 0.5.0 版本引入的，同样从这个版本开始，合约类型不再继承自地址类型，
    不过如果合约有可支付的回退（ payable fallback ）函数或receive 函数，合约类型仍然可以显示转换为
    ``address`` 或 ``address payable`` 。

.. _members-of-addresses:

地址类型成员变量
^^^^^^^^^^^^^^^^
查看所有的成员，可参考 :ref:`address_related`。

* ``balance`` 和 ``transfer`` 成员

可以使用 ``balance`` 属性来查询一个地址的余额，
也可以使用 ``transfer`` 函数向一个可支付地址（payable address）发送 |ether| （以 wei 为单位）：

.. code-block:: solidity

    address x = 0x123;
    address myAddress = this;
    if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);

如果当前合约的余额不够多，则 ``transfer`` 函数会执行失败，或者如果以太转移被接收帐户拒绝， ``transfer`` 函数同样会失败而进行回退。

.. note::
    如果 ``x`` 是一个合约地址，它的代码（更具体来说是, 如果有receive函数,  执行 :ref:`receive-ether-function`, 或者存在fallback函数,执行 :ref:`fallback-function` 函数）会跟 ``transfer`` 函数调用一起执行（这是 EVM 的一个特性，无法阻止）。
    如果在执行过程中用光了 gas 或者因为任何原因执行失败，|ether| 交易会被打回，当前的合约也会在终止的同时抛出异常。

* ``send`` 成员

``send`` 是 ``transfer`` 的低级版本。如果执行失败，当前的合约不会因为异常而终止，但 ``send`` 会返回 ``false``。

.. warning::
    在使用 ``send`` 的时候会有些风险：如果调用栈深度是 1024 会导致发送失败（这总是可以被调用者强制），如果接收者用光了 gas 也会导致发送失败。
    所以为了保证 |ether| 发送的安全，一定要检查 ``send`` 的返回值，使用 ``transfer`` 或者更好的办法：
    使用接收者自己取回资金的模式。

* ``call``， ``delegatecall`` 和 ``staticcall``


为了与不符合 |ABI| 的合约交互，或者要更直接地控制编码，提供了函数 ``call``，``delegatecall`` 和 ``staticcall`` 。
它们都带有一个 ``bytes memory`` 参数和返回执行成功状态（``bool``）和数据（``bytes memory``）。

函数 ``abi.encode``，``abi.encodePacked``，``abi.encodeWithSelector`` 和 ``abi.encodeWithSignature`` 可用于编码结构化数据。

例如：

.. code-block:: solidity

    bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
    (bool success, bytes memory returnData) = address(nameReg).call(payload);
    require(success);



此外，为了与不符合 |ABI| 的合约交互，于是就有了可以接受任意类型任意数量参数的 ``call`` 函数。
这些参数会被打包到以 32 字节为单位的连续区域中存放。
其中一个例外是当第一个参数被编码成正好 4 个字节的情况。
在这种情况下，这个参数后边不会填充后续参数编码，以允许使用函数签名。

.. code-block:: solidity

    address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
    nameReg.call("register", "MyName");
    nameReg.call(bytes4(keccak256("fun(uint256)")), a);

.. warning::
    所有这些函数都是低级函数，应谨慎使用。
    具体来说，任何未知的合约都可能是恶意的，我们在调用一个合约的同时就将控制权交给了它，而合约又可以回调合约，所以要准备好在调用返回时改变相应的状态变量（可参考 :ref:`可重入<re_entance>` )，  与其他合约交互的常规方法是在合约对象上调用函数（x.f()）。


.. note::
    0.5.以前版本的 Solidity 允许这些函数接收任意参数，并且还会以不同方式处理 bytes4 类型的第一个参数。 在版本0.5.0中删除了这些边缘情况。

可以使用 ``gas`` |modifier| 调整提供的 gas 数量：

.. code-block:: solidity

    address(nameReg).call{gas: 1000000}(abi.encodeWithSignature("register(string)", "MyName"));

类似地，也能控制提供的 |ether| 的值：

.. code-block:: solidity

    address(nameReg).call{value: 1 ether}(abi.encodeWithSignature("register(string)", "MyName"));

最后一点，这些 |modifier| 可以联合使用。每个修改器出现的顺序不重要：

.. code-block:: solidity

    address(nameReg).call{gas: 1000000, value: 1 ether}(abi.encodeWithSignature("register(string)", "MyName"));

以类似的方式，可以使用函数 ``delegatecall`` ：区别在于只调用给定地址的代码（函数），其他状态属性如（存储，余额 ...）都来自当前合约。 ``delegatecall`` 的目的是使用另一个合约中的库代码。 用户必须确保两个合约中的存储结构都适合委托调用 （delegatecall）。


.. note::
    在以太坊家园（homestead） 之前，只有 ``callcode`` 函数，它无法访问原始的 ``msg.sender`` 和 ``msg.value`` 值。 此函数已在0.5.0版中删除。

从以太坊拜占庭（byzantium）版本开始 提供了 ``staticcall`` ，它与 ``call`` 基本相同，但如果被调用的函数以任何方式修改状态变量，都将回退。

所有三个函数 ``call`` ， ``delegatecall`` 和 ``staticcall`` 都是非常低级的函数，应该只把它们当作 *最后一招* 来使用，因为它们破坏了 Solidity 的类型安全性。

所有三种方法都提供 ``gas`` 选项，而 ``value`` 选项仅 ``call`` 支持 。

.. note::
    不管是读取状态还是写入状态，最好避免在合约代码中硬编码使用的 gas 值。这可能会引入”错误“，而且 gas 的消耗也是可能会改变的。

* ``code`` 和 ``codehash`` 成员

你可以查询任何智能合约的部署代码。使用 ``.code`` 来获取EVM的字节码，其返回 ``bytes memory`` ，值可能是空。
使用 ``.codehash`` 获得该代码的 Keccak-256哈希值 (为 ``bytes32`` )。注意， ``addr.codehash`` 比使用 ``keccak256(addr.code)`` 更便宜。


.. note::
    所有合约都可以转换为 ``address`` 类型，因此可以使用 ``address(this).balance`` 查询当前合约的余额。


.. index:: ! contract type, ! type; contract

.. _contract_types:

合约类型
--------------

每一个 :ref:`contract<contracts>` 定义都有他自己的类型。

您可以隐式地将合约转换为从他们继承的合约。
合约可以显式转换为 ``address`` 类型。

只有当合约具有 接收receive函数 或 payable 回退函数时，才能显式和 ``address payable`` 类型相互转换
转换仍然使用 ``address(x)`` 执行， 如果合约类型没有接收或payable 回退功能，则可以使用 ``payable(address(x))`` 转换为 ``address payable`` 。


可以参考 :ref:`地址类型<address>`.

.. note::
    在版本0.5.0之前，合约直接从地址类型派生的， 并且 ``address`` 和 ``address payable`` 之间没有区别。

如果声明一个合约类型的局部变量（ ``MyContract c`` ），则可以调用该合约的函数。 注意需要赋相同合约类型的值给它。

您还可以实例化合约（即新创建一个合约对象），参考 :ref:`'使用new创建合约'<creating-contracts>`。

合约和 ``address`` 的数据表示是相同的， 参考 :ref:`ABI<ABI>`。

合约不支持任何运算符。

合约类型的成员是合约的外部函数及 public 的 状态变量。


对于合约  ``C`` 可以使用 ``type(C)`` 获取合约的类型信息，参考 :ref:`类型信息<meta-type>` 。

.. index:: byte array, bytes32

定长字节数组
------------

关键字有：``bytes1``， ``bytes2``， ``bytes3``， ...， ``bytes32``。

运算符：

* 比较运算符： ``<=``， ``<``， ``==``， ``!=``， ``>=``， ``>`` （返回布尔型）
* 位运算符： ``&``， ``|``， ``^`` （按位异或）， ``~`` （按位取反）
* 移位运算符： ``<<`` （左移位）， ``>>`` （右移位）
* 索引访问：如果 ``x`` 是 ``bytesI`` 类型，那么 ``x[k]`` （其中 ``0 <= k < I``）返回第 ``k`` 个字节（只读）。

该类型可以和作为右操作数的无符号整数类型进行移位运算（但返回结果的类型和左操作数类型相同），右操作数表示需要移动的位数。
进行有符号整数位移运算会引发运行时异常。

成员变量：

* ``.length`` 表示这个字节数组的长度（只读）.

.. note::
    可以将 ``bytes1[]`` 当作字节数组使用，但由于填充规则，每个元素会浪费 31 字节（storage存储除外），因此更好地做法是使用 ``bytes``。

.. note::
    在 0.8.0 之前, ``byte`` 用作为 ``bytes1`` 的别名。

变长字节数组
------------

``bytes``:
    变长字节数组，参见 :ref:`arrays`。它并不是值类型。
``string``:
    变长 UTF-8 编码字符串类型，参见 :ref:`arrays`。并不是值类型。

.. index:: address, literal;address

.. _address_literals:

地址字面常量
---------------------------------------

比如像 ``0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF`` 这样的通过了地址校验和测试的十六进制字面常量会作为 ``address`` 类型。
而没有通过校验测试, 长度在 39 到 41 个数字之间的十六进制字面常量，会产生一个错误,您可以在零前面添加（对于整数类型）或在零后面添加（对于bytesNN类型）以消除错误。

.. note::
    混合大小写的地址校验和格式定义在 `EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_ 中。

.. index:: literal, literal;rational

.. _rational_literals:

有理数和整数字面常量
----------------------------

整数字面常量由范围在 0-9 的一串数字组成，表现成十进制。
例如， ``69`` 表示数字 69。
Solidity 中是没有八进制的，因此前置 0 是无效的。

十进制小数字面常量带有一个 ``.``，至少在其一边会有一个数字。
比如： ``1.``， ``.1``，和 ``1.3``。

``2e10`` 形式的科学符号也是支持的，尽管指数必须是整数，但底数可以是小数， ``MeE`` 的值 ``M * 10**E`` 。
比如：， ``-2e10``， ``2e-10``， ``2.5e1``。


为了提高可读性可以在数字之间加上下划线。
例如，十进制 ``123_000``，十六进制 ``0x2eff_abde``，科学十进制表示 1_2e345_678都是有效的。
下划线仅允许在两位数之间，并且不允许下划线连续出现。添加到数字文字中下划线没有额外的语义，下划线会被编译器忽略。

数值字面常量表达式本身支持任意精度，直到被转换成了非常量类型（例如，在常量变量表达式之外有运算，或发生了显示转换）。
这意味着在数值常量表达式中, 计算不会溢出而除法也不会截断。

例如， ``(2**800 + 1) - 2**800`` 的结果是字面常量 ``1`` （属于 ``uint8`` 类型），尽管计算的中间结果已经超过了 |evm| 的机器字长度。
此外， ``.5 * 8`` 的结果是整型 ``4`` （尽管有非整型参与了计算）。

.. warning::
    虽然大多数运算符在字面常量运算时都会产生一个字面常量表达式，但有一些运算符并不遵循这种模式：

    - 三元运算符 (``... ? ... : ...``),
    - 数组下标访问 (``<array>[<index>]``).

    你可能认为像 ``255 + (true ? 1 : 0)`` 或 ``255 + [1, 2, 3][0]`` 这样的表达式等同于直接使用 256 字面常量。
    但事实上，它们是在 ``uint8`` 类型中计算的，会溢出。

只要操作数是整型，任意整型支持的运算符都可以被运用在数值字面常量表达式中。
如果两个中的任一个数是小数，则不允许进行位运算。如果指数是小数的话，也不支持幂运算（因为这样可能会得到一个无理数）。

常量作为左（或基）操作数和整数类型的移位和幂运算时总是执行正确的（指数）操作，不管右（指数）操作数的类型如何。


.. note::
    Solidity 对每个有理数都有对应的数值字面常量类型。
    整数字面常量和有理数字面常量都属于数值字面常量类型。
    除此之外，所有的数值字面常量表达式（即只包含数值字面常量和运算符的表达式）都属于数值字面常量类型。
    因此数值字面常量表达式 ``1 + 2`` 和 ``2 + 1`` 的结果跟有理数三的数值字面常量类型相同。

.. warning::
    在早期版本中（0.4.0之前），整数字面常量的除法也会截断，但在现在的版本中，会将结果转换成一个有理数。即 ``5 / 2`` 并不等于 ``2``，而是等于 ``2.5``。

.. note::
    数值字面常量表达式只要在非字面常量表达式中使用就会转换成非字面常量类型。
    在下面的例子中，尽管我们知道 ``b`` 的值是一个整数，但 ``2.5 + a`` 这部分表达式并不进行类型检查，因此编译不能通过。

.. code-block:: solidity

    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5;

.. index:: literal, literal;string, string

.. _string_literals:

字符串字面常量及类型
---------------------

字符串字面常量是指由双引号或单引号引起来的字符串（ ``"foo"`` 或者 ``'bar'``）。
它们也可以分为多个连续的部分（ ``"foo" "bar"`` 等效于 ``"foobar"``），这在处理长字符串时很有用。
不像在 C 语言中那样带有结束符； ``"foo"`` 相当于 3 个字节而不是 4 个。
和整数字面常量一样，字符串字面常量的类型也可以发生改变，

但它们可以隐式地转换成 ``bytes1``，……， ``bytes32``，如果合适的话，还可以转换成 ``bytes`` 以及 ``string``。

例如： ``bytes32 samevar = "stringliteral"`` 字符串字面常量在赋值给 ``bytes32`` 时被解释为原始的字节形式。

字符串字面常量只能包含可打印的ASCII字符，这意味着他是介于 0x20 和 0x7E 之间的字符。

此外，字符串字面常量支持下面的转义字符：

- ``\<newline>`` (转义实际换行)
- ``\\`` (反斜杠)
- ``\'`` (单引号)
- ``\"`` (双引号)
- ``\b`` (退格)
- ``\f`` (换页)
- ``\n`` (换行符)
- ``\r`` (回车)
- ``\t`` (标签 tab)
- ``\v`` (垂直标签)
- ``\xNN`` (十六进制转义，见下文)
- ``\uNNNN`` (unicode 转义，见下文)


``\xNN`` 表示一个 16 进制值，最终转换成合适的字节，而 ``\uNNNN`` 表示 Unicode 编码值，最终会转换为 UTF-8 的序列。

.. note::

    在0.8.0版本之前，还有三个转义序列: ``\b``, ``\f`` 和 ``\v``。
    它们在其他语言中通常是可用的，但在实践中很少需要。
    如果您确实需要它们，它们仍然可以通过十六进制转义插入，即分别为 ``\x08``, ``\x0c`` 和 ``\x0b``，就像任何其他ASCII字符一样。

以下示例中的字符串长度为十个字节，它以换行符开头，后跟双引号，单引号，反斜杠字符，以及（没有分隔符）字符序列 ``abcdef`` 。

.. code-block:: solidity

    "\n\"\'\\abc\
    def"


任何unicode行终结符（即LF，VF，FF，CR，NEL，LS，PS）都不会被当成字符串字面常量的终止符。 如果前面没有前置 ``\``，则换行符仅终止字符串字面常量。

Unicode 字面常量
-----------------

常规字符串文字只能包含ASCII，而Unicode文字（以关键字unicode为前缀）可以包含任何有效的UTF-8序列。
它们还支持与转义序列完全相同的字符作为常规字符串文字。


.. code-block:: solidity

    string memory a = unicode"Hello 😃";

.. index:: literal, bytes

十六进制字面常量
---------------------

十六进制字面常量以关键字 ``hex`` 打头，后面紧跟着用单引号或双引号引起来的字符串（例如，``hex"001122FF"`` ）。
字符串的内容必须是一个十六进制的字符串，它们的值将使用二进制表示。

它们的内容必须是十六进制数字，可以选择使用单个下划线作为字节边界分隔符。 字面常量的值将是十六进制序列的二进制表示形式。

用空格分隔的多个十六进制字面常量被合并为一个字面常量：
``hex"00112233" hex"44556677"`` 等同于 ``hex"0011223344556677"``


十六进制字面常量跟 :ref:`字符串字面常量 <string_literals>` 很类似，具有相同的转换规则

.. index:: enum

.. _enums:

枚举类型
----------------

枚举是在Solidity中创建用户定义类型的一种方法。 它们是显示所有整型相互转换，但不允许隐式转换。 
从整型显式转换枚举，会在运行时检查整数时候在枚举范围内，否则会导致异常（ :ref:`Panic异常 <assert-and-require>` ）。
枚举需要至少一个成员,默认值是第一个成员，枚举不能多于 256 个成员。

数据表示与C中的枚举相同：选项从“0”开始的无符号整数值表示。


使用 ``type(NameOfEnum).min`` 和 ``type(NameOfEnum).max`` 你可以得到给定枚举的最小值和最大值。


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.8;

    contract test {
        enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
        ActionChoices choice;
        ActionChoices constant defaultChoice = ActionChoices.GoStraight;

        function setGoStraight() public {
            choice = ActionChoices.GoStraight;
        }

        // 由于枚举类型不属于 |ABI| 的一部分，因此对于所有来自 Solidity 外部的调用，
        // "getChoice" 的签名会自动被改成 "getChoice() returns (uint8)"。

        function getChoice() public view returns (ActionChoices) {
            return choice;
        }

        function getDefaultChoice() public pure returns (uint) {
            return uint(defaultChoice);
        }

        function getLargestValue() public pure returns (ActionChoices) {
            return type(ActionChoices).max;
        }

        function getSmallestValue() public pure returns (ActionChoices) {
            return type(ActionChoices).min;
        }
    }

.. note::

    枚举还可以在合约或库定义之外的文件级别上声明。

.. index:: ! user defined value type, custom type

.. _user-defined-value-types:

用户定义的值类型
------------------------

一个用户定义的值类型允许在一个基本的值类型上创建一个零成本的抽象。
这类似于一个别名，但有更严格的类型要求。


用户定义值类型使用 ``type C is V`` 来定义，其中 ``C`` 是新引入的类型的名称， ``V`` 必须是内置的值类型（"底层类型"）。
函数 ``C.wrap`` 被用来从底层类型转换到自定义类型。同样地，函数函数 ``C.unwrap`` 用于从自定义类型转换到底层类型。

类型 ``C`` 没有任何运算符或绑定成员函数。特别是，即使是操作符 ``==`` 也没有定义。也不允许与其他类型进行显式和隐式转换。

自定义类型的值的数据表示则继承自底层类型，并且ABI中也使用底层类型。

下面的例子说明了一个自定义类型 ``UFixed256x18``，代表了一个有18位小数的十进制定点类型，并有一个库来对该类型进行算术操作。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.8;

    // Represent a 18 decimal, 256 bit wide fixed point type using a user defined value type.
    type UFixed256x18 is uint256;

    /// A minimal library to do fixed point operations on UFixed256x18.
    library FixedMath {
        uint constant multiplier = 10**18;

        /// Adds two UFixed256x18 numbers. Reverts on overflow, relying on checked
        /// arithmetic on uint256.
        function add(UFixed256x18 a, UFixed256x18 b) internal pure returns (UFixed256x18) {
            return UFixed256x18.wrap(UFixed256x18.unwrap(a) + UFixed256x18.unwrap(b));
        }
        /// Multiplies UFixed256x18 and uint256. Reverts on overflow, relying on checked
        /// arithmetic on uint256.
        function mul(UFixed256x18 a, uint256 b) internal pure returns (UFixed256x18) {
            return UFixed256x18.wrap(UFixed256x18.unwrap(a) * b);
        }
        /// Take the floor of a UFixed256x18 number.
        /// @return the largest integer that does not exceed `a`.
        function floor(UFixed256x18 a) internal pure returns (uint256) {
            return UFixed256x18.unwrap(a) / multiplier;
        }
        /// Turns a uint256 into a UFixed256x18 of the same value.
        /// Reverts if the integer is too large.
        function toUFixed256x18(uint256 a) internal pure returns (UFixed256x18) {
            return UFixed256x18.wrap(a * multiplier);
        }
    }

注意 ``UFixed256x18.wrap`` 和 ``FixedMath.toUFixed256x18``的签名相同，但执行的是两个完全不同的操作：
``UFixed256x18.wrap`` 函数返回一个与输入的数据表示相同的 ``UFixed256x18``， 而 ``toUFixed256x18``则返回一个具有相同数值的 ``UFixed256x18`` 。


.. index:: ! function type, ! type; function

.. _function_types:

函数类型
----------------

函数类型是一种表示函数的类型。可以将一个函数赋值给另一个函数类型的变量，也可以将一个函数作为参数进行传递，还能在函数调用中返回函数类型变量。
函数类型有两类：
- *内部（internal）* 函数类型
- *外部（external）* 函数类型

内部函数只能在当前合约内被调用（更具体来说，在当前代码块内，包括内部库函数和继承的函数中），因为它们不能在当前合约上下文的外部被执行。
调用一个内部函数是通过跳转到它的入口标签来实现的，就像在当前合约的内部调用一个函数。

外部函数由一个地址和一个函数签名组成，可以通过外部函数调用传递或者返回。

函数类型表示成如下的形式：

.. code-block:: solidity

    function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]

与参数类型相反，返回类型不能为空 —— 如果函数类型不需要返回，则需要删除整个 ``returns (<return types>)`` 部分。

函数类型默认是内部函数，因此不需要声明 ``internal`` 关键字。
 请注意，这仅适用于函数类型，合约中定义的函数明确指定可见性，它们没有默认值。

类型转换：


函数类型 ``A`` 可以隐式转换为函数类型 ``B`` 当且仅当:
它们的参数类型相同，返回类型相同，它们的内部/外部属性是相同的，并且 ``A`` 的状态可变性比 ``B`` 的状态可变性更具限制性，比如：

- ``pure`` 函数可以转换为 ``view`` 和 ``non-payable`` 函数
- ``view`` 函数可以转换为 ``non-payable`` 函数
- ``payable`` 函数可以转换为 ``non-payable`` 函数

其他的转换则不可以。

关于 ``payable`` 和 ``non-payable`` 的规则可能有点令人困惑，但实质上，如果一个函数是 ``payable`` ，这意味着它
也接受零以太的支付，因此它也是 ``non-payable`` 。
另一方面，``non-payable`` 函数将拒绝发送给它的 |ether| ，
所以 ``non-payable`` 函数不能转换为 ``payable`` 函数。


如果当函数类型的变量还没有初始化时就调用它的话会引发一个 :ref:`Panic 异常<assert-and-require>`。
如果在一个函数被 ``delete`` 之后调用它也会发生相同的情况。

如果外部函数类型在 Solidity 的上下文环境以外的地方使用，它们会被视为 ``function`` 类型。
该类型将函数地址紧跟其函数标识一起编码为一个 ``bytes24`` 类型。。

请注意，当前合约的 public 函数既可以被当作内部函数也可以被当作外部函数使用。
如果想将一个函数当作内部函数使用，就用 ``f`` 调用，如果想将其当作外部函数，使用 ``this.f`` 。

一个内部函数可以被分配给一个内部函数类型的变量，无论定义在哪里，包括合约和库的私有、内部和public函数，以及自由函数。
另一方面，外部函数类型只与public和外部合约函数兼容。库是不可以的，因为库使用 ``delegatecall``，并且 :ref:`他们的函数选择器有不同的 ABI 转换 <library-selectors>` 。
接口中声明的函数没有定义，所以指向它们也没有意义。


成员方法：

public（或 external）函数都有下面的成员：

* ``.address`` 返回函数的合约地址。
* ``.selector`` 返回 :ref:`ABI 函数选择器 <abi_function_selector>`

.. note::
  public（或 external）函数过去有额外两个成员： ``.gas(uint)`` 和  ``.value(uint)``  在0.6.2中弃用了，在 0.8.0 中移除了。
  用 ``{gas: ...}`` 和  ``{value: ...}`` 代替， 参考 :ref:`外部函数调用 <external-function-calls>` 了解更多。


下面的例子，显示如何使用成员：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.4  <0.9.0;

    contract Example {
      function f() public payable returns (bytes4) {
        assert(this.f.address == address(this));
        return this.f.selector;
      }
      function g() public {
        this.f{gas: 10, value: 800}();
      }
    }

如果使用内部函数类型的例子：

.. code-block:: solidity

    pragma solidity >=0.4.16  <0.9.0;

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

另外一个使用外部函数类型的例子：

.. code-block:: solidity

    pragma solidity >=0.4.22  <0.9.0;

    contract Oracle {
      struct Request {
        bytes data;
        function(uint) external callback;
      }
      Request[] private requests;
      event NewRequest(uint);
      function query(bytes memory data, function(uint) external callback) public {
        requests.push(Request(data, callback));
        emit NewRequest(requests.length - 1);
      }
      function reply(uint requestID, uint response) public {
        // 这里检查回复来自可信来源
        requests[requestID].callback(response);
      }
    }

    contract OracleUser {
      Oracle constant private ORACLE_CONST = Oracle(address(0x00000000219ab540356cBB839Cbe05303d7705Fa)); // known contract
      uint private exchangeRate;
      function buySomething() public {
        ORACLE_CONST.query("USD", this.oracleResponse);
      }
      function oracleResponse(uint response) public {
        require(
            msg.sender == address(ORACLE_CONST),
            "Only oracle can call this."
        );
        exchangeRate = response;
      }
    }

.. note::
    Lambda 表达式或者内联函数的引入在计划内，但目前还没支持。
