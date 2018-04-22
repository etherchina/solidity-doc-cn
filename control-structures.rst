##################################
表达式和控制结构
##################################

.. index:: ! parameter, parameter;input, parameter;output

Input Parameters and Output Parameters输入参数和输出参数
======================================

与Javascript一样，函数可能需要参数作为输入;
而与Javascript和C不同的是，它们可能返回任意数量的参数作为输出。
unlike in Javascript and C, they may also return arbitrary number of
parameters as output.

Input Parameters输入参数
----------------

输入参数的声明方式与变量相同。但是有一个例外，未使用的参数可以省略参数名。
例如，如果我们希望合约接受有两个整数形参的函数的外部调用，我们会像下面这样写
The input parameters are declared the same way as variables are. As an
exception, unused parameters can omit the variable name.
For example, suppose we want our contract to
accept one kind of external calls with two integers, we would write
something like::

    pragma solidity ^0.4.16;

    contract Simple {
        function taker(uint _a, uint _b) public pure {
            // 用_a和_b实现相关功能.
        }
    }

Output Parameters输出参数
-----------------

输出参数的声明方式在关键词``returns``之后，与输入参数的声明方式相同。
例如，如果我们需要返回两个结果：两个给定整数的和与积，我们应该写作
The output parameters can be declared with the same syntax after the
``returns`` keyword. For example, suppose we wished to return two results:
the sum and the product of the two given integers, then we would
write::

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

输出参数名可以被省略。输出值也可以使用``return``表述指定。
``return``表述也可以返回多值，参阅：ref:`multi-return`。
输出参数被初始化为0,；如果它们没有被显式赋值，它们就会一直为0。
The names of output parameters can be omitted.
The output values can also be specified using ``return`` statements.
The ``return`` statements are also capable of returning multiple
values, see :ref:`multi-return`.
Return parameters are initialized to zero; if they are not explicitly
set, they stay to be zero.

输入参数和输出参数可以在函数体中用作表达式。因此，它们也可用在等号左边被赋值。
Input parameters and output parameters can be used as expressions in
the function body.  There, they are also usable in the left-hand side
of assignment.

.. index:: if, else, while, do/while, for, break, continue, return, switch, goto

控制结构
Control Structures
===================

JavaScript中的大部分控制结构在Solidity中都是可用的，除了``switch`` 和 ``goto``。
因此Solidity中有``if``, ``else``, ``while``, ``do``, ``for``, ``break``, ``continue``, ``return``, ``? :``，这些关键词的语义与C或者JavaScript中表达语义相同。
Most of the control structures from JavaScript are available in Solidity
except for ``switch`` and ``goto``. So
there is: ``if``, ``else``, ``while``, ``do``, ``for``, ``break``, ``continue``, ``return``, ``? :``, with
the usual semantics known from C or JavaScript.

用于表示条件的括号*不可以*被省略，但是如果语句体中是单句表达式，则花括号可以被省略。
Parentheses can *not* be omitted for conditionals, but curly brances can be omitted
around single-statement bodies.

注意与C和JavaScript不同，Solidity中非布尔类型数值不能转换为布尔类型，因此``if (1) { ... }``的写法在Solidity中*无效*。
Note that there is no type conversion from non-boolean to boolean types as
there is in C and JavaScript, so ``if (1) { ... }`` is *not* valid
Solidity.

.. _multi-return:
.. _multi-return:

返回多个值
Returning Multiple Values
-------------------------

当一个函数有多个输出参数时，``return (v0, v1, ...,vn)`` 写法可以返回多个值。不过元素的个数必须与输出参数的个数相同。
When a function has multiple output parameters, ``return (v0, v1, ...,
vn)`` can return multiple values.  The number of components must be
the same as the number of output parameters.

.. index:: ! function;call, function;internal, function;external
.. index:: ! function;call, function;internal, function;external

.. _function-calls:
.. _function-calls:

函数调用
Function Calls
==============

内部函数调用
Internal Function Calls
-----------------------

当前合约中的函数可以被直接调用（“内部”），递归调用也可以，用法可以参见下例。
Functions of the current contract can be called directly ("internally"), also recursively, as seen in
this nonsensical example::

    pragma solidity ^0.4.16;

    contract C {
        function g(uint a) public pure returns (uint ret) { return f(); }
        function f() internal pure returns (uint ret) { return g(7) + f(); }
    }

这些函数调用在EVM中被解释为简单的跳转。这样做的效果在于当前内存不会被清除，这种方法很高效，例如可以将内存引用传递给内部调用的函数。但只有同属相同合约的函数可以被内部调用。
These function calls are translated into simple jumps inside the EVM. This has
the effect that the current memory is not cleared, i.e. passing memory references
to internally-called functions is very efficient. Only functions of the same
contract can be called internally.

外部函数调用
External Function Calls
-----------------------

表达式``this.g(8);``和``c.g(2);``（其中``c``是合约实例）也是有效的函数调用，但是这种情况下，函数被称为“外部”，通过一个消息调用而不是直接跳转调用。
请注意``this``的函数调用不可以在构造函数中使用，因为此时真实的合约实例还没有被创建。
The expressions ``this.g(8);`` and ``c.g(2);`` (where ``c`` is a contract
instance) are also valid function calls, but this time, the function
will be called "externally", via a message call and not directly via jumps.
Please note that function calls on ``this`` cannot be used in the constructor, as the
actual contract has not been created yet.

如果想要调用其他合约的函数，需要外部调用。对于一个外部调用，所有的函数参数都需要被复制到内存。
Functions of other contracts have to be called externally. For an external call,
all function arguments have to be copied to memory.

当调用其他合约的函数时，随函数调用发送的Wei和gas的数量可以分别由特定选项``.value()`` 和 ``.gas()``指定::
When calling functions of other contracts, the amount of Wei sent with the call and
the gas can be specified with special options ``.value()`` and ``.gas()``, respectively::

    pragma solidity ^0.4.0;

    contract InfoFeed {
        function info() public payable returns (uint ret) { return 42; }
    }

    contract Consumer {
        InfoFeed feed;
        function setFeed(address addr) public { feed = InfoFeed(addr); }
        function callFeed() public { feed.info.value(10).gas(800)(); }
    }

``payable``修饰符要用于修饰``info``，否则，`.value()`选项将不可用。
The modifier ``payable`` has to be used for ``info``, because otherwise, the `.value()`
option would not be available.

注意表达式``InfoFeed(addr)``完成显式类型转换，并表达出”我们知道给定地址的合约类型是``InfoFeed``“并且该表达式不执行构造函数。
显式类型转换需要谨慎处理。绝对不要在一个你不清楚类型的合约上执行函数调用。
Note that the expression ``InfoFeed(addr)`` performs an explicit type conversion stating
that "we know that the type of the contract at the given address is ``InfoFeed``" and
this does not execute a constructor. Explicit type conversions have to be
handled with extreme caution. Never call a function on a contract where you
are not sure about its type.

我们也可以直接使用``function setFeed(InfoFeed _feed) { feed = _feed; }``。
注意一个事实，``feed.info.value(10).gas(800)``只（本地地）设置了与函数调用一起发送的Wei值和gas的数量，只有最后的圆括号执行了真正的调用。
We could also have used ``function setFeed(InfoFeed _feed) { feed = _feed; }`` directly.
Be careful about the fact that ``feed.info.value(10).gas(800)``
only (locally) sets the value and amount of gas sent with the function call and only the
parentheses at the end perform the actual call.

如果被调函数所在合约不存在（也就是账户中不包含代码）或者被调用合约本身抛出异常或者gas用完等，函数调用会抛出异常。
Function calls cause exceptions if the called contract does not exist (in the
sense that the account does not contain code) or if the called contract itself
throws an exception or goes out of gas.

