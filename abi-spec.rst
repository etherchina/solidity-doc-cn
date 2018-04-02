.. include:: glossaries.rst
.. index:: abi, application binary interface

.. _ABI:

******************************************
|ABI| 说明
******************************************

基本设计
============

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
The Application Binary Interface is the standard way to interact with contracts in the Ethereum ecosystem, both
from outside the blockchain and for contract-to-contract interaction. Data is encoded according to its type,
as described in this specification.  The encoding is not self describing and thus requires a schema in order to decode.
=======
=======
>>>>>>> etherchina/develop
在 |ethereum| 生态系统中， |ABI| 是从区块链外部与合约进行交互以及合约与合约间进行交互的一种标准方式。
数据会根据其类型按照这份手册中说明的方法进行编码。这种编码并不是可以自描述的，而是需要一种特定的概要（schema）来进行解码。
>>>>>>> etherchina/develop
=======
在以太坊生态系统中，应用二进制接口（即Application Binary Interface，ABI）是从区块链外部与合约进行交互以及合约与合约间进行交互的一种标准方式。
数据会根据其类型按照这份手册中说明的方法进行编码。这种编码并不是可以自描述的，而是需要一种特定的概要（schema）来进行解码。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

我们假定合约函数的接口都是强类型的，且在编译时是可知的和静态的；不提供自我检查机制。我们假定在编译时，所有合约要调用的其他合约接口定义都是可用的。

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
This specification does not address contracts whose interface is dynamic or otherwise known only at run-time. Should these cases become important they can be adequately handled as facilities built within the Ethereum ecosystem.
=======
这份手册并不针对那些动态合约接口或者仅在运行时才可获知的合约接口。如果这种场景变得很重要，你可以使用以太坊生态系统中其他更合适的基础设施来处理它们。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

.. _abi_function_selector:

函数选择器（Function Selector）
<<<<<<< HEAD
=================
=======
这份手册并不针对那些动态合约接口或者仅在运行时才可获知的合约接口。如果这种场景变得很重要，你可以使用 |ethereum| 生态系统中其他更合适的基础设施来处理它们。

.. _abi_function_selector:

=======
这份手册并不针对那些动态合约接口或者仅在运行时才可获知的合约接口。如果这种场景变得很重要，你可以使用 |ethereum| 生态系统中其他更合适的基础设施来处理它们。

.. _abi_function_selector:

>>>>>>> etherchina/develop
|function_selector|
=================================
>>>>>>> etherchina/develop
=======
=================================
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

一个函数调用数据的前4字节，指定了要调用的函数。这就是某个函数签名的Keccak（SHA-3）哈希的前4字节（高位在左的大端序）（译注：这里的“高位在左的大端序“，指最高位字节存储在最低位地址上的一种串行化编码方式，即高位字节在左）。
这种签名被定义为基础原型的规范表达，基础原型即是函数名称加上由括号括起来的参数类型列表，参数类型间由一个逗号分隔开，且没有空格。

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
参数编码（Encoding）
=================
=======
=======
>>>>>>> etherchina/develop
参数编码
=================================
>>>>>>> etherchina/develop
=======
参数编码（Argument Encoding）
=================================
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

从第5字节开始是被编码的参数。这种编码也被用在其他地方，比如，返回值和事件的参数也会被用同样的方式进行编码，而用来指定函数的4个字节则不需要再进行编码。

类型
=============

以下是基础类型：

- ``uint<M>`` ： ``M`` 位的无符号整数， ``0 < M <= 256`` 、 ``M % 8 == 0`` 。例如： ``uint32`` ， ``uint8`` ， ``uint256`` 。

- ``int<M>`` ：以2的补码作为符号的 ``M`` 位整数， ``0 < M <= 256`` 、 ``M % 8 == 0`` 。

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
- ``address``: equivalent to ``uint160``, except for the assumed interpretation and language typing. For computing the function selector, ``address`` is used.
=======
- ``address`` ：除了字面上的意思和语言类型的区别以外，等价于 ``uint160`` 。在计算和函数选择器中，通常使用 ``address`` 。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

- ``uint`` 、 ``int`` ： ``uint256`` 、 ``int256`` 各自的同义词。在计算和函数选择器中，通常使用 ``uint256`` 和 ``int256`` 。

<<<<<<< HEAD
- ``bool``: equivalent to ``uint8`` restricted to the values 0 and 1. For computing the function selector, ``bool`` is used.
=======
- ``address`` ：除了字面上的意思和语言类型的区别以外，等价于 ``uint160`` 。在计算和 |function_selector| 中，通常使用 ``address`` 。
=======
- ``address`` ：除了字面上的意思和语言类型的区别以外，等价于 ``uint160`` 。在计算和 |function_selector| 中，通常使用 ``address`` 。

- ``uint`` 、 ``int`` ： ``uint256`` 、 ``int256`` 各自的同义词。在计算和 |function_selector| 中，通常使用 ``uint256`` 和 ``int256`` 。

