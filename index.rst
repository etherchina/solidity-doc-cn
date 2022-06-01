Solidity 中文文档
================================

译者说明：这里是是 **Solidity官方推荐中文版**，本文档根据当前 `Solidity官方文档 <https://solidity.readthedocs.io/>`_ 最新版本（当前为v0.8.13）进行翻译。


Solidity中文翻译最初由 `HiBlock <http://hiblock.one/>`_ 社区发起，后由 `登链社区 <https://learnblockchain.cn/>`_ 社区持续维护更新。

翻译工作是一个持续的过程（这份文档依旧有部分未完成），我们热情邀请热爱区块链技术的小伙伴一起参与，欢迎加入我们 `翻译小组 <https://github.com/lbc-team>`_ 。

本中文文档大部分情况下，英中直译，但有时为了更好的理解也会使用意译，如需转载请联系Tiny熊（微信：xlbxiong）.


Solidity 是一门面向合约的、为实现智能合约而创建的高级编程语言。智能合约是管理以太坊状态里账户行为的程序。

Solidity是一种针对Ethereum虚拟机（EVM）设计的 `花括号语言 <https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Curly-bracket_languages>`_ 。
它受到了C++、Python和JavaScript的影响。你可以在 :doc:`语言影响 <language-influences>` 部分找到更多关于Solidity受到哪些语言启发的细节。

Solidity 是静态类型语言，支持继承、库和复杂的用户定义类型等特性。

在部署合约时，应该尽量使用最新版本，因为新版本会有一些重大的新特性以及bug修复（除特殊情况）。

.. warning::

  Solidity 每个大版本的设计，都会引入一些与之前版本不兼容的升级，详情可阅读 :doc:`0.8.0 更新列表 <080-breaking-changes>` ，  :doc:`0.7 更新列表 <070-breaking-changes>` , :doc:`0.6 更新列表 <060-breaking-changes>` , :doc:`0.5 更新列表 <050-breaking-changes>` 。


开始使用Solidity
----------------------

**1. 理解合约基础概念**

如果你才接触智能合约概念，推荐从智能合约的概念开始，可以参考：

* :ref:`简单的Solidity合约例子 <simple-smart-contract>`.
* :ref:`区块链基础 <blockchain-basics>`.
* :ref:`以太坊虚拟机 <the-ethereum-virtual-machine>`.


.. hint::
    译者注：理解智能合约及虚拟机是怎么运行，还可阅读这两篇文章   `完全理解以太坊智能合约 <https://learnblockchain.cn/2018/01/04/understanding-smart-contracts/>`_  及  `深入浅出以太坊虚拟机 <https://learnblockchain.cn/2019/04/09/easy-evm/>`_ 。

**2. 了解 Solidity**

一旦熟悉了基础概念，我们推荐人阅读 :doc:`"根据例子学习Solidity " <solidity-by-example>`
以及Solidity详解部分理解语言的核心概念。


**3. 安装Solidity编译器**

有好多方式可以安装Solidity编译器，你可以选择你喜欢的方式，跟着 :ref:`安装指引 <installing-solidity>` 进行安装。

.. hint::
    目前尝试 Solidity 编程的最好的方式是使用 `Remix <https://remix.ethereum.org/>`_ （需要时间加载，请耐心等待）。Remix 是一个基于 Web 浏览器的 IDE，它可以让你编写 Solidity 智能合约，然后部署并运行该智能合约。

.. warning::
    因为软件是人编写的，就会有 bug，所以，创建智能合约也应该遵循软件开发领域熟知的最佳实践。这些实践包括代码审查、测试、审计和正确性证明。也请注意，有时候用户在代码方面比软件的作者更谙熟。最后，区块链本身有些东西需要留意，请参考 :ref:`security_considerations`。

**4. 学习更多**

如果您想了解有关在以太坊上构建去中心化应用程序的更多信息，`以太坊开发者资源网站 <https://ethereum.org/en/developers/>`_ 及 `登链社区 - 以太坊分类 <https://learnblockchain.cn/categories/ethereum//>`_ 可以找到更多以太坊相关的文档、教程以及开发工具、库等。

如果还有问题，你可以尝试搜索或在 `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_ 上提问，或者到我们的 `gitter 频道 <https://gitter.im/ethereum/solidity/>`_ 来。随时欢迎改善 Solidity 或本文档的想法！


目录
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2
   :caption: 基础

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst

.. toctree::
   :maxdepth: 2
   :caption: Solidity 详解

   layout-of-source-files.rst
   structure-of-a-contract.rst
   types.rst
   units-and-global-variables.rst
   control-structures.rst
   contracts.rst
   assembly.rst
   cheatsheet.rst
   grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: 编译器

   using-the-compiler.rst
   analysing-compilation-output.rst
   ir-breaking-changes.rst

.. toctree::
   :maxdepth: 2
   :caption: 深入 Solidity 内部

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimizer.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: 补充材料

   050-breaking-changes.rst
   060-breaking-changes.rst
   070-breaking-changes.rst
   080-breaking-changes.rst
   natspec-format.rst
   security-considerations.rst
   smtchecker.rst
   resources.rst
   path-resolution.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   language-influences.rst
