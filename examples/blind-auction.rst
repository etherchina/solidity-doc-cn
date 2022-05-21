.. include:: glossaries.rst

.. index:: auction;blind, auction;open, blind auction, open auction

***********************
秘密竞价（盲拍）合约
***********************

在本节中，我们将展示如何轻松地在以太坊上创建一个秘密竞价的合约。
我们将从公开拍卖开始，每个人都可以看到出价，然后将此合约扩展到盲拍合约，
在竞标期结束之前无法看到实际出价。

.. _simple_auction:

简单的公开拍卖
===================

以下简单的拍卖合约的总体思路是每个人都可以在投标期内发送他们的出价。
出价已经包含了资金/以太币，来将投标人与他们的投标绑定。
如果最高出价提高了（被其他出价者的出价超过），之前出价最高的出价者可以拿回她的钱。
在投标期结束后，受益人需要手动调用合约来接收他的钱 - 合约不能自己激活接收。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    contract SimpleAuction {
        // 拍卖的参数。
        address payable public beneficiary;
        // 时间是unix的绝对时间戳（自1970-01-01以来的秒数）
        // 或以秒为单位的时间段。
        uint public auctionEnd;

        // 拍卖的当前状态
        address public highestBidder;
        uint public highestBid;

        //可以取回的之前的出价
        mapping(address => uint) pendingReturns;

        // 拍卖结束后设为 true，将禁止所有的变更
        bool ended;

        // 变更触发的事件
        event HighestBidIncreased(address bidder, uint amount);
        event AuctionEnded(address winner, uint amount);

        // Errors 用来定义失败

        // 以下称为 natspec 注释，可以通过三个斜杠来识别。
        // 当用户被要求确认交易时或错误发生时将显示。

        /// The auction has already ended.
        error AuctionAlreadyEnded();
        /// There is already a higher or equal bid.
        error BidNotHighEnough(uint highestBid);
        /// The auction has not ended yet.
        error AuctionNotYetEnded();
        /// The function auctionEnd has already been called.
        error AuctionEndAlreadyCalled();

        /// 以受益者地址 `beneficiaryAddress` 的名义，
        /// 创建一个简单的拍卖，拍卖时间为 `biddingTime` 秒。
        constructor(
            uint biddingTime,
            address payable beneficiaryAddress
        ) {
            beneficiary = beneficiaryAddress;
            auctionEnd = block.timestamp + biddingTime;
        }

        /// 对拍卖进行出价，具体的出价随交易一起发送。
        /// 如果没有在拍卖中胜出，则返还出价。
        function bid() external payable {
            // 参数不是必要的。因为所有的信息已经包含在了交易中。
            // 对于能接收以太币的函数，关键字 payable 是必须的。

            // 如果拍卖已结束，撤销函数的调用。
            if (block.timestamp > auctionEndTime)
                revert AuctionAlreadyEnded();

            // 如果出价不够高，返还你的钱
            if (msg.value <= highestBid)
                revert BidNotHighEnough(highestBid);

            if (highestBid != 0) {
                // 返还出价时，简单地直接调用 highestBidder.send(highestBid) 函数，
                // 是有安全风险的，因为它有可能执行一个非信任合约。
                // 更为安全的做法是让接收方自己提取金钱。
                pendingReturns[highestBidder] += highestBid;
            }
            highestBidder = msg.sender;
            highestBid = msg.value;
            emit HighestBidIncreased(msg.sender, msg.value);
        }

        /// 取回出价（当该出价已被超越）
        function withdraw() external returns (bool) {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // 这里很重要，首先要设零值。
                // 因为，作为接收调用的一部分，
                // 接收者可以在 `send` 返回之前，重新调用该函数。
                pendingReturns[msg.sender] = 0;

                // msg.sender is not of type `address payable` and must be
                // explicitly converted using `payable(msg.sender)` in order
                // use the member function `send()`.
                if (!payable(msg.sender).send(amount)) {
                    // 这里不需抛出异常，只需重置未付款
                    pendingReturns[msg.sender] = amount;
                    return false;
                }
            }
            return true;
        }

        /// 结束拍卖，并把最高的出价发送给受益人
        function auctionEnd() external {
            // 对于可与其他合约交互的函数（意味着它会调用其他函数或发送以太币），
            // 一个好的指导方针是将其结构分为三个阶段：
            // 1. 检查条件
            // 2. 执行动作 (可能会改变条件)
            // 3. 与其他合约交互
            // 如果这些阶段相混合，其他的合约可能会回调当前合约并修改状态，
            // 或者导致某些效果（比如支付以太币）多次生效。
            // 如果合约内调用的函数包含了与外部合约的交互，
            // 则它也会被认为是与外部合约有交互的。

            // 1. 条件
            if (block.timestamp < auctionEndTime)
                revert AuctionNotYetEnded();
            if (ended)
                revert AuctionEndAlreadyCalled();

            // 2. 生效
            ended = true;
            emit AuctionEnded(highestBidder, highestBid);

            // 3. 交互
            beneficiary.transfer(highestBid);
        }
    }

秘密竞拍（盲拍）
=====================

上面的公开拍卖接下来将被扩展为一个秘密竞拍。
秘密竞拍的好处是在投标结束前不会有时间压力。
在一个透明的计算平台上进行秘密竞拍听起来像是自相矛盾，但密码学可以实现它。

