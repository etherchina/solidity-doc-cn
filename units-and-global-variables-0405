**************************************
单元和全局变量
**************************************

.. index:: wei, finney, szabo, ether

以太币单位
===========

以太币单位之间的换算就是数字后面跟着 ``wei`` , ``finney`` , ``szabo`` 或 ``ether`` 来实现的，如果后面没有单位，缺省为Wei。例如 ``2 ether == 2000 finney`` 的逻辑判断值为 ``true`` 。

.. index:: time, seconds, minutes, hours, days, weeks, years

时间单位
==========

秒是最小的时间单位，在时间单位之间，数字后面带有 ``seconds`` ， ``minutes`` ， ``hours`` ， ``days`` ， ``weeks`` 和 ``years`` 的可以进行换算，基本换算关系如下：

 * ``1 == 1 seconds``
 * ``1 minutes == 60 seconds``
 * ``1 hours == 60 minutes``
 * ``1 days == 24 hours``
 * ``1 weeks == 7 days``
 * ``1 years == 365 days``

由于闰秒造成的每年不都是365天、每天不都是24小时`leap seconds <https://en.wikipedia.org/wiki/Leap_second>`_，如果你要利用这些单位进行时间计算时，请注意。基于闰秒不能预测的事实，必须根据外部已知数据对日期库进行准确更正。

这些单位名称不能用在变量中。如果想解释像天数的某些输入变量，你可以参照以下形式::

    function f(uint start, uint daysAfter) public {
        if (now >= start + daysAfter * 1 days) {
          // ...
        }
    }

特殊变量和函数
===============================

有一些常常在全局命名空间里的特殊变量和函数，主要用来提供区块链信息。

.. index:: block, coinbase, difficulty, number, block;number, timestamp, block;timestamp, msg, data, gas, sender, value, now, gas price, origin


区块和交易说明
--------------------------------

- ``block.blockhash(uint blockNumber) returns (bytes32)``: 给定区块的哈希-仅对256个最近区块有效，不包括当前区块
- ``block.coinbase`` (``address``):当前区块挖矿矿工地址
- ``block.difficulty`` (``uint``):当前区块难度
- ``block.gaslimit`` (``uint``):当前区块gas限额
- ``block.number`` (``uint``):当前区块号
- ``block.timestamp`` (``uint``):自unix epoch起始当前区块以秒计的时间戳
- ``msg.data`` (``bytes``):完整调用数据
- ``msg.gas`` (``uint``): 剩余gas
- ``msg.sender`` (``address``):信息发送者（当前调用）
- ``msg.sig`` (``bytes4``):调用数据的前四个字节（函数识别符）
- ``msg.value`` (``uint``):随信息发送的wei数量
- ``now`` (``uint``):目前区块时间戳（ `` block.timestamp`` ）
- ``tx.gasprice`` (``uint``):交易gas价格
- ``tx.origin`` (``address``):交易发起者（完全调用链）

.. note::
    对于每一个外部函数调用，包括 ``msg.sender`` 和 ``msg.value``  在内所有 ``msg`` 成员的值都有可能变化。这里包括对库函数的调用。

.. note::
    除非知道要做什么，你不要把 ``block.timestamp`` 、 ``now`` 和 ``block.blockhash`` 作为随机数的来源。

    时间戳和区块哈希在一定程度上都可能收到挖矿矿工影响。举例来说，在矿区恶意使坏者可能在某个哈希上运行一个赌场支出函数，如果没有收到钱再用另一个不同的哈希。

    当前区块的时间戳要严格大于最后一个区块的时间戳，但是它一定是在权威链上两个连续块时间戳之间的某处，这是确定的。

.. note::
    基于可扩展因素，区块哈希不是对所有区块都有效。你仅仅可以访问最近256个区块的哈希，其余的哈希均为零。

.. index:: assert, revert, require

错误处理
--------------

``assert(bool condition)``:
    如果条件不满足就抛掉-用于内部错误。
``require(bool condition)``:
    如果条件不满足就抛掉-用于输入或者外部组件引起的错误。
``revert()``:
    退出执行并且恢复到状态改变前。

.. index:: keccak256, ripemd160, sha256, ecrecover, addmod, mulmod, cryptography,

数学和密码函数
----------------------------------------

