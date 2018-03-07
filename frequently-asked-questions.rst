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

鉴于交易数据的写入是由矿工决定的而不是由提交者决定的，谁也无法保证交易一定会发生在下一个或未来某一个特定的区块上。这个结论适用于函数调用/交易以及合约的创建。

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

如果想让合约不再可用，建议的做法是修改合约内在逻辑来使其 **失效** ，让所有函数调用都变为无效返回。这样就无法使用这份合约了，而且发送过去的以太币也会被自动退回。

现在正式回答这个问题：在构造器中，将creator赋值为 ``msg.sender`` ，并保存。然后调用 ``selfdestruct(creator);`` 来中止程序并进行退款。

`例子 <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/05_greeter.sol>`_

需要注意的是，如果你已经在合约顶部做了引用 ``import "mortal"`` 并且申明了 
``contract SomeContract is mortal { ...`` ，然后再在已存在此合约的编译器中进行编译（包含 `Remix <https://remix.ethereum.org/>`_），那么 ``kill()`` 就会自动执行。当一份合约被申明为"mortal"时，你可以仿照我的例子，使用 ``contractname.kill.sendTransaction({from:eth.coinbase})`` 来中止它。


在合约中存储以太币
==================

诀窍是在合约中使用 ``{from:someaddress, value: web3.toWei(3,"ether")...}``

参考 `endowment_retriever.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/30_endowment_retriever.sol>`_ 。

使用非固定函数（请求 ``sendTransaction`` ）来对合约中的变量进行递增
===================================================================

参考 `value_incrementer.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/20_value_incrementer.sol>`_ 。

让合约把费用返还给你（不使用 ``selfdestruct(...)`` ）
====================================================

这个例子展示了如何将费用从一份合约发送至一个地址。

参考 `endowment_retriever <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/30_endowment_retriever.sol>`_ 。

调用Solidity方法可以返回一个数组或字符串（``string``）吗？
==========================================================

可以。参考 `array_receiver_and_returner.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/60_array_receiver_and_returner.sol>`_ 。

但是，在 **Solidity内部** 调用一个函数并返回非固定长度的数据（例如 ``uint[]`` 这种未定义长度的数组）时，往往会出现问题。这是EVM自身的限制，我们已经计划在下一次协议升级时解决这个问题。

将非固定长度数据作为外部交易或调用的一部分返回是没问题的。

可以使用嵌套结构给数组进行初始化吗？比如： ``string[] myarray = ["a", "b"];``
=============================================================================

可以。然而需要注意的是，这方法现在只能用于固定长度的内存数组。你甚至可以在返回语句中新建一个嵌套内存数组。听起来很酷，对吧！ 

例子::

    pragma solidity ^0.4.16;

    contract C {
        function f() public pure returns (uint8[5]) {
            string[4] memory adaArr = ["This", "is", "an", "array"];
            return ([1, 2, 3, 4, 5]);
        }
    }

合约的函数可以返回数据结构（ ``struct`` ）吗？
==========================================

可以，但只适用于内部（ ``internal`` ）函数调用。

我从一个返回的枚举类型（ ``enum`` ）中，使用web3.js只得到了整数值。我该如何获取具名数值？
=========================================================================================

虽然Solidity支持枚举类型，但ABI（应用程序二进制接口）并不支持。当前阶段你需要自己去做映射，将来我们可能会提供一些帮助。

嵌套结构可以用来初始化状态变量吗？
==================================

可以，所有类型都可以（甚至包括数据结构）。然而需要注意的是，在数组使用这个方法的时候需要将其定义为静态内存数组。

例子::

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

数据结构（ ``structs`` ）如何使用？
===================================

参考 `struct_and_for_loop_tester.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/65_struct_and_for_loop_tester.sol>`_ 。

循环（ ``for loops`` ）如何使用？
=================================

和JavaScript非常相像。但有一点需要注意：

如果你使用 ``for (var i = 0; i < a.length; i ++) { a[i] = i; }`` ，那么 ``i`` 的数据类型将会是 ``uint8`` ，需要从 ``0`` 开始计数。也就是说，如果 ``a`` 有超过 ``255`` 个元素，那么循环就无法中止，因为 ``i`` 最大只能变为 ``255`` 。

最好使用 ``for (uint i = 0; i < a.length...``

参考 `struct_and_for_loop_tester.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/65_struct_and_for_loop_tester.sol>`_ 。

有没有一些简单的操作字符串的例子（ ``substring`` ， ``indexOf`` ，``charAt`` 等）？
===================================================================================

这里有一些字符串相关的功能性函数 `stringUtils.sol <https://github.com/ethereum/dapp-bin/blob/master/library/stringUtils.sol>`_ ，并且会在将来作扩展。另外，Arachnid有写过 `solidity-stringutils <https://github.com/Arachnid/solidity-stringutils>`_ 。

当前，如果你想修改一个字符串（甚至你只是想获取其长度），首先都必须将其转化为一个 ``bytes`` ::

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


我能拼接两个字符串吗？
======================

目前只能通过手工实现。

为什么大家都选择将合约实例化成一个变量（ ``ContractB b;`` ），然后去执行变量的函数（ ``b.doSomething();`` ），而不是直接调用这个低级函数 ``.call()`` ？
==========================================================================================================================================================================

如果你真实调用函数，编译器会提示诸如参数类型不匹配的问题，如果函数不存在或者不可见，他也会自动帮你打包参数。

参考 `ping.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_ping.sol>`_ and
`pong.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_pong.sol>`_ 。

没被使用的燃料会被自动退回吗？
==============================

是的，马上会退回。例如，作为交易的一部分，在交易完成的同时完成退款。

当返回一个值的时候，比如说 ``uint`` 类型的值, 可以返回一个 ``undefined`` 或者类 "null" 的值吗？
===============================================================================================

这不可能，因为所有的数据类型已经覆盖了全部的取值范围。

替代方案是可以在错误时抛出（ ``throw`` ），这同样能重置整个交易，当你遇到意外情况时不失为一个好的选择。

如果你不想抛出，也可以返回对值::

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


注释会被包含在已部署的合约里吗，而且会增加部署的燃料费吗？
==========================================================

不会，所有执行时非必须的内容都会在编译的时候被移除。
其中就包括注释、变量名和类型名。

如果在调用合约的函数时一起发送了以太币，将会发生什么？
======================================================

就像在创建合约时发送以太币一样，会累加到合约的余额总数上。
你只可以将以太币一起发送至拥有 ``payable`` 修饰符的函数，不然会抛出异常。

合约对合约的交易可以获得交易回执吗？
====================================

不能，合约对合约的函数调用并不会创建前者自己的交易，你必须要去查看全部的交易。这也是为什么很多区块管理器无法正确显示合约对合约发送的以太币。

关键字 ``memory`` 是什么？是用来做什么的？
==========================================

以太坊虚拟机拥有三类存储区域。

第一类是存储（ "storage" ），贮存了合约申明中所有的变量。
虚拟机会为每份合约分别划出一片独立的存储（ "storage" ）区域，并在函数相互调用时持久存在，所以其使用开销非常大。

第二类是内存（ "memory" ），用于暂存数据。其中存储的内容会在函数被调用（包括外部函数）时擦除，所以其使用开销相对较小。

第三类是栈，用于存放本地小变量。使用几乎是免费的，但容量有限。

对绝大部分数据类型来说，由于每次被使用时都会被复制，所以你无法指定将其存储在哪里。

在数据类型中，对所谓存储地点比较重视的是结构和数组。 如果你在函数调用中传递了这类参数，假设它们的数据可以被贮存在存储（storage）或内存（memory）中，那么它们将不会被复制。也就是说，当你在被调用函数中修改了它们的内容，这些修改调用者也是可见的。

不同数据类型的变量会有各自默认的存储地点：

* 状态变量总是会贮存在存储（storage）中
* 函数参数默认存放在内存（memory）中
* 数据结构、数组或映射类型的本地变量，默认会放在存储（storage）中
* 除数据结构、数组及映射类型之外的本地变量，会储存在栈中

例子::

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

函数 ``append`` 能一起作用于 ``data1`` 和 ``data2`` ，并且修改是永久保存的。如果你移除了 ``storage`` 关键字，函数的参数会默认存储于 ``memory`` 。这带来的影响是，在 ``append(data1)`` 或 ``append(data2)`` 被调用的时节，一份全新的状态变量的拷贝会在内存（memory）中被创建， ``append`` 操作的会是这份拷贝（也不支持 ``.push`` -但这又是另一个话题了）。针对这份全新的拷贝的修改，不会反过来影响 ``data1`` 或 ``data2`` 。

一个常见误区就是申明了一个本地变量，就认为它会创建在内存（memory）中，其实它会被创建在存储（storage）中::

    /// 这份合约包含一处错误

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

本地变量 ``x`` 的数据类型是 ``uint[] storage``，但由于存储（storage）不是动态指定的，它需要在使用前通过状态变量赋值。所以 ``x`` 本身不会被分配存储（storage）的空间，取而代之的是，它只是作为存储（storage）中已有变量的别名。 

实际上会发生的是，编译器将 ``x`` 解析为一个存储指针，并默认将指针指向存储（storage）的 ``0`` 位置。这就造成 ``someVariable`` （贮存在存储（storage）的 ``0`` 位置）会被 ``x.push(2)`` 更改。

正确的方法如下::

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
怎样才能在合约中获取一个随机数？（落地一份自动回款的博彩合约）
==============================================================

Getting randomness right is often the crucial part in a crypto project and
most failures result from bad random number generators.
做好随机这件事情，往往是一个加密项目最关键的部分，大部分的失败都来自于使用了低劣的随机数发生器。

If you do not want it to be safe, you build something similar to the `coin flipper <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/35_coin_flipper.sol>`_
but otherwise, rather use a contract that supplies randomness, like the `RANDAO <https://github.com/randao/randao>`_.
如果你不考虑安全性，可以做一个类似于 `coin flipper <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/35_coin_flipper.sol>`_ 的东西，反之，最好调用一份提供随机性的合约，比如 `RANDAO <https://github.com/randao/randao>`_ 。

Get return value from non-constant function from another contract
=================================================================
从另一份合约中的非固定函数获取返回值
====================================

The key point is that the calling contract needs to know about the function it intends to call.
关键点是调用者（合约）需要了解将被调用的函数。

See `ping.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_ping.sol>`_
and `pong.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_pong.sol>`_.
参考 `ping.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_ping.sol>`_
和 `pong.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_pong.sol>`_ 。

Get contract to do something when it is first mined
===================================================
让合约在第一次被挖矿时就开始做些事情
====================================

Use the constructor. Anything inside it will be executed when the contract is first mined.
使用构造函数。在构造函数中写的任何内容都会在第一次被挖矿时执行。

See `replicator.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/50_replicator.sol>`_.
参考 `replicator.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/50_replicator.sol>`_ 。

How do you create 2-dimensional arrays?
=======================================
怎样才能创建二维数组？
======================

See `2D_array.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/55_2D_array.sol>`_.
参考 `2D_array.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/55_2D_array.sol>`_ 。

Note that filling a 10x10 square of ``uint8`` + contract creation took more than ``800,000``
gas at the time of this writing. 17x17 took ``2,000,000`` gas. With the limit at
3.14 million... well, there’s a pretty low ceiling for what you can create right
now.
需要注意的是，用 ``uint8`` 类型的数据填满一个10x10的方阵，再加上合约创建，总共需要花费超过 ``800,000`` 燃料费。如果是17x17需要 ``2,000,000`` 燃料费。然而交易燃料费上限是314万。。。好吧，其实你也玩不了太大的花样。

Note that merely "creating" the array is free, the costs are in filling it.
注意，“创建”数组纯粹是免费的，成本在于填充数组。

Note2: Optimizing storage access can pull the gas costs down considerably, because
32 ``uint8`` values can be stored in a single slot. The problem is that these optimizations
currently do not work across loops and also have a problem with bounds checking.
You might get much better results in the future, though.
还需注意，存储访问的优化可以大大降低燃料的成本，因为一个位置可以存放下32个 ``uint8`` 类型的值。但这类优化目前也存在一些问题：在跨循环的时候不起作用；以及在边界检查时候会出问题。

What happens to a ``struct``'s mapping when copying over a ``struct``?
======================================================================
当我们复制一个 ``数据结构`` 时， ``数据结构`` 中定义的映射会被怎么处理？
========================================================================

This is a very interesting question. Suppose that we have a contract field set up like such::
这是一个非常有意思的问题。假设我们有一份合约，里面的字段设置如下::

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
在这种情况下，由于缺失 "被映射的键列表" ，被复制至userList的数据结构中的映射会被忽视。因此，系统无法找出什么值可以被复制过去。

How do I initialize a contract with only a specific amount of wei?
==================================================================
我应该如何创建一份只包含指定数量wei的合约？
===========================================

Currently the approach is a little ugly, but there is little that can be done to improve it.
In the case of a ``contract A`` calling a new instance of ``contract B``, parentheses have to be used around
``new B`` because ``B.value`` would refer to a member of ``B`` called ``value``.
You will need to make sure that you have both contracts aware of each other's presence and that ``contract B`` has a ``payable`` constructor.
目前实现方式不是太优雅，当然暂时也没有更好的方法。
就拿 ``合约A`` 调用一个 ``合约B`` 的新实例来说，``new B`` 周围必须要加括号，不然 ``B.value`` 就会被认作是一个叫做 ``B`` 的成员在调用 ``value``。
你必须确保两份合约都知道对方的存在，并且 ``合约B`` 拥有 ``payable`` 构造函数。

In this example::
就是这个例子::

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
合约的函数可以接收二维数组吗？
==============================

This is not yet implemented for external calls and dynamic arrays -
you can only use one level of dynamic arrays.
二维数组还无法使用于外部调用和动态数组 - 你只能使用一维的动态数组。

What is the relationship between ``bytes32`` and ``string``? Why is it that ``bytes32 somevar = "stringliteral";`` works and what does the saved 32-byte hex value mean?
========================================================================================================================================================================
``bytes32`` 和 ``string`` 有什么关系吗？为什么 ``bytes32 somevar = "stringliteral";`` 可以生效，还有存下来的32-字节的16进制数值有什么含义？
================================================================================================================================================================================

The type ``bytes32`` can hold 32 (raw) bytes. In the assignment ``bytes32 samevar = "stringliteral";``,
the string literal is interpreted in its raw byte form and if you inspect ``somevar`` and
see a 32-byte hex value, this is just ``"stringliteral"`` in hex.
数据类型 ``bytes32`` 可以存放 32个（原始）字节。在给变量分配值的过程中 ``bytes32 samevar = "stringliteral";``，
字符串已经被逐字翻译成了原始字节。如果你去检查 ``somevar`` ，会发现一个32-字节的16进制数值，这就是用16进制表示的 ``"字符串的文字"`` 。

The type ``bytes`` is similar, only that it can change its length.
数据类型 ``bytes`` 与此类似，只是它的长度可以改变。

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
最终来看，假设 ``bytes`` 储存的是字符串的UTF-8编码，那么它和 ``string`` 基本是等同的。由于 ``string`` 存储是UTF-8编码格式的数据，所以计算字符串中字符的数量的成本是很高的（某些字符的编码甚至大于一个字节）。因此，系统还不支持 ``string s; s.length`` ，甚至不能通过索引访问 ``s[2]`` 。但如果你想访问字符串的低级字节编码，可以使用 ``bytes(s).length`` 和 ``bytes(s)[2]``，它们分别会返回字符串在UTF-8编码下的字节数量（不是字符数量）以及字符串UTF-8编码的第二个字节（不是字符）。

Can a contract pass an array (static size) or string or ``bytes`` (dynamic size) to another contract?
=====================================================================================================
一份合约可以传递一个数组（固定长度）或者一个字符串或者一个 ``bytes`` （不定长度）给另一份合约吗？
=================================================================================================

Sure. Take care that if you cross the memory / storage boundary,
independent copies will be created::
当然可以。但如果不小心跨越了内存 / 存储的边界，一份独立的拷贝就会被创建出来::

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
由于需要在内存中对存储的值创建一份独立的拷贝（默认存储在内存中），所以对 ``g(x)`` 的调用其实并不会对 ``x`` 产生影响。另一方面，由于传递的只是引用而不是一个拷贝， ``h(x)`` 得以成功地修改了 ``x`` 。

Sometimes, when I try to change the length of an array with ex: ``arrayname.length = 7;`` I get a compiler error ``Value must be an lvalue``. Why?
==================================================================================================================================================
有些时候，当我想用类似这样的表达式： ``arrayname.length = 7;`` 来修改数组长度，却会得到一个编译错误 ``Value must be an lvalue`` 。这是为什么？
====================================================================================================================================================

You can resize a dynamic array in storage (i.e. an array declared at the
contract level) with ``arrayname.length = <some new length>;``. If you get the
"lvalue" error, you are probably doing one of two things wrong.
你可以使用 ``arrayname.length = <some new length>;`` 来调整存储中的动态数组的长度（例如在合约层申明的数组）。如果你得到一个 "lvalue" 错误，那么你有可能做错了以下两件事中的一件或全部。

1. You might be trying to resize an array in "memory", or
1. 你在尝试修改长度的数组可能是存在 "内存" 中的，或者

2. You might be trying to resize a non-dynamic array.
2. 你可能在尝试修改一个非动态数组的长度。

::

    int8[] memory memArr;        // 第一种情况
    memArr.length++;             // 非法操作
    int8[5] storageArr;          // 第二种情况
    somearray.length++;          // 非法操作
    int8[5] storage storageArr2; // 第二种情况附加显式定义
    somearray2.length++;         // 合法操作

**Important note:** In Solidity, array dimensions are declared backwards from the way you
might be used to declaring them in C or Java, but they are access as in
C or Java.
**重要提醒：** 在Solidity中，数组维数的申明方向是和在C或Java中的申明方式相反的，但访问方式相同。

For example, ``int8[][5] somearray;`` are 5 dynamic ``int8`` arrays.
举个例子， ``int8[][5] somearray;`` 是5个 ``int8`` 格式的动态数组。

The reason for this is that ``T[5]`` is always an array of 5 ``T``'s,
no matter whether ``T`` itself is an array or not (this is not the
case in C or Java).
这么做的原因是 ``T[5]`` 就总是能被识别为5个 ``T`` ，哪怕 ``T`` 本身就是一个数组（而在C或Java是不一样的）。 

Is it possible to return an array of strings (``string[]``) from a Solidity function?
=====================================================================================
Solidity的函数可以返回一个字符串数组吗（ ``string[]`` ）？
==========================================================

Not yet, as this requires two levels of dynamic arrays (``string`` is a dynamic array itself).
暂时还不可以，因为这要求两个层面的动态数组（ ``string`` 本身就是一种动态数组）

If you issue a call for an array, it is possible to retrieve the whole array? Or must you write a helper function for that?
===========================================================================================================================
如果你发起了一次数组的调用，有可能获得整个数组吗？还是说另外需要写一个帮助函数来实现？
======================================================================================

The automatic :ref:`getter function<getter-functions>`  for a public state variable of array type only returns
individual elements. If you want to return the complete array, you have to
manually write a function to do that.
公有申明的数组类型变量有个自动获取函数 :ref:`getter function<getter-functions>` , 这个函数只会返回单个元素。如果你想获取完整的数组，那么只能再手工写一个函数来实现。

What could have happened if an account has storage value(s) but no code?  Example: http://test.ether.camp/account/5f740b3a43fbb99724ce93a879805f4dc89178b5
==========================================================================================================================================================
如果某个账户只存储了值但没有任何代码，将会发生什么？例子: http://test.ether.camp/account/5f740b3a43fbb99724ce93a879805f4dc89178b5
=================================================================================================================================

The last thing a constructor does is returning the code of the contract.
The gas costs for this depend on the length of the code and it might be
that the supplied gas is not enough. This situation is the only one
where an "out of gas" exception does not revert changes to the state,
i.e. in this case the initialisation of the state variables.
构造函数做的最后一件事情是返回合约的代码。这件事消耗的燃料取决于代码的长度，并且有种可能是提供的燃料不够。那么这就是唯一一种情况下，出现了 "out of gas" 异常然而被改变了的申明却没有被重置，这个改变在这里就是对申明变量的初始化。

https://github.com/ethereum/wiki/wiki/Subtleties

After a successful CREATE operation's sub-execution, if the operation returns x, 5 * len(x) gas is subtracted from the remaining gas before the contract is created. If the remaining gas is less than 5 * len(x), then no gas is subtracted, the code of the created contract becomes the empty string, but this is not treated as an exceptional condition - no reverts happen.
当CREATE操作的某个阶段被成功执行，如果这个操作返回x，那么5 * len(x)的燃料在合约被创建前会从剩余燃料中被扣除。如果剩余的燃料少于5 * len(x)，那么就不扣除，创建的合约代码会变成空字符串，但这时候并不认为是发生了异常 - 不会发生重置。

What does the following strange check do in the Custom Token contract?
======================================================================
在定制代币（Token）的合约中，下面这些奇怪的校验是做什么的？
===========================================================

::

    require((balanceOf[_to] + _value) >= balanceOf[_to]);

Integers in Solidity (and most other machine-related programming languages) are restricted to a certain range.
For ``uint256``, this is ``0`` up to ``2**256 - 1``. If the result of some operation on those numbers
does not fit inside this range, it is truncated. These truncations can have
`serious consequences <https://en.bitcoin.it/wiki/Value_overflow_incident>`_, so code like the one
above is necessary to avoid certain attacks.
在Solidity中的整形（以及大多数其他机器相关的编程语言）都会被限定在一定范围内。
比如 ``uint256`` ，就是从 ``0`` 到 ``2**256 - 1`` 。如果某些对这些数字的操作结果不在这个范围内，那么就会被裁剪。这些裁剪会带来
`严重的后果 <https://en.bitcoin.it/wiki/Value_overflow_incident>`_ ，所以像上面这样的代码需要考虑避免此类攻击。

More Questions?
===============
更多问题？
==========

If you have more questions or your question is not answered here, please talk to us on
`gitter <https://gitter.im/ethereum/solidity>`_ or file an `issue <https://github.com/ethereum/solidity/issues>`_.
如果你有其他问题，或者你的问题在这里找不到答案，请在此联系我们
`gitter <https://gitter.im/ethereum/solidity>`_ or file an `issue <https://github.com/ethereum/solidity/issues>`_ 。