.. 警告::
.. warning::
	任何与其他合约的交互都会强加潜在危险，尤其是在不能预先知道合约代码的情况下。
	当前合约将控制权移交给被调用合约，而被调用合约可能做任何事。即使被调用合约从一个已知父合约继承，合约的继承只需要有一个正确的接口。
	被调用合约的实现可以完全任意，因此会带来危险。此外，请小心万一它再调用你系统中的其他合约，或者甚至在第一次调用返回之前返回到你的调用合约。
	这意味着被调用合约通过函数改变了调用合约的状态变量。一个建议的函数写法是，例如，在你合约中状态变量进行各种变化后再调用外部函数，这样你的合约不易遭受重放攻击的危害。
    Any interaction with another contract imposes a potential danger, especially
    if the source code of the contract is not known in advance. The current
    contract hands over control to the called contract and that may potentially
    do just about anything. Even if the called contract inherits from a known parent contract,
    the inheriting contract is only required to have a correct interface. The
    implementation of the contract, however, can be completely arbitrary and thus,
    pose a danger. In addition, be prepared in case it calls into other contracts of
    your system or even back into the calling contract before the first
    call returns. This means
    that the called contract can change state variables of the calling contract
    via its functions. Write your functions in a way that, for example, calls to
    external functions happen after any changes to state variables in your contract
    so your contract is not vulnerable to a reentrancy exploit.


命名调用和匿名函数参数
Named Calls and Anonymous Function Parameters
---------------------------------------------

函数调用参数也可以按照任意顺序由名称给出，如果它们被包含在``{}``中，
如以下示例中所示。参数列表必须按名称与函数声明中的参数列表重合，但可以按任意顺序排列。
Function call arguments can also be given by name, in any order,
if they are enclosed in ``{ }`` as can be seen in the following
example. The argument list has to coincide by name with the list of
parameters from the function declaration, but can be in arbitrary order.

::

    pragma solidity ^0.4.0;

    contract C {
        function f(uint key, uint value) public {
            // ...
        }

        function g() public {
            // 命名参数
            f({value: 2, key: 3});
        }
    }

省略函数参数名称
Omitted Function Parameter Names
--------------------------------

未使用参数的名称（特别是返回参数）可以省略。这些参数仍然存在于堆栈中，但它们无法访问。
The names of unused parameters (especially return parameters) can be omitted.
Those parameters will still be present on the stack, but they are inaccessible.

::

    pragma solidity ^0.4.16;

    contract C {
        // 省略参数名称
        function func(uint k, uint) public pure returns(uint) {
            return k;
        }
    }

.. index:: ! new, contracts;creating
.. index:: ! new, contracts;creating

.. _creating-contracts:
.. _creating-contracts:

通过``new``创建合约
Creating Contracts via ``new``
==============================

使用关键字``new``可以创建一个新合约。待创建合约的完整代码必须事先知道，因此递归的创建依赖是不可能的。
A contract can create a new contract using the ``new`` keyword. The full
code of the contract being created has to be known in advance, so recursive
creation-dependencies are not possible.

::

    pragma solidity ^0.4.0;

    contract D {
        uint x;
        function D(uint a) public payable {
            x = a;
        }
    }

    contract C {
        D d = new D(4); // 将作为合约C构造函数的一部分执行

        function createD(uint arg) public {
            D newD = new D(arg);
        }

        function createAndEndowD(uint arg, uint amount) public payable {
			//随合约的创建发送ether
            D newD = (new D).value(amount)(arg);
        }
    }

如示例中所示，使用``.value（）``选项创建``D``的实例时可以转发Ether，但是不可能限制gas的数量。如果创建失败（可能因为在堆栈外，或没有足够的余额或其他问题），会引发异常。
As seen in the example, it is possible to forward Ether while creating
an instance of ``D`` using the ``.value()`` option, but it is not possible
to limit the amount of gas.
If the creation fails (due to out-of-stack, not enough balance or other problems),
an exception is thrown.

