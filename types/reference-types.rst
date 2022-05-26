.. include:: glossaries.rst
.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

.. _reference-types:

引用类型
========

引用类型可以通过多个不同的名称修改它的值，而值类型的变量，每次都有独立的副本。因此，必须比值类型更谨慎地处理引用类型。
目前，引用类型包括结构，数组和映射，如果使用引用类型，则必须明确指明数据存储哪种类型的位置（空间）里：

- |memory| 即数据在内存中，因此数据仅在其生命周期内（函数调用期间）有效。不能用于外部调用。
- |storage| 状态变量保存的位置，只要合约存在就一直存储．
- |calldata| 用来保存函数参数的特殊数据位置，是一个只读位置。

.. note::
    译者注：0.6.9 之前 calldata 仅用于外部函数调用参数，0.6.9之后可用于任意函数。

更改数据位置或类型转换将始终产生自动进行一份拷贝，而在同一数据位置内（对于 |storage| 来说）的复制仅在某些情况下进行拷贝。

.. _data-location:

数据位置
---------

所有的引用类型，如 *数组* 和 *结构体* 类型，都有一个额外注解 ``数据位置`` ，来说明数据存储位置。
有三种位置： |memory| 、 |storage| 以及 |calldata| 。
|calldata|  是不可修改的、非持久的函数参数存储区域，效果大多类似 |memory| 。
主要用于外部函数的参数，但也可用于其他变量。

.. note::
    如果可以的话，请尽量使用 ``calldata`` 作为数据位置，因为它将避免复制，并确保不能修改数据。
    函数的返回值中也可以使用 ``calldata`` 数据位置的数组和结构，但是无法给其分配空间。

.. note::
    在0.6.9版本之前，引用类型参数的数据位置有限制，外部函数中使用 ``calldata`` ，公共函数中使用 ``memory`` ，以及内部和私有函数中的 ``memory`` 或 ``storage`` 。
    现在 ``memory`` 和 ``calldata`` 在所有函数中都被允许使用，无论其可见性如何。

.. note::
    在版本0.5.0之前，数据位置可以省略，并且根据变量的类型，函数类型等有默认数据位置，但是所有复杂类型现在必须提供明确的数据位置。


.. _data-location-assignment:

数据位置与赋值行为
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

数据位置不仅仅表示数据如何保存，它同样影响着赋值行为：

* 在 |storage| 和 |memory| 之间两两赋值（或者从 |calldata| 赋值 ），都会创建一份独立的拷贝。
* 从 |memory| 到 |memory| 的赋值只创建引用， 这意味着更改内存变量，其他引用相同数据的所有其他内存变量的值也会跟着改变。
* 从 |storage| 到本地存储变量的赋值也只分配一个引用。
* 其他的向 |storage| 的赋值，总是进行拷贝。 这种情况的示例如对状态变量或 |storage| 的结构体类型的局部变量成员的赋值，即使局部变量本身是一个引用，也会进行一份拷贝（译者注：查看下面 ``ArrayContract`` 合约 更容易理解）。


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;

    contract Tiny {
        uint[] x; // x 的数据存储位置是 storage，　位置可以忽略

        // memoryArray 的数据存储位置是 memory
        function f(uint[] memory memoryArray) public {
            x = memoryArray; // 将整个数组拷贝到 storage 中，可行
            uint[] storage y = x;  // 分配一个指针（其中 y 的数据存储位置是 storage），可行
            y[7]; // 返回第 8 个元素，可行
            y.pop(); // 通过 y 修改 x，可行
            delete x; // 清除数组，同时修改 y，可行

            // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组，
            // 但 storage 是“静态”分配的：
            // y = memoryArray;
            // 下面这一行也不可行，因为这会“重置”指针，
            // 但并没有可以让它指向的合适的存储位置。
            // delete y;

            g(x); // 调用 g 函数，同时移交对 x 的引用
            h(x); // 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
        }

        function g(uint[] storage ) internal pure {}
        function h(uint[] memory) public pure {}
    }

.. index:: ! array

.. _arrays:

数组
-----

数组可以在声明时指定长度，也可以动态调整大小（长度）。

