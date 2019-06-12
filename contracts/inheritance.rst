.. include:: ../glossaries.rst

.. index:: ! inheritance, ! base class, ! contract;base, ! deriving

***********
继承
***********

Solidity 支持多重继承包括多态。

所有的函数调用都是虚拟的，这意味着最终派生的函数会被调用，除非明确给出合约名称或者使用super关键字。

当一个合约从多个合约继承时，在区块链上只有一个合约被创建，所有基类合约的代码被编译到创建的合约中。这意味着对基类合约函数的所有内部调用也只是使用内部函数调用（super.f（..）将使用JUMP跳转而不是消息调用）。

总的来说，Solidity 的继承系统与 `Python的继承系统 <https://docs.python.org/3/tutorial/classes.html#inheritance>`_ 非常
相似，特别是多重继承方面， 但是也有一些 :ref:`不同 <multi-inheritance>` 。

下面的例子进行了详细的说明。

::

    pragma solidity >=0.5.0 <0.7.0;

    contract Owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    // 使用 is 从另一个合约派生。派生合约可以访问所有非私有成员，包括内部（internal）函数和状态变量，
    // 但无法通过 this 来外部访问。
    contract Mortal is Owned {
        function kill() public {
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
    contract Named is Owned, Mortal {
        constructor(bytes32 name) public {
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
    contract PriceFeed is Owned, Mortal, Named("GoldFeed") {
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
        constructor() public { owner = msg.sender; }
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
        constructor() public { owner = msg.sender; }
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
（注意最终的继承序列是——从最终派生合约开始：Final, Base2, Base1, mortal, ownerd）。
在类中使用 super 调用的实际函数在当前类的上下文中是未知的，尽管它的类型是已知的。
这与普通的虚拟方法查找类似。

.. index:: ! constructor

.. _constructor:

构造函数
============

构造函数是使用 ``constructor`` 关键字声明的一个可选函数, 它在创建合约时执行, 可以在其中运行合约初始化代码。

在执行构造函数代码之前, 如果状态变量可以初始化为指定值; 如果不初始化, 则为零。

构造函数运行后, 将合约的最终代码部署到区块链。代码的部署需要 gas 与代码的长度线性相关。
此代码包括所有函数部分是公有接口以及可以通过函数调用访问的所有函数。它不包括构造函数代码或仅从构造函数调用的内部函数。

构造函数可以是公有函数  ``public`` , 也可以是内部函数 ``internal`` 。如果没有构造函数, 合约将假定默认构造函数, 它等效于 ``constructor() public {}`` 。

举例：

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

构造函数作为 ``internal`` 函数，这个合约将标记为 :ref:`抽象合约 <abstract-contract>` 。

.. warning ::
    在 0.4.22 版本之前, 构造函数定义为合约的同名函数，不过语法在0.5.0之后弃用了。



.. index:: ! base;constructor

基类构造函数的参数
===============================


所有基类合约的构造函数将在下面解释的线性化规则被调用。如果基构造函数有参数,
派生合约需要指定所有参数。这可以通过两种方式来实现 ::

    pragma solidity >=0.4.22 <0.7.0;

    contract Base {
        uint x;
        constructor(uint _x) public { x = _x; }
    }

    // 直接在继承列表中指定参数
    contract Derived1 is Base(7) {
        constructor() public {}
    }

    // 或通过派生的构造函数中用 修饰符 "modifier" 
    contract Derived2 is Base {
        constructor(uint _y) Base(_y * _y) public {}
    }

一种方法直接在继承列表中调用基类构造函数（``is Base(7)``）。
另一种方法是像 |modifier| 使用方法一样，
作为派生合约构造函数定义头的一部分，（``Base(_y * _y)``)。
如果构造函数参数是常量并且定义或描述了合约的行为，使用第一种方法比较方便。
如果基类构造函数的参数依赖于派生合约，那么必须使用第二种方法。

参数必须在两种方式中（继承列表或派生构造函数修饰符样式）使用一种 。
在这两个位置都指定参数则会发生错误。

如果派生合约没有给所有基类合约指定参数，则这个合约将是抽象合约。


.. index:: ! inheritance;multiple, ! linearization, ! C3 linearization

.. _multi-inheritance:

多重继承与线性化
======================================

编程语言实现多重继承需要解决几个问题。
一个问题是 `钻石问题 <https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem>`_。
Solidity 借鉴了 Python 的方式并且使用“ `C3 线性化 <https://en.wikipedia.org/wiki/C3_linearization>`_ ”强制一个由基类构成的 DAG（有向无环图）保持一个特定的顺序。
这最终反映为我们所希望的唯一化的结果，但也使某些继承方式变为无效。尤其是，基类在 ``is`` 后面的顺序很重要：列出基类合约的
顺序从 "最基类" 到 "最派生类" 。请注意, 此顺序与 Python 中使用的顺序相反。

另一种简化的解释是, 当一个在不同的合约中多次定义函数被调用时,
, 给定的基类以从右到左 (Python 中从左到右) 按深度优先的方式进行搜索，在第一次匹配的时候停止。
如果基类合约已经搜索过, 则跳过该合约。


在下面的代码中，Solidity 会给出“ Linearization of inheritance graph impossible ”这样的错误。

::

    pragma solidity >=0.4.0 <0.7.0;

    contract X {}
    contract A is X {}
    // 编译出错
    contract C is A, X {}

代码编译出错的原因是 ``C`` 要求 ``X`` 重写 ``A`` （因为定义的顺序是 ``A, X`` ），
但是 ``A`` 本身要求重写 ``X``，无法解决这种冲突。

可以通过一个简单的规则来记忆：
以从“最接近的基类”（most base-like）到“最远的继承”（most derived）的顺序来指定所有的基类。

继承有相同名字的不同类型成员
======================================================

当继承导致一个合约具有相同名字的函数和 |modifier| 时，这会被认为是一个错误。
当事件和 |modifier| 同名，或者函数和事件同名时，同样会被认为是一个错误。
有一种例外情况，状态变量的 getter 函数可以覆盖一个 public 函数。
