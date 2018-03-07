############
贡献方式
############

对于大家的帮助，我们一如既往地感激。

你可以试着 :ref:`building-from-source` 开始，以熟悉 Solidity 的组件和编译流程。这对精通 Solidity 上智能合约的编写也有帮助。

我们特别需要以下方面的帮助：

* 改善文档
* 回复 `StackExchange <https://ethereum.stackexchange.com>`_ 和 `Solidity Gitter <https://gitter.im/ethereum/solidity>`_ 上的用户提问
* 解决并回复 `Solidity's GitHub issues <https://github.com/ethereum/solidity/issues>`_ 上的问题，特别是被标记为 `up-for-grabs <https://github.com/ethereum/solidity/issues?q=is%3Aopen+is%3Aissue+label%3Aup-for-grabs>`_ 的问题，他们是针对外部贡献者的入门问题。

怎样报告问题
====================

请用 `GitHub issues tracker <https://github.com/ethereum/solidity/issues>`_ 来报告问题。汇报问题时，请提供下列细节：

* 你所使用的 Solidity 版本
* 源码（如果可以的话）
* 你在哪个平台上运行代码
* 如何重现该问题
* 该问题的结果是什么
* 预期行为是什么样的

将造成问题的源码进行缩减至最少，总是很有帮助的，并且有时候甚至能澄清误解。

Pull Request 的工作流
==========================

为了进行贡献，请 fork 一个 ``develop`` 分支并在那里进行修改。除了你 *做了什么* 之外，你还需要在 commit 信息中说明，你 *为什么* 做这些修改（除非只是个微小的改动）。

在进行了 fork 之后，如果你还需要从 ``develop`` 分支 pull 任何变更的话（例如，为了解决潜在的合并冲突），请避免使用 ``git merge`` ，而是 ``git rebase`` 你的分支。

此外，如果你在编写一个新功能，请确保你编写了合适的 Boost 测试案例，并将他们放在了 ``test/`` 下。

但是，如果你在进行一个更大的变更，请先与 `Solidity Development Gitter channel <https://gitter.im/ethereum/solidity-dev>`_ 进行商量（该变更不同于上面提到的那个功能，这个变更侧重于编译器和编程语言开发，而不是编程语言的使用）。

最后，请确保你遵守了这个项目的 `coding standards <https://raw.githubusercontent.com/ethereum/cpp-ethereum/develop/CodingStandards.txt>`_ 。还有，虽然我们采用了持续集成测试，但是在提交 pull request 之前，请测试你的代码并确保它能在本地进行编译。

感谢你的帮助！

运行编译器测试
==========================

Solidity 有不同类型的测试，他们包含在应用 ``soltest`` 中。其中一些需要 ``cpp-ethereum`` 客户端运行在测试模式下，另一些需要安装 ``libz3`` 。

若要禁用 z3 测试，可使用 ``./build/test/soltest -- --no-smt`` ，若要执行不需要 ``cpp-ethereum`` 的测试子集，则用 ``./build/test/soltest -- --no-ipc`` 。

对于其他测试，你都需要安装 `cpp-ethereum <https://github.com/ethereum/cpp-ethereum/releases/download/solidityTester/eth>`_ ，并在测试模式下运行它： ``eth --test -d /tmp/testeth`` 。

之后再执行实际的测试文件： ``./build/test/soltest -- --ipcpath /tmp/testeth/geth.ipc`` 。

可以用过滤器来执行一组测试子集： ``soltest -t TestSuite/TestName -- --ipcpath /tmp/testeth/geth.ipc`` ，其中 ``TestName`` 可以是通配符 ``*`` 。

另外， ``scripts/test.sh`` 里有一个测试脚本可执行所有测试，并自动运行 ``cpp-ethereum`` ，如果它在 scripts 路径中的话（但不会去下载它）。

Travis CI 甚至会执行一些额外的测试（包括 ``solc-js`` 和对第三方 Solidity 框架的测试），这些测试需要去编译 Emscripten 目标代码。

Whiskers 模板系统
==========================

*Whiskers* 是一个类似于 `Mustache <https://mustache.github.io>`_ 的模板系统。编译器在各种各样的地方使用 Whiskers 来增强可读性，从而提高代码的可维护性和可验证性。

它的语法与 Mustache 有很大差别：模板标记 ``{{`` 和 ``}}`` 被替换成了 ``<`` 和 ``>`` ，以便增强语法分析，避免与 :ref:`inline-assembly` 的冲突（符号 ``<`` 和 ``>`` 在内联汇编中是无效的，而 ``{`` 和 ``}`` 则被用来限定块）。另一个局限是，列表只会被解析一层，而不是递归解析。未来可能会改变这一个限制。

下面是一个粗略的说明：

任何出现 ``<name>`` 的地方都会被所提供的变量 ``name`` 的字符串值所替换，既不会进行任何转义也不会迭代替换。可以通过 ``<#name>...</name>`` 来限定一个区域。该区域中的内容将进行多次拼接，每次拼接会使用相应变量集中的值替换区域中的 ``<inner>`` 项，模板系统中提供了多少组变量集，就会进行多少次拼接。顶层变量也可以在这样的区域的内部使用。


译者注：对于区域<#name>...</name>的释义，译者参考自：https://github.com/janl/mustache.js#sections
