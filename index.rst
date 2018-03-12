Solidity
========

.. image:: logo.svg
    :width: 120px
    :alt: Solidity logo
    :align: center

Solidity 是一门面向合约的、为实现智能合约而创建的高级编程语言。这门语言受到了 C++，Python 和 Javascript 语言的影响，设计的目的是能在以太坊虚拟机（EVM）上运行。

Solidity 是静态类型语言，支持继承、库和复杂的用户定义类型等特性。

下面您将会看到，使用 Solidity 语言，可以为投票、众筹、秘密竞价（盲拍）、多重签名的钱包以及其他应用创建合约。

.. note::
    目前尝试 Solidity 编程的最好的方式是使用 `Remix <https://remix.ethereum.org/>`_ （需要时间加载，请耐心等待）。Remix 是一个基于 Web 的 IDE，它可以让你编写 Solidity 智能合约，然后部署并运行该智能合约。

.. warning::
    因为软件是人编写的，就会有 bug，所以，创建智能合约也应该遵循软件开发领域熟知的最佳实践。这些实践包括代码审查、测试、审计和正确性证明。也请注意，有时候用户在代码方面比软件的作者更谙熟。最后，区块链本身有些东西需要留意，请参考 :ref:`security_considerations`。

翻译
------------

本文档由社区志愿者翻译成多种语言，但是 `英语版本 <https://github.com/ethereum/solidity/blob/develop/docs/index.rst>`_ 作为主要参考。

* `简体中文版 <https://solidity-cn.readthedocs.io/>`_
* `西班牙语版 <https://solidity-es.readthedocs.io>`_
* `俄语版 <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (已过时)


有用的链接
------------

* `以太坊官网 <https://ethereum.org>`_
* `变更日志 <https://github.com/ethereum/solidity/blob/develop/Changelog.md>`_
* `故事需求列表 <https://www.pivotaltracker.com/n/projects/1189488>`_
* `源代码 <https://github.com/ethereum/solidity/>`_
* `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_
* `Gitter 聊天 <https://gitter.im/ethereum/solidity/>`_

可用的 Solidity 集成
-------------------------------

* `Remix <https://remix.ethereum.org/>`_
    基于浏览器的 IDE，集成了编译器和 Solidity 运行时环境，不需要服务端组件。

* `IntelliJ IDEA plugin <https://plugins.jetbrains.com/plugin/9475-intellij-solidity>`_
    IntelliJ IDEA 的 Solidity 插件（可用于其他所有的 JetBrains IDE）

* `Visual Studio Extension <https://visualstudiogallery.msdn.microsoft.com/96221853-33c4-4531-bdd5-d2ea5acc4799/>`_
    Microsoft Visual Studio 的 Solidity 插件，包含 Solidity 编译器。

* `Package for SublimeText — Solidity language syntax <https://packagecontrol.io/packages/Ethereum/>`_
    SublimeText 编辑器的语法高亮包。

* `Etheratom <https://github.com/0mkara/etheratom>`_
    Atom 编辑器的插件，支持高亮、编译和运行时环境（兼容后端节点和虚拟机）。

* `Atom Solidity Linter <https://atom.io/packages/linter-solidity>`_
    Atom 编辑器的插件，提供 Solidity 语言的 Lint 检查（静态检查）。

* `Atom Solium Linter <https://atom.io/packages/linter-solium>`_
    Atom 的可配置的 Solidty 静态检查器，基于 Solium。

* `Solium <https://github.com/duaraghav8/Solium/>`_
    一种静态检查器，识别和修复 Solidity 中的风格以及安全问题。
    
* `Solhint <https://github.com/protofire/solhint>`_
    一种静态检查器，提供安全和风格指南以及智能合约验证的最佳实践规则。

* `Visual Studio Code extension <http://juan.blanco.ws/solidity-contracts-in-visual-studio-code/>`_
    Microsoft Visual Studio Code 插件，包含语法高亮和 Solidity 编译器。

* `Emacs Solidity <https://github.com/ethereum/emacs-solidity/>`_
    Emacs 编辑器的插件，提供语法高亮和编译错误报告。

* `Vim Solidity <https://github.com/tomlion/vim-solidity/>`_
    Vim 编辑器的插件，提供语法高亮。

* `Vim Syntastic <https://github.com/scrooloose/syntastic>`_
    Vim 编辑器的插件，提供编译检查。

不再维护:

* `Mix IDE <https://github.com/ethereum/mix/>`_
    基于 Qt 的 IDE，可以设计、调试和测试 Solidity 智能合约。

* `Ethereum Studio <https://live.ether.camp/>`_		
    专门的网页 IDE，也提供一个完整以太坊环境的脚本访问。

Solidity 工具列表
--------------

* `Dapp <https://dapp.readthedocs.io>`_
    Solidity 语言的构建工具、包管理器以及部署助手。

* `Solidity REPL <https://github.com/raineorshine/solidity-repl>`_
    一个命令行控制台，可以让你立刻尝试 Solidity 语言。

* `solgraph <https://github.com/raineorshine/solgraph>`_
    可视化的 Solidity 控制流，并能标明潜在的安全漏洞。

* `evmdis <https://github.com/Arachnid/evmdis>`_
    EVM 反汇编程序，可以执行字节码的静态分析，能提供比 EVM 操作更高级的抽象。

* `Doxity <https://github.com/DigixGlobal/doxity>`_
    Solidity 语言的文档生成器。

第三方 Solidity 解析器和语法
-----------------------------------------

* `solidity-parser <https://github.com/ConsenSys/solidity-parser>`_
    JavaScript 的 Solidity 解析器

* `Solidity Grammar for ANTLR 4 <https://github.com/federicobond/solidity-antlr4>`_
    ANTLR 4 解析器生成器的 Solidity 语法

语言文档
----------------------

下面的页面中，我们首先会看到一个使用 Solidity 写的 :ref:`简单智能合约 <simple-smart-contract>`，随后讲解 :ref:`区块链 <blockchain-basics>` 基础，然后是 :ref:`以太坊虚拟机 <the-ethereum-virtual-machine>` 。

下一节会通过给出有用的 :ref:`合约样例 <voting>` ，解释 Solidity 的几个*特性* ，记住你都可以 `在你的浏览器中 <https://remix.ethereum.org>`_ 尝试这些合约！

最后也是最长的一节会深入讲解 Solidity 的所有方面。

如果还有问题，你可以尝试搜索或在 `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_ 上提问，或者到我们的`gitter 频道 <https://gitter.im/ethereum/solidity/>`_ 来。随时欢迎改善 Solidity 或本文档的想法！

目录
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst
   solidity-in-depth.rst
   security-considerations.rst
   using-the-compiler.rst
   metadata.rst
   abi-spec.rst
   julia.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   frequently-asked-questions.rst
