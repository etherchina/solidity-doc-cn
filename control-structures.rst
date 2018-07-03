##################################
表达式和控制结构
##################################

.. index:: ! parameter, parameter;input, parameter;output

输入参数和输出参数
======================================

与 Javascript 一样，函数可能需要参数作为输入;
而与 Javascript 和 C 不同的是，它们可能返回任意数量的参数作为输出。


输入参数
----------------

输入参数的声明方式与变量相同。但是有一个例外，未使用的参数可以省略参数名。
例如，如果我们希望合约接受有两个整数形参的函数的外部调用，我们会像下面这样写
::

    pragma solidity ^0.4.16;

    contract Simple {
        function taker(uint _a, uint _b) public pure {
            // 用 _a 和 _b 实现相关功能.
        }
    }

输出参数
-----------------

输出参数的声明方式在关键词 ``returns`` 之后，与输入参数的声明方式相同。
例如，如果我们需要返回两个结果：两个给定整数的和与积，我们应该写作
::

    pragma solidity ^0.4.16;

    contract Simple {
        function arithmetics(uint _a, uint _b)
            public
            pure
            returns (uint o_sum, uint o_product)
        {
            o_sum = _a + _b;
            o_product = _a * _b;
        }
    }

输出参数名可以被省略。输出值也可以使用 ``return``语句指定。
 ``return`` 语句也可以返回多值，参阅：ref:`multi-return`。
返回的输出参数被初始化为 0；如果它们没有被显式赋值，它们就会一直为 0。


输入参数和输出参数可以在函数体中用作表达式。因此，它们也可用在等号左边被赋值。


.. index:: if, else, while, do/while, for, break, continue, return, switch, goto

控制结构
===================

JavaScript 中的大部分控制结构在 Solidity 中都是可用的，除了 ``switch`` 和 ``goto``。
因此 Solidity 中有 ``if``，``else``，``while``，``do``，``for``，``break``，``continue``，``return``，``? :``这些与在 C 或者 JavaScript 中表达相同语义的关键词。


用于表示条件的括号 *不可以* 被省略，单语句体两边的花括号可以被省略。


注意，与 C 和 JavaScript 不同， Solidity 中非布尔类型数值不能转换为布尔类型，因此 ``if (1) { ... }`` 的写法在 Solidity 中 *无效* 。



.. _multi-return:

返回多个值

-------------------------

当一个函数有多个输出参数时， ``return (v0, v1, ...,vn)`` 写法可以返回多个值。不过元素的个数必须与输出参数的个数相同。


.. index:: ! function;call, function;internal, function;external

.. _function-calls:

函数调用
==============

内部函数调用
-----------------------

当前合约中的函数可以直接（“从内部”）调用，也可以递归调用，就像下边这个荒谬的例子一样
::

    pragma solidity ^0.4.16;

    contract C {
        function g(uint a) public pure returns (uint ret) { return f(); }
        function f() internal pure returns (uint ret) { return g(7) + f(); }
    }

这些函数调用在 EVM 中被解释为简单的跳转。这样做的效果就是当前内存不会被清除，也就是说，通过内部调用在函数之间传递内存引用是非常有效的。


外部函数调用
-----------------------

表达式 ``this.g(8);`` 和 ``c.g(2);``（其中 ``c`` 是合约实例）也是有效的函数调用，但是这种情况下，函数将会通过一个消息调用来被“外部调用”，而不是直接的跳转。
请注意，不可以在构造函数中通过 this 来调用函数，因为此时真实的合约实例还没有被创建。


如果想要调用其他合约的函数，需要外部调用。对于一个外部调用，所有的函数参数都需要被复制到内存。


当调用其他合约的函数时，随函数调用发送的 Wei 和 gas 的数量可以分别由特定选项 ``.value()`` 和 ``.gas()`` 指定::


    pragma solidity ^0.4.0;

    contract InfoFeed {
        function info() public payable returns (uint ret) { return 42; }
    }

    contract Consumer {
        InfoFeed feed;
        function setFeed(address addr) public { feed = InfoFeed(addr); }
        function callFeed() public { feed.info.value(10).gas(800)(); }
    }

``payable`` 修饰符要用于修饰 ``info``，否则，`.value()` 选项将不可用。


