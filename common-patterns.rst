###############
通用模式
###############

.. index:: withdrawal

.. _withdrawal_pattern:

*************************
从合约中提款
*************************

The recommended method of sending funds after an effect
is using the withdrawal pattern. Although the most intuitive
method of sending Ether, as a result of an effect, is a
direct ``send`` call, this is not recommended as it
introduces a potential security risk. You may read
more about this on the :ref:`security_considerations` page.
在某个操作之后发送资金的推荐方式是使用取回（withdrawal）模式。尽管在某个操作之后，最直接地发送以太币方法是一个 ``send`` 调用，
但这并不推荐；因为这会引入一个潜在的安全风险。你可能需要参考 :ref:`security_considerations` 来获取更多信息。

This is an example of the withdrawal pattern in practice in
a contract where the goal is to send the most money to the
contract in order to become the "richest", inspired by
`King of the Ether <https://www.kingoftheether.com/>`_.
这里是一个在合约中使用取回模式的示例，它目标是通过向合约发送最多的钱来成为“最富有的人”，
其灵感来自 `King of the Ether <https://www.kingoftheether.com/>`_。

In the following contract, if you are usurped as the richest,
you will receive the funds of the person who has gone on to
become the new richest.
在下边的合约中，如果你的“最富有”位置被其他人取代，你可以收到取代你成为“最富有”的人发送到合约的资金。

::

    pragma solidity ^0.4.11;

    contract WithdrawalContract {
        address public richest;
        uint public mostSent;

        mapping (address => uint) pendingWithdrawals;

        function WithdrawalContract() public payable {
            richest = msg.sender;
            mostSent = msg.value;
        }

        function becomeRichest() public payable returns (bool) {
            if (msg.value > mostSent) {
                pendingWithdrawals[richest] += msg.value;
                richest = msg.sender;
                mostSent = msg.value;
                return true;
            } else {
                return false;
            }
        }

        function withdraw() public {
            uint amount = pendingWithdrawals[msg.sender];
            // 记住，在发送资金之前将待发金额清零
            // 来防止重入（re-entrancy）攻击
            pendingWithdrawals[msg.sender] = 0;
            msg.sender.transfer(amount);
        }
    }

This is as opposed to the more intuitive sending pattern:
下面是一个相反的直接使用发送模式的例子：

::

    pragma solidity ^0.4.11;

    contract SendContract {
        address public richest;
        uint public mostSent;

        function SendContract() public payable {
            richest = msg.sender;
            mostSent = msg.value;
        }

        function becomeRichest() public payable returns (bool) {
            if (msg.value > mostSent) {
                // 这一行会导致问题（详见下文）
                richest.transfer(msg.value);
                richest = msg.sender;
                mostSent = msg.value;
                return true;
            } else {
                return false;
            }
        }
    }

Notice that, in this example, an attacker could trap the
contract into an unusable state by causing ``richest`` to be
the address of a contract that has a fallback function
which fails (e.g. by using ``revert()`` or by just
consuming more than the 2300 gas stipend). That way,
whenever ``transfer`` is called to deliver funds to the
"poisoned" contract, it will fail and thus also ``becomeRichest``
will fail, with the contract being stuck forever.
注意，在这个例子里，攻击者可以给这个合约设下陷阱，使一个 fallback 函数会失败（例如在 fallback 函数中使用 ``revert()`` 或者
直接在 fallback 函数中使用超过 2300 gas）的合约成为 ``richest``。这样，当这个合约调用 ``transfer`` 来给“下过毒”的合约
发送资金时，调用会失败，从而导致 ``becomeRichest`` 函数失败，这个合约也就被永远卡住了。

In contrast, if you use the "withdraw" pattern from the first example,
the attacker can only cause his or her own withdraw to fail and not the
rest of the contract's workings.
如果在合约中像第一个例子那样使用“取回（withdraw）”模式，那么攻击者只能使他/她自己的“取回”失败，并不会导致整个合约无法运作。

.. index:: access;restricting

******************
限制访问
******************

Restricting access is a common pattern for contracts.
Note that you can never restrict any human or computer
from reading the content of your transactions or
your contract's state. You can make it a bit harder
by using encryption, but if your contract is supposed
to read the data, so will everyone else.
限制访问是合约的一个通用模式。注意，你不可能限制任何人或机器读取你的交易或合约状态内容。
你可以通过加密使这种访问变得困难一些，但如果你想让你的合约读取这些数据，那么其他人也将可以同样做到。

You can restrict read access to your contract's state
by **other contracts**. That is actually the default
unless you declare make your state variables ``public``.
你可以限制**其他合约**读取你的合约状态。
这（其他合约不能读取你的合约状态）是默认的，除非你将合约状态变量声明为 ``public``。

Furthermore, you can restrict who can make modifications
to your contract's state or call your contract's
functions and this is what this section is about.
此外，你可以对谁可以修改你的合约状态或调用你的合约函数加以限制，这是本节要介绍的内容。

.. index:: function;modifier

The use of **function modifiers** makes these
restrictions highly readable.
通过使用**函数 |modifier| **，可以使这些限制变得非常明确。

