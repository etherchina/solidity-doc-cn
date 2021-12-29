.. include :: glossaries.rst
.. index:: ! contract

##########
合约
##########

Solidity 合约类似于面向对象语言中的类。合约中有用于数据持久化的状态变量，和可以修改状态变量的函数。
调用另一个合约实例的函数时，会执行一个 EVM 函数调用，这个操作会切换执行时的上下文，这样，前一个合约的状态变量就不能访问了。

.. index:: ! contract;creation, constructor

**********
创建合约
**********

可以通过以太坊交易“从外部”或从 Solidity 合约内部创建合约。

一些集成开发环境，例如 `Remix <https://remix.ethereum.org/>`_, 通过使用一些用户界面元素使创建过程更加流畅。
在以太坊上编程创建合约最好使用 JavaScript API `web3.js <https://github.com/ethereum/web3.js>`_。
现在，我们已经有了一个叫做 `web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_ 的方法能够更容易的创建合约。

创建合约时，会执行一次构造函数（与合约同名的函数）。构造函数是可选的。只允许有一个构造函数，这意味着不支持重载。

.. index:: constructor;arguments

在内部，构造函数参数在合约代码之后通过 :ref:`ABI 编码 <ABI>` 传递，但是如果你使用 ``web3.js``
则不必关心这个问题。

如果一个合约想要创建另一个合约，那么创建者必须知晓被创建合约的源代码(和二进制代码)。
这意味着不可能循环创建依赖项。

::

    pragma solidity ^0.4.16;

    contract OwnedToken {
        // TokenCreator 是如下定义的合约类型.
        // 不创建新合约的话，也可以引用它。
        TokenCreator creator;
        address owner;
        bytes32 name;

        // 这是注册 creator 和设置名称的构造函数。
        function OwnedToken(bytes32 _name) public {
            // 状态变量通过其名称访问，而不是通过例如 this.owner 的方式访问。
            // 这也适用于函数，特别是在构造函数中，你只能像这样（“内部地”）调用它们，
            // 因为合约本身还不存在。
            owner = msg.sender;
            // 从 `address` 到 `TokenCreator` ，是做显式的类型转换
            // 并且假定调用合约的类型是 TokenCreator，没有真正的方法来检查这一点。
            creator = TokenCreator(msg.sender);
            name = _name;
        }

        function changeName(bytes32 newName) public {
            // 只有 creator （即创建当前合约的合约）能够更改名称 —— 因为合约是隐式转换为地址的，
            // 所以这里的比较是可行的。
            if (msg.sender == address(creator))
                name = newName;
        }

        function transfer(address newOwner) public {
            // 只有当前所有者才能发送 token。
            if (msg.sender != owner) return;
            // 我们也想询问 creator 是否可以发送。
            // 请注意，这里调用了一个下面定义的合约中的函数。
            // 如果调用失败（比如，由于 gas 不足），会立即停止执行。
            if (creator.isTokenTransferOK(owner, newOwner))
                owner = newOwner;
        }
    }

    contract TokenCreator {
        function createToken(bytes32 name)
           public
           returns (OwnedToken tokenAddress)
        {
            // 创建一个新的 Token 合约并且返回它的地址。
            // 从 JavaScript 方面来说，返回类型是简单的 `address` 类型，因为
            // 这是在 ABI 中可用的最接近的类型。
            return new OwnedToken(name);
        }

        function changeName(OwnedToken tokenAddress, bytes32 name)  public {
            // 同样，`tokenAddress` 的外部类型也是 `address` 。
            tokenAddress.changeName(name);
        }

        function isTokenTransferOK(address currentOwner, address newOwner)
            public
            view
            returns (bool ok)
        {
            // 检查一些任意的情况。
            address tokenAddress = msg.sender;
            return (keccak256(newOwner) & 0xff) == (bytes20(tokenAddress) & 0xff);
        }
    }

.. index:: ! visibility, external, public, private, internal

.. _visibility-and-getters:

**********************
可见性和 getter 函数
**********************

由于 Solidity 有两种函数调用（内部调用不会产生实际的 EVM 调用或称为“消息调用”，而外部调用则会产生一个 EVM 调用），
函数和状态变量有四种可见性类型。
函数可以指定为 ``external`` ，``public`` ，``internal`` 或者 ``private``，默认情况下函数类型为 ``public``。
对于状态变量，不能设置为 ``external`` ，默认是 ``internal`` 。

``external`` ：
    外部函数作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。
    一个外部函数 ``f`` 不能从内部调用（即 ``f`` 不起作用，但 ``this.f()`` 可以）。
    当收到大量数据的时候，外部函数有时候会更有效率。
``public`` ：
    public 函数是合约接口的一部分，可以在内部或通过消息调用。对于公共状态变量，
    会自动生成一个 getter 函数（见下面）。
``internal`` ：
    这些函数和状态变量只能是内部访问（即从当前合约内部或从它派生的合约访问），不使用 ``this`` 调用。
``private`` ：
    private 函数和状态变量仅在当前定义它们的合约中使用，并且不能被派生合约使用。

.. note::
    合约中的所有内容对外部观察者都是可见的。设置一些 ``private`` 类型只能阻止其他合约访问和修改这些信息，
    但是对于区块链外的整个世界它仍然是可见的。

可见性标识符的定义位置，对于状态变量来说是在类型后面，对于函数是在参数列表和返回关键字中间。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }

在下面的例子中，``D`` 可以调用 ``c.getData（）`` 来获取状态存储中 ``data`` 的值，但不能调用 ``f`` 。
合约 ``E`` 继承自 ``C`` ，因此可以调用 ``compute``。

::

    // 下面代码编译错误

    pragma solidity ^0.4.0;

    contract C {
        uint private data;

        function f(uint a) private returns(uint b) { return a + 1; }
        function setData(uint a) public { data = a; }
        function getData() public returns(uint) { return data; }
        function compute(uint a, uint b) internal returns (uint) { return a+b; }
    }

    contract D {
        function readData() public {
            C c = new C();
            uint local = c.f(7); // 错误：成员 `f` 不可见
            c.setData(3);
            local = c.getData();
            local = c.compute(3, 5); // 错误：成员 `compute` 不可见
        }
    }

    contract E is C {
        function g() public {
            C c = new C();
            uint val = compute(3, 5); // 访问内部成员（从继承合约访问父合约成员）
        }
    }

.. index:: ! getter;function, ! function;getter
.. _getter-functions:

Getter 函数
================

编译器自动为所有 **public** 状态变量创建 getter 函数。对于下面给出的合约，编译器会生成一个名为 ``data`` 的函数，
该函数不会接收任何参数并返回一个 ``uint`` ，即状态变量 ``data`` 的值。可以在声明时完成状态变量的初始化。

::

    pragma solidity ^0.4.0;

    contract C {
        uint public data = 42;
    }

    contract Caller {
        C c = new C();
        function f() public {
            uint local = c.data();
        }
    }

getter 函数具有外部可见性。如果在内部访问 getter（即没有 ``this.`` ），它被认为一个状态变量。
如果它是外部访问的（即用 ``this.`` ），它被认为为一个函数。

::

    pragma solidity ^0.4.0;

    contract C {
        uint public data;
        function x() public {
            data = 3; // 内部访问
            uint val = this.data(); // 外部访问
        }
    }

下一个例子稍微复杂一些：

::

    pragma solidity ^0.4.0;

    contract Complex {
        struct Data {
            uint a;
            bytes3 b;
            mapping (uint => uint) map;
        }
        mapping (uint => mapping(bool => Data[])) public data;
    }

这将会生成以下形式的函数 ::

    function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
        a = data[arg1][arg2][arg3].a;
        b = data[arg1][arg2][arg3].b;
    }

请注意，因为没有好的方法来提供映射的键，所以结构中的映射被省略。

.. index:: ! function;modifier

.. _modifiers:

******************
函数 |modifier|
******************

使用 |modifier| 可以轻松改变函数的行为。 例如，它们可以在执行函数之前自动检查某个条件。
|modifier| 是合约的可继承属性，
并可能被派生合约覆盖。

::

    pragma solidity ^0.4.11;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;

        // 这个合约只定义一个修饰器，但并未使用： 它将会在派生合约中用到。
        // 修饰器所修饰的函数体会被插入到特殊符号 _; 的位置。
        // 这意味着如果是 owner 调用这个函数，则函数会被执行，否则会抛出异常。
        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }
    }

    contract mortal is owned {
        // 这个合约从 `owned` 继承了 `onlyOwner` 修饰符，并将其应用于 `close` 函数，
        // 只有在合约里保存的 owner 调用 `close` 函数，才会生效。
        function close() public onlyOwner {
            selfdestruct(owner);
        }
    }

    contract priced {
        // 修改器可以接收参数：
        modifier costs(uint price) {
            if (msg.value >= price) {
                _;
            }
        }
    }

    contract Register is priced, owned {
        mapping (address => bool) registeredAddresses;
        uint price;

        function Register(uint initialPrice) public { price = initialPrice; }

        // 在这里也使用关键字 `payable` 非常重要，否则函数会自动拒绝所有发送给它的以太币。
        function register() public payable costs(price) {
            registeredAddresses[msg.sender] = true;
        }

        function changePrice(uint _price) public onlyOwner {
            price = _price;
        }
    }

    contract Mutex {
        bool locked;
        modifier noReentrancy() {
            require(!locked);
            locked = true;
            _;
            locked = false;
        }

        // 这个函数受互斥量保护，这意味着 `msg.sender.call` 中的重入调用不能再次调用  `f`。
        // `return 7` 语句指定返回值为 7，但修改器中的语句 `locked = false` 仍会执行。
        function f() public noReentrancy returns (uint) {
            require(msg.sender.call());
            return 7;
        }
    }

如果同一个函数有多个 |modifier|，它们之间以空格隔开，|modifier| 会依次检查执行。

.. warning::
    在早期的 Solidity 版本中，有 |modifier| 的函数，``return`` 语句的行为表现不同。

|modifier| 或函数体中显式的 return 语句仅仅跳出当前的 |modifier| 和函数体。
返回变量会被赋值，但整个执行逻辑会从前一个 |modifier| 中的定义的 “_” 之后继续执行。

|modifier| 的参数可以是任意表达式，在此上下文中，所有在函数中可见的符号，在 |modifier| 中均可见。
在 |modifier| 中引入的符号在函数中不可见（可能被重载改变）。

.. index:: ! constant

************************
Constant 状态变量
************************

状态变量可以被声明为 ``constant``。在这种情况下，只能使用那些在编译时有确定值的表达式来给它们赋值。
任何通过访问 storage，区块链数据（例如 ``now``, ``this.balance`` 或者 ``block.number``）或执行数据（ ``msg.gas`` ）
或对外部合约的调用来给它们赋值都是不允许的。
在内存分配上有副作用（side-effect）的表达式是允许的，但对其他内存对象产生副作用的表达式则不行。
内建（built-in）函数 ``keccak256``，``sha256``，``ripemd160``，``ecrecover``，``addmod`` 和 ``mulmod`` 是允许的（即使他们确实会调用外部合约）。

允许带有副作用的内存分配器的原因是这将允许构建复杂的对象，比如查找表（lookup-table）。
此功能尚未完全可用。

编译器不会为这些变量预留存储，它们的每次出现都会被替换为相应的常量表达式（这将可能被优化器计算为实际的某个值）。

不是所有类型的状态变量都支持用 constant 来修饰，当前支持的仅有值类型和字符串。

::

    pragma solidity ^0.4.0;

    contract C {
        uint constant x = 32**22 + 8;
        string constant text = "abc";
        bytes32 constant myHash = keccak256("abc");
    }

.. index:: ! functions

.. _functions:

******
函数
******

.. index:: ! view function, function;view

.. _view-functions:

View 函数
==============

可以将函数声明为 ``view`` 类型，这种情况下要保证不修改状态。

下面的语句被认为是修改状态：

#. 修改状态变量。
#. :ref:`产生事件 <events>`。
#. :ref:`创建其它合约 <creating-contracts>`。
#. 使用 ``selfdestruct``。
#. 通过调用发送以太币。
#. 调用任何没有标记为 ``view`` 或者 ``pure`` 的函数。
#. 使用低级调用。
#. 使用包含特定操作码的内联汇编。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + now;
        }
    }

.. note::
  ``constant`` 是 ``view`` 的别名。

.. note::
  Getter 方法被标记为 ``view``。

.. warning::
  编译器没有强制 ``view`` 方法不能修改状态。

.. index:: ! pure function, function;pure

.. _pure-functions:

Pure 函数
==============

函数可以声明为 ``pure`` ，在这种情况下，承诺不读取或修改状态。

除了上面解释的状态修改语句列表之外，以下被认为是从状态中读取：

#. 读取状态变量。
#. 访问 ``this.balance`` 或者 ``<address>.balance``。
#. 访问 ``block``，``tx``， ``msg`` 中任意成员 （除 ``msg.sig`` 和 ``msg.data`` 之外）。
#. 调用任何未标记为 ``pure`` 的函数。
#. 使用包含某些操作码的内联汇编。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }

.. warning::
  编译器没有强制 ``pure`` 方法不能读取状态。

.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fallback 函数
=================

合约可以有一个未命名的函数。这个函数不能有参数也不能有返回值。
如果在一个到合约的调用中，没有其他函数与给定的函数标识符匹配（或没有提供调用数据），那么这个函数（fallback 函数）会被执行。

除此之外，每当合约收到以太币（没有任何数据），这个函数就会执行。此外，为了接收以太币，fallback 函数必须标记为 ``payable``。
如果不存在这样的函数，则合约不能通过常规交易接收以太币。

在这样的上下文中，通常只有很少的 gas 可以用来完成这个函数调用（准确地说，是 2300 gas），所以使 fallback 函数的调用尽量廉价很重要。
请注意，调用 fallback 函数的交易（而不是内部调用）所需的 gas 要高得多，因为每次交易都会额外收取 21000 gas 或更多的费用，用于签名检查等操作。

具体来说，以下操作会消耗比 fallback 函数更多的 gas：

- 写入存储
- 创建合约
- 调用消耗大量 gas 的外部函数
- 发送以太币

请确保您在部署合约之前彻底测试您的 fallback 函数，以确保执行成本低于 2300 个 gas。

.. note::
    即使 fallback 函数不能有参数，仍然可以使用 ``msg.data`` 来获取随调用提供的任何有效数据。

.. warning::
    一个没有定义 fallback 函数的合约，直接接收以太币（没有函数调用，即使用 ``send`` 或 ``transfer``）会抛出一个异常，
    并返还以太币（在 Solidity v0.4.0 之前行为会有所不同）。所以如果你想让你的合约接收以太币，必须实现 fallback 函数。

.. warning::
    一个没有 payable fallback 函数的合约，可以作为 `coinbase transaction` （又名 `miner block reward` ）的接收者或者作为 ``selfdestruct`` 的目标来接收以太币。

    一个合约不能对这种以太币转移做出反应，因此也不能拒绝它们。这是 EVM 在设计时就决定好的，而且 Solidity 无法绕过这个问题。

    这也意味着 ``this.balance`` 可以高于合约中实现的一些手工记帐的总和（即在 fallback 函数中更新的累加器）。

::

    pragma solidity ^0.4.0;

    contract Test {
        // 发送到这个合约的所有消息都会调用此函数（因为该合约没有其它函数）。
        // 向这个合约发送以太币会导致异常，因为 fallback 函数没有 `payable` 修饰符
        function() public { x = 1; }
        uint x;
    }


    // 这个合约会保留所有发送给它的以太币，没有办法返还。
    contract Sink {
        function() public payable { }
    }

    contract Caller {
        function callTest(Test test) public {
            test.call(0xabcdef01); // 不存在的哈希
            // 导致 test.x 变成 == 1。
            // 以下将不会编译，但如果有人向该合约发送以太币，交易将失败并拒绝以太币。
            // test.send(2 ether）;
        }
    }

.. index:: ! overload

.. _overload-function:

函数重载
====================

合约可以具有多个不同参数的同名函数。这也适用于继承函数。以下示例展示了合约 ``A`` 中的重载函数 ``f``。

::

    pragma solidity ^0.4.16;

    contract A {
        function f(uint _in) public pure returns (uint out) {
            out = 1;
        }

        function f(uint _in, bytes32 _key) public pure returns (uint out) {
            out = 2;
        }
    }

重载函数也存在于外部接口中。如果两个外部可见函数仅区别于 Solidity 内的类型而不是它们的外部类型则会导致错误。

::

    // 以下代码无法编译
    pragma solidity ^0.4.16;

    contract A {
        function f(B _in) public pure returns (B out) {
            out = _in;
        }

        function f(address _in) public pure returns (address out) {
            out = _in;
        }
    }

    contract B {
    }


以上两个 ``f`` 函数重载都接受了 ABI 的地址类型，虽然它们在 Solidity 中被认为是不同的。

重载解析和参数匹配
-----------------------------------------

通过将当前范围内的函数声明与函数调用中提供的参数相匹配，可以选择重载函数。
如果所有参数都可以隐式地转换为预期类型，则选择函数作为重载候选项。如果一个候选都没有，解析失败。

.. note::
    返回参数不作为重载解析的依据。

::

    pragma solidity ^0.4.16;

    contract A {
        function f(uint8 _in) public pure returns (uint8 out) {
            out = _in;
        }

        function f(uint256 _in) public pure returns (uint256 out) {
            out = _in;
        }
    }

调用  ``f(50)`` 会导致类型错误，因为 ``50`` 既可以被隐式转换为 ``uint8`` 也可以被隐式转换为 ``uint256``。
另一方面，调用 ``f(256)`` 则会解析为 ``f(uint256)`` 重载，因为 ``256`` 不能隐式转换为 ``uint8``。

.. index:: ! event

.. _events:

******
事件
******

事件允许我们方便地使用 EVM 的日志基础设施。
我们可以在 dapp 的用户界面中监听事件，EVM 的日志机制可以反过来“调用”用来监听事件的 Javascript 回调函数。

事件在合约中可被继承。当他们被调用时，会使参数被存储到交易的日志中 —— 一种区块链中的特殊数据结构。
这些日志与地址相关联，被并入区块链中，只要区块可以访问就一直存在（在 Frontier 和 Homestead 版本中会被永久保存，在 Serenity 版本中可能会改动)。
日志和事件在合约内不可直接被访问（甚至是创建日志的合约也不能访问）。

对日志的 SPV（Simplified Payment Verification）证明是可能的，如果一个外部实体提供了一个带有这种证明的合约，它可以检查日志是否真实存在于区块链中。
但需要留意的是，由于合约中仅能访问最近的 256 个区块哈希，所以还需要提供区块头信息。

最多三个参数可以接收 ``indexed`` 属性，从而使它们可以被搜索：在用户界面上可以使用 indexed 参数的特定值来进行过滤。

如果数组（包括 ``string`` 和 ``bytes``）类型被标记为索引项，则它们的 keccak-256 哈希值会被作为 topic 保存。

除非你用 ``anonymous`` 说明符声明事件，否则事件签名的哈希值是 topic 之一。
同时也意味着对于匿名事件无法通过名字来过滤。

所有非索引参数都将存储在日志的数据部分中。

.. note::
    索引参数本身不会被保存。你只能搜索它们的值（来确定相应的日志数据是否存在），而不能获取它们的值本身。

::

    pragma solidity ^0.4.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // 我们可以过滤对 `Deposit` 的调用，从而用 Javascript API 来查明对这个函数的任何调用（甚至是深度嵌套调用）。
            Deposit(msg.sender, _id, msg.value);
        }
    }

使用 JavaScript API 调用事件的用法如下：

::

    var abi = /* abi 由编译器产生 */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* 地址 */);

    var event = clientReceipt.Deposit();

    // 监视变化
    event.watch(function(error, result){
        // 结果包括对 `Deposit` 的调用参数在内的各种信息。
        if (!error)
            console.log(result);
    });

    // 或者通过回调立即开始观察
    var event = clientReceipt.Deposit(function(error, result) {
        if (!error)
            console.log(result);
    });

.. index:: ! log

日志的底层接口
===========================

通过函数 ``log0``，``log1``， ``log2``， ``log3`` 和 ``log4`` 可以访问日志机制的底层接口。
``logi``  接受 ``i + 1`` 个 ``bytes32`` 类型的参数。其中第一个参数会被用来做为日志的数据部分，
其它的会做为 topic。上面的事件调用可以以相同的方式执行。

::

    pragma solidity ^0.4.10;

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

其中的长十六进制数的计算方法是 ``keccak256("Deposit(address,hash256,uint256)")``，即事件的签名。

其它学习事件机制的资源
==============================================

- `Javascript 文档 <https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events>`_
- `事件使用例程 <https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol>`_
- `如何在 js 中访问它们 <https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js>`_

.. index:: ! inheritance, ! base class, ! contract;base, ! deriving

***********
继承
***********

通过复制包括多态的代码，Solidity 支持多重继承。

所有的函数调用都是虚拟的，这意味着最远的派生函数会被调用，除非明确给出合约名称。

当一个合约从多个合约继承时，在区块链上只有一个合约被创建，所有基类合约的代码被复制到创建的合约中。

总的来说，Solidity 的继承系统与 `Python的继承系统 <https://docs.python.org/3/tutorial/classes.html#inheritance>`_ ，非常
相似，特别是多重继承方面。

