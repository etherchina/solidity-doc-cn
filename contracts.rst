.. index:: ! contract

##########
合约
##########

Solidity 合约类似于面向对象语言中的类。合约中有用于数据持久化的状态变量，和可以修改状态变量的函数。
调用另一个合约实例的函数时，会执行一个 EVM 函数调用，这个操作会切换执行时的上下文，这样，前一个合约的状态变量就不能访问了。

.. index:: ! contract;creation, constructor

******************
创建合约
******************

可以通过以太坊交易 "从外部" 或从 Solidity 合约内部创建合约。
集成开发环境，像 `Remix <https://remix.ethereum.org/>`_, 使用用户界面元素流畅的创建合约。
在以太坊上编程创建合约最好使用 JavaScript API `web3.js <https://github.com/ethereum/web3.js>`_。
从今天开始，有一个名为 `web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_ 的方法能够更容易的创建合约。

创建合约时，会执行一次构造函数（与合约同名的函数）。
构造函数是可选的。 只允许有一个构造函数，这意味着不支持重载。

.. index:: constructor;arguments

在内部，构造函数参数在合约代码之后通过 :ref:`ABI 编码 <ABI>`，但是如果你使用 ``web3.js``
则不必关心这个问题。

如果一个合约想要创建另一个合约，那么创建者必须知晓被创建合约的源代码(和二进制)。
这意味着不可能循环创建依赖项。

::

    pragma solidity ^0.4.16;

    contract OwnedToken {
        // TokenCreator 是如下定义的合约类型.
        // 不创建新合约的话，也可以引用它。
        TokenCreator creator;
        address owner;
        bytes32 name;

        // 这是注册 creator 和分配名称的构造函数。
        function OwnedToken(bytes32 _name) public {
            // 状态变量通过其名称访问，而不是通过例如 this.owner.
            // 这也适用于函数，特别是在构造函数中，你只能那样调用他们（"内部调用"），
            // 因为合约本身还不存在。
            owner = msg.sender;
            // 从 `address` 到 `TokenCreator` ，我们做显式的类型转换
            // 并且假定调用合约的类型是 TokenCreator，没有真正的检查方法。
            creator = TokenCreator(msg.sender);
            name = _name;
        }

        function changeName(bytes32 newName) public {
            // 只有 creator 能够更改名称 -- 因为合约是隐式转换为地址的，
            // 所以这里的比较是可能的。
            if (msg.sender == address(creator))
                name = newName;
        }

        function transfer(address newOwner) public {
            // 只有当前所有者才能传送权证
            if (msg.sender != owner) return;
            // 我们还想确认 creator 是否权证转移是正常操作。
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
            // 创建一个新的权证合约并且返回它的地址。
            // 从 JavaScript 方面来说，返回类型是简单的 `address` 类型，这是因为
            // 这是在 ABI 中最接近的类型。
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

由于 Solidity 有两种函数调用（内部调用不会产生实际的 EVM 调用（也称为
一个“消息调用”）和外部调用），有四种函数可见性类型和状态变量。

函数可以指定为 ``external`` ，``public`` ，``internal`` 或者 ``private`` ，
默认情况下函数类型为 ``public`` 。对于状态变量，不能设置为 ``external`` ，默认
是 ``internal`` 。

``external`` ：
    外部函数作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。一个外部函数
    ``f`` 不能从内部调用（比如 ``f`` 不起作用，但 ``this.f()`` 可以）。
     当收到大量数据的时候，外部函数有时候会更有效率。

``public`` ：
    公共函数是合约接口的一部分，可以在内部或通过消息调用。对于公共状态变量，
    会自动生成一个 getter 函数（见下面）。

``internal`` ：
    这些函数和状态变量只能是内部访问（即从当前合约内部或从它派生的合约访问），不使用 ``this`` 调用。

``private`` ：
    私有函数和状态变量仅在当前定义它们的合约中使用，并且不能被派生合约使用。

.. note::
    合约中的所有内容对外部观察者都是可见的。设置一些 ``private`` 类型只能阻止其他合约访问和修改这些信息，
    但是对于区块链外的整个世界它仍然是可见的。

可见性的标识符的定义位置，对于状态变量来说是在类型后面，对于函数是在参数列表和返回关键字中间。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }

在下面的例子中，``D`` 可以调用 ``c.getData（）`` 来获取 ``data`` 的值，但不能调用 ``f`` 。
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

编译器自动为所有 **公有** 状态变量创建 getter 函数。 对于下面给出的合约，编译器会生成一个名为 ``data``
的函数，该函数不会接收任何参数并返回一个 ``uint`` ，即状态变量 ``data`` 的值。 可以在声明时完成状态
变量的初始化。

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

getter 函数具有外部可见性。 如果在内部访问 getter（即没有 ``this.`` ），它被认为一个状态变量。 如果
它是外部访问的（即用 ``this.`` ），它被认为为一个函数。

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

这将会生成以下形式的函数::

    function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
        a = data[arg1][arg2][arg3].a;
        b = data[arg1][arg2][arg3].b;
    }

请注意，因为没有好的方法来提供映射的键，所以结构中的映射被省略。

.. index:: ! function;modifier

.. _modifiers:

******************
函数修改器
******************

使用修改器可以轻松改变函数的行为。 例如，它们可以在执行该功能之前自动检查条件。 修改器是合约的可继承属性，
并可能被派生合约覆盖。

::

    pragma solidity ^0.4.11;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;

        // 这个合约只定义一个修改器，但并未使用： 它将会在派生合约中用到。
        // 将函数体插入到特殊符号 `_;` 出现的位置。
        // 这意味着如果是 owner 调用这个函数，则函数会被执行，否则则会抛出异常。

        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }
    }

    contract mortal is owned {
        // 这个合约从 `owned` 继承了 `onlyOwner` 修饰符，并并将其应用于 `close` 函数，
        // 只有存储的拥有者调用 `close` 函数，才会生效。

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

        // 重要的是这里也应该提供 `payable` 关键字，否则函数会自动拒绝所有发送给它的以太币。
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

        // 这个函数受互斥量保护，这意味着 `msg.sender.call` 中的可重入调用不能再次调用  `f` 。
        // `return 7` 语句指定返回值为7，但仍然是在修改器中执行语句 `locked = false` 。

        function f() public noReentrancy returns (uint) {
            require(msg.sender.call());
            return 7;
        }
    }

如果同一个函数有多个修改器，它们之间以空格隔开，修饰器会依次检查执行。

.. warning::
    在早期的Solidity版本中，有修改器的函数， ``return`` 语句的行为表现不同。

从修改器或函数体的显式的 return 语句仅仅跳出当前的修改器和函数体。 返回变量会被赋值，但整个执行
逻辑会在前一个修改器后面定义的 "_" 后继续执行。

修改器的参数可以是任意表达式，在上下文中，所有的函数中引入的符号，在修改器中均可见。在修改器中
引入的符号在函数中不可见（可能被重载改变）。

.. index:: ! constant

************************
常量
************************

状态变量可以被声明为 ``constant`` 。这种情况下，必须在编译阶段将他们指定为常量。不允许任何访问 storage，区
块链数据（例如 ``now``, ``this.balance`` 或者 ``block.number``）或执行数据（ ``msg.gas`` ）
或调用外部合约。允许表达式可能会对内存分配产生副作用，但不允许可能会对其他内存对象产生
副作用。 允许内置的函数，比如 ``keccak256``，``sha256``，``ripemd160``，``ecrecover``，``addmod``
和 ``mulmod`` （即使他们确实调用外部合约）。

允许内存分配，从而带来可能的副作用的原因是因为这将允许构建复杂的对象，比如，查找表。
此功能尚未完全可用。

编译器不会为这些变量预留存储，每个使用的常量都会被对应的常量表达式所替换（也许优化器会直接替换为常量表达式的结果值）

不是所有的类型都支持常量，当前支持的仅有值类型和字符串。

::

    pragma solidity ^0.4.0;

    contract C {
        uint constant x = 32**22 + 8;
        string constant text = "abc";
        bytes32 constant myHash = keccak256("abc");
    }

.. index:: ! functions

.. _functions:

*********
函数
*********

.. index:: ! view function, function;view

.. _view-functions:

View 函数
==============

可以将函数声明为 ``view`` 类型，这种情况下要保证不修改状态变量。

下面的语句被认为是修改状态：

#. 写状态变量.
#. :ref:`发送事件 <events>`。
#. :ref:`创建其它合约 <creating-contracts>`。
#. 使用 ``selfdestruct``。
#. 通过调用发送以太币。
#. 调用任何没有标记为 ``view`` 或者 ``pure`` 的函数.
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
  编译器没有强制 ``view`` 方法不能修改状态变量。

.. index:: ! pure function, function;pure

.. _pure-functions:

pure 函数
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
如果合约中没有与给定的函数标识符匹配的函数，将调用未命名函数（或者如果根本没有提供数据）。

此外，当合约收到以太币（没有任何数据），这个函数就会执行。 此外，为了接收以太币，fallback 函数
必须标记为 ``payable``。 如果不存在这样的函数，则合约不能通过常规交易接收以太币。

在这种上下文中，函数调用通常只消耗很少的 gas（准确地说，2300 个 gas ），所以重要的是使 fallback 函数尽可能便宜。
请注意，调用 fallback 函数的交易（而不是内部呼叫）所需的 gas 要高得多，因为每次交易都会额外收取 21000 gas 或更多的费用，
用于签名检查等事情。

特别的，以下操作会消耗比 fallback 函数更多的 gas：

- 写入存储
- 创建合约
- 调用消耗大量 gas 的外部函数
- 发送以太币


请确保您在部署合约之前彻底测试您的 fallback 函数，以确保执行成本低于 2300 个 gas 。

.. note::
    即使 fallback 函数不能有参数，仍然可以使用 ``msg.data`` 来获取随调用提供的任何有效负载。

.. warning::
    一个没有定义 fallback 函数的合约，直接接收以太币（没有函数调用，即使用 ``send`` 或 ``transfer``）会抛出一个异常，
    返还以太币（在Solidity v0.4.0之前行为会有所不同）。 所以如果你想让你的合约接收以太币，必须实现 fallback 函数。

.. warning::

    一个没有可支付的 fallback 函数的合约，可以作为 `coinbase transaction` （又名 `miner block reward` ）的接收者或者作为 ``selfdestruct`` 的目的地来接收以太币。
    对这种以太币转移合约不能作出反应，因此也不能拒绝它们。 这是 EVM 的设计选择，而且 Solidity 无法解决这个问题。
    这也意味着 ``this.balance`` 可以高于合约中实现的一些手工记帐的总和（即，在 fallback 函数中更新的计数器）。
::

    pragma solidity ^0.4.0;

    contract Test {
        // 发送到这个合约的所有消息都会调用此函数（因为该合约没有其它函数）。
        // 向这个合约发送以太币会导致异常，因为 fallback 函数没有  `payable` 修饰符
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

合约可以具有多个不同参数的同名函数。这也适用于继承函数。 以下示例展示了合约 ``A`` 中的重载函数 ``f``。

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

重载函数也存在于外部接口中。 如果两个外部可见函数接收的 Solidity 类型不同但是外部类型相同会导致错误。

::

    // This will not compile
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

通过将当前范围内的函数声明与函数调用中提供的参数相匹配，可以选择重载函数。如果所有参数都可以隐式地转换为预期类型，
则选择函数作为重载候选项。如果没有一个候选，解析失败。

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

调用  ``f(50)`` 会导致类型错误，因为 ``50`` 既可以被隐式转换为 ``uint8`` 也可以被隐式转换为  ``uint256``。
另一方面，调用 ``f(256)`` 则会解析为 ``f(uint256)`` 重载，因为 ``256`` 不能隐式转换为 ``uint8``。

.. index:: ! event

.. _events:

******
事件
******

通过事件可以方便地使用 EVM 日志记录工具，在一个dapp的接口中，它可以反过来 "调用" Javascript 的监听事件的回调。

事件在合约中可被继承。 当他们被调用时，会触发参数存储到交易的日志中 - 一种区块链中的特殊数据结构。
这些日志与地址相关联，被并入区块链中，只要区块可以访问就一直存在(至少 Frontier，Homestead 是这样，但 Serenity 也许不是这样)。
日志和事件在合约内不可直接被访问（甚至是创建日志的合约也不能访问）。

日志的 SPV 验证是可能的，如果一个外部的实体提供了这样验证的合约，它可以实际检查日志在区块链中是否存在。
但需要留意的是，由于合约中仅能访问最近的 256 个区块哈希，所以还需要提供区块头信息。

可以最多有三个参数被设置为 ``indexed``，来设置是否被索引：在用户界面上可以按索引参数的特定值来过滤。

如果数组（包括 ``string`` 和 ``bytes``）类型被标记为索引项，则它存储的 Keccak-256 哈希值作为主题索引。

除非你用 ``anonymous`` 说明符声明事件，否则事件签名的哈希值是主题之一。同时也意味着对于匿名事件无法通过名字来过滤。

所有非索引参数都将存储在日志的数据部分中。

.. note::
    索引参数不会自行存储。 你只能按值进行搜索，但不可能检索值本身。

::

    pragma solidity ^0.4.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // 任何对这个函数的调用（甚至是深度嵌套）都可以过滤被调用的 `Deposit` 来被 JavaScript API 检测到。

            Deposit(msg.sender, _id, msg.value);
        }
    }

在 JavaScript API 的用法如下：

::

    var abi = /* abi as generated by the compiler */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

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
``logi``  表示总共有带 ``i + 1`` 个 ``bytes32`` 类型的参数。其中第一个参数会被用来做为日志的数据部分，
其它的会做为主题。上面的事件调用可以以相同的方式执行。

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

长十六进制数等于 ``keccak256("Deposit(address,hash256,uint256)")``，即事件的签名。

了解事件的其他资源
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

总的继承系统与 `Python's <https://docs.python.org/3/tutorial/classes.html#inheritance>`_, 非常
相似，特别是多重继承方面。

