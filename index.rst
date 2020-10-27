Solidity 最新中文文档
================================

.. image:: logo.svg
    :width: 120px
    :alt: Solidity logo
    :align: center


译者说明：本文档根据当前 `Solidity官方文档 <https://solidity.readthedocs.io/>`_ 最新版本（当前为v0.7.1）进行翻译。
并且由于高质量的翻译和及时的更新，现在是 **Solidity官方推荐中文版** 。

Solidity中文翻译最初由 `HiBlock <http://hiblock.one/>`_ 社区发起，后由 `登链社区 <https://learnblockchain.cn/>`_ 社区持续维护更新。
本翻译工作还得到 `Cell Network <http://www.cellnetwork.io/?utm_souce=learnblockchain>`_ 赞助。

翻译工作是一个持续的过程（这份文档依旧有部分未完成），我们热情邀请热爱区块链技术的小伙伴一起参与，欢迎加入我们 `翻译小组 <https://github.com/lbc-team>`_ 。

本中文文档大部分情况下，英中直译，但有时为了更好的理解也会使用意译，如需转载请联系Tiny熊（微信：xlbxiong）.


Solidity 是一门面向合约的、为实现智能合约而创建的高级编程语言。这门语言受到了 C++，Python 和 Javascript 语言的影响，设计的目的是能在 `以太坊虚拟机（EVM） <https://learnblockchain.cn/2019/04/09/easy-evm/>`_ 上运行。

Solidity 是静态类型语言，支持继承、库和复杂的用户定义类型等特性。

在部署合约时，应该尽量使用最新版本，因为新版本会有一些重大的新特性以及bug修复。

.. warning::

  Solidity 发布 0.5.x 有很多与之前版本不兼容的升级，理解更新可阅读 :doc:`更新列表 <050-breaking-changes>`.
  Solidity 发布 0.6.x 有很多与之前版本不兼容的升级，理解更新可阅读 :doc:`更新列表 <060-breaking-changes>`.

语言文档
----------------------

如果你才接触智能合约概念，推荐从一些 `简单的Solidity合约例子 <introduction-to-smart-contracts.html#simple-smart-contract>`_ 开始，当你想开始尝试了解更多的细节，可以
学习 :doc:`合约样例 <solidity-by-example>` 和 "Solidity详解" 部分 。

你还可以进一步阅读 :ref:`区块链 <blockchain-basics>` 基础，然后是 :ref:`以太坊虚拟机 <the-ethereum-virtual-machine>` 。

.. hint::
    译者注：理解智能合约及虚拟机是怎么运行，还可阅读这两篇文章   `完全理解以太坊智能合约 <https://learnblockchain.cn/2018/01/04/understanding-smart-contracts/>`_  及  `深入浅出以太坊虚拟机 <https://learnblockchain.cn/2019/04/09/easy-evm/>`_ 。

.. hint::
    目前尝试 Solidity 编程的最好的方式是使用 `Remix <https://remix.ethereum.org/>`_ （需要时间加载，请耐心等待）。Remix 是一个基于 Web 浏览器的 IDE，它可以让你编写 Solidity 智能合约，然后部署并运行该智能合约。

.. warning::
    因为软件是人编写的，就会有 bug，所以，创建智能合约也应该遵循软件开发领域熟知的最佳实践。这些实践包括代码审查、测试、审计和正确性证明。也请注意，有时候用户在代码方面比软件的作者更谙熟。最后，区块链本身有些东西需要留意，请参考 :ref:`security_considerations`。

如果还有问题，你可以尝试搜索或在 `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_ 上提问，或者到我们的 `gitter 频道 <https://gitter.im/ethereum/solidity/>`_ 来。随时欢迎改善 Solidity 或本文档的想法！

翻译版本
----------------------

本文档由社区志愿者翻译成多种语言，但是 `英语版本 <https://github.com/ethereum/solidity/blob/develop/docs/index.rst>`_ 作为主要参考。

* `简体中文版 <https://learnblockchain.cn/docs/solidity/>`_ （由 `登链社区 <https://learnblockchain.cn/>`_ `HiBlock社区 <http://hiblock.one/>`_  贡献者翻译）
* `西班牙语版 <https://solidity-es.readthedocs.io>`_
* `俄语版 <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (已过时)
* `韩语版 <https://solidity-kr.readthedocs.io>`_ (in progress)


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
   miscellaneous.rst
   cheatsheet.rst
   grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: 深入 Solidity 内部

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimiser.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: 补充材料

   050-breaking-changes.rst
   060-breaking-changes.rst
   070-breaking-changes.rst
   natspec-format.rst
   security-considerations.rst
   resources.rst
   using-the-compiler.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   brand-guide.rst
