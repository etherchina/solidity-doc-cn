###################
根据例子学习Solidity
###################

.. index:: voting, ballot

.. _voting:

******
投票
******

下面的合约就十分的复杂, 但是的确是展现出了`Solidity`的许多特性. 它实现了一个投票的合约. 当然, 电子投票的主要的问题, 如何给正确的人分配投票权和如何阻止暗箱操作. 在这里我们并不会解决所有的问题, 但是 至少我们将展现如何进行一场**自动化记票和完全透明**的委托投票

主要的思路是是对每次投票创建一个合约, 对每个选项提供一个短的名称. 合约的创建者 就作为这次投票的主持人(chairperson)将会给与其他独立地址的投票权利

这些账户(可投票)的拥有者可以自己投这一票, 或者把自己的票委托(delegate)给他们信任的人.

在这次投票结束的时候, `winningProposal()` (function)将会返回这个票数最多的人


::

    pragma solidity ^0.4.16;

	/// @title 委托投票.
    contract Ballot {
        // 声明一个新的复杂类型将会成为一个变量, 会代表其中的一个投票者)
	    struct Voter {
	        uint weight; // 权重会随着委托数量累积
	        bool voted;  // 如果 true 说明这个人已经投过票了
	        address delegate; // 你的委托人
	        uint vote;   // 投票候选人索引号

	    // 这是一个候选人的数据类型
	    struct Proposal {
	        bytes32 name;   // 短名称 (至多 32 bytes)
	        uint voteCount; // 累计票数
	    }

        address public chairperson;

        // 这里创建里一个状态变量(映射), 用于给每个可能的地址分配一个`Voter`的结构
        mapping(address => Voter) public voters;

        // 一个动态大小的候选人(`proposal`)数组
        Proposal[] public proposals;

        /// 使用候选人名称数组(`proposalNames`)创建一轮新的投票 
        function Ballot(bytes32[] proposalNames) public {
            chairperson = msg.sender;
            voters[chairperson].weight = 1;

            // 对每一个候选人的名字创建一个`proposal`的对象, 
            // 并且添加到数组(Array)的尾部	     
            for (uint i = 0; i < proposalNames.length; i++) {
                // `Proposal({...})` 创建了一个临时的 `Proposal` 对象 ,
                // 并且使用 `proposals.push(...)` 把它追加在`proposals`的末尾.
                proposals.push(Proposal({
                    name: proposalNames[i],
                    voteCount: 0
                }));
            }
        }

        // 给与地址投票者的权益
	    // 只可能是被投票主持人所调用
        function giveRightToVote(address voter) public {
		 	// 如何要求(require)的值是 false , 这个函数将会被终结, 并且回滚所有的已做的改变
	    	// 对于函数的非法调用这将是很好的防止办法
	        // 但是值得注意的是, 这个操作将会消耗当前所有的gas(在未来可能有所改善)
            require((msg.sender == chairperson) && !voters[voter].voted && (voters[voter].weight == 0));
            voters[voter].weight = 1;
        }

        /// 将你的票委托到其他的可信投票者
        function delegate(address to) public {
            // 分配 投票者引用
            Voter storage sender = voters[msg.sender];
            require(!sender.voted);

            // 不允许把票委托给自己
            require(to != msg.sender);

	        // 委托权在被委托人那里将会再次被委托
	        // 事实上这种循环是十分危险的
	        // 因为这件运行十分久
	        // 需要的GAS 可能超出一个区块的可用值
	        // 一方面下, 这个委托就不会被执行
	        // 但是另一方面, 这样的循环将会导致这个合约完全的卡住了
            
            (这里避免一个循环委托的可能性 , 如果A委托给B, B委托给其他人,最终到了A)
            while (voters[to].delegate != address(0)) {
                to = voters[to].delegate;

	            // 发现了一个循环委托, 不允许!
                require(to != msg.sender);
            }

            // 因为 `sender` 是一个引用,
	        // 所以这里改变投票状态 `voters[msg.sender].voted`
            sender.voted = true;
            sender.delegate = to;
            Voter storage delegate = voters[to];
            if (delegate.voted) {
               	// 如果委托人的票已经投过, 
                // 那么这里直接修改被选举人的票数(委托人投的那个)
                proposals[delegate.vote].voteCount += sender.weight;
            } else {
                // 如果委托人还没投票, 
                // 那么我们增加委托人的票的权重
                delegate.weight += sender.weight;
            }
        }

	    /// 投票 (包含委托给你的) 
        /// 给 `proposals[proposal].name`.(被选举人)
        function vote(uint proposal) public {
            Voter storage sender = voters[msg.sender];
            require(!sender.voted);
            sender.voted = true;
            sender.vote = proposal;

            // 如果意向被选举人超出了索引范围, 
            // 将会自动的抛出异常并回滚所有数据.
            proposals[proposal].voteCount += sender.weight;
        }

        /// @dev Computes the winning proposal taking all
        /// previous votes into account.
        function winningProposal() public view
                returns (uint winningProposal)
        {
            uint winningVoteCount = 0;
            for (uint p = 0; p < proposals.length; p++) {
                if (proposals[p].voteCount > winningVoteCount) {
                    winningVoteCount = proposals[p].voteCount;
                    winningProposal = p;
                }
            }
        }

        // Calls winningProposal() function to get the index
        // of the winner contained in the proposals array and then
        // returns the name of the winner
        function winnerName() public view
                returns (bytes32 winnerName)
        {
            winnerName = proposals[winningProposal()].name;
        }
    }

Possible Improvements
=====================

Currently, many transactions are needed to assign the rights
to vote to all participants. Can you think of a better way?

.. index:: auction;blind, auction;open, blind auction, open auction

*************
秘密竞价（盲拍）
*************

In this section, we will show how easy it is to create a
completely blind auction contract on Ethereum.
We will start with an open auction where everyone
can see the bids that are made and then extend this
contract into a blind auction where it is not
possible to see the actual bid until the bidding
period ends.

.. _simple_auction:

Simple Open Auction
===================

The general idea of the following simple auction contract
is that everyone can send their bids during
a bidding period. The bids already include sending
money / ether in order to bind the bidders to their
bid. If the highest bid is raised, the previously
highest bidder gets her money back.
After the end of the bidding period, the
contract has to be called manually for the
beneficiary to receive his money - contracts cannot
activate themselves.

::

    pragma solidity ^0.4.11;

    contract SimpleAuction {
        // Parameters of the auction. Times are either
        // absolute unix timestamps (seconds since 1970-01-01)
        // or time periods in seconds.
        address public beneficiary;
        uint public auctionEnd;

        // Current state of the auction.
        address public highestBidder;
        uint public highestBid;

        // Allowed withdrawals of previous bids
        mapping(address => uint) pendingReturns;

        // Set to true at the end, disallows any change
        bool ended;

        // Events that will be fired on changes.
        event HighestBidIncreased(address bidder, uint amount);
        event AuctionEnded(address winner, uint amount);

        // The following is a so-called natspec comment,
        // recognizable by the three slashes.
        // It will be shown when the user is asked to
        // confirm a transaction.

        /// Create a simple auction with `_biddingTime`
        /// seconds bidding time on behalf of the
        /// beneficiary address `_beneficiary`.
        function SimpleAuction(
            uint _biddingTime,
            address _beneficiary
        ) public {
            beneficiary = _beneficiary;
            auctionEnd = now + _biddingTime;
        }

        /// Bid on the auction with the value sent
        /// together with this transaction.
        /// The value will only be refunded if the
        /// auction is not won.
        function bid() public payable {
            // No arguments are necessary, all
            // information is already part of
            // the transaction. The keyword payable
            // is required for the function to
            // be able to receive Ether.

            // Revert the call if the bidding
            // period is over.
            require(now <= auctionEnd);

            // If the bid is not higher, send the
            // money back.
            require(msg.value > highestBid);

            if (highestBidder != 0) {
                // Sending back the money by simply using
                // highestBidder.send(highestBid) is a security risk
                // because it could execute an untrusted contract.
                // It is always safer to let the recipients
                // withdraw their money themselves.
                pendingReturns[highestBidder] += highestBid;
            }
            highestBidder = msg.sender;
            highestBid = msg.value;
            HighestBidIncreased(msg.sender, msg.value);
        }

        /// Withdraw a bid that was overbid.
        function withdraw() public returns (bool) {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // It is important to set this to zero because the recipient
                // can call this function again as part of the receiving call
                // before `send` returns.
                pendingReturns[msg.sender] = 0;

                if (!msg.sender.send(amount)) {
                    // No need to call throw here, just reset the amount owing
                    pendingReturns[msg.sender] = amount;
                    return false;
                }
            }
            return true;
        }

        /// End the auction and send the highest bid
        /// to the beneficiary.
        function auctionEnd() public {
            // It is a good guideline to structure functions that interact
            // with other contracts (i.e. they call functions or send Ether)
            // into three phases:
            // 1. checking conditions
            // 2. performing actions (potentially changing conditions)
            // 3. interacting with other contracts
            // If these phases are mixed up, the other contract could call
            // back into the current contract and modify the state or cause
            // effects (ether payout) to be performed multiple times.
            // If functions called internally include interaction with external
            // contracts, they also have to be considered interaction with
            // external contracts.

            // 1. Conditions
            require(now >= auctionEnd); // auction did not yet end
            require(!ended); // this function has already been called

            // 2. Effects
            ended = true;
            AuctionEnded(highestBidder, highestBid);

            // 3. Interaction
            beneficiary.transfer(highestBid);
        }
    }

