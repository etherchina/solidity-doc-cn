
.. index:: ! visibility, external, public, private, internal


.. _visibility-and-getters:

**********************
可见性和 getter 函数
**********************


状态变量可见性
=========================

状态变量有 3 种可见性：

``public``
    对于 public 状态变量会自动生成一个  :ref:`getter hanshu <getter-functions>`  函数（见下面）。
    以便其他的合约读取他们的值。
    当在用一个合约里使用是，外部方式访问 (如: ``this.x``) 会调用getter 函数，而内部方式访问 (如: ``x``) 会直接从存储中获取值。
    Setter函数则不会被生成，所以其他合约不能直接修改其值。

``internal``
    内部可见性状态变量只能在它们所定义的合约和派生合同中访问。
    它们不能被外部访问。
    这是状态变量的默认可见性。

``private``
    私有状态变量就像内部变量一样，但它们在派生合约中是不可见的。

.. warning::

    设置为 ``private``或 ``internal``，只能防止其他合约读取或修改信息，但它仍然可以在链外查看到。


函数可见性
===================

由于 Solidity 有两种函数调用：外部调用则会产生一个 EVM 调用，而内部调用不会，
更进一步， 函数可以确定器被内部及派生合约的可访问性，这里有 4 种可见性：


``external``
    外部可见性函数作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。
    一个外部函数 ``f`` 不能从内部调用（即 ``f`` 不起作用，但 ``this.f()`` 可以）。

``public`` 
    public 函数是合约接口的一部分，可以在内部或通过消息调用。

``internal``
    内部可见性函数访问可以在当前合约或派生的合约访问，不可以外部访问。
    由于它们没有通过合约的ABI向外部公开，它们可以接受内部可见性类型的参数：比如映射或存储引用。

``private``
    private 函数和状态变量仅在当前定义它们的合约中使用，并且不能被派生合约使用。

.. warning::
    设置为 ``private``或 ``internal``，只能防止其他合约读取或修改信息，但它仍然可以在链外查看到。

可见性标识符的定义位置，对于状态变量来说是在类型后面，对于函数是在参数列表和返回关键字中间。

.. code-block:: solidity

    pragma solidity  >=0.4.16 <0.9.0;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }

在下面的例子中，``D`` 可以调用 ``c.getData（）`` 来获取状态存储中 ``data`` 的值，但不能调用 ``f`` 。
合约 ``E`` 继承自 ``C`` ，因此可以调用 ``compute``。

.. code-block:: solidity

    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        uint private data;

        function f(uint a) private returns(uint b) { return a + 1; }
        function setData(uint a) public { data = a; }
        function getData() public returns(uint) { return data; }
        function compute(uint a, uint b) internal returns (uint) { return a+b; }
    }

    // 下面代码编译错误
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
该函数没有参数，返回值是一个 ``uint`` 类型，即状态变量 ``data`` 的值。
状态变量的初始化可以在声明时完成。

.. code-block:: solidity

    pragma solidity  >=0.4.16 <0.9.0;

    contract C {
        uint public data = 42;
    }

    contract Caller {
        C c = new C();
        function f() public {
            uint local = c.data();
        }
    }

getter 函数具有外部（external）可见性。如果在内部访问 getter（即没有 ``this.`` ），它被认为一个状态变量。
如果使用外部访问（即用 ``this.`` ），它被认作为一个函数。

.. code-block:: solidity

    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        uint public data;
        function x() public {
            data = 3; // 内部访问
            uint val = this.data(); // 外部访问
        }
    }

如果你有一个数组类型的 ``public`` 状态变量，那么你只能通过生成的 getter 函数访问数组的单个元素。
这个机制以避免返回整个数组时的高成本gas。 可以使用如 ``myArray(0)`` 用于指定参数要返回的单个元素。
如果要在一次调用中返回整个数组，则需要写一个函数，例如：

.. code-block:: solidity

  pragma solidity >=0.4.0 <0.9.0;

  contract arrayExample {
    // public state variable
    uint[] public myArray;

    // 指定生成的Getter 函数
    /*
    function myArray(uint i) public view returns (uint) {
        return myArray[i];
    }
    */

    // 返回整个数组
    function getArray() public view returns (uint[] memory) {
        return myArray;
    }
  }

现在可以使用 ``getArray()`` 获得整个数组，而 ``myArray(i)`` 是返回单个元素。

下一个例子稍微复杂一些：

.. code-block:: solidity

    pragma solidity ^0.4.0 <0.9.0;

    contract Complex {
        struct Data {
            uint a;
            bytes3 b;
            mapping (uint => uint) map;
            uint[3] c;
            uint[] d;
            bytes e;
        }
        mapping (uint => mapping(bool => Data[])) public data;
    }

这将会生成以下形式的函数，在结构体内的映射和数组（byte 数组除外）被省略了，因为没有好办法为单个结构成员或为映射提供一个键。

.. code-block:: solidity

    function data(uint arg1, bool arg2, uint arg3)
        public
        returns (uint a, bytes3 b, bytes memory e)
    {
        a = data[arg1][arg2][arg3].a;
        b = data[arg1][arg2][arg3].b;
        e = data[arg1][arg2][arg3].e;
    }
