


.. index:: ! using for, library

.. _using-for:

*********
Using For
*********

在当前的合约上下里, 指令 ``using A for B;`` 可用于附加库函数（从库 ``A``）到任何类型（``B``）。
这些函数将接收到调用它们的对象作为它们的第一个参数（像 Python 的 ``self`` 变量）。

``using A for *;`` 的效果是，库 ``A`` 中的函数被附加在任意的类型上。

在这两种情况下，所有函数都会被附加一个参数，即使它们的第一个参数类型与对象的类型不匹配。
函数调用和重载解析时才会做类型检查。

``using A for B;`` 指令仅在当前作用域有效，目前仅限于在当前合约中，后续可能提升到全局范围。
通过引入一个模块，不需要再添加代码就可以使用包括库函数在内的数据类型。

让我们用这种方式将 :ref:`libraries` 中的 set 例子重写::

    pragma solidity >=0.4.16 <0.7.0;

    // 这是和之前一样的代码，只是没有注释。
    struct Data { mapping(uint => bool) flags; }

    library Set {

      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
            return false; // 已经存在
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // 不存在
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
    }

    contract C {
        using Set for Data; // 这里是关键的修改
        Data knownValues;

        function register(uint value) public {
            // Here, all variables of type Data have
            // corresponding member functions.
            // The following function call is identical to
            // `Set.insert(knownValues, value)`
            // 这里， Data 类型的所有变量都有与之相对应的成员函数。
            // 下面的函数调用和 `Set.insert(knownValues, value)` 的效果完全相同。
            require(knownValues.insert(value));
        }
    }

也可以像这样扩展基本类型::

    pragma solidity >=0.4.16 <0.7.0;

    library Search {
        function indexOf(uint[] storage self, uint value)
            public
            view
            returns (uint)
        {
            for (uint i = 0; i < self.length; i++)
                if (self[i] == value) return i;
            return uint(-1);
        }
    }

    contract C {
        using Search for uint[];
        uint[] data;

        function append(uint value) public {
            data.push(value);
        }

        function replace(uint _old, uint _new) public {
            // 执行库函数调用
            uint index = data.indexOf(_old);
            if (index == uint(-1))
                data.push(_new);
            else
                data[index] = _new;
        }
    }

注意，所有 external 库调用都是实际的 EVM 函数调用。这意味着如果传递内存或值类型，都将产生一个副本，即使是 ``self`` 变量。
引用存储变量或者 internal 库调用 是唯一不会发生拷贝的情况。