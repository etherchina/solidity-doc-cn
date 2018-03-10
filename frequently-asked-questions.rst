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

什么是交易的“有效载荷（payload）”？
========================

就是随交易一起发送的字节码“数据”。

存在反编译程序吗？
==================

除了 `Porosity <https://github.com/comaeio/porosity>`_ 有点接近之外，Solidity没有严格意义上的反编译程序。由于诸如变量名、注释、代码格式等会在编译过程中丢失，所以完全反编译回源代码是没有可能的。

很多区块链浏览器都能将字节码分解为一系列操作码。

如果区块链上的合约会被第三方使用，那么最好将源代码一起进行发布。

创建一个可以被中止并退款的合约
==============================

首先，需要提醒一下：中止合约听起来是一个好主意，把垃圾打扫干净是个好习惯，但如上所述，合约是不会被真正清理干净的。甚至，被发送至已移除合约的以太币，会从此丢失。

如果想让合约不再可用，建议的做法是修改合约内部状态来使其 **失效** ，让所有函数调用都变为无效返回。这样就无法使用这份合约了，而且发送过去的以太币也会被自动退回。

现在正式回答这个问题：在构造器中，将creator赋值为 ``msg.sender`` ，并保存。然后调用 ``selfdestruct(creator);`` 来中止程序并进行退款。

`例子 <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/05_greeter.sol>`_

