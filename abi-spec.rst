.. index:: abi, application binary interface

.. _ABI:

******************************************
应用二进制接口说明
******************************************

基本设计
============

The Application Binary Interface is the standard way to interact with contracts in the Ethereum ecosystem, both
from outside the blockchain and for contract-to-contract interaction. Data is encoded according to its type,
as described in this specification.  The encoding is not self describing and thus requires a schema in order to decode.
在以太坊生态系统中，应用二进制接口（即Application Binary Interface，ABI）是从区块链外部与合约进行交互以及合约与合约间进行交互的一种标准方式。
数据会按照下文中说明的方法、基于其类型进行编码。这种编码并不是可以自描述的，而是需要一种特定的概要（schema）来进行解码。

We assume the interface functions of a contract are strongly typed, known at compilation time and static. No introspection mechanism will be provided. We assume that all contracts will have the interface definitions of any contracts they call available at compile-time.
我们假定合约函数的接口都是强类型的，且在编译时是可知的和静态的；不提供自我检查机制。我们假定在编译时，所有合约要调用的其他合约接口定义都是可用的。

This specification does not address contracts whose interface is dynamic or otherwise known only at run-time. Should these cases become important they can be adequately handled as facilities built within the Ethereum ecosystem.
这份技术手册并不针对那些动态合约接口或者仅在运行时才可获知的合约接口。如果这种场景变得很重要，你可以使用以太坊生态系统中其他更合适的基础设施来处理它们。

.. _abi_function_selector:

函数选择器（Function Selector）
=================

The first four bytes of the call data for a function call specifies the function to be called. It is the
first (left, high-order in big-endian) four bytes of the Keccak (SHA-3) hash of the signature of the function. The signature is defined as the canonical expression of the basic prototype, i.e.
the function name with the parenthesised list of parameter types. Parameter types are split by a single comma - no spaces are used.
一个函数调用数据的前4字节，指定了要调用的函数。这就是某个函数签名的Keccak（SHA-3）哈希的前4字节（高位在左的大端序）（译注：这里的“高位在左的大端序“，指每字节数据中的8位二进制数据最高位在最左边；且字节流中的最高位字节存储在最低位地址上的一种串行化编码方式）。
这种签名被定义为基础协议的权威表达法，例如，附加了参数类型列表的函数名称，参数类型由一个逗号分隔开，且没有空格。

参数编码（Argument Encoding）
=================

Starting from the fifth byte, the encoded arguments follow. This encoding is also used in other places, e.g. the return values and also event arguments are encoded in the same way, without the four bytes specifying the function.
从第5字节开始是被编码的参数。这种编码也被用在其他地方，比如，返回值和事件的参数也会被用同样的方式进行编码，去掉用来指定函数的4个字节。

类型
=====

The following elementary types exist:
以下是基础类型：

- ``uint<M>``: unsigned integer type of ``M`` bits, ``0 < M <= 256``, ``M % 8 == 0``. e.g. ``uint32``, ``uint8``, ``uint256``.
- ``uint<M>`` ： ``M`` 位的无符号整数， ``0 < M <= 256`` 、 ``M % 8 == 0`` 。例如： ``uint32`` ， ``uint8`` ， ``uint256`` 。

- ``int<M>``: two's complement signed integer type of ``M`` bits, ``0 < M <= 256``, ``M % 8 == 0``.
- ``int<M>`` ：以2的补码作为符号的 ``M`` 位整数， ``0 < M <= 256`` 、 ``M % 8 == 0`` 。

- ``address``: equivalent to ``uint160``, except for the assumed interpretation and language typing. For computing the function selector, ``address`` is used.
- ``address`` ：除了表现上的理解和语言类型的区别以外，等价于 ``uint160`` 。在计算和函数选择器中，通常使用 ``address`` 。

