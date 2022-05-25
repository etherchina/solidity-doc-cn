.. index:: ! error, revert

.. _errors:

*******************************
错误和回退语句
*******************************

Solidity 中的错误（关键字error）提供了一种方便且省gas的方式来向用户解释为什么一个操作会失败。它们可以被定义在合约（包括接口和库）内部和外部。

错误必须与 :ref:`revert 语句 <revert-statement>`一起使用。它会还原当前调用中的发生的所有变化，并将错误数据传回给调用者。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    /// 转账时，没有足够的余额。
    /// @param available balance available.
    /// @param required requested amount to transfer.
    error InsufficientBalance(uint256 available, uint256 required);

    contract TestToken {
        mapping(address => uint) balance;
        function transfer(address to, uint256 amount) public {
            if (amount > balance[msg.sender])
                revert InsufficientBalance({
                    available: balance[msg.sender],
                    required: amount
                });
            balance[msg.sender] -= amount;
            balance[to] += amount;
        }
        // ...
    }


错误不能被重写或覆盖，但是可以继承。只要作用域不同，同一个错误可以在多个地方定义。
只能使用 ``revert`` 语句创建错误实例。

错误产生的数据，会通过revert操作传递给调用者，可以交由链外组件处理或在 :ref:`try/catch 语句 <try-catch>` 中捕获它。
注意，只有外部调用的错误才能被捕获。发生在内部调用或同一函数内的 revert 不能被捕获。

如果错误没有任何参数，错误只需要四个字节的数据，你可以使用:ref:`NatSpec <natspec>`，来进一步解释错误背后的原因，NatSpec 不会存储在链上。
这个方式使得它同时也是一个非常便宜和方便的错误报告功能。

更具体地说，一个错误实例的ABI编码方式与调用相同名称和类型的函数的方式相同，它作为 ``revert`` 操作码的返回数据使用。
这意味着错误数据由一个4字节的选择器和 :ref:`ABI-encoded<abi>` 数据组成。
选择器是错误的签名的keccak256哈希的前四个字节组成。

.. note::
    It is possible for a contract to revert
    with different errors of the same name or even with errors defined in different places
    that are indistinguishable by the caller. For the outside, i.e. the ABI,
    only the name of the error is relevant, not the contract or file where it is defined.

The statement ``require(condition, "description");`` would be equivalent to
``if (!condition) revert Error("description")`` if you could define
``error Error(string)``.
Note, however, that ``Error`` is a built-in type and cannot be defined in user-supplied code.

Similarly, a failing ``assert`` or similar conditions will revert with an error
of the built-in type ``Panic(uint256)``.

.. note::
    Error data should only be used to give an indication of failure, but
    not as a means for control-flow. The reason is that the revert data
    of inner calls is propagated back through the chain of external calls
    by default. This means that an inner call
    can "forge" revert data that looks like it could have come from the
    contract that called it.
