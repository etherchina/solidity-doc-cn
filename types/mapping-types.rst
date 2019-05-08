.. include:: ../glossaries.rst
.. index:: !mapping

.. _mapping-types:

映射
=====

映射类型在声明时的形式为 ``mapping(_KeyType => _ValueType)``。
其中 ``_KeyType`` 可以是任何基本类型，即可以是任何的内建类型加上``bytes`` and ``string``。
而用户定义的类型或复杂的类型如：合约类型、枚举、映射、结构体、即除``bytes`` and ``string`` 之外的数组类型 是不可以作为 ``_KeyType`` 的类型的。

``_ValueType`` 可以是包括映射类型在内的任何类型。

映射可以视作 `哈希表 <https://en.wikipedia.org/wiki/Hash_table>`_ ，它们在实际的初始化过程中创建每个可能的 key，
并将其映射到字节形式全是零的值：一个类型的 :ref:`默认值 <default-value>`。然而下面是映射与哈希表不同的地方：
在映射中，实际上并不存储 key，而是存储它的 ``keccak256`` 哈希值，从而便于查询实际的值。

正因为如此，映射是没有长度的，也没有 key 的集合或 value 的集合的概念。

映射只能是 |storage| 的数据位置，因此只允许作为状态变量 或 作为函数内的 |storage| 引用 或 作为库函数的参数。
它们不能用合约公有函数的参数或返回值。


可以将映射声明为 ``public``，然后来让 Solidity 创建一个 :ref:`getter 函数 <visibility-and-getters>`。
``_KeyType`` 将成为 getter 的必须参数，并且 getter 会返回 ``_ValueType``。

如果 ``_ValueType`` 是一个映射。这时在使用 getter 时将需要递归地传入每个 ``_KeyType`` 参数，比如：

::

    pragma solidity >=0.4.0 <0.7.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingLBC {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(this);
        }
    }


.. note::
  映射不支持迭代，但可以在此之上实现一个这样的数据结构。
  例子可以参考 `可迭代的映射 <https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol>`_。
