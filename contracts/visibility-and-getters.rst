
.. index:: ! visibility, external, public, private, internal

.. _visibility-and-getters:

**********************
可见性和 getter 函数
**********************

由于 Solidity 有两种函数调用（内部调用不会产生实际的 EVM 调用或称为“消息调用”，而外部调用则会产生一个 EVM 调用），
函数和状态变量有四种可见性类型。
函数可以指定为 ``external`` ，``public`` ，``internal`` 或者 ``private``，默认情况下函数类型为 ``public``。
对于状态变量，不能设置为 ``external`` ，默认是 ``internal`` 。

``external`` ：
    外部函数作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。
    一个外部函数 ``f`` 不能从内部调用（即 ``f`` 不起作用，但 ``this.f()`` 可以）。
    当收到大量数据的时候，外部函数有时候会更有效率。
``public`` ：
    public 函数是合约接口的一部分，可以在内部或通过消息调用。对于公共状态变量，
    会自动生成一个 getter 函数（见下面）。
``internal`` ：
    这些函数和状态变量只能是内部访问（即从当前合约内部或从它派生的合约访问），不使用 ``this`` 调用。
``private`` ：
    private 函数和状态变量仅在当前定义它们的合约中使用，并且不能被派生合约使用。

.. note::
    合约中的所有内容对外部观察者都是可见的。设置一些 ``private`` 类型只能阻止其他合约访问和修改这些信息，
    但是对于区块链外的整个世界它仍然是可见的。

可见性标识符的定义位置，对于状态变量来说是在类型后面，对于函数是在参数列表和返回关键字中间。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }

在下面的例子中，``D`` 可以调用 ``c.getData（）`` 来获取状态存储中 ``data`` 的值，但不能调用 ``f`` 。
合约 ``E`` 继承自 ``C`` ，因此可以调用 ``compute``。

::

    // 下面代码编译错误

    pragma solidity ^0.4.0;

    contract C {
        uint private data;

        function f(uint a) private returns(uint b) { return a + 1; }
        function setData(uint a) public { data = a; }
        function getData() public returns(uint) { return data; }
        function compute(uint a, uint b) internal returns (uint) { return a+b; }
    }

    contract D {
        function readData() public {
            C c = new C();
            uint local = c.f(7); // 错误：成员 `f` 不可见
            c.setData(3);
            local = c.getData();
            local = c.compute(3, 5); // 错误：成员 `compute` 不可见
        }
    }

    contract E is C {
        function g() public {
            C c = new C();
            uint val = compute(3, 5); // 访问内部成员（从继承合约访问父合约成员）
        }
    }

.. index:: ! getter;function, ! function;getter
.. _getter-functions:

Getter 函数
================

编译器自动为所有 **public** 状态变量创建 getter 函数。对于下面给出的合约，编译器会生成一个名为 ``data`` 的函数，
该函数不会接收任何参数并返回一个 ``uint`` ，即状态变量 ``data`` 的值。可以在声明时完成状态变量的初始化。

::

    pragma solidity ^0.4.0;

    contract C {
        uint public data = 42;
    }

    contract Caller {
        C c = new C();
        function f() public {
            uint local = c.data();
        }
    }

getter 函数具有外部可见性。如果在内部访问 getter（即没有 ``this.`` ），它被认为一个状态变量。
如果它是外部访问的（即用 ``this.`` ），它被认为为一个函数。

::

    pragma solidity ^0.4.0;

    contract C {
        uint public data;
        function x() public {
            data = 3; // 内部访问
            uint val = this.data(); // 外部访问
        }
    }

下一个例子稍微复杂一些：

::

    pragma solidity ^0.4.0;

    contract Complex {
        struct Data {
            uint a;
            bytes3 b;
            mapping (uint => uint) map;
        }
        mapping (uint => mapping(bool => Data[])) public data;
    }

这将会生成以下形式的函数 ::

    function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
        a = data[arg1][arg2][arg3].a;
        b = data[arg1][arg2][arg3].b;
    }

请注意，因为没有好的方法来提供映射的键，所以结构中的映射被省略。
