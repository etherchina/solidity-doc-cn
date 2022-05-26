.. include:: glossaries.rst

**************************************
单位和全局变量
**************************************

.. index:: wei, finney, szabo, gwei ,ether

|ether| 单位
==============

|ether| 单位之间的换算就是在数字后边加上 ``wei`` ， ``gwei``  或 ``ether`` 来实现的，如果后面没有单位，缺省为 wei。



.. code-block:: Solidity

    assert(1 wei == 1);
    assert(1 gwei == 1e9);
    assert(1 ether == 1e18);

货币单位后缀的的效果相当于乘以10的幂。

.. note::
    从0.7.0开始 ``finney`` 和 ``szabo`` 被移除了。
    译者注：gwei 在solidity 0.6.11  中添加，因此在0.6.11之前的版本中不可用。

.. index:: time, seconds, minutes, hours, days, weeks, years

时间单位
==========

秒是缺省时间单位，在时间单位之间，数字后面带有 ``seconds``、 ``minutes``、 ``hours``、 ``days`` 和 ``weeks`` 的可以进行换算，基本换算关系如下：

* ``1 == 1 seconds``
* ``1 minutes == 60 seconds``
* ``1 hours == 60 minutes``
* ``1 days == 24 hours``
* ``1 weeks == 7 days``

由于闰秒造成的每年不都是 365 天、每天不都是 24 小时 `leap seconds <https://en.wikipedia.org/wiki/Leap_second>`_，所以如果你要使用这些单位计算日期和时间，请注意这个问题。因为闰秒是无法预测的，所以需要借助外部的预言机（oracle，是一种链外数据服务，译者注）来对一个确定的日期代码库进行时间矫正。

.. note::
    ``years`` 已经在 0.5.0 版本去除了，因为闰年的原因。

这些后缀不能直接用在变量后边。如果想用时间单位（例如 days）来将输入变量换算为时间，你可以用如下方式来完成:

.. code-block:: solidity

    function f(uint start, uint daysAfter) public {
        if (block.timestamp >= start + daysAfter * 1 days) {
            // ...
        }
    }


.. _special-variables-functions:

特殊变量和函数
===============================

在全局命名空间中已经存在了（预设了）一些特殊的变量和函数，他们主要用来提供关于区块链的信息或一些通用的工具函数。

.. note::
    译者注： 为了方便理解，可以把这些变量和函数理解为Solidity 语言层面的（原生） API 。


.. index:: abi, block, coinbase, difficulty, encode, number, block;number, timestamp, block;timestamp, msg, data, gas, sender, value, block.timestamp, gas price, origin


区块和交易属性
--------------------------------

- ``blockhash(uint blockNumber) returns (bytes32)``：指定区块的区块哈希 —— 仅可用于最新的 256 个区块且不包括当前区块，否则返回 0 。
- ``block.basefee`` (``uint``): 当前区块的基础费用，参考： (`EIP-3198 <https://eips.ethereum.org/EIPS/eip-3198>`_ 和 `EIP-1559 <https://eips.ethereum.org/EIPS/eip-1559>`_)
- ``block.chainid`` (``uint``): 当前链 id
- ``block.coinbase`` ( ``address`` ): 挖出当前区块的矿工地址
- ``block.difficulty`` ( ``uint`` ): 当前区块难度
- ``block.gaslimit`` ( ``uint`` ): 当前区块 gas 限额
- ``block.number`` ( ``uint`` ): 当前区块号
- ``block.timestamp`` ( ``uint``): 自 unix epoch 起始当前区块以秒计的时间戳
- ``gasleft() returns (uint256)`` ：剩余的 gas
- ``msg.data`` ( ``bytes`` ): 完整的 calldata
- ``msg.sender`` ( ``address`` ): 消息发送者（当前调用）
- ``msg.sig`` ( ``bytes4`` ): calldata 的前 4 字节（也就是函数标识符）
- ``msg.value`` ( ``uint`` ): 随消息发送的 wei 的数量
- ``tx.gasprice`` (``uint``): 交易的 gas 价格
- ``tx.origin`` ( ``address`` ): 交易发起者（完全的调用链）

.. note::
    对于每一个**外部函数**调用，包括 ``msg.sender`` 和 ``msg.value`` 在内所有 ``msg`` 成员的值都会变化。这里包括对库函数的调用。

.. note::
    当合约在链下被评估，而不是在一个区块所包含的交易的背景下被评估时，你不应该假定 `block.*` 和 `tx.*` 是指任何特定区块或交易。这些值是由执行合约的EVM实现提供的，可以是任意的。

