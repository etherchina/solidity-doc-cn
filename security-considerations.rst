.. _security_considerations:

#######################
安全考量
#######################

While it is usually quite easy to build software that works as expected,
it is much harder to check that nobody can use it in a way that was **not** anticipated.

尽管在通常情况下编写一个按照预期运行的软件很简单，
但想要确保没有人会使用出乎意料的方式运行它就要难多了。

In Solidity, this is even more important because you can use smart contracts
to handle tokens or, possibly, even more valuable things. Furthermore, every
execution of a smart contract happens in public and, in addition to that,
the source code is often available.

在Solidity中，这一点尤为重要，因为智能合约可以用来处理令牌，甚至有可能是更有价值的东西。
除此之外，智能合约的每一次执行都是公开的，而且源代码也通常是容易获得的。

Of course you always have to consider how much is at stake:
You can compare a smart contract with a web service that is open to the
public (and thus, also to malicious actors) and perhaps even open source.
If you only store your grocery list on that web service, you might not have
to take too much care, but if you manage your bank account using that web service,
you should be more careful.

当然，你总是需要考虑有多大的风险：
你可以将智能合约与公开的（当然也对恶意用户开放），甚至是开源的网络服务相比较。
如果你只是在某个网络服务上存储你的购物清单，则可能不必太在意，
但如果你使用那个网络服务管理您的银行帐户，
那就需要当心了。

This section will list some pitfalls and general security recommendations but
can, of course, never be complete. Also, keep in mind that even if your
smart contract code is bug-free, the compiler or the platform itself might
have a bug. A list of some publicly known security-relevant bugs of the compiler
can be found in the
:ref:`list of known bugs<known_bugs>`, which is also machine-readable. Note
that there is a bug bounty program that covers the code generator of the
Solidity compiler.

本节将列出一些陷阱和一般性的安全建议，但这绝对不全面。
另外，请时刻注意的是即使你的智能合约代码没有bug，
但编译器或者平台本身可能存在bug。
一个已知的有关编译器的安全相关的bug可以在 :ref:`list of known bugs<known_bugs>`
找到，这个列表也是机器可读的。
请注意其中有一个涵盖了Solidity编译器的代码生成器的bug悬赏项目。

As always, with open source documentation, please help us extend this section
(especially, some examples would not hurt)!
有了开源的文档，请一如既往地帮助我们扩展这一节的内容（尤其是一些例子并不会造成伤害）！

********
陷阱
********

Private Information and Randomness
隐私信息和随机性
==================================

Everything you use in a smart contract is publicly visible, even
local variables and state variables marked ``private``.

在智能合约中你所用的一切都是公开可见的，即便是将本地变量和状态变量标记成 ``private``。

Using random numbers in smart contracts is quite tricky if you do not want
miners to be able to cheat.

如果你不想让矿工作弊的话，在智能合约中使用随机数是一种很狡猾的手段。

Re-Entranc
重入
===========

Any interaction from a contract (A) with another contract (B) and any transfer
of Ether hands over control to that contract (B). This makes it possible for B
to call back into A before this interaction is completed. To give an example,
the following code contains a bug (it is just a snippet and not a
complete contract):

一个合约A与另一个合约B的任何交互以及任何以太币交易的控制权都归属于合约B。
这使得合约B能够在交易结束前回调A中的代码。
举个例子，下面的代码中有一个bug（这只是一个代码段，不是完整的合约）：

::

    pragma solidity ^0.4.0;

    // THIS CONTRACT CONTAINS A BUG - DO NOT USE
    contract Fund {
        /// Mapping of ether shares of the contract.
        mapping(address => uint) shares;
        /// Withdraw your share.
        function withdraw() public {
            if (msg.sender.send(shares[msg.sender]))
                shares[msg.sender] = 0;
        }
    }

The problem is not too serious here because of the limited gas as part
of ``send``, but it still exposes a weakness: Ether transfer can always
include code execution, so the recipient could be a contract that calls
back into ``withdraw``. This would let it get multiple refunds and
basically retrieve all the Ether in the contract. In particular, the
following contract will allow an attacker to refund multiple times
as it uses ``call`` which forwards all remaining gas by default:

这里的问题不是很严重，因为有限的gas也作为 ``send`` 的一部分，但仍然暴露了一个缺陷：
以太币的传输过程中总是可以包含代码执行，所以接收者可以是一个回调进入 ``withdraw`` 的合约。
这就会使其多次得到退款，从而将合约中的全部以太币取走。
特别地，下面的合约将允许一个攻击者多次得到退款，因为它使用了 ``call`` ，默认发送所有剩余的gas。

::

    pragma solidity ^0.4.0;

    // THIS CONTRACT CONTAINS A BUG - DO NOT USE
    contract Fund {
        /// Mapping of ether shares of the contract.
        mapping(address => uint) shares;
        /// Withdraw your share.
        function withdraw() public {
            if (msg.sender.call.value(shares[msg.sender])())
                shares[msg.sender] = 0;
        }
    }

To avoid re-entrancy, you can use the Checks-Effects-Interactions pattern as
outlined further below:

为了避免重入，你可以使用下面的 Checks-Effects-Interactions 模式：

::

    pragma solidity ^0.4.11;

    contract Fund {
        /// Mapping of ether shares of the contract.
        mapping(address => uint) shares;
        /// Withdraw your share.
        function withdraw() public {
            var share = shares[msg.sender];
            shares[msg.sender] = 0;
            msg.sender.transfer(share);
        }
    }

Note that re-entrancy is not only an effect of Ether transfer but of any
function call on another contract. Furthermore, you also have to take
multi-contract situations into account. A called contract could modify the
state of another contract you depend on.

请注意重入不仅是以太币传输的其中一个影响，还包括任何对另一个合约的函数调用。
更进一步说，你也不得不考虑多合约的情况。
一个被调用的合约可以修改你所依赖的另一个合约的状态。

Gas Limit and Loops Gas限制和循环
===================

Loops that do not have a fixed number of iterations, for example, loops that depend on storage values, have to be used carefully:
Due to the block gas limit, transactions can only consume a certain amount of gas. Either explicitly or just due to
normal operation, the number of iterations in a loop can grow beyond the block gas limit which can cause the complete
contract to be stalled at a certain point. This may not apply to ``constant`` functions that are only executed
to read data from the blockchain. Still, such functions may be called by other contracts as part of on-chain operations
and stall those. Please be explicit about such cases in the documentation of your contracts.


必须谨慎使用没有固定迭代次数的循环，例如依赖于存储值的循环：
由于区块gas有限，交易只能消耗一定数量的gas。
无论是明确指出的还是正常运行过程中，循环中迭代的次数都有可能超出区块gas的限制，从而导致合约在某个时刻骤然停止。
这可能不适用于只被用来从区块链中读取数据的 ``常量`` 函数。
尽管如此，这些函数仍然可能会被其它合约当作链上操作的一部分被调用并将其拖延。
请在合同文件中明确说明这些情况。

Sending and Receiving Ether 发送和接收以太币
===========================

- Neither contracts nor "external accounts" are currently able to prevent that someone sends them Ether.
  Contracts can react on and reject a regular transfer, but there are ways
  to move Ether without creating a message call. One way is to simply "mine to"
  the contract address and the second way is using ``selfdestruct(x)``.

- 目前无论是合约还是“外部账户”都不能阻止有人给他们发送以太币。
  合约可以做出翻译并且拒绝一个常规的交易，但还有些方法可以不通过创建消息来发送以太币。
  其中一种方法就是单纯地向合约地址“挖矿”，另一种方法就是使用 `selfdestruct(x)`。

- If a contract receives Ether (without a function being called), the fallback function is executed.
  If it does not have a fallback function, the Ether will be rejected (by throwing an exception).
  During the execution of the fallback function, the contract can only rely
  on the "gas stipend" (2300 gas) being available to it at that time. This stipend is not enough to access storage in any way.
  To be sure that your contract can receive Ether in that way, check the gas requirements of the fallback function
  (for example in the "details" section in Remix).