- ``bool`` ：等价于 ``uint8`` ，取值限定为0或1。在计算和 |function_selector| 中，通常使用 ``bool`` 。
>>>>>>> etherchina/develop

- ``uint`` 、 ``int`` ： ``uint256`` 、 ``int256`` 各自的同义词。在计算和 |function_selector| 中，通常使用 ``uint256`` 和 ``int256`` 。

- ``bool`` ：等价于 ``uint8`` ，取值限定为0或1。在计算和 |function_selector| 中，通常使用 ``bool`` 。
>>>>>>> etherchina/develop

<<<<<<< HEAD
- ``fixed<M>x<N>``: signed fixed-point decimal number of ``M`` bits, ``8 <= M <= 256``, ``M % 8 ==0``, and ``0 < N <= 80``, which denotes the value ``v`` as ``v / (10 ** N)``.
=======
- ``fixed`` 、 ``ufixed`` ： ``fixed128x19`` 、 ``ufixed128x19`` 各自的同义词。在计算和 |function_selector| 中，通常使用 ``fixed128x19`` 和 ``ufixed128x19`` 。
>>>>>>> etherchina/develop
=======
- ``bool`` ：等价于 ``uint8`` ，取值限定为0或1。在计算和函数选择器中，通常使用 ``bool`` 。

- ``fixed<M>x<N>`` ： ``M`` 位的有符号的固定小数位的十进制数字 ``8 <= M <= 256`` 、 ``M % 8 ==0`` 、且 ``0 < N <= 80`` 。其值 ``v`` 即是 ``v / (10 ** N)`` 。（也就是说，这种类型是由M位的二进制数据所保存的，有N位小数的十进制数值。译者注。）
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

- ``ufixed<M>x<N>`` ：无符号的 ``fixed<M>x<N>`` 。

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
- ``fixed``, ``ufixed``: synonyms for ``fixed128x19``, ``ufixed128x19`` respectively. For computing the function selector, ``fixed128x19`` and ``ufixed128x19`` have to be used.
=======
- ``fixed`` 、 ``ufixed`` ： ``fixed128x19`` 、 ``ufixed128x19`` 各自的同义词。在计算和 |function_selector| 中，通常使用 ``fixed128x19`` 和 ``ufixed128x19`` 。
>>>>>>> etherchina/develop
=======
- ``function`` ：一个地址（20字节）之后紧跟一个 |function_selector| （4字节）。编码之后等价于 ``bytes24`` 。
>>>>>>> etherchina/develop
=======
- ``fixed`` 、 ``ufixed`` ： ``fixed128x19`` 、 ``ufixed128x19`` 各自的同义词。在计算和函数选择器中，通常使用 ``fixed128x19`` 和 ``ufixed128x19`` 。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

- ``bytes<M>`` ： ``M`` 字节的二进制类型， ``0 < M <= 32`` 。

<<<<<<< HEAD
<<<<<<< HEAD
- ``function``: an address (20 bytes) folled by a function selector (4 bytes). Encoded identical to ``bytes24``.
=======
- ``function`` ：一个地址（20字节）之后紧跟一个 |function_selector| （4字节）。编码之后等价于 ``bytes24`` 。
>>>>>>> etherchina/develop
=======
- ``function`` ：一个地址（20字节）之后紧跟一个函数选择器（4字节）。编码之后等价于 ``bytes24`` 。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

以下是定长数组类型：

- ``<type>[M]`` ：有 ``M`` 个元素的定长数组， ``M > 0`` ，数组元素为给定类型。

以下是非定长类型：

- ``bytes`` ：动态大小的字节序列。

<<<<<<< HEAD
<<<<<<< HEAD
- ``string``: dynamic sized unicode string assumed to be UTF-8 encoded.
=======
- ``string`` ：动态大小的unicode字符串，通常呈现为UTF-8编码。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

- ``<type>[]`` ：元素为给定类型的变长数组。

<<<<<<< HEAD
<<<<<<< HEAD
Types can be combined to a tuple by enclosing a finite non-negative number
of them inside parentheses, separated by commas:
=======
可以将有限的若干类型放到一对括号中，用逗号分隔开，以此来构成一个元组：
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

- ``(T1,T2,...,Tn)`` ：由 ``T1`` ， ...， ``Tn`` ， ``n >= 0`` 构成的元组。

用元组构成元组、用元组构成数组等等也是可能的。

.. note::
<<<<<<< HEAD
    Solidity supports all the types presented above with the same names with the exception of tuples. The ABI tuple type is utilised for encoding Solidity ``structs``.
=======
可以将有限的若干类型放到一对括号中，用逗号分隔开，以此来构成一个 |tuple| ：

- ``(T1,T2,...,Tn)`` ：由 ``T1`` ， ...， ``Tn`` ， ``n >= 0`` 构成的 |tuple| 。

=======
可以将有限的若干类型放到一对括号中，用逗号分隔开，以此来构成一个 |tuple| ：

- ``(T1,T2,...,Tn)`` ：由 ``T1`` ， ...， ``Tn`` ， ``n >= 0`` 构成的 |tuple| 。