下面的例子进行了详细的说明。

::

    pragma solidity ^0.4.16;

    contract owned {
        function owned() { owner = msg.sender; }
        address owner;
    }

    // 使用`is`从另一个合约派生。派生合约可以访问所有非私有成员，包括
    // 内部函数和状态变量。 不过，这些不可能通过 `this` 来外部访问。

    contract mortal is owned {
        function kill() {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // 这些抽象合约仅用于给编译器提供接口。注意函数没有函数体。如果一个合约没有实现所有函数，则只能用作接口。

    contract Config {
        function lookup(uint id) public returns (address adr);
    }

    contract NameReg {
        function register(bytes32 name) public;
        function unregister() public;
     }

    // 可以多重继承。 请注意，`owned` 也是 `mortal` 的基类，但只有一个 `owned` 实例（如C ++中的虚拟继承）。
    contract named is owned, mortal {
        function named(bytes32 name) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // 函数可以被另一个具有相同名称和相同数量/类型输入的函数重载。如果重载函数有不同类型的输出参数，会导致错误。
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

    // 如果构造函数接受参数，则需要在头文件中提供（或修改器调用样式）派生合约的构造函数（见下文）。
    contract PriceFeed is owned, mortal, named("GoldFeed") {
       function updateInfo(uint newInfo) public {
          if (msg.sender == owner) info = newInfo;
       }

       function get() public view returns(uint r) { return info; }

       uint info;
    }

请注意，我们调用 ``mortal.kill()`` 来调用父合约的销毁请求。这样做法是有问题的，就像
在下面的例子中看到::

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
        function kill() public { /* do cleanup 1 */ mortal.kill(); }
    }

    contract Base2 is mortal {
        function kill() public { /* do cleanup 2 */ mortal.kill(); }
    }

    contract Final is Base1, Base2 {
    }

调用 ``Final.kill()`` 会调用 最远的派生重载函数 ``Base2.kill``，但是会绕过 ``Base1.kill``，
基本上因为它甚至不知道 ``Base1``。解决这个问题的方法是使用 ``super``::

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
        function kill() public { /* do cleanup 1 */ super.kill(); }
    }


    contract Base2 is mortal {
        function kill() public { /* do cleanup 2 */ super.kill(); }
    }

    contract Final is Base1, Base2 {
    }

如果 ``Base2`` 调用 ``super`` 的函数，它不会简单在其基类合约上调用该函数。 相反，它
在最终的继承关系图谱的下一个基类合约中调用这个函数，所以它会调用 ``Base1.kill()`` （注意
最终的继承序列是 -- 从最远派生合约开始：Final, Base2, Base1, mortal, ownerd）。
在类中使用 super 调用的实际函数在当前类的上下文中是未知的，尽管它的类型是已知的。 这与普通的
虚拟方法查找类似。

.. index:: ! constructor

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

一种方法直接在继承列表中调用基类构造函数（``is Base(7)``）。另一种方法是像修改器使用方法一样，
作为派生合约构造函数定义头的一部分，（``Base(_y * _y)``)。
如果构造函数参数是常量并且定义或描述了合约的行为，使用第一种方法比较方便。如果基类构造函数的参数依赖于派生合约，那么
必须使用第二种方法。如果，像这个简单的例子一样，两个地方都用到了，优先使用修改器风格的参数。

