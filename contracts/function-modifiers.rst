.. include:: glossaries.rst
.. index:: ! function;modifier

.. _modifiers:

******************
函数 |modifier|
******************

使用 |modifier| 可以轻松改变函数的行为。 例如，它们可以在执行函数之前自动检查某个条件。
|modifier| 是合约的可继承属性，并可能被派生合约覆盖 , 但前提是它们被标记为 ``virtual``.。 有关详细信息，请参见 :ref:`Modifier 重载 <modifier-overriding>`.


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.1 <0.9.0;

    contract owned {
        constructor() { owner = payable(msg.sender); }

        address owner;

        // 这个合约只定义一个修改器，但并未使用： 它将会在派生合约中用到。
        // 修改器所修饰的函数体会被插入到特殊符号 _; 的位置。
        // 这意味着如果是 owner 调用这个函数，则函数会被执行，否则会抛出异常。
        modifier onlyOwner {
            require(
                msg.sender == owner,
                "Only owner can call this function."
            );
            _;
        }
    }

    contract destructible is owned {
        // 这个合约从 `owned` 继承了 `onlyOwner` 修饰符，并将其应用于 `destroy` 函数，
        // 只有在合约里保存的 owner 调用 `destroy` 函数，才会生效。
        function destroy() public onlyOwner {
            selfdestruct(owner);
        }
    }

    contract priced {
        // 修改器可以接收参数：
        modifier costs(uint price) {
            if (msg.value >= price) {
                _;
            }
        }
    }

    contract Register is priced, destructible {
        mapping (address => bool) registeredAddresses;
        uint price;

        constructor(uint initialPrice) { price = initialPrice; }

        // 在这里也使用关键字 `payable` 非常重要，否则函数会自动拒绝所有发送给它的以太币。
        function register() public payable costs(price) {
            registeredAddresses[msg.sender] = true;
        }

        function changePrice(uint price_) public onlyOwner {
            price = price_;
        }
    }

    contract Mutex {
        bool locked;
        modifier noReentrancy() {
            require(
                !locked,
                "Reentrant call."
            );
            locked = true;
            _;
            locked = false;
        }

        // 这个函数受互斥量保护，这意味着 `msg.sender.call` 中的重入调用不能再次调用  `f`。
        // `return 7` 语句指定返回值为 7，但修改器中的语句 `locked = false` 仍会执行。
        function f() public noReentrancy returns (uint) {
            (bool success,) = msg.sender.call("");
            require(success);
            return 7;
        }
    }

如果你想访问定义在合约 ``C`` 的 |modifier|  ``m`` ， 可以使用 ``C.m`` 去引用它，而不需要使用虚拟表查找。

只能使用在当前合约或在基类合约中定义的 |modifier| , |modifier| 也可以定义在库里面，但是他们被限定在库函数使用。

如果同一个函数有多个 |modifier|，它们之间以空格隔开，|modifier| 会依次检查执行。

修改器不能隐式地访问或改变它们所修饰的函数的参数和返回值。
这些值只能在调用时明确地以参数传递。

|modifier| 或函数体中显式的 return 语句仅仅跳出当前的 |modifier| 和函数体。
返回变量会被赋值，但整个执行逻辑会从前一个 |modifier| 中的定义的 “_” 之后继续执行。


.. warning::
    在早期的 Solidity 版本中，有 |modifier| 的函数， ``return`` 语句的行为表现不同。



用 ``return;`` 从修改器中显式返回并不影响函数返回值。
然而，修改器可以选择完全不执行函数体，在这种情况下，返回的变量被设置为:ref:`默认值<default-value>`，就像该函数是空函数体一样。


``_`` 符号可以在修改器中出现多次，每处都会替换为函数体。

|modifier| 的参数可以是任意表达式，在此上下文中，所有在函数中可见的符号，在 |modifier| 中均可见。
在 |modifier| 中引入的符号在函数中不可见（可能被重载改变）。
