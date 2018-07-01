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

将造成问题的源码缩减到最少，总是很有帮助的，并且有时候甚至能澄清误解。

Pull Request 的工作流
==========================

为了进行贡献，请 fork 一个 ``develop`` 分支并在那里进行修改。除了你 *做了什么* 之外，你还需要在 commit 信息中说明，你 *为什么* 做这些修改（除非只是个微小的改动）。

在进行了 fork 之后，如果你还需要从 ``develop`` 分支 pull 任何变更的话（例如，为了解决潜在的合并冲突），请避免使用 ``git merge`` ，而是 ``git rebase`` 你的分支。

此外，如果你在编写一个新功能，请确保你编写了合适的 Boost 测试案例，并将他们放在了 ``test/`` 下。

但是，如果你在进行一个更大的变更，请先与 `Solidity Development Gitter channel <https://gitter.im/ethereum/solidity-dev>`_ 进行商量（与上文提到的那个功能不同，这个变更侧重于编译器和编程语言开发，而不是编程语言的使用）。

新的特性和 bug 修复会被添加到 ``Changelog.md`` 文件中：使用的时候请遵循上述方式。

最后，请确保你遵守了这个项目的 `编码风格 <https://raw.githubusercontent.com/ethereum/solidity/develop/CODING_STYLE.md>`_ 。还有，虽然我们采用了持续集成测试，但是在提交 pull request 之前，请测试你的代码并确保它能在本地进行编译。

感谢你的帮助！

运行编译器测试
==========================

Solidity 有不同类型的测试，他们包含在应用 ``soltest`` 中。其中一些需要 ``cpp-ethereum`` 客户端运行在测试模式下，另一些需要安装 ``libz3``。

``soltest`` 会从保存在 ``./test/libsolidity/syntaxTests`` 中的测试合约中获取所期待的结果。为了使 soltest 可以找到这些测试，可以使用 ``--testpath`` 命令行参数来指定测试根目录，例如 ``./build/test/soltest -- --testpath ./test``。

若要禁用 z3 测试，可使用 ``./build/test/soltest -- --no-smt --testpath ./test`` ，若要执行不需要 ``cpp-ethereum`` 的测试子集，则用 ``./build/test/soltest -- --no-ipc --testpath ./test``。

对于其他测试，你都需要安装 `cpp-ethereum <https://github.com/ethereum/cpp-ethereum/releases/download/solidityTester/eth>`_ ，并在测试模式下运行它：``eth --test -d /tmp/testeth``。

之后再执行实际的测试文件：``./build/test/soltest -- --ipcpath /tmp/testeth/geth.ipc --testpath ./test``。

可以用过滤器来执行一组测试子集：``soltest -t TestSuite/TestName -- --ipcpath /tmp/testeth/geth.ipc --testpath ./test``，其中 ``TestName`` 可以是通配符 ``*``。

另外， ``scripts/test.sh`` 里有一个测试脚本可执行所有测试，并自动运行 ``cpp-ethereum``，如果它在 scripts 路径中的话（但不会去下载它）。

Travis CI 甚至会执行一些额外的测试（包括 ``solc-js`` 和对第三方 Solidity 框架的测试），这些测试需要去编译 Emscripten 目标代码。

编写和运行语法测试
--------------------------------

就像前文提到的，语法测试存储在单独的合约里。这些文件必须包含注解，为相关的测试标注预想的结果。测试工具将编译并基于给定的预想结果进行检查。

例如：``./test/libsolidity/syntaxTests/double_stateVariable_declaration.sol``

::

    contract test {
        uint256 variable;
        uint128 variable;
    }
    // ----
    // DeclarationError: Identifier already declared.

一个语法测试必须在合约代码之后包含跟在分隔符 ``----`` 之后的测试代码。上边例子中额外的注释则用来描述预想的编译错误或警告。如果合约不会出现编译错误或警告，这部分可以为空。

在上边的例子里，状态变量 ``variable`` 被声明了两次，这是不允许的。这会导致一个 ``DeclarationError`` 来告知标识符已经被声明过了。

用来进行那些测试的工具叫做 ``isoltest``，可以在 ``./test/tools/`` 下找到。它是一个交互工具，允许你使用你喜欢的文本编辑器编辑失败的合约。让我们把第二个 ``variable`` 的声明去掉来使测试失败：

::

    contract test {
        uint256 variable;
    }
    // ----
    // DeclarationError: Identifier already declared.

再次运行 ``./test/isoltest`` 就会得到一个失败的测试：

::

    syntaxTests/double_stateVariable_declaration.sol: FAIL
        Contract:
            contract test {
                uint256 variable;
            }

        Expected result:
            DeclarationError: Identifier already declared.
        Obtained result:
            Success


