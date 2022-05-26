
.. index:: ! constant

.. _constants:

************************************
Constant 和 Immutable  状态变量
************************************

状态变量声明为 ``constant`` (常量)或者 ``immutable`` （不可变量），在这两种情况下，合约一旦部署之后，变量将不在修改。

对于 ``constant`` 常量, 他的值在编译器确定，而对于 ``immutable``, 它的值在部署时确定。

也可以在文件级别定义 ``constant`` 变量（注：0.7.2 之后加入的特性）。


编译器不会为这些变量预留存储位，它们的每次出现都会被替换为相应的常量表达式（它可能被优化器计算为实际的某个值）。

与常规状态变量相比，常量和不可变量的gas成本要低得多。 对于常量，赋值给它的表达式将复制到所有访问该常量的位置，并且每次都会对其进行重新求值。 这样可以进行本地优化。

不可变变量在构造时进行一次求值，并将其值复制到代码中访问它们的所有位置。 对于这些值，将保留32个字节，即使它们适合较少的字节也是如此。 因此，常量有时可能比不可变量更便宜。

不是所有类型的状态变量都支持用 constant 或 ``immutable`` 来修饰，当前仅支持 :ref:`字符串 <strings>` (仅常量) 和 :ref:`值类型 <value-types>` .


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >0.7.4;
    uint constant X = 32**22 + 8;
    
    contract C {

        string constant TEXT = "abc";
        bytes32 constant MY_HASH = keccak256("abc");
        uint immutable decimals;
        uint immutable maxBalance;
        address immutable owner = msg.sender;

        constructor(uint decimals_, address ref) {
            decimals = decimals_;
            // Assignments to immutables can even access the environment.
            maxBalance = ref.balance;
        }

        function isBalanceTooHigh(address _other) public view returns (bool) {
            return _other.balance > maxBalance;
        }
    }


Constant
========
如果状态变量声明为 ``constant`` (常量)。在这种情况下，只能使用那些在编译时有确定值的表达式来给它们赋值。
任何通过访问 storage，区块链数据（例如 ``block.timestamp``, ``address(this).balance`` 或者 ``block.number``）或执行数据（ ``msg.value`` 或 ``gasleft()`` ）
或对外部合约的调用来给它们赋值都是不允许的。

允许可能对内存分配产生副作用（side-effect）的表达式，但那些可能对其他内存对象产生副作用的表达式则不允许。

内建（built-in）函数 ``keccak256`` ， ``sha256`` ， ``ripemd160`` ， ``ecrecover`` ， ``addmod`` 和 ``mulmod`` 是允许的（即使他们确实会调用外部合约， ``keccak256`` 除外）。

允许内存分配器的副作用的原因是它可以构造复杂的对象，例如： 查找表（lookup-table）。 此功能尚不完全可用。



Immutable
==========

声明为不可变量(``immutable``)的变量的限制要比声明为常量(``constant``) 的变量的限制少：可以在合约的构造函数中或声明时为不可变的变量分配任意值。
不可变量只能赋值一次，并且在赋值之后才可以读取。

编译器生成的合约创建代码将在返回合约之前修改合约的运行时代码，方法是将对不可变量的所有引用替换为分配给它们的值。 如果要将编译器生成的运行时代码与实际存储在区块链中的代码进行比较，则这一点很重要。


.. note::
  不可变量可以在声明时赋值，不过只有在合约的构造函数执行时才被视为视为初始化。
  这意味着，你不能用一个依赖于不可变量的值在行内初始化另一个不可变量。
  不过，你可以在合约的构造函数中这样做。

  这是为了防止对状态变量初始化和构造函数顺序的不同解释，特别是继承时，出现问题。


.. note::
  译者注：不可变量(Immutable) 是 Solidity 0.6.5 引入的，因此0.6.5 之前的版本不可用。