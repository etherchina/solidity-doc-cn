.. index:: ! contract;creation, constructor

**********
创建合约
**********

可以通过以太坊交易“从外部”或从 Solidity 合约内部创建合约。

一些集成开发环境，例如 `Remix <https://remix.ethereum.org/>`_, 通过使用一些UI用户界面使创建合约的过程更加顺畅。
在以太坊上通过编程创建合约最好使用 JavaScript API `web3.js <https://github.com/ethereum/web3.js>`_。
现在，我们已经有了一个叫做 `web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_ 的方法能够更容易的创建合约。

创建合约时， 合约的 :ref:`构造函数 <constructor>`  (一个用关键字 ``constructor`` 声明的函数)会执行一次。
构造函数是可选的。只允许有一个构造函数，这意味着不支持重载。

构造函数执行完毕后，合约的最终代码将部署到区块链上。此代码包括所有公共和外部函数以及所有可以通过函数调用访问的函数。 部署的代码没有
包括构造函数代码或构造函数调用的内部函数。

.. index:: constructor;arguments

在内部，构造函数参数在合约代码之后通过 :ref:`ABI 编码 <ABI>` 传递，但是如果你使用 ``web3.js``
则不必关心这个问题。

如果一个合约想要创建另一个合约，那么创建者必须知晓被创建合约的源代码(和二进制代码)。
这意味着不可能循环创建依赖项。

.. code-block:: solidity
    
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    contract OwnedToken {

        // TokenCreator 是后面定义的合约类型.
        // 不创建新合约的话，也可以引用它。
        TokenCreator creator;
        address owner;
        bytes32 name;

        // 这是注册 creator 和设置名称的构造函数。
        constructor(bytes32 name_) {
            
            // 状态变量通过其名称访问，而不是通过例如 this.owner 的方式访问。
            // 这也适用于函数，特别是在构造函数中，你只能像这样（“内部地”）调用它们，
            // 因为合约本身还不存在。
            owner = msg.sender;
            // 从 `address` 到 `TokenCreator` ，是做显式的类型转换
            // 并且假定调用合约的类型是 TokenCreator，没有真正的方法来检查这一点。
            creator = TokenCreator(msg.sender);
            name = name_;
        }

        function changeName(bytes32 newName) public {
            
            // 只有 creator （即创建当前合约的合约）能够更改名称 —— 因为合约是隐式转换为地址的，
            // 所以这里的比较是可行的。
            if (msg.sender == address(creator))
                name = newName;
        }

        function transfer(address newOwner) public {
            // 只有当前所有者才能发送 token。
            if (msg.sender != owner) return;
            // 我们也想询问 creator 是否可以发送。
            // 请注意，这里调用了一个下面定义的合约中的函数。
            // 如果调用失败（比如，由于 gas 不足），会立即停止执行。
            if (creator.isTokenTransferOK(owner, newOwner))
                owner = newOwner;
        }
    }

    contract TokenCreator {
        function createToken(bytes32 name)
        public
        returns (OwnedToken tokenAddress) {
            // 创建一个新的 Token 合约并且返回它的地址。
            // 从 JavaScript 方面来说，返回类型是简单的 `address` 类型，因为
            // 这是在 ABI 中可用的最接近的类型。
            return new OwnedToken(name);
        }

        function changeName(OwnedToken tokenAddress, bytes32 name)  public {
            // 同样，`tokenAddress` 的外部类型也是 `address` 。
            tokenAddress.changeName(name);
        }

        function isTokenTransferOK(address currentOwner, address newOwner)
            public
            view
            returns (bool ok)
        {
            // 检查一些任意的情况。
            address tokenAddress = msg.sender;
            return (keccak256(newOwner) & 0xff) == (bytes20(tokenAddress) & 0xff);
        }
    }