一个元素类型为 ``T``，固定长度为 ``k`` 的数组可以声明为 ``T[k]``，而动态数组声明为 ``T[]``。
举个例子，一个长度为 5，元素类型为 ``uint`` 的动态数组的数组（二维数组），应声明为 ``uint[][5]`` （注意这里跟其它语言比，数组长度的声明位置是反的）。

.. note::
  译者注：作为对比，如在Java中，声明一个包含5个元素、每个元素都是数组的方式为 int[5][]。

在Solidity中， ``X[3]`` 总是一个包含三个 ``X`` 类型元素的数组，即使 ``X`` 本身就是一个数组，这和其他语言也有所不同，比如 C 语言。

数组下标是从 0 开始的，且访问数组时的下标顺序与声明时相反。

如：如果有一个变量为 ``uint[][5] memory x``， 要访问第三个动态数组的第7个元素，使用 x[2][6]，要访问第三个动态数组使用 ``x[2]``。
同样，如果有一个 ``T`` 类型的数组 ``T[5] a`` ， T 也可以是一个数组，那么 ``a[2]`` 总会是 ``T`` 类型。

数组元素可以是任何类型，包括映射或结构体。对类型的限制是映射只能存储在 |storage| 中，并且公开访问函数的参数需要是 :ref:`ABI 类型 <ABI>`。

状态变量标记 ``public`` 的数组，Solidity创建一个 :ref:`getter函数 <visibility-and-getters>` 。
小标数字索引就是 getter函数 的参数。

访问超出数组长度的元素会导致异常（assert 类型异常 ）。 可以使用 ``.push()`` 方法在末尾追加一个新元素，其中 ``.push()`` 追加一个零初始化的元素并返回对它的引用。


.. index:: ! string, ! bytes

.. _strings:

.. _bytes:

``bytes`` 和 ``string`` 也是数组
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``bytes`` 和 ``string`` 类型的变量是特殊的数组。
``bytes`` 类似于 ``bytes1[]``，但它在 |calldata| 和 |memory| 中会被“紧打包”（译者注：将元素连续地存在一起，不会按每 32 字节一单元的方式来存放）。
``string`` 与 ``bytes`` 相同，但不允许用长度或索引来访问。

Solidity没有字符串操作函数，但是可以使用第三方字符串库，我们可以比较两个字符串通过计算他们的 keccak256-hash ，可使用
``keccak256(abi.encodePacked(s1)) == keccak256(abi.encodePacked(s2))`` 和使用 ``string.concat(s1, s2)`` 来拼接字符串。

我们更多时候应该使用 ``bytes`` 而不是 ``bytes1[]`` ，因为Gas 费用更低, 在 |memory| 中使用 ``bytes1[]`` 时，会在元素之间添加31个填充字节。
而在 |storage| 中，由于紧密包装，这没有填充字节， 参考 :ref:`bytes and string <bytes-and-string>` 。
作为一个基本规则，对任意长度的原始字节数据使用 ``bytes``，对任意长度字符串（UTF-8）数据使用 ``string`` 。

如果使用一个长度限制的字节数组，应该使用一个 ``bytes1`` 到 ``bytes32`` 的具体类型，因为它们便宜得多。

.. note::
    如果想要访问以字节表示的字符串 ``s``，请使用 ``bytes(s).length`` / ``bytes(s)[7] = 'x';``。
    注意这时你访问的是 UTF-8 形式的低级 bytes 类型，而不是单个的字符。


.. index:: ! bytes-concat, ! string-concat

.. _bytes-concat:
.. _string-concat:

函数 ``bytes.concat`` 和 ``string.concat``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

可以使用 ``string.concat`` 连接任意数量的 ``string`` 字符串。
该函数返回一个 ``string memory`` ，包含所有参数的内容，无填充方式拼接在一起。
如果你想使用不能隐式转换为 ``string`` 的其他类型作为参数，你需要先把它们转换为 ``string``。

同样， ``bytes.concat`` 函数可以连接任意数量的 ``bytes`` 或 ``bytes1 ... bytes32`` 值。
该函数返回一个 ``bytes memory`` ，包含所有参数的内容，无填充方式拼接在一起。
如果你想使用字符串参数或其他不能隐式转换为 ``bytes`` 的类型，你需要先将它们转换为 ``bytes``或 ``bytes1``/.../ ``bytes32``。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.12;

    contract C {
        string s = "Storage";
        function f(bytes calldata bc, string memory sm, bytes16 b) public view {
            string memory concatString = string.concat(s, string(bc), "Literal", sm);
            assert((bytes(s).length + bc.length + 7 + bytes(sm).length) == bytes(concatString).length);

            bytes memory concatBytes = bytes.concat(bytes(s), bc, bc[:2], "Literal", bytes(sm), b);
            assert((bytes(s).length + bc.length + 2 + 7 + bytes(sm).length + b.length) == concatBytes.length);
        }
    }