- ``uint``, ``int``: synonyms for ``uint256``, ``int256`` respectively. For computing the function selector, ``uint256`` and ``int256`` have to be used.
- ``uint`` 、 ``int`` ： ``uint256`` 、 ``int256`` 各自的同义词。在计算和函数选择器中，通常使用 ``uint256`` 和 ``int256`` 。

- ``bool``: equivalent to ``uint8`` restricted to the values 0 and 1. For computing the function selector, ``bool`` is used.
- ``bool`` ：等价于 ``uint8`` ，取值限定为0或1。在计算和函数选择其中，通常使用 ``bool`` 。

- ``fixed<M>x<N>``: signed fixed-point decimal number of ``M`` bits, ``8 <= M <= 256``, ``M % 8 ==0``, and ``0 < N <= 80``, which denotes the value ``v`` as ``v / (10 ** N)``.
- ``fixed<M>x<N>`` ： ``M`` 位的有符号的固定小数位数字 ``8 <= M <= 256`` 、 ``M % 8 ==0`` 、且 ``0 < N <= 80`` 。其值 ``v`` 即是 ``v / (10 ** N)`` 。

- ``ufixed<M>x<N>``: unsigned variant of ``fixed<M>x<N>``.
- ``ufixed<M>x<N>`` ：  ``fixed<M>x<N>`` 的无符号变量。

- ``fixed``, ``ufixed``: synonyms for ``fixed128x19``, ``ufixed128x19`` respectively. For computing the function selector, ``fixed128x19`` and ``ufixed128x19`` have to be used.
- ``fixed`` 、 ``ufixed`` ： ``fixed128x19`` 、 ``ufixed128x19`` 各自的同义词。在计算和函数选择器中，通常使用 ``fixed128x19`` 和 ``ufixed128x19`` 。

- ``bytes<M>``: binary type of ``M`` bytes, ``0 < M <= 32``.
- ``bytes<M>`` ： ``M`` 字节的二进制类型， ``0 < M <= 32`` 。

- ``function``: an address (20 bytes) folled by a function selector (4 bytes). Encoded identical to ``bytes24``.
- ``function`` ：一个地址（20字节）之后紧跟一个函数选择器（4字节）。编码之后等价于 ``bytes24`` 。

The following (fixed-size) array type exists:
以下是定长数组类型：

- ``<type>[M]``: a fixed-length array of ``M`` elements, ``M > 0``, of the given type.
- ``<type>[M]`` ：有 ``M`` 个元素的定长数组， ``M > 0`` ，数组元素为给定类型。

The following non-fixed-size types exist:
以下是非定长类型：

- ``bytes``: dynamic sized byte sequence.
- ``bytes`` ：动态大小的字节序列。

- ``string``: dynamic sized unicode string assumed to be UTF-8 encoded.
- ``string`` ：动态大小的unicode字符串，通常假定为UTF-8编码。

- ``<type>[]``: a variable-length array of elements of the given type.
- ``<type>[]`` ：元素为给定类型的变长数组。

Types can be combined to a tuple by enclosing a finite non-negative number
of them inside parentheses, separated by commas:
类型可以通过添加非负下标放到一对括号中，用逗号分隔开，以此来构成一个元组：

- ``(T1,T2,...,Tn)``: tuple consisting of the types ``T1``, ..., ``Tn``, ``n >= 0``
- ``(T1,T2,...,Tn)`` ：由 ``T1`` ， ...， ``Tn`` ， ``n >= 0`` 构成的元组。

It is possible to form tuples of tuples, arrays of tuples and so on.
用元组构成元组、用数组构成元组等等也是可能的。

.. note::
    Solidity supports all the types presented above with the same names with the exception of tuples. The ABI tuple type is utilised for encoding Solidity ``structs``.
    除了元组（tuple）一台，Solidity支持以上所有类型的名称。ABI元组是利用Solidity的 ``structs`` 编码得到的。

编码的形式化说明
====================================

We will now formally specify the encoding, such that it will have the following
properties, which are especially useful if some arguments are nested arrays:
我们现在来正式讲述编码，它具有如下属性，如果参数是嵌套的数组，这些属性非常有用：

