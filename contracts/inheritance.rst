.. include:: ../glossaries.rst

.. index:: ! inheritance, ! base class, ! contract;base, ! deriving

***********
继承
***********

通过复制包括多态的代码，Solidity 支持多重继承。

所有的函数调用都是虚拟的，这意味着最远的派生函数会被调用，除非明确给出合约名称。

当一个合约从多个合约继承时，在区块链上只有一个合约被创建，所有基类合约的代码被复制到创建的合约中。

总的来说，Solidity 的继承系统与 `Python的继承系统 <https://docs.python.org/3/tutorial/classes.html#inheritance>`_ ，非常
相似，特别是多重继承方面。

下面的例子进行了详细的说明。

::

    pragma solidity >=0.5.0 <0.7.0;

    contract owned {
        function owned() { owner = msg.sender; }
        address owner;
    }

    // 使用 is 从另一个合约派生。派生合约可以访问所有非私有成员，包括内部函数和状态变量，
    // 但无法通过 this 来外部访问。
    contract mortal is owned {
        function kill() {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // 这些抽象合约仅用于给编译器提供接口。
    // 注意函数没有函数体。
    // 如果一个合约没有实现所有函数，则只能用作接口。
    contract Config {
        function lookup(uint id) public returns (address adr);
    }

    contract NameReg {
        function register(bytes32 name) public;
        function unregister() public;
     }

    // 可以多重继承。请注意，owned 也是 mortal 的基类，
    // 但只有一个 owned 实例（就像 C++ 中的虚拟继承）。
    contract named is owned, mortal {
        function named(bytes32 name) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // 函数可以被另一个具有相同名称和相同数量/类型输入的函数重载。
        // 如果重载函数有不同类型的输出参数，会导致错误。
        // 本地和基于消息的函数调用都会考虑这些重载。
        function kill() public {
            if (msg.sender == owner) {
                Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
                NameReg(config.lookup(1)).unregister();
                // 仍然可以调用特定的重载函数。
                mortal.kill();
            }
        }
    }

    // 如果构造函数接受参数，
    // 则需要在声明（合约的构造函数）时提供，
    // 或在派生合约的构造函数位置以修饰器调用风格提供（见下文）。
    contract PriceFeed is owned, mortal, named("GoldFeed") {
       function updateInfo(uint newInfo) public {
          if (msg.sender == owner) info = newInfo;
       }

       function get() public view returns(uint r) { return info; }

       uint info;
    }

注意，在上边的代码中，我们调用 ``mortal.kill()`` 来“转发”销毁请求。
这样做法是有问题的，在下面的例子中可以看到::

    pragma solidity >=0.4.22 <0.7.0;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* 清除操作 1 */ mortal.kill(); }
    }

    contract Base2 is mortal {
        function kill() public { /* 清除操作 2 */ mortal.kill(); }
    }

    contract Final is Base1, Base2 {
    }

调用 ``Final.kill()`` 时会调用最远的派生重载函数 ``Base2.kill``，但是会绕过 ``Base1.kill``，
主要是因为它甚至都不知道 ``Base1`` 的存在。解决这个问题的方法是使用 ``super``::

    pragma solidity >=0.4.22 <0.7.0;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* 清除操作 1 */ super.kill(); }
    }


    contract Base2 is mortal {
        function kill() public { /* 清除操作 2 */ super.kill(); }
    }

    contract Final is Base1, Base2 {
    }

如果 ``Base2`` 调用 ``super`` 的函数，它不会简单在其基类合约上调用该函数。
相反，它在最终的继承关系图谱的下一个基类合约中调用这个函数，所以它会调用 ``Base1.kill()``
（注意最终的继承序列是——从最远派生合约开始：Final, Base2, Base1, mortal, ownerd）。
在类中使用 super 调用的实际函数在当前类的上下文中是未知的，尽管它的类型是已知的。
这与普通的虚拟方法查找类似。

.. index:: ! constructor

.. _constructor:

构造器
============

A 构造器函数 is an optional function declared with the ``constructor`` keyword
which is executed upon contract creation, and where you can run contract
initialisation code.

Before the constructor code is executed, state variables are initialised to
their specified value if you initialise them inline, or zero if you do not.

After the constructor has run, the final code of the contract is deployed
to the blockchain. The deployment of
the code costs additional gas linear to the length of the code.
This code includes all functions that are part of the public interface
and all functions that are reachable from there through function calls.
It does not include the constructor code or internal functions that are
only called from the constructor.

Constructor functions can be either ``public`` or ``internal``. If there is no
constructor, the contract will assume the default constructor, which is
equivalent to ``constructor() public {}``. For example:

::

    pragma solidity >=0.5.0 <0.7.0;

    contract A {
        uint public a;

        constructor(uint _a) internal {
            a = _a;
        }
    }

    contract B is A(1) {
        constructor() public {}
    }

A constructor set as ``internal`` causes the contract to be marked as :ref:`abstract <abstract-contract>`.

.. warning ::
    Prior to version 0.4.22, constructors were defined as functions with the same name as the contract.
    This syntax was deprecated and is not allowed anymore in version 0.5.0.



.. index:: ! base;constructor

基类构造函数的参数
===============================

派生合约需要提供基类构造函数需要的所有参数。这可以通过两种方式来完成::

    pragma solidity >=0.4.22 <0.7.0;

    contract Base {
        uint x;
        function Base(uint _x) public { x = _x; }
    }

    contract Derived is Base(7) {
        function Derived(uint _y) Base(_y * _y) public {
        }
    }

一种方法直接在继承列表中调用基类构造函数（``is Base(7)``）。
另一种方法是像 |modifier| 使用方法一样，
作为派生合约构造函数定义头的一部分，（``Base(_y * _y)``)。
如果构造函数参数是常量并且定义或描述了合约的行为，使用第一种方法比较方便。
如果基类构造函数的参数依赖于派生合约，那么必须使用第二种方法。
如果像这个简单的例子一样，两个地方都用到了，优先使用 |modifier| 风格的参数。

.. index:: ! inheritance;multiple, ! linearization, ! C3 linearization

.. _multi-inheritance:

多重继承与线性化
======================================

编程语言实现多重继承需要解决几个问题。
一个问题是 `钻石问题 <https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem>`_。
Solidity 借鉴了 Python 的方式并且使用“ `C3 线性化 <https://en.wikipedia.org/wiki/C3_linearization>`_ ”强制一个由基类构成的 DAG（有向无环图）保持一个特定的顺序。
这最终反映为我们所希望的唯一化的结果，但也使某些继承方式变为无效。尤其是，基类在 ``is`` 后面的顺序很重要。
在下面的代码中，Solidity 会给出“ Linearization of inheritance graph impossible ”这样的错误。

::

    // 以下代码编译出错

    pragma solidity >=0.4.0 <0.7.0;

    contract X {}
    contract A is X {}
    contract C is A, X {}

代码编译出错的原因是 ``C`` 要求 ``X`` 重写 ``A`` （因为定义的顺序是 ``A, X`` ），
但是 ``A`` 本身要求重写 ``X``，无法解决这种冲突。

可以通过一个简单的规则来记忆：
以从“最接近的基类”（most base-like）到“最远的继承”（most derived）的顺序来指定所有的基类。

继承有相同名字的不同类型成员
======================================================

当继承导致一个合约具有相同名字的函数和 |modifier| 时，这会被认为是一个错误。
当事件和 |modifier| 同名，或者函数和事件同名时，同样会被认为是一个错误。
有一种例外情况，状态变量的 getter 可以覆盖一个 public 函数。
