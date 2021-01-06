.. include:: glossaries.rst
.. index:: ! event

.. _events:

************
事件 Events
************


Solidity 事件是EVM的日志功能之上的抽象。
应用程序可以通过以太坊客户端的RPC接口订阅和监听这些事件。


事件在合约中可被继承。当他们被调用时，会使参数被存储到交易的日志中 —— 一种区块链中的特殊数据结构。
这些日志与地址相关联，被并入区块链中，只要区块可以访问就一直存在（现在开始会被永久保存，在 Serenity 版本中可能会改动)。
日志和事件在合约内不可直接被访问（甚至是创建日志的合约也不能访问）。


如果外部实体需要该日志实际上存在于区块链中的证明,可以请求日志的Merkle证明.
但需要留意的是，由于合约中仅能访问最近的 256 个区块哈希，所以还需要提供区块头信息。


对日志的证明是可能的，如果一个外部实体提供了一个带有这种证明的合约，它可以检查日志是否真实存在于区块链中。


最多三个参数可以接收 ``indexed`` 属性（它是一个特殊的名为:ref:`"主题" <abi_events>` 的数据结构， 而不是日志的数据部分）。
如果数组（包括 ``string`` 和 ``bytes``）类型被标记为索引项，则它们的 keccak-256 哈希值会被作为 |topic| 保存，这是因为主题仅仅可以保存单个字（32个字节）。

所有非索引 ``indexed`` 参数是  :ref:`ABI-encoded <ABI>` 都将存储在日志的数据部分中。


|topic| 让我们可以可以搜索事件，比如在为某些事件过滤一些区块，还可以按发起事件的合同地址来过滤事件。


例如, 使用如下的 web3.js ``subscribe("logs")``
`方法 <https://web3js.readthedocs.io/en/1.0/web3-eth-subscribe.html#subscribe-logs>`_ 去过滤符合特定地址的|topic| ：


.. code-block:: javascript

    var options = {
        fromBlock: 0,
        address: web3.eth.defaultAccount,
        topics: ["0x0000000000000000000000000000000000000000000000000000000000000000", null, null]
    };
    web3.eth.subscribe('logs', options, function (error, result) {
        if (!error)
            console.log(result);
    })
        .on("data", function (log) {
            console.log(log);
        })
        .on("changed", function (log) {
    });



除非你用 ``anonymous`` 说明符声明事件，否则事件签名的哈希值是一个 |topic| 。
同时也意味着对于匿名事件无法通过名字来过滤。您可以仅按合约地址过滤。 匿名事件的优势是他们部署和调用的成本更低。

::

    pragma solidity  >=0.4.21 <0.8.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // 事件使用 emit 触发事件。
            // 我们可以过滤对 `Deposit` 的调用，从而用 Javascript API 来查明对这个函数的任何调用（甚至是深度嵌套调用）。
            emit Deposit(msg.sender, _id, msg.value);
        }
    }

使用 JavaScript API 调用事件的用法如下：

::

    var abi = /* abi 由编译器产生 */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...xlb67" /* 地址 */);

    var event = clientReceipt.Deposit();

    // 监听变化
    event.watch(function(error, result) {
        // 结果包含 非索引参数 以及 主题 topic
        if (!error)
            console.log(result);
    });

    // 或者通过传入回调函数，立即开始听监
    var event = clientReceipt.Deposit(function(error, result) {
        if (!error)
            console.log(result);
    });

上面的输出如下所示（有删减）：

.. code-block:: json

  {
     "returnValues": {
         "_from": "0x1111…FFFFCCCC",
         "_id": "0x50…sd5adb20",
         "_value": "0x420042"
     },
     "raw": {
         "data": "0x7f…91385",
         "topics": ["0xfd4…b4ead7", "0x7f…1a91385"]
     }
  }



.. index:: ! log

日志的底层接口
===========================

通过函数 ``log0``，``log1``， ``log2``， ``log3`` 和 ``log4`` 也可以访问日志机制的底层接口。
每个函数 ``logi``  接受 ``i + 1`` 个 ``bytes32`` 类型的参数。其中第一个参数会被用来做为日志的数据部分，
其它的会做为 topic。上面的事件调用可以以如下相同的方式执行：

::

    pragma solidity >=0.4.10 <0.8.0;

    contract C {
        function f() public payable {
            bytes32 _id = 0x420042;
            log3(
                bytes32(msg.value),
                bytes32(0x50cb9fe53daa9737b786ab3646f04d0150dc50ef4e75f59509d83667ad5adb20),
                bytes32(msg.sender),
                _id
            );
        }
    }

其中长串的十六进制数的计算方法是 ``keccak256("Deposit(address,hash256,uint256)")``，即事件的签名。

其它学习事件机制的资源
==============================================

- `Javascript 文档 <https://github.com/ethereum/web3.js/blob/1.x/docs/web3-eth-contract.rst#events>`_
- `Web3.js 中文文档 <https://learnblockchain.cn/docs/web3.js/web3-eth-contract.html#id58>`_
- `事件使用例子 <https://github.com/ethchange/smart-exchange/blob/master/lib/contracts/SmartExchange.sol>`_
- `如何在 js 中访问它们 <https://github.com/ethchange/smart-exchange/blob/master/lib/exchange_transactions.js>`_