如果你调用不使用参数调用 ``string.concat`` 或 ``bytes.concat`` 将返回空数组。


.. index:: ! array;allocating, new

创建内存数组
^^^^^^^^^^^^^

可使用 ``new`` 关键字在 |memory| 中基于运行时创建动态长度数组。
与 |storage| 数组相反的是，你 *不能* 通过修改成员变量 ``.push`` 改变 |memory| 数组的大小。

必须提前计算所需的大小或者创建一个新的内存数组并复制每个元素。

在Solidity中的所有变量，新分配的数组元素总是以 :ref:`默认值<default-value>` 初始化。

.. code-block:: solidity

    pragma solidity >=0.4.16 <0.9.0;

    contract TX {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);

            assert(a.length == 7);
            assert(b.length == len);

            a[6] = 8;
        }
    }

.. index:: ! array;literals, !inline;arrays

数组常量
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

数组常量（字面量）是在方括号中（ ``[...]`` ） 包含一个或多个逗号分隔的表达式。例如 ``[1, a, f(3)]`` 。 

数组常量的类型通过以下的方式确定:

它总是一个静态大小的内存数组，其长度为表达式的数量。

数组的基本类型是列表上的第一个表达式的类型，以便所有其他表达式可以隐式地转换为它。如果不可以转换，将出现类型错误。

所有元素都都可以转换为基本类型也是不够的。其中一个元素必须是这种类型的。

在下面的例子中， ``[1, 2, 3]`` 的类型是 ``uint8[3] memory``。 因为每个常量的类型都是 ``uint8`` ，如果你希望结果是 ``uint[3] memory`` 类型，你需要将第一个元素转换为 ``uint`` 。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract LBC {
        function f() public pure {
            g([uint(1), 2, 3]);
        }
        function g(uint[3] memory) public pure {
            // ...
        }
    }


数组常量 ``[1, -1]`` 是无效的，因为第一个表达式类型是 ``uint8`` 而第二个类似是 ``int8`` 他们不可以隐式的相互转换。
为了确保可以运行，你是可以使用例如：  ``[int8(1), -1]`` 。

由于不同类型的固定大小的内存数组不能相互转换(尽管基础类型可以)，如果你想使用二维数组常量，你必须显式地指定一个基础类型:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f() public pure returns (uint24[2][4] memory) {
            uint24[2][4] memory x = [[uint24(0x1), 1], [0xffffff, 2], [uint24(0xff), 3], [uint24(0xffff), 4]];
            // 下面代码无法工作，因为没有匹配内部类型
            // uint[2][4] memory x = [[0x1, 1], [0xffffff, 2], [0xff, 3], [0xffff, 4]];
            return x;
        }
    }

目前需要注意的是，定长的 |memory| 数组并不能赋值给变长的 |memory| 数组，下面的例子是无法运行的：

.. code-block:: solidity

    pragma solidity  >=0.4.0 <0.9.0;

    // 这段代码并不能编译。
    contract LBC {
        function f() public {
            // 这一行引发了一个类型错误，因为 unint[3] memory
            // 不能转换成 uint[] memory。
            uint[] x = [uint(1), 3, 4];
        }
    }

计划在未来移除这样的限制，但目前数组在 ABI 中传递的问题造成了一些麻烦。

如果要初始化动态长度的数组，则必须显示给各个元素赋值:


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract C {
        function f() public pure {
            uint[] memory x = new uint[](3);
            x[0] = 1;
            x[1] = 3;
            x[2] = 4;
        }
    }


.. index:: ! array;length, length, push, pop, !array;push, !array;pop

.. _array-members:

数组成员
^^^^^^^^^^^^

**length**:
    数组有 ``length`` 成员变量表示当前数组的长度。
    一经创建，|memory| 数组的大小就是固定的（但却是动态的，也就是说，它可以根据运行时的参数创建）。


