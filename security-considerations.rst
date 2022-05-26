.. include :: glossaries.rst
.. _security_considerations:

#######################
安全考量
#######################

尽管在通常情况下编写一个按照预期运行的软件很简单，
但想要确保没有人能够以出乎意料的方式使用它就困难多了。

在 Solidity 中，这一点尤为重要，因为智能合约可以用来处理通证，甚至有可能是更有价值的东西。
除此之外，智能合约的每一次执行都是公开的，而且源代码也通常是容易获得的。

当然，你总是需要考虑有多大的风险：
你可以将智能合约与公开的（当然也对恶意用户开放）、甚至是开源的网络服务相比较。
如果你只是在某个网络服务上存储你的购物清单，则可能不必太在意，
但如果你使用那个网络服务管理你的银行帐户，
那就需要特别当心了。

本节将列出一些陷阱和一般性的安全建议，但这绝对不全面。
另外，请时刻注意的是即使你的智能合约代码没有 bug，
但编译器或者平台本身可能存在 bug。
一个已知的编译器安全相关的 bug 列表可以在 :ref:`已知bug列表<known_bugs>` 找到，
这个列表也可以用程序读取。
请注意其中有一个涵盖了 Solidity 编译器的代码生成器的 bug 悬赏项目。

我们的文档是开源的，请一如既往地帮助我们扩展这一节的内容（何况其中一些例子并不会造成损失）！

********
陷阱
********

私有信息和随机性
==================================

在智能合约中你所用的一切都是公开可见的，即便是局部变量和被标记成 ``private`` 的状态变量也是如此。

如果不想让矿工作弊的话，在智能合约中使用随机数会很棘手
（译者注：在智能合约中使用随机数很难保证节点不作弊，
这是因为智能合约中的随机数一般要依赖计算节点的本地时间得到，
而本地时间是可以被恶意节点伪造的，因此这种方法并不安全。
通行的做法是采用 |off_chain| 的第三方服务，比如 Oraclize 来获取随机数）。


.. _re_entance:

重入
===========

任何从合约 A 到合约 B 的交互以及任何从合约 A 到合约 B 的 |ether| 的转移，都会将控制权交给合约 B。
这使得合约 B 能够在交互结束前回调 A 中的代码。
举个例子，下面的代码中有一个 bug（这只是一个代码段，不是完整的合约）：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    // 不要使用这个合约，其中包含一个 bug。
    contract Fund {
        /// 合约中 |ether| 分成的映射。
        mapping(address => uint) shares;
        /// 提取你的分成。
        function withdraw() public {
            if (payable(msg.sender).send(shares[msg.sender]))
                shares[msg.sender] = 0;
        }
    }

这里的问题不是很严重，因为有限的 gas 也作为 ``send`` 的一部分，但仍然暴露了一个缺陷：
|ether| 的传输过程中总是可以包含代码执行，所以接收者可以是一个回调进入 ``withdraw`` 的合约。
这就会使其多次得到退款，从而将合约中的全部 |ether| 提取。
特别地，下面的合约将允许一个攻击者多次得到退款，因为它使用了 ``call`` ，默认发送所有剩余的 gas。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.2 <0.9.0;

    // 不要使用这个合约，其中包含一个 bug。
    contract Fund {
        /// 合约中 |ether| 分成的映射。
        mapping(address => uint) shares;
        /// 提取你的分成。
        function withdraw() public {
            if (payable(msg.sender).call.value(shares[msg.sender])())
                shares[msg.sender] = 0;
        }
    }

为了避免重入，你可以使用下面撰写的“检查-生效-交互”（Checks-Effects-Interactions）模式：

.. code-block:: solidity

    pragma solidity  >=0.6.0 <0.9.0;

    contract Fund {
        /// 合约中 |ether| 分成的映射。
        mapping(address => uint) shares;
        /// 提取你的分成。
        function withdraw() public {
            var share = shares[msg.sender];
            shares[msg.sender] = 0;
            payable(msg.sender).transfer(share);
        }
    }

请注意重入不仅是 |ether| 传输的其中一个影响，还包括任何对另一个合约的函数调用。
更进一步说，你也不得不考虑多合约的情况。
一个被调用的合约可以修改你所依赖的另一个合约的状态。

gas 限制和循环
===================