- 如果一个合约收到了以太币（且没有调用函数），就会执行回退函数。
  如果没有回退函数，那么以太币会被拒收（同时会抛出异常）。
  在回退函数执行过程中，合约只能依靠此时可用的“gas津贴”（2300gas）来执行。
  这笔津贴并不足以任何方式访问存储。
  为了确保你的合约可以通过这种方式收到以太币，请你核对回退函数所需的gas数量
  （在Remix的“详细”章节会举例说明）。

- There is a way to forward more gas to the receiving contract using
  ``addr.call.value(x)()``. This is essentially the same as ``addr.transfer(x)``,
  only that it forwards all remaining gas and opens up the ability for the
  recipient to perform more expensive actions (and it only returns a failure code
  and does not automatically propagate the error). This might include calling back
  into the sending contract or other state changes you might not have thought of.
  So it allows for great flexibility for honest users but also for malicious actors.

- 有一种方法可以通过使用 ``addr.call.value(x)()`` 向接收合约发送更多的gas。
  这在根本上跟 ``addr.transfer(x)`` 是一样的，
  只不过前者发送所有剩余的gas，并且使得接收者有能力执行更加昂贵的步骤
  （它只会放回一个错误代码，而且也不会自动传播这个错误）。
  这可能包括回调发送合约或者你能想到的其它状态改变的情况。
  因此这种方法无论是给良好的用户还是恶意的行为都提供了极大的灵活性。

- If you want to send Ether using ``address.transfer``, there are certain details to be aware of:

- 如果你想要使用 ``address.transfer`` 发送以太币，你需要注意以下几个细节：

  1. If the recipient is a contract, it causes its fallback function to be executed which can, in turn, call back the sending contract.
  2. Sending Ether can fail due to the call depth going above 1024. Since the caller is in total control of the call
     depth, they can force the transfer to fail; take this possibility into account or use ``send`` and make sure to always check its return value. Better yet,
     write your contract using a pattern where the recipient can withdraw Ether instead.
  3. Sending Ether can also fail because the execution of the recipient contract
     requires more than the allotted amount of gas (explicitly by using ``require``,
     ``assert``, ``revert``, ``throw`` or
     because the operation is just too expensive) - it "runs out of gas" (OOG).
     If you use ``transfer`` or ``send`` with a return value check, this might provide a
     means for the recipient to block progress in the sending contract. Again, the best practice here is to use
     a :ref:`"withdraw" pattern instead of a "send" pattern <withdrawal_pattern>`.

  1. 如果接收者是一个合约，它会执行自己的回退函数，从而可以回调发送以太币的合约。
  2. 如果调用的深度超过1024，发送以太币也会失败。因此调用者对调用深度有完全的控制权，他们可以强制使这次发送失效；
     请考虑这种可能性，或者使用 ``send`` 并且确保每次都核对它的返回值。
     更好的方法是使用一种接收者可以取回以太币的方式编写你的合约。
  3. 发送以太币也可能因为接收方合约的执行所需的gas多余给分配的gas而失败
     （可以显式地使用 ``require`` ， ``assert``， ``revert`` ， ``throw`` 或者因为这个操作过于昂贵） - “gas不够用了”。
     如果你使用 ``transfer`` 或者 ``send`` 的同时带有返回值检查，这就为接收者提供了在发送合约中阻断进程的方法。
     再次说明，最佳做法是使用 :ref:`"withdraw" pattern instead of a "send" pattern <withdrawal_pattern>`。

Callstack Depth
===============

External function calls can fail any time because they exceed the maximum
call stack of 1024. In such situations, Solidity throws an exception.
Malicious actors might be able to force the call stack to a high value
before they interact with your contract.

Note that ``.send()`` does **not** throw an exception if the call stack is
depleted but rather returns ``false`` in that case. The low-level functions
``.call()``, ``.callcode()`` and ``.delegatecall()`` behave in the same way.

tx.origin
=========

Never use tx.origin for authorization. Let's say you have a wallet contract like this:

::

    pragma solidity ^0.4.11;

    // THIS CONTRACT CONTAINS A BUG - DO NOT USE
    contract TxUserWallet {
        address owner;

        function TxUserWallet() public {
            owner = msg.sender;
        }

        function transferTo(address dest, uint amount) public {
            require(tx.origin == owner);
            dest.transfer(amount);
        }
    }

Now someone tricks you into sending ether to the address of this attack wallet:

::

    pragma solidity ^0.4.11;

    interface TxUserWallet {
        function transferTo(address dest, uint amount) public;
    }

    contract TxAttackWallet {
        address owner;

        function TxAttackWallet() public {
            owner = msg.sender;
        }

        function() public {
            TxUserWallet(msg.sender).transferTo(owner, msg.sender.balance);
        }
    }

If your wallet had checked ``msg.sender`` for authorization, it would get the address of the attack wallet, instead of the owner address. But by checking ``tx.origin``, it gets the original address that kicked off the transaction, which is still the owner address. The attack wallet instantly drains all your funds.


Minor Details
=============

- In ``for (var i = 0; i < arrayName.length; i++) { ... }``, the type of ``i`` will be ``uint8``, because this is the smallest type that is required to hold the value ``0``. If the array has more than 255 elements, the loop will not terminate.
- The ``constant`` keyword for functions is currently not enforced by the compiler.
  Furthermore, it is not enforced by the EVM, so a contract function that "claims"
  to be constant might still cause changes to the state.
- Types that do not occupy the full 32 bytes might contain "dirty higher order bits".
  This is especially important if you access ``msg.data`` - it poses a malleability risk:
  You can craft transactions that call a function ``f(uint8 x)`` with a raw byte argument
  of ``0xff000001`` and with ``0x00000001``. Both are fed to the contract and both will
  look like the number ``1`` as far as ``x`` is concerned, but ``msg.data`` will
  be different, so if you use ``keccak256(msg.data)`` for anything, you will get different results.

***************
推荐做法
***************

Restrict the Amount of Ether
============================

Restrict the amount of Ether (or other tokens) that can be stored in a smart
contract. If your source code, the compiler or the platform has a bug, these
funds may be lost. If you want to limit your loss, limit the amount of Ether.

Keep it Small and Modular
=========================

Keep your contracts small and easily understandable. Single out unrelated
functionality in other contracts or into libraries. General recommendations
about source code quality of course apply: Limit the amount of local variables,
the length of functions and so on. Document your functions so that others
can see what your intention was and whether it is different than what the code does.

Use the Checks-Effects-Interactions Pattern
===========================================

Most functions will first perform some checks (who called the function,
are the arguments in range, did they send enough Ether, does the person
have tokens, etc.). These checks should be done first.

As the second step, if all checks passed, effects to the state variables
of the current contract should be made. Interaction with other contracts
should be the very last step in any function.

Early contracts delayed some effects and waited for external function
calls to return in a non-error state. This is often a serious mistake
because of the re-entrancy problem explained above.

Note that, also, calls to known contracts might in turn cause calls to
unknown contracts, so it is probably better to just always apply this pattern.

Include a Fail-Safe Mode
========================

While making your system fully decentralised will remove any intermediary,
it might be a good idea, especially for new code, to include some kind
of fail-safe mechanism:

You can add a function in your smart contract that performs some
self-checks like "Has any Ether leaked?",
"Is the sum of the tokens equal to the balance of the contract?" or similar things.
Keep in mind that you cannot use too much gas for that, so help through off-chain
computations might be needed there.

If the self-check fails, the contract automatically switches into some kind
of "failsafe" mode, which, for example, disables most of the features, hands over
control to a fixed and trusted third party or just converts the contract into
a simple "give me back my money" contract.


*******************
形式化验证
*******************

Using formal verification, it is possible to perform an automated mathematical
proof that your source code fulfills a certain formal specification.
The specification is still formal (just as the source code), but usually much
simpler.

Note that formal verification itself can only help you understand the
difference between what you did (the specification) and how you did it
(the actual implementation). You still need to check whether the specification
is what you wanted and that you did not miss any unintended effects of it.
