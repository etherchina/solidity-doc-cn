.. include:: glossaries.rst

********************************
Solidity 源文件结构
********************************



源文件中可以包含任意多个 :ref:`合约定义 <contract_structure>` 、:ref:`导入源文件指令 <import>` 、 :ref:`版本标识 <pragma>` 指令、
:ref:`结构体<structs>` , :ref:`枚举<enums>` 和 :ref:`函数<functions>` 定义.

SPDX License Identifier
=======================

Trust in smart contract can be better established if their source code
is available. Since making source code available always touches on legal problems
with regards to copyright, the Solidity compiler encourages the use
of machine-readable `SPDX license identifiers <https://spdx.org>`_.
Every source file should start with a comment indicating its license:

``// SPDX-License-Identifier: MIT``

The compiler does not validate that the license is part of the
`list allowed by SPDX <https://spdx.org/licenses/>`_, but
it does include the supplied string in the :ref:`bytecode metadata <metadata>`_.

If you do not want to specify a license or if the source code is
not open-source, please use the special value ``UNLICENSED``.

Supplying this comment of course does not free you from other
obligations related to licensing like having to mention
a specific license header in each source file or the
original copyright holder.

The comment is recognized by the compiler anywhere in the file at the
file level, but it is recommended to put it at the top of the file.

More information about how to use SPDX license identifiers
can be found at the `SPDX website <https://spdx.org/ids-how>`_.


.. index:: ! pragma

.. _pragma:

Pragmas
=========

关键字 ``pragma`` 版本标识指令，用来启用某些编译器检查， 版本 |pragma| 指令通常只对本文件有效，所以我们需要把这个版本 |pragma| 添加到项目中所有的源文件。
如果使用了 :ref:`import 导入<import>` 其他的文件, |pragma| 并不会从被导入的文件，加入到导入的文件中。

.. index:: ! pragma, version

.. _version_pragma:

版本标识
============================

为了避免未来被可能引入不兼容更新的编译器所编译，源文件可以（也应该）使用版本 |pragma| 所注解。
我们力图把这类不兼容变更做到尽可能小，但是，Solidity 本身就处在快速的发展之中，所以我们很难保证不引入修改语法的变更。
因此对含重大变更的版本，通读变更日志永远是好办法，变更日志通常会有版本号表明更新点。

版本号的形式通常是 ``0.x.0`` 或者 ``x.0.0``。

版本标识使用如下::

  pragma solidity ^0.5.2;

这样，源文件将既不允许低于 0.5.2 版本的编译器编译，
也不允许高于（包含） ``0.6.0`` 版本的编译器编译（第二个条件因使用 ``^`` 被添加）。
这种做法的考虑是，编译器在 0.6.0 版本之前不会有重大变更，所以可确保源代码始终按预期被编译。
上面例子中不固定编译器的具体版本号，因此编译器的补丁版也可以使用。

可以使用更复杂的规则来指定编译器的版本，表达式遵循 `npm <https://docs.npmjs.com/misc/semver>`_ 版本语义。

.. note::
  Pragma 是 pragmatic information 的简称，微软 Visual C++ `文档 <https://msdn.microsoft.com/zh-cn/library/d9x1s805.aspx>`_ 中译为标识。
  Solidity 中沿用 C ，C++ 等中的编译指令概念，用于告知编译器 **如何** 编译。
  ——译者注

.. note::
  使用版本标准不会改变编译器的版本，它不会启用或关闭任何编译器的功能。
  他仅仅是告知编译器去检查版本是否匹配， 如果不匹配，编译器就会提示一个错误。


ABI Coder Pragma
----------------

By using ``pragma abicoder v1`` or ``pragma abicoder v2`` you can
select between the two implementations of the ABI encoder and decoder.

The new ABI coder (v2) is able to encode and decode arbitrarily nested
arrays and structs. It might produce less optimal code and has not
received as much testing as the old encoder, but is considered
non-experimental as of Solidity 0.6.0. You still have to explicitly
activate it using ``pragma abicoder v2;``. Since it will be
activated by default starting from Solidity 0.8.0, there is the option to select
the old coder using ``pragma abicoder v1;``.

The set of types supported by the new encoder is a strict superset of
the ones supported by the old one. Contracts that use it can interact with ones
that do not without limitations. The reverse is possible only as long as the
non-``abicoder v2`` contract does not try to make calls that would require
decoding types only supported by the new encoder. The compiler can detect this
and will issue an error. Simply enabling ``abicoder v2`` for your contract is
enough to make the error go away.