表达式计算顺序
Order of Evaluation of Expressions
==================================

表达式的计算顺序不被指定（更正式地说，表达式树中一个节点的子节点间的计算顺序未被指定，但它们必然是在节点之前进行计算的）。该规则只能保证语句按顺序执行，布尔表达式的短路执行。更多相关信息，请参阅：ref：`order`。
The evaluation order of expressions is not specified (more formally, the order
in which the children of one node in the expression tree are evaluated is not
specified, but they are of course evaluated before the node itself). It is only
guaranteed that statements are executed in order and short-circuiting for
boolean expressions is done. See :ref:`order` for more information.

.. index:: ! assignment
.. index:: ! assignment

赋值
Assignment
==========

.. index:: ! assignment;destructuring
.. index:: ! assignment;destructuring

解构赋值和返回多值
Destructuring Assignments and Returning Multiple Values
-------------------------------------------------------

Solidity内支持元组类型，即在编译时大小为常量、类型不定的对象列表。这些元组可以同时返回多个值，也可以同时将它们分配给多个变量（或通常LValues）：
Solidity internally allows tuple types, i.e. a list of objects of potentially different types whose size is a constant at compile-time. Those tuples can be used to return multiple values at the same time and also assign them to multiple variables (or LValues in general) at the same time::

    pragma solidity ^0.4.16;

    contract C {
        uint[] data;

        function f() public pure returns (uint, bool, uint) {
            return (7, true, 2);
        }

        function g() public {
			//声明变量并赋值。显式指定类型不可能。
            // Declares and assigns the variables. Specifying the type explicitly is not possible.
            var (x, b, y) = f();
			//为已存在的变量赋值。
            // Assigns to a pre-existing variable.
            (x, y) = (2, 7);
			//交换两个变量值--对于非值存储的类型除外
            // Common trick to swap values -- does not work for non-value storage types.
            (x, y) = (y, x);
			//组件可以省略（该规则也适用于变量声明）。
			//如果如果元组以空组件结束，
			//则其余的值将被丢弃。
            // Components can be left out (also for variable declarations).
            // If the tuple ends in an empty component,
            // the rest of the values are discarded.
								  // 将长度设置为7
            (data.length,) = f(); // Sets the length to 7
			// 左侧也可以做同样的事情。
            // The same can be done on the left side.
			//如果元组以空组件开始，则开始值将被丢弃。
            // If the tuple begins in an empty component, the beginning values are discarded.
							  //将data[3]设置为2
            (,data[3]) = f(); // Sets data[3] to 2
			//组件只能在赋值符号的左侧被省略，但有一个例外：
            // Components can only be left out at the left-hand-side of assignments, with
            // one exception:
            (x,) = (1,);
			//（1，）是指定1分量元组的唯一方法，因为（1）
  			//相当于1。
            // (1,) is the only way to specify a 1-component tuple, because (1) is
            // equivalent to 1.
        }
    }

数组和结构体的并发症
Complications for Arrays and Structs
------------------------------------
赋值语义对于非值类型，诸如数组和结构体来说更复杂一些。
为状态变量*赋值*经常会创建一个独立副本。另一方面，为局部变量赋值时，只为基本类型创建独立副本，例如能与32字节相容的静态类型。如果结构体或数组（包括“字节”和“字符串”）被从状态变量分配给局部变量，局部变量将保留对原始状态变量的引用。对局部变量的第二次赋值不会修改状态变量，只会改变引用。赋值给局部变量的成员（或元素）则*改变*状态变量。
The semantics of assignment are a bit more complicated for non-value types like arrays and structs.
Assigning *to* a state variable always creates an independent copy. On the other hand, assigning to a local variable creates an independent copy only for elementary types, i.e. static types that fit into 32 bytes. If structs or arrays (including ``bytes`` and ``string``) are assigned from a state variable to a local variable, the local variable holds a reference to the original state variable. A second assignment to the local variable does not modify the state but only changes the reference. Assignments to members (or elements) of the local variable *do* change the state.