.. note::
    不要依赖 ``block.timestamp`` 和 ``blockhash`` 产生随机数，除非你明确知道自己做的用意。

    时间戳和区块哈希在一定程度上都可能受到挖矿矿工影响。例如，挖矿社区中的恶意矿工可以用某个给定的哈希来运行赌场合约的 payout 函数，而如果他们没收到钱，还可以用一个不同的哈希重新尝试。

    当前区块的时间戳必须严格大于最后一个区块的时间戳，但这里能确保也需要它是在权威链上的两个连续区块。

.. note::
    基于可扩展因素，区块哈希不是对所有区块都有效。你仅仅可以访问最近 256 个区块的哈希，其余的哈希均为零。

.. note::
   ``blockhash`` 函数之前是使用 ``block.blockhash``， ``block.blockhash`` 在 0.4.22 开始不推荐使用，在 0.5.0 已经移除了。


.. note::
     ``gasleft`` 函数之前是使用 ``msg.gas``,  ``msg.gas`` 在 0.4.21 开始不推荐使用，在 0.5.0 已经移除了。

.. note::
    在 0.7.0,  ``now`` ( ``block.timestamp`` 的别名) 被移除了。

.. index:: abi, encoding, packed

ABI 编码及解码函数
----------------------

- ``abi.decode(bytes memory encodedData, (...)) returns (...)``: 对给定的数据进行ABI解码，而数据的类型在括号中第二个参数给出 。 例如: ``(uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))``
- ``abi.encode(...) returns (bytes)``： :ref:`ABI <ABI>` - 对给定参数进行编码
- ``abi.encodePacked(...) returns (bytes)``：对给定参数执行 :ref:`紧打包编码 <abi_packed_mode>` ，注意，可以不明确打包编码。
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes)``： :ref:`ABI <ABI>` - 对给定第二个开始的参数进行编码，并以给定的函数选择器作为起始的 4 字节数据一起返回
- ``abi.encodeWithSignature(string signature, ...) returns (bytes)``：等价于 ``abi.encodeWithSelector(bytes4(keccak256(signature), ...)``
- ``abi.encodeCall(function functionPointer, (...)) returns (bytes memory)``: 使用tuple类型参数ABI 编码调用 ``functionPointer`` 。执行完整的类型检查, 确保类型匹配函数签名。结果和 ``abi.encodeWithSelector(functionPointer.selector, (...))`` 一致。

.. note::
    这些编码函数可以用来构造函数调用数据，而不用实际进行调用。此外，``keccak256(abi.encodePacked(a, b))``  是一种计算结构化数据的哈希值（尽管我们也应该关注到：使用不同的函数参数类型也有可能会引起“哈希冲突” ）的方式，不推荐使用的 ``keccak256(a, b)`` 。


.. note::
    译者注：关于计算结构化数据的哈希，可以参考 `EIP712 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md>`_ ，这里也有一篇中文文章 `理解 EIP712 类型结构化数据 Hash 与签名 <https://learnblockchain.cn/2019/04/24/token-EIP712/>`_ 。

更多详情请参考 :ref:`ABI <ABI>` 和 :ref:`紧打包编码 <abi_packed_mode>`。

.. index:: bytes members

bytes 成员函数
----------------

- ``bytes.concat(...) returns (bytes memory)``: :ref:`Concatenates variable number of bytes and bytes1, ..., bytes32 arguments to one byte array<bytes-concat>`

.. index:: string members

string 成员函数
-----------------

- ``string.concat(...) returns (string memory)``: :ref:`Concatenates variable number of string arguments to one string array<string-concat>`


.. index:: assert, revert, require

错误处理
--------------

可以参阅专门的章节 :ref:`assert and require<assert-and-require>` 参阅有关错误处理以及何时使用哪个函数的更多详细信息。

``assert(bool condition)``
    如果不满足条件，则会导致Panic 错误，则撤销状态更改 -  用于检查内部错误。

``require(bool condition)``
    如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误。

``require(bool condition, string memory message)``
    如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误，可以同时提供一个错误消息。

``revert()``
    终止运行并撤销状态更改。

``revert(string memory reason)``
    终止运行并撤销状态更改，可以同时提供一个解释性的字符串。

.. index:: keccak256, ripemd160, sha256, ecrecover, addmod, mulmod, cryptography

.. _mathematical-and-cryptographic-functions:

数学和密码学函数
----------------------------------------

``addmod(uint x, uint y, uint k) returns (uint)``
    计算 ``(x + y) % k``，加法会在任意精度下执行，并且加法的结果即使超过 ``2**256`` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 ``k != 0`` 的校验（assert）。

``mulmod(uint x, uint y, uint k) returns (uint)``
    计算 ``(x * y) % k``，乘法会在任意精度下执行，并且乘法的结果即使超过 ``2**256`` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 ``k != 0`` 的校验（assert）。

``keccak256((bytes memory) returns (bytes32)``
    计算 Keccak-256 哈希。

.. note::

    之前 ``keccak256`` 的别名函数 ``sha3`` 在0.5.0中已经移除。

``sha256(bytes memory) returns (bytes32)``
    计算参数的 SHA-256 哈希。

``ripemd160(bytes memory) returns (bytes20)``
    计算参数的 RIPEMD-160 哈希。

``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``
    利用椭圆曲线签名恢复与公钥相关的地址，错误返回零值。

    函数参数对应于 ECDSA签名的值:

    * ``r`` = 签名的前 32 字节
    * ``s`` = 签名的第2个32 字节
    * ``v`` = 签名的最后一个字节

    ``ecrecover`` 返回一个 ``address``, 而不是 ``address payable`` 。他们之前的转换参考 :ref:`address payable<address>` ，如果需要转移资金到恢复的地址。

    可进一步参考 `使用案例 <https://ethereum.stackexchange.com/questions/1777/workflow-on-signing-a-string-with-private-key-followed-by-signature-verificatio>`_ 。


.. warning::

    如果你使用 ``ecrecover`` ，需要了解，在不需要知道相应的私钥下，签名也可以转换为另一个有效签名（可能是另外一个数据的签名）。在 Homestead 硬分叉，这个问题对于 _transaction_ 签名已经解决了(查阅 `EIP-2 <https://eips.ethereum.org/EIPS/eip-2#specification>`_)。
    不过 ``ecrecover`` 没有更改。

    除非需要签名是唯一的，否则这通常不是问题，或者是用它们来识别物品。 OpenZeppelin有一个 `ECDSA助手库 <https://docs.openzeppelin.com/contracts/2.x/api/cryptography#ECDSA>`_ ，可以将其用作 ``ecrecover`` 的”包装“，而不会出现此问题。

.. note::

    在一个私链上，你很有可能碰到由于 ``sha256``、``ripemd160`` 或者 ``ecrecover`` 引起的 Out-of-Gas。这个原因就是他们被当做所谓的预编译合约而执行，并且在第一次收到消息后这些合约才真正存在（尽管合约代码是硬代码）。发送到不存在的合约的消息非常昂贵，所以实际的执行会导致 Out-of-Gas 错误。在你的合约中实际使用它们之前，给每个合约发送一点儿以太币，比如 1 Wei。这在官方网络或测试网络上不是问题。

.. index:: balance, codehash, send, transfer, call, callcode, delegatecall
.. _address_related:

地址成员
---------------

``<address>.balance`` (``uint256``)
    以 Wei 为单位的 :ref:`address` 的余额。

``<address>.code`` (``bytes memory``)
    在 :ref:`address` 上的代码(可以为空)

``<address>.codehash`` (``bytes32``)
    :ref:`address` 的codehash

``<address payable>.transfer(uint256 amount)``
    向 :ref:`address` 发送数量为 amount 的 Wei，失败时抛出异常，使用固定（不可调节）的 2300 gas 的矿工费。

``<address payable>.send(uint256 amount) returns (bool)``
    向 :ref:`address` 发送数量为 amount 的 Wei，失败时返回 ``false``，发送 2300 gas 的矿工费用，不可调节。

``<address>.call(bytes memory) returns (bool, bytes memory)``
    用给定的有效载荷（payload）发出低级 ``CALL`` 调用，返回成功状态及返回数据，发送所有可用 gas，也可以调节 gas。

``<address>.delegatecall(bytes memory) returns (bool, bytes memory)``
    用给定的有效载荷 发出低级 ``DELEGATECALL`` 调用 ，返回成功状态并返回数据，发送所有可用 gas，也可以调节 gas。
    发出低级函数 ``DELEGATECALL``，失败时返回 ``false``，发送所有可用 gas，可调节。

``<address>.staticcall(bytes memory) returns (bool, bytes memory)``
    用给定的有效载荷 发出低级 ``STATICCALL`` 调用 ，返回成功状态并返回数据，发送所有可用 gas，也可以调节 gas。

更多信息，参考 :ref:`address` 部分：

.. warning::

    在执行另一个合约函数时，应该尽可能避免使用 ``.call()`` ，因为它绕过了类型检查，函数存在检查和参数打包。

.. warning::

    使用 ``send`` 有很多危险：如果调用栈深度已经达到 1024（这总是可以由调用者所强制指定），转账会失败；并且如果接收者用光了 gas，转账同样会失败。为了保证以太币转账安全，总是检查 ``send`` 的返回值，利用 ``transfer`` 或者下面更好的方式：
    用这种接收者取回钱的模式。

.. warning::
    由于 EVM 会把对一个不存在的合约的调用作为是成功的。
    Solidity 会在执行外部调用时使用 `extcodesize` 操作码进行额外检查。
    这确保了即将被调用的合约要么实际存在（它包含代码）或者触发一个异常。

    对地址而不是合约实例进行操作的低级调用(如 ``.call()`` , ``.delegatecall()`` , ``.staticcall()`` , ``.send()`` 和 ``.transfer()`` ) 时， **不** 包括这个检查，这使得它们在GAS方面更便宜，但也更不安全。
    
.. note::

   在版本0.5.0之前，Solidity允许通过合约实例来访问地址的成员，例如 ``this.balance`` ，不过现在禁止这样做，必须显式转换为地址后访问，如： ``address（this）.balance`` 。

.. note::
   如果在通过低级函数 delegatecall 发起调用时需要访问存储中的变量，那么这两个合约的存储布局需要一致，以便被调用的合约代码可以正确地通过变量名访问合约的存储变量。
   这不是指在库函数调用（高级的调用方式）时所传递的存储变量指针需要满足那样情况。

.. note::
    在 0.5.0 版本以前, ``.call``, ``.delegatecall`` and ``.staticcall`` 仅仅返回成功状态，没有返回值。

.. note::
    在 0.5.0 版本以前, 还有一个 ``callcode`` 函数，现在已经去除。

.. index:: this, selfdestruct

合约相关
----------------

``this`` (当前的合约类型)
    当前合约，可以显示转换为 :ref:`address`。

``selfdestruct(address payable recipient)``
    销毁合约，并把余额发送到指定 :ref:`address`。

  请注意， ``selfdestruct`` 具有从EVM继承的一些特性：

  - 接收合约的 receive 函数 不会执行。
  - 合约仅在交易结束时才真正被销毁，并且 ``revert``  可能会“撤消”销毁。

此外，当前合约内的所有函数都可以被直接调用，包括当前函数。


.. note::
    在 0.5.0 之前, 还有一个 ``suicide`` ，它和 ``selfdestruct`` 语义是一样的。

.. index:: type, creationCode, runtimeCode

.. _meta-type:

类型信息
-------------------

表达式 ``type(X)`` 可用于检索参数 ``X`` 的类型信息。 目前，此功能还比较有限( ``X`` 仅能是合约和整型)，但是未来应该会扩展。

用于合约类型 ``C`` 支持以下属性:

``type(C).name``:
    获得合约名

``type(C).creationCode``:
    获得包含创建合约字节码的内存字节数组。它可以在内联汇编中构建自定义创建例程，尤其是使用 ``create2`` 操作码。
    不能在合约本身或派生的合约访问此属性。 因为会引起循环引用。


``type(C).runtimeCode``
    获得合约的运行时字节码的内存字节数组。这是通常由 ``C`` 的构造函数部署的代码。
    如果 ``C`` 有一个使用内联汇编的构造函数，那么可能与实际部署的字节码不同。 还要注意库在部署时修改其运行时字节码以防范定期调用（guard against
    regular calls）。
    与 ``.creationCode`` 有相同的限制，不能在合约本身或派生的合约访问此属性。 因为会引起循环引用。


除上面的属性, 下面的属性在接口类型``I``下可使用:

``type(I).interfaceId``:
    返回接口``I`` 的 ``bytes4`` 类型的接口 ID，接口 ID 参考： `EIP-165 <https://learnblockchain.cn/docs/eips/eip-165.html>`_ 定义的，
    接口 ID 被定义为 ``XOR`` （异或） 接口内所有的函数的函数选择器（除继承的函数。

对于整型 ``T`` 有下面的属性可访问：


``type(T).min``
    ``T`` 的最小值。

``type(T).max``
    ``T`` 的最大值。
