.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

***********************
合约结构
***********************


在 Solidity 中，合约类似于面向对象编程语言中的类。
每个合约中可以包含 :ref:`structure-state-variables`、 :ref:`structure-functions`、
:ref:`structure-function-modifiers`、:ref:`structure-events`、 :ref:`structure-struct-types`、
和 :ref:`structure-enum-types` 的声明，且合约可以从其他合约继承。

.. _structure-state-variables:

状态变量
===============

状态变量是永久地存储在合约存储中的值。

::

    pragma solidity ^0.4.0;

    contract SimpleStorage {
        uint storedData; // 状态变量
        // ...
    }

有效的状态变量类型参阅 :ref:`types` 章节，
对状态变量可见性有可能的选择参阅 :ref:`visibility-and-getters` 。

.. _structure-functions:

函数
=========

函数是合约中代码的可执行单元。
::

    pragma solidity ^0.4.0;

    contract SimpleAuction {
        function bid() public payable { // 函数
            // ...
        }
    }

:ref:`function-calls` 可发生在合约内部或外部，且函数对其他合约有不同程度的可见性（ :ref:`visibility-and-getters`）。 

.. _structure-function-modifiers:

函数修饰器
==================

函数修饰器可以用来以声明的方式改良函数语义（参阅合约章节中 :ref:`modifiers`）。 

::

    pragma solidity ^0.4.11;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // 修饰器
            require(msg.sender == seller);
            _;
        }
        
        function abort() public onlySeller { // Modifier usage
            // ...
        }
    }

.. _structure-events:

事件
======

事件是与以太坊虚拟机日志工具的方便接口。
::

    pragma solidity ^0.4.0;
    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // 事件

        function bid() public payable {
            // ...
            HighestBidIncreased(msg.sender, msg.value); // 触发事件
        }
    }

有关如何声明事件和如何在 dapp 中使用事件的信息，参阅合约章节中的 :ref:`events`。

.. _structure-struct-types:

结构类型
=============

结构是可以将几个变量分组的自定义类型（参阅类型章节中的 :ref:`structs`）。
::

    pragma solidity ^0.4.0;

    contract Ballot {
        struct Voter { // 结构
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }

.. _structure-enum-types:

枚举类型
==========

枚举可用来创建有一定数量的值的自定义类型（参阅类型章节中的 :ref:`enums`）。 

::

    pragma solidity ^0.4.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // 枚举
    }