Blind Auction
=============

The previous open auction is extended to a blind auction
in the following. The advantage of a blind auction is
that there is no time pressure towards the end of
the bidding period. Creating a blind auction on a
transparent computing platform might sound like a
contradiction, but cryptography comes to the rescue.

During the **bidding period**, a bidder does not
actually send her bid, but only a hashed version of it.
Since it is currently considered practically impossible
to find two (sufficiently long) values whose hash
values are equal, the bidder commits to the bid by that.
After the end of the bidding period, the bidders have
to reveal their bids: They send their values
unencrypted and the contract checks that the hash value
is the same as the one provided during the bidding period.

Another challenge is how to make the auction
**binding and blind** at the same time: The only way to
prevent the bidder from just not sending the money
after he won the auction is to make her send it
together with the bid. Since value transfers cannot
be blinded in Ethereum, anyone can see the value.

The following contract solves this problem by
accepting any value that is larger than the highest
bid. Since this can of course only be checked during
the reveal phase, some bids might be **invalid**, and
this is on purpose (it even provides an explicit
flag to place invalid bids with high value transfers):
Bidders can confuse competition by placing several
high or low invalid bids.


::

    pragma solidity ^0.4.11;

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

        // Allowed withdrawals of previous bids
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);

        /// Modifiers are a convenient way to validate inputs to
        /// functions. `onlyBefore` is applied to `bid` below:
        /// The new function body is the modifier's body where
        /// `_` is replaced by the old function body.
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

        /// Place a blinded bid with `_blindedBid` = keccak256(value,
        /// fake, secret).
        /// The sent ether is only refunded if the bid is correctly
        /// revealed in the revealing phase. The bid is valid if the
        /// ether sent together with the bid is at least "value" and
        /// "fake" is not true. Setting "fake" to true and sending
        /// not the exact amount are ways to hide the real bid but
        /// still make the required deposit. The same address can
        /// place multiple bids.
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

        /// Reveal your blinded bids. You will get a refund for all
        /// correctly blinded invalid bids and for all bids except for
        /// the totally highest.
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
                    // Bid was not actually revealed.
                    // Do not refund deposit.
                    continue;
                }
                refund += bid.deposit;
                if (!fake && bid.deposit >= value) {
                    if (placeBid(msg.sender, value))
                        refund -= value;
                }
                // Make it impossible for the sender to re-claim
                // the same deposit.
                bid.blindedBid = bytes32(0);
            }
            msg.sender.transfer(refund);
        }

        // This is an "internal" function which means that it
        // can only be called from the contract itself (or from
        // derived contracts).
        function placeBid(address bidder, uint value) internal
                returns (bool success)
        {
            if (value <= highestBid) {
                return false;
            }
            if (highestBidder != 0) {
                // Refund the previously highest bidder.
                pendingReturns[highestBidder] += highestBid;
            }
            highestBid = value;
            highestBidder = bidder;
            return true;
        }

        /// Withdraw a bid that was overbid.
        function withdraw() public {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // It is important to set this to zero because the recipient
                // can call this function again as part of the receiving call
                // before `transfer` returns (see the remark above about
                // conditions -> effects -> interaction).
                pendingReturns[msg.sender] = 0;

                msg.sender.transfer(amount);
            }
        }

        /// End the auction and send the highest bid
        /// to the beneficiary.
        function auctionEnd()
            public
            onlyAfter(revealEnd)
        {
            require(!ended);
            AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }
    }


