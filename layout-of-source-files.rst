.. include:: glossaries.rst

********************************
Solidity 源文件结构
********************************

源文件中可以包含任意数量的
:ref:`合约定义<contract_structure>` , 源文件引入 , :ref:`pragma<pragma>` 、 :ref:`using for<using-for>` 指令和 :ref:`struct<structs>` , :ref:`enum<enums>` , :ref:`function<functions>` , :ref:`error<errors>`
以及 :ref:`常量<constants>` 定义。


SPDX 版权许可标识
=======================

如果开源智能合约的源代码，就可以更好地建立对其的信任。由于提供源代码总是涉及到版权方面的法律问题，Solidity编译器鼓励使用机器可读的 `SPDX 许可标识 <https://spdx.org>`_ 。
每个源文件都应该以这样的注释开始以说明其版权许可证。


``// SPDX-License-Identifier: MIT``

编译器不会验证许可证是否属于 `SPDX版权许可列表 <https://spdx.org/licenses/>`_ ，但它会在 :ref:`bytecode metadata <metadata>` 中包含提供的字符串。

如果你不想指定一个许可证，或者如果源代码不开源，请使用特殊值 ``UNLICENSED`` 。

请注意， ``UNLICENSED`` （不存在于SPDX许可证列表中）与 ``UNLICENSE`` （授予所有人所有权利）不同。


Solidity 遵循 `the npm 建议 <https://docs.npmjs.com/cli/v7/configuring-npm/package-json#license>`_ 。
当然，提供这个注释并不能使你摆脱与许可有关的其他义务，比如必须在每个源文件中声明一个特定的许可标识符或原始版权人。

版权注释在文件的任何位置都可以被编译器识别，但建议把它放在文件的顶部。

关于如何使用SPDX许可标识符的更多信息可以在 `SPDX website <https://spdx.org/ids-how>`_ 找到。



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

可以使用更复杂的规则来指定编译器的版本，表达式遵循 `npm <https://docs.npmjs.com/cli/v6/using-npm/semver>`_ 版本语义。

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

.. index:: source file, ! import, module, source unit

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

.. code-block:: solidity

  import "filename";

``filename`` 部分称为导入路径（ *import path*）。
此语句将从 “filename” 中**导入所有的全局符号到当前全局作用域**中（不同于 ES6，Solidity 是向后兼容的）。

这种形式已经不建议使用，因为它会无法预测地污染当前命名空间。
如果在“filename”中添加新的符号，则会自动添加出现在所有导入 “filename” 的文件中。 更好的方式是明确导入的具体
符号。

像下面这样，创建了新的 ``symbolName`` 全局符号，他的成员都来自与导入的 ``"filename"`` 文件中的全局符号，如：
.. code-block:: solidity

  import * as symbolName from "filename";

然后所有全局符号都以``symbolName.symbol``格式提供。
此语法的变体不属于ES6，但可能有用：

.. code-block:: solidity

  import "filename" as symbolName;

它等价于 ``import * as symbolName from "filename";``。


如果存在命名冲突，则可以在导入时重命名符号。例如，下面的代码创建了新的全局符号 ``alias`` 和 ``symbol2`` ，引用的 ``symbol1`` 和 ``symbol2`` 来自 "filename" 。

.. code-block:: solidity

  import {symbol1 as alias, symbol2} from "filename";

.. index:: virtual filesystem, source unit name, import; path, filesystem path, import callback, Remix IDE

导入路径
---------

In order to be able to support reproducible builds on all platforms, the Solidity compiler has to
abstract away the details of the filesystem where source files are stored.
For this reason import paths do not refer directly to files in the host filesystem.
Instead the compiler maintains an internal database (*virtual filesystem* or *VFS* for short) where
each source unit is assigned a unique *source unit name* which is an opaque and unstructured identifier.
The import path specified in an import statement is translated into a source unit name and used to
find the corresponding source unit in this database.

Using the :ref:`Standard JSON <compiler-api>` API it is possible to directly provide the names and
content of all the source files as a part of the compiler input.
In this case source unit names are truly arbitrary.
If, however, you want the compiler to automatically find and load source code into the VFS, your
source unit names need to be structured in a way that makes it possible for an :ref:`import callback
<import-callback>` to locate them.
When using the command-line compiler the default import callback supports only loading source code
from the host filesystem, which means that your source unit names must be paths.
Some environments provide custom callbacks that are more versatile.
For example the `Remix IDE <https://remix.ethereum.org/>`_ provides one that
lets you `import files from HTTP, IPFS and Swarm URLs or refer directly to packages in NPM registry
<https://remix-ide.readthedocs.io/en/latest/import.html>`_.

For a complete description of the virtual filesystem and the path resolution logic used by the compiler see :ref:`Path Resolution <path-resolution>`.


.. index:: ! comment, natspec

注释
========

可以使用单行注释（``//``）和多行注释（``/*...*/``）

.. code-block:: solidity

  // 这是一个单行注释。

  /*
  这是一个
  多行注释。
  */


.. note::
  
  单行注释由任何 unicode 行终止符(如采用UTF-8编码的：LF，VF，FF，CR，NEL，LS或PS) 终止。 在注释之后终止符代码仍然是源码的一部分。如果它不是ASCII符号（NEL，LS和PS），它会导致解析器错误。

此外，有另一种注释称为 NatSpec 注释，详细可参考 :ref:`编程风格指南<style_guide_natspec>` 。

NatSpec注释使用 3 斜杠(``///``) 或两个星号注释，应该直接在函数声明和语句上方使用。