>>>>>>> etherchina/develop
用 |tuple| 构成 |tuple| 、用 |tuple| 构成数组等等也是可能的。

.. note::
    除了 |tuple| 以外，Solidity支持以上所有类型的名称。ABI |tuple| 是利用Solidity的 ``structs`` 编码得到的。
<<<<<<< HEAD
>>>>>>> etherchina/develop
=======
>>>>>>> etherchina/develop
=======
    除了元组（tuple）以外，Solidity支持以上所有类型的名称。ABI元组是利用Solidity的 ``structs`` 编码得到的。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

编码的形式化说明
====================================

我们现在来正式讲述编码，它具有如下属性，如果参数是嵌套的数组，这些属性非常有用：

属性：

  1、读取的次数取决于参数数组结构中的最大深度；也就是说，要取得 ``a_i[k][l][r]`` 需要读取4次。在先前的ABI版本中，在最糟的情况下，读取的次数会随着动态参数的总数而线性地增长。

  2、一个变量或数组元素的数据，不会被插入其他的数据，并且是可以再定位的；也就是说，它们只会使用相对的“地址”。

我们需要区分静态和动态类型。静态类型会被直接编码，动态类型则会在当前数据块之后单独分配的位置被编码。

**定义：** 以下类型被称为“动态”：

* ``bytes``
* ``string``
<<<<<<< HEAD
<<<<<<< HEAD
* ``T[]`` for any ``T``
* ``T[k]`` for any dynamic ``T`` and any ``k > 0``
* ``(T1,...,Tk)`` if any ``Ti`` is dynamic for ``1 <= i <= k``
=======
* 任意类型 `T` 的变长数组 ``T[]``
* 任意动态类型 `T` 的定长数组 ``T[k]`` （ ``k > 0`` ）
* ``Ti`` （ ``1 <= i <= k`` ）为任意动态类型的 |tuple|  ``(T1,...,Tk)``
<<<<<<< HEAD
>>>>>>> etherchina/develop
=======
>>>>>>> etherchina/develop
=======
* 任意类型 `T` 的变长数组 ``T[]``
* 任意动态类型 `T` 的定长数组 ``T[k]`` （ ``k > 0`` ）
* ``Ti`` （ ``1 <= i <= k`` ）为任意动态类型的元组 ``(T1,...,Tk)``
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

所有其他类型都被称为“静态”。

**定义：** ``len(a)`` 是一个二进制字符串 ``a`` 的字节长度。 ``len(a)`` 的类型被呈现为 ``uint256`` 。

我们把实际的编码 ``enc`` 定义为一个由ABI类型到二进制字符串的值的映射；因而，当且仅当 ``X`` 的类型是动态的， ``len(enc(X))`` （即 ``X`` 经编码后的实际长度，译者注）才会依赖于 ``X`` 的值。

**定义：** 对任意ABI值 ``X`` ，我们根据 ``X`` 的实际类型递归地定义 ``enc(X)`` 。

- ``(T1,...,Tk)`` 对于 ``k >= 0`` 且任意类型 ``T1``, ..., ``Tk``

  ``enc(X) = head(X(1)) ... head(X(k-1)) tail(X(0)) ... tail(X(k-1))``

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
  where ``X(i)`` is the ``ith`` component of the value, and
  ``head`` and ``tail`` are defined for ``Ti`` being a static type as
=======
=======
>>>>>>> etherchina/develop
  这里， ``X(i)`` 是 |tuple| 的第 ``i`` 个要素，并且
  当 ``Ti`` 为静态类型时， ``head`` 和 ``tail`` 被定义为
>>>>>>> etherchina/develop
=======
  这里， ``X(i)`` 是元组的第 ``i`` 个要素，并且
  当 ``Ti`` 为静态类型时， ``head`` 和 ``tail`` 被定义为
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

    ``head(X(i)) = enc(X(i))`` and ``tail(X(i)) = ""`` （空字符串）

  否则，比如 ``Ti`` 是动态类型时，它们被定义为

    ``head(X(i)) = enc(len(head(X(0)) ... head(X(k-1)) tail(X(0)) ... tail(X(i-1))))``
    ``tail(X(i)) = enc(X(i))``

  注意，在动态类型的情况下，由于head部分的长度仅取决于类型而非值，所以 ``head(X(i))`` 是定义明确的。它的值是从 ``enc(X)`` 的开头算起的， ``tail(X(i))`` 的起始位在 ``enc(X)`` 中的偏移量。

- ``T[k]`` 对于任意 ``T`` 和 ``k`` ：

  ``enc(X) = enc((X[0], ..., X[k-1]))``

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
  i.e. it is encoded as if it were a tuple with ``k`` elements
  of the same type.
=======
  即是说，它就像是个由相同类型的 ``k`` 个元素组成的 |tuple| 那样被编码的。
>>>>>>> etherchina/develop
=======
  即是说，它就像是个由相同类型的 ``k`` 个元素组成的 |tuple| 那样被编码的。