.. index:: purchase, remote purchase, escrow

********************
安全的远程购买
********************

::

    pragma solidity ^0.4.11;

    contract Purchase {
        uint public value;
        address public seller;
        address public buyer;
        enum State { Created, Locked, Inactive }
        State public state;

        // Ensure that `msg.value` is an even number.
        // Division will truncate if it is an odd number.
        // Check via multiplication that it wasn't an odd number.
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

        /// Abort the purchase and reclaim the ether.
        /// Can only be called by the seller before
        /// the contract is locked.
        function abort()
            public
            onlySeller
            inState(State.Created)
        {
            Aborted();
            state = State.Inactive;
            seller.transfer(this.balance);
        }

        /// Confirm the purchase as buyer.
        /// Transaction has to include `2 * value` ether.
        /// The ether will be locked until confirmReceived
        /// is called.
        function confirmPurchase()
            public
            inState(State.Created)
            condition(msg.value == (2 * value))
            payable
        {
            PurchaseConfirmed();
            buyer = msg.sender;
            state = State.Locked;
        }

        /// Confirm that you (the buyer) received the item.
        /// This will release the locked ether.
        function confirmReceived()
            public
            onlyBuyer
            inState(State.Locked)
        {
            ItemReceived();
            // It is important to change the state first because
            // otherwise, the contracts called using `send` below
            // can call in again here.
            state = State.Inactive;

            // NOTE: This actually allows both the buyer and the seller to
            // block the refund - the withdraw pattern should be used.

            buyer.transfer(value);
            seller.transfer(this.balance);
        }
    }

********************
微支付通道
********************

To be written.