注意，表达式 ``InfoFeed(addr)`` 进行了一个的显式类型转换，说明”我们知道给定地址的合约类型是 ``InfoFeed`` “并且这不会执行构造函数。
显式类型转换需要谨慎处理。绝对不要在一个你不清楚类型的合约上执行函数调用。


我们也可以直接使用 ``function setFeed(InfoFeed _feed) { feed = _feed; }`` 。
注意一个事实，``feed.info.value(10).gas(800)`` 只（局部地）设置了与函数调用一起发送的 Wei 值和 gas 的数量，只有最后的圆括号执行了真正的调用。


如果被调函数所在合约不存在（也就是账户中不包含代码）或者被调用合约本身抛出异常或者 gas 用完等，函数调用会抛出异常。


.. 警告::

	任何与其他合约的交互都会强加潜在危险，尤其是在不能预先知道合约代码的情况下。
	当前合约将控制权移交给被调用合约，而被调用合约可能做任何事。即使被调用合约从一个已知父合约继承，继承的合约也只需要有一个正确的接口就可以了。
	被调用合约的实现可以完全任意，因此会带来危险。此外，请小心万一它再调用你系统中的其他合约，或者甚至在第一次调用返回之前返回到你的调用合约。
	这意味着被调用合约可以通过它自己的函数改变调用合约的状态变量。。一个建议的函数写法是，例如，在你合约中状态变量进行各种变化后再调用外部函数，这样，你的合约就不会轻易被滥用的重入 (reentrancy) 所影响



具名调用和匿名函数参数
---------------------------------------------

函数调用参数也可以按照任意顺序由名称给出，如果它们被包含在 ``{}`` 中，
如以下示例中所示。参数列表必须按名称与函数声明中的参数列表相符，但可以按任意顺序排列。
::

    pragma solidity ^0.4.0;

    contract C {
        function f(uint key, uint value) public {
            // ...
        }

        function g() public {
            // 具名参数
            f({value: 2, key: 3});
        }
    }

省略函数参数名称
--------------------------------

未使用参数的名称（特别是返回参数）可以省略。这些参数仍然存在于堆栈中，但它们无法访问。
::

    pragma solidity ^0.4.16;

    contract C {
        // 省略参数名称
        function func(uint k, uint) public pure returns(uint) {
            return k;
        }
    }

.. index:: ! new, contracts;creating

.. _creating-contracts:

通过 ``new`` 创建合约
==============================

使用关键字 ``new`` 可以创建一个新合约。待创建合约的完整代码必须事先知道，因此递归的创建依赖是不可能的。
::

    pragma solidity ^0.4.0;

    contract D {
        uint x;
        function D(uint a) public payable {
            x = a;
        }
    }

    contract C {
        D d = new D(4); // 将作为合约 C 构造函数的一部分执行

        function createD(uint arg) public {
            D newD = new D(arg);
        }

        function createAndEndowD(uint arg, uint amount) public payable {
		    //随合约的创建发送 ether
            D newD = (new D).value(amount)(arg);
        }
    }

如示例中所示，使用 ``.value（）`` 选项创建 ``D`` 的实例时可以转发 Ether，但是不可能限制 gas 的数量。如果创建失败（可能因为栈溢出，或没有足够的余额或其他问题），会引发异常。

表达式计算顺序

==================================

表达式的计算顺序不是特定的（更准确地说，表达式树中某节点的字节点间的计算顺序不是特定的，但它们的结算肯定会在节点自己的结算之前）。该规则只能保证语句按顺序执行，布尔表达式的短路执行。更多相关信息，请参阅：:ref:`order`。


.. index:: ! assignment

赋值
==========

.. index:: ! assignment;destructuring

解构赋值和返回多值
-------------------------------------------------------