>>>>>>> etherchina/develop
=======
  即是说，它就像是个由相同类型的 ``k`` 个元素组成的元组那样被编码的。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

- ``T[]`` 当 ``X`` 有 ``k`` 个元素（ ``k`` 被呈现为类型 ``uint256`` ）：

  ``enc(X) = enc(k) enc([X[1], ..., X[k]])``

  即是说，它就像是个由静态大小 ``k`` 的数组那样被编码的，且由元素的个数作为前缀。

- 具有 ``k`` （呈现为类型 ``uint256`` ）长度的 ``bytes`` ：

  ``enc(X) = enc(k) pad_right(X)`` ，即是说，字节数被编码为 ``uint256`` ，紧跟着实际的 ``X`` 的字节码序列，再在前边（左边）补上可以使 ``len(enc(X))`` 成为32的倍数的最少数量的0值字节数据。

- ``string`` ：

  ``enc(X) = enc(enc_utf8(X))`` ，即是说， ``X`` 被utf-8编码，且在后续编码中将这个值解释为 ``bytes`` 类型。注意，在随后的编码中使用的长度是其utf-8编码的字符串的字节数，而不是其字符数。

- ``uint<M>`` ： ``enc(X)`` 是在 ``X`` 的大端序编码的前边（左边）补充若干0值字节以使其长度成为32的倍数。
- ``address`` ：与 ``uint160`` 的情况相同。
- ``int<M>`` ： ``enc(X)`` 是在 ``X`` 的大端序的2的补码编码的高位（左侧）添加若干字节数据以使其长度成为32的倍数；对于负数，添加值为 ``0xff`` （即8位全为1，译者注）的字节数据，对于正数，添加0值（即8位全为0，译者注）字节数据。
- ``bool`` ：与 ``uint8`` 的情况相同， ``1`` 用来表示 ``true`` ， ``0`` 表示 ``false`` 。
- ``fixed<M>x<N>`` ： ``enc(X)`` 就是 ``enc(X * 10**N)`` ，其中 ``X * 10**N`` 可以理解为 ``int256`` 。
- ``fixed`` ：与 ``fixed128x19`` 的情况相同。
- ``ufixed<M>x<N>`` ： ``enc(X)`` 就是 ``enc(X * 10**N)`` ，其中 ``X * 10**N`` 可以理解为 ``uint256`` 。
- ``ufixed`` ：与 ``ufixed128x19`` 的情况相同。
- ``bytes<M>`` ： ``enc(X)`` 就是 ``X`` 的字节序列加上为使长度称为32的倍数而添加的若干0值字节。

注意，对于任意的 ``X`` , ``len(enc(X))`` 都是32的倍数。

|function_selector| 和参数编码
=======================================

大体而言，一个以  ``a_1, ..., a_n`` 为参数的对 ``f`` 函数的调用，会被编码为

  ``function_selector(f) enc((a_1, ..., a_n))``

``f`` 的返回值 ``v_1, ..., v_k`` 会被编码为

  ``enc((v_1, ..., v_k))``

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
i.e. the values are combined into a tuple and encoded.
=======
也就是说，返回值会被组合为一个 |tuple| 进行编码。
>>>>>>> etherchina/develop
=======
也就是说，返回值会被组合为一个 |tuple| 进行编码。
>>>>>>> etherchina/develop
=======
也就是说，返回值会被组合为一个元组进行编码。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

例子
========

给定一个合约：

::

    pragma solidity ^0.4.16;

    contract Foo {
      function bar(bytes3[2]) public pure {}
      function baz(uint32 x, bool y) public pure returns (bool r) { r = x > 32 || y; }
      function sam(bytes, bool, uint[]) public pure {}
    }


这样，对于我们的例子 ``Foo`` ，如果我们想用 ``69`` 和 ``true`` 做参数调用 ``baz`` ，我们总共需要传送68字节，可以分解为：

- ``0xcdcd77c0`` ：方法ID。这源自ASCII格式的 ``baz(uint32,bool)`` 签名的Keccak哈希的前4字节。
- ``0x0000000000000000000000000000000000000000000000000000000000000045`` ：第一个参数，一个被用0值字节补充到32字节的uint32值 ``69`` 。
- ``0x0000000000000000000000000000000000000000000000000000000000000001`` ：第二个参数，一个被用0值字节补充到32字节的boolean值 ``true`` 。

合起来就是::

    0xcdcd77c000000000000000000000000000000000000000000000000000000000000000450000000000000000000000000000000000000000000000000000000000000001

它返回一个 ``bool`` 。比如它返回 ``false`` ，那么它的输出将是一个字节数组 ``0x0000000000000000000000000000000000000000000000000000000000000000`` ，一个bool值。

如果我们想用 ``["abc", "def"]`` 做参数调用 ``bar`` ，我们总共需要传送68字节，可以分解为：

