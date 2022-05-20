
.. index:: ! contract;abstract, ! abstract contract

.. _abstract-contract:

******************
抽象合约
******************

如果未实现合约中的至少一个函数，则需要将合约标记为 abstract。
即使实现了所有功能，合同也可能被标记为abstract。


如下例所示，可以使用关键字 ``abstract`` 定义抽象合约合约, 由于``utterance()`` 函数没有具体的实现（没有实现体 ``{ }`` , 而是以 ``;`` 结尾：

.. code-block:: solidity

    pragma solidity >=0.6.0 <0.9.0;

    abstract contract Feline {
        function utterance() public returns (bytes32);
    }


这样的抽象合约不能直接实例化。 如果抽象合约本身确实都有实现所有定义的函数，也是正确的。
下例显示了抽象合约作为基类的用法：

.. code-block:: solidity

    pragma solidity >=0.6.0 <0.9.0;

    abstract contract Feline {
      function utterance() public pure returns (bytes32);
    }

    contract Cat is Feline {
      function utterance() public pure returns (bytes32) { return "miaow"; }
    }

如果合约继承自抽象合约，并且没有通过重写来实现所有未实现的函数， 它依然需要标记为抽象 abstract 合约.



请注意，没有实现的函数与 :ref:`Function Type <function_types>` 不同，即使它们的语法看起来非常相似。

没有实现的函数示例（函数声明）：

.. code-block:: solidity

    function foo(address) external returns (address);

函数类型的示例（变量声明，其中变量的类型为“函数”）：

.. code-block:: solidity

    function(address) external returns (address) foo;


抽象合约将合约的定义与其实现脱钩，从而提供了更好的可扩展性和自文档性，并简化了诸如 `Template方法 <https://en.wikipedia.org/wiki/Template_method_pattern>`_ 的模式并消除了代码重复。抽象合约的使用方式与接口 interface 中定义方法的使用方式相同。 抽象合约的设计者可以这样说“我的任何继承都必须实施此方法”。


.. note::

  抽象合约不能用一个无实现的函数重写一个实现了的虚函数。
