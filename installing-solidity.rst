.. index:: ! installing

.. _installing-solidity:

################################
安装Solidity编译器
################################

版本
==========

Solidity的版本遵循 `语义化版本原则 <https://semver.org>`_ ，除了正式发布的版本外，Solidity还有一个**每日开发构建**（nightly development builds）可用。这个每晚构建的版本不保证能够正常工作，尽管尽了最大的努力，但仍可能包含无文档的或重大的改动。我们建议使用最新版本。 下面的包安装程序将使用最新版本。

Remix
=====

  推荐使用 Remix 来开发简单合约和快速学习 Solidity。

`Remix <https://remix.ethereum.org/>`_ 可在线使用，而无需安装任何东西。如果你想离线使用，可按 https://github.com/ethereum/browser-solidity/tree/gh-pages 的页面说明下载 zip 文件来使用。
该页面有进一步详细说明如何安装 Solidity 命令行编译器到你计算机上。如果你刚好要处理大型合约，或者需要更多的编译选项，那么你应该选择使用命令行编译器 solc。

.. _solcjs:

npm / Node.js
=============

使用 `npm` 可以便捷地安装Solidity编译器solcjs。但该 `solcjs` 程序的功能相对于本页下面的所有其他选项都要少。在 :ref:`commandline-compiler` 一章中，我们假定你使用的是完整功能的编译器。 所以，如果你是从 npm 安装 solcjs ，就此打住，直接跳到 `solc-js  <https://github.com/ethereum/solc-js>`_ 去了解。


注意: `solc-js <https://github.com/ethereum/solc-js>`_ 是利用 Emscripten 从 C++ 版的 solc 跨平台编译为 JavaScript 的，因此，可在 JavaScript 项目中使用 solcjs（如同 Remix）。
具体介绍请查看 `solc-js <https://github.com/ethereum/solc-js>`_ 代码库。

.. code:: bash

    npm install -g solc

.. note::

    在命令行中，使用 `solcjs` 而非 `solc` 。
    `solcjs` 的命令行选项同 `solc` 和一些工具（如 `geth` )是不兼容的，因此不要期望 `solcjs` 能像 `solc` 一样工作。

Docker
======

我们为solc编译器提供了最新的docker版本。``stable``仓库里的是已发布的版本，``nightly``
仓库则是在开发分支中的带有不稳定变更的版本。

.. code:: bash

    docker run ethereum/solc:stable solc --version

目前，docker 镜像只含有 solc 的可执行程序，因此你需要额外的工作去链接源和输出目录。


二进制包
===============

可在`solidity/releases <https://github.com/ethereum/solidity/releases>`_下载 Solidity 的二进制安装包。

对于Ubuntu，我们也提供PPAs。通过以下命令，可获取最新的稳定版本：

.. code:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install solc

当然，你也可安装尝鲜的开发者版本：

.. code:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo add-apt-repository ppa:ethereum/ethereum-dev
    sudo apt-get update
    sudo apt-get install solc

同时，也提供可安装`所有支持的Linux版本 <https://snapcraft.io/docs/core/install>`_下的`snap package <https://snapcraft.io/>`_。通过以下命令，可获取最新的稳定版本：

.. code:: bash

    sudo snap install solc

或者，如果你想测试 develop 分支下的最新变更，可通过如下方式安装开发者版本：

.. code:: bash

    sudo snap install solc --edge

同样，Arch Linux 也有提供安装包，但仅限于最新的开发者版本：

.. code:: bash

    pacman -S solidity

在写本文时，Homebrew 上还没有提供预构建的二进制包（因为我们从 Jenkins 迁移到了 TravisCI ）。 我们将尽快提供 homebrew 下的二进制安装包，但至少从源码构建的方式还是行得通的：

.. code:: bash

    brew update
    brew upgrade
    brew tap ethereum/ethereum
    brew install solidity
    brew linkapps solidity


如果你需要特定版本的 Solidity ，你需要从 Github 上安装一个 Homebrew formula。
你可查阅
`solidity.rb commits on Github <https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb>`_
的提交记录，去寻找包含``solidity.rb``文件改动的特殊提交。然后使用``brew``进行安装：


.. code:: bash

    brew unlink solidity
    # Install 0.4.8
    brew install https://raw.githubusercontent.com/ethereum/homebrew-ethereum/77cce03da9f289e5a3ffe579840d3c5dc0a62717/solidity.rb

Gentoo Linux 下也提供了安装包，可使用``emerge``进行安装：

.. code:: bash

    emerge dev-lang/solidity

.. _building-from-source:

从源代码编译
====================

克隆代码库
--------------------

执行以下命令，克隆源代码：