属性：

  1. The number of reads necessary to access a value is at most the depth of the value inside the argument array structure, i.e. four reads are needed to retrieve ``a_i[k][l][r]``. In a previous version of the ABI, the number of reads scaled linearly with the total number of dynamic parameters in the worst case.
  1、读取的次数取决于参数数组结构中的最大深度，例如要取得 ``a_i[k][l][r]`` 需要读取4次。在先前的ABI版本中，在最糟的情况下，读取的次数会随着动态参数的总数而线性地增长。

  2. The data of a variable or array element is not interleaved with other data and it is relocatable, i.e. it only uses relative "addresses"
  2、一个变量或数组元素的数据，不会被插入其他的数据，并且是不可再定位的，例如，它们只会使用相对的“地址”。

We distinguish static and dynamic types. Static types are encoded in-place and dynamic types are encoded at a separately allocated location after the current block.
我们需要区分静态和动态类型。静态类型会在原地（即当前区块，译者注）被编码，动态类型则会在当前区块之后的独立分配的位置被编码（即在实际执行时才会被编码，译者注）。

**Definition:** The following types are called "dynamic":
**定义：** 以下类型被称为“动态”：

* ``bytes``
* ``string``
* 任意 `T` 的 ``T[]``
* 任意动态 `T` 的 ``T[k]`` 且任意 ``k > 0``
* ``(T1,...,Tk)`` ，任意动态 ``1 <= i <= k`` 的 ``Ti``

All other types are called "static".
所有其他类型都被称为“静态”。

**Definition:** ``len(a)`` is the number of bytes in a binary string ``a``.
The type of ``len(a)`` is assumed to be ``uint256``.

We define ``enc``, the actual encoding, as a mapping of values of the ABI types to binary strings such
that ``len(enc(X))`` depends on the value of ``X`` if and only if the type of ``X`` is dynamic.

**Definition:** For any ABI value ``X``, we recursively define ``enc(X)``, depending
on the type of ``X`` being

- ``(T1,...,Tk)`` for ``k >= 0`` and any types ``T1``, ..., ``Tk``

  ``enc(X) = head(X(1)) ... head(X(k-1)) tail(X(0)) ... tail(X(k-1))``

  where ``X(i)`` is the ``ith`` component of the value, and
  ``head`` and ``tail`` are defined for ``Ti`` being a static type as

    ``head(X(i)) = enc(X(i))`` and ``tail(X(i)) = ""`` (the empty string)

  and as

    ``head(X(i)) = enc(len(head(X(0)) ... head(X(k-1)) tail(X(0)) ... tail(X(i-1))))``
    ``tail(X(i)) = enc(X(i))``

  otherwise, i.e. if ``Ti`` is a dynamic type.

  Note that in the dynamic case, ``head(X(i))`` is well-defined since the lengths of
  the head parts only depend on the types and not the values. Its value is the offset
  of the beginning of ``tail(X(i))`` relative to the start of ``enc(X)``.

- ``T[k]`` for any ``T`` and ``k``:

  ``enc(X) = enc((X[0], ..., X[k-1]))``

  i.e. it is encoded as if it were a tuple with ``k`` elements
  of the same type.

- ``T[]`` where ``X`` has ``k`` elements (``k`` is assumed to be of type ``uint256``):

  ``enc(X) = enc(k) enc([X[1], ..., X[k]])``

  i.e. it is encoded as if it were an array of static size ``k``, prefixed with
  the number of elements.

- ``bytes``, of length ``k`` (which is assumed to be of type ``uint256``):

  ``enc(X) = enc(k) pad_right(X)``, i.e. the number of bytes is encoded as a
    ``uint256`` followed by the actual value of ``X`` as a byte sequence, followed by
    the minimum number of zero-bytes such that ``len(enc(X))`` is a multiple of 32.

