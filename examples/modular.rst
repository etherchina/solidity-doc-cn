.. index:: contract;modular, modular contract

*********************
库合约使用
*********************

通过在合约中引入模块化方法（），可以帮助我们减少溢出等风险。
在下面的合约例子中，引入 ``Balances`` :ref:`库 <libraries>` 合约使用了 ``move`` 方法， 去检查地址余额是否符合预期。

现在大家只需了解下库的作用，后面的文档有 :ref:`更多关于库的使用 <libraries>`。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.5.0  <0.9.0;

    library Balances {
        function move(mapping(address => uint256) storage balances, address from, address to, uint amount) internal {
            require(balances[from] >= amount);
            require(balances[to] + amount >= balances[to]);
            balances[from] -= amount;
            balances[to] += amount;
        }
    }

    contract Token {
        mapping(address => uint256) balances;
        using Balances for *;   // 引入库
        mapping(address => mapping (address => uint256)) allowed;

        event Transfer(address from, address to, uint amount);
        event Approval(address owner, address spender, uint amount);


        function transfer(address to, uint amount) external returns (bool success) {
            balances.move(msg.sender, to, amount);
            emit Transfer(msg.sender, to, amount);
            return true;

        }

        function transferFrom(address from, address to, uint amount) external returns (bool success) {
            require(allowed[from][msg.sender] >= amount);
            allowed[from][msg.sender] -= amount;
            balances.move(from, to, amount);   // 使用了库方法
            emit Transfer(from, to, amount);
            return true;
        }

        function approve(address spender, uint tokens) external returns (bool success) {
            require(allowed[msg.sender][spender] == 0, "");
            allowed[msg.sender][spender] = tokens;
            emit Approval(msg.sender, spender, tokens);
            return true;
        }

        function balanceOf(address tokenOwner) external view returns (uint balance) {
            return balances[tokenOwner];
        }
    }