- ``0xfce353f6`` ：方法ID。源自 ``bar(bytes3[2])`` 的签名。
- ``0x6162630000000000000000000000000000000000000000000000000000000000`` ：第一个参数的第一部分，一个 ``bytes3`` 值 ``"abc"`` （左对齐）。
- ``0x6465660000000000000000000000000000000000000000000000000000000000`` ：第一个参数的第二部分，一个 ``bytes3`` 值 ``"def"`` （左对齐）。

合起来就是::

    0xfce353f661626300000000000000000000000000000000000000000000000000000000006465660000000000000000000000000000000000000000000000000000000000

如果我们想用 ``"dave"`` 、 ``true`` 和 ``[1,2,3]`` 作为参数调用 ``sam`` ，我们总共需要传送292字节，可以分解为：

- ``0xa5643bf2`` ：方法ID。源自 ``sam(bytes,bool,uint256[])`` 的签名。注意， ``uint`` 被替换为了它的权威代表 ``uint256`` 。
- ``0x0000000000000000000000000000000000000000000000000000000000000060`` ：第一个参数（动态类型）的数据部分的位置，即从参数编码块开始位置算起的字节数。在这里，是 ``0x60`` 。
- ``0x0000000000000000000000000000000000000000000000000000000000000001`` ：第二个参数：boolean的true。
- ``0x00000000000000000000000000000000000000000000000000000000000000a0`` ：第三个参数（动态类型）的数据部分的位置，由字节数计量。在这里，是 ``0xa0`` 。
- ``0x0000000000000000000000000000000000000000000000000000000000000004`` ：第一个参数的数据部分，以字节数组的元素个数作为开始，在这里，是4。
- ``0x6461766500000000000000000000000000000000000000000000000000000000`` ：第一个参数的内容： ``"dave"`` 的UTF-8编码（在这里等同于ASCII编码），并在右侧（低位）用0值字节补充到32字节。
- ``0x0000000000000000000000000000000000000000000000000000000000000003`` ：第三个参数的数据部分，以数组的元素个数作为开始，在这里，是3。
- ``0x0000000000000000000000000000000000000000000000000000000000000001`` ：第三个参数的第一个数组元素。
- ``0x0000000000000000000000000000000000000000000000000000000000000002`` ：第三个参数的第二个数组元素。
- ``0x0000000000000000000000000000000000000000000000000000000000000003`` ：第三个参数的第三个数组元素。

合起来就是::

    0xa5643bf20000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000a0000000000000000000000000000000000000000000000000000000000000000464617665000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000003

动态类型的使用
====================

用参数 ``(0x123, [0x456, 0x789], "1234567890", "Hello, world!")`` 进行对函数 ``f(uint,uint32[],bytes10,bytes)`` 的调用会通过以下方式进行编码：

取得 ``sha3("f(uint256,uint32[],bytes10,bytes)")`` 的前4字节，也就是 ``0x8be65246`` 。
然后我们对所有4个参数的头部进行编码。对静态类型 ``uint256`` 和 ``bytes10`` 是可以直接传过去的值；对于动态类型 ``uint32[]`` 和 ``bytes`` ，我们使用的字节数偏移量是它们的数据区域的起始位置，由需编码的值的开始位置算起（也就是说，不计算包含了函数签名的前4字节），这就是：

 - ``0x0000000000000000000000000000000000000000000000000000000000000123`` （ ``0x123`` 补充到32字节）
 - ``0x0000000000000000000000000000000000000000000000000000000000000080`` （第二个参数的数据部分起始位置的偏移量，4*32字节，正好是头部的大小）
 - ``0x3132333435363738393000000000000000000000000000000000000000000000`` （ ``"1234567890"`` 从右边补充到32字节）
 - ``0x00000000000000000000000000000000000000000000000000000000000000e0`` （第四个参数的数据部分起始位置的偏移量 = 第一个动态参数的数据部分起始位置的偏移量 + 第一个动态参数的数据部分的长度 = 4\*32 + 3\*32，参考后文）

在此之后，跟着第一个动态参数的数据部分 ``[0x456, 0x789]`` ：

 - ``0x0000000000000000000000000000000000000000000000000000000000000002`` （数组元素个数，2）
 - ``0x0000000000000000000000000000000000000000000000000000000000000456`` （第一个数组元素）
 - ``0x0000000000000000000000000000000000000000000000000000000000000789`` （第二个数组元素）