必须谨慎使用没有固定迭代次数的循环，例如依赖于 |storage| 值的循环：
由于区块 gas 有限，交易只能消耗一定数量的 gas。
无论是明确指出的还是正常运行过程中的，循环中的数次迭代操作所消耗的 gas 都有可能超出区块的 gas 限制，从而导致整个合约在某个时刻骤然停止。
这可能不适用于只被用来从区块链中读取数据的 ``view`` 函数。
尽管如此，这些函数仍然可能会被其它合约当作 |on_chain| 操作的一部分来调用，并使那些操作骤然停止。
请在合约代码的说明文档中明确说明这些情况。

发送和接收 |ether|
===========================

- 目前无论是合约还是“外部账户”都不能阻止有人给它们发送 |ether|。
  合约可以对一个正常的转账做出反应并拒绝它，但还有些方法可以不通过创建消息来发送 |ether|。
  其中一种方法就是单纯地向合约地址“挖矿”，另一种方法就是使用 ``selfdestruct(x)`` 。

- 如果一个合约收到了 |ether| （且没有函数被调用），就会执行 fallback 函数。
  如果没有 fallback 函数，那么 |ether| 会被拒收（同时会抛出异常）。
  在 fallback 函数执行过程中，合约只能依靠此时可用的“gas 津贴”（2300 gas）来执行。
  这笔津贴并不足以用来完成任何方式的 |storage| 访问。
  为了确保你的合约可以通过这种方式收到 |ether|，请你核对 fallback 函数所需的 gas 数量
  （在 Remix 的“详细”章节会举例说明）。

- 有一种方法可以通过使用 ``addr.call.value(x)()`` 向接收合约发送更多的 gas。
  这本质上跟 ``addr.transfer(x)`` 是一样的，
  只不过前者发送所有剩余的 gas，并且使得接收者有能力执行更加昂贵的操作
  （它只会返回一个错误代码，而且也不会自动传播这个错误）。
  这可能包括回调发送合约或者你想不到的其它状态改变的情况。
  因此这种方法无论是给诚实用户还是恶意行为者都提供了极大的灵活性。

- 如果你想要使用 ``address.transfer`` 发送 |ether| ，你需要注意以下几个细节：

  1. 如果接收者是一个合约，它会执行自己的 fallback 函数，从而可以回调发送 |ether| 的合约。
  2. 如果调用的深度超过 1024，发送 |ether| 也会失败。由于调用者对调用深度有完全的控制权，他们可以强制使这次发送失败；
     请考虑这种可能性，或者使用 ``send`` 并且确保每次都核对它的返回值。
     更好的方法是使用一种接收者可以取回 |ether| 的方式编写你的合约。
  3. 发送 |ether| 也可能因为接收方合约的执行所需的 gas 多于分配的 gas 数量而失败
     （确切地说，是使用了 ``require`` ， ``assert`` ， ``revert`` ， ``throw`` 或者因为这个操作过于昂贵） - “gas 不够用了”。
     如果你使用 ``transfer`` 或者 ``send`` 的同时带有返回值检查，这就为接收者提供了在发送合约中阻断进程的方法。
     再次说明，最佳实践是使用 :ref:`“取回”模式而不是“发送”模式<withdrawal_pattern>`。

调用栈深度
===============

如果调用栈超过了1024上限，外部函数调用随时会失败。在这种情况下，Solidity 会抛出一个异常。

恶意行为者也许能够在与你的合约交互之前就强制将调用栈设置在一个比较高的值。

注意，由于 `Tangerine Whistle <https://eips.ethereum.org/EIPS/eip-608>`_ 硬分叉， `EIP150 规则 <https://eips.ethereum.org/EIPS/eip-150>`_ 使得调用栈深度攻击无法实施。还要注意的是，调用栈和表达式栈是不相关的，尽管两者都有1024个栈槽的大小限制。


请注意，使用 ``.send()`` 时如果超出调用栈 **并不会** 抛出异常，而是会返回 ``false``。
低级的函数比如 ``.call()`` ， ``.callcode()`` 和 ``.delegatecall()`` 也都是这样的。


授权代理 Proxies
==================