- ``string``:

  ``enc(X) = enc(enc_utf8(X))``, i.e. ``X`` is utf-8 encoded and this value is interpreted as of ``bytes`` type and encoded further. Note that the length used in this subsequent encoding is the number of bytes of the utf-8 encoded string, not its number of characters.

- ``uint<M>``: ``enc(X)`` is the big-endian encoding of ``X``, padded on the higher-order (left) side with zero-bytes such that the length is a multiple of 32 bytes.
- ``address``: as in the ``uint160`` case
- ``int<M>``: ``enc(X)`` is the big-endian two's complement encoding of ``X``, padded on the higher-order (left) side with ``0xff`` for negative ``X`` and with zero bytes for positive ``X`` such that the length is a multiple of 32 bytes.
- ``bool``: as in the ``uint8`` case, where ``1`` is used for ``true`` and ``0`` for ``false``
- ``fixed<M>x<N>``: ``enc(X)`` is ``enc(X * 10**N)`` where ``X * 10**N`` is interpreted as a ``int256``.
- ``fixed``: as in the ``fixed128x19`` case
- ``ufixed<M>x<N>``: ``enc(X)`` is ``enc(X * 10**N)`` where ``X * 10**N`` is interpreted as a ``uint256``.
- ``ufixed``: as in the ``ufixed128x19`` case
- ``bytes<M>``: ``enc(X)`` is the sequence of bytes in ``X`` padded with zero-bytes to a length of 32.

Note that for any ``X``, ``len(enc(X))`` is a multiple of 32.

函数选择器和参数编码
=======================================

All in all, a call to the function ``f`` with parameters ``a_1, ..., a_n`` is encoded as

  ``function_selector(f) enc((a_1, ..., a_n))``

and the return values ``v_1, ..., v_k`` of ``f`` are encoded as

  ``enc((v_1, ..., v_k))``

i.e. the values are combined into a tuple and encoded.

例子
========

Given the contract:

::

    pragma solidity ^0.4.16;

    contract Foo {
      function bar(bytes3[2]) public pure {}
      function baz(uint32 x, bool y) public pure returns (bool r) { r = x > 32 || y; }
      function sam(bytes, bool, uint[]) public pure {}
    }


Thus for our ``Foo`` example if we wanted to call ``baz`` with the parameters ``69`` and ``true``, we would pass 68 bytes total, which can be broken down into:

- ``0xcdcd77c0``: the Method ID. This is derived as the first 4 bytes of the Keccak hash of the ASCII form of the signature ``baz(uint32,bool)``.
- ``0x0000000000000000000000000000000000000000000000000000000000000045``: the first parameter, a uint32 value ``69`` padded to 32 bytes
- ``0x0000000000000000000000000000000000000000000000000000000000000001``: the second parameter - boolean ``true``, padded to 32 bytes

In total::

    0xcdcd77c000000000000000000000000000000000000000000000000000000000000000450000000000000000000000000000000000000000000000000000000000000001

It returns a single ``bool``. If, for example, it were to return ``false``, its output would be the single byte array ``0x0000000000000000000000000000000000000000000000000000000000000000``, a single bool.

If we wanted to call ``bar`` with the argument ``["abc", "def"]``, we would pass 68 bytes total, broken down into:

- ``0xfce353f6``: the Method ID. This is derived from the signature ``bar(bytes3[2])``.
- ``0x6162630000000000000000000000000000000000000000000000000000000000``: the first part of the first parameter, a ``bytes3`` value ``"abc"`` (left-aligned).
- ``0x6465660000000000000000000000000000000000000000000000000000000000``: the second part of the first parameter, a ``bytes3`` value ``"def"`` (left-aligned).

In total::

    0xfce353f661626300000000000000000000000000000000000000000000000000000000006465660000000000000000000000000000000000000000000000000000000000

If we wanted to call ``sam`` with the arguments ``"dave"``, ``true`` and ``[1,2,3]``, we would pass 292 bytes total, broken down into:

- ``0xa5643bf2``: the Method ID. This is derived from the signature ``sam(bytes,bool,uint256[])``. Note that ``uint`` is replaced with its canonical representation ``uint256``.
- ``0x0000000000000000000000000000000000000000000000000000000000000060``: the location of the data part of the first parameter (dynamic type), measured in bytes from the start of the arguments block. In this case, ``0x60``.
- ``0x0000000000000000000000000000000000000000000000000000000000000001``: the second parameter: boolean true.
- ``0x00000000000000000000000000000000000000000000000000000000000000a0``: the location of the data part of the third parameter (dynamic type), measured in bytes. In this case, ``0xa0``.
- ``0x0000000000000000000000000000000000000000000000000000000000000004``: the data part of the first argument, it starts with the length of the byte array in elements, in this case, 4.
- ``0x6461766500000000000000000000000000000000000000000000000000000000``: the contents of the first argument: the UTF-8 (equal to ASCII in this case) encoding of ``"dave"``, padded on the right to 32 bytes.
- ``0x0000000000000000000000000000000000000000000000000000000000000003``: the data part of the third argument, it starts with the length of the array in elements, in this case, 3.
- ``0x0000000000000000000000000000000000000000000000000000000000000001``: the first entry of the third parameter.
- ``0x0000000000000000000000000000000000000000000000000000000000000002``: the second entry of the third parameter.
- ``0x0000000000000000000000000000000000000000000000000000000000000003``: the third entry of the third parameter.

In total::

    0xa5643bf20000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000a0000000000000000000000000000000000000000000000000000000000000000464617665000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000003

动态类型的使用
====================

A call to a function with the signature ``f(uint,uint32[],bytes10,bytes)`` with values ``(0x123, [0x456, 0x789], "1234567890", "Hello, world!")`` is encoded in the following way:

We take the first four bytes of ``sha3("f(uint256,uint32[],bytes10,bytes)")``, i.e. ``0x8be65246``.
Then we encode the head parts of all four arguments. For the static types ``uint256`` and ``bytes10``, these are directly the values we want to pass, whereas for the dynamic types ``uint32[]`` and ``bytes``, we use the offset in bytes to the start of their data area, measured from the start of the value encoding (i.e. not counting the first four bytes containing the hash of the function signature). These are:

 - ``0x0000000000000000000000000000000000000000000000000000000000000123`` (``0x123`` padded to 32 bytes)
 - ``0x0000000000000000000000000000000000000000000000000000000000000080`` (offset to start of data part of second parameter, 4*32 bytes, exactly the size of the head part)
 - ``0x3132333435363738393000000000000000000000000000000000000000000000`` (``"1234567890"`` padded to 32 bytes on the right)
 - ``0x00000000000000000000000000000000000000000000000000000000000000e0`` (offset to start of data part of fourth parameter = offset to start of data part of first dynamic parameter + size of data part of first dynamic parameter = 4\*32 + 3\*32 (see below))

After this, the data part of the first dynamic argument, ``[0x456, 0x789]`` follows:

 - ``0x0000000000000000000000000000000000000000000000000000000000000002`` (number of elements of the array, 2)
 - ``0x0000000000000000000000000000000000000000000000000000000000000456`` (first element)
 - ``0x0000000000000000000000000000000000000000000000000000000000000789`` (second element)

Finally, we encode the data part of the second dynamic argument, ``"Hello, world!"``:

 - ``0x000000000000000000000000000000000000000000000000000000000000000d`` (number of elements (bytes in this case): 13)
 - ``0x48656c6c6f2c20776f726c642100000000000000000000000000000000000000`` (``"Hello, world!"`` padded to 32 bytes on the right)

All together, the encoding is (newline after function selector and each 32-bytes for clarity):

