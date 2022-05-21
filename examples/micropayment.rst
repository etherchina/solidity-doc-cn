********************
微支付通道合约
********************

译者注：本文其实和很多  `线下扩容 <https://wiki.learnblockchain.cn/ethereum/layer-2.html>`_ 的思路很类似，有兴趣可延伸阅读。

来看看如何在以太坊上实现一个支付通道。
通过使用密码签名技术可以在相同的参与者之间 **安全的、重复的、免手续费** 的转移以太币。学习这个示例子，我们需要先了解签名和验证签名以及如何建立支付通道。


创建及验证签名
=================================

想象一下 Alice 想发送一些以太币给 Bob, 即 Alice 发送者，而 Bob 是接收者。

Alice仅仅需要发送一条在链下密码学签名后的信息给Bob（比如通过消息），编写检查也是类似的。

Alice 和 Bob 用签名去授权交易，这可以通过以太坊智能合约来实现。Alice 将创建一个简单的智能合约来发送他的以太币，发送的函数不再是她在发起交易的时候执行，她将让 Bob 来执行并支付交易费。

合约工作有以下几步：

    1. Alice 部署 ``ReceiverPays`` 合约, 并附上足够的以太来负担支付通道的付款。
    2. Alice 通过自己的私钥签名来授权一个支付。
    3. Alice 发送签名信息给Bob，这个信息是不需要保密的（稍后解释），用什么发送也无关紧要。
    4. Bob 通过把签名信息提交给合约来索取这笔支付， 合约将验证信息的真实性并发送金额。


创建签名
----------------------

Alice 不需要和以太坊网络进行交互就可以完成签名，这个过程是完全离线的。
在这个指引里, 我们将通过使用 `web3.js <https://github.com/ethereum/web3.js>`_ and `MetaMask <https://metamask.io>`_ 在浏览器里完成签名, 方法在 `EIP-712 <https://github.com/ethereum/EIPs/pull/712>`_ 有描述。

.. code-block:: javascript

    /// 先计算一个hash
    var hash = web3.utils.sha3("message to sign");
    web3.eth.personal.sign(hash, web3.eth.defaultAccount, function () { console.log("Signed"); });

.. note::
   ``web3.eth.personal.sign`` 会关注待签名信息的长度， 因为我们先计算了hash，这个信息将总是32字节，因此长度前缀也总是相同。


哪些内容需要签名
----------------

为了合约能实现支付功能，签名消息必须包括：

    1. 收款人地址
    2. 发送金额
    3. 能够保护重放攻击的信息

所谓重放攻击是指一个被授权的支付消息被重复使用，为了避免重放攻击，我们引入一个 nonce (以太坊链上交易也是使用这个方式来防止重放攻击), 它表示一个账号已经发送交易的次数。智能合约将检查 nonce 是否使用过。

另外一种重放攻击可能发生的情形是这样的：所有者部署 ``ReceiverPays`` 合约之后，进行了一些支付，然后其销毁了合约， 随后又再次部署 ``ReceiverPays`` 合约， 这时新的合约无法知道先前部署合约的 nonce，所以攻击者可以再次利用先前的支付信息。
Alice 可以通过在签名信息中加入合约地址来阻止这个攻击。

下面的 ``claimPayment()`` 前两行，就是用来防止重放攻击。

打包参数
-----------------

我们已经知道哪些信息需要包含到签名消息里，我们需要把这些信息合并在一起，计算 hash 然后 签名。很简单，先拼接数据，然后 `ethereumjs-abi <https://github.com/ethereumjs/ethereumjs-abi>`_ 库提供了  ``soliditySHA3`` that mimics the behaviour of
函数类似于 Solidity 的 ``keccak256`` 函数应用在 ``abi.encodePacked`` 的输出结果上，下面是JavaScript 为  ``ReceiverPays`` 实现签名的代码：

.. code-block:: javascript

    // recipient 表示向谁付款.
    // amount, 单位 wei, 指定发送金额数量.
    // nonce 保护重放攻击
    // contractAddress 保护跨合约重放攻击
    function signPayment(recipient, amount, nonce, contractAddress, callback) {
        var hash = "0x" + abi.soliditySHA3(
            ["address", "uint256", "uint256", "address"],
            [recipient, amount, nonce, contractAddress]
        ).toString("hex");

        web3.eth.personal.sign(hash, web3.eth.defaultAccount, callback);
    }

在Solidity中还原消息签名者
-----------------------------------------

通常, ECDSA（椭圆曲线数字签名算法） 包含两个参数, ``r`` and ``s``. 在以太坊中签名包含第三个参数 ``v``, 它可以用于验证哪一个账号的私钥签署了这个消息。
Solidity 提供了一个内建函数 :ref:`ecrecover <mathematical-and-cryptographic-functions>` 它接受 ``r``, ``s`` and ``v`` 作为参数并且返回签名这的地址。

提取签名参数
-----------------------------------