如果你的合约可以作为一个代理，也就是说，如果它可以用用户提供的数据调用任意的合约，那么用户基本上可以承担代理合约的身份。即使你有其他的保护措施，最好是建立你的合约系统，使代理没有任何权限（甚至对自己也没有）。如果需要，你可以使用第二个代理来完成。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.0;
    contract ProxyWithMoreFunctionality {
        PermissionlessProxy proxy;

        function callOther(address addr, bytes memory payload) public
                returns (bool, bytes memory) {
            return proxy.callOther(addr, payload);
        }
        // Other functions and other functionality
    }

    // 这是完整的合约，它没有其他功能，不需要任何权限就可以工作。
    contract PermissionlessProxy {
        function callOther(address addr, bytes memory payload) public
                returns (bool, bytes memory) {
            return addr.call(payload);
        }
    }


tx.origin问题
=============

永远不要使用 tx.origin 做身份认证。假设你有一个如下的钱包合约：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    // 不要使用这个合约，其中包含一个 bug。
    contract TxUserWallet {
        address owner;

        constructor() {
            owner = msg.sender;
        }

        function transferTo(address dest, uint amount) public {
            // BUG 在这里，你应该使用 msg.sender 取代 tx.origin
            require(tx.origin == owner);
            dest.transfer(amount);
        }
    }

现在有人欺骗你，将 |ether| 发送到了这个恶意钱包的地址：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    interface TxUserWallet {
        function transferTo(address dest, uint amount) public;
    }

    contract TxAttackWallet {
        address payable owner;

        constructor() {
            owner = payable(msg.sender);
        }

        function() public {
            TxUserWallet(msg.sender).transferTo(owner, msg.sender.balance);
        }
    }

如果你的钱包通过核查 ``msg.sender`` 来验证发送方身份，你就会得到恶意钱包的地址，而不是所有者的地址。
但是通过核查 ``tx.origin`` ，得到的就会是启动交易的原始地址，它仍然会是所有者的地址。
恶意钱包会立即将你的资金抽出。

.. _underflow-overflow:

整型溢出问题
=========================================

与许多编程语言一样，Solidity的整数类型上并不是真实整数。
当值很小时，它们类似于整数，但不能表示任意大的数字。

以下代码导致溢出，因为 ``uint8`` 中加法的的结果太大:

.. code-block:: solidity

  uint8 x = 255;
  uint8 y = 1;
  return x + y;

Solidity 有两种模式来处理这些溢出。检查和不检查或 "wrapping" 模式。

默认的检查模式将检测到溢出并触发失败断言。你可以使用 ``unchecked { ... }`` 停用检查，使溢出被无声地忽略。
如果用 ``unchecked { ... }`` ，上面的代码将返回 ``0`` 

即使在检查模式下，也不要认为你受到了保护，不会出现溢出错误。
在这种模式下，溢出总是会被还原。如果不可能避免溢出，这可能会导致智能合约被卡在某个状态下。

通常，你的了解补码的限制，它甚至对有符号的数字有一些更特殊的边缘情况。

试着用 ``require`` 去限制输入值的大小在一个合理的范围，并且使用 :ref:`SMT checker<smt_checker>` 查找潜在的溢出错误。

.. _clearing-mappings:

清理映射
=================

Solidity 类型 ``mapping`` （见 :ref:`mapping-types` ）是一个仅在存储空间有效的键值数据结构，它不会跟踪被赋值的键。 正因为如此，如果没有键的额外信息是没法清理映射的。

如果一个 ``mapping`` 被用作动态存储数组的基本类型，删除或弹出数组时对 ``mapping`` 元素是没有影响的。 
还有类似的情况，例如，如果 ``mapping`` 被用作一个 ``struct`` 的成员字段的类型，而该结构作为是动态存储数组的基本类型。
在包含了 ``mapping`` 的结构体或数组的赋值时， ``mapping`` 也会被忽略。


.. code-block:: solidity

    pragma solidity >=0.6.0 <0.9.0;

    contract Map {
        mapping (uint => uint)[] array;

        function allocate(uint newMaps) public {
            for (uint i = 0; i < newMaps; i++)
                array.push();
        }

        function writeMap(uint map, uint key, uint value) public {
            array[map][key] = value;
        }

        function readMap(uint map, uint key) public view returns (uint) {
            return array[map][key];
        }

        function eraseMaps() public {
            delete array;
        }
    }


考虑一下上面的例子和下面的调用序列： ``allocate(10)`` , ``writeMap(4, 128, 256)``， 此时，调用 ``readMap(4, 128)`` 返回256。