.. index:: ! inheritance;multiple, ! linearization, ! C3 linearization

多重继承与线性化
======================================

编程语言实现多重继承需要解决几个问题。一个问题是 `钻石问题 <https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem>`_。
Solidity 借鉴了 Python 的方式并且使用 "`C3 线性化 <https://en.wikipedia.org/wiki/C3_linearization>`_" 强制将基类合约转换一个有向无环图(DAG)
的特定顺序。这导致了我们所希望的单调性，但是却禁止了某些继承图。特别是，基类在 ``is`` 后面的顺序很重要。在下面的代码中，Solidity 将会报错
"Linearization of inheritance graph impossible" 。

::

    // 下面代码编译出错

    pragma solidity ^0.4.0;

    contract X {}
    contract A is X {}
    contract C is A, X {}

原因是 ``C`` 请求 ``X`` 重写 ``A`` （因为定义的顺序是 ``A, X``），但是 ``A``本身要求重写
``X``，无法解决这种冲突。

一个指定基类合约的继承顺序的简单原则是从 "most base-like" 到 "most derived"。

继承有相同名字的不同类型成员
======================================================

一种错误情况是继承导致一个合约同时存在相同名字的修改器和函数时。另一种错误情况是继承导致的事件和修改器重名，函数和修改器重名。
有一种例外情况，状态变量的 getter 可以覆盖一个公有函数。