最后，我们将第二个动态参数的数据部分 ``"Hello, world!"`` 进行编码：

 - ``0x000000000000000000000000000000000000000000000000000000000000000d`` （元素个数，在这里是字节数：13）
 - ``0x48656c6c6f2c20776f726c642100000000000000000000000000000000000000`` （ ``"Hello, world!"`` 从右边补充到32字节）

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
All together, the encoding is (newline after function selector and each 32-bytes for clarity):
=======
最后，合并到一起的编码就是（为了清晰，在 |function_selector| 和每32字节之后加了换行）：
>>>>>>> etherchina/develop
=======
最后，合并到一起的编码就是（为了清晰，在 |function_selector| 和每32字节之后加了换行）：
>>>>>>> etherchina/develop
=======
最后，合并到一起的编码就是（为了清晰，在函数选择器和每32字节之后加了换行）：
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

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

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
Events are an abstraction of the Ethereum logging/event-watching protocol. Log entries provide the contract's address, a series of up to four topics and some arbitrary length binary data. Events leverage the existing function ABI in order to interpret this (together with an interface spec) as a properly typed structure.
=======
事件，是 |ethereum| 的日志/事件监视协议的一个抽象。日志项提供了合约的地址、一系列的主题（最高4项）和一些任意长度的二进制数据。为了使用合适的类型数据结构来演绎这些功能（与接口定义一起），事件沿用了既存的ABI函数。
>>>>>>> etherchina/develop
=======
事件，是 |ethereum| 的日志/事件监视协议的一个抽象。日志项提供了合约的地址、一系列的主题（最高4项）和一些任意长度的二进制数据。为了使用合适的类型数据结构来演绎这些功能（与接口定义一起），事件沿用了既存的ABI函数。
>>>>>>> etherchina/develop
=======
事件，是以太坊的日志/事件监视协议的一个抽象。日志项提供了合约的地址、一系列的主题（最高4项）和一些任意长度的二进制数据。为了使用合适的类型数据结构来演绎这些功能（与接口定义一起），事件沿用了既存的ABI函数。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

给定了事件名称和事件参数之后，我们将其分解为两个子集：已索引的和未索引的。已索引的部分，最多有3个，被用来与事件签名的Keccak哈希一起组成日志项的主题。未索引的部分就组成了事件的字节数组。

这样，一个使用ABI的日志项就可以描述为：

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
- ``address``: the address of the contract (intrinsically provided by Ethereum);
- ``topics[0]``: ``keccak(EVENT_NAME+"("+EVENT_ARGS.map(canonical_type_of).join(",")+")")`` (``canonical_type_of`` is a function that simply returns the canonical type of a given argument, e.g. for ``uint indexed foo``, it would return ``uint256``). If the event is declared as ``anonymous`` the ``topics[0]`` is not generated;
- ``topics[n]``: ``EVENT_INDEXED_ARGS[n - 1]`` (``EVENT_INDEXED_ARGS`` is the series of ``EVENT_ARGS`` that are indexed);
- ``data``: ``abi_serialise(EVENT_NON_INDEXED_ARGS)`` (``EVENT_NON_INDEXED_ARGS`` is the series of ``EVENT_ARGS`` that are not indexed, ``abi_serialise`` is the ABI serialisation function used for returning a series of typed values from a function, as described above).
=======
=======
>>>>>>> etherchina/develop
- ``address`` ：合约地址（由 |ethereum| 真正提供）；
- ``topics[0]`` ： ``keccak(EVENT_NAME+"("+EVENT_ARGS.map(canonical_type_of).join(",")+")")`` （ ``canonical_type_of`` 是一个可以返回给定参数的权威类型的函数，例如，对 ``uint indexed foo`` 它会返回 ``uint256`` ）。如果事件被声明为 ``anonymous`` ，那么 ``topics[0]`` 不会被生成；
- ``topics[n]`` ： ``EVENT_INDEXED_ARGS[n - 1]`` （ ``EVENT_INDEXED_ARGS`` 是已索引的 ``EVENT_ARGS`` ）；
- ``data`` ： ``abi_serialise(EVENT_NON_INDEXED_ARGS)`` （ ``EVENT_NON_INDEXED_ARGS`` 是未索引的 ``EVENT_ARGS`` ， ``abi_serialise`` 是一个用来从某个函数返回一系列类型值的ABI序列化函数，就像上文所讲的那样）。
>>>>>>> etherchina/develop
=======
- ``address`` ：合约地址（由以太坊真正提供）；
- ``topics[0]`` ： ``keccak(EVENT_NAME+"("+EVENT_ARGS.map(canonical_type_of).join(",")+")")`` （ ``canonical_type_of`` 是一个可以返回给定参数的权威类型的函数，例如，对 ``uint indexed foo`` 它会返回 ``uint256`` ）。如果事件被声明为 ``anonymous`` ，那么 ``topics[0]`` 不会被生成；
- ``topics[n]`` ： ``EVENT_INDEXED_ARGS[n - 1]`` （ ``EVENT_INDEXED_ARGS`` 是已索引的 ``EVENT_ARGS`` ）；
- ``data`` ： ``abi_serialise(EVENT_NON_INDEXED_ARGS)`` （ ``EVENT_NON_INDEXED_ARGS`` 是未索引的 ``EVENT_ARGS`` ， ``abi_serialise`` 是一个用来从某个函数返回一系列类型值的ABI序列化函数，就像上文所讲的那样）。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

