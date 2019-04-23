Solidity 0.5.7 翻译
================================

.. image:: logo.svg
    :width: 120px
    :alt: Solidity logo
    :align: center


译者说明：本文档根据当前`最新官方版本v0.5.7 <https://solidity.readthedocs.io/>`_ 进行翻译，本翻译最初 `HiBlock <http://hiblock.one/>`_ 社区发起，后经过`深入浅出区块链社区 <https://learnblockchain.cn/>`_ 社区成员根据最新版本补充翻译。

本中文文档大部分情况下，英中直译，但有时为了更好的理解也会使用意译，如需转载请联系Tiny熊（微信：xlbxiong）.

Solidity 是一门面向合约的、为实现智能合约而创建的高级编程语言。这门语言受到了 C++，Python 和 Javascript 语言的影响，设计的目的是能在 `以太坊虚拟机（EVM） <https://learnblockchain.cn/2019/04/09/easy-evm/>`_ 上运行。

Solidity 是静态类型语言，支持继承、库和复杂的用户定义类型等特性。

在部署合约时，应该尽量使用最新版本，因为新版本会有一些重大的新特性以及bug修复。

语言文档
----------------------

如果你才接触智能合约概念，推荐从一些 `简单的Solidity合约例子 <introduction-to-smart-contracts.html#simple-smart-contract>`_ 开始，当你想开始尝试了解更多的细节，可以
学习 `合约样例 <solidity-by-example.html>`_ 和 `深入理解Solidity <solidity-in-depth.html>`_ 。

你还可以进一步阅读 :ref:`区块链 <blockchain-basics>` 基础，然后是 :ref:`以太坊虚拟机 <the-ethereum-virtual-machine>` 。


.. note::
    目前尝试 Solidity 编程的最好的方式是使用 `Remix <https://remix.ethereum.org/>`_ （需要时间加载，请耐心等待）。Remix 是一个基于 Web 浏览器的 IDE，它可以让你编写 Solidity 智能合约，然后部署并运行该智能合约。

.. warning::
    因为软件是人编写的，就会有 bug，所以，创建智能合约也应该遵循软件开发领域熟知的最佳实践。这些实践包括代码审查、测试、审计和正确性证明。也请注意，有时候用户在代码方面比软件的作者更谙熟。最后，区块链本身有些东西需要留意，请参考 :ref:`security_considerations`。

如果还有问题，你可以尝试搜索或在 `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_ 上提问，或者到我们的`gitter 频道 <https://gitter.im/ethereum/solidity/>`_ 来。随时欢迎改善 Solidity 或本文档的想法！

翻译版本
----------------------

本文档由社区志愿者翻译成多种语言，但是 `英语版本 <https://github.com/ethereum/solidity/blob/develop/docs/index.rst>`_ 作为主要参考。

* `简体中文版 <https://solidity-cn.readthedocs.io/>`_ （由 `HiBlock社区 <http://hiblock.one/>`_ `深入浅出区块链社区 <https://learnblockchain.cn/>`_ 贡献者翻译）
* `西班牙语版 <https://solidity-es.readthedocs.io>`_
* `俄语版 <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (已过时)
* `韩语版 <http://solidity-kr.readthedocs.io>`_ (in progress)


目录
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 4
   :caption: Solidity 手册

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst
   solidity-in-depth.rst
   security-considerations.rst
   using-the-compiler.rst
   metadata.rst
   abi-spec.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   frequently-asked-questions.rst
