.. include:: glossaries.rst

**************************************
单元和全局变量
**************************************

.. index:: wei, finney, szabo, ether

|ether| 单位
==============

|ether| 单位之间的换算就是在数字后边加上 ``wei``、 ``finney``、 ``szabo`` 或 ``ether`` 来实现的，如果后面没有单位，缺省为 Wei。例如 ``2 ether == 2000 finney`` 的逻辑判断值为 ``true``。

.. index:: time, seconds, minutes, hours, days, weeks, years

时间单位
==========

秒是缺省时间单位，在时间单位之间，数字后面带有 ``seconds``、 ``minutes``、 ``hours``、 ``days``、 ``weeks`` 和 ``years`` 的可以进行换算，基本换算关系如下：

 * ``1 == 1 seconds``
 * ``1 minutes == 60 seconds``
 * ``1 hours == 60 minutes``
 * ``1 days == 24 hours``
 * ``1 weeks == 7 days``
 * ``1 years == 365 days``

由于闰秒造成的每年不都是 365 天、每天不都是 24 小时 `leap seconds <https://en.wikipedia.org/wiki/Leap_second>`_，所以如果你要使用这些单位计算日期和时间，请注意这个问题。因为闰秒是无法预测的，所以需要借助外部的预言机（oracle，是一种链外数据服务，译者注）来对一个确定的日期代码库进行时间矫正。

.. note::
    ``years`` 后缀已经不推荐使用了，因为从 0.5.0 版本开始将不再支持。

这些后缀不能直接用在变量后边。如果想用时间单位（例如 days）来将输入变量换算为时间，你可以用如下方式来完成：

::

    function f(uint start, uint daysAfter) public {
        if (now >= start + daysAfter * 1 days) {
            // ...
        }
    }

特殊变量和函数
===============================

在全局命名空间中已经存在了（预设了）一些特殊的变量和函数，他们主要用来提供关于区块链的信息或一些通用的工具函数。

.. index:: abi, block, coinbase, difficulty, encode, number, block;number, timestamp, block;timestamp, msg, data, gas, sender, value, now, gas price, origin


区块和交易属性
--------------------------------

- ``block.blockhash(uint blockNumber) returns (bytes32)``：指定区块的区块哈希——仅可用于最新的 256 个区块且不包括当前区块；而 blocks 从 0.4.22 版本开始已经不推荐使用，由 ``blockhash(uint blockNumber)`` 代替
- ``block.coinbase`` (``address``): 挖出当前区块的矿工地址
- ``block.difficulty`` (``uint``): 当前区块难度
- ``block.gaslimit`` (``uint``): 当前区块 gas 限额
- ``block.number`` (``uint``): 当前区块号
- ``block.timestamp`` (``uint``): 自 unix epoch 起始当前区块以秒计的时间戳
- ``gasleft() returns (uint256)``：剩余的 gas
- ``msg.data`` (``bytes``): 完整的 calldata
- ``msg.gas`` (``uint``): 剩余 gas - 自 0.4.21 版本开始已经不推荐使用，由 ``gesleft()`` 代替
- ``msg.sender`` (``address``): 消息发送者（当前调用）
- ``msg.sig`` (``bytes4``): calldata 的前 4 字节（也就是函数标识符）
- ``msg.value`` (``uint``): 随消息发送的 wei 的数量
- ``now`` (``uint``): 目前区块时间戳（``block.timestamp``）
- ``tx.gasprice`` (``uint``): 交易的 gas 价格
- ``tx.origin`` (``address``): 交易发起者（完全的调用链）

.. note::
    对于每一个**外部函数**调用，包括 ``msg.sender`` 和 ``msg.value`` 在内所有 ``msg`` 成员的值都会变化。这里包括对库函数的调用。

.. note::
    不要依赖 ``block.timestamp``、 ``now`` 和 ``blockhash`` 产生随机数，除非你知道自己在做什么。

    时间戳和区块哈希在一定程度上都可能受到挖矿矿工影响。例如，挖矿社区中的恶意矿工可以用某个给定的哈希来运行赌场合约的 payout 函数，而如果他们没收到钱，还可以用一个不同的哈希重新尝试。

    当前区块的时间戳必须严格大于最后一个区块的时间戳，但这里唯一能确保的只是它会是在权威链上的两个连续区块的时间戳之间的数值。
    
.. note::
    基于可扩展因素，区块哈希不是对所有区块都有效。你仅仅可以访问最近 256 个区块的哈希，其余的哈希均为零。

.. index:: abi, encoding, packed

ABI 编码函数
----------------------