下面的例子进行了详细的说明。

::

    pragma solidity ^0.4.16;

    contract owned {
        function owned() { owner = msg.sender; }
        address owner;
    }

    // 使用 is 从另一个合约派生。派生合约可以访问所有非私有成员，包括内部函数和状态变量，
    // 但无法通过 this 来外部访问。
    contract mortal is owned {
        function kill() {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // 这些抽象合约仅用于给编译器提供接口。
    // 注意函数没有函数体。
    // 如果一个合约没有实现所有函数，则只能用作接口。
    contract Config {
        function lookup(uint id) public returns (address adr);
    }

    contract NameReg {
        function register(bytes32 name) public;
        function unregister() public;
     }

    // 可以多重继承。请注意，owned 也是 mortal 的基类，
    // 但只有一个 owned 实例（就像 C++ 中的虚拟继承）。
    contract named is owned, mortal {
        function named(bytes32 name) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // 函数可以被另一个具有相同名称和相同数量/类型输入的函数重载。
        // 如果重载函数有不同类型的输出参数，会导致错误。
        // 本地和基于消息的函数调用都会考虑这些重载。
        function kill() public {
            if (msg.sender == owner) {
                Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
                NameReg(config.lookup(1)).unregister();
                // 仍然可以调用特定的重载函数。
                mortal.kill();
            }
        }
    }

    // 如果构造函数接受参数，
    // 则需要在声明（合约的构造函数）时提供，
    // 或在派生合约的构造函数位置以修饰器调用风格提供（见下文）。
    contract PriceFeed is owned, mortal, named("GoldFeed") {
       function updateInfo(uint newInfo) public {
          if (msg.sender == owner) info = newInfo;
       }

       function get() public view returns(uint r) { return info; }

       uint info;
    }

注意，在上边的代码中，我们调用 ``mortal.kill()`` 来“转发”销毁请求。
这样做法是有问题的，在下面的例子中可以看到::

    pragma solidity ^0.4.0;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* 清除操作 1 */ mortal.kill(); }
    }

    contract Base2 is mortal {
        function kill() public { /* 清除操作 2 */ mortal.kill(); }
    }

    contract Final is Base1, Base2 {
    }