在 **投标期间** ，投标人实际上并没有发送她的出价，而只是发送一个哈希版本的出价。
由于目前几乎不可能找到两个（足够长的）值，其哈希值是相等的，因此投标人可通过该方式提交报价。
在投标结束后，投标人必须公开他们的出价：他们不加密的发送他们的出价，合约检查出价的哈希值是否与投标期间提供的相同。

另一个挑战是如何使拍卖同时做到 **绑定和秘密** :
唯一能阻止投标者在她赢得拍卖后不付款的方式是，让她将钱连同出价一起发出。
但由于资金转移在以太坊中不能被隐藏，因此任何人都可以看到转移的资金。

下面的合约通过接受任何大于最高出价的值来解决这个问题。
当然，因为这只能在披露阶段进行检查，有些出价可能是 **无效** 的，
并且，这是故意的(与高出价一起，它甚至提供了一个明确的标志来标识无效的出价):
投标人可以通过设置几个或高或低的无效出价来迷惑竞争对手。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    contract BlindAuction {
        struct Bid {
            bytes32 blindedBid;
            uint deposit;
        }

        address payable public beneficiary;
        uint public biddingEnd;
        uint public revealEnd;
        bool public ended;

        mapping(address => Bid[]) public bids;

        address public highestBidder;
        uint public highestBid;

        // 可以取回的之前的出价
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);


        // Errors that describe failures.

        /// The function has been called too early.
        /// Try again at `time`.
        error TooEarly(uint time);
        /// The function has been called too late.
        /// It cannot be called after `time`.
        error TooLate(uint time);
        /// The function auctionEnd has already been called.
        error AuctionEndAlreadyCalled();

        /// 使用 modifier 可以更便捷的校验函数的入参。
        /// `onlyBefore` 会被用于后面的 `bid` 函数：
        /// 新的函数体是由 modifier 本身的函数体，并用原函数体替换 `_;` 语句来组成的。
        modifier onlyBefore(uint time) {
            if (block.timestamp >= time) revert TooLate(time);
            _;
        }
        modifier onlyAfter(uint time) {
            if (block.timestamp <= time) revert TooEarly(time);
            _;
        }

        constructor(
            uint biddingTime,
            uint revealTime,
            address payable beneficiaryAddress
        ) {
            beneficiary = beneficiaryAddress;
            biddingEnd = block.timestamp + biddingTime;
            revealEnd = biddingEnd + revealTime;
        }

        /// 可以通过 `blindedBid` = keccak256(value, fake, secret)
        /// 设置一个秘密竞拍。
        /// 只有在出价披露阶段被正确披露，已发送的以太币才会被退还。
        /// 如果与出价一起发送的以太币至少为 “value” 且 “fake” 不为真，则出价有效。
        /// 将 “fake” 设置为 true ，然后发送满足订金金额但又不与出价相同的金额是隐藏实际出价的方法。
        /// 同一个地址可以放置多个出价。
        function bid(bytes32 blindedBid)
            external
            payable
            onlyBefore(biddingEnd)
        {
            bids[msg.sender].push(Bid({
                blindedBid: blindedBid,
                deposit: msg.value
            }));
        }

        /// 披露你的秘密竞拍出价。
        /// 对于所有正确披露的无效出价以及除最高出价以外的所有出价，你都将获得退款。
        function reveal(
            uint[] calldata values,
            bool[] calldata fake,
            bytes32[] calldata secret
        )
            external
            onlyAfter(biddingEnd)
            onlyBefore(revealEnd)
        {
            uint length = bids[msg.sender].length;
            require(values.length == length);
            require(fake.length == length);
            require(secret.length == length);

            uint refund;
            for (uint i = 0; i < length; i++) {
                Bid storage bid = bids[msg.sender][i];
                (uint value, bool fake, bytes32 secret) =
                        (values[i], fake[i], secret[i]);
                if (bid.blindedBid != keccak256(value, fake, secret)) {
                    // 出价未能正确披露
                    // 不返还订金
                    continue;
                }
                refund += bid.deposit;
                if (!fake && bid.deposit >= value) {
                    if (placeBid(msg.sender, value))
                        refund -= value;
                }
                // 使发送者不可能再次认领同一笔订金
                bid.blindedBid = bytes32(0);
            }
            msg.sender.transfer(refund);
        }

        // 这是一个 "internal" 函数， 意味着它只能在本合约（或继承合约）内被调用
        function placeBid(address bidder, uint value) internal
                returns (bool success)
        {
            if (value <= highestBid) {
                return false;
            }
            if (highestBidder != address(0)) {
                // 返还之前的最高出价
                pendingReturns[highestBidder] += highestBid;
            }
            highestBid = value;
            highestBidder = bidder;
            return true;
        }

        /// 取回出价（当该出价已被超越）
        function withdraw() external {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // 这里很重要，首先要设零值。
                // 因为，作为接收调用的一部分，
                // 接收者可以在 `transfer` 返回之前重新调用该函数。（可查看上面关于‘条件 -> 影响 -> 交互’的标注）
                pendingReturns[msg.sender] = 0;

                msg.sender.transfer(amount);
            }
        }

        /// 结束拍卖，并把最高的出价发送给受益人
        function auctionEnd()
            external
            onlyAfter(revealEnd)
        {
            if (ended) revert AuctionEndAlreadyCalled();
            emit AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }
    }