::

    0x8be65246
      0000000000000000000000000000000000000000000000000000000000000123
      0000000000000000000000000000000000000000000000000000000000000080
      3132333435363738393000000000000000000000000000000000000000000000
      00000000000000000000000000000000000000000000000000000000000000e0
      0000000000000000000000000000000000000000000000000000000000000002
      0000000000000000000000000000000000000000000000000000000000000456
      0000000000000000000000000000000000000000000000000000000000000789
      000000000000000000000000000000000000000000000000000000000000000d
      48656c6c6f2c20776f726c642100000000000000000000000000000000000000

事件
======

Events are an abstraction of the Ethereum logging/event-watching protocol. Log entries provide the contract's address, a series of up to four topics and some arbitrary length binary data. Events leverage the existing function ABI in order to interpret this (together with an interface spec) as a properly typed structure.

Given an event name and series of event parameters, we split them into two sub-series: those which are indexed and those which are not. Those which are indexed, which may number up to 3, are used alongside the Keccak hash of the event signature to form the topics of the log entry. Those which are not indexed form the byte array of the event.

In effect, a log entry using this ABI is described as:

- ``address``: the address of the contract (intrinsically provided by Ethereum);
- ``topics[0]``: ``keccak(EVENT_NAME+"("+EVENT_ARGS.map(canonical_type_of).join(",")+")")`` (``canonical_type_of`` is a function that simply returns the canonical type of a given argument, e.g. for ``uint indexed foo``, it would return ``uint256``). If the event is declared as ``anonymous`` the ``topics[0]`` is not generated;
- ``topics[n]``: ``EVENT_INDEXED_ARGS[n - 1]`` (``EVENT_INDEXED_ARGS`` is the series of ``EVENT_ARGS`` that are indexed);
- ``data``: ``abi_serialise(EVENT_NON_INDEXED_ARGS)`` (``EVENT_NON_INDEXED_ARGS`` is the series of ``EVENT_ARGS`` that are not indexed, ``abi_serialise`` is the ABI serialisation function used for returning a series of typed values from a function, as described above).

For all fixed-length Solidity types, the ``EVENT_INDEXED_ARGS`` array contains the 32-byte encoded value directly. However, for *types of dynamic length*, which include ``string``, ``bytes``, and arrays, ``EVENT_INDEXED_ARGS`` will contain the *Keccak hash* of the encoded value, rather than the encoded value directly. This allows applications to efficiently query for values of dynamic-length types (by setting the hash of the encoded value as the topic), but leaves applications unable to decode indexed values they have not queried for. For dynamic-length types, application developers face a trade-off between fast search for predetermined values (if the argument is indexed) and legibility of arbitrary values (which requires that the arguments not be indexed). Developers may overcome this tradeoff and achieve both efficient search and arbitrary legibility by defining events with two arguments — one indexed, one not — intended to hold the same value.

JSON
====

The JSON format for a contract's interface is given by an array of function and/or event descriptions.
A function description is a JSON object with the fields:

- ``type``: ``"function"``, ``"constructor"``, or ``"fallback"`` (the :ref:`unnamed "default" function <fallback-function>`);
- ``name``: the name of the function;
- ``inputs``: an array of objects, each of which contains:

  * ``name``: the name of the parameter;
  * ``type``: the canonical type of the parameter (more below).
  * ``components``: used for tuple types (more below).

- ``outputs``: an array of objects similar to ``inputs``, can be omitted if function doesn't return anything;
- ``payable``: ``true`` if function accepts ether, defaults to ``false``;
- ``stateMutability``: a string with one of the following values: ``pure`` (:ref:`specified to not read blockchain state <pure-functions>`), ``view`` (:ref:`specified to not modify the blockchain state <view-functions>`), ``nonpayable`` and ``payable`` (same as ``payable`` above).
- ``constant``: ``true`` if function is either ``pure`` or ``view``

``type`` can be omitted, defaulting to ``"function"``.

Constructor and fallback function never have ``name`` or ``outputs``. Fallback function doesn't have ``inputs`` either.

Sending non-zero ether to non-payable function will throw. Don't do it.

