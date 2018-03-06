********************************
Solidity源代码文件结构
********************************

源文件中可以包含任意多个合约定义、包括嵌入和杂注指令。 

.. index:: ! pragma, version

.. _version_pragma:

版本 Pragma 指令
============================

为了避免未来高版本的编译器可能带来的兼容性变化，源文件可以（也应该）被所谓的版本杂注所注解。 
我们尽量保持最小变更，特别是语义变化而引起语法变更的，但有时也很难避免。
因此，对于有重大变化的新版本，建议通读变更日志。
大版本的版本号始终是 ``0.x.0`` 或者 ``x.0.0`` 形式。  

版本杂注使用方式如下::

  pragma solidity ^0.4.0;

这样的源文件既不允许被低于版本 0.4.0 的编译器编译，也不允许被高于（含） ``0.5.0`` 版本的编译器编译（被 ``^`` 限制）。 
这种做法的考虑是，在版本 0.5.0 之前不会有任何重大改变，所以可确保源代码始终按照意愿被编译。
我们这里不固定编译器的具体版本号，因此编译器的补丁版也可以使用。

可以为编译器指定更复杂的编译规则，指令表达式遵循 `npm <https://docs.npmjs.com/misc/semver>`_ 版本语义。

.. note::
  Pragma 是 pragmatic information 的简称，微软 Visual C++ 官方文档中中译为杂注。 
  Solidity 中沿用 C ， C++ 等中的编译指令概念，用于告知编译器**如何**编译。
  - 译者注

.. index:: source file, ! import

.. _import:

导入其他源文件
============================

语法与语义
--------------------

虽然 Solidity 中没有 "default export" 语法，
但是 Solidity 支持非常类似于 JavaScript（从 ES6 起）中可用的导入语句。

在全局层面上，可使用如下格式的导入语句：
::

  import "filename";

此语句将从 “filename” 中导入所有的全局符号到当前全局作用域中（不同于 ES6 但 Solidity 向后兼容）。 

::

  import * as symbolName from "filename";

...创建一个新的全局符号 ``symbolName``，成员均是来自 ``"filename"`` 中全局符号。

::

  import {symbol1 as alias, symbol2} from "filename";

...创建新的全局符号 ``alias`` 和 ``symbol2``。``symbol1`` 和 ``symbol2`` 分别从 ``"filename"`` 引入。

另外一种语法不属于 ES6，但或许更方便一些：

::

  import "filename" as symbolName;

这条语句等同于 ``import * as symbolName from "filename";``。

路径
-----

上面 ``filename`` 始终被视为以 ``/`` 作为目录分割符的文件路径，如果以 ``.`` 或者 ``..`` 开头，则视为相对路径，否则视为绝对路径。
其中``.`` 表示当前目录，``..`` 表示父目录，但只有当 ``.`` 或 ``..`` 后面跟随的是 ``/`` 时才视为当前目录或父目录。


用 ``import "./x" as x;`` 语句导入当前源文件同目录下的文件 ``x`` 。 
如果用 ``import "x" as x;``来代替，可引入一个不同的文件（在全局 ``include directory`` 中）。

路径取决于编译器（如下）到底是怎样解析路径的。通常，目录层次不必严格映射到本地文件系统，它也可以映射到通过比如 ipfs，http 或者 git 发现的资源。

在实际的编译器中使用
-----------------------

当编译器被调用后，它不仅能指定如何发现路径的第一个元素，还可指定路径前缀重映射。
例如，``github.com/ethereum/dapp-bin/library`` 被重映射到 ``/usr/local/dapp-bin/library`` ，此时编译器从这里读取文件。
如果多个重映射被指定，优先尝试导入路径最长的一个。
它还可以将比如``""`` 被重映射到 ``"/usr/local/include/solidity"``来进行“回退映射”。
同时，这些重映射可取决于上下文，允许你配置要导入的包，比如不同版本的同名库。 


**solc**:


对于 solc（命令行编译器），这些重映射以 ``context:prefix=target`` 参数形式提供。
其中，``context`` 和 ``=target`` 可选（此时 target 默认为 prefix ）。
所有重映射值都是常规文件被编译（包括他们的依赖），这个机制完全是向后兼容的（只要没文件名包含 = 或 :），
因此不是一个突破性变化。在 ``content`` 目录下或之下的所有以 ``prefix`` 开头的导入文件
都将被用 ``target`` 替换 ``prefix`` 来重定向。

举个例子，如果想克隆 ``github.com/ethereum/dapp-bin/`` 到本地 ``/usr/local/dapp-bin`` ，可在源文件中如下使用：  

::

  import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;

再运行编译器：

.. code-block:: bash

  solc github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ source.sol

举个更复杂的例子，假设你想依赖非常旧版本的 dapp-bin 上的一些模块。 
旧版本 dapp-bin 签出到 ``/usr/local/dapp-bin_old`` ，那么可使用：

.. code-block:: bash

  solc module1:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ \
       module2:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin_old/ \
       source.sol

以便 ``module2`` 下所有导入都指向旧版本，而 ``module1`` 指向新版本。

注意， solc 只允许包含来自特定目录的文件：

它们必须位于显式指定源文件或重映射目标中的一个目录（或子目录）中。
如果你想直接绝对包括，只需添加重映射 ``= /``。

如果有多个重映射指向一个有效文件，那选择最长公共前缀的重映射。

**Remix**:

`Remix <https://remix.ethereum.org/>`_ 提供了一个为 github 的自动重映射，将通过网络自动获取文件。
如可使用 ``import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;`` 导入一个键迭代器。

以后可能支持其他源码平台。


.. index:: ! comment, natspec

注释
========

可以使用单行注释（``//``）和多行注释（``/*...*/``）

::

  // This is a single-line comment.

  /*
  This is a
  multi-line comment.
  */


此外，有另一种注释称为 natspec 注释，其文档尚未编写。 
它们用三个反斜杠（``///``）或双星块（``/** ... */``）编辑，它应直接在函数声明或语句上使用。
可在注释中使用 `Doxygen <https://en.wikipedia.org/wiki/Doxygen>`_ 样式
的标签来文档化函数，标注形式校验通过的条件，并提供一个 **确认信息**，可在用户尝试调用一个函数时提示。  

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