如果我们调用 ``eraseMaps``，状态变量 ``array`` 的长度被清零，但是因为它的 ``mapping`` 元素不能被清零，映射的信息在合约的存储中保持不变。

删除 ``array`` 后，调用 ``allocate(5)`` 允许我们访问 ``array[4]``，调用 ``readMap(4, 128)`` 返回256， 甚至不需要再调用 ``writeMap``。

如果你的  ``mapping`` 信息需要被删除，可以考虑使用一个类似于 `iterable mapping <https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol>`_, 允许你遍历键，并在适当的 ``mapping`` 中删除它们的值。



细枝末节
=============

- 在 ``for (var i = 0; i < arrayName.length; i++) { ... }`` 中， ``i`` 的类型会变为 ``uint8`` ，
  因为这是保存 ``0`` 值所需的最小类型。如果数组超过 255 个元素，则循环不会终止。
- 不占用完整 32 字节的类型可能包含“脏高位”。这在当你访问 ``msg.data`` 的时候尤为重要 —— 它带来了延展性风险：
  你既可以用原始字节 ``0xff000001`` 也可以用 ``0x00000001`` 作为参数来调用函数 ``f(uint8 x)`` 以构造交易。
  这两个参数都会被正常提供给合约，并且 ``x`` 的值看起来都像是数字 ``1``，
  但 ``msg.data`` 会不一样，所以如果你无论怎么使用 ``keccak256(msg.data)``，你都会得到不同的结果。

***************
推荐做法
***************

认真对待警告
=======================

如果编译器警告了你什么事，你应该修改它，即使你不认为这个特定的警告不会产生安全隐患，因为那也有可能埋藏着其他的问题。
我们给出的任何编译器警告，都可以通过轻微的修改来去掉。

始终使用最新版本的编译器，以了解所有最近引入的警告。

编译器发出的 ``info`` 类型的信息并不危险，它只是代表额外的建议和可选信息，编译器认为这些信息可能对用户有用。


限定 |ether| 的数量
============================

限定 |storage| 在一个智能合约中 |ether| （或者其它通证）的数量。
如果你的源代码、编译器或者平台出现了 bug，可能会导致这些资产丢失。
如果你想控制你的损失，就要限定 |ether| 的数量。

保持合约简练且模块化
=========================

保持你的合约短小精炼且易于理解。
找出无关于其它合约或库的功能。
有关源码质量可以采用的一般建议：
限制局部变量的数量以及函数的长度等等。
将实现的函数文档化，这样别人看到代码的时候就可以理解你的意图，并判断代码是否按照正确的意图实现。

使用“检查-生效-交互”（Checks-Effects-Interactions）模式
============================================================

大多数函数会首先做一些检查工作（例如谁调用了函数，参数是否在取值范围之内，它们是否发送了足够的 |ether| ，用户是否具有通证等等）。
这些检查工作应该首先被完成。

第二步，如果所有检查都通过了，应该接着进行会影响当前合约状态变量的那些处理。
与其它合约的交互应该是任何函数的最后一步。

早期合约延迟了一些效果的产生，为了等待外部函数调用以非错误状态返回。
由于上文所述的重入问题，这通常会导致严重的后果。

请注意，对已知合约的调用反过来也可能导致对未知合约的调用，所以最好是一直保持使用这个模式编写代码。

包含故障-安全（Fail-Safe）模式
====================================

尽管将系统完全去中心化可以省去许多中间环节，但包含某种故障-安全模式仍然是好的做法，尤其是对于新的代码来说：

你可以在你的智能合约中增加一个函数实现某种程度上的自检查，比如“ |ether| 是否会泄露？”，
“通证的总和是否与合约的余额相等？”等等。
请记住，你不能使用太多的 gas，所以可能需要通过 |off_chain| 计算来辅助。

如果自检查没有通过，合约就会自动切换到某种“故障安全”模式，
例如，关闭大部分功能，将控制权交给某个固定的可信第三方，或者将合约转换成一个简单的“退回我的钱”合约。

请求同行 Review
===================

越多的人检查代码，发现的问题就越多。
请别人检查你的代码也有助于交叉检查你的代码
这很容易理解 —— 这也是优秀智能合约的一个非常重要的标准。

