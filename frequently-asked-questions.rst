###########################
常见问题
###########################

这份清单最早是由 `fivedogit <mailto:fivedogit@gmail.com>`_ 收集整理的。


***************
基本问题
***************

合约范本
========

请参考由fivedogit收集整理的一些 `合约范本 <https://github.com/fivedogit/solidity-baby-steps/tree/master/contracts/>`_，另外请为Solidity的每一个特征建立一份 `测试合约 <https://github.com/ethereum/solidity/blob/develop/test/libsolidity/SolidityEndToEndTest.cpp>`_。

创建并发布一个最基本的能用的合约
================================

这是个最简单的例子： `greeter <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/05_greeter.sol>`_。

可以在特定的区块上进行操作吗？(比如发布一个合约或执行一笔交易)
==============================================================

鉴于交易数据的写入是由矿工决定的而不是由提交者决定的，谁也无法保证交易一定会发生在下一个或未来某一个特定的区块上。这个结论适用于功能调用/交易以及合约的创建。

如果你希望你的合约被定时调用，可以使用：
`alarm clock <http://www.ethereum-alarm-clock.com/>`_。

什么是交易的“有效载荷”？
========================

就是随交易一起发送的字节码“数据”。

存在反编译程序吗？
==================

除了 `Porosity <https://github.com/comaeio/porosity>`_ 有点接近之外，Solidity没有严格意义上的反编译程序。由于诸如变量名、注释、代码格式等会在编译过程中丢失，所以完全反编译回源代码是没有可能的。

很多区块链管理器都能将字节码分解为一系列操作码。

如果区块链上的合约会被第三方使用，那么最好将源代码一起进行发布。

创建一个可以被中止并退款的合约
==============================

首先，需要提醒一下：中止合约听起来是一个好主意，把垃圾打扫干净是个好习惯，但如上所述，合约是不会被真正清理干净的。更有甚者，被发送至已移除合约的以太币，会从此丢失。

如果想让合约不再可用，建议的做法是修改合约内在逻辑来使其 **失效** ，让所有功能调用都变为无效返回。这样就无法使用这份合约了，而且发送过去的以太币也会被自动退回。

现在正式回答这个问题：在构造器中，将creator赋值为``msg.sender``，并保存。然后调用``selfdestruct(creator);``来中止程序并进行退款。

`例子 <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/05_greeter.sol>`_

