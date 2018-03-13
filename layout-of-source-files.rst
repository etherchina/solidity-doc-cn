********************************
Solidity 源文件结构
********************************

源文件中可以包含任意多个合约定义、导入指令和杂注指令。

.. index:: ! pragma, version

.. _version_pragma:

版本杂注
============================

为了避免未来被可能引入不兼容变更的编译器所编译，源文件可以（也应该）被所谓的版本杂注所注解。
我们力图把这类变更做到尽可能小，特别是，我们需要以一种当修改语义时必须同步修改语法的方式引入变更，当然这有时候也难以做到。
因此，至少对含重大变更的版本，通读变更日志永远是好办法。
这些版本的版本号始终是 ``0.x.0`` 或者 ``x.0.0`` 形式。

版本杂注使用如下::

  pragma solidity ^0.4.0;

这样，源文件将既不允许被低于 0.4.0 版本的编译器编译，
也不允许被高于（包含） ``0.5.0`` 版本的编译器编译（第二个条件因使用 ``^`` 被添加）。
这种做法的考虑是，编译器在 0.5.0 版本之前不会有重大变更，所以可确保源代码始终按预期被编译。
上面例子中不固定编译器的具体版本号，因此编译器的补丁版也可以使用。

可以对编译器版本指定更复杂的规则，表达式遵循 `npm <https://docs.npmjs.com/misc/semver>`_ 版本语义。

.. note::
  Pragma 是 pragmatic information 的简称，微软 Visual C++ `文档 <https://msdn.microsoft.com/zh-cn/library/d9x1s805.aspx>`_ 中中译为杂注。
  Solidity 中沿用 C ，C++ 等中的编译指令概念，用于告知编译器 **如何** 编译。
  ——译者注

.. index:: source file, ! import

.. _import:

导入其他源文件
============================

语法与语义
--------------------

虽然 Solidity 不知道 "default export" 为何物，
但是 Solidity 所支持的导入语句，其语法同 JavaScript（从 ES6 起）非常类似。

.. note::
  ES6 即 ECMAScript 6.0，ES6是 JavaScript 语言的下一代标准，已经在 2015 年 6 月正式发布。
  ——译者注

在全局层面上，可使用如下格式的导入语句：
::

  import "filename";

此语句将从 “filename” 中导入所有的全局符号到当前全局作用域中（不同于 ES6，Solidity 是向后兼容的）。

::

  import * as symbolName from "filename";

...创建一个新的全局符号 ``symbolName``，其成员均来自 ``"filename"`` 中全局符号。

::

  import {symbol1 as alias, symbol2} from "filename";

...创建新的全局符号 ``alias`` 和 ``symbol2``。``symbol1`` 和 ``symbol2`` 分别从 ``"filename"`` 引入。

另一种语法不属于 ES6，但或许更简便：

::

  import "filename" as symbolName;

这条语句等同于 ``import * as symbolName from "filename";``。

路径
-----

上面 ``filename`` 始终被视为以 ``/`` 当做目录分割符、``.`` 当做当前目录和``..`` 当做父目录的路径。
路径以当前目录 ``.`` 或父目录 ``..`` 开头时，才能被视为相对路径，否则一律视为绝对路径。
但只有当``.`` 或 ``..`` 后面跟随的字符是 ``/`` 时，它们才能被当做当前目录或父目录。


用 ``import "./x" as x;`` 语句导入当前源文件同目录下的文件 ``x`` 。
如果用 ``import "x" as x;``代替，可引入不同的文件（在全局 ``include directory`` 中）。

最终导入哪个文件取决于编译器（见下文）到底是怎样解析路径的。
通常，目录层次不必严格映射到本地文件系统，
它也可以映射到通过比如 ipfs，http 或者 git 发现的资源。

在实际的编译器中使用
-----------------------

当编译器被调用后，它不仅能指定如何发现路径的第一个元素，还可指定路径前缀重映射。
例如，``github.com/ethereum/dapp-bin/library`` 被重映射到 ``/usr/local/dapp-bin/library`` ，
此时编译器将从重映射位置读取文件。如果指定了多个重映射，优先尝试重映射路径最长的一个。
这允许将比如``""`` 被映射到 ``"/usr/local/include/solidity"``来进行“回退映射”。
同时，这些重映射可取决于上下文，允许你配置要导入的包，比如不同版本的同名库。

**solc**:

对于 solc（命令行编译器），这些重映射以 ``context:prefix=target`` 参数形式提供。
其中，``context:`` 和 ``=target`` 部分是可选的（此时 target 默认为 prefix ）。
所有重映射值都是常规文件被编译（包括他们的依赖），这个机制完全是向后兼容的（只要没文件名包含 = 或 : ），
因此这不是一个破坏性修改。
在 ``content`` 目录下或之下的所有以 ``prefix`` 开头的导入文件都将被用 ``target`` 替换 ``prefix`` 来重定向。

举个例子，如果你已克隆 ``github.com/ethereum/dapp-bin/`` 到本地 ``/usr/local/dapp-bin`` ，可在源文件中如下使用导入：

::

  import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;

然后运行编译器：

.. code-block:: bash

  solc github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ source.sol

举个更复杂的例子，假设你依靠很旧的 dapp-bin 版本上的一些模块。
旧版本 dapp-bin 签出到 ``/usr/local/dapp-bin_old`` ，此时你可使用：

.. code-block:: bash

  solc module1:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ \
module2:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin_old/ \
source.sol

这样， ``module2`` 中所有导入都指向旧版本，而 ``module1`` 中的导入则获取新版本。

注意， solc 只允许包含来自特定目录的文件：它们必须位于显式指定源文件或重映射 target 中的目录（或子目录）中。
如果你想直接用绝对路径来包含文件，只需添加重映射 ``= /``。

如果有多个重映射指向一个有效文件，那选择最长公共前缀的重映射。

**Remix**:

`Remix <https://remix.ethereum.org/>`_ 提供一个为 github 源代码平台的自动重映射，它将通过网络自动获取文件：
比如，你可使用 ``import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;`` 导入一个 map 迭代器。

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


此外，有另一种注释称为 natspec 注释，其文档还尚未编写。
它们用三个反斜杠注释（``///``）或双星号开头的块注释（``/** ... */``），它们应该直接在函数声明或语句上使用。
可在注释中使用 `Doxygen <https://en.wikipedia.org/wiki/Doxygen>`_ 样式的标签来文档化函数、
标注形式校验通过的条件，和提供一个当用户试图调用一个函数时显示给用户的 **确认文本**。

在下面的例子中，记录合约的标题、两个入参和两个返回值的说明：

::

  pragma solidity ^0.4.0;

  /** @title Shape calculator. */
  contract shapeCalculator {
    /** @dev Calculates a rectangle's surface and perimeter.
    * @param w Width of the rectangle.
    * @param h Height of the rectangle.
    * @return s The calculated surface.
    * @return p The calculated perimeter.
    */
    function rectangle(uint w, uint h) returns (uint s, uint p) {
    s = w * h;
    p = 2 * (w + h);
    }
  }