.. note::
  This pragma applies to all the code defined in the file where it is activated,
  regardless of where that code ends up eventually. This means that a contract
  whose source file is selected to compile with ABI coder v1
  can still contain code that uses the new encoder
  by inheriting it from another contract. This is allowed if the new types are only
  used internally and not in external function signatures.

.. note::
  Up to Solidity 0.7.4, it was possible to select the ABI coder v2
  by using ``pragma experimental ABIEncoderV2``, but it was not possible
  to explicitly select coder v1 because it was the default.


.. index:: ! pragma, experimental

.. _experimental_pragma:

标注实验性功能
-------------------

第2个标注是用来标注实验性阶段的功能，它可以用来启用一些新的编译器功能或语法特性。
当前支持下面的一些实验性标注:


ABIEncoderV2
~~~~~~~~~~~~~~~~

从Solidity 0.7.4开始，  ABI coder v2 不在作为实验特性，而是可以通过``pragma abicoder v2``  启用，查看上面。

.. _smt_checker:

SMTChecker
~~~~~~~~~~~~~~

当我们使用自己编译的 Solidity 编译器（ 参考 :ref:`build instructions<smt_solvers_build>`） 的时候，这个模块可以启用。
在 Ubuntu PPA 发布的编译器的版本里 SMT solver 功能是启用的，但是其他版本如：solc-js,  Docker 镜像版本, Windows 二进制版本，静态编译的 Linux 二进制版本 是没有启用的。


.. note::
  译者注： SMT 全称是：Satisfiability modulo theories，用来“验证程序等价性”。


使用 ``pragma experimental SMTChecker;``, 就可以获得 SMT solver 额外的安全检查。但是这个模块目前不支持 Solidity 的全部语法特性，因此有可能输出一些警告信息。

.. index:: source file, ! import

.. _import:

导入其他源文件
============================

语法与语义
--------------------


Solidity 支持的导入语句来模块化代码，其语法跟 JavaScript（从 ES6 起）非常类似。
尽管 Solidity 不支持 `default export <https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#Description>`_  。

.. note::
  ES6 即 ECMAScript 6.0，ES6是 JavaScript 语言的下一代标准，已经在 2015 年 6 月正式发布。
  ——译者注

在全局层面上，可使用如下格式的导入语句：
::

  import "filename";

此语句将从 “filename” 中**导入所有的全局符号到当前全局作用域**中（不同于 ES6，Solidity 是向后兼容的）。

这种形式已经不建议使用，因为它会无法预测地污染当前命名空间。
如果在“filename”中添加新的符号，则会自动添加出现在所有导入 “filename” 的文件中。 更好的方式是明确导入的具体
符号。

向下面这样，创建了新的 ``symbolName`` 全局符号，他的成员都来自与导入的 ``"filename"`` 文件中的全局符号，如：
::

  import * as symbolName from "filename";

然后所有全局符号都以``symbolName.symbol``格式提供。
此语法的变体不属于ES6，但可能有用：

::

  import "filename" as symbolName;

它等价于 ``import * as symbolName from "filename";``。


如果存在命名冲突，则可以在导入时重命名符号。例如，下面的代码创建了新的全局符号 ``alias`` 和 ``symbol2`` ，引用的
``symbol1`` 和 ``symbol2`` 来自 "filename" 。

::

  import {symbol1 as alias, symbol2} from "filename";

路径
-----

上文中的 filename 总是会按路径来处理，以 ``/`` 作为目录分割符、以 ``.`` 标示当前目录、以 ``..`` 表示父目录。 
当 ``.`` 或 ``..`` 后面跟随的字符是 ``/`` 时，它们才能被当做当前目录或父目录。
只有路径以当前目录 ``.`` 或父目录 ``..`` 开头时，才能被视为相对路径。


用 ``import "./filename" as symbolName;`` 语句导入当前源文件同目录下的文件 ``filename`` 。
如果用 ``import "filename" as symbolName;`` 代替，可能会引入不同的（如在全局 ``include directory`` 中）文件。

最终导入哪个文件取决于编译器（见下文 :ref:`import-compiler`）到底是怎样解析路径的。
通常，目录层次不必严格映射到本地文件系统，
它也可以映射到能通过诸如 ipfs，http 或者 git 发现的资源。

