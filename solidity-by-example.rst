############################
根据例子学习Solidity
############################

.. include:: glossaries.rst
.. index:: voting, ballot

.. _voting:

******
投票
******

以下的合约相当复杂，但展示了很多Solidity的功能。它实现了一个投票合约。
当然，电子投票的主要问题是如何将投票权分配给正确的人员以及如何防止被操纵。
我们不会在这里解决所有的问题，但至少我们会展示如何进行委托投票，同时，计票又是 **自动和完全透明的** 。

我们的想法是为每个（投票）表决创建一份合约，为每个选项提供简称。
然后作为合约的创造者——即主席，将给予每个独立的地址以投票权。

地址后面的人可以选择自己投票，或者委托给他们信任的人来投票。

在投票时间结束时，``winningProposal()`` 将返回获得最多投票的提案。


::

    pragma solidity ^0.4.16;

    /// @title 委托投票
    contract Ballot {
        // 这里声明了一个新的复合类型用于稍后的变量
        // 它用来表示一个选民
        struct Voter {
            uint weight; // 计票的权重
            bool voted;  // 若为真，代表该人已投票
            address delegate; // 被委托人
            uint vote;   // 投票提案的索引
        }

        // 提案的类型
        struct Proposal {
            bytes32 name;   // 简称（最长32个字节）
            uint voteCount; // 得票数
        }

        address public chairperson;

        // 这声明了一个状态变量，为每个可能的地址存储一个 `Voter`。
        mapping(address => Voter) public voters;

        // 一个 `Proposal` 结构类型的动态数组
        Proposal[] public proposals;

        /// 为 `proposalNames` 中的每个提案，创建一个新的（投票）表决
        function Ballot(bytes32[] proposalNames) public {
            chairperson = msg.sender;
            voters[chairperson].weight = 1;
            //对于提供的每个提案名称，
            //创建一个新的 Proposal 对象并把它添加到数组的末尾。
            for (uint i = 0; i < proposalNames.length; i++) {
                // `Proposal({...})` 创建一个临时 Proposal 对象，
                // `proposals.push(...)` 将其添加到 `proposals` 的末尾
                proposals.push(Proposal({
                    name: proposalNames[i],
                    voteCount: 0
                }));
            }
        }

        // 授权 `voter` 对这个（投票）表决进行投票
        // 只有 `chairperson` 可以调用该函数。
        function giveRightToVote(address voter) public {
            // 若 `require` 的入参判定为 `false`，
            // 则终止函数，恢复所有对状态和以太币账户的变动，并且也不会消耗 gas 。
            // 如果函数被错误的调用，使用require，将是一个很好的选择。
            require(
                (msg.sender == chairperson) &&
                !voters[voter].voted &&
                (voters[voter].weight == 0)
            );
            voters[voter].weight = 1;
        }

        /// 把你的投票委托到投票者 `to`。
        function delegate(address to) public {
            // 传引用
            Voter storage sender = voters[msg.sender];
            require(!sender.voted);

            // 委托给自己是不允许的
            require(to != msg.sender);

            // 委托是可以传递的，只要被委托者 `to` 也设置了委托。
            // 一般来说，这种循环委托是危险的。因为，如果传递的链条太长，
            // 则可能需消耗的gas要多于区块中剩余的（大于区块设置的gasLimit），
            // 这种情况下，委托不会被执行。
            // 而在另一些情况下，如果形成闭环，则会让合约完全卡住。
            while (voters[to].delegate != address(0)) {
                to = voters[to].delegate;

                // 不允许闭环委托
                require(to != msg.sender);
            }

            // `sender` 是一个引用, 相当于对 `voters[msg.sender].voted` 进行修改
            sender.voted = true;
            sender.delegate = to;
            Voter storage delegate_ = voters[to];
            if (delegate_.voted) {
                // 若被委托者已经投过票了，直接增加得票数
                proposals[delegate_.vote].voteCount += sender.weight;
            } else {
                // 若被委托者还没投票，增加委托者的权重
                delegate_.weight += sender.weight;
            }
        }

        /// 把你的票(包括委托给你的票)，
        /// 投给提案 `proposals[proposal].name`.
        function vote(uint proposal) public {
            Voter storage sender = voters[msg.sender];
            require(!sender.voted);
            sender.voted = true;
            sender.vote = proposal;

            // 如果 `proposal` 超过了数组的范围，则会自动抛出异常，并恢复所有的改动
            proposals[proposal].voteCount += sender.weight;
        }

        /// @dev 结合之前所有的投票，计算出最终胜出的提案
        function winningProposal() public view
                returns (uint winningProposal_)
        {
            uint winningVoteCount = 0;
            for (uint p = 0; p < proposals.length; p++) {
                if (proposals[p].voteCount > winningVoteCount) {
                    winningVoteCount = proposals[p].voteCount;
                    winningProposal_ = p;
                }
            }
        }

        // 调用 winningProposal() 函数以获取提案数组中获胜者的索引，并以此返回获胜者的名称
        function winnerName() public view
                returns (bytes32 winnerName_)
        {
            winnerName_ = proposals[winningProposal()].name;
        }
    }