调用 ``Final.kill()`` 时会调用最远的派生重载函数 ``Base2.kill``，但是会绕过 ``Base1.kill``，
主要是因为它甚至都不知道 ``Base1`` 的存在。解决这个问题的方法是使用 ``super``::

    pragma solidity ^0.4.0;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* 清除操作 1 */ super.kill(); }
    }


    contract Base2 is mortal {
        function kill() public { /* 清除操作 2 */ super.kill(); }
    }

    contract Final is Base1, Base2 {
    }

如果 ``Base2`` 调用 ``super`` 的函数，它不会简单在其基类合约上调用该函数。
相反，它在最终的继承关系图谱的下一个基类合约中调用这个函数，所以它会调用 ``Base1.kill()``
（注意最终的继承序列是——从最远派生合约开始：Final, Base2, Base1, mortal, ownerd）。
在类中使用 super 调用的实际函数在当前类的上下文中是未知的，尽管它的类型是已知的。
这与普通的虚拟方法查找类似。

.. index:: ! base;constructor

基类构造函数的参数
===============================

派生合约需要提供基类构造函数需要的所有参数。这可以通过两种方式来完成::

    pragma solidity ^0.4.0;

    contract Base {
        uint x;
        function Base(uint _x) public { x = _x; }
    }

    contract Derived is Base(7) {
        function Derived(uint _y) Base(_y * _y) public {
        }
    }

