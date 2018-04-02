**************************************
单位和全局变量
**************************************

.. index:: wei, finney, szabo, ether


.. 索引:: wei, finney, szabo, ether

Ether Units
===========

以太币单位

A literal number can take a suffix of ``wei``, ``finney``, ``szabo`` or ``ether`` to convert between the subdenominations of Ether, where Ether currency numbers without a postfix are assumed to be Wei, e.g. ``2 ether == 2000 finney`` evaluates to ``true``.

以太币单位之间的换算就是数字后面可以跟着”wei”、“finney”、“szabo”、“ether”，如果没有缺省为Wei，例如“2 ether==2000 finney”的逻辑判断值为“true”。

.. index:: time, seconds, minutes, hours, days, weeks, years


.. 索引：： time, seconds, minutes, hours, days, weeks, years

Time Units
==========


时间单位

Suffixes like ``seconds``, ``minutes``, ``hours``, ``days``, ``weeks`` and
``years`` after literal numbers can be used to convert between units of time where seconds are the base
unit and units are considered naively in the following way:


秒是最小的时间单位，在时间单位之间，数字后面带有“seconds”、“mimutes”、“hours”、“days”、“weeks”和“years”的可以进行换算，基本换算关系如下：

 * ``1 == 1 seconds``
 * ``1 minutes == 60 seconds``
 * ``1 hours == 60 minutes``
 * ``1 days == 24 hours``
 * ``1 weeks == 7 days``
 * ``1 years == 365 days``

Take care if you perform calendar calculations using these units, because
not every year equals 365 days and not even every day has 24 hours
because of `leap seconds <https://en.wikipedia.org/wiki/Leap_second>`_.
Due to the fact that leap seconds cannot be predicted, an exact calendar
library has to be updated by an external oracle.


由于闰秒造成的每年不都是365天、每天不都是24小时，如果你要利用这些单位进行时间计算时，请注意这些情况。<https://en.wikipedia.org/wiki/Leap_second>`_.基于闰秒现象不能预测的事实，必须根据外部数据日期库进行准确修改。

These suffixes cannot be applied to variables. If you want to
interpret some input variable in e.g. days, you can do it in the following way::


这些单位名称不能用于变量中。如果想解释某些输入变量比如：天数，你可以参照一下形式：

    function f(uint start, uint daysAfter) public {
        if (now >= start + daysAfter * 1 days) {
          // ...
        }
    }

Special Variables and Functions
===============================


特殊变量和函数

There are special variables and functions which always exist in the global
namespace and are mainly used to provide information about the blockchain.


有一些常常在全局命名空间里的特殊变量和函数，主要用来提供区块链信息。

.. index:: block, coinbase, difficulty, number, block;number, timestamp, block;timestamp, msg, data, gas, sender, value, now, gas price, origin


.. 索引：：block、coinbase、difficulty、number、block；number、 timestamp、 block；timestamp、msg、data、gas、sender、value、now、gas price、origin

Block and Transaction Properties
--------------------------------


区块和交易属性