``addmod(uint x, uint y, uint k) returns (uint)``:
    计算 ``(x + y) % k`` ，这是在任何精度下执行加法，且不包在 ``2**256`` 。声明在0.5.0版本 ``k != 0`` 。
``mulmod(uint x, uint y, uint k) returns (uint)``:
    计算 ``(x * y) % k`` ，这是在任何精度下执行乘法，且不包在 ``2**256`` 。声明在0.5.0版本 ``k != 0`` 。
``keccak256(...) returns (bytes32)``:
    计算ref: `(tightly packed) arguments <abi_packed_mode>`的Ethereum-SHA-3哈希。
``sha256(...) returns (bytes32)``:
    计算ref:`(tightly packed) arguments <abi_packed_mode>`的SHA-256哈希。
``sha3(...) returns (bytes32)``:
     ``keccak256` `的别名
``ripemd160(...) returns (bytes20)``:
    计算ref:`(tightly packed) arguments <abi_packed_mode>`的RIPEMD-160哈希。
``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``:
    利用椭圆曲线签名恢复与公钥相关的地址，错误返回零值。
    (`example usage <https://ethereum.stackexchange.com/q/1777/222>`_)

上面的 "tightly packed" 意味着实参没有任何空格连在一起。下面几种情况都是等价的::

    keccak256("ab", "c")
    keccak256("abc")
    keccak256(0x616263)
    keccak256(6382179)
    keccak256(97, 98, 99)

如果需要有空格，可以使用显式类型转换： ``keccak256("\x00\x12")`` 和``keccak256(uint16(0x12))`` 是一样的。

注意：用需要存储他们的最少字节对常量进行打包。例如： ``keccak256(0) == keccak256(uint8(0))`` ， ``keccak256(0x12345678) == keccak256(uint32(0x12345678))`` 。

在一个私链上，你很有可能碰到由于 ``sha256`` ， ``ripemd160`` 或者 ``ecrecover`` 引起的燃料耗尽。这个原因就是他们被当做所谓的预编译合约而执行，此外，在第一次收到信息后（尽管合约代码是硬代码）这些合约才真正存在。对于不存在合约的信息花费很贵，因此执行的时候碰到燃料耗尽错误。对于此类问题的变通方法就是，在你实际合约中用到它们时，给每一个合约发送比如1Wei的费用。这在官方网络或测试网络上没有声明。

.. index:: balance, send, transfer, call, callcode, delegatecall
.. _address_related:

地址有关事项
---------------

``<address>.balance`` (``uint256``):
    以Wei计的ref:`address`的余额
``<address>.transfer(uint256 amount)``:
    把以Wei计、给定数量的费用发送给ref:`address`，失败时抛掉，奉送2300gas的费用，不得更改
``<address>.send(uint256 amount) returns (bool)``:
    把以Wei计、给定数量的费用发送给ref:`address`，失败时返回 ``false`` ，奉送2300gas的费用，不得更改
``<address>.call(...) returns (bool)``:
    发出低级 ``CALL`` ，失败时返回 ``false`` ， 奉送所有可用gas，不得更改
``<address>.callcode(...) returns (bool)``:
    发出低级 ``CALLCODE`` ，失败时返回 ``false`` ， 奉送所有可用gas，不得更改
``<address>.delegatecall(...) returns (bool)``:
    发出低级 ``DELEGATECALL`` ，失败时返回 ``false`` ， 奉送所有可用gas，不得更改

更多信息，参考ref:`address`部分：

.. warning::
有很多使用``send`` 的危险情况：如果调用叠加深度在1024（这总是可能调用者强制的）处，将导致转账失败，此外，如果接受者花完gas，这也可能导致失败。为了保证以太币转账安全，总是检查 ``send`` 的返回值，利用 ``transfer`` 或者下面更好的方式：
用这种接收者取回钱的模式。

.. note::
     ``callcode`` 的功用失效且将被删除。

.. index:: this, selfdestruct

合约有关事项
----------------

``this`` (current contract's type):
    当前合约，可以明确转换为：ref:`address`

``selfdestruct(address recipient)``:
    撕毁当前合约，把钱寄到给定的：ref:`address`

``suicide(address recipient)``:
     ``selfdestruct`` 的别名

此外，可以调用当前合约内的所有函数，直接包括当前函数。
