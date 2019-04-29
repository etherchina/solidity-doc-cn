
.. index:: ! contract;interface, ! interface contract

.. _interfaces:

**********
接口
**********

接口类似于抽象合约，但是它们不能实现任何函数。还有进一步的限制：

#. 无法继承其他合约或接口。
#. 无法定义构造函数。
#. 无法定义变量。
#. 无法定义结构体
#. 无法定义枚举。

将来可能会解除这里的某些限制。

接口基本上仅限于合约 ABI 可以表示的内容，并且 ABI 和接口之间的转换应该不会丢失任何信息。

接口由它们自己的关键字表示：

::

    pragma solidity >=0.5.0 <0.7.0;

    interface Token {
        function transfer(address recipient, uint amount) public;
    }

就像继承其他合约一样，合约可以继承接口。