**push()**:
    动态的 |storage| 数组以及 ``bytes`` 类型（ ``string`` 类型不可以）都有一个 ``push()`` 的成员函数，它用来添加新的零初始化元素到数组末尾，并返回元素引用．
    因此可以这样：　 ``x.push().t = 2`` 或 ``x.push() = b``.

**push(x)**:
    动态的 |storage| 数组以及 ``bytes`` 类型（ ``string`` 类型不可以）都有一个 ``push(ｘ)`` 的成员函数，用来在数组末尾添加一个给定的元素，这个函数没有返回值．

**pop()**:
    变长的 |storage| 数组以及 ``bytes`` 类型（ ``string`` 类型不可以）都有一个 ``pop()`` 的成员函数， 它用来从数组末尾删除元素。 同样的会在移除的元素上隐含调用 :ref:`delete` 。


.. note::
    通过 ``push()``　增加 |storage| 数组的长度具有固定的 gas 消耗，因为 |storage| 总是被零初始化，而通过　``pop()``　减少长度则依赖移除与元素的大小（size）．　如果元素是数组,则成本是很高的,因为它包括已删除的元素的清理，类似于在这些元素上调用 :ref:`delete` 。

.. note::
    如果需要在外部（external）函数中使用多维数组，这需要启用ABI coder v2。
    公有（public）函数中是支持的使用多维数组。

.. note::
    在Byzantium（在2017-10-16日4370000区块上进行硬分叉升级）之前的EVM版本中，无法访问从函数调用返回动态数组。 如果要调用返回动态数组的函数，请确保 EVM 在拜占庭模式上运行。

.. code-block:: solidity


    pragma solidity >=0.6.0 <0.9.0;

    contract ArrayContract {
        uint[2**20] aLotOfIntegers;

        // 注意下面的代码并不是一对动态数组，
        // 而是一个数组元素为一对变量的动态数组（也就是数组元素为长度为 2 的定长数组的动态数组）。
        // 因为  T[] 总是 T 的动态数组, 尽管 T 是数组
        // 所有的状态变量的数据位置都是 storage
        bool[2][] pairsOfFlags;

        // newPairs 存储在 memory 中 (仅当它是公有的合约函数)
        function setAllFlagPairs(bool[2][] memory newPairs) public {

         // 向一个 storage 的数组赋值会对 ``newPairs`` 进行拷贝，并替代整个 ``pairsOfFlags`` 数组
            pairsOfFlags = newPairs;
        }

        struct StructType {
            uint[] contents;
            uint moreInfo;
        }
        StructType s;

        function f(uint[] memory c) public {
            // 保存引用
            StructType storage g = s;

            // 同样改变了 ``s.moreInfo``.
            g.moreInfo = 2;

            // 进行了拷贝，因为 ``g.contents`` 不是本地变量，而是本地变量的成员
            g.contents = c;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // 访问不存在的索引将引发异常
            pairsOfFlags[index][0] = flagA;
            pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
           // 使用 push 和 pop 是更改数组长度的唯一方法

            if (newSize < pairsOfFlags.length) {
                while (pairsOfFlags.length > newSize)
                    pairsOfFlags.pop();
            } else if (newSize > pairsOfFlags.length) {
                while (pairsOfFlags.length < newSize)
                    pairsOfFlags.push();
            }
        }

        function clear() public {
            // 这些完全清除了数组
            delete pairsOfFlags;
            delete aLotOfIntegers;
            // 效果相同（和上面）
            pairsOfFlags.length = new bool[2][](0);
        }

        bytes byteData;

        function byteArrays(bytes memory data) public {
            // 字节数组（bytes）不一样，它们在没有填充的情况下存储。
            // 可以被视为与 uint8 [] 相同
            byteData = data;
            for (uint i = 0; i < 7; i++)
                byteData.push();
            byteData[3] = 0x08;
            delete byteData[2];
        }

        function addFlag(bool[2] memory flag) public returns (uint) {
            pairsOfFlags.push(flag);
            return pairsOfFlags.length;
        }

        function createMemoryArray(uint size) public pure returns (bytes memory) {
            // 使用`new`创建动态内存数组：
            uint[2][] memory arrayOfPairs = new uint[2][](size);

            // 内联（Inline）数组始终是静态大小的，如果只使用字面常量，则必须至少提供一种类型。
            arrayOfPairs[0] = [uint(1), 2];

            // 创建一个动态字节数组：
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = byte(uint8(i));
            return b;
        }
    }


