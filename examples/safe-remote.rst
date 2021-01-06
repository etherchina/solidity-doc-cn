.. include:: glossaries.rst

.. index:: purchase, remote purchase, escrow

********************
安全的远程购买合约
********************

Purchasing goods remotely currently requires multiple parties that need to trust each other.
The simplest configuration involves a seller and a buyer. The buyer would like to receive
an item from the seller and the seller would like to get money (or an equivalent)
in return. The problematic part is the shipment here: There is no way to determine for
sure that the item arrived at the buyer.

There are multiple ways to solve this problem, but all fall short in one or the other way.
In the following example, both parties have to put twice the value of the item into the
contract as escrow. As soon as this happened, the money will stay locked inside
the contract until the buyer confirms that they received the item. After that,
the buyer is returned the value (half of their deposit) and the seller gets three
times the value (their deposit plus the value). The idea behind
this is that both parties have an incentive to resolve the situation or otherwise
their money is locked forever.

This contract of course does not solve the problem, but gives an overview of how
you can use state machine-like constructs inside a contract.


::

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >0.6.99 <0.8.0;

    contract Purchase {
        uint public value;
        address payable public seller;
        address payable public buyer;

        enum State { Created, Locked, Release, Inactive }

        State public state;


        modifier condition(bool _condition) {
            require(_condition);
            _;
        }

        modifier onlyBuyer() {
            require(
                msg.sender == buyer,
                "Only buyer can call this."
            );
            _;
        }

        modifier onlySeller() {
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        modifier inState(State _state) {
            require(
                state == _state,
                "Invalid state."
            );
            _;
        }

        event Aborted();
        event PurchaseConfirmed();
        event ItemReceived();
        event SellerRefunded();

        //确保 `msg.value` 是一个偶数。
        //如果它是一个奇数，则它将被截断。
        //通过乘法检查它不是奇数。
        constructor() payable {
            seller = msg.sender;
            value = msg.value / 2;
            require((2 * value) == msg.value, "Value has to be even.");
        }


        ///中止购买并回收以太币。
        ///只能在合约被锁定之前由卖家调用。
        function abort()
            public
            onlySeller
            inState(State.Created)
        {
            emit Aborted();
            state = State.Inactive;
            seller.transfer(address(this).balance);
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
            // It is important to change the state first because
            // otherwise, the contracts called using `send` below
            // can call in again here.
            state = State.Release;

            buyer.transfer(value);
        }

        /// This function refunds the seller, i.e.
        /// pays back the locked funds of the seller.
        function refundSeller()
            public
            onlySeller
            inState(State.Release)
        {
            emit SellerRefunded();
            // It is important to change the state first because
            // otherwise, the contracts called using `send` below
            // can call in again here.
            state = State.Inactive;

            seller.transfer(3 * value);
        }
    }



    }