- ``abi.encode(...) returns (bytes)``： :ref:`ABI <ABI>` - 对给定参数进行编码
- ``abi.encodePacked(...) returns (bytes)``：对给定参数执行 :ref:`紧打包编码 <abi_packed_mode>`
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes)``： :ref:`ABI <ABI>` - 对给定参数进行编码，并以给定的函数选择器作为起始的 4 字节数据一起返回
- ``abi.encodeWithSignature(string signature, ...) returns (bytes)``：等价于 ``abi.encodeWithSelector(bytes4(keccak256(signature), ...)``

.. note::
    这些编码函数可以用来构造函数调用数据，而不用实际进行调用。此外，``keccak256(abi.encodePacked(a, b))`` 是更准确的方法来计算在未来版本不推荐使用的 ``keccak256(a, b)``。

更多详情请参考 :ref:`ABI <ABI>` 和 :ref:`紧打包编码 <abi_packed_mode>`。

.. index:: assert, revert, require

错误处理
--------------

``assert(bool condition)``:
    如果条件不满足，则使当前交易没有效果 — 用于检查内部错误。
``require(bool condition)``:
    如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误。 
``require(bool condition, string message)``:
    如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误，可以同时提供一个错误消息。
``revert()``:
    终止运行并撤销状态更改。
``revert(string reason)``:
    终止运行并撤销状态更改，可以同时提供一个解释性的字符串。

.. index:: keccak256, ripemd160, sha256, ecrecover, addmod, mulmod, cryptography,

数学和密码学函数
----------------------------------------

``addmod(uint x, uint y, uint k) returns (uint)``:
    计算 ``(x + y) % k``，加法会在任意精度下执行，并且加法的结果即使超过 ``2**256`` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 ``k != 0`` 的校验（assert）。
``mulmod(uint x, uint y, uint k) returns (uint)``:
    计算 ``(x * y) % k``，乘法会在任意精度下执行，并且乘法的结果即使超过 ``2**256`` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 ``k != 0`` 的校验（assert）。
``keccak256(...) returns (bytes32)``:
    计算 :ref:`(tightly packed) arguments <abi_packed_mode>` 的 Ethereum-SHA-3 （Keccak-256）哈希。
``sha256(...) returns (bytes32)``:
    计算 :ref:`(tightly packed) arguments <abi_packed_mode>` 的 SHA-256 哈希。
``sha3(...) returns (bytes32)``:
     等价于 keccak256。
``ripemd160(...) returns (bytes20)``:
    计算 :ref:`(tightly packed) arguments <abi_packed_mode>` 的 RIPEMD-160 哈希。
``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)`` ：
    利用椭圆曲线签名恢复与公钥相关的地址，错误返回零值。
    (`example usage <https://ethereum.stackexchange.com/q/1777/222>`_)

上文中的“tightly packed”是指不会对参数值进行 padding 处理（就是说所有参数值的字节码是连续存放的，译者注），这意味着下边这些调用都是等价的：

    keccak256("ab", "c")
    keccak256("abc")
    keccak256(0x616263)
    keccak256(6382179)
    keccak256(97, 98, 99)

如果需要 padding，可以使用显式类型转换：``keccak256("\x00\x12")`` 和 ``keccak256(uint16(0x12))`` 是一样的。

请注意，常量值会使用存储它们所需要的最少字节数进行打包。例如：``keccak256(0) == keccak256(uint8(0))``，``keccak256(0x12345678) == keccak256(uint32(0x12345678))``。

在一个私链上，你很有可能碰到由于 ``sha256``、``ripemd160`` 或者 ``ecrecover`` 引起的 Out-of-Gas。原因是因为这些密码学函数在以太坊虚拟机(EVM)中以“预编译合约”形式存在的，且在第一次收到消息后才被真正存在（尽管合约代码是EVM中已存在的硬编码）。因此发送到不存在的合约的消息非常昂贵，所以实际的执行会导致 Out-of-Gas 错误。在你实际使用你的合约之前，给每个合约发送一点儿以太币，比如 1 Wei。这在官方网络或测试网络上不是问题。

.. index:: balance, send, transfer, call, callcode, delegatecall
.. _address_related:

地址相关
---------------

``<address>.balance`` (``uint256``):
    以 Wei 为单位的 :ref:`address` 的余额。
``<address>.transfer(uint256 amount)``:
    向 :ref:`address` 发送数量为 amount 的 Wei，失败时抛出异常，发送 2300 gas 的矿工费，不可调节。
``<address>.send(uint256 amount) returns (bool)``:
    向 :ref:`address` 发送数量为 amount 的 Wei，失败时返回 ``false``，发送 2300 gas 的矿工费用，不可调节。
``<address>.call(...) returns (bool)``:
    发出低级函数 ``CALL``，失败时返回 ``false``，发送所有可用 gas，可调节。
``<address>.callcode(...) returns (bool)``：
    发出低级函数 ``CALLCODE``，失败时返回 ``false``，发送所有可用 gas，可调节。
``<address>.delegatecall(...) returns (bool)``:
    发出低级函数 ``DELEGATECALL``，失败时返回 ``false``，发送所有可用 gas，可调节。

更多信息，参考 :ref:`address` 部分：

.. warning::
    使用 send 有很多危险：如果调用栈深度已经达到 1024（这总是可以由调用者所强制指定），转账会失败；并且如果接收者用光了 gas，转账同样会失败。为了保证以太币转账安全，总是检查 ``send`` 的返回值，利用 ``transfer`` 或者下面更好的方式：
    用这种接收者取回钱的模式。

.. note::
   如果在通过低级函数 delegatecall 发起调用时需要访问存储中的变量，那么这两个合约的存储中的变量定义顺序需要一致，以便被调用的合约代码可以正确地通过变量名访问合约的存储变量。
   这当然不是指像在高级的库函数调用时所传递的存储变量指针那样的情况。

.. note::
    不鼓励使用 ``callcode``，并且将来它会被移除。

.. index:: this, selfdestruct

合约相关
----------------

``this`` (current contract's type):
    当前合约，可以明确转换为 :ref:`address`。

``selfdestruct(address recipient)``:
    销毁合约，并把余额发送到指定 :ref:`address`。

``suicide(address recipient)``:
    与 selfdestruct 等价，但已不推荐使用。

此外，当前合约内的所有函数都可以被直接调用，包括当前函数。
