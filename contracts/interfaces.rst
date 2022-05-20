
.. index:: ! contract;interface, ! interface contract

.. _interfaces:

**********
接口
**********

接口类似于抽象合约，但是它们不能实现任何函数。还有进一步的限制：

- 无法继承其他合约,不过可以继承其他接口。
- 接口中所有的函数都需要是 external，尽管在合约里可以是public
- 无法定义构造函数。
- 无法定义状态变量。
- 不可以声明修改器。

将来可能会解除这里的某些限制。

接口基本上仅限于合约 ABI 可以表示的内容，并且 ABI 和接口之间的转换不应该丢失任何信息。

接口由它们自己的关键字表示：

.. code-block:: solidity

    pragma solidity >=0.6.2 <0.9.0;

    interface Token {
        enum TokenType { Fungible, NonFungible }
        struct Coin { string obverse; string reverse; }
        function transfer(address recipient, uint amount) external;
    }

就像继承其他合约一样，合约可以继承接口。接口中的函数都会隐式的标记为 ``virtual`` ，意味着他们会被重写并不需要 ``override`` 关键字。
但是不表示重写（overriding）函数可以再次重写，仅仅当重写的函数标记为 ``virtual`` 才可以再次重写。

接口可以继承其他的接口，遵循同样继承规则。

.. code-block:: solidity

    pragma solidity >=0.6.2 <0.9.0;

    interface ParentA {
        function test() external returns (uint256);
    }

    interface ParentB {
        function test() external returns (uint256);
    }

    interface SubInterface is ParentA, ParentB {
        // 必须重新定义 test 函数，以表示兼容父合约含义
        function test() external override(ParentA, ParentB) returns (uint256);
    }


定义在接口或其他类合约（ contract-like）结构体里的类型，可以在其他的合约里用这样的方式访问： ``Token.TokenType`` 或 ``Token.Coin``.

.. warning:

    从 :doc:`Solidity 0.5.0 版本 <050-breaking-changes>` 开始接口里可以支持声明 ``enum`` 类型了。
