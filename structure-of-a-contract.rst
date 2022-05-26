.. include:: glossaries.rst
.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

***********************
合约结构
***********************


在 Solidity 语言中，合约类似于其他面向对象编程语言中的**类**。

每个合约中可以包含 :ref:`structure-state-variables`、 :ref:`structure-functions`、
:ref:`structure-function-modifiers`, :ref:`structure-events`, :ref:`structure-errors`, :ref:`structure-struct-types` 和 :ref:`structure-enum-types` 的声明，且合约可以从其他合约继承。

还有一些特殊的合约，如： :ref:`库<libraries>` 和 :ref:`接口<interfaces>`.

专门的 :ref:`合约<contracts>` 章节会比本节包含更多的内容，本节用于帮助我们合约包含哪些内容，做一个简单的入门。

.. _structure-state-variables:

状态变量
===============

状态变量是永久地存储在合约存储中的值。

.. code-block:: solidity

    pragma solidity >=0.4.0 <0.9.0;

    contract TinyStorage {
        uint storedXlbData; // 状态变量
        // ...
    }

有效的状态变量类型参阅 :ref:`types` 章节，
对状态变量可见性有可能的选择参阅 :ref:`visibility-and-getters` 。

.. _structure-functions:

函数
=========

函数是代码的可执行单元。函数通常在合约内部定义，但也可以在合约外定义。


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.1 <0.9.0;

    contract TinyAuction {
        function Mybid() public payable { // 定义函数
            // ...
        }
    }

    // Helper function defined outside of a contract
    function helper(uint x) pure returns (uint) {
        return x * 2;
    }

:ref:`function-calls` 可发生在合约内部或外部，且函数对其他合约有不同程度的可见性（ :ref:`visibility-and-getters`）。

:ref:`函数<functions>` 可以接受 :ref:`参数和返回值<function-parameters-return-variables>`。

.. _structure-function-modifiers:

函数 |modifier|
==================

函数 |modifier| 可以用来以声明的方式修改函数语义（参阅合约章节中 :ref:`函数修改器<modifiers>`）。

重载（Overloading）, 表示有同样的 |modifier| 名称但是有不同的参数的情况，这是不允许的。

而例如函数或 |modifier| 则可以被 :ref:`重写（overridden） <modifier-overriding>`.


.. code-block:: solidity

    pragma solidity >=0.4.22 <0.9.0;

    contract MyPurchase {
        address public seller;

        modifier onlySeller() { // 修改器
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        function abort() public onlySeller { // 修改器用法
            // ...
        }
    }

.. _structure-events:

事件 Event
============

事件是能方便地调用以太坊虚拟机日志功能的接口。

.. code-block:: solidity

    pragma solidity >=0.4.21 <0.9.0;
    contract TinyAuction {
        event HighestBidIncreased(address bidder, uint amount); // 事件

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // 触发事件
        }
    }

参阅合约章节中的 :ref:`events` 了解如何声明和在 DApp 中使用。


.. _structure-errors:

错误(Errors)
============

Solidity 为应对失败，允许用户定义 ``error`` 来描述错误的名称和数据。
错误可以在  :ref:`revert statements <revert-statement>` 中使用，

跟用错误字符串相比， ``error`` 更便宜并且允许你编码额外的数据，还可以用 NatSpec 为用户去描述错误。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    /// 没有足够的资金用于转账， 参数 `requested` 表示需要的资金，`available` 表示仅有的资金。

    error NotEnoughFunds(uint requested, uint available);

    contract Token {
        mapping(address => uint) balances;
        function transfer(address to, uint amount) public {
            uint balance = balances[msg.sender];
            if (balance < amount)
                revert NotEnoughFunds(amount, balance);
            balances[msg.sender] -= amount;
            balances[to] += amount;
            // ...
        }
    }

在合约的 :ref:`errors` 查看更多的信息。

.. _structure-struct-types:

结构体
=============

结构体是可以将几个变量分组的自定义类型（参阅类型章节中的 :ref:`structs`）。

.. code-block:: solidity

    pragma solidity >=0.4.0 <0.9.0;

    contract TinyBallot {
        struct Voter { // 结构体
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }

.. _structure-enum-types:

枚举类型
==========

枚举可用来创建由一定数量的“常量值”构成的自定义类型（参阅类型章节中的 :ref:`enums`）。

.. code-block:: solidity

    pragma solidity >=0.4.0 <0.9.0;

    contract Upchain {
        enum State { Created, Locked, InValid } // 枚举
    }