一种方法直接在继承列表中调用基类构造函数（``is Base(7)``）。
另一种方法是像 |modifier| 使用方法一样，
作为派生合约构造函数定义头的一部分，（``Base(_y * _y)``)。
如果构造函数参数是常量并且定义或描述了合约的行为，使用第一种方法比较方便。
如果基类构造函数的参数依赖于派生合约，那么必须使用第二种方法。
如果像这个简单的例子一样，两个地方都用到了，优先使用 |modifier| 风格的参数。

.. index:: ! inheritance;multiple, ! linearization, ! C3 linearization

多重继承与线性化
======================================

编程语言实现多重继承需要解决几个问题。
一个问题是 `钻石问题 <https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem>`_。
Solidity 借鉴了 Python 的方式并且使用“ `C3 线性化 <https://en.wikipedia.org/wiki/C3_linearization>`_ ”强制一个由基类构成的 DAG（有向无环图）保持一个特定的顺序。
这最终反映为我们所希望的唯一化的结果，但也使某些继承方式变为无效。尤其是，基类在 ``is`` 后面的顺序很重要。
在下面的代码中，Solidity 会给出“ Linearization of inheritance graph impossible ”这样的错误。

::

    // 以下代码编译出错

    pragma solidity ^0.4.0;

    contract X {}
    contract A is X {}
    contract C is A, X {}