.. index:: ! contract;abstract, ! abstract contract

.. _abstract-contract:

******************
抽象合约
******************

合约函数可以缺少实现,如下例所示（请注意函数声明头由 ``;`` 结尾）
::
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

如果合约继承自抽象合约，并且不通过重写实现所有未实现的函数，那么它本身就是抽象的。

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

将来可能会解除这些限制。

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

库与合约类似，但其用途是在指定的地址仅部署一次，并且代码被使用 EVM 的 ``DELEGATECALL`` (Homestead 之前使用 ``CALLCODE`` 关键字)特性。
这意味着如果库函数被调用，它的代码在调用合约的上下文中执行，即 ``this`` 指向调用合约，特别是可以访问调用合约的存储。
因为一个合约是一个独立的代码块，它仅可以访问调用合约明确提供的状态变量（否则无法命名它们）。 如果不修改状态变量（即，如果是 ``view`` 或者 ``pure`` 函数），
库函数只能被直接调用（即不使用 ``DELEGATECALL`` 关键字），因为库被假定为无状态的。特别是，除非绕过 Solidity 类型系统，否则库不可能破坏。

使用库的合约，可以认为库是隐式的基类合约。虽然它们在继承关系中不会显式可见，但调用库函数与调用显式的基类合约十分类似（如果 ``L`` 是库的话，
 使用 ``L.f()`` 调用库函数）。此外，就像库是基类一样，对所有使用库的合约，库的 ``internal`` 函数都是可见的。当然，需要使用内部调用约定
 来调用内部函数，这意味着所有所有内部类型，内存类型都是通过引用而不是复制来传递。为了在 EVM 中实现这些，内部库函数的代码和从其中调用的所有
 函数都在编译阶段被拉取到调用合约中，然后使用一个 ``JUMP`` 调用来代替 ``DELEGATECALL``。