.. index:: ! scoping, declarations, default value
.. index:: ! scoping, declarations, default value

.. _default-value:
.. _default-value:

范围界定和声明
Scoping and Declarations
========================

变量声明后将有默认初始值，其初始值字节表示全部为零。任何类型变量的“默认值”是其对应类型的典型“零状态”。例如，``bool``类型的默认值是``false``。``uint``或``int``类型的默认值是``0``。对于静态大小的数组和``bytes1``到``bytes32``，每个单独的元素将被初始化为与其类型相对应的默认值。
最后，对于动态大小的数组，``bytes``和 ``string``类型，其默认缺省值是一个空数组或字符串。
A variable which is declared will have an initial default value whose byte-representation is all zeros.
The "default values" of variables are the typical "zero-state" of whatever the type is. For example, the default value for a ``bool``
is ``false``. The default value for the ``uint`` or ``int`` types is ``0``. For statically-sized arrays and ``bytes1`` to ``bytes32``, each individual
element will be initialized to the default value corresponding to its type. Finally, for dynamically-sized arrays, ``bytes``
and ``string``, the default value is an empty array or string.

无论变量在哪里声明，只要是在函数中，变量都将存在于*整个函数*的范围内。这种情况是因为Solidity继承了JavaScript的范围规则。这与许多语言形成对比，在这些语言中变量的范围为变量声明处到函数体结束。
因此，下面的代码是非法的，并且会导致编译失败，报错为``Identifier already declared``
A variable declared anywhere within a function will be in scope for the *entire function*, regardless of where it is declared.
This happens because Solidity inherits its scoping rules from JavaScript.
This is in contrast to many languages where variables are only scoped where they are declared until the end of the semantic block.
As a result, the following code is illegal and cause the compiler to throw an error, ``Identifier already declared``::

	//这不会编译
    // This will not compile

    pragma solidity ^0.4.16;

    contract ScopingErrors {
        function scoping() public {
            uint i = 0;

            while (i++ < 1) {
                uint same1 = 0;
            }

            while (i++ < 2) {
							   // 非法，same1的第二次声明
                uint same1 = 0;// Illegal, second declaration of same1
            }
        }

        function minimalScoping() public {
            {
                uint same2 = 0;
            }

            {
							   // 非法，same2的第二次声明
                uint same2 = 0;// Illegal, second declaration of same2
            }
        }

        function forLoopScoping() public {
            for (uint same3 = 0; same3 < 1; same3++) {
            }
													  // 非法，same2的第二次声明
            for (uint same3 = 0; same3 < 1; same3++) {// Illegal, second declaration of same3
            }
        }
    }

除此之外，如果声明了一个变量，它将在函数的开头初始化为其默认值。因此，下面的代码是合法的，尽管这种写法不推荐::
In addition to this, if a variable is declared, it will be initialized at the beginning of the function to its default value.
As a result, the following code is legal, despite being poorly written::

    pragma solidity ^0.4.0;

    contract C {
        function foo() public pure returns (uint) {
			// baz被隐式初始化为0
            // baz is implicitly initialized as 0
            uint bar = 5;
            if (true) {
                bar += baz;
            } else {		  // 不会执行
                uint baz = 10;// never executes
            }
            return bar;// returns 5
        }
    }

.. index:: ! exception, ! throw, ! assert, ! require, ! revert
.. index:: ! exception, ! throw, ! assert, ! require, ! revert

错误处理：Assert, Require, Revert and Exceptions
Error handling: Assert, Require, Revert and Exceptions
======================================================

