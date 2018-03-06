.. index:: ! contract

##########
合约
##########
Solidity的智能合约类似于面向对象语言中的“类”。包含会持续保存的状态变量数据以及能够改变它们值的函数。调用另外一个合约（实例）的函数就会执行一个EVM的函数调用，从而切换到这些变量不可以被防问的另外一个上下文环境。


.. index:: ! contract;creation, constructor

********
创建合约
********

合约可以通过以太币网络的交易从外部创建，也可以从Solidity内部创建。
集成开发环境，比如`Remix <https://remix.ethereum.org/>`_，可以让你从图形界面无缝创建一个合约。
以太币网络里面的程序化创建合约最好的办法是通过JavaScript API `web3.js <https://github.com/ethereum/web3.js>`_.
现在有一个方法调用来实现创建合约 `web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_。

当一个合约创建时，它的构造函数（一个和合约同名的函数）会被执行一次。
构造函数是可选的。且只允许一个构造函数，这就意味着构造函数是不可以重载的。

.. index:: constructor;arguments

从内部机制来说，构造函数的参数是附在合约代码本身之后通过:ref:`ABI encoded <ABI>`传递的，但如果你用"web3.js"，则用不着关心这个。
如果一个合约需要创建另一个合约，这个合约需要知道被创建的合约的代码（以及二进制码）。这也意味不可以循环创建。

::

    pragma solidity ^0.4.16;

    contract OwnedToken {
        //TokenCreator是一个合约类型，它的定义在后面
        //在没有使用它定义一个新的合约之前都可以引用它
        TokenCreator creator;
        address owner;
        bytes32 name;

        //这是一个构造函数用来登记合约创造者并且赋予一个名字
        function OwnedToken(bytes32 _name) public {
            //状态变量可通过它们的名字来访问
            //而不是用类似于 this.owner.这也适用于函数特别是构造函数，
            //你只能“内部的”去调用这些构造函数，因为这时候合约本身还不存在。
            owner = msg.sender;
            //我们用一个显式的类型转换把address换为TokenCreator,并且假设调用的合约就是TokenCreator
            //这里并没有一个真正的办法确认这一点
            creator = TokenCreator(msg.sender);
            name = _name;
        }

        function changeName(bytes32 newName) public {
            //只有合约创造者可以修改名字
            //比较是可以的，因为合约类型可以显式的转为地址类型
            if (msg.sender == address(creator))
                name = newName;
        }

        function transfer(address newOwner) public {
            //只有当前的拥有者才可以进行代币转帐
            if (msg.sender != owner) return;
            //我们还需要问合约创造者转帐是否可行。注意这里调用了一个在后面定义的函数，
            //如果调用失败（比如gas用完）,程序运行将立即停止。
            if (creator.isTokenTransferOK(owner, newOwner))
                owner = newOwner;
        }
    }

    contract TokenCreator {
        function createToken(bytes32 name)
           public
           returns (OwnedToken tokenAddress)
        {            
            // 创建一个新的代币合约并且返回它的地址.
            // 从JavaScriptp的角度来看，返回值就是一个简单的地址。  
            // 这是ABI里最常用的类型。
            return new OwnedToken(name);
        }

        function changeName(OwnedToken tokenAddress, bytes32 name)  public {
            // 这里外部类型又一次是个简单的地址类型。
            tokenAddress.changeName(name);
        }

        function isTokenTransferOK(address currentOwner, address newOwner)
            public
            view
            returns (bool ok)
        {
            // 这里随意增加了一些条件.
            address tokenAddress = msg.sender;
            return (keccak256(newOwner) & 0xff) == (bytes20(tokenAddress) & 0xff);
        }
    }

.. index:: ! visibility, external, public, private, internal

.. _visibility-and-getters:

***************
可见性和Getters
***************

由于Solidity知道两种函数调用：内部函数internal，它并不产生一个实际的EVM调用
（也称之为消息调用）和外部函数external（会产生EVM调用）。所以函数和状态变量
有总共有四种可见性。

函数能被指定为“external”，“public”，“internal”或者“private”，
这里缺省的是公共的。状态变量是不可以是外部的，缺省的是内部的。


``external``:
    外部函数是合约界面的一部分，也就意味着可以通过交易或者其它合约来调用。
    一个外部函数 ``f`` 不能从内部调用 (比如， ``f()`` 不可以,  ``this.f()`` 可以).
    外部函数在接受一些大的数据数组时，有时效率会更高。

``public``:
    公共的也是合约界面的一部分，即可以在内部调用也可以通过消息调用。
    对于一个公共的状态变量来说，一个用于查询的getter函数会被自动创建（见下面）。

``internal``:
    这些函数和状态变量只能从内部防问（比如从当前合约或者从它派生合约里调用）。
    这种情况不需要使用 ``this``。

``private``:
    私人函数和状态变量只能从定义它们的合约内部可，派生合约则不可以防问。

..note::
    合约内的任何东西对一个外部观察者来说都是可见的，把某个东西标成 ``private``。
    只是防止其它的合约来防问和修改。但对区块链以外的世界来说它仍然是可见的（译注：通过察看区块链数据）。

可见性标识符位于状态变量类型之后，以及函数的参数列表和返回参数之间。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }

在下面这个例子里, ``D``, 可以调用 ``c.getData()`` 来读取存在状态存贮区的``data``的值, 但是不能调用``f``。
合约 ``E`` 是由 ``C`` 派生, 所以可以调用 ``compute``。

::

    // This will not compile

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
            uint local = c.f(7); // error: member `f` is not visible
            c.setData(3);
            local = c.getData();
            local = c.compute(3, 5); // error: member `compute` is not visible
        }
    }

    contract E is C {
        function g() public {
            C c = new C();
            uint val = compute(3, 5); // access to internal member (from derived to parent contract)
        }
    }

.. index:: ! getter;function, ! function;getter
.. _getter function:

Getter 函数
===========

编译器会为所有的 **public** 状态变量自动创建一个getter函数。在下面这个合约里,
编译器会创建一个叫做 ``data`` 的函数，该函数不带任何参数，只返回一个 ``uint``,
也就是状态变量 ``data`` 的值. 状态变量的值可以在类型定义时初始化。

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

getter函数有外部可见性。一个符号在内部被防问时 (例如：不带 ``this.``),
它就被当作状态变量， 如果是从外部防问(例如：带有 ``this.`` ), 它就被当作函数.

::

    pragma solidity ^0.4.0;

    contract C {
        uint public data;
        function x() public {
            data = 3; // internal access
            uint val = this.data(); // external access
        }
    }

下一个例子要稍微复杂点
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

这个将会创建一个如下形式的函数::

    function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
        a = data[arg1][arg2][arg3].a;
        b = data[arg1][arg2][arg3].b;
    }

注意这个结构里mapping被忽略，因为没有一个好的办法去提供一个关键值key去mapping。

.. index:: ! function;modifier

.. _modifiers:

*********
函数修饰符
*********

修饰符（Modifiers）能够很容易地改变函数的行为，例如，可以在运行函数之前自动检查一个条件。
修饰符是合约的可继承属性，因此也可以被派生出的函数重载。

::

    pragma solidity ^0.4.11;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;

        // 这个合约只是定义了修饰符但没有用它
        // 它将在派生合约中使用
        // 这个函数体在修饰符里出现`_;`的地方被代入 
        // 这意味着如果是所有者(Owner)调用这个函数，函数就会运行，否则就会抛出一个意外（exception）错误
        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }
    }

    contract mortal is owned {
        // 这个合约从 `owned` 处继承了`onlyOwner` 修饰符
        // 并且应用到`close` 函数, 从而达到只有店主可以调用这个函数的效果
        function close() public onlyOwner {
            selfdestruct(owner);
        }
    }

    contract priced {
        // 修饰符也可以接受参数:
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

        // 这里使用了`payable` 这个关键词非常重要，否则这个函数将会自动拒绝所有送给它的以太币Ether
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

        /// 这个函数使用互拆功能来进行保护，这就意味着在 `msg.sender.call` 里不能够再次重入调用 `f`
        /// `return 7` 把 7 赋予返回值，不过也执行了修饰符里的 `locked = false` 语句。
        function f() public noReentrancy returns (uint) {
            require(msg.sender.call());
            return 7;
        }
    }

多个修饰符应用于一个函数时，要用空格隔开的列表方式并且按顺序实施。

.. warning::
    -  在Solidity早期版本中， 在带有修饰符的函数里，``return`` 语句的执行效果会有不同。

在修饰符或者函数体中显式使用 returns 仅仅是离开了当前的修饰符或者函数体。
返回值变量也会被赋值，但控制流在前一个修饰的 "_"之后继续运行。

修饰符的任意表达是允许的，在这种情况下，函数内的所有变量都对修饰符可见。
但修饰行内定义的符号在函数里是不可见的（因为它们可能被重载）。

.. index:: ! constant

************
常量状态变量
************

状态变量可以被定义为常量（ ``constant``）.在这种情况下，该变量必须被赋予一个编译时就确定的常量，
任何表达式用来防问存贮器，区块链数据（比如 ``now``, ``this.balance`` 或者
``block.number``) 或者运行数据(``msg.gas``) 或者调用外部合约都是不允许的。表达式如果有可能用到内存分配是可以允许的，
但如果对其它内存对象有影响的则不可以，内置函数``keccak256``, ``sha256``, ``ripemd160``,
``ecrecover``, ``addmod`` and ``mulmod`` 是允许的（虽然他们确实会调用外部合约）。

允许有内存外配的原因是这样就有可能构建复杂的对象比如，查询表lookup-tables，但这个功能目前还没实现。

编译器并不为这些常量保留存贮区域，这些常量每次都会被一个常数表达式所替代（也可能会被优化器计算出单一数值替代）。
并非所有类型的常量到目前为止都已实现，现在只支持值类型和字符串。

::

    pragma solidity ^0.4.0;

    contract C {
        uint constant x = 32**22 + 8;
        string constant text = "abc";
        bytes32 constant myHash = keccak256("abc");
    }

.. index:: ! functions

.. _functions:

****
函数
****

.. index:: ! view function, function;view

.. _view-functions:

View 函数
==============

函数可以定义为 ``view`` 这种情况下函数可以确保不会修改状态。

下列语句可以被认为是修改了状态：

#. 写入到状态变量 。
#. 发出一个事件 :ref:`Emitting events <events>`。
#. 创建另一个合约 :ref:`Creating other contracts <creating-contracts>`。
#. 使用自我销毁 ``selfdestruct``。
#. 通过调用来发送以太币。
#. 调用任何没有标为 ``view`` 或 ``pure``的函数。
#. 使用低级调用。
#. 使用包含特定操作码的嵌入式汇编。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + now;
        }
    }

.. note::
  ``constant`` 是 ``view`` 的一个别名。

.. note::
  Getter 方法被标示为 ``view``。

.. warning::
  编译器目前为止并不强制 ``view`` 方法不要去修改状态。

.. index:: ! pure function, function;pure

.. _pure-functions:

纯函数 pure function
==============

函数可以被定义为纯 ``pure`` 的，代表函数不会读也不会修改状态。

在上述会修改状态的语句列表之外，下列语句会被认为会读取状态。
#. 读取状态变量 Reading from state variables。
#. 访问 ``this.balance`` or ``<address>.balance``。
#. 访问 ``block``, ``tx``, ``msg`` 的任何成员(除了 ``msg.sig`` and ``msg.data``)。
#. 调用任何没有标示为 ``pure``的函数。
#. 使用包含特定代码的嵌入式汇编。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }

.. warning::
  编译器到目前为止并不强制 ``pure`` 方法不去读状态。

.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fallback 回调函数
=================

一个合约可以有最多一个的无名函数，这个函数不能有任何参数也不能返回任何东西。
当一个合约被调用时但其函数标识符和它的函数没有一个是匹配的（或者压根就没提供任何数据）。

还有，如果一个合约收到一个纯粹的以太币转账交易（不带数据），为了接收这些以太币，fallback函数必须被标为 ``payable``。
如果没有这样的函数存在，合约就不能接受常规的以太币转帐交易。

在这种情况下，通常只有很少的gas可以用于函数调用（准确地说是2300 gas），所以让fallback函数花得尽可能少是非常重要的。
要意识到由fallback调用的一个交易（和一个内部调用相比较）所需要的gas要高得多，因为一个交易要被收取21000 gas来用于签名验证等方面。

特别的，下列操作会消耗的gas比给fallback函数配备的gas更多:

- 写一个存贮 Writing to storage
- 创建一个合约 Creating a contract
- 调用一个消耗大量gas的外部函数 Calling an external function which consumes a large amount of gas
- 发送以太币 Sending Ether

在部署一个合约前请确认你彻底测试你的fallback，确保运行成本少于2300 gas。

.. note::
    虽然fallback 函数不能有参数，但它还是可以用 ``msg.data`` 来读取这个调用中带的 payload 。

.. warning::
    直接接收以太币的合约(没使用函数调用比如： ``send`` or ``transfer``)，
    但并没有定义fallback函数的会抛出一个例外错误（exception）,并将以太币送回 (这一点在Solidity v0.4.0之前有所不同)。
    所以如果你想让你的合约接收以太币，你就必须实现一个 fallback 函数。

.. warning::
    一个不带有payable fallback函数的合约可以作为 `coinbase transaction` (又叫 `挖矿奖励`)的
    接受方的方式来接受以太币或者作为一个自我毁灭 ``selfdestruct``的目的方。
  
    对于这种交易，合约即不能和它互动也不能拒绝。这是EVM和Solidity设计的选择使得其不能绕过这个问题。
    
    这也意味着这 ``this.balance`` 可能比合约里手动计算出来的余额要高（例如： 在回调函数里有一个计数器来计算余额）。
    
::

    pragma solidity ^0.4.0;

    contract Test {
        // 这个函数在合约收到任何消息时都会被调用
        // （这个合约没有其它函数）。
        // 发送以太币到这个合约会引起例外错误，
        // 因为这个回调函数没有“payable”修饰符
        function() public { x = 1; }
        uint x;
    }


    // 这个合约接受所有送给它的以太币，并且没有任何办法可以退回去。
    contract Sink {
        function() public payable { }
    }

    contract Caller {
        function callTest(Test test) public {
            test.call(0xabcdef01); // 这个哈希不存在
            // 结果就是 test.x 变成 == 1。

            // 下面这句不会编译，但如果有人发送以太币
            // 到那个合约，交易就会失败，发送的以太币也会被拒收。
            //test.send(2 ether);
        }
    }

.. index:: ! overload

.. _overload-function:

函数重载
========

一个合约可以有多个同名但参数不同的函数，这也适用于继承函数。下面这个例子给出在合约 ``A`` 范围内重载 ``f`` 。

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

重载函数也表现在外部接口。如果两个外部可见函数有不同的Solidity类型但外部类型相同则会出现一个错误。::

    // 这个不会被编译
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


上述两个重载的 ``f`` 函数虽然在Solidity内部被认为是不同的，最终在ABI中都被认为都是用来接受地址作为参数的，属同一个类型。

重载解析和参数匹配
-----------------

重载函数的选择是通过匹配当前范围内的函数定义和函数调用时所使用的参数来进行。函数的参数如果能够隐含匹配，
则该函数会被选为重载候选函数。如果没有一个重载候选函数，则解析失败。

.. note::
    返回参数并不参与重载解析。

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

调用 ``f(50)`` 会生成一个类型错误因为 ``250`` 能被同时隐含地转换为 ``uint8``和
``uint256`` 类型. 另一方面 ``f(256)`` 会被认为指定用 ``f(uint256)`` 重载，因为 ``256`` 不能够隐含地转为 ``uint8``。

.. index:: ! event

.. _events:

****
事件
****

事件允许一种方便的方法来使用 EVM 日志功能，且可以用来“调用”一个dapp用户接口里列出来的JavaScript的回调函数。

事件是合约里的可继承成员。当调用时，会生成一些参数并且存在交易的日志里 - 区块链里的一个特别的数据结构。
这些日志是和合约的地址相对应并且被关联进区块链，只要这个区块是可防问的，这些数据就会存在那里。
（在Frontier 和 Homestead里是一直存在的，但在Serenity里可能会被改掉）。在合约内部是不能防问日志和事件数据的
（就算是创建这些数据的合约也不能）。

SPV 验证日志是可行的，因此一个外部实体提供一个合约加上一个证明，它能检查这个日志确实存在于这个区块链内部。
但是要注意，一定要有区块头，因为合约只能看到最后256个区块哈希。

最多可以接受三个参数作为 ``indexed`` ，然后用这些对应的参数进行搜索：在用户接口中还可以用索引参数的特定值来过滤搜索。

如果数组（包括 ``string`` 和 ``bytes``) 用于索引参数，会用存贮它们的Keccak-256来替代。

事件签名的哈希是其中一个标题，除非你用 ``anonymous``修饰符定义这个事件。这就意味着你不能用指定名字过滤一个无名事件。

所有非索引参数会被存在日志的数据部分。

.. note::
    索引参数本身不会被存贮。你只能用参数值来搜索，但不能读取值本身。

::

    pragma solidity ^0.4.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // 所有调用这个函数（甚至深层嵌套）都
            // 能被JavaScript 的 API 通过过滤
            // “Deposit” 的调用而检测到。
            Deposit(msg.sender, _id, msg.value);
        }
    }

下面这个例子给出 JavaScript API 的使用:

::

    var abi = /* abi 由编译器生成 */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* 地址 */);

    var event = clientReceipt.Deposit();

    // 观察变化
    event.watch(function(error, result){
        // 结果会包含各种信息，包括调用 “Deposit" 时的各个参数。
        if (!error)
            console.log(result);
    });

    // 或者传递一个回调函数来立即开始观察。
    var event = clientReceipt.Deposit(function(error, result) {
        if (!error)
            console.log(result);
    });

.. index:: ! log

日志的低层接口
=============

可以通过函数 ``log0``, ``log1``, ``log2``, ``log3`` and ``log4`` 来防问日志机制的低层接口。
``logi`` 用 ``i + 1``个 ``bytes32`` 类型的参数, 这里第一个参数会被用于日志的数据部分，其它参数作为标题。
上述事件调用可以用下面的方法同样完成。

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

这里的长十六进制数等于 ``keccak256("Deposit(address,hash256,uint256)")``, 也就是事件的签名。

理解事件的附加资源
================

- `Javascript 文档 <https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events>`_
- `事件使用例集 <https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol>`_
- `怎样在js中防问 <https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js>`_

.. index:: ! inheritance, ! base class, ! contract;base, ! deriving

****
继承
****

Solidity 通过包括多态性的代码拷贝来支持多重继承。

所有函数调用都是虚拟的，也主意味着派生函数会被调用，除非显式给定合约名称。

当一个合约从多个合约继承时，只有一个合约会在区块链中被创建，所有基类代码都会被拷贝到被创建的合约。

整体继承体系非常类似于 `Python's <https://docs.python.org/3/tutorial/classes.html#inheritance>`_，
特别是在涉及到多重继承时。
下面的例子给出细节。