Solidity 内部允许元组 (tuple) 类型，也就是一个在编译时元素数量固定的对象列表，列表中的元素可以是不同类型的对象。这些元组可以用来同时返回多个数值，也可以用它们来同时给多个变量（或通常的 LValues）
::

    pragma solidity ^0.4.16;

    contract C {
        uint[] data;

        function f() public pure returns (uint, bool, uint) {
            return (7, true, 2);
        }

        function g() public {
            //声明变量并赋值。显式指定类型不可能。
            var (x, b, y) = f();
            //为已存在的变量赋值。
            (x, y) = (2, 7);
            //交换两个值的通用窍门——但不适用于非值类型的存储 (storage) 变量。
            (x, y) = (y, x);
            //元组的末尾元素可以省略（这也适用于变量声明）。
            //如果如果元组以空元素结束，
            //则其余的值将被丢弃。							  
            (data.length,) = f(); // 将长度设置为 7
            //左侧也可以做同样的事情。
            //如果元组以空元素开始，则初始值将被丢弃。
            (,data[3]) = f(); //将 data[3] 设置为 2
            //省略元组中末尾元素的写法，仅可以在赋值操作的左侧使用，除了这个例外：
            (x,) = (1,);
            //(1,) 是指定单元素元组的唯一方法，因为 (1)
            //相当于 1。
        }
    }

数组和结构体的复杂性
------------------------------------
赋值语义对于像数组和结构体这样的非值类型来说会有些复杂。
为状态变量 *赋值* 经常会创建一个独立副本。另一方面，对局部变量的赋值只会为基本类型（即 32 字节以内的静态类型）创建独立的副本。如果结构体或数组（包括 ``bytes`` 和 ``string``）被从状态变量分配给局部变量，局部变量将保留对原始状态变量的引用。对局部变量的第二次赋值不会修改状态变量，只会改变引用。赋值给局部变量的成员（或元素）则 *改变* 状态变量。

.. index:: ! scoping, declarations, default value

.. _default-value:

作用域和声明
========================

变量声明后将有默认初始值，其初始值字节表示全部为零。任何类型变量的“默认值”是其对应类型的典型“零状态”。例如， ``bool`` 类型的默认值是 ``false`` 。 ``uint`` 或 ``int`` 类型的默认值是 ``0`` 。对于静态大小的数组和 ``bytes1`` 到 ``bytes32`` ，每个单独的元素将被初始化为与其类型相对应的默认值。
最后，对于动态大小的数组， ``bytes`` 和 ``string`` 类型，其默认缺省值是一个空数组或字符串。


无论变量在哪里声明，只要是在函数中，变量都将存在于 *整个函数* 的范围内。这种情况是因为 Solidity 继承了 JavaScript 的范围规则。这与许多语言形成对比，在这些语言中变量的范围为变量声明处到函数体结束。
因此，下面的代码是非法的，并且会导致编译失败，报错为 ``Identifier already declared`` 
::

    //此处不会编译

    pragma solidity ^0.4.16;

    contract ScopingErrors {
        function scoping() public {
            uint i = 0;

            while (i++ < 1) {
                uint same1 = 0;
            }

            while (i++ < 2) {
                uint same1 = 0;// 非法，same1 的第二次声明
            }
        }

        function minimalScoping() public {
            {
                uint same2 = 0;
            }

            {							   
                uint same2 = 0;// 非法，same2 的第二次声明
            }
        }

        function forLoopScoping() public {
            for (uint same3 = 0; same3 < 1; same3++) {
            }													  
            for (uint same3 = 0; same3 < 1; same3++) {// 非法，same3 的第二次声明
            }
        }
    }

除此之外，如果声明了一个变量，它将在函数的开头初始化为其默认值。因此，下面的代码是合法的，尽管这种写法不推荐::

    pragma solidity ^0.4.0;

    contract C {
        function foo() public pure returns (uint) {
            // baz 被隐式初始化为 0
            uint bar = 5;
            if (true) {
                bar += baz;
            } else {
                uint baz = 10;// 不会执行
            }
            return bar;// returns 5
        }
    }

.. index:: ! exception, ! throw, ! assert, ! require, ! revert

错误处理：Assert, Require, Revert and Exceptions
======================================================

Solidity 使用状态恢复异常来处理错误。这种异常将撤消对当前调用（及其所有子调用）中的状态所做的所有更改，并且还向调用者标记错误。
便利函数 ``assert`` 和 ``require`` 可用于检查条件并在条件不满足时抛出异常。``assert`` 函数只能用于测试内部错误，并检查非变量。
 ``require`` 函数用于确认条件有效性，例如输入变量，或合约状态变量是否满足条件，或验证外部合约调用返回的值。
