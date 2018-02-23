Solidity
========

.. image:: logo.svg
    :width: 120px
    :alt: Solidity logo
    :align: center

Solidity是一门面向合约的、为实现智能合约而创建的高级编程语言。这门语言受C++，Python和Javascript语言的影响，设计的目的是能在以太坊虚拟机（EVM）上运行。

Solidity是静态类型语言，支持继承、库和复杂的用户定义类型等特性。

下面您将会看到，使用Solidity语言，可以为投票、众筹、秘密竞价（盲拍）、多重签名的钱包以及其他应用创建合约。

.. note::
    目前尝试Solidity编程的最好的方式是使用
    `Remix <https://remix.ethereum.org/>`_
    （需要时间加载，请耐心等待）。

.. warning::
    因为软件是人写的，就会有bug，所以，创建智能合约也应该遵循软件开发领域熟知的最佳实践。这些实践包括代码审查、测试、审计和正确性证明。

    同时也要注意：代码的用户有时比作者更有信心。
    最后，区块链本身有些东西需要留意，请参考 :ref:`security_considerations`。

翻译 -- 测试
------------

本文档由社区志愿者翻译成多种语言，但是英语版本作为主要参考。

* `简体中文版 <https://solidity-doc-cn.readthedocs.io/>`_
* `西班牙语版 <https://solidity-es.readthedocs.io>`_
* `俄语版 <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (已过时)


有用的链接
------------

* `以太坊官网 <https://ethereum.org>`_

* `变更日志 <https://github.com/ethereum/solidity/blob/develop/Changelog.md>`_

* `故事需求列表 <https://www.pivotaltracker.com/n/projects/1189488>`_

* `源代码 <https://github.com/ethereum/solidity/>`_

* `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_

* `Gitter聊天 <https://gitter.im/ethereum/solidity/>`_

可用的Solidity集成
-------------------------------

* `Remix <https://remix.ethereum.org/>`_
    基于浏览器的IDE，集成了编译器和Solidity运行时环境，不需要服务端组件。

* `IntelliJ IDEA plugin <https://plugins.jetbrains.com/plugin/9475-intellij-solidity>`_
    IntelliJ IDEA的Solidity插件（可用于其他所有的JetBrains IDEs）

* `Visual Studio Extension <https://visualstudiogallery.msdn.microsoft.com/96221853-33c4-4531-bdd5-d2ea5acc4799/>`_
    Microsoft Visual Studio的Solidity插件，包含Solidity编译器。

* `Package for SublimeText — Solidity language syntax <https://packagecontrol.io/packages/Ethereum/>`_
    SublimeText编辑器的语法高亮包。

* `Etheratom <https://github.com/0mkara/etheratom>`_
    Atom编辑器的插件，支持高亮、编译和运行时环境（后端节点，虚拟机兼容）。

* `Atom Solidity Linter <https://atom.io/packages/linter-solidity>`_
    Atom编辑器的插件，提供Solidity语言的Lint检查（静态检查）。

* `Atom Solium Linter <https://atom.io/packages/linter-solium>`_
    Atom的可配置的Solidty静态检查器，基于Solium。

* `Solium <https://github.com/duaraghav8/Solium/>`_
    一种静态检查器，识别和修复Solidity中的风格以及安全问题。
    
* `Solhint <https://github.com/protofire/solhint>`_
    一种静态检查器，提供安全和风格指南以及智能合约验证的最佳实践规则。

* `Visual Studio Code extension <http://juan.blanco.ws/solidity-contracts-in-visual-studio-code/>`_
    Microsoft Visual Studio Code插件，包含语法高亮和Solidity编译器。

* `Emacs Solidity <https://github.com/ethereum/emacs-solidity/>`_
    Emacs编辑器的插件，提供语法高亮和编译错误报告。

* `Vim Solidity <https://github.com/tomlion/vim-solidity/>`_
    Vim编辑器的插件，提供语法高亮。

* `Vim Syntastic <https://github.com/scrooloose/syntastic>`_
    Vim编辑器的插件，提供编译检查。

停止使用:

* `Mix IDE <https://github.com/ethereum/mix/>`_
    基于Qt的IDE，可以设计、调试和测试Solidity智能合约。

* `Ethereum Studio <https://live.ether.camp/>`_		
    专门的网页IDE，也提供一个完整以太坊环境的脚本访问。

Solidity工具列表
--------------

* `Dapp <https://dapp.readthedocs.io>`_
    Solidity语言的构建工具，包管理器以及部署助手。

* `Solidity REPL <https://github.com/raineorshine/solidity-repl>`_
    一个命令行控制台，可以立刻尝试Solidity语言。

* `solgraph <https://github.com/raineorshine/solgraph>`_
    可视化的Solidity控制流，并能标明潜在的安全漏洞。

* `evmdis <https://github.com/Arachnid/evmdis>`_
    EVM反汇编程序，可以执行字节码的静态分析，能提供比EVM操作更高级的抽象。

* `Doxity <https://github.com/DigixGlobal/doxity>`_
    Solidity语言的文档生成器。

第三方Solidity解析器和语法
-----------------------------------------

* `solidity-parser <https://github.com/ConsenSys/solidity-parser>`_
    JavaScript的Solidity解析器

* `Solidity Grammar for ANTLR 4 <https://github.com/federicobond/solidity-antlr4>`_
    ANTLR 4解析器生成器的Solidity语法

语言文档
----------------------

下面的页面中，我们首先会看到一个使用Solidity写的 :ref:`简单智能合约 <simple-smart-contract>`，随后讲解 :ref:`区块链 <blockchain-basics>` 基础，然后是 
:ref:`以太坊虚拟机 <the-ethereum-virtual-machine>` 。

下一节会通过给出有用的 
:ref:`合约样例 <voting>` ，解释Solidity的几个*特性* ，
记住你都可以
`在你的浏览器中 <https://remix.ethereum.org>`_ 尝试这些合约！

最后也是最长的一节会深入讲解Solidity的所有方面。

如果还有问题，你可以尝试搜索或在 `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_ 上提问，或者到我们的
`gitter频道 <https://gitter.im/ethereum/solidity/>`_ 来。
随时欢迎改善Solidity或本文档的想法！

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
