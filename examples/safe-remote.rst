.. include:: ../glossaries.rst

.. index:: purchase, remote purchase, escrow

********************
安全的远程购买合约
********************

译者注，在官方文档中，这个合约没有任何的说明，这个合约译者认为作者是想展示合约 |modifier| 的用法。

现在大家只需了解下 |modifier| 的作用，后面的文档有 :ref:`更多关于函数修饰器的使用 <modifiers>`。

::

    pragma solidity >=0.4.22 <0.7.0;

    contract Purchase {
        uint public value;
        address payable public seller;
        address payable public buyer;
        enum State { Created, Locked, Inactive }
        State public state;

        //确保 `msg.value` 是一个偶数。
        //如果它是一个奇数，则它将被截断。
        //通过乘法检查它不是奇数。
        constructor() public payable {
            seller = msg.sender;
            value = msg.value / 2;
            require((2 * value) == msg.value, "Value has to be even.");
        }

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
            // 首先修改状态很重要，否则的话，由 `transfer` 所调用的合约可以回调进这里（再次接收以太币）。
            state = State.Inactive;

            // 注意: 这实际上允许买方和卖方阻止退款 - 应该使用取回模式。
            buyer.transfer(value);
            seller.transfer(address(this).balance);
        }
    }
