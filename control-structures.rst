##################################
表达式和控制结构
##################################


.. index:: if, else, while, do/while, for, break, continue, return, switch, goto

控制结构
===================

JavaScript 中的大部分控制结构在 Solidity 中都是可用的，除了 ``switch`` 和 ``goto``。
因此 Solidity 中有 ``if``， ``else``， ``while``， ``do``， ``for``， ``break``， ``continue``， ``return``， ``? :`` 这些与在 C 或者 JavaScript 中表达相同语义的关键词。

Solidity还支持 ``try``/ ``catch`` 语句形式的异常处理，但仅用于 :ref:`外部函数调用 <external-function-calls>`　和合约创建调用。
使用:ref:`revert 语句 <revert-statement>` 可以触发一个"错误"。

用于表示条件的括号 *不可以* 被省略，单语句体两边的花括号可以被省略。


注意，与 C 和 JavaScript 不同， Solidity 中非布尔类型数值不能转换为布尔类型，因此 ``if (1) { ... }`` 的写法在 Solidity 中 *无效* 。



.. index:: ! function;call, function;internal, function;external

.. _function-calls:

函数调用
==============

.. _internal-function-calls:

内部函数调用
-----------------------

当前合约中的函数可以直接（“从内部”）调用，也可以递归调用，就像下边这个无意义的例子一样。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

    // 编译器会有警告提示
    contract C {
        function g(uint a) public pure returns (uint ret) { return f(); }
        function f() internal pure returns (uint ret) { return g(7) + f(); }
    }

这些函数调用在 EVM 中被解释为简单的跳转。这样做的效果就是当前内存不会被清除，例如，函数之间通过传递内存引用进行内部调用是非常高效的。
只能在同一合约实例的函数，可以进行内部调用。

只有在同一合约的函数可以内部调用。仍然应该避免过多的递归调用, 因为每个内部函数调用至少使用一个堆栈槽, 并且最多有1024堆栈槽可用。

.. _external-function-calls:

外部函数调用
-----------------------

方式也可以使用表达式 ``this.g(8);`` 和 ``c.g(2);`` 进行调用，其中 ``c`` 是合约实例， ``g`` 合约内实现的函数，但是这两种方式调用函数，称为“外部调用”，它是通过消息调用来进行，而不是直接的代码跳转。
请注意，不可以在构造函数中通过 this 来调用函数，因为此时真实的合约实例还没有被创建。


如果想要调用其他合约的函数，需要外部调用。对于一个外部调用，所有的函数参数都需要被复制到内存。

.. note::
    从一个合约到另一个合约的函数调用不会创建自己的交易, 它是作为整个交易的一部分的消息调用。

当调用其他合约的函数时，需要在函数调用是指定发送的 Wei 和 gas 数量，可以使用特定选项　``{value: 10, gas: 10000}``

请注意，不建议明确指定gas，因为操作码的 gas 消耗将来可能会发生变化。
任何发送给合约 Wei  将被添加到目标合约的总余额中：

.. code-block:: solidity

    pragma solidity >=0.6.2 <0.9.0;

    contract InfoFeed {
        function info() public payable returns (uint ret) { return 42; }
    }

    contract Consumer {
        InfoFeed feed;
        function setFeed(InfoFeed addr) public { feed = addr; }
        function callFeed() public { feed.info{value: 10, gas: 800}(); }
    }

``payable`` 修饰符要用于修饰 ``info`` 函数，否则， ``value`` 选项将不可用。

.. warning::
  注意 ``feed.info{value: 10, gas: 800}`` 仅（局部地）设置了与函数调用一起发送的 ``Wei`` 值和 ``gas`` 的数量，只有最后的小括号才执行了真正的调用。
  因此， ``feed.info{value: 10, gas: 800}`` 是没有调用函数的， ``value`` 和 ``gas`` 设置是无效的。

由于EVM认为可以调用不存在的合约的调用，因此在 Solidity 语言层面里会使用 ``extcodesize`` 操作码来检查要调用的合约是否确实存在（包含代码），如果不存在该合约，则抛出异常。

如果返回数据在调用后被解码，则跳过这个检查，因此ABI解码器将捕捉到不存在的合约的情况。

请注意，这个检查在 :ref:`低级别调用<address_related>` 时不被执行，这些调用是对地址而不是合约实例进行操作。