.. note::
    通常使用相对引用 ``import "./filename.sol";`` 并且避免使用 ``..`` ，后面这种方式可以使用全局路径并设置映射，下面会有解释。

.. _import-compiler:

在实际的编译器中使用
-----------------------

当运行编译器时，它不仅能指定如何发现路径的第一个元素，还可指定路径前缀 |remapping|。
例如，``github.com/ethereum/dapp-bin/library`` 会被重映射到 ``/usr/local/dapp-bin/library`` ，
此时编译器将从重映射位置读取文件。如果重映射到多个路径，优先尝试重映射路径最长的一个。
这允许将比如 ``""`` 被映射到 ``"/usr/local/include/solidity"`` 来进行“回退重映射”。
同时，这些重映射可取决于上下文，允许你配置要导入的包，比如同一个库的不同版本。

**solc**:

对于 solc（命令行编译器），这些重映射以 ``context:prefix=target`` 形式的参数提供。
其中，``context:`` 和 ``=target`` 部分是可选的（此时 target 默认为 prefix ）。
所有重映射的值都是被编译过的常规文件（包括他们的依赖），这个机制完全是向后兼容的（只要文件名不包含 = 或 : ），
因此这不是一个破坏性修改。
在 ``content`` 目录或其子目录中的源码文件中，所有导入语句里以 ``prefix`` 开头的导入文件都将被用 ``target`` 替换 ``prefix`` 来重定向。

举个例子，如果你已克隆 ``github.com/ethereum/dapp-bin/`` 到本地 ``/usr/local/dapp-bin`` ，
可在源文件中使用：

::

  import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;

然后运行编译器：

.. code-block:: bash

  solc github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ source.sol

举个更复杂的例子，假设你依赖了一些使用了非常旧版本的 dapp-bin 的模块。
旧版本的 dapp-bin 已经被 checkout 到 ``/usr/local/dapp-bin_old`` ，此时你可使用：

.. code-block:: bash

  solc module1:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ \
  module2:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin_old/ \
  source.sol

这样， ``module2`` 中的所有导入都指向旧版本，而 ``module1`` 中的导入则获取新版本。


.. note::

  solc 只允许包含来自特定目录的文件：它们必须位于显式地指定的源文件目录（或子目录）中，或者重映射的目标目录（或子目录）中。
  如果你想直接用绝对路径来包含文件，只需添加重映射 ``/=/``。

如果有多个重映射指向一个有效文件，那么具有最长公共前缀的重映射会被选用。

**Remix**:

`Remix <https://remix.ethereum.org/>`_ 提供一个为 github 源代码平台的自动重映射，它将通过网络自动获取文件：
比如，你可以使用:

.. code-block::

  import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;

导入一个 map 迭代器。

未来， Remix 可能支持其他源代码平台。

.. index:: ! comment, natspec

注释
========

可以使用单行注释（``//``）和多行注释（``/*...*/``）

::

  // 这是一个单行注释。

  /*
  这是一个
  多行注释。
  */


.. note::
  
  单行注释由任何 unicode 行终止符(如采用utf8编码的：LF，VF，FF，CR，NEL，LS或PS) 终止。 在注释之后终止符代码仍然是源码的一部分。如果它不是ascii符号（NEL，LS和PS），它会导致解析器错误。

此外，有另一种注释称为 natspec 注释，可参考 :ref:`风格指南- 描述注释 <natspec>`。


它们是用三个反斜杠（``///``）或双星号开头的块（``/** ... */``）书写，它们应该直接在函数声明或语句上使用。
可在注释中使用 `Doxygen <https://en.wikipedia.org/wiki/Doxygen>`_ 样式的标签来文档化函数、
标注形式校验通过的条件，和提供一个当用户试图调用一个函数时显示给用户的 **确认性文字**。

在下面的例子中，我们记录了合约的标题，并解释了两个传入参书和两个返回值的翻译。

::
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.21 <0.8.0;

  /** @title 形状计算器。 */
  contract tinyCalculator {
      /** @dev 求矩形表明面积与周长。
      * @param w 矩形宽度。
      * @param h 矩形高度。
      * @return s 求得表面积。
      * @return p 求得周长。
      */
      function rectangle(uint w, uint h) returns (uint s, uint p) {
          s = w * h;
          p = 2 * (w + h);
      }
  }