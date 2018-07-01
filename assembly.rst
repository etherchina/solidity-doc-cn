#################
Solidity汇编
#################

.. index:: ! assembly, ! asm, ! evmasm

Solidity定义了一种汇编语言，在没有Solidity的情况下也可以使用。这种汇编语言也可以嵌入到Solidity源代码中当作“内联汇编”使用。我们从如何使用内联汇编开始，介绍它如何区别于独立汇编语言，然后详细讲述这种汇编语言。

.. _inline-assembly:

内联汇编
===============

为了实现更细粒度的控制，尤其是为了通过编写库来增强语言，可以利用接近虚拟机的语言将内联汇编中与Solidity语句结合在一起使用。由于EVM是堆栈机，因此通常很难正确定位堆栈插槽的地址，并为堆栈上正确的操作码提供参数。Solidity的内联汇编试图提供以下功能，解决手写汇编代码时出现的问题和其他问题：

* 函数式操作码： ``mul(1, add(2, 3))`` 而不是 ``push1 3 push1 2 add push1 1 mul``
* 汇编局部变量： ``let x := add(2, 3)  let y := mload(0x40)  x := add(x, y)``
* 读取外部变量： ``function f(uint x) public { assembly { x := sub(x, 1) } }``
* 标签： ``let x := 10  repeat: x := sub(x, 1) jumpi(repeat, eq(x, 0))``
* 循环： ``for { let i := 0 } lt(i, x) { i := add(i, 1) } { y := mul(2, y) }``
* if 语句： ``if slt(x, 0) { x := sub(0, x) }``
* switch 语句： ``switch x case 0 { y := mul(x, 2) } default { y := 0 }``
* 函数调用： ``function f(x) -> y { switch x case 0 { y := 1 } default { y := mul(x, f(sub(x, 1))) }   }``

现在我们详细讲解内联汇编语言。

.. warning::
    内联汇编是一种在底层访问以太坊虚拟机的语言。这没有Solidity提供的多个重要安全特点。

.. note::
    TODO：写出内联汇编的范围规则是如何存在细微差别、比如使用内部库函数时产生的复杂性。此外，还要编写有关编译器定义的符号。

例子
-------

下面例子展示了一个库代码访问另一个合约的代码，并加载到一个字节变量中。这对于“常规 Solidity”来说根本不可能，汇编库以这种方式来增强语言。

.. code::

    pragma solidity ^0.4.0;

    library GetCode {
        function at(address _addr) public view returns (bytes o_code) {
            assembly {
                // 获取代码大小，这需要汇编语言
                let size := extcodesize(_addr)
                // 分配输出字节数组 – 这也可以不用汇编语言来实现
                // 利用语句 o_code = new bytes（size）
                o_code := mload(0x40)
                // 包括补位在内新的“memory end”
                mstore(0x40, add(o_code, and(add(add(size, 0x20), 0x1f), not(0x1f))))
                // 存储内存长度
                mstore(o_code, size)
                // 实际获取代码，这需要汇编语言
                extcodecopy(_addr, add(o_code, 0x20), 0, size)
            }
        }
    }

在优化器无法生成高效代码的情况下，内联汇编也可能更有好处。请注意，由于编译器没有执行检查，编写汇编语言代码变得更加困难，因此只有在确实知道自己要做什么的时候，你才能将它应用到复杂事务中。

.. code::

    pragma solidity ^0.4.16;

    library VectorSum {
        // 由于当前优化器在数组读写中不能删除边界检查，函数执行效率变低。
        //
        function sumSolidity(uint[] _data) public view returns (uint o_sum) {
            for (uint i = 0; i < _data.length; ++i)
                o_sum += _data[i];
        }

        // 我们知道我们只能访问定义长度内的数组元素，所以我们可以避免这种检查。由于第一个包含数组长度，需要把0x20加到数组中。
        //
        //
        function sumAsm(uint[] _data) public view returns (uint o_sum) {
            for (uint i = 0; i < _data.length; ++i) {
                assembly {
                    o_sum := add(o_sum, mload(add(add(_data, 0x20), mul(i, 0x20))))
                }
            }
        }

        // 和上面一样，但是要在内联汇编内完成整个代码。
        function sumPureAsm(uint[] _data) public view returns (uint o_sum) {
            assembly {
               // 加载长度（前32字节）
               let len := mload(_data)

               // 略过长度字段。
               //
               // 保持临时变量以便它可以在原地增加。
               //
               // 注意：在汇编块后 incrementing _data 将出现 unusable_data 变量。
               //
               let data := add(_data, 0x20)

               // 迭代到边界。
               for
                   { let end := add(data, len) }
                   lt(data, end)
                   { data := add(data, 0x20) }
               {
                   o_sum := add(o_sum, mload(data))
               }
            }
        }
    }


语法
------

和Solidity一样，Assembly也会解析注释、文字和标识符，所以你可以使用通常的//和/ * * /来注释。内联汇编程序由{…}来标记，在这些大括号内可以使用以下内容（更多详细信息请参阅后面部分）。

 - 文字，比如 0x123、42或“abc”（不超过32个字符的字符串）
 - 操作码（在“instruction style”内），比如 mload sload dup1 sstore，操作码列表请看后面
 - 函数式操作码，比如 add（1，mlod（0））
 - 标签，比如 name
 - 变量声明，比如 let x := 7、let x := add（y，3）或者 let x（给 empty（0）赋初始值）
 - 标识符（标签或者汇编局部变量以及用作内联汇编时的外部变量），比如 jump（name）、3 x add
 - 赋值（在“instruction style”内），比如 3 =: x
 - 函数式赋值，比如 x := add（y，3）
 - 块内局部变量的范围，比如{let x := 3 {let y := add（x，1）}}

