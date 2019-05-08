.. include:: ../glossaries.rst
.. index:: !mapping

.. _mapping-types:

映射
=====

映射类型在声明时的形式为 ``mapping(_KeyType => _ValueType)``。
其中 ``_KeyType`` 可以是除了映射、变长数组、合约、枚举以及结构体以外的几乎所有类型。
``_ValueType`` 可以是包括映射类型在内的任何类型。

映射可以视作 `哈希表 <https://en.wikipedia.org/wiki/Hash_table>`_ ，它们在实际的初始化过程中创建每个可能的 key，
并将其映射到字节形式全是零的值：一个类型的 :ref:`默认值 <default-value>`。然而下面是映射与哈希表不同的地方：
在映射中，实际上并不存储 key，而是存储它的 ``keccak256`` 哈希值，从而便于查询实际的值。

正因为如此，映射是没有长度的，也没有 key 的集合或 value 的集合的概念。

只有状态变量（或者在 internal 函数中的对于存储变量的引用）可以使用映射类型。。

可以将映射声明为 ``public``，然后来让 Solidity 创建一个 :ref:`getter <visibility-and-getters>`。
``_KeyType`` 将成为 getter 的必须参数，并且 getter 会返回 ``_ValueType``。

``_ValueType`` 也可以是一个映射。这时在使用 getter 时将将需要递归地传入每个 ``_KeyType`` 参数。

::

    pragma solidity ^0.4.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingUser {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(this);
        }
    }


.. note::
  映射不支持迭代，但可以在此之上实现一个这样的数据结构。
  例子可以参考 `可迭代的映射 <https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol>`_。
