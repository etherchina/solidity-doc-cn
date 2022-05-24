
.. index:: ! type;conversion, ! cast

.. _types-conversion-elementary-types:

基本类型之间的转换
==================

隐式转换
---------


在某些情况下，编译器会自动进行隐式类型转换， 这些情况包括: 在赋值, 参数传递给函数以及应用运算符时。
通常，如果可以进行值类型之间的隐式转换， 并且不会丢失任何信息。 都是可以隐式类型转换


例如, ``uint8`` 可以转换成 ``uint16``， ``int128`` 转换成 ``int256``，但 ``int8`` 不能转换成 ``uint256``
（因为 ``uint256`` 不能涵盖某些值，例如， ``-1``）。


如果将运算符应用于不同的类型，则编译器将尝试将其中一个操作数隐式转换为另一个操作数的类型（赋值也是如此）。
这意味着操作始终以操作数之一的类型执行。


有关可能进行哪些隐式转换的更多详细信息， 请查阅有关类型本身的内容。


在下面的示例中，加法的操作数 ``y`` 和 ``z``  没有相同的类型，但是 ``uint8`` 可以被隐式转换为 ``uint16``，相反却不可以。 因此，
在执行加法之前，将 ``y`` 转换为  ``z`` 的类型，　在 ``uint16``　类型中。 表达式　 ``y + z`` 　的结果类型是　``uint16``　。
在执行加法之后。　因为它被赋值给　``uint32``　类型的变量，又进行了另一个隐式转换.


.. code-block:: solidity

    uint8 y;
    uint16 z;
    uint32 x = y + z;


显式转换
---------

如果某些情况下编译器不支持隐式转换，但是你很清楚你要做的结果，这种情况可以考虑显式转换。
注意这可能会发生一些无法预料的后果，因此一定要进行测试，确保结果是你想要的！
下面的示例是将一个 ``int8`` 类型的负数转换成 ``uint``：

.. code-block:: solidity

    int8 y = -3;
    uint x = uint(y);

这段代码的最后， ``x`` 的值将是 ``0xfffff..fd`` （64 个 16 进制字符），因为这是 -3 的 256 位补码形式。

如果一个类型显式转换成更小的类型，相应的高位将被舍弃:

.. code-block:: solidity

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // 此时 b 的值是 0x5678


如果将整数显式转换为更大的类型，则将填充左侧（即在更高阶的位置）。
转换结果依旧等于原来整数:

.. code-block:: solidity

    uint16 a = 0x1234;
    uint32 b = uint32(a); // b 为 0x00001234 now
    assert(a == b);


定长字节数组转换则有所不同， 他们可以被认为是单个字节的序列和转换为较小的类型将切断序列:

.. code-block:: solidity

    bytes2 a = 0x1234;
    bytes1 b = bytes1(a); // b 为 0x12

如果将定长字节数组显式转换为更大的类型，将按正确的方式填充。 以固定索引访问转换后的字节将在和之前的值相等
（如果索引仍然在范围内）：

.. code-block:: solidity
    
    bytes2 a = 0x1234;
    bytes4 b = bytes4(a); // b 为 0x12340000
    assert(a[0] == b[0]);
    assert(a[1] == b[1]);

因为整数和定长字节数组在截断（或填充）时行为是不同的，
如果整数和定长字节数组有相同的大小，则允许他们之间进行显式转换， 如果要在不同的大小的整数和定长字节数组之间进行转换
，必须使用一个中间类型来明确进行所需截断和填充的规则::

.. code-block:: solidity

    bytes2 a = 0x1234;
    uint32 b = uint16(a);           // b 为 0x00001234
    uint32 c = uint32(bytes4(a));   // c 为 0x12340000
    uint8 d = uint8(uint16(a));     // d 为 0x34
    uint8 e = uint8(bytes1(a));     // e 为 0x12

``bytes`` 数组和 ``bytes`` calldata 切片可以显示转换为固定长度的 bytes 类型 (``bytes1``/.../``bytes32``).
如果数组比固定长度的 bytes 类型，则在末尾处会发生截断。
如果数组比目标类型短，它将在末尾用零填充。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.5;

    contract C {
        bytes s = "abcdefgh";
        function f(bytes calldata c, bytes memory m) public view returns (bytes16, bytes3) {
            require(c.length == 16, "");
            bytes16 b = bytes16(m);  // if length of m is greater than 16, truncation will happen
            b = bytes16(s);  // padded on the right, so result is "abcdefgh\0\0\0\0\0\0\0\0"
            bytes3 b1 = bytes3(s); // truncated, b1 equals to "abc"
            b = bytes16(c[:8]);  // also padded with zeros
            return (b, b1);
        }
    }


.. _types-conversion-literals:

字面常量与基本类型的转换
=================================================

整型与字面常量转换
-------------------

十进制和十六进制字面常量可以隐式转换为任何足以表示它而不会截断的整数类型 ：

.. code-block:: solidity

    uint8 a = 12; //  可行
    uint32 b = 1234; // 可行
    uint16 c = 0x123456; // 失败, 会截断为 0x3456

.. note::
    在 0.8.0 之前, 任何十进制和十六进制常量都可以显示转化为整型，不过从0.8.0开始，只有在匹配数据范围时，才能进行这个转换，就像隐式转换那样。 

定长字节数组与字面常量转换
-----------------------------

十进制字面常量不能隐式转换为定长字节数组。十六进制字面常量可以是，但仅当十六进制数字大小完全符合定长字节数组长度。
不过零值例外，零的十进制和十六进制字面常量都可以转换为任何定长字节数组类型：

.. code-block:: solidity

    bytes2 a = 54321; // 不可行
    bytes2 b = 0x12; // 不可行
    bytes2 c = 0x123; // 不可行
    bytes2 d = 0x1234; // 可行
    bytes2 e = 0x0012; // 可行
    bytes4 f = 0; // 可行
    bytes4 g = 0x0; // 可行


字符串字面常量和十六进制字符串字面常量可以隐式转换为定长字节数组，如果它们的字符数与字节类型的大小相匹配::

.. code-block:: solidity
    
    bytes2 a = hex"1234"; // 可行
    bytes2 b = "xy"; // 可行
    bytes2 c = hex"12"; // 不可行
    bytes2 d = hex"123"; // n不可行
    bytes2 e = "x"; // 不可行
    bytes2 f = "xyz"; // 不可行

地址类型
---------

参考 :ref:`address_literals` ，通过校验和测试的正确大小的十六进制字面常量会作为 ``address`` 类型。没有其他字面常量可以隐式转换为 ``address`` 类型。

从 ``bytes20`` 或其他整型显示转换为 ``address`` 类型时，都会作为 ``address payable`` 类型。

一个地址 ``address a`` 可以通过``payable(a)``　转换为　 ``address payable``  类型.