::

    pragma solidity ^0.4.16;

    contract owned {
        function owned() { owner = msg.sender; }
        address owner;
    }

    // 使用 `is` 来继承其它合约. 派生合约能够防问不能在外部通过 ‘this'来防问的所有非私用成员以及内部函数和状态变量。
    
    contract mortal is owned {
        function kill() {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // 这个抽象类只是用于为编译器提供接口。注意这些函数没有函数体。
    // 如果一个合约在没有实现它的所有函数的情况下只能被用作为一个接口。
    contract Config {
        function lookup(uint id) public returns (address adr);
    }

    contract NameReg {
        function register(bytes32 name) public;
        function unregister() public;
     }

    // 多继承是可行的。注意 “owned” 是 “mortal” 的一个基类， 但这里还是只有一个 “owned” 的实例
    // （就像是C++的虚拟继承）。
    contract named is owned, mortal {
        function named(bytes32 name) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // 函数能够被另一个名称及输入参数的数量及类型相同的函数重载。如果重载函数有一个不同的输出参数就会引起一个错误。
        // 所有本地和基于消息的函数都能够重载。
        function kill() public {
            if (msg.sender == owner) {
                Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
                NameReg(config.lookup(1)).unregister();
                // 还是有可能调用一个特定的被重载函数。
                mortal.kill();
            }
        }
    }

    // 如果一个构造函数带有一个参数，那就必须在派生类的的头部（或者在派生类构造函数的修饰符调用中）
    // 提供这个参数（见下方）。
    contract PriceFeed is owned, mortal, named("GoldFeed") {
       function updateInfo(uint newInfo) public {
          if (msg.sender == owner) info = newInfo;
       }

       function get() public view returns(uint r) { return info; }

       uint info;
    }

注意在上面这个例子中，我们调用 ``mortal.kill()`` 去传递这个析构请求，这么做是有问题的，请看下面这个例子::

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

对 ``Final.kill()`` 的调用将会调用最近派生父类的 ``Base2.kill`` 的调用，但这个函数会旁路
``Base1.kill``，主要是因为它根本就不知道 ``Base1``.  解决这个的方法是用 ``super``::

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

如果 ``Base2`` 调用一个 ``super`` 函数，它不是简单的调用其中一个基类的对应函数，实际上它调用了
在继承体系里挨着它的下一个基类的对应函数。因此，这里它会调用 ``Base1.kill()`` （注意实际的继承序列
--从最近的基类合约开始：Final，Base2，Base1，mortal，owned）。当一个类在使用super调用函数的上下文里，
它虽然知道函数的类型，但并不清楚实际那个函数被调用。这类似于一般的虚拟方法查找。

.. index:: ! base;constructor

基类构造参数
===========

派生类需要提供基类构造函数的参数，可以用下列两种方法::

    pragma solidity ^0.4.0;

    contract Base {
        uint x;
        function Base(uint _x) public { x = _x; }
    }

    contract Derived is Base(7) {
        function Derived(uint _y) Base(_y * _y) public {
        }
    }

一个方法是直接放在继承列表里（``is Base(7)``）。另一个方法是将其作为一个构造函数头部的一部分并会被调用的修饰符。
（``Base(_y * _y)``）。如果构造函数参数是一个常数且用来定义合约的行为或者用来描述合约，则第一种方法更方便一些。
如果基类构造函数的参数依赖于这些派生类，则只能用第二种方法。如果同时使用两个方法，则使用修饰符传递参数的方法会优先执行。

.. index:: ! inheritance;multiple, ! linearization, ! C3 linearization

多继承和线性化
=============

允许多继承的语言必须面对一些严重的问题。  一个是 `钻石问题 <https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem>`_.
Solidity沿用了Python的方法并且用 "`C3 线性化 <https://en.wikipedia.org/wiki/C3_linearization>`_"去强制使用一个基类DAG（和向无环图）
的特定顺序。这样会有一个一致顺序，但有时会导致一些继承图是不允许的。基类在 ``is`` 指示符后的顺序是尤其重要的。在下面这个例子里，
Solidity会给出一个继承图线性化错误。

::

    // 这个不能被编译

    pragma solidity ^0.4.0;

    contract X {}
    contract A is X {}
    contract C is A, X {}

错误的原因是 ``C`` 要求 ``X`` 去重载 ``A`` （通过指定 ``A， X`` 这样的顺序），但是 ``A`` 自身又要去继承 ``X``，这就会形成一个无法解决的矛盾。

一个简装的规则是记住基类的顺序是从“最远基类”到“最近基类”的顺序。

继承同名的不同成员
================

当继承导致一个合约里有同名的函数和修饰符时，会被当作一个错误。事件和修饰符同名，函数和事件同名都会导致这个错误。
例外的情况是，一个状态变量的getter可以重载一个public函数。

.. index:: ! contract;abstract, ! abstract contract

*******
抽象合约
*******

下面这个例子中，合约函数可以不含具体的实现方法（注意函数定义头是以 ``;``结尾的）::

    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

这样的合约不能被编译（就算它包含其它有具体实现方法的函数），但是他们可以被用作基类。

    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }

如果一个合约从抽象合约继承并且没有实际实现上述的空函数，那么它自身也算是抽象合约。

.. index:: ! contract;interface, ! interface contract

****
接口
****

接口类似于抽象合约，但是他们不能包含函数的具体实现。还有一些具体限制：

#. 不能继承其它合约或接口。
#. 不能定义构造函数。
#. 不能定义变量。
#. 不能定义结构。
#. 不能定义枚举。

其中的一些限制将来有可能会取消。

接口基本上限制于一个合约的ABI所能代表的内容，在ABI和接口之间进行转换时应该能够做到不掉失任何信息。

接口有自己的关键词：

::

    pragma solidity ^0.4.11;

    interface Token {
        function transfer(address recipient, uint amount) public;
    }

合约可以像继承其它合约一样继承接口。

.. index:: ! library, callcode, delegatecall

.. _libraries:

**
库
**

库类似于合约，但它们的目的是库存只会在一个特定地址被部署一次，并且库代码通过使用EVM中的 ``DELEGATECALL`` 
来实现重用。（Homestead之前用``CALLCODE``）。这就意味着库函数被调用时，它们的代码是在调用它们的合约上下文里
执行的。比如 ``this`` 指向了调用合约，而且能够防问调用合约的存贮空间。库是一段被隔离的源代码，它只能在被显式
提供的情况下防问调用合约里的状态变量（否则根本无法命名）。库函数在不修改状态变量的情况下（例如是 ``view`` 
或 ``pure`` 函数）只能被直接调用（例如不能使用``DELEGATECALL``）。因为库是假定没有状态的。除非Solidity
的类型系统被绕开，否则库是不可以被销毁的。

库可以被看作是调用它们的合约的隐含基类合约，它们在继承体系里不是显性可见的。但是调用库函数看起来就像调用一个显性
的基类函数（ ``L.f()`` 这里 ``L`` 是库名）. 更进一步，库里的 ``internal`` 函数是在合约里可见的。就像库是一个基类合约一样。
当然，调用一个内部函数是要使用内部函数调用转换的。也就意味着内部类型可以被传递，内存类型是通过引用而不是拷贝来传递的。
要在EVM中实现这一点，调用合约在编译时里就要包含内部库函数以及它们库自身调用的函数的代码，并且用 ``JUMP`` 来代替 ``DELEGATECALL``。

.. index:: using for, set

下面这个例子演示了如何使用库（请查阅一个更高级的方法去实现一个集合的例子 :ref:`using for <using-for>` ).

