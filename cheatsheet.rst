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
- ``require(bool condition, string memory message)``: abort execution and revert state changes if
  condition is ``false`` (use for malformed input or error in external component). Also provide error message.
- ``revert()``: abort execution and revert state changes
- ``revert(string memory message)``: abort execution and revert state changes providing an explanatory string
- ``blockhash(uint blockNumber) returns (bytes32)``: hash of the given block - only works for 256 most recent blocks
- ``keccak256(bytes memory) returns (bytes32)``: compute the Keccak-256 hash of the input
- ``sha256(bytes memory) returns (bytes32)``: compute the SHA-256 hash of the input
- ``ripemd160(bytes memory) returns (bytes20)``: compute the RIPEMD-160 hash of the input
- ``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``: recover address associated with
  the public key from elliptic curve signature, return zero on error
- ``addmod(uint x, uint y, uint k) returns (uint)``: compute ``(x + y) % k`` where the addition is performed with
  arbitrary precision and does not wrap around at ``2**256``. Assert that ``k != 0`` starting from version 0.5.0.
- ``mulmod(uint x, uint y, uint k) returns (uint)``: compute ``(x * y) % k`` where the multiplication is performed
  with arbitrary precision and does not wrap around at ``2**256``. Assert that ``k != 0`` starting from version 0.5.0.
- ``this`` (current contract's type): the current contract, explicitly convertible to ``address`` or ``address payable``
- ``super``: the contract one level higher in the inheritance hierarchy
- ``selfdestruct(address payable recipient)``: destroy the current contract, sending its funds to the given address
- ``<address>.balance`` (``uint256``): balance of the :ref:`address` in Wei
- ``<address>.code`` (``bytes memory``): code at the :ref:`address` (can be empty)
- ``<address>.codehash`` (``bytes32``): the codehash of the :ref:`address`
- ``<address payable>.send(uint256 amount) returns (bool)``: send given amount of Wei to :ref:`address`,
  returns ``false`` on failure
- ``<address payable>.transfer(uint256 amount)``: send given amount of Wei to :ref:`address`, throws on failure
- ``type(C).name`` (``string``): the name of the contract
- ``type(C).creationCode`` (``bytes memory``): creation bytecode of the given contract, see :ref:`Type Information<meta-type>`.
- ``type(C).runtimeCode`` (``bytes memory``): runtime bytecode of the given contract, see :ref:`Type Information<meta-type>`.
- ``type(I).interfaceId`` (``bytes4``): value containing the EIP-165 interface identifier of the given interface, see :ref:`Type Information<meta-type>`.
- ``type(T).min`` (``T``): the minimum value representable by the integer type ``T``, see :ref:`Type Information<meta-type>`.
- ``type(T).max`` (``T``): the maximum value representable by the integer type ``T``, see :ref:`Type Information<meta-type>`.

.. note::
    When contracts are evaluated off-chain rather than in context of a transaction included in a
    block, you should not assume that ``block.*`` and ``tx.*`` refer to values from any specific
    block or transaction. These values are provided by the EVM implementation that executes the
    contract and can be arbitrary.

.. note::
    Do not rely on ``block.timestamp`` or ``blockhash`` as a source of randomness,
    unless you know what you are doing.

    Both the timestamp and the block hash can be influenced by miners to some degree.
    Bad actors in the mining community can for example run a casino payout function on a chosen hash
    and just retry a different hash if they did not receive any money.

    The current block timestamp must be strictly larger than the timestamp of the last block,
    but the only guarantee is that it will be somewhere between the timestamps of two
    consecutive blocks in the canonical chain.

.. note::
    The block hashes are not available for all blocks for scalability reasons.
    You can only access the hashes of the most recent 256 blocks, all other
    values will be zero.

.. note::
    In version 0.5.0, the following aliases were removed: ``suicide`` as alias for ``selfdestruct``,
    ``msg.gas`` as alias for ``gasleft``, ``block.blockhash`` as alias for ``blockhash`` and
    ``sha3`` as alias for ``keccak256``.
.. note::
    In version 0.7.0, the alias ``now`` (for ``block.timestamp``) was removed.

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