.. note::
    
    当使用高级别的方式调用 :ref:`预编译合约时 <precompiledContracts>` 也需要注意，因为因为根据上面的逻辑，编译器认为它们不存在，即使它们执行代码并返回数据。

如果被调用合约本身抛出异常或者 gas 用完等，函数调用也会抛出异常。


.. warning::

    任何与其他合约的交互都会产生潜在危险，尤其是在不能预先知道合约代码的情况下。
    交互时当前合约会将控制权移交给被调用合约，而被调用合约可能做任何事。即使被调用合约从一个已知父合约继承，继承的合约也只需要有一个正确的接口就可以了。
    被调用合约的实现可以完全任意的实现，因此会带来危险。
    此外，请小心这个交互调用在返回之前再回调我们的合约，这意味着被调用合约可以通过它自己的函数改变调用合约的状态变量。
    一个建议的函数写法是，例如，在合约中状态变量进行各种变化后再调用外部函数，这样，你的合约就不会轻易被滥用的重入攻击 (reentrancy) 所影响

.. note::
    在Solidity 0.6.2之前，建议指定余额和gas的方法是使用 ``f.value(x).gas(g)()`` 。
    这个方式在0.6.2时被弃用，在Solidity 0.7.0中开始不再允许使用。


具名调用和匿名函数参数
---------------------------------------------

函数调用参数也可以按照任意顺序由名称给出，如果它们被包含在 ``{ }`` 中，
如以下示例中所示。参数列表必须按名称与函数声明中的参数列表相符，但可以按任意顺序排列。

.. code-block:: solidity

    pragma solidity >=0.4.0 <0.9.0;

    contract C {
        mapping(uint => uint) data;

        function f() public {
            set({value: 2, key: 3});
        }

        function set(uint key, uint value) public {
            data[key] = value;
        }

    }

省略函数参数名称
--------------------------------

未使用参数的名称（特别是返回参数）可以省略。这些参数仍然存在于堆栈中，但它们无法访问。

.. code-block:: solidity

    pragma solidity >=0.4.22 <0.9.0;

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

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;

    contract D {
        uint x;
        function D(uint a) payable {
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
            D newD = (new D){value:amount}(arg);
        }
    }

如示例中所示，通过使用 ``value`` 选项创建 ``D`` 的实例时可以附带发送 Ether，但是不能限制 gas 的数量。
如果创建失败（可能因为栈溢出，或没有足够的余额或其他问题），会引发异常。


加“盐”的合约创建 / create2
-----------------------------------

在创建合约时，将根据创建合约的地址和每次创建合约交易时的计数器(nonce)来计算合约的地址。

如果你指定了一个可选的 ``salt`` （一个bytes32值），那么合约创建将使用另一种机制(create2)来生成新合约的地址：

它将根据给定的盐值，创建合约的字节码和构造函数参数来计算创建合约的地址。


特别注意，这里不再使用计数器（“nonce”）。 这样可以在创建合约时提供更大的灵活性：你可以在创建新合约之前就推导出（将要创建的）合约地址。 
甚至是，还可以依赖此地址（即便它还不存在）来创建其他合约。一个主要用例场景是充当链下交互仲裁合约，仅在有争议时才需要创建。


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.7.0;

    contract D {
        uint public x;
        constructor(uint a) {
            x = a;
        }
    }

    contract C {
        function createDSalted(bytes32 salt, uint arg) public {
            /// 这个复杂的表达式只是告诉我们，如何预先计算地址。
            /// 这里仅仅用来说明。
            /// 实际上，你仅仅需要 ``new D{salt: salt}(arg)``.
            address predictedAddress = address(uint160(uint(keccak256(abi.encodePacked(
                bytes1(0xff),
                address(this),
                salt,
                keccak256(abi.encodePacked(
                    type(D).creationCode,
                    arg
                ))
            )))));

            D d = new D{salt: salt}(arg);
            require(address(d) == predictedAddress);
        }
    }

.. warning::

    使用 create2 创建合约还有一些特别之处。 合约销毁后可以在同一地址重新创建。不过，即使创建字节码（creation bytecode）相同（这是要求，因为否则地址会发生变化），该新创建的合约也可能有不同的部署字节码（deployed bytecode）。 
    这是因为构造函数可以使用两次创建合约之间可能已更改的外部状态，并在存储合约时将其合并到部署字节码中。



