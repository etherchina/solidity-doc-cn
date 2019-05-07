资源
---------

常用资源
~~~~~~~~~~

* `以太坊官网 <https://ethereum.org>`_

* `变更日志 <https://github.com/ethereum/solidity/blob/develop/Changelog.md>`_

* `Solidity源码 <https://github.com/ethereum/solidity/>`_

* `Ethereum Stackexchange <https://ethereum.stackexchange.com/>`_

* `Language Users Chat <https://gitter.im/ethereum/solidity/>`_

* `Compiler Developers Chat <https://gitter.im/ethereum/solidity-dev/>`_

Solidity IDE及编辑器
~~~~~~~~~~~~~~~~~~~~~

* 常用:

    * `EthFiddle <https://ethfiddle.com/>`_
        基于浏览器的 IDE，可以编写和分享Solidity 代码，需要服务端。

    * `Remix <https://remix.ethereum.org/>`_
        基于浏览器的 IDE，集成了编译器和 Solidity 运行时环境，不需要服务端组件。

    * `Solhint <https://github.com/protofire/solhint>`_
        一种静态检查器，提供安全和风格指南以及智能合约验证的最佳实践规则。

    * `Solidity IDE <https://github.com/System-Glitch/Solidity-IDE>`_
        基于浏览器的 IDE，集成了编译器，支持 Ganache 和本地文件系统。

    * `Solium <https://github.com/duaraghav8/Solium/>`_
        一种静态检查器，识别和修复 Solidity 中的风格以及安全问题。

    * `Superblocks Lab <https://lab.superblocks.com/>`_
        基于浏览器的 IDE， 集成了基于浏览器的虚拟机以及 Metamask ，可以一键部署到主网和测试网。

* Atom:

    * `Etheratom <https://github.com/0mkara/etheratom>`_
       Atom 编辑器的插件，支持高亮、编译和运行时环境（兼容后端节点和虚拟机）。

    * `Atom Solidity Linter <https://atom.io/packages/linter-solidity>`_
        Atom 编辑器的插件，提供 Solidity 语言的 Lint 检查（静态检查）。

    * `Atom Solium Linter <https://atom.io/packages/linter-solium>`_
        Atom 的可配置的 Solidty 静态检查器，基于 Solium。

* Eclipse:

   * `YAKINDU Solidity Tools <https://yakindu.github.io/solidity-ide/>`_
        基于Eclipse的 IDE. 具有根据上下文代码补全和帮助，代码导航，语法高亮，内置编译器，快速修复和模板功能。

* Emacs:

    * `Emacs Solidity <https://github.com/ethereum/emacs-solidity/>`_
        Emacs 编辑器的插件，提供语法高亮和编译错误报告。

* IntelliJ:

    * `IntelliJ IDEA plugin <https://plugins.jetbrains.com/plugin/9475-intellij-solidity>`_
        IntelliJ IDEA 的 Solidity 插件（可用于其他所有的 JetBrains IDE）

* Sublime:

    * `Package for SublimeText - Solidity language syntax <https://packagecontrol.io/packages/Ethereum/>`_
        SublimeText 编辑器的语法高亮包。

* Vim:

    * `Vim Solidity <https://github.com/tomlion/vim-solidity/>`_
        Vim 编辑器的插件，提供语法高亮。

    * `Vim Syntastic <https://github.com/scrooloose/syntastic>`_
        Vim 编辑器的插件，提供编译检查。

* Visual Studio Code:

    * `Visual Studio Code extension <http://juan.blanco.ws/solidity-contracts-in-visual-studio-code/>`_
        Microsoft Visual Studio Code 插件，包含语法高亮和 Solidity 编译器。

不再使用的:

* `Mix IDE <https://github.com/ethereum/mix/>`_
    基于 Qt 的 IDE，可以设计、调试和测试 Solidity 智能合约。

* `Ethereum Studio <https://live.ether.camp/>`_
    专门的网页 IDE，也提供一个完整以太坊环境的脚本访问。

* `Visual Studio Extension <https://visualstudiogallery.msdn.microsoft.com/96221853-33c4-4531-bdd5-d2ea5acc4799/>`_
     Microsoft Visual Studio Code 插件，包含 Solidity 编译器。

Solidity 工具
~~~~~~~~~~~~~~~~~~

* `Dapp <https://dapp.tools/dapp/>`_
    Solidity 语言的构建工具、包管理器以及部署助手。

* `Solidity REPL <https://github.com/raineorshine/solidity-repl>`_
    一个命令行控制台，可以让你立刻尝试 Solidity 语言。

* `solgraph <https://github.com/raineorshine/solgraph>`_
    可视化的 Solidity 控制流，并能高亮标明潜在的安全漏洞。

* `Doxity <https://github.com/DigixGlobal/doxity>`_
    Solidity 语言的文档生成器。

* `evmdis <https://github.com/Arachnid/evmdis>`_
    EVM 反汇编程序，可以执行字节码的静态分析，能提供比 EVM 操作更高级的抽象。

* `ABI to solidity interface converter <https://gist.github.com/chriseth/8f533d133fa0c15b0d6eaf3ec502c82b>`_
    从智能合约的ABI生成合约接口的脚本。

* `Securify <https://securify.ch/>`_
    智能合约的全自动在线静态分析器，提供基于漏洞模式的安全报告。

* `Sūrya <https://github.com/ConsenSys/surya/>`_
    一个智能合约系统实用工具，提供大量可视化输出和有关合约结构的信息。 还支持查询函数调用图。

* `EVM Lab <https://github.com/ethereum/evmlab/>`_
    一个与EVM交互工具包， 包括VM，Etherchain API 以及 Gas 消耗 的跟踪查看器。

* `Universal Mutator <https://github.com/agroce/universalmutator>`_
    A tool for mutation generation ，可配置的规则，支持Solidity和Vyper 。

.. note::
  变量名称，注释和源代码格式等信息在编译过程中丢失，无法完全恢复原始源代码。 无法反编译智能合约以查看原始源代码。

第三方 Solidity 解析器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* `solidity-parser <https://github.com/ConsenSys/solidity-parser>`_
    Solidity parser for JavaScript

* `Solidity Grammar for ANTLR 4 <https://github.com/federicobond/solidity-antlr4>`_
    Solidity grammar for the ANTLR 4 parser generator