Solidity使用状态恢复异常来处理错误。这种异常将撤消对当前调用（及其所有子调用）中的状态所做的所有更改，并且还向调用者标记错误。
便利函数``assert``和``require``可用于检查条件并在条件不满足时抛出异常。``assert``函数只能用于测试内部错误，并检查非变量。
``require``函数用于确认条件有效性，例如输入变量，或合约状态变量是否满足条件，或验证外部合约调用返回的值。
如果使用得当，分析工具可以评估您的合约，以确定条件和函数调用是否合理，工具对函数的调用可能会触发失败的``assert``。
正确运行的代码不应该运行到失败的断言语句; 如果发生这种情况，您应该找到您合约中的错误并修正。
Solidity uses state-reverting exceptions to handle errors. Such an exception will undo all changes made to the
state in the current call (and all its sub-calls) and also flag an error to the caller.
The convenience functions ``assert`` and ``require`` can be used to check for conditions and throw an exception
if the condition is not met. The ``assert`` function should only be used to test for internal errors, and to check invariants.
The ``require`` function should be used to ensure valid conditions, such as inputs, or contract state variables are met, or to validate return values from calls to external contracts.
If used properly, analysis tools can evaluate your contract to identify the conditions and function calls which will reach a failing ``assert``. 
Properly functioning code should never reach a failing assert statement; if this happens there is a bug in your contract which you should fix.

还有另外两种触发异常的方法：``revert``函数可以用来标记错误并恢复当前的调用。
将来还可能在``revert``调用中包含有关错误的详细信息。``throw``关键字也可以用来替代``revert（）``。
There are two other ways to trigger exceptions: The ``revert`` function can be used to flag an error and
revert the current call. In the future it might be possible to also include details about the error
in a call to ``revert``. The ``throw`` keyword can also be used as an alternative to ``revert()``.

.. 注意::
.. note::
	从0.4.13版本开始，``throw``这个关键字被弃用，并且将来会被逐渐淘汰。
    From version 0.4.13 the ``throw`` keyword is deprecated and will be phased out in the future.

当子调用发生异常时，它们会自动“冒泡”（即重新抛出异常）。这个规则的例外是``send``和低级函数``call``, ``delegatecall`` 和 ``callcode``--如果这些函数发生异常，将返回false，而不是“冒泡”。
When exceptions happen in a sub-call, they "bubble up" (i.e. exceptions are rethrown) automatically. Exceptions to this rule are ``send``
and the low-level functions ``call``, ``delegatecall`` and ``callcode`` -- those return ``false`` in case
of an exception instead of "bubbling up".

.. 警告::
.. warning::
	
	作为EVM设计的一部分，如果被调用合约帐户不存在，则低级别的``call``, ``delegatecall`` 和 ``callcode``将返回success。因此如果需要使用低级函数时，必须在调用之前检查被调用合约是否存在。
    The low-level ``call``, ``delegatecall`` and ``callcode`` will return success if the called account is non-existent, as part of the design of EVM. Existence must be checked prior to calling if desired.

捕捉异常还未实现。
Catching exceptions is not yet possible.

在下面的例子中，你可以看到如何使用``require``方便地检查输入条件以及``assert``如何用于内部错误检查::
In the following example, you can see how ``require`` can be used to easily check conditions on inputs
and how ``assert`` can be used for internal error checking::

    pragma solidity ^0.4.0;

    contract Sharer {
        function sendHalf(address addr) public payable returns (uint balance) {
										 //只允许偶数
            require(msg.value % 2 == 0); // Only allow even numbers
            uint balanceBeforeTransfer = this.balance;
            addr.transfer(msg.value / 2);
			//由于转移函数在失败时抛出异常并且不能在这里回调，因此我们应该没有办法仍然有一半的钱。
            // Since transfer throws an exception on failure and
            // cannot call back here, there should be no way for us to
            // still have half of the money.
            assert(this.balance == balanceBeforeTransfer - msg.value / 2);
            return this.balance;
        }
    }

