.. index:: ! contract

##########
合约
##########

Solidity合约类似于面向对象语言中的类。合约中有用于数据持久化的状态变量，和可以操作他们的函数。
调用另一个合约实例的函数时，会执行一个EVM函数调用，这个操作会切换执行时的上下文，这样，前一个
合约的状态变量就不能访问了。

.. index:: ! contract;creation, constructor
.. 索引:: ! 合约；创建，构造函数

******************
创建合约
******************

可以通过以太坊交易 "从外部" 或从Solidity合约内部创建合约。
集成开发环境，像 `Remix <https://remix.ethereum.org/>`_, 使用用户界面元素流畅的创建合约。
在以太坊上编程创建合约最好使用JavaScript API `web3.js <https://github.com/ethereum/web3.js>`_。
截至今天，它有一个名为 `web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_ 的方法能够更容易的创建合约。

创建合约时，其构造函数（与合约同名的函数）执行一次。
构造函数是可选的。 只允许一个构造函数，这意味着不支持重载。

.. 索引:: 构造函数；参数

在内部，构造函数参数在合约代码之后通过 :ref:`ABI 编码 <ABI>`，但是如果你使用 ``web3.js``
则不必关心这个问题。

如果一个合约想要创建另一个合约，那么创建者必须知晓被创建合约的源代码(和二进制)。
这意味着不可能循环创建依赖项。

::

    pragma solidity ^0.4.16;

    contract OwnedToken {
        // TokenCreator是如下定义的合约类型.
        // 只要不用于创建新合约就可以引用它。
        TokenCreator creator;
        address owner;
        bytes32 name;

        // 这是注册creator和分配名称的构造函数。
        function OwnedToken(bytes32 _name) public {
            // 状态变量通过其名称访问，而不是通过例如this.owner.
            // 这也适用于函数，特别是在构造函数中，你只能那样调用他们（"内部调用"，
            // 因为合约本身还不存在。
            owner = msg.sender;
            // 从 `address` 到 `TokenCreator`，我们做显式的类型转换
            // 并且假定调用合约的类型是TokenCreator，没有真正的检查方法。
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
            // 我们还想确认creator是否权证转移是正常操作。
            // 请注意，这里调用了一个下面定义的合约中的函数
            // 如果调用失败（比如，由于gas不足），会立即停止执行。
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
            // 从JavaScript方面来说，返回类型是简单的`address`类型，这是因为
            // 这是在ABI中最接近的类型。
            return new OwnedToken(name);
        }

        function changeName(OwnedToken tokenAddress, bytes32 name)  public {
            // 同样，`tokenAddress`的外部类型也是`address`。
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

.. 索引:: ! 可见性，外部函数，公共函数/变量，私有函数/变量，内部函数

.. _可见性和getter函数：

**********************
可见性和getter函数
**********************

由于Solidity知道两种函数调用（内部调用不会产生实际的EVM调用（也称为
一个“消息呼叫”）和外部调用），有四种函数可见性类型和状态变量。

函数可以指定为``external``，``public``，``internal`` 或者``private``，
默认情况下函数类型为``public``。对于状态变量，不能设置为``external``，默认
是``internal``。

``external``：
    外部函数作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。一个外部函数
    ``f``不能从内部调用（比如``f``不起作用，但``this.f()``可以）。
     当收到大量数据的时候，外部函数有时候会更有效率。

``public``：
    公共函数是合约接口的一部分，可以在内部或通过消息调用。对于公共状态变量，
    会自动生成一个getter函数（见下面）。

``internal``：
    这些函数和状态变量只能是内部访问（即从当前合约内部或从它派生的合约访问），不使用“this”调用。


``private``:
    Private functions and state variables are only
    visible for the contract they are defined in and not in
    derived contracts.

``private``：
    私有函数和状态变量仅在当前定义它们的合约中使用，并且不能被派生合约使用。

.. 注意::
    合约中的所有内容对外部观察者都是可见的。设置一些 ``private``类型只能
    阻止其他合约访问和修改这些信息，但是对于区块链外的整个世界它仍然是可见的。

可见性的标识符的定义位置，对于状态变量来说是在类型后面，对于函数是在参数列表和返回关键字中间。

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }

在下面的例子中，``D``可以调用``c.getData（）``来获取``data``的值，但不能调用``f``。
合约``E``继承自``C``，因此可以调用``compute``。

::

    // 这不会编译

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

.. 索引:: ! getter;函数, ! 函数;getter
.. _getter函数:

Getter 函数
================

编译器自动为所有**公有**状态变量创建getter函数。 对于下面给出的合约，编译器会生成一个名为
``data``的函数，该函数不会接收任何参数并返回一个``uint``，即状态变量``data``的值。 可以在声
明时完成状态变量的初始化。

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

getter函数具有外部可见性。 如果在内部访问getter（即没有``this.``），它被认为一个状态变量。 如果
它是外部访问的（即用``this.``），它被认为为一个函数。

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
.. 索引:: ! 函数;修饰符

.. _修饰符:

******************
函数修饰符
******************

Modifiers can be used to easily change the behaviour of functions.  For example,
they can automatically check a condition prior to executing the function. Modifiers are
inheritable properties of contracts and may be overridden by derived contracts.

::

    pragma solidity ^0.4.11;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;

        // This contract only defines a modifier but does not use
        // it: it will be used in derived contracts.
        // The function body is inserted where the special symbol
        // `_;` in the definition of a modifier appears.
        // This means that if the owner calls this function, the
        // function is executed and otherwise, an exception is
        // thrown.
        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }
    }

    contract mortal is owned {
        // This contract inherits the `onlyOwner` modifier from
        // `owned` and applies it to the `close` function, which
        // causes that calls to `close` only have an effect if
        // they are made by the stored owner.
        function close() public onlyOwner {
            selfdestruct(owner);
        }
    }

    contract priced {
        // Modifiers can receive arguments:
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

        // It is important to also provide the
        // `payable` keyword here, otherwise the function will
        // automatically reject all Ether sent to it.
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

        /// This function is protected by a mutex, which means that
        /// reentrant calls from within `msg.sender.call` cannot call `f` again.
        /// The `return 7` statement assigns 7 to the return value but still
        /// executes the statement `locked = false` in the modifier.
        function f() public noReentrancy returns (uint) {
            require(msg.sender.call());
            return 7;
        }
    }

Multiple modifiers are applied to a function by specifying them in a
whitespace-separated list and are evaluated in the order presented.

.. warning::
    In an earlier version of Solidity, ``return`` statements in functions
    having modifiers behaved differently.

Explicit returns from a modifier or function body only leave the current
modifier or function body. Return variables are assigned and
control flow continues after the "_" in the preceding modifier.

Arbitrary expressions are allowed for modifier arguments and in this context,
all symbols visible from the function are visible in the modifier. Symbols
introduced in the modifier are not visible in the function (as they might
change by overriding).

.. index:: ! constant

************************
Constant State Variables
************************

State variables can be declared as ``constant``. In this case, they have to be
assigned from an expression which is a constant at compile time. Any expression
that accesses storage, blockchain data (e.g. ``now``, ``this.balance`` or
``block.number``) or
execution data (``msg.gas``) or make calls to external contracts are disallowed. Expressions
that might have a side-effect on memory allocation are allowed, but those that
might have a side-effect on other memory objects are not. The built-in functions
``keccak256``, ``sha256``, ``ripemd160``, ``ecrecover``, ``addmod`` and ``mulmod``
are allowed (even though they do call external contracts).

The reason behind allowing side-effects on the memory allocator is that it
should be possible to construct complex objects like e.g. lookup-tables.
This feature is not yet fully usable.

The compiler does not reserve a storage slot for these variables, and every occurrence is
replaced by the respective constant expression (which might be computed to a single value by the optimizer).

Not all types for constants are implemented at this time. The only supported types are
value types and strings.

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
Functions
*********

.. index:: ! view function, function;view

.. _view-functions:

View Functions
==============

Functions can be declared ``view`` in which case they promise not to modify the state.

The following statements are considered modifying the state:

#. Writing to state variables.
#. :ref:`Emitting events <events>`.
#. :ref:`Creating other contracts <creating-contracts>`.
#. Using ``selfdestruct``.
#. Sending Ether via calls.
#. Calling any function not marked ``view`` or ``pure``.
#. Using low-level calls.
#. Using inline assembly that contains certain opcodes.

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + now;
        }
    }

.. note::
  ``constant`` is an alias to ``view``.

.. note::
  Getter methods are marked ``view``.

.. warning::
  The compiler does not enforce yet that a ``view`` method is not modifying state.

.. index:: ! pure function, function;pure

.. _pure-functions:

Pure Functions
==============

Functions can be declared ``pure`` in which case they promise not to read from or modify the state.

In addition to the list of state modifying statements explained above, the following are considered reading from the state:

#. Reading from state variables.
#. Accessing ``this.balance`` or ``<address>.balance``.
#. Accessing any of the members of ``block``, ``tx``, ``msg`` (with the exception of ``msg.sig`` and ``msg.data``).
#. Calling any function not marked ``pure``.
#. Using inline assembly that contains certain opcodes.

::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }

.. warning::
  The compiler does not enforce yet that a ``pure`` method is not reading from the state.

.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fallback Function
=================

A contract can have exactly one unnamed function. This function cannot have
arguments and cannot return anything.
It is executed on a call to the contract if none of the other
functions match the given function identifier (or if no data was supplied at
all).

Furthermore, this function is executed whenever the contract receives plain
Ether (without data). Additionally, in order to receive Ether, the fallback function
must be marked ``payable``. If no such function exists, the contract cannot receive
Ether through regular transactions.

In such a context, there is usually very little gas available to the function call (to be precise, 2300 gas), so it is important to make fallback functions as cheap as possible. Note that the gas required by a transaction (as opposed to an internal call) that invokes the fallback function is much higher, because each transaction charges an additional amount of 21000 gas or more for things like signature checking.

In particular, the following operations will consume more gas than the stipend provided to a fallback function:

- Writing to storage
- Creating a contract
- Calling an external function which consumes a large amount of gas
- Sending Ether

Please ensure you test your fallback function thoroughly to ensure the execution cost is less than 2300 gas before deploying a contract.

.. note::
    Even though the fallback function cannot have arguments, one can still use ``msg.data`` to retrieve
    any payload supplied with the call.

.. warning::
    Contracts that receive Ether directly (without a function call, i.e. using ``send`` or ``transfer``)
    but do not define a fallback function
    throw an exception, sending back the Ether (this was different
    before Solidity v0.4.0). So if you want your contract to receive Ether,
    you have to implement a fallback function.

.. warning::
    A contract without a payable fallback function can receive Ether as a recipient of a `coinbase transaction` (aka `miner block reward`)
    or as a destination of a ``selfdestruct``.

    A contract cannot react to such Ether transfers and thus also cannot reject them. This is a design choice of the EVM and Solidity cannot work around it.

    It also means that ``this.balance`` can be higher than the sum of some manual accounting implemented in a contract (i.e. having a counter updated in the fallback function).

::

    pragma solidity ^0.4.0;

    contract Test {
        // This function is called for all messages sent to
        // this contract (there is no other function).
        // Sending Ether to this contract will cause an exception,
        // because the fallback function does not have the `payable`
        // modifier.
        function() public { x = 1; }
        uint x;
    }


    // This contract keeps all Ether sent to it with no way
    // to get it back.
    contract Sink {
        function() public payable { }
    }

    contract Caller {
        function callTest(Test test) public {
            test.call(0xabcdef01); // hash does not exist
            // results in test.x becoming == 1.

            // The following will not compile, but even
            // if someone sends ether to that contract,
            // the transaction will fail and reject the
            // Ether.
            //test.send(2 ether);
        }
    }

.. index:: ! overload

.. _overload-function:

Function Overloading
====================

A Contract can have multiple functions of the same name but with different arguments.
This also applies to inherited functions. The following example shows overloading of the
``f`` function in the scope of contract ``A``.

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

Overloaded functions are also present in the external interface. It is an error if two
externally visible functions differ by their Solidity types but not by their external types.

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


Both ``f`` function overloads above end up accepting the address type for the ABI although
they are considered different inside Solidity.

Overload resolution and Argument matching
-----------------------------------------

Overloaded functions are selected by matching the function declarations in the current scope
to the arguments supplied in the function call. Functions are selected as overload candidates
if all arguments can be implicitly converted to the expected types. If there is not exactly one
candidate, resolution fails.

.. note::
    Return parameters are not taken into account for overload resolution.

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

Calling ``f(50)`` would create a type error since ``250`` can be implicitly converted both to ``uint8``
and ``uint256`` types. On another hand ``f(256)`` would resolve to ``f(uint256)`` overload as ``256`` cannot be implicitly
converted to ``uint8``.

.. index:: ! event

.. _events:

******
Events
******

Events allow the convenient usage of the EVM logging facilities,
which in turn can be used to "call" JavaScript callbacks in the user interface
of a dapp, which listen for these events.

Events are
inheritable members of contracts. When they are called, they cause the
arguments to be stored in the transaction's log - a special data structure
in the blockchain. These logs are associated with the address of
the contract and will be incorporated into the blockchain
and stay there as long as a block is accessible (forever as of
Frontier and Homestead, but this might change with Serenity). Log and
event data is not accessible from within contracts (not even from
the contract that created them).

SPV proofs for logs are possible, so if an external entity supplies
a contract with such a proof, it can check that the log actually
exists inside the blockchain.  But be aware that block headers have to be supplied because
the contract can only see the last 256 block hashes.

Up to three parameters can
receive the attribute ``indexed`` which will cause the respective arguments
to be searched for: It is possible to filter for specific values of
indexed arguments in the user interface.

If arrays (including ``string`` and ``bytes``) are used as indexed arguments, the
Keccak-256 hash of it is stored as topic instead.

The hash of the signature of the event is one of the topics except if you
declared the event with ``anonymous`` specifier. This means that it is
not possible to filter for specific anonymous events by name.

All non-indexed arguments will be stored in the data part of the log.

.. note::
    Indexed arguments will not be stored themselves.  You can only
    search for the values, but it is impossible to retrieve the
    values themselves.

::

    pragma solidity ^0.4.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // Any call to this function (even deeply nested) can
            // be detected from the JavaScript API by filtering
            // for `Deposit` to be called.
            Deposit(msg.sender, _id, msg.value);
        }
    }

The use in the JavaScript API would be as follows:

::

    var abi = /* abi as generated by the compiler */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

    var event = clientReceipt.Deposit();

    // watch for changes
    event.watch(function(error, result){
        // result will contain various information
        // including the argumets given to the `Deposit`
        // call.
        if (!error)
            console.log(result);
    });

    // Or pass a callback to start watching immediately
    var event = clientReceipt.Deposit(function(error, result) {
        if (!error)
            console.log(result);
    });

.. index:: ! log

Low-Level Interface to Logs
===========================

It is also possible to access the low-level interface to the logging
mechanism via the functions ``log0``, ``log1``, ``log2``, ``log3`` and ``log4``.
``logi`` takes ``i + 1`` parameter of type ``bytes32``, where the first
argument will be used for the data part of the log and the others
as topics. The event call above can be performed in the same way as

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

where the long hexadecimal number is equal to
``keccak256("Deposit(address,hash256,uint256)")``, the signature of the event.

Additional Resources for Understanding Events
==============================================

- `Javascript documentation <https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events>`_
- `Example usage of events <https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol>`_
- `How to access them in js <https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js>`_

.. index:: ! inheritance, ! base class, ! contract;base, ! deriving

***********
Inheritance
***********

Solidity supports multiple inheritance by copying code including polymorphism.

All function calls are virtual, which means that the most derived function
is called, except when the contract name is explicitly given.

When a contract inherits from multiple contracts, only a single
contract is created on the blockchain, and the code from all the base contracts
is copied into the created contract.

The general inheritance system is very similar to
`Python's <https://docs.python.org/3/tutorial/classes.html#inheritance>`_,
especially concerning multiple inheritance.

