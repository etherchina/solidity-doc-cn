**********
速查表
**********

.. index:: precedence

.. _order:

运算符优先级
================================

以下是运算符的优先级顺序，以优先级顺序列出。


+------------+-------------------------------------+--------------------------------------------+
| 优先级      | 描述                                | 运算符                                     |
+============+=====================================+============================================+
| *1*        | 后缀自加和自减                       | ``++``, ``--``                            |
+            +-------------------------------------+--------------------------------------------+
|            | New expression                      | ``new <typename>``                        |
+            +-------------------------------------+--------------------------------------------+
|            | Array subscripting                  | ``<array>[<index>]``                       |
+            +-------------------------------------+--------------------------------------------+
|            | Member access                       | ``<object>.<member>``                      |
+            +-------------------------------------+--------------------------------------------+
|            | Function-like call                  | ``<func>(<args...>)``                      |
+            +-------------------------------------+--------------------------------------------+
|            | Parentheses                         | ``(<statement>)``                          |
+------------+-------------------------------------+--------------------------------------------+
| *2*        | Prefix increment and decrement      | ``++``, ``--``                             |
+            +-------------------------------------+--------------------------------------------+
|            | Unary minus                         | ``-``                                      |
+            +-------------------------------------+--------------------------------------------+
|            | Unary operations                    | ``delete``                                 |
+            +-------------------------------------+--------------------------------------------+
|            | Logical NOT                         | ``!``                                      |
+            +-------------------------------------+--------------------------------------------+
|            | Bitwise NOT                         | ``~``                                      |
+------------+-------------------------------------+--------------------------------------------+
| *3*        | Exponentiation                      | ``**``                                     |
+------------+-------------------------------------+--------------------------------------------+
| *4*        | Multiplication, division and modulo | ``*``, ``/``, ``%``                        |
+------------+-------------------------------------+--------------------------------------------+
| *5*        | Addition and subtraction            | ``+``, ``-``                               |
+------------+-------------------------------------+--------------------------------------------+
| *6*        | Bitwise shift operators             | ``<<``, ``>>``                             |
+------------+-------------------------------------+--------------------------------------------+
| *7*        | Bitwise AND                         | ``&``                                      |
+------------+-------------------------------------+--------------------------------------------+
| *8*        | Bitwise XOR                         | ``^``                                      |
+------------+-------------------------------------+--------------------------------------------+
| *9*        | Bitwise OR                          | ``|``                                      |
+------------+-------------------------------------+--------------------------------------------+
| *10*       | Inequality operators                | ``<``, ``>``, ``<=``, ``>=``               |
+------------+-------------------------------------+--------------------------------------------+
| *11*       | Equality operators                  | ``==``, ``!=``                             |
+------------+-------------------------------------+--------------------------------------------+
| *12*       | Logical AND                         | ``&&``                                     |
+------------+-------------------------------------+--------------------------------------------+
| *13*       | Logical OR                          | ``||``                                     |
+------------+-------------------------------------+--------------------------------------------+
| *14*       | Ternary operator                    | ``<conditional> ? <if-true> : <if-false>`` |
+            +-------------------------------------+--------------------------------------------+
|            | Assignment operators                | ``=``, ``|=``, ``^=``, ``&=``, ``<<=``,    |
|            |                                     | ``>>=``, ``+=``, ``-=``, ``*=``, ``/=``,   |
|            |                                     | ``%=``                                     |
+------------+-------------------------------------+--------------------------------------------+
| *15*       | Comma operator                      | ``,``                                      |
+------------+-------------------------------------+--------------------------------------------+

.. index:: assert, block, coinbase, difficulty, number, block;number, timestamp, block;timestamp, msg, data, gas, sender, value, gas price, origin, revert, require, keccak256, ripemd160, sha256, ecrecover, addmod, mulmod, cryptography, this, super, selfdestruct, balance, codehash, send

全局变量
================


- ``abi.decode(bytes memory encodedData, (...)) returns (...)``: :ref:`ABI <ABI>`- 对提供的数据进行解码，类型在括号中作为第二个参数给出。
  示例: ``(uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))``