需要注意的是，如果你已经在合约顶部做了引用 ``import "mortal"`` 并且申明了 
``contract SomeContract is mortal { ...`` ，然后再在已存在此合约的编译器中进行编译（包含 `Remix <https://remix.ethereum.org/>`_），那么 ``kill()`` 就会自动执行。当一份合约被申明为"mortal"时，你可以仿照我的例子，使用
 ``contractname.kill.sendTransaction({from:eth.coinbase})`` 来中止它。


在合约中存储以太币
==================

诀窍是在合约中使用 ``{from:someaddress, value: web3.toWei(3,"ether")...}``

参考 `endowment_retriever.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/30_endowment_retriever.sol>`_ 。

使用非恒定方程（请求 ``sendTransaction`` ）来对合约中的变量进行递增
=================================================================

参考 `value_incrementer.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/20_value_incrementer.sol>`_ 。

让合约把费用返还给你（不使用 ``selfdestruct(...)`` ）
===================================================

这个例子展示了如何将费用从一份合约发送至一个地址。

参考 `endowment_retriever <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/30_endowment_retriever.sol>`_ 。

调用Solidity方法可以返回一个数组或字符串（``string``）吗？
==========================================================

可以。参考 `array_receiver_and_returner.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/60_array_receiver_and_returner.sol>`_ 。

What is problematic, though, is returning any variably-sized data (e.g. a
variably-sized array like ``uint[]``) from a fuction **called from within Solidity**.
This is a limitation of the EVM and will be solved with the next protocol update.
What is problematic, though, is returning any variably-sized data (e.g. a
variably-sized array like ``uint[]``) from a fuction **called from within Solidity**.
This is a limitation of the EVM and will be solved with the next protocol update.

Returning variably-sized data as part of an external transaction or call is fine.

Is it possible to in-line initialize an array like so: ``string[] myarray = ["a", "b"];``
=========================================================================================

Yes. However it should be noted that this currently only works with statically sized memory arrays. You can even create an inline memory
array in the return statement. Pretty cool, huh?

Example::

    pragma solidity ^0.4.16;

    contract C {
        function f() public pure returns (uint8[5]) {
            string[4] memory adaArr = ["This", "is", "an", "array"];
            return ([1, 2, 3, 4, 5]);
        }
    }

Can a contract function return a ``struct``?
============================================

Yes, but only in ``internal`` function calls.

If I return an ``enum``, I only get integer values in web3.js. How to get the named values?
===========================================================================================

Enums are not supported by the ABI, they are just supported by Solidity.
You have to do the mapping yourself for now, we might provide some help
later.

Can state variables be initialized in-line?
===========================================

Yes, this is possible for all types (even for structs). However, for arrays it
should be noted that you must declare them as static memory arrays.

Examples::

    pragma solidity ^0.4.0;

    contract C {
        struct S {
            uint a;
            uint b;
        }

        S public x = S(1, 2);
        string name = "Ada";
        string[4] adaArr = ["This", "is", "an", "array"];
    }

    contract D {
        C c = new C();
    }

How do structs work?
====================

See `struct_and_for_loop_tester.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/65_struct_and_for_loop_tester.sol>`_.

How do for loops work?
======================

Very similar to JavaScript. There is one point to watch out for, though:

If you use ``for (var i = 0; i < a.length; i ++) { a[i] = i; }``, then
the type of ``i`` will be inferred only from ``0``, whose type is ``uint8``.
This means that if ``a`` has more than ``255`` elements, your loop will
not terminate because ``i`` can only hold values up to ``255``.

Better use ``for (uint i = 0; i < a.length...``

See `struct_and_for_loop_tester.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/65_struct_and_for_loop_tester.sol>`_.

What are some examples of basic string manipulation (``substring``, ``indexOf``, ``charAt``, etc)?
==================================================================================================

There are some string utility functions at `stringUtils.sol <https://github.com/ethereum/dapp-bin/blob/master/library/stringUtils.sol>`_
which will be extended in the future. In addition, Arachnid has written `solidity-stringutils <https://github.com/Arachnid/solidity-stringutils>`_.

For now, if you want to modify a string (even when you only want to know its length),
you should always convert it to a ``bytes`` first::

    pragma solidity ^0.4.0;

    contract C {
        string s;

        function append(byte c) public {
            bytes(s).push(c);
        }

        function set(uint i, byte c) public {
            bytes(s)[i] = c;
        }
    }


Can I concatenate two strings?
==============================

You have to do it manually for now.

Why is the low-level function ``.call()`` less favorable than instantiating a contract with a variable (``ContractB b;``) and executing its functions (``b.doSomething();``)?
=============================================================================================================================================================================

If you use actual functions, the compiler will tell you if the types
or your arguments do not match, if the function does not exist
or is not visible and it will do the packing of the
arguments for you.

See `ping.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_ping.sol>`_ and
`pong.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_pong.sol>`_.

Is unused gas automatically refunded?
=====================================

Yes and it is immediate, i.e. done as part of the transaction.

When returning a value of say ``uint`` type, is it possible to return an ``undefined`` or "null"-like value?
============================================================================================================

This is not possible, because all types use up the full value range.

You have the option to ``throw`` on error, which will also revert the whole
transaction, which might be a good idea if you ran into an unexpected
situation.

If you do not want to throw, you can return a pair::

    pragma solidity ^0.4.16;

    contract C {
        uint[] counters;

        function getCounter(uint index)
            public
            view
            returns (uint counter, bool error) {
                if (index >= counters.length)
                    return (0, true);
                else
                    return (counters[index], false);
        }

        function checkCounter(uint index) public view {
            var (counter, error) = getCounter(index);
            if (error) {
                // ...
            } else {
                // ...
            }
        }
    }


Are comments included with deployed contracts and do they increase deployment gas?
==================================================================================

No, everything that is not needed for execution is removed during compilation.
This includes, among others, comments, variable names and type names.

What happens if you send ether along with a function call to a contract?
========================================================================

It gets added to the total balance of the contract, just like when you send ether when creating a contract.
You can only send ether along to a function that has the ``payable`` modifier,
otherwise an exception is thrown.

Is it possible to get a tx receipt for a transaction executed contract-to-contract?
===================================================================================

No, a function call from one contract to another does not create its own transaction,
you have to look in the overall transaction. This is also the reason why several
block explorer do not show Ether sent between contracts correctly.

What is the ``memory`` keyword? What does it do?
================================================

The Ethereum Virtual Machine has three areas where it can store items.

The first is "storage", where all the contract state variables reside.
Every contract has its own storage and it is persistent between function calls
and quite expensive to use.

The second is "memory", this is used to hold temporary values. It
is erased between (external) function calls and is cheaper to use.

The third one is the stack, which is used to hold small local variables.
It is almost free to use, but can only hold a limited amount of values.

For almost all types, you cannot specify where they should be stored, because
they are copied everytime they are used.

The types where the so-called storage location is important are structs
and arrays. If you e.g. pass such variables in function calls, their
data is not copied if it can stay in memory or stay in storage.
This means that you can modify their content in the called function
and these modifications will still be visible in the caller.

There are defaults for the storage location depending on which type
of variable it concerns:

* state variables are always in storage
* function arguments are in memory by default
* local variables of struct, array or mapping type reference storage by default
* local variables of value type (i.e. neither array, nor struct nor mapping) are stored in the stack

Example::

    pragma solidity ^0.4.0;

    contract C {
        uint[] data1;
        uint[] data2;

        function appendOne() public {
            append(data1);
        }

        function appendTwo() public {
            append(data2);
        }

        function append(uint[] storage d) internal {
            d.push(1);
        }
    }

The function ``append`` can work both on ``data1`` and ``data2`` and its modifications will be
stored permanently. If you remove the ``storage`` keyword, the default
is to use ``memory`` for function arguments. This has the effect that
at the point where ``append(data1)`` or ``append(data2)`` is called, an
independent copy of the state variable is created in memory and
``append`` operates on this copy (which does not support ``.push`` - but that
is another issue). The modifications to this independent copy do not
carry back to ``data1`` or ``data2``.

A common mistake is to declare a local variable and assume that it will
be created in memory, although it will be created in storage::

    /// THIS CONTRACT CONTAINS AN ERROR

    pragma solidity ^0.4.0;

    contract C {
        uint someVariable;
        uint[] data;

        function f() public {
            uint[] x;
            x.push(2);
            data = x;
        }
    }

The type of the local variable ``x`` is ``uint[] storage``, but since
storage is not dynamically allocated, it has to be assigned from
a state variable before it can be used. So no space in storage will be
allocated for ``x``, but instead it functions only as an alias for
a pre-existing variable in storage.

What will happen is that the compiler interprets ``x`` as a storage
pointer and will make it point to the storage slot ``0`` by default.
This has the effect that ``someVariable`` (which resides at storage
slot ``0``) is modified by ``x.push(2)``.

The correct way to do this is the following::

    pragma solidity ^0.4.0;

    contract C {
        uint someVariable;
        uint[] data;

        function f() public {
            uint[] x = data;
            x.push(2);
        }
    }

******************
高级问题
******************

How do you get a random number in a contract? (Implement a self-returning gambling contract.)
=============================================================================================

Getting randomness right is often the crucial part in a crypto project and
most failures result from bad random number generators.

If you do not want it to be safe, you build something similar to the `coin flipper <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/35_coin_flipper.sol>`_
but otherwise, rather use a contract that supplies randomness, like the `RANDAO <https://github.com/randao/randao>`_.

Get return value from non-constant function from another contract
=================================================================

The key point is that the calling contract needs to know about the function it intends to call.

See `ping.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_ping.sol>`_
and `pong.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_pong.sol>`_.

Get contract to do something when it is first mined
===================================================

Use the constructor. Anything inside it will be executed when the contract is first mined.

See `replicator.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/50_replicator.sol>`_.

How do you create 2-dimensional arrays?
=======================================

See `2D_array.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/55_2D_array.sol>`_.

Note that filling a 10x10 square of ``uint8`` + contract creation took more than ``800,000``
gas at the time of this writing. 17x17 took ``2,000,000`` gas. With the limit at
3.14 million... well, there’s a pretty low ceiling for what you can create right
now.

Note that merely "creating" the array is free, the costs are in filling it.

Note2: Optimizing storage access can pull the gas costs down considerably, because
32 ``uint8`` values can be stored in a single slot. The problem is that these optimizations
currently do not work across loops and also have a problem with bounds checking.
You might get much better results in the future, though.

What happens to a ``struct``'s mapping when copying over a ``struct``?
======================================================================

This is a very interesting question. Suppose that we have a contract field set up like such::

    struct User {
        mapping(string => string) comments;
    }

    function somefunction public {
       User user1;
       user1.comments["Hello"] = "World";
       User user2 = user1;
    }

In this case, the mapping of the struct being copied over into the userList is ignored as there is no "list of mapped keys".
Therefore it is not possible to find out which values should be copied over.

How do I initialize a contract with only a specific amount of wei?
==================================================================

Currently the approach is a little ugly, but there is little that can be done to improve it.
In the case of a ``contract A`` calling a new instance of ``contract B``, parentheses have to be used around
``new B`` because ``B.value`` would refer to a member of ``B`` called ``value``.
You will need to make sure that you have both contracts aware of each other's presence and that ``contract B`` has a ``payable`` constructor.
In this example::

    pragma solidity ^0.4.0;

    contract B {
        function B() public payable {}
    }

    contract A {
        address child;

        function test() public {
            child = (new B).value(10)(); //construct a new B with 10 wei
        }
    }

Can a contract function accept a two-dimensional array?
=======================================================

This is not yet implemented for external calls and dynamic arrays -
you can only use one level of dynamic arrays.

What is the relationship between ``bytes32`` and ``string``? Why is it that ``bytes32 somevar = "stringliteral";`` works and what does the saved 32-byte hex value mean?
========================================================================================================================================================================

The type ``bytes32`` can hold 32 (raw) bytes. In the assignment ``bytes32 samevar = "stringliteral";``,
the string literal is interpreted in its raw byte form and if you inspect ``somevar`` and
see a 32-byte hex value, this is just ``"stringliteral"`` in hex.

The type ``bytes`` is similar, only that it can change its length.

Finally, ``string`` is basically identical to ``bytes`` only that it is assumed
to hold the UTF-8 encoding of a real string. Since ``string`` stores the
data in UTF-8 encoding it is quite expensive to compute the number of
characters in the string (the encoding of some characters takes more
than a single byte). Because of that, ``string s; s.length`` is not yet
supported and not even index access ``s[2]``. But if you want to access
the low-level byte encoding of the string, you can use
``bytes(s).length`` and ``bytes(s)[2]`` which will result in the number
of bytes in the UTF-8 encoding of the string (not the number of
characters) and the second byte (not character) of the UTF-8 encoded
string, respectively.


Can a contract pass an array (static size) or string or ``bytes`` (dynamic size) to another contract?
=====================================================================================================

Sure. Take care that if you cross the memory / storage boundary,
independent copies will be created::

    pragma solidity ^0.4.16;

    contract C {
        uint[20] x;

        function f() public {
            g(x);
            h(x);
        }

        function g(uint[20] y) internal pure {
            y[2] = 3;
        }

        function h(uint[20] storage y) internal {
            y[3] = 4;
        }
    }

The call to ``g(x)`` will not have an effect on ``x`` because it needs
to create an independent copy of the storage value in memory
(the default storage location is memory). On the other hand,
``h(x)`` successfully modifies ``x`` because only a reference
and not a copy is passed.

Sometimes, when I try to change the length of an array with ex: ``arrayname.length = 7;`` I get a compiler error ``Value must be an lvalue``. Why?
==================================================================================================================================================

You can resize a dynamic array in storage (i.e. an array declared at the
contract level) with ``arrayname.length = <some new length>;``. If you get the
"lvalue" error, you are probably doing one of two things wrong.

1. You might be trying to resize an array in "memory", or

2. You might be trying to resize a non-dynamic array.

::

    int8[] memory memArr;        // Case 1
    memArr.length++;             // illegal
    int8[5] storageArr;          // Case 2
    somearray.length++;          // legal
    int8[5] storage storageArr2; // Explicit case 2
    somearray2.length++;         // legal

**Important note:** In Solidity, array dimensions are declared backwards from the way you
might be used to declaring them in C or Java, but they are access as in
C or Java.

For example, ``int8[][5] somearray;`` are 5 dynamic ``int8`` arrays.

The reason for this is that ``T[5]`` is always an array of 5 ``T``'s,
no matter whether ``T`` itself is an array or not (this is not the
case in C or Java).

Is it possible to return an array of strings (``string[]``) from a Solidity function?
=====================================================================================

Not yet, as this requires two levels of dynamic arrays (``string`` is a dynamic array itself).

If you issue a call for an array, it is possible to retrieve the whole array? Or must you write a helper function for that?
===========================================================================================================================

The automatic :ref:`getter function<getter-functions>`  for a public state variable of array type only returns
individual elements. If you want to return the complete array, you have to
manually write a function to do that.


What could have happened if an account has storage value(s) but no code?  Example: http://test.ether.camp/account/5f740b3a43fbb99724ce93a879805f4dc89178b5
==========================================================================================================================================================

The last thing a constructor does is returning the code of the contract.
The gas costs for this depend on the length of the code and it might be
that the supplied gas is not enough. This situation is the only one
where an "out of gas" exception does not revert changes to the state,
i.e. in this case the initialisation of the state variables.

https://github.com/ethereum/wiki/wiki/Subtleties

After a successful CREATE operation's sub-execution, if the operation returns x, 5 * len(x) gas is subtracted from the remaining gas before the contract is created. If the remaining gas is less than 5 * len(x), then no gas is subtracted, the code of the created contract becomes the empty string, but this is not treated as an exceptional condition - no reverts happen.


What does the following strange check do in the Custom Token contract?
======================================================================

::

    require((balanceOf[_to] + _value) >= balanceOf[_to]);

Integers in Solidity (and most other machine-related programming languages) are restricted to a certain range.
For ``uint256``, this is ``0`` up to ``2**256 - 1``. If the result of some operation on those numbers
does not fit inside this range, it is truncated. These truncations can have
`serious consequences <https://en.bitcoin.it/wiki/Value_overflow_incident>`_, so code like the one
above is necessary to avoid certain attacks.


More Questions?
===============

If you have more questions or your question is not answered here, please talk to us on
`gitter <https://gitter.im/ethereum/solidity>`_ or file an `issue <https://github.com/ethereum/solidity/issues>`_.