.. index:: using for, set

下面的示例说明如何使用库（但是请务必检查 :ref:`using for <using-for>` 有一个实现 set 更好的例子）。

::

    pragma solidity ^0.4.16;

    library Set {
      // 我们定义了一个新的结构体数据类型,用于在调用合约中保存数据。
      struct Data { mapping(uint => bool) flags; }

      // 注意第一个参数是 "storage reference" 类型，因此在调用中参数传递的只是它的存储地址而不是内容。
      // 这是库函数的一个特性。如果该函数可以被视为对象的方法，则习惯称第一个参数为 `self`。
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
            // 不需要库的特定实例就可以调用库函数，因为当前合约就是 "instance"。
            require(Set.insert(knownValues, value));
        }
        // 如果我们愿意，我们也可以在这个合约中直接访问 knownValues.flags
    }

当然，你不必按照这种方式去使用库：它们也可以在不定义结构数据类型的情况下使用。函数也不需要任何存储
引用参数，库可以出现在任何位置并且可以有多个存储引用参数。

调用 ``Set.contains``， ``Set.insert`` 和 ``Set.remove`` 都被编译为外部调用（ ``DELEGATECALL`` ）。
如果使用库，请注意实际执行的是外部函数调用。
``msg.sender``， ``msg.value`` 和 ``this`` 在调用中将保留它们的值，（在 Homestead 之前，
因为使用了 ``CALLCODE``，改变了 ``msg.sender`` 和 ``msg.value``)。