表达式计算顺序
==================================

表达式的计算顺序不是特定的（更准确地说，表达式树中某节点的字节点间的计算顺序不是特定的，但它们的结算肯定会在节点自己的结算之前）。该规则只能保证语句按顺序执行，布尔表达式的短路执行。


.. index:: ! assignment

赋值
==========

.. index:: ! assignment;destructuring

解构赋值和返回多值
-------------------------------------------------------

Solidity 内部允许元组 (tuple) 类型，也就是一个在编译时元素数量固定的对象列表，列表中的元素可以是不同类型的对象。这些元组可以用来同时返回多个数值，也可以用它们来同时给多个新声明的变量或者既存的变量（或通常的 LValues）：

.. code-block:: solidity

    pragma solidity >=0.5.0 <0.9.0;

    contract C {
        uint index;

        function f() public pure returns (uint, bool, uint) {
            return (7, true, 2);
        }

        function g() public {
            //基于返回的元组来声明变量并赋值
            (uint x, bool b, uint y) = f();
            //交换两个值的通用窍门——但不适用于非值类型的存储 (storage) 变量。
            (x, y) = (y, x);
            //元组的末尾元素可以省略（这也适用于变量声明）。
            (index,,) = f(); // 设置 index 为 7
        }
    }


不可能混合变量声明和非声明变量复制, 即以下是无效的: ``(x, uint y) = (1, 2);``

.. note::
    在  0.5.0 版本之前，给具有更少的元素数的元组赋值都可以可能的，无论是在左边还是右边（比如在最后空出若干元素）。现在，这已经不允许了，赋值操作的两边应该具有相同个数的组成元素。

.. warning::
    当涉及引用类型时，在同时分配给多个变量时要小心, 因为这可能会导致意外的复制行为。


数组和结构体的复杂性
------------------------------------
赋值语义对于像数组和结构体(包括 ``bytes`` 和 ``string``) 这样的非值类型来说会有些复杂。


参考 :ref:`数据位置及赋值行为 <data-location-assignment>` 了解更多 。

在下面的示例中, 对 ``g(x)`` 的调用对 ``x`` 没有影响, 因为它在内存中创建了存储值独立副本。但是, ``h(x)`` 成功修改 ``x`` , 因为只传递引用而不传递副本。


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.22 <0.9.0;

     contract C {
        uint[20] x;

         function f() public {
            g(x);
            h(x);
        }

         function g(uint[20] memory y) internal pure {
            y[2] = 3;
        }

         function h(uint[20] storage y) internal {
            y[3] = 4;
        }
    }

.. index:: ! scoping, declarations, default value

.. _default-value:

作用域和声明
========================
变量声明后将有默认初始值，其初始值字节表示全部为零。任何类型变量的“默认值”是其对应类型的典型“零状态”。例如， ``bool`` 类型的默认值是 ``false`` 。
``uint`` 或 ``int`` 类型的默认值是 ``0`` 。对于静态大小的数组和 ``bytes1`` 到 ``bytes32`` ，每个单独的元素将被初始化为与其类型相对应的默认值。
最后，对于动态大小的数组 ``bytes`` 和 ``string`` 类型，其默认缺省值是一个空数组或空字符串。

对于 ``enum`` 类型, 默认值是第一个成员。

Solidity 中的作用域规则遵循了 C99（与其他很多语言一样）：变量将会从它们被声明之后可见，直到一对 ``{ }`` 块的结束。作为一个例外，在 for 循环语句中初始化的变量，其可见性仅维持到 for 循环的结束。

对于参数形式的变量（例如：函数参数、修饰器参数、catch参数等等）在其后接着的代码块内有效。
这些代码块是函数的实现，catch 语句块等。


那些定义在代码块之外的变量，比如函数、合约、自定义类型等等，并不会影响它们的作用域特性。这意味着你可以在实际声明状态变量的语句之前就使用它们，并且递归地调用函数。

基于以上的规则，下边的例子不会出现编译警告，因为那两个变量虽然名字一样，但却在不同的作用域里。