需要注意的是，如果你已经在合约顶部做了引用 ``import "mortal"`` 并且声明了 
``contract SomeContract is mortal { ...`` ，然后再在已存在此合约的编译器中进行编译（包含 `Remix <https://remix.ethereum.org/>`_），那么 ``kill()`` 就会自动执行。当一份合约被声明为"mortal"时，你可以仿照我的例子，使用 ``contractname.kill.sendTransaction({from:eth.coinbase})`` 来中止它。


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

调用Solidity方法可以返回一个数组或字符串（ ``string`` ）吗？
==========================================================

可以。参考 `array_receiver_and_returner.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/60_array_receiver_and_returner.sol>`_ 。

但是，在 **Solidity内部** 调用一个函数并返回变长数据（例如 ``uint[]`` 这种变长数组）时，往往会出现问题。这是EVM自身的限制，我们已经计划在下一次协议升级时解决这个问题。

将变长数据作为外部交易或调用的一部分返回是没问题的。

数组可以使用单句（in-line）的方式来初始化吗？比如： ``string[] myarray = ["a", "b"];``
=============================================================================

可以。然而需要注意的是，这方法现在只能用于定长内存数组。你甚至可以在返回语句中用单句（in-line）的方式新建一个内存数组。听起来很酷，对吧！ 

例子::

    pragma solidity ^0.4.16;

    contract C {
        function f() public pure returns (uint8[5]) {
            string[4] memory adaArr = ["This", "is", "an", "array"];
            return ([1, 2, 3, 4, 5]);
        }
    }

合约的函数可以返回结构（ ``struct`` ）吗？
==========================================

可以，但只适用于内部（ ``internal`` ）函数调用。

我从一个返回的枚举类型（ ``enum`` ）中，使用web3.js只得到了整数值。我该如何获取具名数值？
=========================================================================================

虽然Solidity支持枚举类型，但ABI（应用程序二进制接口）并不支持。当前阶段你需要自己去做映射，将来我们可能会提供一些帮助。

可以使用单句（in-line）的方式来初始化状态变量吗？
==================================

可以，所有类型都可以（甚至包括结构）。然而需要注意的是，在数组使用这个方法的时候需要将其定义为静态内存数组。

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

结构（ ``structs`` ）如何使用？
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

如果你调用实际的成员函数，编译器会提示诸如参数类型不匹配的问题，如果函数不存在或者不可见，他也会自动帮你打包参数。

参考 `ping.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_ping.sol>`_ and
`pong.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_pong.sol>`_ 。

没被使用的gas会被自动退回吗？
==============================

是的，马上会退回。也就是说，作为交易的一部分，在交易完成的同时完成退款。

当返回一个值的时候，比如说 ``uint`` 类型的值, 可以返回一个 ``undefined`` 或者类 "null" 的值吗？
===============================================================================================

这不可能，因为所有的数据类型已经覆盖了全部的取值范围。

替代方案是可以在错误时抛出（ ``throw`` ），这同样能复原整个交易，当你遇到意外情况时不失为一个好的选择。

如果你不想抛出，也可以返回一对值（a pair）::

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


注释会被包含在已部署的合约里吗，而且会增加部署的gas吗？
==========================================================

不会，所有执行时非必须的内容都会在编译的时候被移除。
其中就包括注释、变量名和类型名。

如果在调用合约的函数时一起发送了以太币，将会发生什么？
======================================================

就像在创建合约时发送以太币一样，会累加到合约的余额总数上。
你只可以将以太币一起发送至拥有 ``payable`` 修饰符的函数，不然会抛出异常。

合约对合约的交易可以获得交易回执吗？
====================================

不能，合约对合约的函数调用并不会创建前者自己的交易，你必须要去查看全部的交易。这也是为什么很多区块浏览器无法正确显示合约对合约发送的以太币。

关键字 ``memory`` 是什么？是用来做什么的？
==========================================

以太坊虚拟机拥有三类存储区域。

第一类是存储（ "storage" ），贮存了合约声明中所有的变量。
虚拟机会为每份合约分别划出一片独立的存储（ "storage" ）区域，并在函数相互调用时持久存在，所以其使用开销非常大。

第二类是内存（ "memory" ），用于暂存数据。其中存储的内容会在函数被调用（包括外部函数）时擦除，所以其使用开销相对较小。

第三类是栈，用于存放小型的局部变量。使用几乎是免费的，但容量有限。

对绝大部分数据类型来说，由于每次被使用时都会被复制，所以你无法指定将其存储在哪里。

在数据类型中，对所谓存储地点比较重视的是结构和数组。 如果你在函数调用中传递了这类参数，假设它们的数据可以被贮存在存储（storage）或内存（memory）中，那么它们将不会被复制。也就是说，当你在被调用函数中修改了它们的内容，这些修改对调用者也是可见的。

不同数据类型的变量会有各自默认的存储地点：

* 状态变量总是会贮存在存储（storage）中
* 函数参数默认存放在内存（memory）中
* 结构、数组或映射类型的局部变量，默认会放在存储（storage）中
* 除结构、数组及映射类型之外的局部变量，会储存在栈中

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

一个常见误区就是声明了一个局部变量，就认为它会创建在内存（memory）中，其实它会被创建在存储（storage）中::

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

局部变量 ``x`` 的数据类型是 ``uint[] storage``，但由于存储（storage）不是动态指定的，它需要在使用前通过状态变量赋值。所以 ``x`` 本身不会被分配存储（storage）的空间，取而代之的是，它只是作为存储（storage）中已有变量的别名。 

实际上会发生的是，编译器将 ``x`` 解析为一个存储指针，并默认将指针指向存储槽（storage slot） ``0`` 。这就造成 ``someVariable`` （贮存在存储槽（storage slot） ``0`` ）会被 ``x.push(2)`` 更改。（在本例中，两个合约变量 someVariable 和 data 会被预先分配到两个存储槽（storage slot）中，即存储槽（storage slot） ``0`` 和 存储槽（storage slot） ``1`` 。上面的程序会使局部变量 x 变成指向保存了变量 someVariable 的存储槽 ``0`` 的指针。译者注。）

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

怎样才能在合约中获取一个随机数？（实施一份自动回款的博彩合约）
==============================================================

做好随机这件事情，往往是一个加密项目最关键的部分，大部分的失败都来自于使用了低劣的随机数发生器。

如果你不考虑安全性，可以做一个类似于 `coin flipper <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/35_coin_flipper.sol>`_ 的东西，反之，最好调用一份可以提供随机性的合约，比如 `RANDAO <https://github.com/randao/randao>`_ 。

从另一份合约中的非固定函数获取返回值
====================================

关键点是调用者（合约）需要了解将被调用的函数。

参考 `ping.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_ping.sol>`_
和 `pong.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_pong.sol>`_ 。

让合约在首次被挖出时就开始做些事情
====================================

使用构造函数。在构造函数中写的任何内容都会在首次被挖出时执行。

参考 `replicator.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/50_replicator.sol>`_ 。

怎样才能创建二维数组？
======================

参考 `2D_array.sol <https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/55_2D_array.sol>`_ 。

需要注意的是，用 ``uint8`` 类型的数据填满一个10x10的方阵，再加上合约创建，总共需要花费超过 ``800,000`` 的gas。如果是17x17需要 ``2,000,000`` 的gas。然而交易的gas上限是314万。。。好吧，其实你也玩不了太大的花样。

注意，“创建”数组纯粹是免费的，成本在于填充数组。

还需注意，优化存储访问可以大大降低gas的花费，因为一个存储槽（storage slot）可以存放下32个 ``uint8`` 类型的值。但这类优化目前也存在一些问题：在跨循环的时候不起作用；以及在边界检查时候会出问题。当然，在未来这种情况会得到改观。

当我们复制一个结构（ ``struct`` ）时， 结构 （ ``struct`` ）中定义的映射会被怎么处理？
========================================================================

这是一个非常有意思的问题。假设我们有一份合约，里面的字段设置如下::

    struct User {
        mapping(string => string) comments;
    }

    function somefunction public {
       User user1;
       user1.comments["Hello"] = "World";
       User user2 = user1;
    }

在这种情况下，由于缺失 "被映射的键列表" ，被复制至userList的结构中的映射会被忽视。因此，系统无法找出什么值可以被复制过去。

我应该如何初始化一份只包含指定数量wei的合约？
===========================================

目前实现方式不是太优雅，当然暂时也没有更好的方法。
就拿 ``合约A`` 调用一个 ``合约B`` 的新实例来说，``new B`` 周围必须要加括号，不然 ``B.value`` 会被认作是 ``B`` 的一个成员函数，叫做 ``value`` 。
你必须确保两份合约都知道对方的存在，并且 ``合约B`` 拥有 ``payable`` 构造函数。

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

合约的函数可以接收二维数组吗？
==============================

二维数组还无法使用于外部调用和动态数组 - 你只能使用一维的动态数组。

``bytes32`` 和 ``string`` 有什么关系吗？为什么 ``bytes32 somevar = "stringliteral";`` 可以生效，还有保存下来的那个32-字节的16进制数值有什么含义吗？
========================================================================================================================================================================================

数据类型 ``bytes32`` 可以存放 32个（原始）字节。在给变量分配值的过程中 ``bytes32 samevar = "stringliteral";``，
字符串已经被逐字翻译成了原始字节。如果你去检查 ``somevar`` ，会发现一个32-字节的16进制数值，这就是用16进制表示的 ``"字符串的文字"`` 。

数据类型 ``bytes`` 与此类似，只是它的长度可以改变。

最终来看，假设 ``bytes`` 储存的是字符串的UTF-8编码，那么它和 ``string`` 基本是等同的。由于 ``string`` 存储的是UTF-8编码格式的数据，所以计算字符串中字符数量的成本是很高的（某些字符的编码甚至大于一个字节）。因此，系统还不支持 ``string s; s.length`` ，甚至不能通过索引访问 ``s[2]`` 。但如果你想访问字符串的下级字节编码，可以使用 ``bytes(s).length`` 和 ``bytes(s)[2]``，它们分别会返回字符串在UTF-8编码下的字节数量（不是字符数量）以及字符串UTF-8编码的第二个字节（不是字符）。

一份合约可以传递一个数组（固定长度）或者一个字符串或者一个 ``bytes`` （不定长度）给另一份合约吗？
=================================================================================================

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

由于会在内存中对存储的值创建一份独立的拷贝（默认存储在内存中），所以对 ``g(x)`` 的调用其实并不会对 ``x`` 产生影响。另一方面，由于传递的只是引用而不是一个拷贝， ``h(x)`` 得以成功地修改了 ``x`` 。

有些时候，当我想用类似这样的表达式： ``arrayname.length = 7;`` 来修改数组长度，却会得到一个编译错误 ``Value must be an lvalue`` 。这是为什么？
====================================================================================================================================================

你可以使用 ``arrayname.length = <some new length>;`` 来调整存储中的动态数组（也就是在合约级别声明的数组）的长度。如果你得到一个 "lvalue" 错误，那么你有可能做错了以下两件事中的一件或全部。

1. 你在尝试修改长度的数组可能是存在 "内存" 中的，或者

2. 你可能在尝试修改一个非动态数组的长度。

::

    int8[] memory memArr;        // 第一种情况
    memArr.length++;             // 非法操作
    int8[5] storageArr;          // 第二种情况
    somearray.length++;          // 非法操作
    int8[5] storage storageArr2; // 第二种情况附加显式定义
    somearray2.length++;         // 合法操作

**重要提醒：** 在Solidity中，数组维数的声明方向是和在C或Java中的声明方向相反的，但访问方式相同。

举个例子， ``int8[][5] somearray;`` 是5个 ``int8`` 格式的动态数组。

这么做的原因是， ``T[5]`` 总是能被识别为5个 ``T`` 的数组，哪怕 ``T`` 本身就是一个数组（而在C或Java是不一样的）。 

Solidity的函数可以返回一个字符串数组吗（ ``string[]`` ）？
==========================================================

暂时还不可以，因为这要求两个维度都是动态数组（ ``string`` 本身就是一种动态数组）。

如果你发起了一次获取数组的调用，有可能获得整个数组吗？还是说另外需要写一个辅助函数来实现？
========================================================================================

一个数组类型的公共状态变量会有一个自动的获取函数 :ref:`getter function<getter-functions>` , 这个函数只会返回单个元素。如果你想获取完整的数组，那么只能再手工写一个函数来实现。

如果某个账户只存储了值但没有任何代码，将会发生什么？例子: http://test.ether.camp/account/5f740b3a43fbb99724ce93a879805f4dc89178b5
=================================================================================================================================

构造函数做的最后一件事情是返回合约的代码。这件事消耗的gas取决于代码的长度，其中有种可能的情况是提供的gas不够。这是唯一的一种情况下，出现了 "out of gas" 异常却不会去复原改变了的状态，这个改变在这里就是对状态变量的初始化。

https://github.com/ethereum/wiki/wiki/Subtleties

当CREATE操作的某个阶段被成功执行，如果这个操作返回x，那么5 * len(x)的gas在合约被创建前会从剩余gas中被扣除。如果剩余的gas少于5 * len(x)，那么就不进行gas扣除，而是把创建的合约代码改变成空字符串，但这时候并不认为是发生了异常 - 不会发生复原。

在定制代币（Token）的合约中，下面这些奇怪的校验是做什么的？
===========================================================

::

    require((balanceOf[_to] + _value) >= balanceOf[_to]);

在Solidity（以及大多数其他机器相关的编程语言）中的整型都会被限定在一定范围内。
比如 ``uint256`` ，就是从 ``0`` 到 ``2**256 - 1`` 。如果针对这些数字进行操作的结果不在这个范围内，那么就会被截断。这些截断会带来
`严重的后果 <https://en.bitcoin.it/wiki/Value_overflow_incident>`_ ，所以像上面这样的代码需要考虑避免此类攻击。

更多问题？
==========

如果你有其他问题，或者你的问题在这里找不到答案，请在此联系我们
`gitter <https://gitter.im/ethereum/solidity>`_ 或者提交一个 `issue <https://github.com/ethereum/solidity/issues>`_ 。