以下示例显示如何使用库的内存类型和内部函数来实现自定义类型，无需外部函数调用的开销：

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

由于编译器无法知道库的部署位置，链接器需要填入这些地址必的最终字节码（请参阅 :ref:`commandline-compiler` 以了解如何使用连接器的命令行工具）。 如果这些地址没有作为参数传递给编译器，
编译后的十六进制代码将包含 ``__Set______`` 形式的占位符（其中 ``Set`` 是库的名称）。可以手动填写地址来
替换库中十六进制编码的所有40个符号。

与合约相比，库的限制：

- 没有状态变量
- 不能够继承或被继承
- 不能接受以太币

（将来有可能会解除这些限制）

库的调用保护
=============================

正如介绍中所述，除调用 ``view`` 或者 ``pure`` 库函数之外，通过 ``CALL`` 而不是 ``DELEGATECALL``
或者 ``CALLCODE`` 的库的代码，将会恢复。

EVM 没有为合约提供检测是否使用 ``CALL`` 的直接方式，但是合约可以使用 ``ADDRESS`` 操作码找出正在运行
的“位置”。生成的代码通过比较这个地址和构造时的地址来确定调用模式。

更具体地说，库的运行时代码总是由 push 指令启用，它在编译时是 20 字节的零。当部署代码运行时，这个常数
被内存中的当前地址替换，修改后的代码存储在合约中。在运行时，这导致部署时地址是第一个被 push 到堆栈上的常数，
对于任何 non-view 和 non-pure 函数，调度器代码都将对比当前地址与这个常数是否一致。

.. index:: ! using for, library

.. _using-for:

*********
Using For
*********

指令 ``using A for B;`` 可用于附加库函数（从库 ``A``）到任何类型（``B``）。
调用对象将作为这些函数的第一个参数（像 Python 的 ``self`` 变量）。

``using A for *;`` 的效果是，库 ``A`` 中的函数被附加在任意的类型上。

在这两种情况下，所有函数，即使那些与对象类型不匹配的第一参数类型的函数，也被附加上了。
函数调用和重载解析时才会做类型检查。

``using A for B;`` 指令仅在当前作用域有效，仅限于在当前合约中，后续可能提升到全局范围。
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
        using Set for Set.Data; // 这是关键的修改
        Set.Data knownValues;

        function register(uint value) public {
            // 这里，Set.Data 类型的所有变量都有对应的成员函数。
            // 以下函数调用与 `Set.insert(knownValues, value)` 效果相同。
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

注意，所有库调用都是实际的 EVM 函数调用。这意味着如果传递内存或值类型，将执行一个拷贝副本，即使是 ``self`` 变量。
使用存储引用变量是唯一不会发生拷贝的情况。