Details are given in the following example.

::

    pragma solidity ^0.4.16;

    contract owned {
        function owned() { owner = msg.sender; }
        address owner;
    }

    // Use `is` to derive from another contract. Derived
    // contracts can access all non-private members including
    // internal functions and state variables. These cannot be
    // accessed externally via `this`, though.
    contract mortal is owned {
        function kill() {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // These abstract contracts are only provided to make the
    // interface known to the compiler. Note the function
    // without body. If a contract does not implement all
    // functions it can only be used as an interface.
    contract Config {
        function lookup(uint id) public returns (address adr);
    }

    contract NameReg {
        function register(bytes32 name) public;
        function unregister() public;
     }

    // Multiple inheritance is possible. Note that `owned` is
    // also a base class of `mortal`, yet there is only a single
    // instance of `owned` (as for virtual inheritance in C++).
    contract named is owned, mortal {
        function named(bytes32 name) {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // Functions can be overridden by another function with the same name and
        // the same number/types of inputs.  If the overriding function has different
        // types of output parameters, that causes an error.
        // Both local and message-based function calls take these overrides
        // into account.
        function kill() public {
            if (msg.sender == owner) {
                Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
                NameReg(config.lookup(1)).unregister();
                // It is still possible to call a specific
                // overridden function.
                mortal.kill();
            }
        }
    }

    // If a constructor takes an argument, it needs to be
    // provided in the header (or modifier-invocation-style at
    // the constructor of the derived contract (see below)).
    contract PriceFeed is owned, mortal, named("GoldFeed") {
       function updateInfo(uint newInfo) public {
          if (msg.sender == owner) info = newInfo;
       }

       function get() public view returns(uint r) { return info; }

       uint info;
    }

Note that above, we call ``mortal.kill()`` to "forward" the
destruction request. The way this is done is problematic, as
seen in the following example::

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

A call to ``Final.kill()`` will call ``Base2.kill`` as the most
derived override, but this function will bypass
``Base1.kill``, basically because it does not even know about
``Base1``.  The way around this is to use ``super``::

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

If ``Base2`` calls a function of ``super``, it does not simply
call this function on one of its base contracts.  Rather, it
calls this function on the next base contract in the final
inheritance graph, so it will call ``Base1.kill()`` (note that
the final inheritance sequence is -- starting with the most
derived contract: Final, Base2, Base1, mortal, owned).
The actual function that is called when using super is
not known in the context of the class where it is used,
although its type is known. This is similar for ordinary
virtual method lookup.

.. index:: ! base;constructor

Arguments for Base Constructors
===============================

Derived contracts need to provide all arguments needed for
the base constructors. This can be done in two ways::

    pragma solidity ^0.4.0;

    contract Base {
        uint x;
        function Base(uint _x) public { x = _x; }
    }

    contract Derived is Base(7) {
        function Derived(uint _y) Base(_y * _y) public {
        }
    }

One way is directly in the inheritance list (``is Base(7)``).  The other is in
the way a modifier would be invoked as part of the header of
the derived constructor (``Base(_y * _y)``). The first way to
do it is more convenient if the constructor argument is a
constant and defines the behaviour of the contract or
describes it. The second way has to be used if the
constructor arguments of the base depend on those of the
derived contract. If, as in this silly example, both places
are used, the modifier-style argument takes precedence.

.. index:: ! inheritance;multiple, ! linearization, ! C3 linearization

Multiple Inheritance and Linearization
======================================

Languages that allow multiple inheritance have to deal with
several problems.  One is the `Diamond Problem <https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem>`_.
Solidity follows the path of Python and uses "`C3 Linearization <https://en.wikipedia.org/wiki/C3_linearization>`_"
to force a specific order in the DAG of base classes. This
results in the desirable property of monotonicity but
disallows some inheritance graphs. Especially, the order in
which the base classes are given in the ``is`` directive is
important. In the following code, Solidity will give the
error "Linearization of inheritance graph impossible".

::

    // This will not compile

    pragma solidity ^0.4.0;

    contract X {}
    contract A is X {}
    contract C is A, X {}

The reason for this is that ``C`` requests ``X`` to override ``A``
(by specifying ``A, X`` in this order), but ``A`` itself
requests to override ``X``, which is a contradiction that
cannot be resolved.

A simple rule to remember is to specify the base classes in
the order from "most base-like" to "most derived".

Inheriting Different Kinds of Members of the Same Name
======================================================

When the inheritance results in a contract with a function and a modifier of the same name, it is considered as an error.
This error is produced also by an event and a modifier of the same name, and a function and an event of the same name.
As an exception, a state variable getter can override a public function.

.. index:: ! contract;abstract, ! abstract contract

******************
Abstract Contracts
******************

Contract functions can lack an implementation as in the following example (note that the function declaration header is terminated by ``;``)::

    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

Such contracts cannot be compiled (even if they contain
implemented functions alongside non-implemented functions),
but they can be used as base contracts::

    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }

If a contract inherits from an abstract contract and does not implement all non-implemented functions by overriding, it will itself be abstract.

.. index:: ! contract;interface, ! interface contract

**********
Interfaces
**********

Interfaces are similar to abstract contracts, but they cannot have any functions implemented. There are further restrictions:

#. Cannot inherit other contracts or interfaces.
#. Cannot define constructor.
#. Cannot define variables.
#. Cannot define structs.
#. Cannot define enums.

Some of these restrictions might be lifted in the future.

Interfaces are basically limited to what the Contract ABI can represent, and the conversion between the ABI and
an Interface should be possible without any information loss.

Interfaces are denoted by their own keyword:

::

    pragma solidity ^0.4.11;

    interface Token {
        function transfer(address recipient, uint amount) public;
    }

Contracts can inherit interfaces as they would inherit other contracts.

.. index:: ! library, callcode, delegatecall

.. _libraries:

************
Libraries
************

Libraries are similar to contracts, but their purpose is that they are deployed
only once at a specific address and their code is reused using the ``DELEGATECALL``
(``CALLCODE`` until Homestead)
feature of the EVM. This means that if library functions are called, their code
is executed in the context of the calling contract, i.e. ``this`` points to the
calling contract, and especially the storage from the calling contract can be
accessed. As a library is an isolated piece of source code, it can only access
state variables of the calling contract if they are explicitly supplied (it
would have no way to name them, otherwise). Library functions can only be
called directly (i.e. without the use of ``DELEGATECALL``) if they do not modify
the state (i.e. if they are ``view`` or ``pure`` functions),
because libraries are assumed to be stateless. In particular, it is
not possible to destroy a library unless Solidity's type system is circumvented.

Libraries can be seen as implicit base contracts of the contracts that use them.
They will not be explicitly visible in the inheritance hierarchy, but calls
to library functions look just like calls to functions of explicit base
contracts (``L.f()`` if ``L`` is the name of the library). Furthermore,
``internal`` functions of libraries are visible in all contracts, just as
if the library were a base contract. Of course, calls to internal functions
use the internal calling convention, which means that all internal types
can be passed and memory types will be passed by reference and not copied.
To realize this in the EVM, code of internal library functions
and all functions called from therein will at compile time be pulled into the calling
contract, and a regular ``JUMP`` call will be used instead of a ``DELEGATECALL``.

.. index:: using for, set

The following example illustrates how to use libraries (but
be sure to check out :ref:`using for <using-for>` for a
more advanced example to implement a set).

::

    pragma solidity ^0.4.16;

    library Set {
      // We define a new struct datatype that will be used to
      // hold its data in the calling contract.
      struct Data { mapping(uint => bool) flags; }

      // Note that the first parameter is of type "storage
      // reference" and thus only its storage address and not
      // its contents is passed as part of the call.  This is a
      // special feature of library functions.  It is idiomatic
      // to call the first parameter `self`, if the function can
      // be seen as a method of that object.
      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
              return false; // already there
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // not there
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
            // The library functions can be called without a
            // specific instance of the library, since the
            // "instance" will be the current contract.
            require(Set.insert(knownValues, value));
        }
        // In this contract, we can also directly access knownValues.flags, if we want.
    }

Of course, you do not have to follow this way to use
libraries: they can also be used without defining struct
data types. Functions also work without any storage
reference parameters, and they can have multiple storage reference
parameters and in any position.

The calls to ``Set.contains``, ``Set.insert`` and ``Set.remove``
are all compiled as calls (``DELEGATECALL``) to an external
contract/library. If you use libraries, take care that an
actual external function call is performed.
``msg.sender``, ``msg.value`` and ``this`` will retain their values
in this call, though (prior to Homestead, because of the use of ``CALLCODE``, ``msg.sender`` and
``msg.value`` changed, though).

The following example shows how to use memory types and
internal functions in libraries in order to implement
custom types without the overhead of external function calls:

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
                // too bad, we have to add a limb
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

As the compiler cannot know where the library will be
deployed at, these addresses have to be filled into the
final bytecode by a linker
(see :ref:`commandline-compiler` for how to use the
commandline compiler for linking). If the addresses are not
given as arguments to the compiler, the compiled hex code
will contain placeholders of the form ``__Set______`` (where
``Set`` is the name of the library). The address can be filled
manually by replacing all those 40 symbols by the hex
encoding of the address of the library contract.

Restrictions for libraries in comparison to contracts:

- No state variables
- Cannot inherit nor be inherited
- Cannot receive Ether

(These might be lifted at a later point.)

Call Protection For Libraries
=============================

As mentioned in the introduction, if a library's code is executed
using a ``CALL`` instead of a ``DELEGATECALL`` or ``CALLCODE``,
it will revert unless a ``view`` or ``pure`` function is called.

The EVM does not provide a direct way for a contract to detect
whether it was called using ``CALL`` or not, but a contract
can use the ``ADDRESS`` opcode to find out "where" it is
currently running. The generated code compares this address
to the address used at construction time to determine the mode
of calling.

More specifically, the runtime code of a library always starts
with a push instruction, which is a zero of 20 bytes at
compilation time. When the deploy code runs, this constant
is replaced in memory by the current address and this
modified code is stored in the contract. At runtime,
this causes the deploy time address to be the first
constant to be pushed onto the stack and the dispatcher
code compares the current address against this constant
for any non-view and non-pure function.

.. index:: ! using for, library

.. _using-for:

*********
Using For
*********

The directive ``using A for B;`` can be used to attach library
functions (from the library ``A``) to any type (``B``).
These functions will receive the object they are called on
as their first parameter (like the ``self`` variable in
Python).

The effect of ``using A for *;`` is that the functions from
the library ``A`` are attached to any type.

In both situations, all functions, even those where the
type of the first parameter does not match the type of
the object, are attached. The type is checked at the
point the function is called and function overload
resolution is performed.

The ``using A for B;`` directive is active for the current
scope, which is limited to a contract for now but will
be lifted to the global scope later, so that by including
a module, its data types including library functions are
available without having to add further code.

Let us rewrite the set example from the
:ref:`libraries` in this way::

    pragma solidity ^0.4.16;

    // This is the same code as before, just without comments
    library Set {
      struct Data { mapping(uint => bool) flags; }

      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
            return false; // already there
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // not there
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
        using Set for Set.Data; // this is the crucial change
        Set.Data knownValues;

        function register(uint value) public {
            // Here, all variables of type Set.Data have
            // corresponding member functions.
            // The following function call is identical to
            // `Set.insert(knownValues, value)`
            require(knownValues.insert(value));
        }
    }

It is also possible to extend elementary types in that way::

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
            // This performs the library function call
            uint index = data.indexOf(_old);
            if (index == uint(-1))
                data.push(_new);
            else
                data[index] = _new;
        }
    }

Note that all library calls are actual EVM function calls. This means that
if you pass memory or value types, a copy will be performed, even of the
``self`` variable. The only situation where no copy will be performed
is when storage reference variables are used.