这里，在获得了结果之后打印了预想的结果，但也提供了编辑/更新/跳过当前合约或直接退出的办法，``isoltest`` 提供了下列测试失败选项：

- edit：``isoltest`` 会尝试打开先前用 ``isoltest --editor /path/to/editor`` 所指定的编辑器。如果没设定路径，则会产生一个运行时错误。如果指定了编辑器，这将打开编辑器并允许你修改合约代码。
- update：更新测试中的合约。这将会移除包含了不匹配异常的注解，或者增加缺失的预想结果。然后测试会重新开始。
- skip：跳过当前测试的执行。
- quit：退出 ``isoltest``。

在上边的情况自动更新合约会把它变为：

::

    contract test {
        uint256 variable;
    }
    // ----

并重新运行测试。它将会通过：

::

    Re-running test case...
    syntaxTests/double_stateVariable_declaration.sol: OK


.. note::

    请为合约文件取个名字，它应该是可以自我解释正在测试什么的那种名字，例如 ``double_variable_declaration.sol``。不要在一个文件中放多个合约，``isoltest`` 目前无法分别识别它们。


通过 AFL 运行 Fuzzer
==========================

Fuzzing 是一种测试技术，它可以通过运行多少不等的随机输入来找出异常的执行状态（片段故障、异常等等）。现代的 fuzzer 已经可以很聪明地在输入中进行直接的查询。
我们有一个专门的程序叫做 ``solfuzzer``，它可以将源代码作为输入，当发生一个内部编译错误、片段故障或者类似的错误时失败，但当代码包含错误的时候则不会失败。
通过这种方法，fuzzing 工具可以找到那些编译级别的内部错误。

我们主要使用 `AFL <http://lcamtuf.coredump.cx/afl/>`_ 来进行 fuzzing 测试。你需要手工下载和构建 AFL。然后用 AFL 作为编译器来构建 Solidity（或直接构建 ``solfuzzer``）：

::

    cd build
    # if needed
    make clean
    cmake .. -DCMAKE_C_COMPILER=path/to/afl-gcc -DCMAKE_CXX_COMPILER=path/to/afl-g++
    make solfuzzer

然后，你需要一个源文件例子。这将使 fuzzer 可以更容易地找到错误。你可以从语法测试目录下拷贝一些文件或者从文档中提取一些测试文件或其他测试：

::

    mkdir /tmp/test_cases
    cd /tmp/test_cases
    # extract from tests:
    path/to/solidity/scripts/isolate_tests.py path/to/solidity/test/libsolidity/SolidityEndToEndTest.cpp
    # extract from documentation:
    path/to/solidity/scripts/isolate_tests.py path/to/solidity/docs docs

AFL 的文档指出，账册（初始的输入文件）不应该太大。每个文件本身不应该超过 1 kB，并且每个功能最多只能有一个输入文件；所以最好从少量的输入文件开始。
此外还有一个叫做 ``afl-cmin`` 的工具，可以将输入文件整理为可以具有近似行为的二进制代码。

现在运行 fuzzer（``-m`` 参数将使用的内存大小扩展为 60 MB）：

::

    afl-fuzz -m 60 -i /tmp/test_cases -o /tmp/fuzzer_reports -- /path/to/solfuzzer

fuzzer 会将导致失败的源文件创建在 ``/tmp/fuzzer_reports`` 中。通常它会找到产生相似错误的类似的源文件。
你可以使用 ``scripts/uniqueErrors.sh`` 工具来过滤重复的错误。

Whiskers 模板系统
==========================

*Whiskers* 是一个类似于 `Mustache <https://mustache.github.io>`_ 的模板系统。编译器在各种各样的地方使用 Whiskers 来增强可读性，从而提高代码的可维护性和可验证性。

它的语法与 Mustache 有很大差别：模板标记 ``{{`` 和 ``}}`` 被替换成了 ``<`` 和 ``>`` ，以便增强语法分析，避免与 :ref:`inline-assembly` 的冲突（符号 ``<`` 和 ``>`` 在内联汇编中是无效的，而 ``{`` 和 ``}`` 则被用来限定块）。另一个局限是，列表只会被解析一层，而不是递归解析。未来可能会改变这一个限制。

下面是一个粗略的说明：

任何出现 ``<name>`` 的地方都会被所提供的变量 ``name`` 的字符串值所替换，既不会进行任何转义也不会迭代替换。可以通过 ``<#name>...</name>`` 来限定一个区域。该区域中的内容将进行多次拼接，每次拼接会使用相应变量集中的值替换区域中的 ``<inner>`` 项，模板系统中提供了多少组变量集，就会进行多少次拼接。顶层变量也可以在这样的区域的内部使用。


译者注：对于区域<#name>...</name>的释义，译者参考自：https://github.com/janl/mustache.js#sections