.. code-block:: solidity

    pragma solidity >=0.5.0 <0.9.0;
    contract C {
        function minimalScoping() pure public {
            {
                uint same;
                same = 1;
            }

            {
                uint same;
                same = 3;
            }
        }
    }

作为 C99 作用域规则的特例，请注意在下边的例子里，第一次对 ``x`` 的赋值会改变上一层中声明的变量值。如果外层声明的变量被“覆盖”（就是说被在内部作用域中由一个同名变量所替代）你会得到一个警告。

.. code-block:: solidity

    pragma solidity >=0.5.0 <0.9.0;
    // 有警告
    contract C {
        function f() pure public returns (uint) {
            uint x = 1;
            {
                x = 2; // 这个赋值会影响在外层声明的变量
                uint x;
            }
            return x; // x has value 2
        }
    }

.. warning::
    在 Solidity 0.5.0 之前的版本，作用域规则都沿用了 Javascript 的规则，即一个变量可以声明在函数的任意位置，都可以使他在整个函数范围内可见。而这种规则会从 0.5.0 版本起被打破。从 0.5.0 版本开始，下面例子中的代码段会导致编译错误。

.. code-block:: solidity

    // 这将无法编译通过

    pragma solidity >=0.5.0 <0.9.0;
    contract C {
        function f() pure public returns (uint) {
            x = 2;
            uint x;
            return x;
        }
    }


.. index:: ! safe math, safemath, checked, unchecked
.. _unchecked:

算术运算的检查模式与非检查模式
=================================

当对无限制整数执行算术运算，其结果超出结果类型的范围，这是就发生了上溢出或下溢出。

在Solidity 0.8.0之前，算术运算总是会在发生溢出的情况下进行“截断”，从而得靠引入额外检查库来解决这个问题（如 OpenZepplin 的 SafeMath）。

而从Solidity 0.8.0开始，所有的算术运算默认就会进行溢出检查，额外引入库将不再必要。

如果想要之前“截断”的效果，可以使用 ``unchecked`` 代码块：


.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.0;
    contract C {
        function f(uint a, uint b) pure public returns (uint) {
            // 减法溢出会返回“截断”的结果
            unchecked { return a - b; }
        }
        function g(uint a, uint b) pure public returns (uint) {
            // 溢出会抛出异常
            return a - b;
        }
    }

调用 ``f(2, 3)`` 将返回 ``2**256-1``, 而 ``g(2, 3)`` 会触发失败异常。


``unchecked`` 代码块可以在代码块中的任何位置使用，但不可以替代整个函数代码块，同样不可以嵌套。

此设置仅影响语法上位于 ``unchecked`` 块内的语句。
在块中调用的函数不会此影响。

.. note::
    为避免歧义，不能在 ``unchecked`` 块中使用 `` _;`` 。

下面的这个运算操作符会进行溢出检查，如果上溢出或下溢会触发失败异常。
如果在非检查模式代码块中使用，将不会出现错误:


``++``, ``--``, ``+``, 减 ``-``,  负 ``-``, ``*``, ``/``, ``%``, ``**``

``+=``, ``-=``, ``*=``, ``/=``, ``%=``

.. warning::
    除 0（或除 0取模）的异常是不能被 ``unchecked`` 忽略的。

.. note::

    位运算不会执行上溢或下溢检查。
    这在使用位移位(``<<``, ``>>``, ``<<=``, ``>>=``)来代替整数除法和2指数时尤其明显。
    例如 ``type(uint256).max << 3`` 不会回退，而 ``type(uint256).max * 8`` 会失败回退。

.. note::
    ``int x = type(int).min; -x;`` 中的第 2 句会溢出，因为负数的范围比正整数的范围大 1（译者注：这样最小的负数就没有对应的正整数了） 。


显式类型转换将始终截断并且不会导致失败的断言，但是从整数到枚举类型的转换例外。

.. index:: ! exception, ! throw, ! assert, ! require, ! revert, ! errors

.. _assert-and-require:

错误处理及异常：Assert, Require, Revert
======================================================

Solidity 使用状态恢复异常来处理错误。这种异常将撤消对当前调用（及其所有子调用）中的状态所做的所有更改，并且还向调用者标记错误。