操作码
-------

本文档不是以太坊虚拟机的详细描述，但后面列表可以作为操作码参考。

如果一个操作码需要参数（总是来自堆栈顶部），它们会在括号中给出。请注意：参数顺序可以看作是在非函数式中颠倒而来的（下面解释）。 标有“-”的操作码不会将一个目标推送到堆栈中，标有*的操作码是特殊的，而所有其他操作码都会将一个目标推送到堆栈中。

下面讲述中，mem [a…b]表示从位置 a 开始至（不包括）位置 b 的内存字节数，storage[p]表示位置 p 处的存储内容。

Pushi 和 jumpdest 这两个操作码不能直接用。

在语法中，操作码可以表示为预定义的标识符。

+-------------------------+------+-----------------------------------------------------------------+
| stop                    + `-`  | 停止执行，等同于 return（ 0，0 ）                                 |
+-------------------------+------+-----------------------------------------------------------------+
| add(x, y)               |      | x + y                                                           |
+-------------------------+------+-----------------------------------------------------------------+
| sub(x, y)               |      | x - y                                                           |
+-------------------------+------+-----------------------------------------------------------------+
| mul(x, y)               |      | x * y                                                           |
+-------------------------+------+-----------------------------------------------------------------+
| div(x, y)               |      | x / y                                                           |
+-------------------------+------+-----------------------------------------------------------------+
| sdiv(x, y)              |      | x / y，对于二进制补码的符号数字                                   |
+-------------------------+------+-----------------------------------------------------------------+
| mod(x, y)               |      | x % y                                                           |
+-------------------------+------+-----------------------------------------------------------------+
| smod(x, y)              |      | x % y，对于二进制补码的符号数字                                   |
+-------------------------+------+-----------------------------------------------------------------+
| exp(x, y)               |      | x 的 y 次幂                                                     |
+-------------------------+------+-----------------------------------------------------------------+
| not(x)                  |      | ~x，对 x 的每一位取负                                            |
+-------------------------+------+-----------------------------------------------------------------+
| lt(x, y)                |      | 如果 x < y 为 1，否则为 0                                        |
+-------------------------+------+-----------------------------------------------------------------+
| gt(x, y)                |      | 如果 x > y 为 1，否则为 0                                        |
+-------------------------+------+-----------------------------------------------------------------+
| slt(x, y)               |      | 如果 x < y 为 1，否则为 0，对于二进制补码的符号数字                |
+-------------------------+------+-----------------------------------------------------------------+
| sgt(x, y)               |      | 如果 x > y 为 1，否则为 0，对于二进制补码的符号数字                |
+-------------------------+------+-----------------------------------------------------------------+
| eq(x, y)                |      | 如果 x == y 为 1，否则为 0                                       |
+-------------------------+------+-----------------------------------------------------------------+
| iszero(x)               |      | 如果 x == 0 为 1，否则为 0                                       |
+-------------------------+------+-----------------------------------------------------------------+
| and(x, y)               |      | x 和 y 的按位与                                                  |
+-------------------------+------+-----------------------------------------------------------------+
| or(x, y)                |      | x 和 y 的按位或                                                  |
+-------------------------+------+-----------------------------------------------------------------+
| xor(x, y)               |      | x 和 y 的按位异或                                                |
+-------------------------+------+-----------------------------------------------------------------+
| byte(n, x)              |      | x 的第 n 个字节，此处第 0 个字节就是最高有效字节                   |
+-------------------------+------+-----------------------------------------------------------------+
| addmod(x, y, m)         |      | 任意精度的（ x + y ）%  m                                        |
+-------------------------+------+-----------------------------------------------------------------+
| mulmod(x, y, m)         |      | 任意精度的（ x * y ）% m                                         |
+-------------------------+------+-----------------------------------------------------------------+
| signextend(i, x)        |      | 从最低有效位开始计数的第（ i * 8 + 7 ）个的符号                    |
+-------------------------+------+-----------------------------------------------------------------+
| keccak256(p, n)         |      | keccak ( mem [ p ... ( p + n )))                                |
+-------------------------+------+-----------------------------------------------------------------+
| sha3(p, n)              |      | keccak ( mem [ p ... ( p + n )))                                |
+-------------------------+------+-----------------------------------------------------------------+
| jump(label)             | `-`  | 跳转到标签 / 符号位                                              |
+-------------------------+------+-----------------------------------------------------------------+
| jumpi(label, cond)      | `-`  | 如果条件为非零，跳转到标签                                        |
+-------------------------+------+-----------------------------------------------------------------+
| pc                      |      | 当前代码位置                                                     |
+-------------------------+------+-----------------------------------------------------------------+
| pop(x)                  | `-`  | 删除 x 推送的元素                                                |
+-------------------------+------+-----------------------------------------------------------------+
| dup1 ... dup16          |      | 将第 i 个堆栈槽复制到顶部（从顶部算起）                            |
+-------------------------+------+-----------------------------------------------------------------+
| swap1 ... swap16        | `*`  | 交换最上面的和下部的第 i 个堆栈槽                                 |
+-------------------------+------+-----------------------------------------------------------------+
| mload(p)                |      | mem [ p … （ p + 32 ））                                        |
+-------------------------+------+-----------------------------------------------------------------+
| mstore(p, v)            | `-`  | mem [ p … （ p + 32 ）） := v                                   |
+-------------------------+------+-----------------------------------------------------------------+
| mstore8(p, v)           | `-`  | mem [ p ] := v & 0xff  — 仅修改一个字节                          |
+-------------------------+------+-----------------------------------------------------------------+
| sload(p)                |      | storage [ p ]                                                   |
+-------------------------+------+-----------------------------------------------------------------+
| sstore(p, v)            | `-`  | storage [ p ] := v                                              |
+-------------------------+------+-----------------------------------------------------------------+
| msize                   |      | 内存大小，比如最大可读写内存索引                                   |
+-------------------------+------+-----------------------------------------------------------------+
| gas                     |      | 执行可用的 gas                                                   |
+-------------------------+------+-----------------------------------------------------------------+
| address                 |      | 当前合约/执行引文的地址                                           |
+-------------------------+------+-----------------------------------------------------------------+
| balance(a)              |      | 地址 a 以 Wei 计的余额                                           |
+-------------------------+------+-----------------------------------------------------------------+
| caller                  |      | 调用发起者（代表调用除外）                                        |
+-------------------------+------+-----------------------------------------------------------------+
| callvalue               |      | 与当前调用一起发送的 Wei 数                                       |
+-------------------------+------+-----------------------------------------------------------------+
| calldataload(p)         |      | 从位置 p （ 32 字节） 处开始调用数据                              |
+-------------------------+------+-----------------------------------------------------------------+
| calldatasize            |      | 以字节计算的调用数据大小                                          |
+-------------------------+------+-----------------------------------------------------------------+
| calldatacopy(t, f, s)   | `-`  | 从位置 f 处的调用数据拷贝 s 个字节到位置 t 处的内存中               |
+-------------------------+------+-----------------------------------------------------------------+
| codesize                |      | 当前合约 / 执行引文的代码大小                                     |
+-------------------------+------+-----------------------------------------------------------------+
| codecopy(t, f, s)       | `-`  | 从位置 f 处的代码中拷贝 s 个字节到位置 t 的内存中                  |
+-------------------------+------+-----------------------------------------------------------------+
| extcodesize(a)          |      | 地址 a 处的代码大小                                              |
+-------------------------+------+-----------------------------------------------------------------+
| extcodecopy(a, t, f, s) | `-`  | 和 codecopy（ t，f，s ）类似，但要考虑位置 a 的代码                |
+-------------------------+------+-----------------------------------------------------------------+
| returndatasize          |      | 最后一个 returndata 的大小                                       |
+-------------------------+------+-----------------------------------------------------------------+
| returndatacopy(t, f, s) | `-`  | 把位置 f 处 returndata 的 s 个字节拷贝到位置 t 处的内存中          |
+-------------------------+------+-----------------------------------------------------------------+
| create(v, p, s)         |      | 利用代码 mem [ p … （ p + s ）） 产生新合约、发送 v Wei 且返回     |
|                         |      | 新地址                                                          |
+-------------------------+------+-----------------------------------------------------------------+
| create2(v, n, p, s)     |      | 利用 keccak256（< address > . n . keccak256                     |
|                         |      | （ mem [ p….（ p + s ）））位置的代码 mem [ p … （ p + s ））     |
|                         |      |  产生新合约、发送 v Wei 且返回新地址1                             |
+-------------------------+------+-----------------------------------------------------------------+
| call(g, a, v, in,       |      | 输入 mem [ in … （ in + insize ）） 提供 g 个gas和 v Wei、输出    |
| insize, out, outsize)   |      | mem [ ou t… （ out + outsize ））在位置 a 处调用合约，错误时返回 0 |
|                         |      | （比如 out of gas）， 正确返回 1                                  |
|                         |      |                                                                 |
+-------------------------+------+-----------------------------------------------------------------+
| callcode(g, a, v, in,   |      | 与调用等价、但仅使用 a 中的代码且没有驻留在当前合同的上下文中        |
| insize, out, outsize)   |      |                                                                 |
+-------------------------+------+-----------------------------------------------------------------+
| delegatecall(g, a, in,  |      | 与 callcode 等价且不保留调用者和调用值                            |
| insize, out, outsize)   |      |                                                                 |
+-------------------------+------+-----------------------------------------------------------------+
| staticcall(g, a, in,    |      | 与 call（ g，a，0，in，insize，out，outsize ）等价但不允许状态修改 |
| insize, out, outsize)   |      |                                                                 |
+-------------------------+------+-----------------------------------------------------------------+
| return(p, s)            | `-`  | 终止运行，返回数据 mem [ p … （ p + s ））                        |
+-------------------------+------+-----------------------------------------------------------------+
| revert(p, s)            | `-`  | 终止运行，翻转状态变化，返回数据 mem [ p … （ p + s ））           |
+-------------------------+------+-----------------------------------------------------------------+
| selfdestruct(a)         | `-`  | 终止运行，销毁当前合约并且把钱返回给 a                             |
+-------------------------+------+-----------------------------------------------------------------+
| invalid                 | `-`  | 以无效指令终止运行                                               |
+-------------------------+------+-----------------------------------------------------------------+
| log0(p, s)              | `-`  | 没有标题的日志和数据 mem [ p … （ p + s ））                      |
+-------------------------+------+-----------------------------------------------------------------+
| log1(p, s, t1)          | `-`  | 标题为 t1 的日志和数据 mem [ p … （ p + s ））                   |
+-------------------------+------+-----------------------------------------------------------------+
| log2(p, s, t1, t2)      | `-`  | 标题为 t1和t2 的日志和数据 mem [ p … （ p + s ））                |
+-------------------------+------+-----------------------------------------------------------------+
| log3(p, s, t1, t2, t3)  | `-`  | 标题为 t1、t2 和t3 的日志和数据 mem [ p … （ p + s ））           |
+-------------------------+------+-----------------------------------------------------------------+
| log4(p, s, t1, t2, t3,  | `-`  | 标题为 t1、t2、t3 和 t4 的日志和数据 mem [ p … （ p + s ））      |
| t4)                     |      |                                                                 |
+-------------------------+------+-----------------------------------------------------------------+
| origin                  |      | 交易发起者                                                       |
+-------------------------+------+-----------------------------------------------------------------+
| gasprice                |      | gas 交易价格                                                    |
+-------------------------+------+-----------------------------------------------------------------+
| blockhash(b)            |      | 区块 nr b 的哈希—仅适用于不包括当前区块的最后 256 个区块           |
+-------------------------+------+-----------------------------------------------------------------+
| coinbase                |      | 当前矿工收益                                                     |
+-------------------------+------+-----------------------------------------------------------------+
| timestamp               |      | 从 epoch 开始、以秒计的当前区块时间戳                             |
+-------------------------+------+-----------------------------------------------------------------+
| number                  |      | 当前区块号码                                                     |
+-------------------------+------+-----------------------------------------------------------------+
| difficulty              |      | 当前区块难度                                                     |
+-------------------------+------+-----------------------------------------------------------------+
| gaslimit                |      | 当前区块的块 gas 上限                                            |
+-------------------------+------+-----------------------------------------------------------------+

文字
--------

你可以键入十进制或十六进制符号来使用整型常量，并自动生成相应的 PUSHi 指令。下面将创建代码：2 加 3 等于 5、计算按位、和字符串“abc”相连。字符串存储为左对齐，不能超过 32 个字节。

.. code::

    assembly { 2 3 add "abc" and }

函数风格
-----------------

你可以在操作码之后键入操作码，它们将以字节码结尾。例如，把 3 加到位置 0x80 处的内存中就是

.. code::

    3 0x80 mload add 0x80 mstore

由于通常很难看到某些操作码的实际参数是什么，所以 Solidity 内联汇编还提供了一种“函数式”表示法，其中相同的代码编写如下

.. code::

    mstore(0x80, add(mload(0x80), 3))

函数式表达式不能在内部使用指令方式，即 1 mstore（0x80，add）是无效汇编语句，它必须写成 mstore（0x80，add（2，1））这种形式。对于不带参数的操作码，括号可以省略。

请注意：在函数式中参数的顺序与指令方式相反。如果使用函数式，第一个参数将会在堆栈顶部结束。


访问外部变量和函数
------------------------------------------

通过简单使用它们名称就可以访问 Solidity 变量和其他标识符。对于内存变量，这会将地址而不是值推送到堆栈中。存储变量则不同：存储的值可能不占用完整的存储槽，因此“地址”由槽和槽内的字节偏移量组成。为了获取变量 x 所指向的槽，你可以使用 x_slot 并获取你使用的 x_offset 的字节偏移量。

在赋值时（见下文），我们甚至可以使用本地 Solidity 变量来赋值。

也可以访问内联汇编的外部函数：汇编将推入它们的入口标签（应用虚函数解析）。在 Solidity中的调用语义是：

 - the caller pushes return label, arg1, arg2, ..., argn
 - the call returns with ret1, ret2, ..., retm

这个特性使用起来还是有点麻烦，因为在调用过程中堆栈偏移量发生了根本变化，因此对局部变量的引用将会出错。

.. code::

    pragma solidity ^0.4.11;

    contract C {
        uint b;
        function f(uint x) public returns (uint r) {
            assembly {
                r := mul(x, sload(b_slot)) // ignore the offset, we know it is zero
            }
        }
    }

标签
------

EVM 汇编的另一个问题是 jump 和 jumpi 函数使用绝对地址，这些绝对地址很容易改变。 Solidity 内联汇编提供了标签，以便更容易地使用 jump。请注意，标签具有底层特征，只用循环、if 和 switch 指令（参见下文），没有标签也能写出高效汇编代码。以下代码计算斐波那契数列中的一个元素。

.. code::

    {
        let n := calldataload(4)
        let a := 1
        let b := a
    loop:
        jumpi(loopend, eq(n, 0))
        a add swap1
        n := sub(n, 1)
        jump(loop)
    loopend:
        mstore(0, a)
        return(0, 0x20)
    }

请注意：只有汇编器知道当前堆栈高度时，才能自动访问堆栈变量。如果 jump 起点和终点具有不同的堆栈高度，访问将失败。使用这种 jump 仍然很好，但在这种情况下，你应该不会只是访问任何堆栈变量（即使是汇编变量）。

此外，堆栈高度分析器还可以通过操作码（而不是根据控制流）检查代码操作码，因此在下面的情况下，汇编器对标签 2 处的堆栈高度会产生错误的印象：

.. code::

    {
        let x := 8
        jump(two)
        one:
            // 这里的堆栈高度是 2（因为我们推送了 x 和 7），
            // 但因为它从堆栈顶部到尾部读取，汇编器认为它是 1。
            //
            // 在这里访问堆栈变量 x 会导致错误。
            x := 9
            jump(three)
        two:
            7 // 把某物推到堆栈中
            jump(one)
        three:
    }

汇编局部变量声明
----------------------------------

你可以使用 let 关键字来声明只在内联汇编中可见的变量，实际上只在当前的｛…｝—块中可见。下面发生的事情应该是：let 指令将创建一个为变量保留的新堆栈槽，并在到达块末尾时自动删除。你需要为变量提供一个初始值，它可以只是 0，但它也可以是一个复杂的函数式表达式。

.. code::

    pragma solidity ^0.4.16;

    contract C {
        function f(uint x) public view returns (uint b) {
            assembly {
                let v := add(x, 1)
                mstore(0x80, v)
                {
                    let y := add(sload(v), 1)
                    b := y
                } // 在这里 y 是“deallocated”
                b := add(b, v)
            } // 在这里 v 是“deallocated”
        }
    }


赋值
-----------

可以给汇编局部变量和函数局部变量赋值。请注意：当给指向内存或存储的变量赋值时，你只是更改指针而不是数据。

有两种赋值方式：函数方式和指令方式。对于函数式赋值方式（变量：= 值），你需要在函数式表达式中提供一个值，这个值恰好可以产生一个堆栈值；对于指令方式赋值（=： variable），仅从堆栈顶部获取。对于这两种方式，冒号指向变量名称。赋值是通过用新值替换堆栈中的变量值来实现的。

.. code::

    {
        let v := 0 // functional-style assignment as part of variable declaration
        let g := add(v, 2)
        sload(10)
        =: v // instruction style assignment, puts the result of sload(10) into v
    }

If
--

if语句可以用于有条件地执行代码。没有“else”部分，如果需要多种选择，你可以考虑使用“switch”（见下文）。

.. code::

    {
        if eq(value, 0) { revert(0, 0) }
    }

代码主体的花括号是必需的。

Switch
------

作为“if / else”的非常基础版本，你可以使用 switch 语句。它计算表达式的值并与几个常量进行比较。选出与匹配常数对应的分支。与某些编程语言容易出错的情况不同，控制流不会从一种情形继续执行到下一种情形。可能存在一个反馈或称为缺省的缺省情形。

.. code::

    {
        let x := 0
        switch calldataload(4)
        case 0 {
            x := calldataload(0x24)
        }
        default {
            x := calldataload(0x44)
        }
        sstore(0, div(x, 2))
    }

Case 列表里面不需要大括号，但 case 主体确实需要。

循环
-----

汇编语言支持一个简单的 for-style循环。For-style 循环有一个头，它包含起始、条件和后迭代等部分。条件必须是函数表达式，而另外两个部分都是块。如果起始部分声明了某个变量，这些变量的作用域可以扩展到正文中（包括条件和后迭代部分）。

下面例子是计算内存中区域的总和。

.. code::

    {
        let x := 0
        for { let i := 0 } lt(i, 0x100) { i := add(i, 0x20) } {
            x := add(x, mload(i))
        }
    }

For 循环也可以写成像 while 循环一样：只需将起始和后迭代两个部分为空。

.. code::

    {
        let x := 0
        let i := 0
        for { } lt(i, 0x100) { } {     // while(i < 0x100)
            x := add(x, mload(i))
            i := add(i, 0x20)
        }
    }

函数
---------

汇编语言允许定义底层函数。底层函数需要从堆栈中取出它们的参数（并返回 PC），并将结果放入堆栈。调用函数的方式与执行函数式操作码相同。

函数可以在任何地方定义，并且在声明它们的块中可见。函数内部不能访问在函数之外定义的局部变量。没有明确的 return 声明。

如果调用返回多个值的函数，则必须使用 a，b：= f（x）或 a，b：= f（x）方式给它们赋值一个元组。

下面例子通过平方和乘法实现幂函数计算的。

.. code::

    {
        function power(base, exponent) -> result {
            switch exponent
            case 0 { result := 1 }
            case 1 { result := base }
            default {
                result := power(mul(base, base), div(exponent, 2))
                switch mod(exponent, 2)
                    case 1 { result := mul(base, result) }
            }
        }
    }

注意事项
---------------

内联汇编语言可能具有相当高级的外观，但实际上它是非常低级的编程语言。函数调用、循环、if 语句和 switch 语句通过简单的重写规则进行转换，然后，汇编器为你做的唯一事情就是重新组织函数式操作码、管理 jump 标签、计算访问变量的堆栈高度，还有到达块尾部时删除局部汇编变量的堆栈槽。特别是对于最后两种情况，汇编程序仅从堆栈顶部到尾部计算堆栈高度，而不一定要遵循控制流程，这一点非常重要。此外， swap 等操作只会交换堆栈内容，而不会交换变量位置。

Solidity 惯例
-----------------------

与EVM汇编语言相比，Solidity 能够识别小于256位的类型，例如 uint24。为了提高效率，大多数算术运算只将它们视为 256 位数字，仅在必要时清除高阶位，即在它们写入内存或执行比较之前不久实施。这意味着，如果从内联汇编中访问这样的变量，你必须首先手动清除更高阶位。

Solidity以一种非常简单的方式管理内存：内存中的位置 0x40 有一个“空闲内存指针”。如果你打算分配内存，只需从此处开始使用内存，然后相应地更新指针即可。

在 Solidity 中，内存数组的元素总是占用 32 个字节的倍数（是的，对于 byte[]是正确的，但对于 bytes and string 就不是这样）。多维内存数组是指向内存数组的指针。动态数组的长度存储在数组的第一个插槽中，紧接着是数组元素。

.. warning::
    静态大小的内存数组没有长度字段，但它很快就会增加，以便在静态大小和动态大小的数组之间实现更好的转换，所以请不要用长度字段。


独立汇编
===================

以上内联汇编描述的汇编语言也可以单独使用，实际上，计划是将用作 Solidity 编译器的中间语言。在这种意义下，它试图实现以下几个目标：

1、即使代码是由 Solidity 的编译器生成的，用它编写的程序应该也是可读的。
2、从汇编到字节码的翻译应该尽可能少地包含“意外”。
3、控制流应该易于检测，以帮助进行形式验证和优化。

为了实现第一个和最后一个目标，汇编提供了高级结构：如循环、if 语句、switch 语句和函数调用。应该可以编写不使用明确的 WAP、DUP、JUMP 和 JUMPI语句的汇编程序，因为前两个混淆了数据流，而最后两个混淆了控制流。此外，形式为 mul（add（x，y），7）的函数语句优于如 7 y x add mul 操作码语句，因为在第一种形式中更容易查看哪个操作数用于哪个操作码。

第二个目标是通过引入一个脱钩阶段来实现的，这只能以常规方式删除高级结构，仍然允许检查生成的低级汇编代码。汇编器执行的唯一非本地操作是用户定义标识符（函数、变量、...）的名称查找，它遵循非常简单和常规的域内规则以及从堆栈中清除局部变量。

作用域：声明的标识符（标签、变量、函数、汇编）仅在声明的块中可见（包括当前块中的嵌套块）。即使它们在作用范围内，越过函数边界访问局部变量也是非法的。阴影化是禁止的。在声明之前不能访问局部变量，但标签、函数和汇编是可以的。汇编是特殊块，用于如返回运行时间代码或创建合同等。在子汇编中没有可见的外部汇编标识符。

如果控制流经过块尾部，则会插入与块声明的局部变量数量相匹配的 pop 指令。无论何时引用局部变量，代码生成器都需要知道在当前堆栈中的相对位置，因此，需要跟踪当前所谓的堆栈高度。在经过块尾部时删除所有局部变量，因此块前后的堆栈高度应该相同。如果情况并非如此，则会发出警告。

为什么我们使用像 switch、for 和 function 等更高级结构：

使用 switch、for 和 functions，应该可以编写复杂的代码，而无需使用 jump 或 jumpi。 这使得分析控制流程变得更加容易，可以改进形式验证和优化。

此外，如果允许手动跳转，计算堆栈高度相当复杂。需要知道堆栈中所有局部变量的位置，否则在块结束时既不会自动引用局部变量，也不会从堆栈中自动删除局部变量。脱钩机制可以正确地将操作插入无法访问的块中，以便在没有持续控制流的跳转情况下正确调整堆栈高度。

例子：

我们将按照 Solidity 的汇编实例实施脱钩操作。我们考虑以下 Solidity 程序的运行时间字节码：

    pragma solidity ^0.4.16;

    contract C {
      function f(uint x) public pure returns (uint y) {
        y = 1;
        for (uint i = 0; i < x; i++)
          y = 2 * y;
      }
    }

产生的汇编语言如下：

    {
      mstore(0x40, 0x60) // store the "free memory pointer"
      // 函数分发器
      switch div(calldataload(0), exp(2, 226))
      case 0xb3de648b {
        let (r) = f(calldataload(4))
        let ret := $allocate(0x20)
        mstore(ret, r)
        return(ret, 0x20)
      }
      default { revert(0, 0) }
      // 内存分配器
      function $allocate(size) -> pos {
        pos := mload(0x40)
        mstore(0x40, add(pos, size))
      }
      // 合约函数
      function f(x) -> y {
        y := 1
        for { let i := 0 } lt(i, x) { i := add(i, 1) } {
          y := mul(2, y)
        }
      }
    }

经过脱钩阶段后的代码如下：

    {
      mstore(0x40, 0x60)
      {
        let $0 := div(calldataload(0), exp(2, 226))
        jumpi($case1, eq($0, 0xb3de648b))
        jump($caseDefault)
        $case1:
        {
          // 函数调用—我们把返回标签和参数推入堆栈中
          $ret1 calldataload(4) jump(f)
          // 这是无法访问的代码。添加了操作码。操作码反映了堆栈高度上函数的作用：删除了参数并引入了返回值。
          //
          //
          pop pop
          let r := 0
          $ret1: // the actual return point
          $ret2 0x20 jump($allocate)
          pop pop let ret := 0
          $ret2:
          mstore(ret, r)
          return(ret, 0x20)
          // 尽管它没有用处，但 jump 会自动插入，因为脱钩过程是一种纯粹的句法操作，不会分析控制流
          //
          //
          jump($endswitch)
        }
        $caseDefault:
        {
          revert(0, 0)
          jump($endswitch)
        }
        $endswitch:
      }
      jump($afterFunction)
      allocate:
      {
        // 我们跳过引入函数参数的执行性不到的代码
        jump($start)
        let $retpos := 0 let size := 0
        $start:
        // 输出变量与参数具有相同范围，并且实实在在地分配。
        //
        let pos := 0
        {
          pos := mload(0x40)
          mstore(0x40, add(pos, size))
        }
        // 代码通过返回值替换参数并跳回。
        swap1 pop swap1 jump
        // 再次校正堆栈高度执行不到的代码。
        0 0
      }
      f:
      {
        jump($start)
        let $retpos := 0 let x := 0
        $start:
        let y := 0
        {
          let i := 0
          $for_begin:
          jumpi($for_end, iszero(lt(i, x)))
          {
            y := mul(2, y)
          }
          $for_continue:
          { i := add(i, 1) }
          jump($for_begin)
          $for_end:
        } // 这里为 i 插入pop 指令
        swap1 pop swap1 jump
        0 0
      }
      $afterFunction:
      stop
    }


汇编运行四个阶段：

1、解析
2、脱钩（去删 switch、for 和函数）
3、操作码流生成
4、字节码生成

我们将以非正式方式来讲解第一步到第三步。更正式细节在后面。


解析/语法
-----------------

解析器任务如下：

- 将字节流转换为代币流，不使用 C ++ 风格的注释（对源引用存在特殊注释，我们不会在这里解释它）。
- 根据下面语法，将代币流转换为 AST。
- 利用所定义块（注释到 AST 节点）的寄存器进行注册，注明从哪个地方开始访问变量。

汇编词法分析器遵循由 Solidity 汇编词法分析器定义的规则。

空格用于分隔代币，它由字符空格、制表符和换行符组成。注释格式是常规的 JavaScript / C ++ 一样，并且同样解释为空格。

语法：

    AssemblyBlock = '{' AssemblyItem* '}'
    AssemblyItem =
        Identifier |
        AssemblyBlock |
        AssemblyExpression |
        AssemblyLocalDefinition |
        AssemblyAssignment |
        AssemblyStackAssignment |
        LabelDefinition |
        AssemblyIf |
        AssemblySwitch |
        AssemblyFunctionDefinition |
        AssemblyFor |
        'break' |
        'continue' |
        SubAssembly
    AssemblyExpression = AssemblyCall | Identifier | AssemblyLiteral
    AssemblyLiteral = NumberLiteral | StringLiteral | HexLiteral
    Identifier = [a-zA-Z_$] [a-zA-Z_0-9]*
    AssemblyCall = Identifier '(' ( AssemblyExpression ( ',' AssemblyExpression )* )? ')'
    AssemblyLocalDefinition = 'let' IdentifierOrList ( ':=' AssemblyExpression )?
    AssemblyAssignment = IdentifierOrList ':=' AssemblyExpression
    IdentifierOrList = Identifier | '(' IdentifierList ')'
    IdentifierList = Identifier ( ',' Identifier)*
    AssemblyStackAssignment = '=:' Identifier
    LabelDefinition = Identifier ':'
    AssemblyIf = 'if' AssemblyExpression AssemblyBlock
    AssemblySwitch = 'switch' AssemblyExpression AssemblyCase*
        ( 'default' AssemblyBlock )?
    AssemblyCase = 'case' AssemblyExpression AssemblyBlock
    AssemblyFunctionDefinition = 'function' Identifier '(' IdentifierList? ')'
        ( '->' '(' IdentifierList ')' )? AssemblyBlock
    AssemblyFor = 'for' ( AssemblyBlock | AssemblyExpression )
        AssemblyExpression ( AssemblyBlock | AssemblyExpression ) AssemblyBlock
    SubAssembly = 'assembly' Identifier AssemblyBlock
    NumberLiteral = HexNumber | DecimalNumber
    HexLiteral = 'hex' ('"' ([0-9a-fA-F]{2})* '"' | '\'' ([0-9a-fA-F]{2})* '\'')
    StringLiteral = '"' ([^"\r\n\\] | '\\' .)* '"'
    HexNumber = '0x' [0-9a-fA-F]+
    DecimalNumber = [0-9]+


脱钩
----------

AST 转换删除了 for、switch 和函数结构。结果仍然可以由同一个解析器解析，但它不会使用某些结构。如果添加上 jumpdests，只跳转但不会继续，将添加有关堆栈内容的信息，除非没有访问外部作用域的局部变量，或者堆栈高度与前一条指令相同。

伪码：

    desugar item: AST -> AST =
    match item {
    AssemblyFunctionDefinition('function' name '(' arg1, ..., argn ')' '->' ( '(' ret1, ..., retm ')' body) ->
      <name>:
      {
        jump($<name>_start)
        let $retPC := 0 let argn := 0 ... let arg1 := 0
        $<name>_start:
        let ret1 := 0 ... let retm := 0
        { desugar(body) }
        swap and pop items so that only ret1, ... retm, $retPC are left on the stack
        jump
        0 (1 + n times) to compensate removal of arg1, ..., argn and $retPC
      }
    AssemblyFor('for' { init } condition post body) ->
      {
        init // cannot be its own block because we want variable scope to extend into the body
        // 不能是自己的块，因为我们想要变量范围扩展到代码主体，找到我这样没有标签 $ forI_ *
        $forI_begin:
        jumpi($forI_end, iszero(condition))
        { body }
        $forI_continue:
        { post }
        jump($forI_begin)
        $forI_end:
      }
    'break' ->
      {
        // 用标签 $ forI_end 找到最近的封闭作用域
        pop all local variables that are defined at the current point
        but not at $forI_end
        jump($forI_end)
        0 (as many as variables were removed above)
      }
    'continue' ->
      {
        // 用标签 $forI_continue 找到最近的封闭作用域
        pop all local variables that are defined at the current point
        but not at $forI_continue
        jump($forI_continue)
        0 (as many as variables were removed above)
      }
    AssemblySwitch(switch condition cases ( default: defaultBlock )? ) ->
      {
        // 如果没有 $switchI* 的标签和变量就找到了I
        let $switchI_value := condition
        for each of cases match {
          case val: -> jumpi($switchI_caseJ, eq($switchI_value, val))
        }
        if default block present: ->
          { defaultBlock jump($switchI_end) }
        for each of cases match {
          case val: { body } -> $switchI_caseJ: { body jump($switchI_end) }
        }
        $switchI_end:
      }
    FunctionalAssemblyExpression( identifier(arg1, arg2, ..., argn) ) ->
      {
        if identifier is function <name> with n args and m ret values ->
          {
            // 如果 $funcallI_* 不存在就找到了I
            $funcallI_return argn  ... arg2 arg1 jump(<name>)
            pop (n + 1 times)
            if the current context is `let (id1, ..., idm) := f(...)` ->
              let id1 := 0 ... let idm := 0
              $funcallI_return:
            else ->
              0 (m times)
              $funcallI_return:
              turn the functional expression that leads to the function call
              into a statement stream
          }
        else -> desugar(children of node)
      }
    default node ->
      desugar(children of node)
    }

操作流生成
------------------------

在操作码流生成期间，我们会跟踪计数器中的当前堆栈高度，使得通过名称访问堆栈变量成为可能。每个修改堆栈的操作码以及每个用堆栈调整注释的标签都可以更改堆栈高度。每次引入一个新的局部变量时，它都会与当前堆栈高度一起注册。如果访问一个变量（拷贝变量值或给变量赋值），则根据当前堆栈高度和引入这个变量时的堆栈高度之间的差异选择适当的 DUP 或 SWAP 指令。

伪码：

    codegen item: AST -> opcode_stream =
    match item {
    AssemblyBlock({ items }) ->
      join(codegen(item) for item in items)
      if last generated opcode has continuing control flow:
        POP for all local variables registered at the block (including variables
        introduced by labels)
        warn if the stack height at this point is not the same as at the start of the block
    Identifier(id) ->
      lookup id in the syntactic stack of blocks
      match type of id
        Local Variable ->
          DUPi where i = 1 + stack_height - stack_height_of_identifier(id)
        Label ->
          // reference to be resolved during bytecode generation
          PUSH<bytecode position of label>
        SubAssembly ->
          PUSH<bytecode position of subassembly data>
    FunctionalAssemblyExpression(id ( arguments ) ) ->
      join(codegen(arg) for arg in arguments.reversed())
      id (which has to be an opcode, might be a function name later)
    AssemblyLocalDefinition(let (id1, ..., idn) := expr) ->
      register identifiers id1, ..., idn as locals in current block at current stack height
      codegen(expr) - assert that expr returns n items to the stack
    FunctionalAssemblyAssignment((id1, ..., idn) := expr) ->
      lookup id1, ..., idn in the syntactic stack of blocks, assert that they are variables
      codegen(expr)
      for j = n, ..., i:
      SWAPi where i = 1 + stack_height - stack_height_of_identifier(idj)
      POP
    AssemblyAssignment(=: id) ->
      look up id in the syntactic stack of blocks, assert that it is a variable
      SWAPi where i = 1 + stack_height - stack_height_of_identifier(id)
      POP
    LabelDefinition(name:) ->
      JUMPDEST
    NumberLiteral(num) ->
      PUSH<num interpreted as decimal and right-aligned>
    HexLiteral(lit) ->
      PUSH32<lit interpreted as hex and left-aligned>
    StringLiteral(lit) ->
      PUSH32<lit utf-8 encoded and left-aligned>
    SubAssembly(assembly <name> block) ->
      append codegen(block) at the end of the code
    dataSize(<name>) ->
      assert that <name> is a subassembly ->
      PUSH32<size of code generated from subassembly <name>>
    linkerSymbol(<lit>) ->
      PUSH32<zeros> and append position to linker table
    }
