.. include:: ../glossaries.rst
.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

.. _reference-types:

引用类型
========

比起之前讨论过的值类型，在处理复杂的类型（即占用的空间超过 256 位的类型）时，我们需要更加谨慎。
由于拷贝这些类型变量的开销相当大，我们不得不考虑它的存储位置，是将它们保存在 ** |memory| ** （并不是永久存储）中，
还是 ** |storage| ** （保存状态变量的地方）中。

.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

数据位置
---------

所有的复杂类型，即 *数组* 和 *结构* 类型，都有一个额外属性，“数据位置”，说明数据是保存在 |memory| 中还是 |storage| 中。
根据上下文不同，大多数时候数据有默认的位置，但也可以通过在类型名后增加关键字 ``storage`` 或 ``memory`` 进行修改。
函数参数（包括返回的参数）的数据位置默认是 ``memory``，
局部变量的数据位置默认是 ``storage``，状态变量的数据位置强制是 ``storage`` （这是显而易见的）。

也存在第三种数据位置， ``calldata`` ，这是一块只读的，且不会永久存储的位置，用来存储函数参数。
外部函数的参数（非返回参数）的数据位置被强制指定为 ``calldata`` ，效果跟 ``memory`` 差不多。

数据位置的指定非常重要，因为它们影响着赋值行为：
在 |storage| 和 |memory| 之间两两赋值，或者 |storage| 向状态变量（甚至是从其它状态变量）赋值都会创建一份独立的拷贝。
然而状态变量向局部变量赋值时仅仅传递一个引用，而且这个引用总是指向状态变量，因此后者改变的同时前者也会发生改变。
另一方面，从一个 |memory| 存储的引用类型向另一个 |memory| 存储的引用类型赋值并不会创建拷贝。

::

    pragma solidity ^0.4.0;

    contract C {
        uint[] x; // x 的数据存储位置是 storage

        // memoryArray 的数据存储位置是 memory
        function f(uint[] memoryArray) public {
            x = memoryArray; // 将整个数组拷贝到 storage 中，可行
            var y = x;  // 分配一个指针（其中 y 的数据存储位置是 storage），可行
            y[7]; // 返回第 8 个元素，可行
            y.length = 2; // 通过 y 修改 x，可行
            delete x; // 清除数组，同时修改 y，可行
            // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组， /
            // 但 storage 是“静态”分配的：
            // y = memoryArray;
            // 下面这一行也不可行，因为这会“重置”指针，
            // 但并没有可以让它指向的合适的存储位置。
            // delete y;
            
            g(x); // 调用 g 函数，同时移交对 x 的引用
            h(x); // 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
        }

        function g(uint[] storage storageArray) internal {}
        function h(uint[] memoryArray) public {}
    }

总结
^^^^^

强制指定的数据位置：
 - 外部函数的参数（不包括返回参数）： calldata
 - 状态变量： storage

默认数据位置：
 - 函数参数（包括返回参数）： memory
 - 所有其它局部变量： storage

.. index:: ! array

.. _arrays:

数组
-----

数组可以在声明时指定长度，也可以动态调整大小。
对于 |storage| 的数组来说，元素类型可以是任意的（即元素也可以是数组类型，映射类型或者结构体）。
对于 |memory| 的数组来说，元素类型不能是映射类型，如果作为 public 函数的参数，它只能是 ABI 类型。

一个元素类型为 ``T``，固定长度为 ``k`` 的数组可以声明为 ``T[k]``，而动态数组声明为 ``T[]``。
举个例子，一个长度为 5，元素类型为 ``uint`` 的动态数组的数组，应声明为 ``uint[][5]`` （注意这里跟其它语言比，数组长度的声明位置是反的）。
要访问第三个动态数组的第二个元素，你应该使用 x[2][1]（数组下标是从 0 开始的，且访问数组时的下标顺序与声明时相反，也就是说，x[2] 是从右边减少了一级）。。

``bytes`` 和 ``string`` 类型的变量是特殊的数组。
``bytes`` 类似于 ``byte[]``，但它在 calldata 中会被“紧打包”（译者注：将元素连续地存在一起，不会按每 32 字节一单元的方式来存放）。
``string`` 与 ``bytes`` 相同，但（暂时）不允许用长度或索引来访问。

.. note::
    如果想要访问以字节表示的字符串 ``s``，请使用 ``bytes(s).length`` / ``bytes(s)[7] = 'x';``。
    注意这时你访问的是 UTF-8 形式的低级 bytes 类型，而不是单个的字符。

可以将数组标识为 ``public``，从而让 Solidity 创建一个 :ref:`getter <visibility-and-getters>`。
之后必须使用数字下标作为参数来访问 getter。

.. index:: ! array;allocating, new

创建内存数组
^^^^^^^^^^^^^

可使用 ``new`` 关键字在内存中创建变长数组。
与 |storage| 数组相反的是，你 *不能* 通过修改成员变量 ``.length`` 改变 |memory| 数组的大小。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);
            // 这里我们有 a.length == 7 以及 b.length == len
            a[6] = 8;
        }
    }

.. index:: ! array;literals, !inline;arrays

数组字面常数 / 内联数组
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

数组字面常数是写作表达式形式的数组，并且不会立即赋值给变量。

::

    pragma solidity ^0.4.16;

    contract C {
        function f() public pure {
            g([uint(1), 2, 3]);
        }
        function g(uint[3] _data) public pure {
            // ...
        }
    }

