.. include:: ../glossaries.rst

.. index:: storage, state variable, mapping

************************************
状态变量在储存中的布局
************************************

.. _storage-inplace-encoding:

合约的状态变量以一种紧凑的方式存储在区块链存储中，以这样的方式，有时多个值会使用同一个存储槽。
除了动态大小的数组和 |mapping| （见下文），数据的存储方式是从位置 ``0`` 开始连续放置在 |storage| 中。
对于每个变量，根据其类型确定字节大小。

存储大小少于 32 字节的多个变量会被打包到一个 |storage_slot| 中，规则如下：

- |storage_slot| 的第一项会以低位对齐的方式储存。
- 值类型仅使用存储它们所需的字节。
- 如果 |storage_slot| 中的剩余空间不足以储存一个值类型，那么它会被存入下一个 |storage_slot| 。
- 结构体（struct）和数组数据总是会开启一个新插槽（但结构体或数组中的各元素，则按规则紧密打包）。
- 结构体和数组之后的数据也或开启一个新插槽。

对于使用继承的合约，状态变量的排序由C3线性化合约顺序（ 顺序从最基类合约开始）确定。如果上述规则成立，那么来自不同的合约的状态变量会共享一个 |storage_slot| 。

结构体和数组中的成员变量会存储在一起，就像它们单独声明时一样。

.. warning::
    在使用小于 32 字节的元素（变量）时，合约的 gas 使用量可能会高于使用 32 字节的元素。这是因为 |evm| 每次操作 32 个字节，
    所以如果元素比 32 字节小，|evm| 必须执行额外的操作以便将其大小缩减到到所需的大小。

    当我们在处理状态变量时，利用编译器会将多个元素缩减的存储大小打包到一个 |storage_slot| 中，也许是有益，因为可以合并多次读写为单个操作。
    
    如果你不是在同一时间读或写一个槽中的所有值，这可能会适得其反。
    当一个值被写入一个多值存储槽时，必须先读取该存储槽，然后将其与新值合并，避免破坏同一槽中的其他数据，再写入。

    当处理函数参数或 |memory| 中的值时，因为编译器不会打包这些值，所以没有什么额外的益处。

    最后，为了允许 |evm| 对此进行优化，请确保 |storage| 中的变量和 ``struct`` 成员的书写顺序允许它们被紧密地打包。
    例如，应该按照 ``uint128，uint128，uint256`` 的顺序来声明状态变量，而不是使用 ``uint128，uint256，uint128`` ，
    因为前者只占用两个 |storage_slot|，而后者将占用三个。

.. note::

    |storage| 中状态变量的布局被认为是 solidity 外部接口的一部分， 因此 |storage| 变量指针可以传递给库（library）函数。
    这意味着，本节所述规则的任何变更均被视为语言破坏性变更，并且由于其关键性质，在执行之前应该非常仔细地考虑，在发生这种破坏性变化的情况下，我们希望发布一种兼容模式，在这种模式下，编译器将生成支持旧布局的字节码。



映射和动态数组
===========================

.. _storage-hashed-encoding:

由于 |mapping| 和动态数组不可预知大小，不能在状态变量之间存储他们。相反，他们自身根据 :ref:`以上规则 <storage-inplace-encoding>` 仅占用 32 个字节，然后他们包含的元素的存储的其实位置，则是通过 Keccak-256 哈希计算来确定。

假设 |mapping| 或动态数组根据上述存储规则最终可确定某个位置 ``p`` 。
对于动态数组，此插槽中会存储数组中元素的数量（字节数组和字符串除外，:ref:`见下文 <bytes-and-string>`）。
对于 |mapping| ，该插槽未被使用（为空），但它仍是需要的，以确保两个彼此挨着 |mapping| ，他们的内容在不同的位置上。

数组的元素会从 ``keccak256(p)`` 开始； 它的布局方式与静态大小的数组相同。一个元素接着一个元素，如果元素的长度不超过16字节，就有可能共享存储槽。

动态数组的数组会递归地应用这一规则，例如，如何确定 ``x[i][j]`` 元素的位置，其中 ``x`` 的类型是 ``uint24[][]``，计算方法如下（假设 ``x``本身存储在槽 ``p``）:
槽位于 ``keccak256(keccak256(p) + i) + floor(j / floor(256 / 24))`` 且可以从槽数据 ``v``得到元素内容，使用 ``(v >> ((j % floor(256 / 24)) * 24)) & type(uint24).max``.

|mapping| 中的键 ``k`` 所对应的槽会位于 ``keccak256(h(k) . p)`` ，其中 ``.`` 是连接符， ``h`` 是一个函数，根据键的类型：

