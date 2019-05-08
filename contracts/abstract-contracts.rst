
.. index:: ! contract;abstract, ! abstract contract

.. _abstract-contract:

******************
抽象合约
******************

合约函数可以缺少实现，如下例所示（请注意函数声明头由 ``;`` 结尾）::

    pragma solidity >=0.4.0 <0.7.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

这些合约无法成功编译（即使它们除了未实现的函数还包含其他已经实现了的函数），但他们可以用作基类合约::

    pragma solidity >=0.4.0 <0.7.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }

如果合约继承自抽象合约，并且没有通过重写来实现所有未实现的函数，那么它本身就是抽象的。


Note that a function without implementation is different from a `函数类型 <https://learnblockchain.cn/docs/solidity/types.html#function-types>`_ even though their syntax looks very similar.

Example of function without implementation (a function declaration)::

    function foo(address) external returns (address);

Example of a Function Type (a variable declaration, where the variable is of type ``function``)::

    function(address) external returns (address) foo;

Abstract contracts decouple the definition of a contract from its implementation providing better extensibility and self-documentation and
facilitating patterns like the `Template method <https://en.wikipedia.org/wiki/Template_method_pattern>`_ and removing code duplication.
Abstract contracts are useful in the same way that defining methods in an interface is useful. It is a way for the designer of the abstract contract to say "any child of mine must implement this method".