An event description is a JSON object with fairly similar fields:

- ``type``: always ``"event"``
- ``name``: the name of the event;
- ``inputs``: an array of objects, each of which contains:

  * ``name``: the name of the parameter;
  * ``type``: the canonical type of the parameter (more below).
  * ``components``: used for tuple types (more below).
  * ``indexed``: ``true`` if the field is part of the log's topics, ``false`` if it one of the log's data segment.

- ``anonymous``: ``true`` if the event was declared as ``anonymous``.

For example,

::

    pragma solidity ^0.4.0;

    contract Test {
      function Test() public { b = 0x12345678901234567890123456789012; }
      event Event(uint indexed a, bytes32 b);
      event Event2(uint indexed a, bytes32 b);
      function foo(uint a) public { Event(a, b); }
      bytes32 b;
    }

would result in the JSON:

.. code:: json

  [{
  "type":"event",
  "inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
  "name":"Event"
  }, {
  "type":"event",
  "inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
  "name":"Event2"
  }, {
  "type":"function",
  "inputs": [{"name":"a","type":"uint256"}],
  "name":"foo",
  "outputs": []
  }]

Handling tuple types
--------------------

Despite that names are intentionally not part of the ABI encoding they do make a lot of sense to be included
in the JSON to enable displaying it to the end user. The structure is nested in the following way:

An object with members ``name``, ``type`` and potentially ``components`` describes a typed variable.
The canonical type is determined until a tuple type is reached and the string description up
to that point is stored in ``type`` prefix with the word ``tuple``, i.e. it will be ``tuple`` followed by
a sequence of ``[]`` and ``[k]`` with
integers ``k``. The components of the tuple are then stored in the member ``components``,
which is of array type and has the same structure as the top-level object except that
``indexed`` is not allowed there.

As an example, the code

::

    pragma solidity ^0.4.19;
    pragma experimental ABIEncoderV2;

    contract Test {
      struct S { uint a; uint[] b; T[] c; }
      struct T { uint x; uint y; }
      function f(S s, T t, uint a) public { }
      function g() public returns (S s, T t, uint a) {}
    }

would result in the JSON:

.. code:: json

  [
    {
      "name": "f",
      "type": "function",
      "inputs": [
        {
          "name": "s",
          "type": "tuple",
          "components": [
            {
              "name": "a",
              "type": "uint256"
            },
            {
              "name": "b",
              "type": "uint256[]"
            },
            {
              "name": "c",
              "type": "tuple[]",
              "components": [
                {
                  "name": "x",
                  "type": "uint256"
                },
                {
                  "name": "y",
                  "type": "uint256"
                }
              ]
            }
          ]
        },
        {
          "name": "t",
          "type": "tuple",
          "components": [
            {
              "name": "x",
              "type": "uint256"
            },
            {
              "name": "y",
              "type": "uint256"
            }
          ]
        },
        {
          "name": "a",
          "type": "uint256"
        }
      ],
      "outputs": []
    }
  ]

.. _abi_packed_mode:

非标准打包模式
========================

Solidity supports a non-standard packed mode where:

- no :ref:`function selector <abi_function_selector>` is encoded,
- types shorter than 32 bytes are neither zero padded nor sign extended and
- dynamic types are encoded in-place and without the length.

As an example encoding ``int1, bytes1, uint16, string`` with values ``-1, 0x42, 0x2424, "Hello, world!"`` results in ::

    0xff42242448656c6c6f2c20776f726c6421
      ^^                                 int1(-1)
        ^^                               bytes1(0x42)
          ^^^^                           uint16(0x2424)
              ^^^^^^^^^^^^^^^^^^^^^^^^^^ string("Hello, world!") without a length field

More specifically, each statically-sized type takes as many bytes as its range has
and dynamically-sized types like ``string``, ``bytes`` or ``uint[]`` are encoded without
their length field. This means that the encoding is ambiguous as soon as there are two
dynamically-sized elements.