- 值类型， ``h`` 与在内存中存储值的方式相同的方式将值填充为32字节。
- 对于字符串和字节数组， ``h(k)`` 只是未填充的数据。

如果映射值是一个非值类型，计算槽位置标志着数据的开始位置。例如，如果值是结构类型，你必须添加一个与结构成员相对应的偏移量才能到达该成员。

例如，考虑下面的合约：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;


    contract C {
        struct S { uint16 a; uint16 b; uint256 c; }
        uint x;
        mapping(uint => mapping(uint => S)) data;
    }


让我们计算一下 ``data[4][9].c`` 的存储位置。映射本身的位置是 ``1``（ 前面有32字节变量 ``x`` ）。
因此 ``data[4]`` 存储在 ``keccak256(uint256(4) . uint256(1))``。 ``data[4]`` 的类型又是一个映射， ``data[4][9]`` 的数据开始于槽位
``keccak256(uint256(9). keccak256(uint256(4). uint256(1))``。

在结构 ``S`` 的成员 ``c`` 中的槽位偏移是 ``1``，因为 ``a`` 和 ``b``被装在一个槽位中。
最后 ``data[4][9].c`` 的插槽位置是 ``keccak256(uint256(9) . keccak256(uint256(4) . uint256(1)) + 1``.
该值的类型是 ``uint256``，所以它使用一个槽。



.. _bytes-and-string:

``bytes`` 和 ``string``
------------------------

``bytes`` 和 ``string`` 编码是一样的。 

一般来说，编码与 ``bytes1[]`` 类似，即有一个槽用于存放数组本身同时还有一个数据区，数据区位置使用槽的 ``keccak256`` hash计算。
然而，对于短字节数组（短于32字节），数组元素与长度一起存储在同一个槽中。

具体地说：如果数据长度小于等于 ``31`` 字节，则元素存储在高位字节（左对齐），最低位字节存储值 ``length * 2``。
如果数据长度大于等于 32 字节，则在主插槽 ``p`` 存储 ``length * 2 + 1`` ，数据照常存储在 ``keccak256(p)`` 中。
因此，可以通过检查是否设置了最低位：短（未设置最低位）和长（设置最低位）来区分短数组和长数组。

.. note::
  目前不支持处理无效编码的插槽，但可能在将来添加。
  如果你通过IR编译，读取一个无效的编码槽会导致 ``Panic(0x22)`` 错误。

JSON 输出
===========

.. _storage-layout-top-level:

合约的存储布局可以通过 :ref:`standard JSON interface <compiler-api>` 获取到。
输出JSON对象包含 2 个字段 ``storage`` 和 ``types`` 。  

``storage`` 对象时一个数组，它的每个元素有如下的形式：


.. code::


    {
        "astId": 2,
        "contract": "fileA:A",
        "label": "x",
        "offset": 0,
        "slot": "0",
        "type": "t_uint256"
    }

上面是文件： ``fileA`` 合约： ``contract A { uint x; }`` 存储布局，每个字段说明如下：


- ``astId`` 是状态变量声明的AST节点的id。
- ``contract`` 是合约的名称，包括其路径作为前缀。
- ``label`` 是状态变量的名称。
- ``offset`` 是根据编码在存储槽内以字节为单位的偏移量。
- ``slot`` 是状态变量所在或开始的存储槽。这个数字可能非常大，因此它的JSON值被表示为一个字符串。
- ``type`` 是一个标识符，作为变量类型信息的关键(如下所述)。


给定的 ``type``，在本例中 ``t_uint256`` 代表 ``types`` 中的一个元素，其形式为：


.. code::

    {
        "encoding": "inplace",
        "label": "uint256",
        "numberOfBytes": "32",
    }

而

- ``encoding`` 数据在存储中如何编码，可能的数值是：

  - ``inplace``: 数据在存储中连续排列 (见 :ref:`前面状态变量储存结构 <storage-inplace-encoding>`).
  - ``mapping``: Keccak-256基于哈希的方法 (见 :ref:`前面前面映射和动态数组 <storage-hashed-encoding>`).
  - ``dynamic_array``: Keccak-256基于哈希的方法 (见 :ref:`前面映射和动态数组 <storage-hashed-encoding>`).
  - ``bytes``: 单槽或基于Keccak-256哈希的方法，取决于数据大小 (见 :ref:`前面bytes <bytes-and-string>`).

- ``label`` 是规范的类型名称 。
- ``numberOfBytes`` 是使用的字节数(十进制字符串)
  注意，如果 ``numberOfBytes>32`` 意味着使用了一个以上的槽。

除了上述四个外，有些类型还有额外的信息。映射包含其 ``key`` 和 ``value`` 类型(再次引用该类型映射中元素类型)，数组有其 ``base`` 类型，结构以与顶层 ``storage`` 相同的格式列出其 ``members`` (见 :ref:`前面JSON 输出 <storage-layout-top-level>`).

.. note ::
  合约的存储布局的JSON输出格式仍被认为是实验性的，即使在Solidity的非突破性版本更新中也可能会发生变化。


下面的例子显示了一个合约和它的存储布局，包含值类型和引用类型、被编码打包的类型和嵌套类型。


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;
    contract A {
        struct S {
            uint128 a;
            uint128 b;
            uint[2] staticArray;
            uint[] dynArray;
        }

        uint x;
        uint y;
        S s;
        address addr;
        mapping (uint => mapping (address => bool)) map;
        uint[] array;
        string s1;
        bytes b1;
    }

.. code:: json

    {
      "storage": [
        {
          "astId": 15,
          "contract": "fileA:A",
          "label": "x",
          "offset": 0,
          "slot": "0",
          "type": "t_uint256"
        },
        {
          "astId": 17,
          "contract": "fileA:A",
          "label": "y",
          "offset": 0,
          "slot": "1",
          "type": "t_uint256"
        },
        {
          "astId": 20,
          "contract": "fileA:A",
          "label": "s",
          "offset": 0,
          "slot": "2",
          "type": "t_struct(S)13_storage"
        },
        {
          "astId": 22,
          "contract": "fileA:A",
          "label": "addr",
          "offset": 0,
          "slot": "6",
          "type": "t_address"
        },
        {
          "astId": 28,
          "contract": "fileA:A",
          "label": "map",
          "offset": 0,
          "slot": "7",
          "type": "t_mapping(t_uint256,t_mapping(t_address,t_bool))"
        },
        {
          "astId": 31,
          "contract": "fileA:A",
          "label": "array",
          "offset": 0,
          "slot": "8",
          "type": "t_array(t_uint256)dyn_storage"
        },
        {
          "astId": 33,
          "contract": "fileA:A",
          "label": "s1",
          "offset": 0,
          "slot": "9",
          "type": "t_string_storage"
        },
        {
          "astId": 35,
          "contract": "fileA:A",
          "label": "b1",
          "offset": 0,
          "slot": "10",
          "type": "t_bytes_storage"
        }
      ],
      "types": {
        "t_address": {
          "encoding": "inplace",
          "label": "address",
          "numberOfBytes": "20"
        },
        "t_array(t_uint256)2_storage": {
          "base": "t_uint256",
          "encoding": "inplace",
          "label": "uint256[2]",
          "numberOfBytes": "64"
        },
        "t_array(t_uint256)dyn_storage": {
          "base": "t_uint256",
          "encoding": "dynamic_array",
          "label": "uint256[]",
          "numberOfBytes": "32"
        },
        "t_bool": {
          "encoding": "inplace",
          "label": "bool",
          "numberOfBytes": "1"
        },
        "t_bytes_storage": {
          "encoding": "bytes",
          "label": "bytes",
          "numberOfBytes": "32"
        },
        "t_mapping(t_address,t_bool)": {
          "encoding": "mapping",
          "key": "t_address",
          "label": "mapping(address => bool)",
          "numberOfBytes": "32",
          "value": "t_bool"
        },
        "t_mapping(t_uint256,t_mapping(t_address,t_bool))": {
          "encoding": "mapping",
          "key": "t_uint256",
          "label": "mapping(uint256 => mapping(address => bool))",
          "numberOfBytes": "32",
          "value": "t_mapping(t_address,t_bool)"
        },
        "t_string_storage": {
          "encoding": "bytes",
          "label": "string",
          "numberOfBytes": "32"
        },
        "t_struct(S)13_storage": {
          "encoding": "inplace",
          "label": "struct A.S",
          "members": [
            {
              "astId": 3,
              "contract": "fileA:A",
              "label": "a",
              "offset": 0,
              "slot": "0",
              "type": "t_uint128"
            },
            {
              "astId": 5,
              "contract": "fileA:A",
              "label": "b",
              "offset": 16,
              "slot": "0",
              "type": "t_uint128"
            },
            {
              "astId": 9,
              "contract": "fileA:A",
              "label": "staticArray",
              "offset": 0,
              "slot": "1",
              "type": "t_array(t_uint256)2_storage"
            },
            {
              "astId": 12,
              "contract": "fileA:A",
              "label": "dynArray",
              "offset": 0,
              "slot": "3",
              "type": "t_array(t_uint256)dyn_storage"
            }
          ],
          "numberOfBytes": "128"
        },
        "t_uint128": {
          "encoding": "inplace",
          "label": "uint128",
          "numberOfBytes": "16"
        },
        "t_uint256": {
          "encoding": "inplace",
          "label": "uint256",
          "numberOfBytes": "32"
        }
      }
    }
