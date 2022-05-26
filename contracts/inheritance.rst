.. include:: glossaries.rst

.. index:: ! inheritance, ! base class, ! contract;base, ! deriving

***********
继承
***********

Solidity 支持多重继承包括多态。

所有的函数调用都是虚拟的，这意味着最终派生的函数会被调用，除非明确给出合约名称或者使用super关键字。

当一个合约从多个合约继承时，在区块链上只有一个合约被创建，所有基类合约（或称为父合约）的代码被编译到创建的合约中。这意味着对基类合约函数的所有内部调用也只是使用内部函数调用（super.f（..）将使用JUMP跳转而不是消息调用）。

状态变量覆盖被视为错误。 派生合约不可以在声明已经是基类合约中可见的状态变量具有相同的名称 ``x``

总的来说，Solidity 的继承系统与 `Python的继承系统 <https://docs.python.org/3/tutorial/classes.html#inheritance>`_ 非常
相似，特别是多重继承方面， 但是也有一些 :ref:`不同 <multi-inheritance>` 。

下面的例子进行了详细的说明。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    contract Owned {
        constructor() public { owner = payable(msg.sender); }
        address payable owner;
    }

    // 使用 is 从另一个合约派生。派生合约可以访问所有非私有成员，包括内部（internal）函数和状态变量，
    // 但无法通过 this 来外部访问。
    contract Destructible is Owned {

     // 关键字`virtual`表示该函数可以在派生类中“overriding”。
        
        function destroy() virtual public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // 这些抽象合约仅用于给编译器提供接口。
    // 注意函数没有函数体。
    // 如果一个合约没有实现所有函数，则只能用作接口。
    abstract contract Config {
        function lookup(uint id) public virtual returns (address adr);
    }

    abstract contract NameReg {
        function register(bytes32 name) public virtual;
        function unregister() public virtual;
     }

    // 可以多重继承。请注意，owned 也是 Destructible 的基类，
    // 但只有一个 owned 实例（就像 C++ 中的虚拟继承）。
    contract Named is Owned, Destructible {
        constructor(bytes32 name) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // 函数可以被另一个具有相同名称和相同数量/类型输入的函数重载。
        // 如果重载函数有不同类型的输出参数，会导致错误。
        // 本地和基于消息的函数调用都会考虑这些重载。

 //如果要覆盖函数，则需要使用 `override` 关键字。 如果您想再次覆盖此函数，则需要再次指定`virtual`关键字。

        function destroy() public virtual override {
            if (msg.sender == owner) {
                Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
                NameReg(config.lookup(1)).unregister();
                // 仍然可以调用特定的重载函数。
                Destructible.destroy();
            }
        }
    }

    // 如果构造函数接受参数，
    // 则需要在声明（合约的构造函数）时提供，
    // 或在派生合约的构造函数位置以修改器调用风格提供（见下文）。
    contract PriceFeed is Owned, Destructible, Named("GoldFeed") {
        function updateInfo(uint newInfo) public {
            if (msg.sender == owner) info = newInfo;
        }

        // Here, we only specify `override` and not `virtual`.
        // This means that contracts deriving from `PriceFeed`
        // cannot change the behaviour of `destroy` anymore.
        function destroy() public override(Destructible, Named) { Named.destroy(); }

        function get() public view returns(uint r) { return info; }

        uint info;
    }

注意，在上边的代码中，我们调用 ``Destructible.destroy()`` 来“转发”销毁请求。
这样做法是有问题的，在下面的例子中可以看到：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    contract owned {
        constructor() { owner = payable(msg.sender); }
        address owner;
    }

    contract Destructible is owned {
        function destroy() public virtual {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is Destructible {
        function destroy() public virtual override  { /* 清除操作 1 */ Destructible.destroy(); }
    }

    contract Base2 is Destructible {
        function destroy() public { /* 清除操作 2 */ Destructible.destroy(); }
    }

    contract Final is Base1, Base2 {
        function destroy() public override(Base1, Base2) { Base2.destroy(); }
    }


。 解决此问题的方法是使用超级：

调用 ``Final.destroy()`` 时会调用  ``Base2.destroy``， 因为我们在最终重写中显式指定了它。
但是此函数将绕过 ``Base1.destroy``, 解决这个问题的方法是使用 ``super``：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    contract owned {
        constructor() { owner = payable(msg.sender); }
        address owner;
    }

    contract Destructible is owned {
        function destroy() virtual public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is Destructible {
        function destroy() public virtual override { /* 清除操作 1 */ super.destroy(); }
    }


    contract Base2 is Destructible {
        function destroy() public  virtual override { /* 清除操作 2 */ super.destroy(); }
    }

    contract Final is Base1, Base2 {
        function destroy() public override(Base1, Base2) { super.destroy(); }
    }

如果 ``Base2`` 调用 ``super`` 的函数，它不会简单在其基类合约上调用该函数。
相反，它在最终的继承关系图谱的下一个基类合约中调用这个函数，所以它会调用 ``Base1.destroy()``
（注意最终的继承序列是——从最终派生合约开始：Final, Base2, Base1, Destructible, ownerd）。
在类中使用 super 调用的实际函数在当前类的上下文中是未知的，尽管它的类型是已知的。
这与普通的虚拟方法查找类似。

.. index:: ! overriding;function

.. _function-overriding:

函数重写(Overriding)
=====================

父合约标记为 ``virtual`` 函数可以在继承合约里重写(overridden)以更改他们的行为。重写的函数需要使用关键字  ``override``  修饰。 


重写函数只能将覆盖函数的可见性从 ``external`` 更改为  ``public`` 。

可变性可以按照以下顺序更改为更严格的一种：
``nonpayable`` 可以被 ``view`` 和 ``pure`` 覆盖。 ``view`` 可以被 ``pure`` 覆盖。
``payable`` 是一个例外，不能更改为任何其他可变性。

以下示例演示了可变性和可见性的变化：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    contract Base
    {
        function foo() virtual external view {}
    }

    contract Middle is Base {}

    contract Inherited is Middle
    {
        function foo() override public pure {}
    }

对于多重继承，如果有多个父合约有相同定义的函数， ``override`` 关键字后必须指定所有父合约名。

例如：

.. code-block:: solidity

    pragma solidity >=0.7.0 <0.9.0;

    contract Base1
    {
        function foo() virtual public {}
    }

    contract Base2
    {
        function foo() virtual public {}
    }

    contract Inherited is Base1, Base2
    {
        // 继承自两个基类合约定义的foo(), 必须显示的指定 override 
        function foo() public override(Base1, Base2) {}
    }


不过如果（重写的）函数继承自一个公共的父合约， ``override`` 是可以不用显示指定的。
例如：

.. code-block:: solidity

    pragma solidity >=0.7.0 <0.9.0;

    contract A { function f() public pure{} }
    contract B is A {}
    contract C is A {}
    // 不用显示  override 
    contract D is B, C {}


更正式地说，如果存在父合约是签名函数的所有重写路径的一部分，则不需要重写（直接或间接）从多个基础继承的函数，并且（1）父合约实现了该函数，从当前合约到父合约的路径都没有提到具有该签名的函数，或者（2）父合约没有实现该函数，并且存在从当前合约到该父合约的所有路径中，最多只能提及该函数。

从这个意义上说，签名函数的重写路径是通过继承图的路径，该路径始于所考虑的合约，并终止于提及具有该签名的函数的合约。

如果函数没有标记为 ``virtual`` ， 那么派生合约将不能更改函数的行为（即不能重写）。

.. note::

  ``private`` 的函数是不可以标记为 ``virtual`` 的。

.. note::

  除接口之外（因为接口会自动作为 ``virtual`` ），没有实现的函数必须标记为 ``virtual``

.. note::

  从 Solidity 0.8.8 开始, 在重写接口函数时不再要求 ``override`` 关键字，除非函数在多个父合约定义。


如果getter 函数的参数和返回值都和外部函数一致时，外部（external）函数是可以被 public 的状态变量被重写的，例如：

.. code-block:: solidity

    pragma solidity >=0.7.0 <0.9.0;

    contract A
    {
        function f() external view virtual returns(uint) { return 5; }
    }

    contract B is A
    {
        uint public override f;
    }

.. note::

  尽管public 的状态变量可以重写外部函数，但是public 的状态变量不能被重写。


.. _modifier-overriding:

.. index:: ! overriding;modifier

修改器重写
===================

修改器重写也可以被重写，工作方式和 :ref:`函数重写 <function-overriding>` 类似。
需要被重写的修改器也需要使用 ``virtual`` 修饰， ``override`` 则同样修饰重载，例如：

.. code-block:: solidity

    pragma solidity >=0.7.0 <0.9.0;

    contract Base
    {
        modifier foo() virtual {_;}
    }

    contract Inherited is Base
    {
        modifier foo() override {_;}
    }


如果是多重继承，所有直接父合约必须显示指定override， 例如：

.. code-block:: solidity

    pragma solidity >=0.7.0 <0.9.0;

    contract Base1
    {
        modifier foo() virtual {_;}
    }

    contract Base2
    {
        modifier foo() virtual {_;}
    }

    contract Inherited is Base1, Base2
    {
        modifier foo() override(Base1, Base2) {_;}
    }



.. index:: ! constructor

.. _constructor:

构造函数
============

构造函数是使用 ``constructor`` 关键字声明的一个可选函数, 它在创建合约时执行, 可以在其中运行合约初始化代码。

在执行构造函数代码之前, 如果状态变量可以初始化为指定值; 如果不初始化, 则为 :ref:`默认值<default-value>` 。

构造函数运行后, 将合约的最终代码部署到区块链。代码的部署需要 gas 与代码的长度线性相关。
此代码包括所有函数部分是公有接口以及可以通过函数调用访问的所有函数。它不包括构造函数代码或仅从构造函数调用的内部函数。


如果没有构造函数, 合约将假定采用默认构造函数, 它等效于 ``constructor() {}`` 。

举例：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >0.6.99 <0.8.0;

    abstract contract A {
        uint public a;

        constructor(uint a) {
            a = a;
        }
    }

    contract B is A(1) {
        constructor() {}
    }


构造函数可以使用内部（internal）参数（例如指向存储的指针），在这个例子中，合约必须标记为 :ref:`抽象合约 <abstract-contract>` ，因为参数不能赋值被外部赋值，而仅能通过派生的合约。

.. warning ::
    在 0.4.22 版本之前, 构造函数定义为合约的同名函数，不过语法在0.5.0之后弃用了。

.. warning ::
    在 0.7.0 版本之前, 你需要通过 ``internal`` 或 ``public`` 指定构造函数的可见性。


.. index:: ! base;constructor

基类构造函数的参数
===============================


所有基类合约的构造函数将在下面解释的线性化规则被调用。如果基构造函数有参数,
派生合约需要指定所有参数。这可以通过两种方式来实现：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >0.6.99 <0.8.0;

    contract Base {
        uint x;
        constructor(uint x) { x = x; }
    }

    // 直接在继承列表中指定参数
    contract Derived1 is Base(7) {
        constructor() {}
    }

    // 或通过派生的构造函数中用 修饰符 "modifier"
    contract Derived2 is Base {
        constructor(uint y) Base(y * y) {}
    }

一种方法直接在继承列表中调用基类构造函数（``is Base(7)``）。
另一种方法是像 |modifier| 使用方法一样，
作为派生合约构造函数定义头的一部分，（``Base(y * y)``)。
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

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.8.0;

    contract X {}
    contract A is X {}
    // 编译出错
    contract C is A, X {}

代码编译出错的原因是 ``C`` 要求 ``X`` 重写 ``A`` （因为定义的顺序是 ``A, X`` ），
但是 ``A`` 本身要求重写 ``X``，无法解决这种冲突。

译者注：可以通过一个简单的规则来记忆：
以从“最接近的基类”（most base-like）到“最远的继承”（most derived）的顺序来指定所有的基类。

由于必须显式覆盖从多个基类继承的函数，因此C3线性化在实践中并不是太重要。

当继承层次结构中有多个构造函数时，继承线性化特别重要。 构造函数将始终以线性化顺序执行，无论在继承合约的构造函数中提供其参数的顺序如何。 例如：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >0.6.99 <0.8.0;

    contract Base1 {
        constructor() {}
    }

    contract Base2 {
        constructor() {}
    }

    //  构造函数以以下顺序执行:
    //  1 - Base1
    //  2 - Base2
    //  3 - Derived1
    contract Derived1 is Base1, Base2 {
        constructor() Base1() Base2() {}
    }

    // 构造函数以以下顺序执行:
    //  1 - Base2
    //  2 - Base1
    //  3 - Derived2
    contract Derived2 is Base2, Base1 {
        constructor() Base2() Base1() {}
    }

    // 构造函数仍然以以下顺序执行:
    //  1 - Base2
    //  2 - Base1
    //  3 - Derived3
    contract Derived3 is Base2, Base1 {
        constructor() Base1() Base2() {}
    }



继承有相同名字的不同类型成员
======================================================

当继承时合约出现了一下相同名字会被认为是一个错误：

  - 函数和 |modifier| 同名
  - 函数和事件同名
  - 事件和 |modifier| 同名

有一种例外情况，状态变量的 getter 函数可以覆盖 external 函数。


