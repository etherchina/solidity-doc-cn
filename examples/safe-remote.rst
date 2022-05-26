.. include:: glossaries.rst

.. index:: purchase, remote purchase, escrow

********************
安全的远程购买合约
********************

目前，远程购买商品需要多方相互信任。
最简单的情况涉及一个卖家和一个买家。买方希望从卖方那里收到一件物品，卖方希望得到金钱（或等价物）作为回报。有问题的部分是快递。没有办法确定物品是否到达买方手中。

有多种方法来解决这个问题，但都有这样或那样的不足之处。
在下面的例子中，双方都要把物品价值的两倍作为担保费放入合约。只要发生状况，钱就会一直锁在合约里面，直到买方确认收到物品。
而这之后，买方可以退回物品价值（他担保费的一半），卖方得到三倍的价值（他们的押金加上价值）。这背后的想法是，双方都有动力去解决这个问题，否则他们的钱就永远被锁定了。

这个合约当然不能解决问题，但它概述了你如何在合约中使用类似状态机的结构。


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;

    contract Purchase {
        uint public value;
        address payable public seller;
        address payable public buyer;

        enum State { Created, Locked, Release, Inactive }

        State public state;


        modifier condition(bool condition_) {
            require(condition_);
            _;
        }

        /// Only the buyer can call this function.
        error OnlyBuyer();
        /// Only the seller can call this function.
        error OnlySeller();
        /// The function cannot be called at the current state.
        error InvalidState();
        /// The provided value has to be even.
        error ValueNotEven();


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
            seller = payable(msg.sender);
            value = msg.value / 2;
            if ((2 * value) != msg.value)
                revert ValueNotEven();
        }


        ///中止购买并回收以太币。
        ///只能在合约被锁定之前由卖家调用。
        function abort()
            external
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
            external
            inState(State.Created)
            condition(msg.value == (2 * value))
            payable
        {
            emit PurchaseConfirmed();
            buyer = payable(msg.sender);
            state = State.Locked;
        }

        /// 确认你（买家）已经收到商品。
        /// 这会释放被锁定的以太币。
        function confirmReceived()
            external
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
            external
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