.. include:: glossaries.rst

###############
通用模式
###############

.. index:: withdrawal

.. _withdrawal_pattern:

*************************
从合约中提款
*************************

在某个操作之后发送资金的推荐方式是使用取回（withdrawal）模式。尽管在某个操作之后，最直接地发送以太币方法是一个 ``send`` 调用，
但这并不推荐；因为这会引入一个潜在的安全风险。你可能需要参考 :ref:`security_considerations` 来获取更多信息。

这里是一个在合约中使用取回模式的示例，它目标是通过向合约发送最多的钱来成为“最富有的人”，
其灵感来自 `King of the Ether <https://www.kingoftheether.com/>`_。

在下边的合约中，如果你的“最富有”位置被其他人取代，你可以收到取代你成为“最富有”的人发送到合约的资金。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    contract WithdrawalContract {
        address public richest;
        uint public mostSent;

        mapping (address => uint) pendingWithdrawals;


        /// The amount of Ether sent was not higher than
        /// the currently highest amount.
        error NotEnoughEther();

        constructor() payable {
            richest = msg.sender;
            mostSent = msg.value;
        }

        function becomeRichest() public payable {
            if (msg.value <= mostSent) revert NotEnoughEther();
            pendingWithdrawals[richest] += msg.value;
            richest = msg.sender;
            mostSent = msg.value;
        }

        function withdraw() public {
            uint amount = pendingWithdrawals[msg.sender];
            // 记住，在发送资金之前将待发金额清零
            // 来防止重入（re-entrancy）攻击
            pendingWithdrawals[msg.sender] = 0;
            payable(msg.sender).transfer(amount);
        }
    }

下面是一个相反的直接使用发送模式的例子：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    contract SendContract {
        address payable public richest;
        uint public mostSent;


        /// 发送的数量比当前最大的数量少时，触发这个错误
        error NotEnoughEther();

        constructor() payable {
            richest = payable(msg.sender);
            mostSent = msg.value;
        }

        function becomeRichest() public payable returns (bool) {
            if (msg.value <= mostSent) revert NotEnoughEther();

            // 这一行会导致问题（详见下文）
            richest.transfer(msg.value);
            richest = payable(msg.sender);
            mostSent = msg.value;
        }
    }

注意，在这个例子里，攻击者可以给这个合约设下陷阱，使其进入不可用状态，比如通过使一个 fallback 或 receive 函数会失败的合约成为 ``richest``
（可以在 fallback 函数中调用 ``revert()`` 或者直接在 fallback 函数中使用超过 2300 gas 来使其执行失败）。这样，当这个合约调用 ``transfer`` 来给“下过毒”的合约
发送资金时，调用会失败，从而导致 ``becomeRichest`` 函数失败，这个合约也就被永远卡住了。

如果在合约中像第一个例子那样使用“取回（withdraw）”模式，那么攻击者只能使他/她自己的“取回”失败，并不会导致整个合约无法运作。

.. index:: access;restricting

******************
限制访问
******************

限制访问是合约的一个通用模式。注意，你不可能限制任何人或机器读取你的交易内容或合约状态。
你可以通过加密使这种访问变得困难一些，但如果你想让你的合约读取这些数据，那么其他人也将可以做到。

你可以限制 **其他合约** 读取你的合约状态。
这（其他合约不能读取你的合约状态）是默认的，除非你将合约状态变量声明为 ``public``。

此外，你可以对谁可以修改你的合约状态或调用你的合约函数加以限制，这是本节要介绍的内容。

.. index:: function;modifier

通过使用“函数 |modifier|”，可以使这些限制变得非常明确。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    contract AccessRestriction {
        // 这些将在构造阶段被赋值
        // 其中，`msg.sender` 是创建这个合约的账户。
        address public owner = msg.sender;
        uint public creationTime = block.timestamp;


        // 下面列出了该合约可能产生的错误

        /// 当前操作者没有被授权
        error Unauthorized();

        /// 被过早调用
        error TooEarly();

        /// 没有足够的Ether
        error NotEnoughEther();

        // 修改器可以用来更改
        // 一个函数的函数体。
        // 如果使用这个修改器，
        // 它会预置一个检查，仅允许
        // 来自特定地址的
        // 函数调用。
        modifier onlyBy(address account)
        {
            if (msg.sender != account)
                revert Unauthorized();

            // 不要忘记写 `_;`！
            // 它会被实际使用这个修改器的
            // 函数体所替代。
            _;
        }

        // 使 `newOwner` 成为这个合约的
        // 新所有者。
        function changeOwner(address newOwner)
            public
            onlyBy(owner)
        {
            owner = newOwner;
        }

        modifier onlyAfter(uint time) {
            if (block.timestamp < time)
                revert TooEarly();
            _;
        }

        // 抹掉所有者信息。
        // 仅允许在合约创建成功 6 周以后
        // 的时间被调用。
        function disown()
            public
            onlyBy(owner)
            onlyAfter(creationTime + 6 weeks)
        {
            delete owner;
        }

        // 这个修改器要求对函数调用
        // 绑定一定的费用。
        // 如果调用方发送了过多的费用，
        // 他/她会得到退款，但需要先执行函数体。
        // 这在 0.4.0 版本以前的 Solidity 中很危险，
        // 因为很可能会跳过 `_;` 之后的代码。
        modifier costs(uint amount) {
            if (msg.value < amount)
                revert NotEnoughEther();
            _;
            if (msg.value > amount)
                payable(msg.sender).send(msg.value - amount);
        }

        function forceOwnerChange(address newOwner)
            public
            payable
            costs(200 ether)
        {
            owner = newOwner;
            // 这只是示例条件
            if (uint160(owner) & 0 == 1)
                // 这无法在 0.4.0 版本之前的
                // Solidity 上进行退还。
                return;
            // 退还多付的费用
        }
    }