.. code:: bash

    git clone --recursive https://github.com/ethereum/solidity.git
    cd solidity

如果你想参与 Solidity 的开发, 你可 Fork Solidity 后，用你个人的 Fork 库作为第二远程源：

.. code:: bash

    cd solidity
    git remote add personal git@github.com:[username]/solidity.git

Solidity 有 Git 子模块，需确保完全加载它们：

.. code:: bash

   git submodule update --init --recursive

先决条件 - macOS
---------------------

在 macOS 中，需确保有安装最新版的
`Xcode <https://developer.apple.com/xcode/download/>`_，
Xcode 包含 `Clang C++ 编译器 <https://en.wikipedia.org/wiki/Clang>`_， 而
`Xcode IDE <https://en.wikipedia.org/wiki/Xcode>`_ 和其他苹果开发工具是 OS X 下编译 C++ 应用所必须的。
如果你是第一次安装 Xcode 或者刚好更新了 Xcode 新版本，则在使用命令行构建前，需同意 Xcode 的使用协议：

.. code:: bash

    sudo xcodebuild -license accept

Solidity 在 OS X 下构建，必须 `安装 Homebrew <http://brew.sh>`_
包管理器来安装依赖。
如果你想从头开始，这里是 `卸载 Homebrew 的方法
<https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/FAQ.md#how-do-i-uninstall-homebrew>`_。


先决条件 - Windows
-----------------------

在Windows下构建Solidity，需下载的依赖软件包：

+------------------------------+-------------------------------------------------------+
| 软件                           备注                                            |
+==============================+=======================================================+
| `Git for Windows`_           | 从Github上获取源码的命令行工具  |
+------------------------------+-------------------------------------------------------+
| `CMake`_                     | 跨平台构建文件生成器                 |
+------------------------------+-------------------------------------------------------+
| `Visual Studio 2015`_        | C++ 编译开发环境                    |
+------------------------------+-------------------------------------------------------+

.. _Git for Windows: https://git-scm.com/download/win
.. _CMake: https://cmake.org/download/
.. _Visual Studio 2015: https://www.visualstudio.com/products/vs-2015-product-editions


外部依赖
---------------------

在 macOS、Windows和其他 Linux 发行版上，有一个脚本可以“一键”安装所需的外部依赖库。本来是需要人工参与的多步操作，现在可一键执行:

.. code:: bash

    ./scripts/install_deps.sh

Windows 下执行：

.. code:: bat

    scripts\install_deps.bat


命令行构建
------------------

**确保你已安装外部依赖（见上面）**

Solidity 使用 CMake 来配置构建。Linux、macOS 和其他 Unix系统上的构建方式都差不多：

.. code:: bash

    mkdir build
    cd build
    cmake .. && make

也有更简单的：

.. code:: bash

    #note: 将安装 solc 和 soltest 到 usr/local/bin 目录
    ./scripts/build.sh

对于 Windows 执行：

.. code:: bash

    mkdir build
    cd build
    cmake -G "Visual Studio 14 2015 Win64" ..

命令的最后一行会在 build 目录下创建一个 **solidity.sln** 文件，双击后，默认会使用 Visual Studio 打开。我们建议在VS上创建 RelWithDebugInfo 配置文件。

或者用命令创建：

.. code:: bash

    cmake --build . --config RelWithDebInfo

CMake参数
=============

如果你对 CMake 命令选项有兴趣，可执行 ``cmake .. -LH`` 进行查看。

版本号字符串详解
============================

Solidity 版本名包含四部分：

- 版本号
- 预发布版本号，通常为 ``develop.YYYY.MM.DD`` 或者 ``nightly.YYYY.MM.DD``
- 以 ``commit.GITHASH`` 格式展示的提交号
- 含平台和编译器的详细信息的多条目内容

如果本地有修改，则 commit 部分有后缀 ``.mod``。

此四部分按照 Semver 要求组成，第2部分等同 Semver 预发布版本号，第三和四部分组成 Semver 版本编译信息。

发行版样例：``0.4.8+commit.60cc1668.Emscripten.clang``.

预发布版样例： ``0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang``

版本信息详情
======================================

在版本发布之后，补丁版本号会增加，因为我们假定只有修补程序级别发生变化。当变更被合并后，版本应该根据semver和更改的严重程度进行调整。最后，发行版本总是与当前夜间版本的版本的版本号一致，但没有``预发布版``说明符。

例如：

0. 0.4.0 版本发布
1. 从现在开始，每晚构建为 0.4.1 版本
2. 引入非破坏性变更 - 不改变版本号
3. 引入破坏性变更 - 版本跳跃到 0.5.0
4. 0.5.0 版本发布

该方式与 :ref:`version pragma <version_pragma>` 一起运行良好。