代码编译出错的原因是 ``C`` 要求 ``X`` 重写 ``A`` （因为定义的顺序是 ``A, X`` ），
但是 ``A`` 本身要求重写 ``X``，无法解决这种冲突。

可以通过一个简单的规则来记忆：
以从“最接近的基类”（most base-like）到“最远的继承”（most derived）的顺序来指定所有的基类。

继承有相同名字的不同类型成员
======================================================

当继承导致一个合约具有相同名字的函数和 |modifier| 时，这会被认为是一个错误。
当事件和 |modifier| 同名，或者函数和事件同名时，同样会被认为是一个错误。
有一种例外情况，状态变量的 getter 可以覆盖一个 public 函数。

.. index:: ! contract;abstract, ! abstract contract

******************
抽象合约
******************

合约函数可以缺少实现，如下例所示（请注意函数声明头由 ``;`` 结尾）::

    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

这些合约无法成功编译（即使它们除了未实现的函数还包含其他已经实现了的函数），但他们可以用作基类合约::

    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }

如果合约继承自抽象合约，并且没有通过重写来实现所有未实现的函数，那么它本身就是抽象的。

.. index:: ! contract;interface, ! interface contract

**********
接口
**********

接口类似于抽象合约，但是它们不能实现任何函数。还有进一步的限制：