如果异常在子调用发生，那么异常会自动冒泡到顶层（例如：异常会重新抛出），除非他们在 ``try/catch`` 语句中捕获了错误。
但是如果是在 ``send`` 和 低级别如： ``call``, ``delegatecall`` 和 ``staticcall`` 的调用里发生异常时， 他们会返回 ``false`` （第一个返回值） 而不是冒泡异常。 

.. warning::
    注意：根据 EVM 的设计，如果被调用的地址不存在，低级别函数 ``call``, ``delegatecall`` 和 ``staticcall`` 也或第一个返回值同样是 ``true``。
    如果需要，请在调用之前检查账号的存在性。


异常可以包含错误数据，以 :ref:`error 示例 <errors>` 的形式传回给调用者。
内置的错误 ``Error(string)`` 和 ``Panic(uint256)`` 被作为特殊函数使用，下面将解释。
``Error`` 用于 "常规" 错误条件，而 ``Panic`` 用于在（无bug）代码中不应该出现的错误。



用 ``assert`` 检查异常(Panic) 和 ``require`` 检查错误(Error)
----------------------------------------------------------------

函数 ``assert`` 和 ``require`` 可用于检查条件并在条件不满足时抛出异常。

``assert`` 函数会创建一个 ``Panic(uint256)`` 类型的错误。
同样的错误在以下列出的特定情形会被编译器创建。

``assert`` 函数应该只用于测试内部错误，检查不变量，正常的函数代码永远不会产生Panic, 甚至是基于一个无效的外部输入时。
如果发生了，那就说明出现了一个需要你修复的 bug。如果使用得当，语言分析工具可以识别出那些会导致 Panic 的 ``assert`` 条件和函数调用。

下列情况将会产生一个Panic异常：
错误数据会提供的错误码编号，用来指示Panic的类型：

#. 0x00: 用于常规编译器插入的Panic。
#. 0x01: 如果你调用 ``assert`` 的参数（表达式）结果为 false 。
#. 0x11: 在 ``unchecked { ... }`` 外，如果算术运算结果向上或向下溢出。
#. 0x12; 如果你用零当除数做除法或模运算（例如 ``5 / 0`` 或 ``23 % 0`` ）。
#. 0x21: 如果你将一个太大的数或负数值转换为一个枚举类型。
#. 0x22: 如果你访问一个没有正确编码的存储byte数组.
#. 0x31: 如果在空数组上 ``.pop()`` 。
#. 0x32: 如果你访问 ``bytesN`` 数组（或切片）的索引太大或为负数。(例如： ``x[i]`` 而 ``i >= x.length`` 或 ``i < 0``).
#. 0x41: 如果你分配了太多的内内存或创建了太大的数组。
#. 0x51: 如果你调用了零初始化内部函数类型变量。


 ``require`` 函数可以创建无错误提示的错误，也可以创建一个 ``Error(string)`` 类型的错误。 ``require`` 函数应该用于确认条件有效性，例如输入变量，或合约状态变量是否满足条件，或验证外部合约调用返回的值。

.. note::

    当前不可以使用混合使用 require 和自定义错误，而是需要使用  ``if (!condition) revert CustomError();``  。

下列情况将会产生一个 ``Error(string)`` （或无错误提示）的错误：


#. 如果你调用 ``require(x)`` ，而 ``x`` 结果为 ``false`` 。
#. 如果你使用 ``revert()`` 或者 ``revert("description")`` 。
#. 如果你在不包含代码的合约上执行外部函数调用。
#. 如果你通过合约接收以太币，而又没有 ``payable`` 修饰符的公有函数（包括构造函数和 fallback 函数）。
#. 如果你的合约通过公有 getter 函数接收 Ether 。

在下面的情况下，来自外部调用的错误数据（如果提供的话）被转发，这意味可能 `Error` 或 `Panic` 都有可能触发。

#. 如果 ``.transfer()`` 失败。 
#. 如果你通过消息调用调用某个函数，但该函数没有正确结束（例如, 它耗尽了 gas，没有匹配函数，或者本身抛出一个异常），不包括使用低级别 ``call`` ， ``send`` ， ``delegatecall`` ， ``callcode`` 或  ``staticcall`` 的函数调用。低级操作不会抛出异常，而通过返回 ``false`` 来指示失败。
#. 如果你使用 ``new`` 关键字创建合约，但合约创建 :ref:`没有正确结束<creating-contracts>` 。