- ``block.blockhash(uint blockNumber) returns (bytes32)``: hash of the given block - only works for 256 most recent blocks excluding current
- ``block.blockhash(uintblockNumber) returns (bytes32)``:给定区块的哈希-仅对256个最近区块有效，不包括当前区块
- ``block.coinbase`` (``address``): current block miner's address
- ``block.coinbase`` (``address``): 当前区块挖矿矿工地址
- ``block.difficulty`` (``uint``): current block difficulty
- ``block.difficulty`` (``uint``): 当前区块难度
- ``block.gaslimit`` (``uint``): current block gaslimit
- ``block.gaslimit`` (``uint``): 当前区块燃料限额
- ``block.number`` (``uint``): current block number
- ``block.number`` (``uint``): 当前区块号
- ``block.timestamp`` (``uint``): current block timestamp as seconds since unix epoch
- ``block.timestamp`` (``uint``): 自unixepoch起始当前区块以秒计的时间戳
- ``msg.data`` (``bytes``): complete calldata
- ``msg.data`` (``bytes``): 完整调用数据
- ``msg.gas`` (``uint``): remaining gas
- ``msg.gas`` (``uint``): 剩余燃料
- ``msg.sender`` (``address``): sender of the message (current call)
- ``msg.sender`` (``address``): 发送信息者（当前调用）
- ``msg.sig`` (``bytes4``): first four bytes of the calldata (i.e. function identifier)
- ``msg.sig`` (``bytes4``): 调用数据前四个字节（函数识别符）
- ``msg.value`` (``uint``): number of wei sent with the message
- ``msg.value`` (``uint``): 随信息发送的wei数量
- ``now`` (``uint``): current block timestamp (alias for ``block.timestamp``)
- ``now`` (``uint``): 目前区块时间戳（“block.timestamp”别名）
- ``tx.gasprice`` (``uint``): gas price of the transaction
- ``tx.gasprice`` (``uint``): 交易燃料价格
- ``tx.origin`` (``address``): sender of the transaction (full call chain)
- ``tx.origin`` (``address``): 交易发起者（完全调用链）

.. note::
    The values of all members of ``msg``, including ``msg.sender`` and
    ``msg.value`` can change for every **external** function call.
    This includes calls to library functions.
.. 注解::
    包括“msg.sender”在内所有成员“msg”值，对于每一个外部函数调用“msg.value”应该有所变化。这里包括库函数调用。

.. note::
    Do not rely on ``block.timestamp``, ``now`` and ``block.blockhash`` as a source of randomness,
    unless you know what you are doing.
.. 注解::
   除非知道要做什么，你不要信赖“block.timestamp”、“now”和“block.blockhash”作为随机数的来源。

    Both the timestamp and the block hash can be influenced by miners to some degree.
    Bad actors in the mining community can for example run a casino payout function on a chosen hash
    and just retry a different hash if they did not receive any money.
    时间戳和区块哈希在一定程度上都可能收到挖矿矿工影响。举例来说，在矿区坏人可能在某个哈希上运行一个赌场支出函数，如果没有收到钱再用另一个不同的哈希。

    The current block timestamp must be strictly larger than the timestamp of the last block,
    but the only guarantee is that it will be somewhere between the timestamps of two
    consecutive blocks in the canonical chain.
    当前区块时间戳要严格大于最后一个区块的时间戳，但是它一定是在权威链上两个连续块时间戳之间的某处，这是确定的。

.. note::
    The block hashes are not available for all blocks for scalability reasons.
    You can only access the hashes of the most recent 256 blocks, all other
    values will be zero.
.. 注解::
   基于可扩展因素，区块哈希不是对所有区块都有效。你仅仅可以访问最近256个区块的哈希，其余的哈希均为零。

.. index:: assert, revert, require
.. 索引:: assert、revert、require

Error Handling
--------------

错误处理

``assert(bool condition)``:
    throws if the condition is not met - to be used for internal errors.
``assert(bool condition)``:
    如果条件不满足就抛掉-用于内部错误。
``require(bool condition)``:
    throws if the condition is not met - to be used for errors in inputs or external components.
``require(bool condition)``:
    如果条件不满足就抛掉-用于输入或者外部组件引起的错误。
``revert()``:
    abort execution and revert state changes
``revert()``:
    退出执行并且恢复到状态改变前。

.. index:: keccak256, ripemd160, sha256, ecrecover, addmod, mulmod, cryptography,
.. 索引:: keccak256、ripemd160、sha256、ecrecover、addmod、mulmod、cryptography 

Mathematical and Cryptographic Functions
----------------------------------------
数学和密码学函数

``addmod(uint x, uint y, uint k) returns (uint)``:
    compute ``(x + y) % k`` where the addition is performed with arbitrary precision and does not wrap around at ``2**256``. Assert that ``k != 0`` starting from version 0.5.0.
