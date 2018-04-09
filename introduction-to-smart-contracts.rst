###############################
智能合约概述
###############################

.. _simple-smart-contract:

***********************
简单的智能合约
***********************

让我们先看一下最基本的例子。现在就算你都不理解也不要紧，后面我们会有更深入的讲解。

Storage
=======

::

    pragma solidity ^0.4.0;

    contract SimpleStorage {
        uint storedData;

        function set(uint x) public {
            storedData = x;
        }

        function get() public constant returns (uint) {
            return storedData;
        }
    }

第一行就是告诉大家源代码使用Solidity版本0.4.0写的，并且使用0.4.0以上版本运行也没问题（最高到0.5.0，但是不包含0.5.0）。这是为了确保合约不会在新的编译器版本中突然行为异常。关键字 ``pragma`` 的含义是，一般来说，pragmas（编译指令）是告知编译器如何处理源代码的指令的（例如，`pragma once <https://en.wikipedia.org/wiki/Pragma_once>`_）。

Solidity中合约的含义就是一组代码（它的 *函数*)和数据（它的 *状态*），它们位于以太坊区块链的一个特定地址上。 代码行``uint storedData;`` 声明一个类型为``uint``(256位无符号整数）的状态变量，叫做``storedData``。 你可以认为它是数据库里的一个位置，可以通过调用管理数据库代码的函数进行查询和变更。对于以太坊来说，上述的合约就是拥有合约（owing contract）。在这种情况下，函数``set``和``get``可以用来变更或取出变量的值。

要访问一个状态变量，并不需要像``this.``这样的前缀，虽然这是其他语言常见的做法。

这个合约只做了下面的这些事（由于以太坊构建的基础架构的原因）：允许任何人存储一个单独的数字，这个数字可以被世界上任何人访问，没有（可行的）办法阻止你发布这个数字。当然，任何人都可以再次调用``set``，传入不同的值，覆盖你的数字，但是这个数字会被存储在区块链的历史记录中。随后，我们会看到怎样加上访问控制，然后就只有你才能改变这个数字。

.. note::
    所有的标识符（合约名称，函数名称和变量名称）都只能使用ASCII字符集。UTF-8编码的数据可以用字符串变量的形式存储。

.. warning::
    小心使用Unicode文本，因为长得相像（甚至一样）的字符有不同的码点，因此会被编码成不同的字符数组。

.. index:: ! subcurrency

子货币（Subcurrency）例子
===================

下面的例子实现了一个加密货币的最简单的形式。确实可以无中生有地产生出币，但是只有创建合约的人才能做到。(实现一个不同的发行计划也不难）。而且，任何人都可以给其他人发币，不需要注册用户名和密码——所需要的只是以太坊密钥对。
::

    pragma solidity ^0.4.20; // should actually be 0.4.21

    contract Coin {
        // 关键字"public"让这些变量可以从外部读取
        address public minter;
        mapping (address => uint) public balances;

        // 轻客户端可以通过事件针对变化作出高效的反应
        event Sent(address from, address to, uint amount);

        // 这是构造函数，只有当合约创建时运行
        function Coin() public {
            minter = msg.sender;
        }

        function mint(address receiver, uint amount) public {
            if (msg.sender != minter) return;
            balances[receiver] += amount;
        }

        function send(address receiver, uint amount) public {
            if (balances[msg.sender] < amount) return;
            balances[msg.sender] -= amount;
            balances[receiver] += amount;
            emit Sent(msg.sender, receiver, amount);
        }
    }

这个合约引入了一些新的概念，让我们逐一解读。

 ``address public minter;`` 这一行声明了一个address类型的状态变量，可以被公开访问。``address``类型是一个160位的值，不允许任何算数操作。这种类型适合存储合约地址或外部人员的密钥对。关键字``public``自动生成一个函数，允许你在这个合约之外访问这个状态变量的当前值。没有这个关键字，其他的合约没有办法访问这个变量。由编译器生成的函数的代码大致如下所示
 ::

    function minter() returns (address) { return minter; }

当然，加一个和上面完全一样的函数是不能用的，因为我们会有同名的一个函数和一个变量，但是希望你能理解大意——编译器帮助你实现了。

.. index:: mapping

The next line, ``mapping (address => uint) public balances;`` also
creates a public state variable, but it is a more complex datatype.
The type maps addresses to unsigned integers.
Mappings can be seen as `hash tables <https://en.wikipedia.org/wiki/Hash_table>`_ which are
virtually initialized such that every possible key exists and is mapped to a
value whose byte-representation is all zeros. This analogy does not go
too far, though, as it is neither possible to obtain a list of all keys of
a mapping, nor a list of all values. So either keep in mind (or
better, keep a list or use a more advanced data type) what you
added to the mapping or use it in a context where this is not needed,
like this one. The :ref:`getter function<getter-functions>` created by the ``public`` keyword
is a bit more complex in this case. It roughly looks like the
following::

    function balances(address _account) public view returns (uint) {
        return balances[_account];
    }

As you see, you can use this function to easily query the balance of a
single account.

.. index:: event

The line ``event Sent(address from, address to, uint amount);`` declares
a so-called "event" which is emitted in the last line of the function
``send``. User interfaces (as well as server applications of course) can
listen for those events being emitted on the blockchain without much
cost. As soon as it is emitted, the listener will also receive the
arguments ``from``, ``to`` and ``amount``, which makes it easy to track
transactions. In order to listen for this event, you would use ::

    Coin.Sent().watch({}, '', function(error, result) {
        if (!error) {
            console.log("Coin transfer: " + result.args.amount +
                " coins were sent from " + result.args.from +
                " to " + result.args.to + ".");
            console.log("Balances now:\n" +
                "Sender: " + Coin.balances.call(result.args.from) +
                "Receiver: " + Coin.balances.call(result.args.to));
        }
    })

Note how the automatically generated function ``balances`` is called from
the user interface.

.. index:: coin

The special function ``Coin`` is the
constructor which is run during creation of the contract and
cannot be called afterwards. It permanently stores the address of the person creating the
contract: ``msg`` (together with ``tx`` and ``block``) is a magic global variable that
contains some properties which allow access to the blockchain. ``msg.sender`` is
always the address where the current (external) function call came from.

Finally, the functions that will actually end up with the contract and can be called
by users and contracts alike are ``mint`` and ``send``.
If ``mint`` is called by anyone except the account that created the contract,
nothing will happen. On the other hand, ``send`` can be used by anyone (who already
has some of these coins) to send coins to anyone else. Note that if you use
this contract to send coins to an address, you will not see anything when you
look at that address on a blockchain explorer, because the fact that you sent
coins and the changed balances are only stored in the data storage of this
particular coin contract. By the use of events it is relatively easy to create
a "blockchain explorer" that tracks transactions and balances of your new coin.

.. _blockchain-basics:

*****************
区块链基础
*****************

Blockchains as a concept are not too hard to understand for programmers. The reason is that
most of the complications (mining, `hashing <https://en.wikipedia.org/wiki/Cryptographic_hash_function>`_, `elliptic-curve cryptography <https://en.wikipedia.org/wiki/Elliptic_curve_cryptography>`_, `peer-to-peer networks <https://en.wikipedia.org/wiki/Peer-to-peer>`_, etc.)
are just there to provide a certain set of features and promises. Once you accept these
features as given, you do not have to worry about the underlying technology - or do you have
to know how Amazon's AWS works internally in order to use it?

.. index:: transaction

Transactions
============

A blockchain is a globally shared, transactional database.
This means that everyone can read entries in the database just by participating in the network.
If you want to change something in the database, you have to create a so-called transaction
which has to be accepted by all others.
The word transaction implies that the change you want to make (assume you want to change
two values at the same time) is either not done at all or completely applied. Furthermore,
while your transaction is applied to the database, no other transaction can alter it.

As an example, imagine a table that lists the balances of all accounts in an
electronic currency. If a transfer from one account to another is requested,
the transactional nature of the database ensures that if the amount is
subtracted from one account, it is always added to the other account. If due
to whatever reason, adding the amount to the target account is not possible,
the source account is also not modified.

Furthermore, a transaction is always cryptographically signed by the sender (creator).
This makes it straightforward to guard access to specific modifications of the
database. In the example of the electronic currency, a simple check ensures that
only the person holding the keys to the account can transfer money from it.

.. index:: ! block

Blocks
======

One major obstacle to overcome is what, in Bitcoin terms, is called a "double-spend attack":
What happens if two transactions exist in the network that both want to empty an account,
a so-called conflict?

The abstract answer to this is that you do not have to care. An order of the transactions
will be selected for you, the transactions will be bundled into what is called a "block"
and then they will be executed and distributed among all participating nodes.
If two transactions contradict each other, the one that ends up being second will
be rejected and not become part of the block.

These blocks form a linear sequence in time and that is where the word "blockchain"
derives from. Blocks are added to the chain in rather regular intervals - for
Ethereum this is roughly every 17 seconds.

As part of the "order selection mechanism" (which is called "mining") it may happen that
blocks are reverted from time to time, but only at the "tip" of the chain. The more
blocks that are added on top, the less likely it is. So it might be that your transactions
are reverted and even removed from the blockchain, but the longer you wait, the less
likely it will be.


.. _the-ethereum-virtual-machine:

.. index:: !evm, ! ethereum virtual machine

****************************
以太坊虚拟机
****************************

Overview
========

The Ethereum Virtual Machine or EVM is the runtime environment
for smart contracts in Ethereum. It is not only sandboxed but
actually completely isolated, which means that code running
inside the EVM has no access to network, filesystem or other processes.
Smart contracts even have limited access to other smart contracts.

.. index:: ! account, address, storage, balance

Accounts
========

There are two kinds of accounts in Ethereum which share the same
address space: **External accounts** that are controlled by
public-private key pairs (i.e. humans) and **contract accounts** which are
controlled by the code stored together with the account.

The address of an external account is determined from
the public key while the address of a contract is
determined at the time the contract is created
(it is derived from the creator address and the number
of transactions sent from that address, the so-called "nonce").

Regardless of whether or not the account stores code, the two types are
treated equally by the EVM.

Every account has a persistent key-value store mapping 256-bit words to 256-bit
words called **storage**.

Furthermore, every account has a **balance** in
Ether (in "Wei" to be exact) which can be modified by sending transactions that
include Ether.

.. index:: ! transaction

Transactions
============

A transaction is a message that is sent from one account to another
account (which might be the same or the special zero-account, see below).
It can include binary data (its payload) and Ether.

If the target account contains code, that code is executed and
the payload is provided as input data.

If the target account is the zero-account (the account with the
address ``0``), the transaction creates a **new contract**.
As already mentioned, the address of that contract is not
the zero address but an address derived from the sender and
its number of transactions sent (the "nonce"). The payload
of such a contract creation transaction is taken to be
EVM bytecode and executed. The output of this execution is
permanently stored as the code of the contract.
This means that in order to create a contract, you do not
send the actual code of the contract, but in fact code that
returns that code.

.. index:: ! gas, ! gas price

Gas
===

Upon creation, each transaction is charged with a certain amount of **gas**,
whose purpose is to limit the amount of work that is needed to execute
the transaction and to pay for this execution. While the EVM executes the
transaction, the gas is gradually depleted according to specific rules.

The **gas price** is a value set by the creator of the transaction, who
has to pay ``gas_price * gas`` up front from the sending account.
If some gas is left after the execution, it is refunded in the same way.

If the gas is used up at any point (i.e. it is negative),
an out-of-gas exception is triggered, which reverts all modifications
made to the state in the current call frame.

.. index:: ! storage, ! memory, ! stack

Storage, Memory and the Stack
=============================

Each account has a persistent memory area which is called **storage**.
Storage is a key-value store that maps 256-bit words to 256-bit words.
It is not possible to enumerate storage from within a contract
and it is comparatively costly to read and even more so, to modify
storage. A contract can neither read nor write to any storage apart
from its own.

The second memory area is called **memory**, of which a contract obtains
a freshly cleared instance for each message call. Memory is linear and can be
addressed at byte level, but reads are limited to a width of 256 bits, while writes
can be either 8 bits or 256 bits wide. Memory is expanded by a word (256-bit), when
accessing (either reading or writing) a previously untouched memory word (ie. any offset
within a word). At the time of expansion, the cost in gas must be paid. Memory is more
costly the larger it grows (it scales quadratically).

The EVM is not a register machine but a stack machine, so all
computations are performed on an area called the **stack**. It has a maximum size of
1024 elements and contains words of 256 bits. Access to the stack is
limited to the top end in the following way:
It is possible to copy one of
the topmost 16 elements to the top of the stack or swap the
topmost element with one of the 16 elements below it.
All other operations take the topmost two (or one, or more, depending on
the operation) elements from the stack and push the result onto the stack.
Of course it is possible to move stack elements to storage or memory,
but it is not possible to just access arbitrary elements deeper in the stack
without first removing the top of the stack.

.. index:: ! instruction

Instruction Set
===============

The instruction set of the EVM is kept minimal in order to avoid
incorrect implementations which could cause consensus problems.
All instructions operate on the basic data type, 256-bit words.
The usual arithmetic, bit, logical and comparison operations are present.
Conditional and unconditional jumps are possible. Furthermore,
contracts can access relevant properties of the current block
like its number and timestamp.

.. index:: ! message call, function;call

Message Calls
=============

Contracts can call other contracts or send Ether to non-contract
accounts by the means of message calls. Message calls are similar
to transactions, in that they have a source, a target, data payload,
Ether, gas and return data. In fact, every transaction consists of
a top-level message call which in turn can create further message calls.

A contract can decide how much of its remaining **gas** should be sent
with the inner message call and how much it wants to retain.
If an out-of-gas exception happens in the inner call (or any
other exception), this will be signalled by an error value put onto the stack.
In this case, only the gas sent together with the call is used up.
In Solidity, the calling contract causes a manual exception by default in
such situations, so that exceptions "bubble up" the call stack.

As already said, the called contract (which can be the same as the caller)
will receive a freshly cleared instance of memory and has access to the
call payload - which will be provided in a separate area called the **calldata**.
After it has finished execution, it can return data which will be stored at
a location in the caller's memory preallocated by the caller.

Calls are **limited** to a depth of 1024, which means that for more complex
operations, loops should be preferred over recursive calls.

.. index:: delegatecall, callcode, library

Delegatecall / Callcode and Libraries
=====================================

There exists a special variant of a message call, named **delegatecall**
which is identical to a message call apart from the fact that
the code at the target address is executed in the context of the calling
contract and ``msg.sender`` and ``msg.value`` do not change their values.

This means that a contract can dynamically load code from a different
address at runtime. Storage, current address and balance still
refer to the calling contract, only the code is taken from the called address.

This makes it possible to implement the "library" feature in Solidity:
Reusable library code that can be applied to a contract's storage, e.g. in
order to  implement a complex data structure.

.. index:: log

Logs
====

It is possible to store data in a specially indexed data structure
that maps all the way up to the block level. This feature called **logs**
is used by Solidity in order to implement **events**.
Contracts cannot access log data after it has been created, but they
can be efficiently accessed from outside the blockchain.
Since some part of the log data is stored in `bloom filters <https://en.wikipedia.org/wiki/Bloom_filter>`_, it is
possible to search for this data in an efficient and cryptographically
secure way, so network peers that do not download the whole blockchain
("light clients") can still find these logs.

.. index:: contract creation

Create
======

Contracts can even create other contracts using a special opcode (i.e.
they do not simply call the zero address). The only difference between
these **create calls** and normal message calls is that the payload data is
executed and the result stored as code and the caller / creator
receives the address of the new contract on the stack.

.. index:: selfdestruct

Self-destruct
=============

The only possibility that code is removed from the blockchain is
when a contract at that address performs the ``selfdestruct`` operation.
The remaining Ether stored at that address is sent to a designated
target and then the storage and code is removed from the state.

.. warning:: Even if a contract's code does not contain a call to ``selfdestruct``,
  it can still perform that operation using ``delegatecall`` or ``callcode``.

.. note:: The pruning of old contracts may or may not be implemented by Ethereum
  clients. Additionally, archive nodes could choose to keep the contract storage
  and code indefinitely.

.. note:: Currently **external accounts** cannot be removed from the state.