你可以选择给 ``require`` 提供一个消息字符串，但 ``assert`` 不行。

.. note::
    
    如果你没有为 ``require`` 提供一个字符串参数，它会用空错误数据进行 revert， 甚至不包括错误选择器。

在下例中，你可以看到如何轻松使用 ``require`` 检查输入条件以及如何使用 ``assert`` 检查内部错误.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.5.0 <0.9.0;

    contract Sharer {
        function sendHalf(address addr) public payable returns (uint balance) {
            require(msg.value % 2 == 0, "Even value required.");
            uint balanceBeforeTransfer = this.balance;
            addr.transfer(msg.value / 2);
			
            // 由于转账函数在失败时抛出异常并且不会调用到以下代码，因此我们应该没有办法检查仍然有一半的钱。
            assert(this.balance == balanceBeforeTransfer - msg.value / 2);
            return this.balance;
        }
    }


在内部， Solidity 对异常执行回退操作（指令 ``0xfd`` ），从而让 EVM 回退对状态所做的所有更改。回退的原因是无法安全地继续执行，因为无法达到预期的结果。
因为我们想要保持交易的原子性，最安全的动作是回退所有的更改，并让整个交易（或至少调用）没有任何新影响。

在这两种情况下，调用者都可以使用 ``try``/ ``catch`` 来应对此类失败，但是被调用函数的更改将始终被还原。

.. note::

  请注意， 在0.8.0 之前，Panic异常使用 ``invalid`` 指令，其会消耗了所有可用的 gas。
  使用 ``require`` 的异常，在 Metropolis 版本之前会消耗所有的 gas。


.. _revert-statement:

``revert``
----------

可以使用 ``revert`` 语句和 ``revert`` 函数来直接触发回退。

``revert`` 语句将一个自定义的错误作为直接参数，没有括号：

    revert CustomError(arg1, arg2);

由于向后兼容，还有一个 ``revert()`` 函数，它使用圆括号接受一个字符串：

    revert();
    revert("description");

错误数据将被传回给调用者，以便在那里捕获到错误数据。
使用 ``revert()`` 会触发一个没有任何错误数据的回退，而 ``revert("description")`` 会产生一个 ``Error(string)`` 错误。


使用一个自定义的错误实例通常会比字符串描述便宜得多。因为你可以使用错误名来描述它，它只被编码为四个字节。更长的描述可以通过NatSpec提供，这不会产生任何费用。

下面的例子显示了如何使用一个错误字符串和一个自定义错误实例，他们和 ``revert`` 或相应的 ``require`` 一起使用。

.. code-block:: solidity

    contract VendingMachine {
        address owner;
        error Unauthorized();
        function buy(uint amount) public payable {
            if (amount > msg.value / 2 ether)
                revert("Not enough Ether provided.");
            // 另一个可选的方式:
            require(
                amount <= msg.value / 2 ether,
                "Not enough Ether provided."
            );

            // 以下执行购买逻辑
        }
        function withdraw() public {
            if (msg.sender != owner)
                revert Unauthorized();

            payable(msg.sender).transfer(address(this).balance);
        }
    }


只要参数没有额外的附加效果，使用 ``if (!condition) revert(...);`` 和 ``require(condition, ...);`` 是等价的，例如当参数是字符串的情况。


.. note::
    ``require`` 是一个像其他函数一样可被执行的函数。
    意味着，所有的参数在函数被执行之前就都会被执行。
    尤其，在 ``require(condition, f())`` 里，函数 ``f`` 会被执行，即便 ``condition`` 为 True .

如果是调用 ``Error(string)`` 函数，这里提供的字符串将经过 :ref:`ABI 编码 <ABI>` 。
在上边的例子里， ``revert("Not enough Ether provided.");`` 会产生如下的十六进制错误返回值：

.. code::

    0x08c379a0                                                         // Error(string) 的函数选择器
    0x0000000000000000000000000000000000000000000000000000000000000020 // 数据的偏移量（32）
    0x000000000000000000000000000000000000000000000000000000000000001a // 字符串长度（26）
    0x4e6f7420656e6f7567682045746865722070726f76696465642e000000000000 // 字符串数据（"Not enough Ether provided." 的 ASCII 编码，26字节）

提示信息可以通过 ``try/catch`` （下面介绍）来获取到。

