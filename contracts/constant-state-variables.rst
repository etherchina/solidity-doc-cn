
.. index:: ! constant

************************
Constant 状态变量
************************

状态变量可以被声明为 ``constant``。在这种情况下，只能使用那些在编译时有确定值的表达式来给它们赋值。
任何通过访问 storage，区块链数据（例如 ``now``, ``this.balance`` 或者 ``block.number``）或执行数据（ ``msg.gas`` ）
或对外部合约的调用来给它们赋值都是不允许的。
在内存分配上有边界效应（side-effect）的表达式是允许的，但对其他内存对象产生边界效应的表达式则不行。
内建（built-in）函数 ``keccak256``，``sha256``，``ripemd160``，``ecrecover``，``addmod`` 和 ``mulmod`` 是允许的（即使他们确实会调用外部合约）。

允许带有边界效应的内存分配器的原因是这将允许构建复杂的对象，比如查找表（lookup-table）。
此功能尚未完全可用。

编译器不会为这些变量预留存储，它们的每次出现都会被替换为相应的常量表达式（这将可能被优化器计算为实际的某个值）。

不是所有类型的状态变量都支持用 constant 来修饰，当前支持的仅有值类型和字符串。

::

    pragma solidity >=0.4.0 <0.7.0;

    contract C {
        uint constant x = 32**22 + 8;
        string constant text = "abc";
        bytes32 constant myHash = keccak256("abc");
    }