一个更专用地限制函数调用的方法将在下一个例子中介绍。

.. index:: state machine

*************
状态机
*************

合约通常会像状态机那样运作，这意味着它们有特定的 **阶段**，使它们有不同的表现或者仅允许特定的不同函数被调用。
一个函数调用通常会结束一个阶段，并将合约转换到下一个阶段（特别是如果一个合约是以 **交互** 来建模的时候）。
通过达到特定的 **时间** 点来达到某些阶段也是很常见的。

一个典型的例子是盲拍（blind auction）合约，它起始于“接受盲目出价”，
然后转换到“公示出价”，最后结束于“确定拍卖结果”。

.. index:: function;modifier

函数 |modifier| 可以用在这种情况下来对状态进行建模，并确保合约被正常的使用。

示例
=======

在下边的示例中， |modifier| ``atStage`` 确保了函数仅在特定的阶段才可以被调用。

根据时间来进行的自动阶段转换，是由 |modifier| ``timedTransitions`` 来处理的，
它应该用在所有函数上。

.. note::
    |modifier| **的顺序非常重要**。
    如果 atStage 和 timedTransitions 要一起使用，
    请确保在 timedTransitions 之后声明 atStage，
    以便新的状态可以
    首先被反映到账户中。

最后， |modifier| ``transitionNext`` 能够用来在函数执行结束时自动转换到下一个阶段。

.. note::
    |modifier| **可以被忽略**。
    以下特性仅在 0.4.0 版本之前的 Solidity 中有效：
    由于 |modifier| 是通过简单的替换代码
    而不是使用函数调用来提供的，
    如果函数本身使用了 return，那么 transitionNext |modifier|
    的代码是可以被忽略的。
    如果你希望这么做，
    请确保你在这些函数中手工调用了 nextStage。
    从 0.4.0 版本开始，即使函数明确地 return 了，
    |modifier| 的代码也会执行。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;


    contract StateMachine {
        enum Stages {
            AcceptingBlindedBids,
            RevealBids,
            AnotherStage,
            AreWeDoneYet,
            Finished
        }

        /// Function cannot be called at this time.
        error FunctionInvalidAtThisStage();
        
        // 这是当前阶段。
        Stages public stage = Stages.AcceptingBlindedBids;

        uint public creationTime = block.timestamp;

        modifier atStage(Stages stage_) {
            if (stage != stage_)
                revert FunctionInvalidAtThisStage();
            _;
        }

        function nextStage() internal {
            stage = Stages(uint(stage) + 1);
        }

        // 执行基于时间的阶段转换。
        // 请确保首先声明这个修改器，
        // 否则新阶段不会被带入账户。
        modifier timedTransitions() {
            if (stage == Stages.AcceptingBlindedBids &&
                        block.timestamp >= creationTime + 10 days)
                nextStage();
            if (stage == Stages.RevealBids &&
                    block.timestamp >= creationTime + 12 days)
                nextStage();
            // 由交易触发的其他阶段转换
            _;
        }

        // 这里的修改器顺序非常重要！
        function bid()
            public
            payable
            timedTransitions
            atStage(Stages.AcceptingBlindedBids)
        {
            // 我们不会在这里实现实际功能（因为这仅是个代码示例，译者注）
        }

        function reveal()
            public
            timedTransitions
            atStage(Stages.RevealBids)
        {
        }

        // 这个修改器在函数执行结束之后
        // 使合约进入下一个阶段。
        modifier transitionNext()
        {
            _;
            nextStage();
        }

        function g()
            public
            timedTransitions
            atStage(Stages.AnotherStage)
            transitionNext
        {
        }

        function h()
            public
            timedTransitions
            atStage(Stages.AreWeDoneYet)
            transitionNext
        {
        }

        function i()
            public
            timedTransitions
            atStage(Stages.Finished)
        {
        }
    }
