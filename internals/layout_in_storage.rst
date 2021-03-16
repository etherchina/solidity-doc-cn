.. include:: ../glossaries.rst

.. index:: storage, state variable, mapping

************************************
状态变量储存结构
************************************

.. _storage-inplace-encoding:

静态大小的变量（除 |mapping| 和动态数组之外的所有类型）都从位置 ``0`` 开始连续放置在 |storage| 中。如果可能的话，存储大小少于 32 字节的多个变量会被打包到一个 |storage_slot| 中，规则如下：

- |storage_slot| 的第一项会以低位对齐（即右对齐）的方式储存。
- 基本类型仅使用存储它们所需的字节。
- 如果 |storage_slot| 中的剩余空间不足以储存一个基本类型，那么它会被移入下一个 |storage_slot| 。
- 结构体（struct）和数组数据总是会占用一整个新插槽（但结构体或数组中的各项，都会以这些规则进行打包）。

对于使用继承的合约，状态变量的排序由C3线性化合约顺序（ 顺序从最基类合约开始）确定。如果上述规则成立，那么来自不同的合约的状态变量会共享一个 |storage_slot| 。

结构体和数组中的成员变量会存储在一起，就像它们在显式声明中的一样。

.. warning::
    在使用小于 32 字节的元素（变量）时，合约的 gas 使用量可能会高于使用 32 字节的元素。这是因为 |evm| 每次操作 32 个字节，
    所以如果元素比 32 字节小，|evm| 必须执行额外的操作以便将其大小缩减到到所需的大小。

    当我们在处理状态变量时，只有当编译器会将多个元素打包到一个 |storage_slot| 中，使用缩减的大小（小于32字节）的变量才更有益处。因为它会将多个读或写合并为单次操作。
    而在处理函数参数或 |memory| 中的值时，因为编译器不会打包这些值，所以没有什么益处。

    最后，为了允许 |evm| 对此进行优化，请确保 |storage| 中的变量和 ``struct`` 成员的书写顺序允许它们被紧密地打包。
    例如，应该按照 ``uint128，uint128，uint256`` 的顺序来声明状态变量，而不是使用 ``uint128，uint256，uint128`` ，
    因为前者只占用两个 |storage_slot|，而后者将占用三个。

.. note::

    由于 |storage| 中的指针可以传递给库（library） ，所以 |storage| 中状态变量的布局被认为是 solidity 外部接口的一部分。这意味着，本节所述规则的任何变更均被视为语言的重大变更，由于其关键性，请在执行前应仔细考虑。



映射和动态数组
===========================

.. _storage-hashed-encoding:

由于 |mapping| 和动态数组的大小是不可预知的，他们使用 Keccak-256 哈希计算来找到值的位置或数组的起始位置。
这些起始位置本身的数值总是会占满堆栈插槽。

|mapping| 或动态数组本身会根据上述规则来在某个位置 ``p`` 处占用一个（未填充的）存储中的插槽（或递归地将该规则应用到 |mapping| 的 |mapping| 或数组的数组）。
对于动态数组，此插槽中会存储数组中元素的数量（字节数组和字符串除外，:ref:`见下文 <bytes-and-string>`）。

对于 |mapping| ，该插槽未被使用（但它仍是需要的，以使两个相同的 |mapping| 在彼此之后会使用不同的散列分布）。数组的数据会位于 ``keccak256(p)`` ； |mapping| 中的键 ``k`` 所对应的值会位于 ``keccak256(k . p)`` ，
其中 ``.`` 是连接符。如果该值又是一个非基本类型，则通过添加 ``keccak256(k . p)`` 作为偏移量来找到位置。

所以对于以下合约片段``data[4][9].b`` 的位置将是 ``keccak256(uint256(9) . keccak256(uint256(4) . uint256(1))) + 1`` 。


    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;


    contract C {
        struct S { uint a; uint b; }
        uint x;
        mapping(uint => mapping(uint => S)) data;
    }

.. _bytes-and-string:

``bytes`` 和 ``string``
------------------------

``bytes`` 和 ``string`` 编码是一样的。 对于短字节数组，它们将数据和它们的长度存储在同一个插槽中。 具体地说：如果数据长度小于等于  ``31`` 字节，则它存储在高位字节（左对齐），最低位字节存储 ``length * 2``。
如果数据长度超出 31 字节，则在主插槽存储 ``length * 2 + 1`` ，数据照常存储在 ``keccak256(slot)`` 中。

这意味着可以通过检查是否设置了最低位：短（未设置）和长（设置）来区分短数组和长数组。

.. note::
  目前不支持处理无效编码的插槽，但可能在将来添加。

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


.. code::

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

.. code::

    "storageLayout": {
      "storage": [
        {
          "astId": 14,
          "contract": "fileA:A",
          "label": "x",
          "offset": 0,
          "slot": "0",
          "type": "t_uint256"
        },
        {
          "astId": 16,
          "contract": "fileA:A",
          "label": "y",
          "offset": 0,
          "slot": "1",
          "type": "t_uint256"
        },
        {
          "astId": 18,
          "contract": "fileA:A",
          "label": "s",
          "offset": 0,
          "slot": "2",
          "type": "t_struct(S)12_storage"
        },
        {
          "astId": 20,
          "contract": "fileA:A",
          "label": "addr",
          "offset": 0,
          "slot": "6",
          "type": "t_address"
        },
        {
          "astId": 26,
          "contract": "fileA:A",
          "label": "map",
          "offset": 0,
          "slot": "7",
          "type": "t_mapping(t_uint256,t_mapping(t_address,t_bool))"
        },
        {
          "astId": 29,
          "contract": "fileA:A",
          "label": "array",
          "offset": 0,
          "slot": "8",
          "type": "t_array(t_uint256)dyn_storage"
        },
        {
          "astId": 31,
          "contract": "fileA:A",
          "label": "s1",
          "offset": 0,
          "slot": "9",
          "type": "t_string_storage"
        },
        {
          "astId": 33,
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
        "t_struct(S)12_storage": {
          "encoding": "inplace",
          "label": "struct A.S",
          "members": [
            {
              "astId": 2,
              "contract": "fileA:A",
              "label": "a",
              "offset": 0,
              "slot": "0",
              "type": "t_uint128"
            },
            {
              "astId": 4,
              "contract": "fileA:A",
              "label": "b",
              "offset": 16,
              "slot": "0",
              "type": "t_uint128"
            },
            {
              "astId": 8,
              "contract": "fileA:A",
              "label": "staticArray",
              "offset": 0,
              "slot": "1",
              "type": "t_array(t_uint256)2_storage"
            },
            {
              "astId": 11,
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