下列情况将会产生一个``assert``式异常：
An ``assert``-style exception is generated in the following situations:

#. 如果你访问数组的索引太大或为负数（即``x [i]``，其中``i> = x.length``或``i<0``）。
#. 如果你访问固定长度``bytesN``的索引太大或为负数。
#. 如果你用零当除数做除法或模运算（例如``5 / 0``或``23％0``）。
#. 如果你移位负数位。
#. 如果你将一个太大或负数值转换为一个枚举类型。
#. 如果你调用内部函数类型的零初始化变量。
#. 如果你调用``assert``函数的参数值为false。
#. If you access an array at a too large or negative index (i.e. ``x[i]`` where ``i >= x.length`` or ``i < 0``).
#. If you access a fixed-length ``bytesN`` at a too large or negative index.
#. If you divide or modulo by zero (e.g. ``5 / 0`` or ``23 % 0``).
#. If you shift by a negative amount.
#. If you convert a value too big or negative into an enum type.
#. If you call a zero-initialized variable of internal function type.
#. If you call ``assert`` with an argument that evaluates to false.

下列情况将会产生一个``require``式异常：
A ``require``-style exception is generated in the following situations:

#. 调用``throw``。
#. 调用``require``函数的参数值为``false``。
#. 如果你通过消息调用调用某个函数，但该函数没有正确结束（例如，它耗尽了gas，没有匹配函数，或者本身抛出一个异常），上述函数不包括低级别的操作``call``, ``send``, ``delegatecall`` 或者 ``callcode``。低级操作不会抛出异常，而通过返回“false”来指示失败。
#. 如果你使用``new``关键字创建合约，但合约没有正确创建（请参阅上条有关”未正确完成“的定义）。
#. 如果你对不包含代码的合约执行外部函数调用。
#. 如果你的合约通过一个没有``payable``修饰符的公有函数（包括构造函数和回退函数）接收Ether。
#. 如果你的合约通过公有getter函数接收Ether。
#. 如果``.transfer（）``失败。
#. Calling ``throw``.
#. Calling ``require`` with an argument that evaluates to ``false``.
#. If you call a function via a message call but it does not finish properly (i.e. it runs out of gas, has no matching function, or throws an exception itself), except when a low level operation ``call``, ``send``, ``delegatecall`` or ``callcode`` is used.  The low level operations never throw exceptions but indicate failures by returning ``false``.
#. If you create a contract using the ``new`` keyword but the contract creation does not finish properly (see above for the definition of "not finish properly").
#. If you perform an external function call targeting a contract that contains no code.
#. If your contract receives Ether via a public function without ``payable`` modifier (including the constructor and the fallback function).
#. If your contract receives Ether via a public getter function.
#. If a ``.transfer()`` fails.

在内部，Solidity对一个``require``式的异常执行回退操作（指令``0xfd``）并执行一个无效操作（指令``0xfe``）来引发``assert``式异常。
在这两种情况下，都会导致EVM回退对该状态所做的所有更改。回退的原因是不能继续安全地执行，因为没有实现预期的效果。
因为我们想保留交易的原子性，所以最安全的做法是回退所有更改并使整个交易（或至少是调用）无效。
请注意，``assert``式异常消耗了所有可用的调用gas，而从Metropolis版本起``require``式的异常不会消耗任何gas。
Internally, Solidity performs a revert operation (instruction ``0xfd``) for a ``require``-style exception and executes an invalid operation
(instruction ``0xfe``) to throw an ``assert``-style exception. In both cases, this causes
the EVM to revert all changes made to the state. The reason for reverting is that there is no safe way to continue execution, because an expected effect
did not occur. Because we want to retain the atomicity of transactions, the safest thing to do is to revert all changes and make the whole transaction
(or at least call) without effect. Note that ``assert``-style exceptions consume all gas available to the call, while
``require``-style exceptions will not consume any gas starting from the Metropolis release.