数组字面常数是一种定长的 |memory| 数组类型，它的基础类型由其中元素的普通类型决定。
例如，``[1, 2, 3]`` 的类型是 ``uint8[3] memory``，因为其中的每个字面常数的类型都是 ``uint8``。
正因为如此，有必要将上面这个例子中的第一个元素转换成 ``uint`` 类型。
目前需要注意的是，定长的 |memory| 数组并不能赋值给变长的 |memory| 数组，下面是个反例：

::

    // 这段代码并不能编译。

    pragma solidity ^0.4.0;

    contract C {
        function f() public {
            // 这一行引发了一个类型错误，因为 unint[3] memory
            // 不能转换成 uint[] memory。
            uint[] x = [uint(1), 3, 4];
        }
    }

已经计划在未来移除这样的限制，但目前数组在 ABI 中传递的问题造成了一些麻烦。

.. index:: ! array;length, length, push, !array;push

成员
^^^^^^

**length**:
    数组有 ``length`` 成员变量表示当前数组的长度。
    动态数组可以在 |storage| （而不是 |memory| ）中通过改变成员变量 ``.length`` 改变数组大小。
    并不能通过访问超出当前数组长度的方式实现自动扩展数组的长度。
    一经创建，|memory| 数组的大小就是固定的（但却是动态的，也就是说，它依赖于运行时的参数）。

**push**:
    变长的 |storage| 数组以及 ``bytes`` 类型（而不是 ``string`` 类型）都有一个叫做 ``push`` 的成员函数，它用来附加新的元素到数组末尾。
    这个函数将返回新的数组长度。

.. warning::
    在外部函数中目前还不能使用多维数组。

.. warning::
    由于 |evm| 的限制，不能通过外部函数调用返回动态的内容。
    例如，如果通过 web3.js 调用 ``contract C { function f() returns (uint[]) { ... } }`` 中的 ``f`` 函数，它会返回一些内容，但通过 Solidity 不可以。

    目前唯一的变通方法是使用大型的静态数组。

::

    pragma solidity ^0.4.16;

    contract ArrayContract {
        uint[2**20] m_aLotOfIntegers;
        // 注意下面的代码并不是一对动态数组，
        // 而是一个数组元素为一对变量的动态数组（也就是数组元素为长度为 2 的定长数组的动态数组）。
        bool[2][] m_pairsOfFlags;
        // newPairs 存储在 memory 中 —— 函数参数默认的存储位置

        function setAllFlagPairs(bool[2][] newPairs) public {
            // 向一个 storage 的数组赋值会替代整个数组
            m_pairsOfFlags = newPairs;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // 访问一个不存在的数组下标会引发一个异常
            m_pairsOfFlags[index][0] = flagA;
            m_pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // 如果 newSize 更小，那么超出的元素会被清除
            m_pairsOfFlags.length = newSize;
        }

        function clear() public {
            // 这些代码会将数组全部清空
            delete m_pairsOfFlags;
            delete m_aLotOfIntegers;
            // 这里也是实现同样的功能
            m_pairsOfFlags.length = 0;
        }

        bytes m_byteData;

        function byteArrays(bytes data) public {
            // 字节的数组（语言意义中的 byte 的复数 ``bytes``）不一样，因为它们不是填充式存储的，
            // 但可以当作和 "uint8[]" 一样对待
            m_byteData = data;
            m_byteData.length += 7;
            m_byteData[3] = byte(8);
            delete m_byteData[2];
        }

        function addFlag(bool[2] flag) public returns (uint) {
            return m_pairsOfFlags.push(flag);
        }

        function createMemoryArray(uint size) public pure returns (bytes) {
            // 使用 `new` 创建动态 memory 数组：
            uint[2][] memory arrayOfPairs = new uint[2][](size);
            // 创建一个动态字节数组：
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = byte(i);
            return b;
        }
    }


.. index:: ! struct, ! type;struct

.. _structs:

结构体
-------

Solidity 支持通过构造结构体的形式定义新的类型，以下是一个结构体使用的示例：

::

    pragma solidity ^0.4.11;

    contract CrowdFunding {
        // 定义的新类型包含两个属性。
        struct Funder {
            address addr;
            uint amount;
        }

        struct Campaign {
            address beneficiary;
            uint fundingGoal;
            uint numFunders;
            uint amount;
            mapping (uint => Funder) funders;
        }

        uint numCampaigns;
        mapping (uint => Campaign) campaigns;

        function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
            campaignID = numCampaigns++; // campaignID 作为一个变量返回
            // 创建新的结构体示例，存储在 storage 中。我们先不关注映射类型。
            campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // 以给定的值初始化，创建一个新的临时 memory 结构体，
            // 并将其拷贝到 storage 中。
            // 注意你也可以使用 Funder(msg.sender, msg.value) 来初始化。
            c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
            c.amount += msg.value;
        }

        function checkGoalReached(uint campaignID) public returns (bool reached) {
            Campaign storage c = campaigns[campaignID];
            if (c.amount < c.fundingGoal)
                return false;
            uint amount = c.amount;
            c.amount = 0;
            c.beneficiary.transfer(amount);
            return true;
        }
    }

上面的合约只是一个简化版的众筹合约，但它已经足以让我们理解结构体的基础概念。
结构体类型可以作为元素用在映射和数组中，其自身也可以包含映射和数组作为成员变量。

尽管结构体本身可以作为映射的值类型成员，但它并不能包含自身。
这个限制是有必要的，因为结构体的大小必须是有限的。

注意在函数中使用结构体时，一个结构体是如何赋值给一个局部变量（默认存储位置是 |storage| ）的。
在这个过程中并没有拷贝这个结构体，而是保存一个引用，所以对局部变量成员的赋值实际上会被写入状态。

当然，你也可以直接访问结构体的成员而不用将其赋值给一个局部变量，就像这样，
``campaigns[campaignID].amount = 0``。