``addmod(uint x, uint y, uint k) returns (uint)``:
    计算“（x+y）%k”,这是在任何精度下执行加法，且不包在“2**256”。声明在0.5.0版本“k != 0”。
``mulmod(uint x, uint y, uint k) returns (uint)``:
    compute ``(x * y) % k`` where the multiplication is performed with arbitrary precision and does not wrap around at ``2**256``. Assert that ``k != 0`` starting from version 0.5.0.
``mulmod(uint x, uint y, uint k) returns (uint)``:
    计算“（x*y）%k”,这是在任何精度下执行乘法，且不包在“2**256”。声明在0.5.0版本“k != 0”。
``keccak256(...) returns (bytes32)``:
    compute the Ethereum-SHA-3 (Keccak-256) hash of the :ref:`(tightly packed) arguments <abi_packed_mode>`
``keccak256(...) returns (bytes32)``:
    计算ref: “(tightly packed) arguments <abi_packed_mode>”的Ethereum-SHA-3哈希。
``sha256(...) returns (bytes32)``:
    compute the SHA-256 hash of the :ref:`(tightly packed) arguments <abi_packed_mode>`
``sha256(...) returns (bytes32)``:
    计算ref: “(tightly packed) arguments <abi_packed_mode>”的SHA-256哈希。
``sha3(...) returns (bytes32)``:
    alias to ``keccak256``
``sha3(...) returns (bytes32)``:
    “keccak256”的别名。
``ripemd160(...) returns (bytes20)``:
    compute RIPEMD-160 hash of the :ref:`(tightly packed) arguments <abi_packed_mode>`
``ripemd160(...) returns (bytes20)``:
    计算ref: “(tightly packed) arguments <abi_packed_mode>”的RIPEMD-160哈希。
``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``:
    recover the address associated with the public key from elliptic curve signature or return zero on error
    (`example usage <https://ethereum.stackexchange.com/q/1777/222>`_)
``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``:
    利用椭圆曲线签名恢复与公钥相关的地址，错误返回零值。（‘应用实例’<https://ethereum.stackexchange.com/q/1777/222>`_ ）
In the above, "tightly packed" means that the arguments are concatenated without padding.
This means that the following are all identical::
上面的“tightly packed”实参没有任何空格连在一起。这意味着下面几种情况是等价的：

    keccak256("ab", "c")
    keccak256("abc")
    keccak256(0x616263)
    keccak256(6382179)
    keccak256(97, 98, 99)

If padding is needed, explicit type conversions can be used: ``keccak256("\x00\x12")`` is the
same as ``keccak256(uint16(0x12))``.
如果需要有空格，可以使用明显类型转换：“keccak256("\x00\x12")”和“keccak256(uint16(0x12))”是一样的。

Note that constants will be packed using the minimum number of bytes required to store them.
This means that, for example, ``keccak256(0) == keccak256(uint8(0))`` and
``keccak256(0x12345678) == keccak256(uint32(0x12345678))``.
注意：用需要存储他们的最少字节进行对常量进行打包。这就意味着，例如：“keccak256(0) == keccak256(uint8(0))”，“keccak256(0x12345678) == keccak256(uint32(0x12345678))”。

It might be that you run into Out-of-Gas for ``sha256``, ``ripemd160`` or ``ecrecover`` on a *private blockchain*. The reason for this is that those are implemented as so-called precompiled contracts and these contracts only really exist after they received the first message (although their contract code is hardcoded). Messages to non-existing contracts are more expensive and thus the execution runs into an Out-of-Gas error. A workaround for this problem is to first send e.g. 1 Wei to each of the contracts before you use them in your actual contracts. This is not an issue on the official or test net.
在一个私链上，你很有可能碰到由于“sha256”、“ripemd160”或者“ecrecover”引起的燃料耗尽这种情况。这个原因就是他们被当做所谓的预编译合约而执行，还有在第一次收到信息后（尽管合约代码是硬代码）这些合约才真正存在。对于不存在合约的信息花费很贵，因此执行碰到燃料耗尽错误。对于此类问题的变通方法就是，在你实际合约中使用时，给每一个合约发送比如1Wei的费用。这在官网或测试网上没有说明。