如果使用得当，分析工具可以评估你的合约，并标示出那些会使 ``assert`` 失败的条件和函数调用。
正常工作的代码不会导致一个 assert 语句的失败；如果这发生了，那就说明出现了一个需要你修复的 bug。


还有另外两种触发异常的方法： ``revert`` 函数可以用来标记错误并恢复当前的调用。
将来还可能在 ``revert`` 调用中包含有关错误的详细信息。 ``throw`` 关键字也可以用来替代 ``revert()`` 。


.. 注意::
       从 0.4.13 版本开始，``throw`` 这个关键字被弃用，并且将来会被逐渐淘汰。

当子调用发生异常时，它们会自动“冒泡”（即重新抛出异常）。这个规则的例外是 ``send`` 和低级函数 ``call`` ， ``delegatecall`` 和 ``callcode`` --如果这些函数发生异常，将返回 false ，而不是“冒泡”。


.. 警告::
    作为 EVM 设计的一部分，如果被调用合约帐户不存在，则低级函数 ``call`` ， ``delegatecall`` 和 ``callcode`` 将返回 success。因此如果需要使用低级函数时，必须在调用之前检查被调用合约是否存在。
	
Catching exceptions is not yet possible.
异常捕获还未实现

In the following example, you can see how ``require`` can be used to easily check conditions on inputs
and how ``assert`` can be used for internal error checking::
在下例中，你可以看到如何轻松使用``require``检查输入条件以及如何使用``assert``检查内部错误::
    pragma solidity ^0.4.0;

    contract Sharer {
        function sendHalf(address addr) public payable returns (uint balance) {
            require(msg.value % 2 == 0); // 只接受偶数
            uint balanceBeforeTransfer = this.balance;
            addr.transfer(msg.value / 2);
			//由于转移函数在失败时抛出异常并且不能在这里回调，因此我们应该没有办法仍然有一半的钱。
            assert(this.balance == balanceBeforeTransfer - msg.value / 2);
            return this.balance;
        }
    }

下列情况将会产生一个 ``assert`` 式异常：

#. 如果你访问数组的索引太大或为负数（例如 ``x[i]`` 其中 ``i >= x.length`` 或 ``i < 0``）。
#. 如果你访问固定长度 ``bytesN`` 的索引太大或为负数。
#. 如果你用零当除数做除法或模运算（例如 ``5 / 0`` 或 ``23 % 0`` ）。
#. 如果你移位负数位。
#. 如果你将一个太大或负数值转换为一个枚举类型。
#. 如果你调用内部函数类型的零初始化变量。
#. 如果你调用 ``assert`` 的参数（表达式）最终结算为 false。



下列情况将会产生一个 ``require`` 式异常：


#. 调用 ``throw`` 。
#. 如果你调用 ``require`` 的参数（表达式）最终结算为 ``false`` 。
#. 如果你通过消息调用调用某个函数，但该函数没有正确结束（它耗尽了 gas，没有匹配函数，或者本身抛出一个异常），上述函数不包括低级别的操作 ``call`` ， ``send`` ， ``delegatecall`` 或者 ``callcode`` 。低级操作不会抛出异常，而通过返回 ``false`` 来指示失败。
#. 如果你使用 ``new`` 关键字创建合约，但合约没有正确创建（请参阅上条有关”未正确完成“的定义）。
#. 如果你对不包含代码的合约执行外部函数调用。
#. 如果你的合约通过一个没有 ``payable`` 修饰符的公有函数（包括构造函数和 fallback 函数）接收 Ether。
#. 如果你的合约通过公有 getter 函数接收 Ether 。
#. 如果 ``.transfer()`` 失败。


在内部， Solidity 对一个 ``require`` 式的异常执行回退操作（指令 ``0xfd`` ）并执行一个无效操作（指令 ``0xfe`` ）来引发 ``assert`` 式异常。
在这两种情况下，都会导致 EVM 回退对状态所做的所有更改。回退的原因是不能继续安全地执行，因为没有实现预期的效果。
因为我们想保留交易的原子性，所以最安全的做法是回退所有更改并使整个交易（或至少是调用）不产生效果。
请注意， ``assert`` 式异常消耗了所有可用的调用 gas ，而从 Metropolis 版本起 ``require`` 式的异常不会消耗任何 gas。