#. 无法继承其他合约或接口。
#. 无法定义构造函数。
#. 无法定义变量。
#. 无法定义结构体
#. 无法定义枚举。

将来可能会解除这里的某些限制。

接口基本上仅限于合约 ABI 可以表示的内容，并且 ABI 和接口之间的转换应该不会丢失任何信息。

接口由它们自己的关键字表示：

::

    pragma solidity ^0.4.11;

    interface Token {
        function transfer(address recipient, uint amount) public;
    }

就像继承其他合约一样，合约可以继承接口。

.. index:: ! library, callcode, delegatecall

.. _libraries:

************
库
************

库与合约类似，它们只需要在特定的地址部署一次，并且它们的代码可以通过 EVM 的 ``DELEGATECALL``
(Homestead 之前使用 ``CALLCODE`` 关键字)特性进行重用。
这意味着如果库函数被调用，它的代码在调用合约的上下文中执行，即 ``this`` 指向调用合约，特别是可以访问调用合约的存储。
因为每个库都是一段独立的代码，所以它仅能访问调用合约明确提供的状态变量（否则它就无法通过名字访问这些变量）。
因为我们假定库是无状态的，所以如果它们不修改状态（也就是说，如果它们是 ``view`` 或者 ``pure`` 函数），
库函数仅可以通过直接调用来使用（即不使用 ``DELEGATECALL`` 关键字），
特别是，除非能规避 Solidity 的类型系统，否则是不可能销毁任何库的。