::

    pragma solidity ^0.4.16;

    library Set {
      //定义一个新的数据类型用于在调用合约里保存数据。
      struct Data { mapping(uint => bool) flags; }

      // 注意第一个参数是类型是 "storage
      // 引用" 因此在调用中被传递的只是存贮的地址而不是内容。这是库函数的一个特别之处。
      // 把第一个参数称之为 `self`是一种常见的做法，就好象是函数可以被看作是一个对象的方法。
      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
              return false; // 已经在里面。 
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // 不在里面。
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
            // 这个库函数可以在没有实例化的情况下被调用
            // 因为“实例”其实就是当前的合约。 
            require(Set.insert(knownValues, value));
        }
        // 在这个合约里，如果你愿意，也可以直接防问knownValues.flags。
    }

当然，你也不是一定要按这个方法去使用库：没有定义结构数据类型的情况下也可以用。函数也可以在没有任何存贮引用的情况下工作，
它们也可以在任何位置有多个存贮引用作为参数。

调用 ``Set.contains``, ``Set.insert`` and ``Set.remove``都被编译为调用外部合约/库。使用库时要注意，实际是执行一个外部调用。
在调用中，``msg.sender``, ``msg.value`` and ``this`` 会保持它们原来的值。虽然 （在Homestead之前, 因为用了 ``CALLCODE``,
``msg.sender`` 和 ``msg.value`` 会变化）。