对于所有定长的Solidity类型，  ``EVENT_INDEXED_ARGS`` 数组会直接包含32字节的编码值。然而，对于 *动态长度的类型* ，包含 ``string`` 、 ``bytes`` 和数组，
``EVENT_INDEXED_ARGS`` 会包含编码值的 *Keccak哈希* 而不是直接包含编码值。这样就允许应用程序更有效地查询动态长度类型的值（通过把编码值的哈希设定为主题），
但也使应用程序不能对它们还没查询过的已索引的值进行解码。对于动态长度的类型，应用程序开发者面临在对预先设定的值（如果参数已被索引）的快速检索和对任意数据的清晰处理（需要参数不被索引）之间的权衡。
开发者们可以通过定义两个参数（一个已索引、一个未索引）保存同一个值的方式来解决这种权衡，从而既获得高效的检索又能清晰地处理任意数据。

JSON
=======

合约接口的JSON格式是由一个函数和/或事件描述的数组所给定的。一个函数的描述是一个有如下字段的JSON对象：

- ``type`` ： ``"function"`` 、 ``"constructor"`` 或 ``"fallback"`` （ :ref:`未命名的 "缺省" 函数 <fallback-function>` ）
- ``name`` ：函数名称；
- ``inputs`` ：对象数组，每个数组对象会包含：

<<<<<<< HEAD
<<<<<<< HEAD
  * ``name``: the name of the parameter;
  * ``type``: the canonical type of the parameter (more below).
  * ``components``: used for tuple types (more below).

- ``outputs``: an array of objects similar to ``inputs``, can be omitted if function doesn't return anything;
- ``payable``: ``true`` if function accepts ether, defaults to ``false``;
- ``stateMutability``: a string with one of the following values: ``pure`` (:ref:`specified to not read blockchain state <pure-functions>`), ``view`` (:ref:`specified to not modify the blockchain state <view-functions>`), ``nonpayable`` and ``payable`` (same as ``payable`` above).
- ``constant``: ``true`` if function is either ``pure`` or ``view``
=======
  * ``name`` ：参数名称；
  * ``type`` ：参数的权威类型（详见下文）
  * ``components`` ：供 |tuple| 类型使用（详见下文）

- ``outputs`` ：一个类似于 ``inputs`` 的对象数组，如果函数无返回值时可以被省略；
- ``payable`` ：如果函数接受 |ether| ，为 ``true`` ；缺省为 ``false`` ；
- ``stateMutability`` ：为下列值之一： ``pure`` （ :ref:`指定为不读取区块链状态 <pure-functions>` ）， ``view`` （ :ref:`指定为不修改区块链状态 <view-functions>` ）， ``nonpayable`` 和 ``payable`` （与上文 ``payable`` 一样）。
- ``constant`` ：如果函数被指定为 ``pure`` 或 ``view`` 则为 ``true`` 。
>>>>>>> etherchina/develop
=======
  * ``name`` ：参数名称；
  * ``type`` ：参数的权威类型（详见下文）
  * ``components`` ：供元组类型使用（详见下文）

- ``outputs`` ：一个类似于 ``inputs`` 的对象数组，如果函数无返回值时可以被省略；
- ``payable`` ：如果函数接受以太币，为 ``true`` ；缺省为 ``false`` ；
- ``stateMutability`` ：为下列值之一： ``pure`` （ :ref:`指定为不读取区块链状态 <pure-functions>` ）， ``view`` （ :ref:`指定为不修改区块链状态 <view-functions>` ）， ``nonpayable`` 和 ``payable`` （与上文 ``payable`` 一样）。
- ``constant`` ：如果函数被指定为 ``pure`` 或 ``view`` 则为 ``true`` 。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

``type`` 可以被省略，缺省为 ``"function"`` 。

Constructor和fallback函数没有 ``name`` 或 ``outputs`` 。Fallback函数也没有 ``inputs`` 。

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
Sending non-zero ether to non-payable function will throw. Don't do it.
=======
向non-payable（即不接受 |ether| ）的函数发送非零值的 |ether| 会导致其丢失。不要这么做。
>>>>>>> etherchina/develop
=======
向non-payable（即不接受 |ether| ）的函数发送非零值的 |ether| 会导致其丢失。不要这么做。
>>>>>>> etherchina/develop
=======
向non-payable（即不接受以太币）的函数发送非零值的以太币会导致其丢失。不要这么做。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

一个事件描述是一个有极其相似字段的JSON对象：

- ``type`` ：总是 ``"event"`` ；
- ``name`` ：事件名称；
- ``inputs`` ：对象数组，每个数组对象会包含：

<<<<<<< HEAD
<<<<<<< HEAD
  * ``name``: the name of the parameter;
  * ``type``: the canonical type of the parameter (more below).
  * ``components``: used for tuple types (more below).
  * ``indexed``: ``true`` if the field is part of the log's topics, ``false`` if it one of the log's data segment.
=======
  * ``name`` ：参数名称；
  * ``type`` ：参数的权威类型（相见下文）；
  * ``components`` ：供 |tuple| 类型使用（详见下文）；
  * ``indexed`` ：如果此字段是日志的一个主题，则为 ``true`` ；否则为 ``false`` 。