库可以看作是使用他们的合约的隐式的基类合约。虽然它们在继承关系中不会显式可见，但调用库函数与调用显式的基类合约十分类似
（如果 ``L`` 是库的话，可以使用 ``L.f()`` 调用库函数）。此外，就像库是基类合约一样，对所有使用库的合约，库的 ``internal`` 函数都是可见的。
当然，需要使用内部调用约定来调用内部函数，这意味着所有内部类型，内存类型都是通过引用而不是复制来传递。
为了在 EVM 中实现这些，内部库函数的代码和从其中调用的所有函数都在编译阶段被拉取到调用合约中，然后使用一个 ``JUMP`` 调用来代替 ``DELEGATECALL``。


.. index:: using for, set

下面的示例说明如何使用库（但也请务必看看 :ref:`using for <using-for>` 有一个实现 set 更好的例子）。

::

    pragma solidity ^0.4.16;

    library Set {
      // 我们定义了一个新的结构体数据类型，用于在调用合约中保存数据。
      struct Data { mapping(uint => bool) flags; }

      // 注意第一个参数是“storage reference”类型，因此在调用中参数传递的只是它的存储地址而不是内容。
      // 这是库函数的一个特性。如果该函数可以被视为对象的方法，则习惯称第一个参数为 `self` 。
      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
              return false; // 已经存在
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // 不存在
          self.flags[value] = false;
          return true;
      }

      function contains(Data storage self, uint value)
          public
          view
          returns (bool)
      {
          return self.flags[value];
      }
    }

    contract C {
        Set.Data knownValues;

        function register(uint value) public {
            // 不需要库的特定实例就可以调用库函数，
            // 因为当前合约就是“instance”。
            require(Set.insert(knownValues, value));
        }
        // 如果我们愿意，我们也可以在这个合约中直接访问 knownValues.flags。
    }

当然，你不必按照这种方式去使用库：它们也可以在不定义结构数据类型的情况下使用。
函数也不需要任何存储引用参数，库可以出现在任何位置并且可以有多个存储引用参数。

调用 ``Set.contains``，``Set.insert`` 和 ``Set.remove`` 都被编译为外部调用（ ``DELEGATECALL`` ）。
如果使用库，请注意实际执行的是外部函数调用。
``msg.sender``， ``msg.value`` 和 ``this`` 在调用中将保留它们的值，
（在 Homestead 之前，因为使用了 ``CALLCODE``，改变了 ``msg.sender`` 和 ``msg.value``)。

以下示例展示了如何在库中使用内存类型和内部函数来实现自定义类型，而无需支付外部函数调用的开销：

::

    pragma solidity ^0.4.16;

    library BigInt {
        struct bigint {
            uint[] limbs;
        }

        function fromUint(uint x) internal pure returns (bigint r) {
            r.limbs = new uint[](1);
            r.limbs[0] = x;
        }

        function add(bigint _a, bigint _b) internal pure returns (bigint r) {
            r.limbs = new uint[](max(_a.limbs.length, _b.limbs.length));
            uint carry = 0;
            for (uint i = 0; i < r.limbs.length; ++i) {
                uint a = limb(_a, i);
                uint b = limb(_b, i);
                r.limbs[i] = a + b + carry;
                if (a + b < a || (a + b == uint(-1) && carry > 0))
                    carry = 1;
                else
                    carry = 0;
            }
            if (carry > 0) {
                // 太差了，我们需要增加一个 limb
                uint[] memory newLimbs = new uint[](r.limbs.length + 1);
                for (i = 0; i < r.limbs.length; ++i)
                    newLimbs[i] = r.limbs[i];
                newLimbs[i] = carry;
                r.limbs = newLimbs;
            }
        }

        function limb(bigint _a, uint _limb) internal pure returns (uint) {
            return _limb < _a.limbs.length ? _a.limbs[_limb] : 0;
        }

        function max(uint a, uint b) private pure returns (uint) {
            return a > b ? a : b;
        }
    }

    contract C {
        using BigInt for BigInt.bigint;

        function f() public pure {
            var x = BigInt.fromUint(7);
            var y = BigInt.fromUint(uint(-1));
            var z = x.add(y);
        }
    }