- ``abi.encode(...) returns (bytes memory)``: :ref:`ABI <ABI>` - 对给定的参数进行编码
- ``abi.encodePacked(...) returns (bytes memory)``: 给指定的参数执行 :ref:`packed encoding <abi_packed_mode>` ， 请注意，这种编码可能会有歧义!（参数和编码可能出现多对一的情况）
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory)``: :ref:`ABI <ABI>`- 为给定的 4 字节选择器和随后的参数进行编码。
- ``abi.encodeCall(function functionPointer, (...)) returns (bytes memory)``: 对 ``functionPointer`` 指向的函数调用及元组中的参数进行编码，执行完整的类型检查，确保类型与函数签名相符。结果等于 ``abi.encodeWithSelector(functionPointer.selector, (...))``
- ``abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)``: 等于 ``abi.encodeWithSelector(bytes4(keccak256(bytes(signature)), ...)``
- ``bytes.concat(...) returns (bytes memory)``: :ref:`将可变数量的参数串联成一个字节数组<bytes-concat>`
- ``string.concat(...) returns (string memory)``: :ref:`将可变数量的参数串联成一个字符串<string-concat>`
- ``block.basefee`` (``uint``): 当前区块的基础gas fee ， 参考 (`EIP-3198 <https://eips.ethereum.org/EIPS/eip-3198>`_ 和 `EIP-1559 <https://eips.ethereum.org/EIPS/eip-1559>`_)
- ``block.chainid`` (``uint``): 当前 chain id
- ``block.coinbase`` (``address payable``): 当前区块矿工的地址
- ``block.difficulty`` (``uint``): 当前区块难度
- ``block.gaslimit`` (``uint``): 当前区块gaslimit
- ``block.number`` (``uint``): 当前区块号
- ``block.timestamp`` (``uint``): 当前区块时间戳（以Unix epoch依赖的秒数）
- ``gasleft() returns (uint256)``: 剩余 gas
- ``msg.data`` (``bytes``): 完整的 calldata 数据
- ``msg.sender`` (``address``): 消息调用者 (当前调用)
- ``msg.sig`` (``bytes4``): calldata的前 4 个字节 (如：函数签名)
- ``msg.value`` (``uint``): 与消息一起发送的以太币（wei为单位）
- ``tx.gasprice`` (``uint``): 交易的gas 价格
- ``tx.origin`` (``address``): 交易的发起者 (完整的调用链下，最初的发起者)
- ``assert(bool condition)``: 如果条件为 ``false`` ， 终止执行并回退状态改变 (用于内部错误)
- ``require(bool condition)``: 如果条件为 ``false`` ， 终止执行并回退状态改变  (用于检查错误输入，或外部组件的错误)
- ``require(bool condition, string memory message)``: 如果条件为 ``false`` ， 终止执行并回退状态改变  (用于检查错误输入，或外部组件的错误)，同时提供错误信息。
- ``revert()``: 终止执行并回退状态改变
- ``revert(string memory message)``: 终止执行并回退状态改变，同时提供错误解释信息。
- ``blockhash(uint blockNumber) returns (bytes32)``: 指定块的区块hash - 仅最近 256 个区块有效
- ``keccak256(bytes memory) returns (bytes32)``: 计算输入参数的 Keccak-256 哈希
- ``sha256(bytes memory) returns (bytes32)``: 计算输入参数的 SHA-256 哈希
- ``ripemd160(bytes memory) returns (bytes20)``: 计算输入参数的 RIPEMD-160 哈希
- ``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``: 从椭圆曲线签名中恢复出与公钥关联的地址，出错时返回零。
- ``addmod(uint x, uint y, uint k) returns (uint)``: 计算 ``(x + y) % k`` ，其中加法以任意精度执行，不会在 ``2**256`` 处溢出。从 0.5.0 开始要求 ``k != 0`` 。
- ``mulmod(uint x, uint y, uint k) returns (uint)``: 计算 ``(x * y) % k`` ，其中乘法以任意精度执行，不会在 ``2**256`` 处溢出。从 0.5.0 开始要求 ``k != 0`` 。
- ``this`` (current contract's type): 当前合约，可以显式转换为 ``address`` 或 ``address payable``
- ``super``: 继承树的上层合约
- ``selfdestruct(address payable recipient)``: 销毁合约，把合约的剩余资金（以太币）发送到指定的地址。
- ``<address>.balance`` (``uint256``):  :ref:`address` 的余额，以 wei 为单位
- ``<address>.code`` (``bytes memory``): :ref:`address` 的代码 (可以为空)
- ``<address>.codehash`` (``bytes32``): :ref:`address` 的代码 hash
- ``<address payable>.send(uint256 amount) returns (bool)``:  发送 ``amount`` 数量（单位wei）的以太币到 :ref:`address` ， 失败返回 ``false`` 。
- ``<address payable>.transfer(uint256 amount)``: 发送 ``amount`` 数量（单位wei）的以太币到 :ref:`address` ， 失败时抛出异常。
- ``type(C).name`` (``string``): 合约的名称
- ``type(C).creationCode`` (``bytes memory``): 合约的创建字节码，参考 :ref:`类型信息<meta-type>`.
- ``type(C).runtimeCode`` (``bytes memory``): 合约的运行时字节码，参考 :ref:`类型信息<meta-type>`.
- ``type(I).interfaceId`` (``bytes4``): 包含给定接口的EIP-165接口标识符 , 参考 :ref:`类型信息<meta-type>`.
- ``type(T).min`` (``T``): 所在整型 ``T`` 的最小值, 参考 :ref:`类型信息<meta-type>`.
- ``type(T).max`` (``T``): 所在整型 ``T`` 的最大值, 参考 :ref:`类型信息<meta-type>`.

.. note::
    当合约在链外而不是在包含的交易的区块中下被执行时，你不应该假定 ``block.*`` 和 ``tx.*`` 是某特定区块或交易的值。这些值是由执行合约的EVM实现提供的，其值可以是任意的。

.. note::
    不要依赖 ``block.timestamp`` 或 ``blockhash`` 作为随机源，除非你明确知道你所做的事情。

    时间戳和区块哈希值都可以在一定程度上受到矿工的影响。例如，矿工团体中的不良行为者可以在某个依赖随机数的赌场支付功能上，在没有获利情况下重试另一个哈希值。

    当前区块的时间戳必须严格大于上一个区块的时间戳。
    但唯一的能保证是：它将规范链中两个连续区块的时间戳。


.. note::
    由于可扩展的原因，不是所有块哈希都可用。你只能访问最近256个块的哈希值，其他所有值将为零。


.. note::
    在 0.5.0 版本，以下别名移除了: ``suicide``（作为 ``selfdestruct`` 的别名）, 
    ``msg.gas`` （ ``gasleft`` 的的别名）, ``block.blockhash`` （ ``blockhash`` 的别名）以及 ``sha3`` （ ``keccak256`` 的别名）。
.. note::
    在0.7.0版本，别名 ``now`` ( ``block.timestamp`` 的别名) 被移除了。

.. index:: visibility, public, private, external, internal

函数可见性
==============================

.. code-block:: solidity

    function myFunction() <visibility specifier> returns (bool) {
        return true;
    }

- ``public``: visible externally and internally (creates a :ref:`getter function<getter-functions>` for storage/state variables)
- ``private``: only visible in the current contract
- ``external``: only visible externally (only for functions) - i.e. can only be message-called (via ``this.func``)
- ``internal``: only visible internally


.. index:: modifiers, pure, view, payable, constant, anonymous, indexed

修饰符
=========

- ``pure`` for functions: Disallows modification or access of state.
- ``view`` for functions: Disallows modification of state.
- ``payable`` for functions: Allows them to receive Ether together with a call.
- ``constant`` for state variables: Disallows assignment (except initialisation), does not occupy storage slot.
- ``immutable`` for state variables: Allows exactly one assignment at construction time and is constant afterwards. Is stored in code.
- ``anonymous`` for events: Does not store event signature as topic.
- ``indexed`` for event parameters: Stores the parameter as topic.
- ``virtual`` for functions and modifiers: Allows the function's or modifier's
  behaviour to be changed in derived contracts.
- ``override``: States that this function, modifier or public state variable changes
  the behaviour of a function or modifier in a base contract.

保留关键字
=================

These keywords are reserved in Solidity. They might become part of the syntax in the future:

``after``, ``alias``, ``apply``, ``auto``, ``byte``, ``case``, ``copyof``, ``default``,
``define``, ``final``, ``implements``, ``in``, ``inline``, ``let``, ``macro``, ``match``,
``mutable``, ``null``, ``of``, ``partial``, ``promise``, ``reference``, ``relocatable``,
``sealed``, ``sizeof``, ``static``, ``supports``, ``switch``, ``typedef``, ``typeof``,
``var``.