::

    pragma solidity ^0.4.11;

    contract AccessRestriction {
        // 这些将在构造阶段被赋值
        // 其中，`msg.sender` 是
        // 创建这个合约的账户。
        address public owner = msg.sender;
        uint public creationTime = now;

        // 修饰器可以用来更改
        // 一个函数的函数体。
        // 如果使用这个修饰器，
        // 它会预置一个检查，仅允许
        // 来自特定地址的
        // 函数调用。
        modifier onlyBy(address _account)
        {
            require(msg.sender == _account);
            // 不要忘记写“_;“！
            // 它会被实际使用这个修饰器的
            // 函数体所替代。
            _;
        }

        /// 使 `_newOwner` 成为这个合约的
        /// 新所有者。
        function changeOwner(address _newOwner)
            public
            onlyBy(owner)
        {
            owner = _newOwner;
        }

        modifier onlyAfter(uint _time) {
            require(now >= _time);
            _;
        }

        /// 抹掉所有者信息。
        /// 仅允许在合约创建成功 6 周以后
        /// 的时间被调用。
        function disown()
            public
            onlyBy(owner)
            onlyAfter(creationTime + 6 weeks)
        {
            delete owner;
        }

        // 这个修饰器要求对函数调用
        // 绑定一定的费用。
        // 如果调用方发送了过多的费用，
        // 他/她会得到退款，但需要先执行函数体。
        // 这在 0.4.0 版本以前的 Solidity 中很危险，
        // 因为很可能会跳过 `_;` 之后的代码。
        modifier costs(uint _amount) {
            require(msg.value >= _amount);
            _;
            if (msg.value > _amount)
                msg.sender.send(msg.value - _amount);
        }

        function forceOwnerChange(address _newOwner)
            public
            costs(200 ether)
        {
            owner = _newOwner;
            // 这只是示例条件
            if (uint(owner) & 0 == 1)
                // 这无法在 0.4.0 版本之前的
                // Solidity 上进行退还。
                return;
            // 退还多付的费用
        }
    }

A more specialised way in which access to function
calls can be restricted will be discussed
in the next example.
一个更专用地限制函数调用的方法将在下一个例子中介绍。

.. index:: state machine

*************
状态机
*************

Contracts often act as a state machine, which means
that they have certain **stages** in which they behave
differently or in which different functions can
be called. A function call often ends a stage
and transitions the contract into the next stage
(especially if the contract models **interaction**).
It is also common that some stages are automatically
reached at a certain point in **time**.
合约通常会像状态机那样运作，这意味着它们有特定的**阶段**，使它们有不同的表现或者仅允许特定的不同函数被调用。
一个函数调用通常会结束一个阶段，并将合约转换到下一个阶段（特别是如果一个合约是以**交互**来建模的时候）。
通过达到特定的**时间**点来达到某些阶段也是很常见的。

An example for this is a blind auction contract which
starts in the stage "accepting blinded bids", then
transitions to "revealing bids" which is ended by
"determine auction outcome".
这种实例可以是一个盲拍（blind auction）合约，它起始于“接受盲目出价”，
然后转换到“公示出价”，最后结束于“确定拍卖结果”。

.. index:: function;modifier

Function modifiers can be used in this situation
to model the states and guard against
incorrect usage of the contract.
函数 |modifier| 可以用在这种情况下来对状态进行建模，并确保合约被正常的使用。

示例
=======

In the following example,
the modifier ``atStage`` ensures that the function can
only be called at a certain stage.
在下边的示例中， |modifier| ``atStage`` 确保了函数仅在特定的阶段才可以被调用。

Automatic timed transitions
are handled by the modifier ``timeTransitions``, which
should be used for all functions.
根据时间来进行的自动阶段转换，是由 |modifier| ``timeTransitions`` 来处理的，
它应该用在所有函数上。

.. note::
    ** |modifier| 的顺序非常重要**。
    If atStage is combined如果 atStage 和 timedTransitions 要一起使用，
    with timedTransitions, make sure that you mention请确保在 timedTransitions 之后声明 atStage，
    it after the latter, so that the new stage is以便新的状态可以
    taken into account.首先被反映到账户中。

Finally, the modifier ``transitionNext`` can be used
to automatically go to the next stage when the
function finishes.
最后， |modifier| ``transitionNext`` 能够用来在函数执行结束时自动转换到下一个阶段。

.. note::
    **Modifier May be Skipped**.
    This only applies to Solidity before version 0.4.0:
    Since modifiers are applied by simply replacing
    code and not by using a function call,
    the code in the transitionNext modifier
    can be skipped if the function itself uses
    return. If you want to do that, make sure
    to call nextStage manually from those functions.
    Starting with version 0.4.0, modifier code
    will run even if the function explicitly returns.

::

    pragma solidity ^0.4.11;

    contract StateMachine {
        enum Stages {
            AcceptingBlindedBids,
            RevealBids,
            AnotherStage,
            AreWeDoneYet,
            Finished
        }

        // This is the current stage.
        Stages public stage = Stages.AcceptingBlindedBids;

        uint public creationTime = now;

        modifier atStage(Stages _stage) {
            require(stage == _stage);
            _;
        }

        function nextStage() internal {
            stage = Stages(uint(stage) + 1);
        }

        // Perform timed transitions. Be sure to mention
        // this modifier first, otherwise the guards
        // will not take the new stage into account.
        modifier timedTransitions() {
            if (stage == Stages.AcceptingBlindedBids &&
                        now >= creationTime + 10 days)
                nextStage();
            if (stage == Stages.RevealBids &&
                    now >= creationTime + 12 days)
                nextStage();
            // The other stages transition by transaction
            _;
        }

        // Order of the modifiers matters here!
        function bid()
            public
            payable
            timedTransitions
            atStage(Stages.AcceptingBlindedBids)
        {
            // We will not implement that here
        }

        function reveal()
            public
            timedTransitions
            atStage(Stages.RevealBids)
        {
        }

        // This modifier goes to the next stage
        // after the function is done.
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
