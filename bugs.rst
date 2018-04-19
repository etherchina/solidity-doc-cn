.. index:: Bugs

.. _known_bugs:

##################
已知bug列表
##################

在下面，你可以找到一个 JSON 格式的列表，上面列出了 Solidity 编译器上一些已知的安全相关的 bug。
该文件被放置于 `Github 仓库 <https://github.com/ethereum/solidity/blob/develop/docs/bugs.json>`_ 。
该列表可以追溯到0.3.0版本，上面的已知 bug 只存在于未列出的（修复）之前的版本。

这里，还有另外一个 `bugs_by_version.json <https://github.com/ethereum/solidity/blob/develop/docs/bugs_by_version.json>`_ 文件。
该文件可用于查询特定的某个编译器版本会受哪些 bug 影响。

合约的源文件检查工具以及其他与合约交互的工具，需在以下的（可疑）场景中查阅该列表：

 - 如果合约是用每日构建版本的编译器编译，而不是发布版本的编译器，那就有点可疑了。此列表不跟踪未发布或每日构建版本的编译器。
 - 较轻微的怀疑情况是，如果一份合约创建时，是由一个并不是最近版本的编译器编译的。对于由其他合约创建的合约，您必须沿着创建链追溯最初事务，并使用该事务的日期作为创建日期。
 - 高度可疑的情况是，如果一份合约由一个包含已知 bug 的编译器编译，但在合约创建时，新的已经包含了（对该bug的）修复的编译器版本已经发布。

这份包含已知 bug 的 JSON 文件实际上是一个对象数组，每个对象对应一个 bug，并包含以下的 keys :

name
    赋予该 bug 的唯一的名字
summary
    对该 bug 的简要描述
description
    对该 bug 的详细描述
link
    包含更多详尽信息的链接，可选
introduced
    第一个包含该 bug 的编译器的发布版本，可选
fixed
    第一个不再包含该 bug 的编译器的发布版本
publish
    该 bug 被广为人知的日期，可选
severity
    bug 的严重性： very low， low， medium， high。综合考虑了在合约测试中的可发现性、发生的可能性和发生后的潜在损害。
conditions
    触发该 bug 所需满足的条件。当前，这是一个包含了 ``optimizer`` 布尔值的对象，这意味着只有打开优化器选项时，才会触发该 bug。
    如果没有给出任何条件，则假设此 bug 始终存在。

.. literalinclude:: bugs.json
   :language: js