>>>>>>> etherchina/develop
=======
  * ``name`` ：参数名称；
  * ``type`` ：参数的权威类型（相见下文）；
  * ``components`` ：供元组类型使用（详见下文）；
  * ``indexed`` ：如果此字段是日志的一个主题，则为 ``true`` ；否则为 ``false`` 。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

- ``anonymous`` ：如果事件被声明为 ``anonymous`` ，则为 ``true`` 。

例如，

::

    pragma solidity ^0.4.0;

    contract Test {
      function Test() public { b = 0x12345678901234567890123456789012; }
      event Event(uint indexed a, bytes32 b);
      event Event2(uint indexed a, bytes32 b);
      function foo(uint a) public { Event(a, b); }
      bytes32 b;
    }

可由如下JSON来表示：

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

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
Handling tuple types
=======
处理 |tuple| 类型
>>>>>>> etherchina/develop
=======
处理 |tuple| 类型
>>>>>>> etherchina/develop
=======
处理元组类型
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"
--------------------

尽管名称被有意地不作为ABI编码的一部分，但将它们包含进JSON来显示给最终用户是非常合理的。其结构会按下列方式进行嵌套：

<<<<<<< HEAD
<<<<<<< HEAD
An object with members ``name``, ``type`` and potentially ``components`` describes a typed variable.
The canonical type is determined until a tuple type is reached and the string description up
to that point is stored in ``type`` prefix with the word ``tuple``, i.e. it will be ``tuple`` followed by
a sequence of ``[]`` and ``[k]`` with
integers ``k``. The components of the tuple are then stored in the member ``components``,
which is of array type and has the same structure as the top-level object except that
``indexed`` is not allowed there.
=======
一个拥有 ``name`` 、 ``type`` 和潜在的 ``components`` 成员的对象描述了某种类型的变量。
直至到达一个 |tuple| 类型且到那点的存储在 ``type`` 属性中的字符串以 ``tuple`` 为前缀，也就是说，在 ``tuple`` 之后紧跟一个 ``[]`` 或有整数 ``k`` 的 ``[k]`` ，才能确定一个 |tuple| 。
|tuple| 的组件元素会被存储在成员 ``components`` 中，它是一个数组类型，且与顶级对象具有同样的结构，只是在这里不允许已索引的（ ``indexed`` ）数组元素。
<<<<<<< HEAD
>>>>>>> etherchina/develop
=======
>>>>>>> etherchina/develop
=======
一个拥有 ``name`` 、 ``type`` 和潜在的 ``components`` 成员的对象描述了某种类型的变量。
直至到达一个元组类型且到那点的存储在 ``type`` 属性中的字符串以 ``tuple`` 为前缀，也就是说，在 ``tuple`` 之后紧跟一个 ``[]`` 或有整数 ``k`` 的 ``[k]`` ，才能确定一个元组。
元组的组件元素会被存储在成员 ``components`` 中，它是一个数组类型，且与顶级对象具有同样的结构，只是在这里不允许已索引的（ ``indexed`` ）数组元素。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

作为例子，代码

::

    pragma solidity ^0.4.19;
    pragma experimental ABIEncoderV2;

    contract Test {
      struct S { uint a; uint[] b; T[] c; }
      struct T { uint x; uint y; }
      function f(S s, T t, uint a) public { }
      function g() public returns (S s, T t, uint a) {}
    }

可由如下JSON来表示：

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

Solidity支持一种非标准打包模式：

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
- no :ref:`function selector <abi_function_selector>` is encoded,
- types shorter than 32 bytes are neither zero padded nor sign extended and
- dynamic types are encoded in-place and without the length.
=======
=======
>>>>>>> etherchina/develop
- :ref:`函数选择器<abi_function_selector>` 不进行编码，
- 长度低于32字节的类型，既不会进行补0操作，也不会进行符号扩展，以及
- 动态类型会直接进行编码，并且不包含长度信息。
>>>>>>> etherchina/develop
=======
- :ref:`函数选择器 <abi_function_selector>` 不进行编码，
- 长度低于32字节的类型，既不会进行补0操作，也不会进行符号扩展，以及
- 动态类型会直接进行编码，并且不包含长度信息。
>>>>>>> parent of a49b1d9... Revert "Merge remote-tracking branch 'etherchina/develop' into develop"

例如，对 ``int1, bytes1, uint16, string`` 用数值 ``-1, 0x42, 0x2424, "Hello, world!"`` 进行编码将生成如下结果 ::

    0xff42242448656c6c6f2c20776f726c6421
      ^^                                 int1(-1)
        ^^                               bytes1(0x42)
          ^^^^                           uint16(0x2424)
              ^^^^^^^^^^^^^^^^^^^^^^^^^^ string("Hello, world!") without a length field

更具体地说，每个静态大小的类型都尽可能多地按它们的数值范围使用了字节数，而动态大小的类型，像 ``string`` 、 ``bytes`` 或 ``uint[]`` ，在编码时没有包含其长度信息。
这意味着一旦有两个动态长度的元素，编码就会变得有歧义了。