.. index:: ! array;slice

.. _array-slices:

数组切片
------------

数组切片是数组连续部分的视图，用法如：``x[start:end]`` ， ``start`` 和 ``end`` 是 uint256 类型（或结果为 uint256 的表达式）。
``x[start:end]`` 的第一个元素是 ``x[start]`` ， 最后一个元素是   ``x[end - 1]`` 。

如果 ``start`` 比 ``end`` 大或者 ``end`` 比数组长度还大，将会抛出异常。

``start`` 和 ``end`` 都可以是可选的： ``start`` 默认是 0， 而 ``end`` 默认是数组长度。

数组切片没有任何成员。 它们可以隐式转换为其“背后”类型的数组，并支持索引访问。 索引访问也是相对于切片的开始位置。
数组切片没有类型名称，这意味着没有变量可以将数组切片作为类型，它们仅存在于中间表达式中。



.. note::
    目前数组切片，仅可使用于 calldata 数组.

数组切片在 ABI解码数据的时候非常有用，如：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.5 <0.9.0;

    contract Proxy {
        /// 被当前合约管理的 客户端合约地址
        address client;

        constructor(address client_) {
            client = client_;
        }

        /// 在进行参数验证之后，转发到由client实现的 "setOwner(address)" 
        function forward(bytes calldata payload) external {
            bytes4 sig = bytes4(payload[:4]);

            // 由于截断行为，与执行 bytes4(payload) 是相同的
            // bytes4 sig = bytes4(payload);

            if (sig == bytes4(keccak256("setOwner(address)"))) {
                address owner = abi.decode(payload[4:], (address));
                require(owner != address(0), "Address of owner cannot be zero.");
            }
            (bool status,) = client.delegatecall(payload);
            require(status, "Forwarded call failed.");
        }
    }



.. index:: ! struct, ! type;struct

.. _structs:

结构体
-------

Solidity 支持通过构造结构体的形式定义新的类型，以下是一个结构体使用的示例：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    // 定义的新类型包含两个属性。
    // 在合约外部声明结构体可以使其被多个合约共享。 在这里，这并不是真正需要的。
    struct Funder {
        address addr;
        uint amount;
    }

    contract CrowdFunding {

        // 也可以在合约内部定义结构体，这使得它们仅在此合约和衍生合约中可见。
        struct Campaign {
            address beneficiary;
            uint fundingGoal;
            uint numFunders;
            uint amount;
            mapping (uint => Funder) funders;
        }

        uint numCampaigns;
        mapping (uint => Campaign) campaigns;

        function newCampaign(address payable beneficiary, uint goal) public returns (uint campaignID) {
            campaignID = numCampaigns++; // campaignID 作为一个变量返回

            // 不能使用 "campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0)" 
            // 因为RHS（right hand side）会创建一个包含映射的内存结构体 "Campaign"
            Campaign storage c = campaigns[campaignID];
            c.beneficiary = beneficiary;
            c.fundingGoal = goal;
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

上面的合约只是一个简化版的 `众筹合约 <https://learnblockchain.cn/2018/02/28/ico-crowdsale/>`_，但它已经足以让我们理解结构体的基础概念。
结构体类型可以作为元素用在映射和数组中，其自身也可以包含映射和数组作为成员变量。

尽管结构体本身可以作为映射的值类型成员，但它并不能包含自身。
这个限制是有必要的，因为结构体的大小必须是有限的。

注意在函数中使用结构体时，一个结构体是如何赋值给一个存储位置是 |storage| 的局部变量。
在这个过程中并没有拷贝这个结构体，而是保存一个引用，所以对局部变量成员的赋值实际上会被写入状态。

当然，你也可以直接访问结构体的成员而不用将其赋值给一个局部变量，就像这样，
``campaigns[campaignID].amount = 0``。


.. note::
    在 Solidity 0.7.0 之前，在 |memory| 结构体包含仅 |storage| 的类型（例如映射）可以允许类似上例中的 ``campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0)`` 赋值，它会直接忽略映射类型。