下面这个例子示例了在库里如何在避免外部函数调用开销的情况下，使用内存类型和内部调用去实现一个定制类型。

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
                // 不好，我们必须增加一个limb
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

在编译器不知道库会被部署在什么地址的情况下，它们的地址在最终的二进制码（bytecode)中只能用一个链接来填充。
（参见 :ref:`commandline-compiler` 如何使用命行行编译器来链接）。
如果地址没有作为参数传给编译器，编译后的十六进制码会包含一个``__Set______``形式的占位符（这里 ``Set`` 是库名）。
实际地址可以通过用库合约地址的40个十六进制字符替换的方法手动填入。

Restrictions for libraries in comparison to contracts:
和合约相比库的限制：

- 没有状态变量
- 不能继承或被继承
- 不能接收以太币

（以有有可能会取消。）

库的调用保护
===========

在介绍部分提到过，除了在调用了一个 ``view`` 或 ``pure`` 函数外，如果一个库的代码是用 ``CALL`` 而不是用 `DELEGATECALL`` 
或者 ``CALLCODE``来调用的话，结果就会被复原。

EVM并没有提供一个直接的方法去检测是否用了 ``CALL`` 来调用，但是一个合约能用 ``ADDRESS`` 操作码来确定它当前在什么地方运行。
生成的代码比较当前（this）的地址和构造时的地址来确定调用的模式。

更详细的说，库的运行代码在编译时总是以一个为零的20字节push指令开头。当部署代码执行时。这个常数在内存中会被当前的地址所取代，
然后这段修改过的代码会被存在合约中。而在运行时，这个会导致部署地址是被第一个压进堆栈的常数，并且调度代码会为所有的non-view 
和 non-pure函数比较当前的地址和这个常数值。

.. index:: ! using for, library

.. _using-for:

*************
using for 指令
*************

指示符 ``using A for B;`` 能够把任何库函数（来自库 ``A``）关联至任何类型（``B``）。
这些函数会通过调用时的的第一个参数来接收对象。（就像Python中的``self``）。

``using A for *;`` 的效果是来自库A的函数关联到任意类型。
 
在以上两种情况下，所有的函数就算它们的第一个参数并不和一个对象的类型相匹配，也会被关联。类型检查是在函数被调用时才会进行，
然后执行函数重载方案。

``using A for B;`` 指示符只在当前范围内有效，当前只限于一个合约内但以后会提到全局范围，因为通过把一个模块包括进来，
它的数据类型还有库函数都可以不用增加任何代码而直接可用。

下面我们用这种方式来重写这个例子:ref:`libraries` ::

    pragma solidity ^0.4.16;

    // 和之前的代码一样，只是没有注释
    library Set {
      struct Data { mapping(uint => bool) flags; }

      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
            return false; // 已经在这里
            self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // 还没有
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
        using Set for Set.Data; // 这里是关键的不同
        Set.Data knownValues;

        function register(uint value) public {
            // 这里，所有Set.Data类型的变量都有对应的成员函数。
            // 下面这个函数调用等同于 `Set.insert(knownValues, value)`
            require(knownValues.insert(value));
        }
    }

这种方式也可以扩展到基础类型 ::

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
            // 这里执行库函数的调用
            uint index = data.indexOf(_old);
            if (index == uint(-1))
                data.push(_new);
            else
                data[index] = _new;
        }
    }

注意所有库的调用实际上是EVM函数调用。这主意味着如果你传递过去一个内存变量或值变量，就算用了``self`` 变量，
库函数也只会在拷贝上执行。只有在传递变量的存贮引用时，才不会在拷贝上运行。