可能的优化
=====================

当前，为了把投票权分配给所有参与者，需要执行很多交易。你有没有更好的主意？

.. index:: auction;blind, auction;open, blind auction, open auction

***********************
秘密竞价（盲拍）
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

::

    pragma solidity ^0.4.21;

    contract SimpleAuction {
        // 拍卖的参数。
        address public beneficiary;
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

        // 以下是所谓的 natspec 注释，可以通过三个斜杠来识别。
        // 当用户被要求确认交易时将显示。

        /// 以受益者地址 `_beneficiary` 的名义，
        /// 创建一个简单的拍卖，拍卖时间为 `_biddingTime` 秒。
        function SimpleAuction(
            uint _biddingTime,
            address _beneficiary
        ) public {
            beneficiary = _beneficiary;
            auctionEnd = now + _biddingTime;
        }

        /// 对拍卖进行出价，具体的出价随交易一起发送。
        /// 如果没有在拍卖中胜出，则返还出价。
        function bid() public payable {
            // 参数不是必要的。因为所有的信息已经包含在了交易中。
            // 对于能接收以太币的函数，关键字 payable 是必须的。

            // 如果拍卖已结束，撤销函数的调用。
            require(now <= auctionEnd);

            // 如果出价不够高，返还你的钱
            require(msg.value > highestBid);

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
        function withdraw() public returns (bool) {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // 这里很重要，首先要设零值。
                // 因为，作为接收调用的一部分，
                // 接收者可以在 `send` 返回之前，重新调用该函数。
                pendingReturns[msg.sender] = 0;

                if (!msg.sender.send(amount)) {
                    // 这里不需抛出异常，只需重置未付款
                    pendingReturns[msg.sender] = amount;
                    return false;
                }
            }
            return true;
        }

        /// 结束拍卖，并把最高的出价发送给受益人
        function auctionEnd() public {
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
            require(now >= auctionEnd); // 拍卖尚未结束
            require(!ended); // 该函数已被调用

            // 2. 生效
            ended = true;
            emit AuctionEnded(highestBidder, highestBid);

            // 3. 交互
            beneficiary.transfer(highestBid);
        }
    }

秘密竞拍（盲拍）
=====================

之前的公开拍卖接下来将被扩展为一个秘密竞拍。
秘密竞拍的好处是在投标结束前不会有时间压力。
在一个透明的计算平台上进行秘密竞拍听起来像是自相矛盾，但密码学可以实现它。

在 **投标期间** ，投标人实际上并没有发送她的出价，而只是发送一个哈希版本的出价。
由于目前几乎不可能找到两个（足够长的）值，其哈希值是相等的，因此投标人可通过该方式提交报价。
在投标结束后，投标人必须公开他们的出价：他们不加密的发送他们的出价，合约检查出价的哈希值是否与投标期间提供的相同。

另一个挑战是如何使拍卖同时做到 **绑定和秘密** :
唯一能阻止投标者在她赢得拍卖后不付款的方式是，让她将钱连同出价一起发出。
但由于资金转移在 |ethereum| 中不能被隐藏，因此任何人都可以看到转移的资金。

下面的合约通过接受任何大于最高出价的值来解决这个问题。
当然，因为这只能在披露阶段进行检查，有些出价可能是 **无效** 的，
并且，这是故意的(与高出价一起，它甚至提供了一个明确的标志来标识无效的出价):
投标人可以通过设置几个或高或低的无效出价来迷惑竞争对手。

