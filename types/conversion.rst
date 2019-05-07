
.. index:: ! type;conversion, ! cast

.. _types-conversion-elementary-types:

基本类型之间的转换
==================

隐式转换
---------

如果一个运算符用在两个不同类型的变量之间，那么编译器将隐式地将其中一个类型转换为另一个类型（不同类型之间的赋值也是一样）。
一般来说，只要值类型之间的转换在语义上行得通，而且转换的过程中没有信息丢失，那么隐式转换基本都是可以实现的：
``uint8`` 可以转换成 ``uint16``，``int128`` 转换成 ``int256``，但 ``int8`` 不能转换成 ``uint256``
（因为 ``uint256`` 不能涵盖某些值，例如，``-1``）。
更进一步来说，无符号整型可以转换成跟它大小相等或更大的字节类型，但反之不能。
任何可以转换成 ``uint160`` 的类型都可以转换成 ``address`` 类型。

显式转换
---------

如果某些情况下编译器不支持隐式转换，但是你很清楚你要做什么，这种情况可以考虑显式转换。
注意这可能会发生一些无法预料的后果，因此一定要进行测试，确保结果是你想要的！
下面的示例是将一个 ``int8`` 类型的负数转换成 ``uint``：

::

    int8 y = -3;
    uint x = uint(y);

这段代码的最后，``x`` 的值将是 ``0xfffff..fd`` （64 个 16 进制字符），因为这是 -3 的 256 位补码形式。

如果一个类型显式转换成更小的类型，相应的高位将被舍弃 ::

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // 此时 b 的值是 0x5678

If an integer is explicitly converted to a larger type, it is padded on the left (i.e. at the higher order end).
The result of the conversion will compare equal to the original integer::

    uint16 a = 0x1234;
    uint32 b = uint32(a); // b will be 0x00001234 now
    assert(a == b);

Fixed-size bytes types behave differently during conversions. They can be thought of as
sequences of individual bytes and converting to a smaller type will cut off the
sequence::

    bytes2 a = 0x1234;
    bytes1 b = bytes1(a); // b will be 0x12

If a fixed-size bytes type is explicitly converted to a larger type, it is padded on
the right. Accessing the byte at a fixed index will result in the same value before and
after the conversion (if the index is still in range)::

    bytes2 a = 0x1234;
    bytes4 b = bytes4(a); // b will be 0x12340000
    assert(a[0] == b[0]);
    assert(a[1] == b[1]);

Since integers and fixed-size byte arrays behave differently when truncating or
padding, explicit conversions between integers and fixed-size byte arrays are only allowed,
if both have the same size. If you want to convert between integers and fixed-size byte arrays of
different size, you have to use intermediate conversions that make the desired truncation and padding
rules explicit::

    bytes2 a = 0x1234;
    uint32 b = uint16(a); // b will be 0x00001234
    uint32 c = uint32(bytes4(a)); // c will be 0x12340000
    uint8 d = uint8(uint16(a)); // d will be 0x34
    uint8 e = uint8(bytes1(a)); // e will be 0x12

.. _types-conversion-literals:

字面常量与基本类型的转换
=================================================

整型
-------------

Decimal and hexadecimal number literals can be implicitly converted to any integer type
that is large enough to represent it without truncation::

    uint8 a = 12; // fine
    uint32 b = 1234; // fine
    uint16 c = 0x123456; // fails, since it would have to truncate to 0x3456

定长字节数组
----------------------

Decimal number literals cannot be implicitly converted to fixed-size byte arrays. Hexadecimal
number literals can be, but only if the number of hex digits exactly fits the size of the bytes
type. As an exception both decimal and hexadecimal literals which have a value of zero can be
converted to any fixed-size bytes type::

    bytes2 a = 54321; // not allowed
    bytes2 b = 0x12; // not allowed
    bytes2 c = 0x123; // not allowed
    bytes2 d = 0x1234; // fine
    bytes2 e = 0x0012; // fine
    bytes4 f = 0; // fine
    bytes4 g = 0x0; // fine

String literals and hex string literals can be implicitly converted to fixed-size byte arrays,
if their number of characters matches the size of the bytes type::

    bytes2 a = hex"1234"; // fine
    bytes2 b = "xy"; // fine
    bytes2 c = hex"12"; // not allowed
    bytes2 d = hex"123"; // not allowed
    bytes2 e = "x"; // not allowed
    bytes2 f = "xyz"; // not allowed

地址类型
---------

As described in :ref:`address_literals`, hex literals of the correct size that pass the checksum
test are of ``address`` type. No other literals can be implicitly converted to the ``address`` type.

Explicit conversions from ``bytes20`` or any integer type to ``address`` result in ``address payable``.


.. index:: ! type;deduction, ! var

.. _type-deduction:

类型推断(已弃用)
=======================

为了方便起见，没有必要每次都精确指定一个变量的类型，编译器会根据分配该变量的第一个表达式的类型自动推断该变量的类型 ::

    uint24 x = 0x123;
    var y = x;

这里 ``y`` 的类型将是 ``uint24``。不能对函数参数或者返回参数使用 ``var``。

.. warning::
    类型只能从第一次赋值中推断出来，因此以下代码中的循环是无限的，
    原因是``i`` 的类型是 ``uint8``，而这个类型变量的最大值比 ``2000`` 小。
    ``for (var i = 0; i < 2000; i++) { ... }``
