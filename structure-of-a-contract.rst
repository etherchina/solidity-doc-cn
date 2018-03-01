.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

***********************
合约代码结构
***********************


在 Solidity 中合约类似于面向对象编程语言中的类。
每个合约中可定义 :ref:`structure-state-variables`、 :ref:`structure-functions`、
:ref:`structure-function-modifiers`、:ref:`structure-events`、 :ref:`structure-struct-types`、
和 :ref:`structure-enum-types`，且合约可从其他合约继承。

.. _structure-state-variables:

状态变量
===============

状态变量是永久存储在合约存储器中的值。

::

    pragma solidity ^0.4.0;

    contract SimpleStorage {
        uint storedData; // State variable
        // ...
    }

在 :ref:`types` 章节查看有效的状态变量类型。状态变量的可见性见 :ref:`visibility-and-getters`。

.. _structure-functions:

函数
=========

函数是合约中的可执行单元。
::

    pragma solidity ^0.4.0;

    contract SimpleAuction {
        function bid() public payable { // Function
            // ...
        }
    }

:ref:`function-calls` 可在合约内部或外部执行，且函数对不同其他合约有不同的可见性（详见合约章节中的 :ref:`visibility-and-getters`）。 

.. _structure-function-modifiers:

函数修饰符
==================

函数修饰符可以用来以声明的方式修改函数语义（详见合约章节的 :ref:`modifiers`)）。 

::

    pragma solidity ^0.4.11;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // Modifier
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

事件是记录日志到 EVM（太坊虚拟机）日志组件的便利入口。

::

    pragma solidity ^0.4.0;

    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // Event

        function bid() public payable {
            // ...
            HighestBidIncreased(msg.sender, msg.value); // Triggering event
        }
    }

参见合约章节中 :ref:`events` 以了解如何定义事件和如何在 dapp 中使用事件。 

.. _structure-struct-types:

结构类型
=============

结构是可以分组定义多个变量的自定义类型（详见类型章节中的 :ref:`structs`）。

::

    pragma solidity ^0.4.0;

    contract Ballot {
        struct Voter { // Struct
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }

.. _structure-enum-types:

枚举类型
==========

枚举可用来创建拥有有限的多个值的自定义类型（详见类型章节中的 :ref:`enums`）。 

::

    pragma solidity ^0.4.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // Enum
    }