由于编译器无法知道库的部署位置，我们需要通过链接器将这些地址填入最终的字节码中
（请参阅 :ref:`commandline-compiler` 以了解如何使用命令行编译器来链接字节码）。
如果这些地址没有作为参数传递给编译器，编译后的十六进制代码将包含 ``__Set______`` 形式的占位符（其中 ``Set`` 是库的名称）。
可以手动填写地址来将那 40 个字符替换为库合约地址的十六进制编码。

与合约相比，库的限制：

- 没有状态变量
- 不能够继承或被继承
- 不能接收以太币

（将来有可能会解除这些限制）

库的调用保护
=============================

如果库的代码是通过 ``CALL`` 来执行，而不是 ``DELEGATECALL`` 或者 ``CALLCODE`` 那么执行的结果会被回退，
除非是对 ``view`` 或者 ``pure`` 函数的调用。

EVM 没有为合约提供检测是否使用 ``CALL`` 的直接方式，但是合约可以使用 ``ADDRESS`` 操作码找出正在运行的“位置”。
生成的代码通过比较这个地址和构造时的地址来确定调用模式。

更具体地说，库的运行时代码总是从一个 push 指令开始，它在编译时是 20 字节的零。当部署代码运行时，这个常数
被内存中的当前地址替换，修改后的代码存储在合约中。在运行时，这导致部署时地址是第一个被 push 到堆栈上的常数，
对于任何 non-view 和 non-pure 函数，调度器代码都将对比当前地址与这个常数是否一致。

.. index:: ! using for, library

.. _using-for:

*********
Using For
*********

指令 ``using A for B;`` 可用于附加库函数（从库 ``A``）到任何类型（``B``）。
这些函数将接收到调用它们的对象作为它们的第一个参数（像 Python 的 ``self`` 变量）。

``using A for *;`` 的效果是，库 ``A`` 中的函数被附加在任意的类型上。

在这两种情况下，所有函数都会被附加一个参数，即使它们的第一个参数类型与对象的类型不匹配。
函数调用和重载解析时才会做类型检查。

``using A for B;`` 指令仅在当前作用域有效，目前仅限于在当前合约中，后续可能提升到全局范围。
通过引入一个模块，不需要再添加代码就可以使用包括库函数在内的数据类型。

让我们用这种方式将 :ref:`libraries` 中的 set 例子重写::

    pragma solidity ^0.4.16;

    // 这是和之前一样的代码，只是没有注释。
    library Set {
      struct Data { mapping(uint => bool) flags; }

      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
            return false; // 已经存在
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // 不存在
          self.flags[value] = false;
          return true;
      }

      function contains(Data storage self, uint value)
          public
          view
          returns (bool)
      {
          return self.flags[value];
      }
    }

    contract C {
        using Set for Set.Data; // 这里是关键的修改
        Set.Data knownValues;

        function register(uint value) public {
            // Here, all variables of type Set.Data have
            // corresponding member functions.
            // The following function call is identical to
            // `Set.insert(knownValues, value)`
            // 这里， Set.Data 类型的所有变量都有与之相对应的成员函数。
            // 下面的函数调用和 `Set.insert(knownValues, value)` 的效果完全相同。
            require(knownValues.insert(value));
        }
    }

也可以像这样扩展基本类型::

    pragma solidity ^0.4.16;

    library Search {
        function indexOf(uint[] storage self, uint value)
            public
            view
            returns (uint)
        {
            for (uint i = 0; i < self.length; i++)
                if (self[i] == value) return i;
            return uint(-1);
        }
    }

    contract C {
        using Search for uint[];
        uint[] data;

        function append(uint value) public {
            data.push(value);
        }

        function replace(uint _old, uint _new) public {
            // 执行库函数调用
            uint index = data.indexOf(_old);
            if (index == uint(-1))
                data.push(_new);
            else
                data[index] = _new;
        }
    }

注意，所有库调用都是实际的 EVM 函数调用。这意味着如果传递内存或值类型，都将产生一个副本，即使是 ``self`` 变量。
使用存储引用变量是唯一不会发生拷贝的情况。 