.. note::
    ``revert()``之前有一个同样用法的 ``throw`` ，它在0.4.13版本弃用，在0.5.0移除。


.. _try-catch:

``try/catch``
---------------------

外部调用的失败，可以通过  try/catch 语句来捕获，例如：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.8.1;

    interface DataFeed { function getData(address token) external returns (uint value); }

    contract FeedConsumer {
        DataFeed feed;
        uint errorCount;
        function rate(address token) public returns (uint value, bool success) {
            // 如果错误超过 10 次，永久关闭这个机制
            require(errorCount < 10);
            try feed.getData(token) returns (uint v) {
                return (v, true);
            } catch Error(string memory /*reason*/) {
                // This is executed in case
                // revert was called inside getData
                // and a reason string was provided.
                errorCount++;
                return (0, false);
            }  catch Panic(uint /*errorCode*/) {
                // This is executed in case of a panic,
                // i.e. a serious error like division by zero
                // or overflow. The error code can be used
                // to determine the kind of error.
                errorCount++;
                return (0, false);
            } catch (bytes memory /*lowLevelData*/) {
                // This is executed in case revert() was used。
                errorCount++;
                return (0, false);
            }
        }
    }

``try`` 关键词后面必须有一个表达式，代表外部函数调用或合约创建（ ``new ContractName()``）。

在表达式上的错误不会被捕获（例如，如果它是一个复杂的表达式，还涉及内部函数调用），只有外部调用本身发生的revert 可以捕获。
接下来的 ``returns`` 部分（是可选的）声明了与外部调用返回的类型相匹配的返回变量。
在没有错误的情况下，这些变量被赋值，合约将继续执行第一个成功块内代码。
如果到达成功块的末尾，则在 ``catch`` 块之后继续执行。

Solidity 根据错误的类型，支持不同种类的捕获代码块：


- ``catch Error(string memory reason) { ... }``: 如果错误是由 ``revert("reasonString")`` 或 ``require(false, "reasonString")`` （或导致这种异常的内部错误）引起的，则执行这个catch子句。

- ``catch Panic(uint errorCode) { ... }``: 如果错误是由 panic 引起的（如： ``assert`` 失败，除以0，无效的数组访问，算术溢出等），将执行这个catch子句。

- ``catch (bytes memory lowLevelData) { ... }``: 如果错误签名不符合任何其他子句，如果在解码错误信息时出现了错误，或者如果异常没有一起提供错误数据。在这种情况下，子句声明的变量提供了对低级错误数据的访问。

- ``catch { ... }``: 如果你对错误数据不感兴趣，你可以直接使用 ``catch { ... }`` (甚至是作为唯一的catch子句) 而不是前面几个catch子句。


有计划在未来支持其他类型的错误数据。
``Error`` 和 ``Panic`` 字符串目前是按原样解析的，不作为标识符处理。

为了捕捉所有的错误情况，你至少要有子句 ``catch { ... }`` 或 ``catch (bytes memory lowLevelData) { ... }``.

在 ``returns`` 和 ``catch`` 子句中声明的变量只在后面的块的范围内有效。


.. note::

    如果在 try/catch 语句内部返回的数据解码过程中发生错误，这将导致当前执行的合约出现异常，如此，它不会在catch子句中被捕获到。
    如果在 ``catch Error(string memory reason)`` 的解码过程中出现错误，并且有一个低级的catch子句，那么这个错误就会在低级catch子句被捕获。

.. note::
    
    如果执行到一个catch子句，那么外部调用的状态改变已经被回退了。
    如果执行到了成功块，那么外部调用的状态改变是有效的。
    如果状态改变已经被回退，那么要么在catch块中继续执行，要么是try/catch语句的执行本身被回退（例如由于上面提到的解码失败或由于没有提供低级别的catch子句时）。

.. note::

    调用失败背后的原因可能是多方面的。请不要认为错误信息是直接来自被调用的合约。
    错误可能发生在调用链的更深处，而被调用的合约只是转发了（冒泡）错误。
    另外，这可能是由于 out-of-gas 情况，而不是一个逻辑错误状况：
    调用者总是在调用中保留至少1/64的gas，这样即使被调合约gas用完，调用方仍有一些gas预留（处理剩余逻辑）。