使用 web3.js 签名的数据，``r``, ``s`` 和 ``v`` 是连接在一起的，第一步是把各部分分离出来。
我们可以在客户端这这个操作，但是在合约上实现就仅仅需要一个参数而不是三个参数， 分离一个大的直接数组到各个部分工作量比较大，所以我们在  ``splitSignature`` 函数（在本节的结尾可以看到这个函数）里使用 :doc:`内联汇编 <assembly>` 来完成这个工作。

计算信息的Hash
--------------------------

合约需要知道哪些参数被签名了，以便它可以从参数中重建信息用来验证签名。在函数 ``claimPayment`` 中的 ``prefixed`` 和 ``recoverSigner`` 就是用来做这个事情。

ReceiverPays 完整合约代码
----------------------------------

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    contract ReceiverPays {
        address owner = msg.sender;

        mapping(uint256 => bool) usedNonces;

        constructor() payable {}

        // 收款方认领付款
        function claimPayment(uint256 amount, uint256 nonce, bytes memory signature) external {
            require(!usedNonces[nonce]);
            usedNonces[nonce] = true;

            // 重建在客户端签名的信息
            bytes32 message = prefixed(keccak256(abi.encodePacked(msg.sender, amount, nonce, this)));

            require(recoverSigner(message, signature) == owner);

            payable(msg.sender).transfer(amount);
        }

        /// destroy the contract and reclaim the leftover funds.
        function kill() external {
            require(msg.sender == owner);
            selfdestruct(payable(msg.sender));
        }

        /// 第三方方法，分离签名信息的 v r s
        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // first 32 bytes, after the length prefix.
                r := mload(add(sig, 32))
                // second 32 bytes.
                s := mload(add(sig, 64))
                // final byte (first byte of the next 32 bytes).
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// 加入一个前缀，因为在eth_sign签名的时候会加上。
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


编写一个简单的支付通道
================================

Alice 现在可以创建一个简单但完整支付通道，支付通道通过加密签名可以重复安全的转移以太币，并且无需付费。

什么是支付通道？
--------------------------

支付通道允许在无需发生交易的情况下多次转移以太。这意味着可以避免与交易相关的延迟和费用。 我们将探讨两方（Alice和Bob）之间的简单单向支付通道。 它涉及三个步骤：

    1. Alice 附加一些以太创建智能合约，可以称为“打开”了支付通道
    2. Alice会签署一些消息指明给接收者付款金额。 每次付款都会重复此步骤。
    3. Bob“关闭”支付通道，取回以太币，并将剩余部分发送回发送者。

.. note::
  只有步骤1和3需要以太坊交易，步骤2意味着发送者通过离线方法（例如电子消息）将加密签名的消息发送给接收者。 这意味着只需要两个交易就可以支持任意数量（次数）的以太币转账。

Bob 保证会收到资金，因为智能合约托管以太并根据合法的签名消息来执行。 合约还可以强制超时执行，即使收款人拒绝关闭通道，Alice也能保证最终收回资金。 付款通道的参与者可以决定支付通道打开的持续时间。
对于短期交易，例如为网络访问的每一分钟支付一次网费，或者是长期的，例如向员工支付小时工资，支付可能持续数月或数年。

打开支付通道
---------------------------

要打开支付通道，Alice 需要部署智能合约，附加要托管的以太币并指定预期的收款人，以及通道存在有效时间。 合约的 ``SimplePaymentChannel`` 函数就是来做这个事情，代码在本节末尾。

进行支付
---------------

Alice 通过向 Bob 发送签名消息来付款。该步骤完全在以太坊网络之外执行。
消息由发送者以加密方式签名，然后直接传输给收款人。

每条消息都包含以下信息：

    * 智能合约的地址，用于防止交叉合约重放攻击。
    * 到目前为止所发送的以太总量。

在一系列转账结束时，付款通道仅需关闭一次。因此，只有一条消息被兑换。 这就是为什么每条消息都指定了以太的累计总量，而不是每次的微支付金额。 收款人自然而然的会选择兑换最新消息，因为这是以太总数最高的消息。
每条信息包含的nonce 将不再需要，因为智能合约仅执行一条信息。

包含合约地址用于防止一个支付通道的消息被用于不同的通道。


以下是修改后的JavaScript代码，用于对上一节中的消息进行加密签名：

.. code-block:: javascript

    function constructPaymentMessage(contractAddress, amount) {
        return abi.soliditySHA3(
            ["address", "uint256"],
            [contractAddress, amount]
        );
    }

    function signMessage(message, callback) {
        web3.eth.personal.sign(
            "0x" + message.toString("hex"),
            web3.eth.defaultAccount,
            callback
        );
    }

    // contractAddress is used to prevent cross-contract replay attacks.
    // amount, in wei, specifies how much Ether should be sent.

    function signPayment(contractAddress, amount, callback) {
        var message = constructPaymentMessage(contractAddress, amount);
        signMessage(message, callback);
    }


关闭状态通道
---------------------------

当Bob准备好收到他们的资金时，就可以通过调用智能合约上的 ``关闭`` 功能来关闭支付通道。
关闭通道会向接收方支付所欠的以太币并销毁合约，剩余的以太币返回Alice。为了关闭通道，Bob需要提供 Alice 签名过的消息。

智能合约必须验证信息是否包含发送者的有效签名。执行此验证的过程与上面收款人使用的方法相同。
Solidity函数 ``isValidSignature`` 和 ``recoverSigner`` 就是完成这个工作。

只有付款通道收款人可以调用 ``close`` 函数，其会选择最近的付款消息，因为该消息有最高的付款总额。
如果允许发送者调用此函数，他们可以提供较低金额的消息，来欺骗收款人。

函数会验证签名的消息是否与给定的参数匹配，如果匹配，收款人将收到应得的部分，余下的部分通过 ``selfdestruct`` 返还给发送者。
可以在完整的合约代码中看到 ``close`` 函数。


通道有效期
-------------------

Bob可以随时关闭支付通道，但如果他没有这样做，Alice 需要一种方法来收回他们托管的资金。
一个方法是在合约部署时设置 *到期时间* ，一旦达到那个时间，Alice 就可以调用 ``claimTimeout`` 收回他们的资金。 可以在完整的合约代码中查看 ``claimTimeout`` 函数。

调用此功能后，Bob无法再接收任何以太币，因此，Bob必须在到期前关闭频道。


完整合约代码
-----------------

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    contract SimplePaymentChannel {
        address payable public sender;      // The account sending payments.
        address payable public recipient;   // The account receiving the payments.
        uint256 public expiration;  // Timeout in case the recipient never closes.

        constructor (address payable recipientAddress, uint256 duration)
            public
            payable
        {
            sender = payable(msg.sender);
            recipient = recipientAddress;
            expiration = block.timestamp + duration;
        }

        function isValidSignature(uint256 amount, bytes memory signature)
            internal
            view
            returns (bool)
        {
            bytes32 message = prefixed(keccak256(abi.encodePacked(this, amount)));

            // check that the signature is from the payment sender
            return recoverSigner(message, signature) == sender;
        }

        /// the recipient can close the channel at any time by presenting a
        /// signed amount from the sender. the recipient will be sent that amount,
        /// and the remainder will go back to the sender
        function close(uint256 amount, bytes memory signature) external {
            require(msg.sender == recipient);
            require(isValidSignature(amount, signature));

            recipient.transfer(amount);
            selfdestruct(sender);
        }

        /// the sender can extend the expiration at any time
        function extend(uint256 newExpiration) external {
            require(msg.sender == sender);
            require(newExpiration > expiration);

            expiration = newExpiration;
        }

        /// 如果过期过期时间已到，而收款人没有关闭通道，可执行此函数，销毁合约并返还余额
        function claimTimeout() external {
            require(block.timestamp >= expiration);
            selfdestruct(sender);
        }

        /// All functions below this are just taken from the chapter
        /// 'creating and verifying signatures' chapter.

        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // first 32 bytes, after the length prefix
                r := mload(add(sig, 32))
                // second 32 bytes
                s := mload(add(sig, 64))
                // final byte (first byte of the next 32 bytes)
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// builds a prefixed hash to mimic the behavior of eth_sign.
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


.. note::
  函数 ``splitSignature`` 没有做足够的安全检查，完整的产品里应该使用严格测试的库，如： `openzepplin 的实现版本  <https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol>`_ 。


验证支付
------------------

与上一节不同，付款通道中的消息不是马上赎回。 收款人会跟踪最新消息及在关闭付款通道时兑换它。 这意味着接收者对每条消息进行验证就至关重要。
否则，无法保证收款人能够最终获得付款。

收款人使用以下过程验证每条消息：

    1. 验证信息中的合约地址是否与付款通道匹配。
    2. 验证新金额是否为预期金额。
    3. 确认新金额不超过托管的以太币总额。
    4. 验证签名是否有效并来自通道的付款方。


我们使用 `ethereumjs-util <https://github.com/ethereumjs/ethereumjs-util>`_
库来编写验证过程，这里使用 JavaScript ，当然实现的方式有很多。下面的代码借鉴了 上面的 ``constructMessage`` 函数:

.. code-block:: javascript

    // this mimics the prefixing behavior of the eth_sign JSON-RPC method.
    function prefixed(hash) {
        return ethereumjs.ABI.soliditySHA3(
            ["string", "bytes32"],
            ["\x19Ethereum Signed Message:\n32", hash]
        );
    }

    function recoverSigner(message, signature) {
        var split = ethereumjs.Util.fromRpcSig(signature);
        var publicKey = ethereumjs.Util.ecrecover(message, split.v, split.r, split.s);
        var signer = ethereumjs.Util.pubToAddress(publicKey).toString("hex");
        return signer;
    }

    function isValidSignature(contractAddress, amount, signature, expectedSigner) {
        var message = prefixed(constructPaymentMessage(contractAddress, amount));
        var signer = recoverSigner(message, signature);
        return signer.toLowerCase() ==
            ethereumjs.Util.stripHexPrefix(expectedSigner).toLowerCase();
    }