.. index:: balance, send, transfer, call, callcode, delegatecall
.. _address_related:
.. 索引:: balance, send, transfer, call, callcode, delegatecall
.. 相关地址:


Address Related
---------------
地址相关事项

``<address>.balance`` (``uint256``):
    balance of the :ref:`address` in Wei
``<address>.balance`` (``uint256``):
    以Wei计的ref:“address”的余额。
``<address>.transfer(uint256 amount)``:
    send given amount of Wei to :ref:`address`, throws on failure, forwards 2300 gas stipend, not adjustable
``<address>.transfer(uint256 amount)``:
    把以Wei计、给定数量的费用发送给ref:“address”，失败时抛掉，奉送2300燃料的薪酬，不公平。
``<address>.send(uint256 amount) returns (bool)``:
    send given amount of Wei to :ref:`address`, returns ``false`` on failure, forwards 2300 gas stipend, not adjustable
``<address>.send(uint256 amount) returns (bool)``:
    把以Wei计、给定数量的费用发送给ref:“address”，失败时返回“false”，奉送2300燃料的薪酬，不公平。
``<address>.call(...) returns (bool)``:
    issue low-level ``CALL``, returns ``false`` on failure, forwards all available gas, adjustable
``<address>.call(...) returns (bool)``:
    声明低级“调用”，失败时返回“false”， 奉送所有可用燃料，不公平。
``<address>.callcode(...) returns (bool)``:
    issue low-level ``CALLCODE``, returns ``false`` on failure, forwards all available gas, adjustable
``<address>.callcode(...) returns (bool)``:
    声明低级“调用码”，失败时返回“false”， 奉送所有可用燃料，不公平。
``<address>.delegatecall(...) returns (bool)``:
    issue low-level ``DELEGATECALL``, returns ``false`` on failure, forwards all available gas, adjustable
``<address>.delegatecall(...) returns (bool)``:
    声明低级“代表调用”，失败时返回“false”， 奉送所有可用燃料，不公平。
For more information, see the section on :ref:`address`.
更多信息请参考ref:“address”部分.

.. warning::
    There are some dangers in using ``send``: The transfer fails if the call stack depth is at 1024
    (this can always be forced by the caller) and it also fails if the recipient runs out of gas. So in order
    to make safe Ether transfers, always check the return value of ``send``, use ``transfer`` or even better:
    Use a pattern where the recipient withdraws the money.
.. 警告::
    有很多使用“send”的危险情况：如果调用叠加深度在1024（这总是可能调用者强制的），这导致转账失败，还有如果接受者花完燃料这也可能失败。为了保证以太币转账安全，总是检查“send”的返回值，利用“transfer”或者下面更好的方式：用这种接受者取回钱的模式。

.. note::
    The use of ``callcode`` is discouraged and will be removed in the future.
.. 注意:: 
    “callcode”的功用失效且将被删除。

.. index:: this, selfdestruct
.. 索引:: this, selfdestruct

Contract Related
----------------
合约相关事项

``this`` (current contract's type):
    the current contract, explicitly convertible to :ref:`address`
``this`` (current contract's type):
    当前合约，可以明确转换为：ref:“address”。

``selfdestruct(address recipient)``:
    destroy the current contract, sending its funds to the given :ref:`address`
``selfdestruct(address recipient)``:
    撕毁当前合约，把钱寄到给定地址：ref:“address”。

``suicide(address recipient)``:
    alias to ``selfdestruct``
``suicide(address recipient)``:
    “selfdestruct”的别名。

Furthermore, all functions of the current contract are callable directly including the current function.
此外，可以调用当前合约内的所有函数，直接包括当前函数。

