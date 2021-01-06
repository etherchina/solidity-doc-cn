.. include:: glossaries.rst
.. index:: !mapping

.. _mapping-types:

映射
=====

映射类型在声明时的形式为 ``mapping(_KeyType => _ValueType)``。
其中 ``_KeyType`` 可以是任何基本类型，即可以是任何的内建类型加上``bytes`` 和 ``string``。
而用户定义的类型或复杂的类型如：合约类型、枚举、映射、结构体、即除 ``bytes`` 和 ``string`` 之外的数组类型 是不可以作为 ``_KeyType`` 的类型的。

``_ValueType`` 可以是包括映射类型在内的任何类型。

映射可以视作 `哈希表 <https://en.wikipedia.org/wiki/Hash_table>`_ ，它们在实际的初始化过程中创建每个可能的 key，
并将其映射到字节形式全是零的值：一个类型的 :ref:`默认值 <default-value>`。然而下面是映射与哈希表不同的地方：
在映射中，实际上并不存储 key，而是存储它的 ``keccak256`` 哈希值，从而便于查询实际的值。

正因为如此，映射是没有长度的，也没有 key 的集合或 value 的集合的概念。
，因此如果没有其他信息键的信息是无法被删除（请参阅 :ref:`clearing-mappings` ）。


映射只能是 |storage| 的数据位置，因此只允许作为状态变量 或 作为函数内的 |storage| 引用 或 作为库函数的参数。
它们不能用合约公有函数的参数或返回值。

这些限制同样适用于包含映射的数组和结构体。

可以将映射声明为 ``public`` ，然后来让 Solidity 创建一个 :ref:`getter 函数 <visibility-and-getters>`。
``_KeyType`` 将成为 getter 的必须参数，并且 getter 会返回 ``_ValueType`` 。

如果 ``_ValueType`` 是一个映射。这时在使用 getter 时将需要递归地传入每个 ``_KeyType`` 参数，　


在下面的示例中，　``MappingExample`` 　合约定义了一个公共　``balances``　映射，键类型为 ``address``，值类型为 ``uint``，　
将以太坊地址映射为 无符号整数值。 由于　``uint``　是值类型，因此getter返回与该类型匹配的值，
可以在　``MappingLBC``　合约中看到合约在指定地址返回该值。


::

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.8.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingLBC {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(this);
        }
    }


下面的例子是　`ERC20 token <https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol>`_　的简单版本．
``_allowances`` 是一个嵌套mapping的例子．
``_allowances`` 用来记录其他的账号，可以允许从其账号使用多少数量的币．


::

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.8.0;

    contract MappingExample {

        mapping (address => uint256) private _balances;
        mapping (address => mapping (address => uint256)) private _allowances;

        event Transfer(address indexed from, address indexed to, uint256 value);
        event Approval(address indexed owner, address indexed spender, uint256 value);

        function allowance(address owner, address spender) public view returns (uint256) {
            return _allowances[owner][spender];
        }

        function transferFrom(address sender, address recipient, uint256 amount) public returns (bool) {
            _transfer(sender, recipient, amount);
            approve(sender, msg.sender, amount);
            return true;
        }

        function approve(address owner, address spender, uint256 amount) public returns (bool) {
            require(owner != address(0), "ERC20: approve from the zero address");
            require(spender != address(0), "ERC20: approve to the zero address");

            _allowances[owner][spender] = amount;
            emit Approval(owner, spender, amount);
            return true;
        }

        function _transfer(address sender, address recipient, uint256 amount) internal {
            require(sender != address(0), "ERC20: transfer from the zero address");
            require(recipient != address(0), "ERC20: transfer to the zero address");

            _balances[sender] -= amount;
            _balances[recipient] += amount;
            emit Transfer(sender, recipient, amount);
        }
    }


.. index:: !iterable mappings
.. _iterable-mappings:

可迭代映射
-----------------


映射本身是无法遍历的，即无法枚举所有的键。不过，可以在它们之上实现一个数据结构来进行迭代。 例如，以下代码实现了
``IterableMapping`` 库，然后　``User`` 合约可以添加数据，　``sum``　函数迭代求和所有值。


::

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.8.0;

    struct IndexValue { uint keyIndex; uint value; }
    struct KeyFlag { uint key; bool deleted; }

    struct itmap {
        mapping(uint => IndexValue) data;
        KeyFlag[] keys;
        uint size;
    }

    library IterableMapping {
        function insert(itmap storage self, uint key, uint value) internal returns (bool replaced) {
            uint keyIndex = self.data[key].keyIndex;
            self.data[key].value = value;
            if (keyIndex > 0)
                return true;
            else {
                keyIndex = self.keys.length;

                self.keys.push();
                self.data[key].keyIndex = keyIndex + 1;
                self.keys[keyIndex].key = key;
                self.size++;
                return false;
            }
        }

        function remove(itmap storage self, uint key) internal returns (bool success) {
            uint keyIndex = self.data[key].keyIndex;
            if (keyIndex == 0)
                return false;
            delete self.data[key];
            self.keys[keyIndex - 1].deleted = true;
            self.size --;
        }

        function contains(itmap storage self, uint key) internal view returns (bool) {
            return self.data[key].keyIndex > 0;
        }

        function iterate_start(itmap storage self) internal view returns (uint keyIndex) {
            return iterate_next(self, uint(-1));
        }

        function iterate_valid(itmap storage self, uint keyIndex) internal view returns (bool) {
            return keyIndex < self.keys.length;
        }

        function iterate_next(itmap storage self, uint keyIndex) internal view returns (uint r_keyIndex) {
            keyIndex++;
            while (keyIndex < self.keys.length && self.keys[keyIndex].deleted)
                keyIndex++;
            return keyIndex;
        }

        function iterate_get(itmap storage self, uint keyIndex) internal view returns (uint key, uint value) {
            key = self.keys[keyIndex].key;
            value = self.data[key].value;
        }
    }

    // 如何使用
    contract User {
        // Just a struct holding our data.
        itmap data;
        // Apply library functions to the data type.
        using IterableMapping for itmap;

        // Insert something
        function insert(uint k, uint v) public returns (uint size) {
            // This calls IterableMapping.insert(data, k, v)
            data.insert(k, v);
            // We can still access members of the struct,
            // but we should take care not to mess with them.
            return data.size;
        }

        // Computes the sum of all stored data.
        function sum() public view returns (uint s) {
            for (
                uint i = data.iterate_start();
                data.iterate_valid(i);
                i = data.iterate_next(i)
            ) {
                (, uint value) = data.iterate_get(i);
                s += value;
            }
        }
    }