::

    pragma solidity ^0.4.21;

    contract BlindAuction {
        struct Bid {
            bytes32 blindedBid;
            uint deposit;
        }

        address public beneficiary;
        uint public biddingEnd;
        uint public revealEnd;
        bool public ended;

        mapping(address => Bid[]) public bids;

        address public highestBidder;
        uint public highestBid;

        // 可以取回的之前的出价
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);

        /// 使用 modifier 可以更便捷的校验函数的入参。
        /// `onlyBefore` 会被用于后面的 `bid` 函数：
        /// 新的函数体是由 modifier 本身的函数体，并用原函数体替换 `_;` 语句来组成的。
        modifier onlyBefore(uint _time) { require(now < _time); _; }
        modifier onlyAfter(uint _time) { require(now > _time); _; }

        function BlindAuction(
            uint _biddingTime,
            uint _revealTime,
            address _beneficiary
        ) public {
            beneficiary = _beneficiary;
            biddingEnd = now + _biddingTime;
            revealEnd = biddingEnd + _revealTime;
        }

        /// 可以通过 `_blindedBid` = keccak256(value, fake, secret)
        /// 设置一个秘密竞拍。
        /// 只有在出价披露阶段被正确披露，已发送的以太币才会被退还。
        /// 如果与出价一起发送的以太币至少为 “value” 且 “fake” 不为真，则出价有效。
        /// 将 “fake” 设置为 true ，然后发送满足订金金额但又不与出价相同的金额是隐藏实际出价的方法。
        /// 同一个地址可以放置多个出价。
        function bid(bytes32 _blindedBid)
            public
            payable
            onlyBefore(biddingEnd)
        {
            bids[msg.sender].push(Bid({
                blindedBid: _blindedBid,
                deposit: msg.value
            }));
        }

        /// 披露你的秘密竞拍出价。
        /// 对于所有正确披露的无效出价以及除最高出价以外的所有出价，你都将获得退款。
        function reveal(
            uint[] _values,
            bool[] _fake,
            bytes32[] _secret
        )
            public
            onlyAfter(biddingEnd)
            onlyBefore(revealEnd)
        {
            uint length = bids[msg.sender].length;
            require(_values.length == length);
            require(_fake.length == length);
            require(_secret.length == length);

            uint refund;
            for (uint i = 0; i < length; i++) {
                var bid = bids[msg.sender][i];
                var (value, fake, secret) =
                        (_values[i], _fake[i], _secret[i]);
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
            if (highestBidder != 0) {
                // 返还之前的最高出价
                pendingReturns[highestBidder] += highestBid;
            }
            highestBid = value;
            highestBidder = bidder;
            return true;
        }

        /// 取回出价（当该出价已被超越）
        function withdraw() public {
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
            public
            onlyAfter(revealEnd)
        {
            require(!ended);
            emit AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }
    }


.. index:: purchase, remote purchase, escrow

********************
安全的远程购买
********************

::

    pragma solidity ^0.4.21;

    contract Purchase {
        uint public value;
        address public seller;
        address public buyer;
        enum State { Created, Locked, Inactive }
        State public state;

        //确保 `msg.value` 是一个偶数。
        //如果它是一个奇数，则它将被截断。
        //通过乘法检查它不是奇数。
        function Purchase() public payable {
            seller = msg.sender;
            value = msg.value / 2;
            require((2 * value) == msg.value);
        }

        modifier condition(bool _condition) {
            require(_condition);
            _;
        }

        modifier onlyBuyer() {
            require(msg.sender == buyer);
            _;
        }

        modifier onlySeller() {
            require(msg.sender == seller);
            _;
        }

        modifier inState(State _state) {
            require(state == _state);
            _;
        }

        event Aborted();
        event PurchaseConfirmed();
        event ItemReceived();

        ///中止购买并回收以太币。
        ///只能在合约被锁定之前由卖家调用。
        function abort()
            public
            onlySeller
            inState(State.Created)
        {
            emit Aborted();
            state = State.Inactive;
            seller.transfer(this.balance);
        }

        /// 买家确认购买。
        /// 交易必须包含 `2 * value` 个以太币。
        /// 以太币会被锁定，直到 confirmReceived 被调用。
        function confirmPurchase()
            public
            inState(State.Created)
            condition(msg.value == (2 * value))
            payable
        {
            emit PurchaseConfirmed();
            buyer = msg.sender;
            state = State.Locked;
        }

        /// 确认你（买家）已经收到商品。
        /// 这会释放被锁定的以太币。
        function confirmReceived()
            public
            onlyBuyer
            inState(State.Locked)
        {
            emit ItemReceived();
            // 首先修改状态很重要，否则的话，由 `transfer` 所调用的合约可以回调进这里（再次接收以太币）。
            state = State.Inactive;

            // 注意: 这实际上允许买方和卖方阻止退款 - 应该使用取回模式。
            buyer.transfer(value);
            seller.transfer(this.balance);
        }
    }

********************
微支付通道
********************

To be written.
