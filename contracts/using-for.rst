


.. index:: ! using for, library

.. _using-for:

**********
Using For
**********

在当前的合约上下里, 指令 ``using A for B;`` 可用于附加库函数（从库 ``A``）到任何类型（ ``B``）作为成员函数。
这些函数将接收到调用它们的对象作为它们的第一个参数（像 Python 的 ``self`` 变量）。

Using For 可在文件或合约内部及合约级都是有效的。

第一部分 ``A`` 可以是以下之一：

- 一些库或文件级的函数列表(``using {f, g, h, L.t} for uint;``)， 仅是那些函数被附加到类型。
- 库名称 (``using L for uint;``) ，库里所有的函数(包括 public 和 internal 函数) 被附加到类型上。

在文件级，第二部分 ``B`` 必须是一个显式类型（不用指定数据位置）

在合约内，你可以使用 ``using L for *;``， 表示库 ``L`` 中的函数被附加在所有类型上。

如果你指定一个库， 库内所有函数都会被加载，即使它们的第一个参数类型与对象的类型不匹配。类型检查会在函数调用和重载解析时执行。

如果你使用函数列表 (``using {f, g, h, L.t} for uint;``)， 那么类型 (``uint``) 会隐式的转换为这些函数的第一个参数。
即便这些函数中没有一个被调用，这个检查也会进行。

``using A for B;`` 指令仅在当前作用域有效（要么是合约中，或当前模块、或源码单元），包括在作用域内的所有函数，在合约或模块之外则无效。

当 ``using for`` 指令在文件级别使用，并应用于一个用户定义类型（在用一个文件定义的文件级别的用户类型）， ``global`` 关键字可以添加到末尾。
产生的效果是，这些函数被附加到使用该类型的任何地方（包括其他文件），而不仅仅是声明处所在的作用域。

让我们重写一下来自 :ref:`libraries` 一节中的例子，使用文件级函数而不是库函数的方式重写。


.. code-block:: solidity

    struct Data { mapping(uint => bool) flags; }
    // Now we attach functions to the type.
    // The attached functions can be used throughout the rest of the module.
    // If you import the module, you have to
    // repeat the using directive there, for example as
    //   import "flags.sol" as Flags;
    //   using {Flags.insert, Flags.remove, Flags.contains}
    //     for Flags.Data;
    using {insert, remove, contains} for Data;

    function insert(Data storage self, uint value)
        returns (bool)
    {
        if (self.flags[value])
            return false; // already there
        self.flags[value] = true;
        return true;
    }

    function remove(Data storage self, uint value)
        returns (bool)
    {
        if (!self.flags[value])
            return false; // not there
        self.flags[value] = false;
        return true;
    }

    function contains(Data storage self, uint value)
        public
        view
        returns (bool)
    {
        return self.flags[value];
    }


    contract C {
        Data knownValues;

        function register(uint value) public {
            // Here, all variables of type Data have
            // corresponding member functions.
            // The following function call is identical to
            // `Set.insert(knownValues, value)`
            require(knownValues.insert(value));
        }
    }

也可以像这样扩展内置基本类型，在下面的例子中，我们将使用库：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.13;

    library Search {
        function indexOf(uint[] storage self, uint value)
            public
            view
            returns (uint)
        {
            for (uint i = 0; i < self.length; i++)
                if (self[i] == value) return i;
            return type(uint).max;
        }
    }

    using Search for uint[];

    contract C {
        using Search for uint[];
        uint[] data;

        function append(uint value) public {
            data.push(value);
        }

        function replace(uint from, uint to) public {
            // 执行库函数调用
            uint index = data.indexOf(from);
            if (index == type(uint).max)
                data.push(to);
            else
                data[index] = to;
        }
    }

注意，所有 external 库调用都是实际的 EVM 函数调用。这意味着如果传递内存或值类型，都将产生一个副本，即使是 ``self`` 变量。
引用存储变量或者 internal 库调用 是唯一不会发生拷贝的情况。