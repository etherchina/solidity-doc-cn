
.. index: memory layout

****************
内存布局
****************

Solidity保留了四个32字节的插槽，字节范围(包括端点)特定用途如下：


- ``0x00`` - ``0x3f`` (64 字节): 用于哈希方法的暂存空间（临时空间）
- ``0x40`` - ``0x5f`` (32 字节): 当前分配的内存大小(也作为空闲内存指针)
- ``0x60`` - ``0x7f`` (32 字节): 零位插槽

暂存空间可以在语句之间使用 (例如在内联汇编中)。 零位插槽用作动态内存数组的初始值，并且永远不应写入（空闲内存指针最初指向``0x80``）.


Solidity 总是将新对象放在空闲内存指针上，并且内存永远不会被释放(将来可能会改变)。

Solidity 中的内存数组中的元素始终占据32字节的倍数（对于 ``byte[]`` 总是这样，但对于 ``bytes`` 和 ``string`` 而言则不是）。

多维内存数组是指向内存数组的指针，动态数组的长度存储在数组的第一个插槽中，然后是数组元素。


.. warning::
  Solidity中有一些需要临时存储区的操作需要大于64个字节， 因此无法放入暂存空间。
  它们将被放置在空闲内存指向的位置，但是由于使用寿命短，指针不会更新。
  内存可以归零，也可以不归零。 因此，不应指望空闲内存指针指向归零内存区域。

  尽管使用``msize``到达绝对归零的内存区域似乎是一个好主意，但使用此类非临时指针而不更新空闲内存指针可能会产生意外结果。

Differences to Layout in Storage
==================================

As described above the layout in memory is different from the layout in
:ref:`storage<storage-inplace-encoding>`. Below there are some examples.

Example for Difference in Arrays
--------------------------------

The following array occupies 32 bytes (1 slot) in storage, but 128
bytes (4 items with 32 bytes each) in memory.

::

    uint8[4] a;



Example for Difference in Struct Layout
---------------------------------------

The following struct occupies 96 bytes (3 slots of 32 bytes) in storage,
but 128 bytes (4 items with 32 bytes each) in memory.


::

    struct S {
        uint a;
        uint b;
        uint8 c;
        uint8 d;
    }