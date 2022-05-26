.. index:: ! installing

.. _installing-solidity:

################################
安装Solidity编译器
################################

版本
==========

Solidity的版本遵循 `语义化版本原则 <https://semver.org>`_， 此外，带有主版本0（即0.x.y）的补丁级版本将不会包含破坏性的变化。这意味着用0.x.y版本编译的代码可在0.x.z版本中编译，其中z>y。

作为发布版本的补充， **每日开发构建** （nightly development builds）也是可用的。这个每日开发构建不保证能正常工作，尽管尽了最大的努力，但仍可能包含未记录的和／或重大的改动。我们推荐使用最新的发布版本。下面的包安装程序将使用最新发布(release)版本。
这是因为我们会定期引入突破性的更新，带来新的功能和修复错误。我们目前使用0.x版本号 `来表示这种快速变化的速度 <https://semver.org/#spec-item-4>`_ 。


Remix
======

*我们推荐使用 Remix 来开发简单合约和快速学习 Solidity。*

`Remix <https://remix.ethereum.org/>`_ 可在线使用，而无需安装任何东西。如果你想离线使用，可按 https://github.com/ethereum/remix-live/tree/gh-pages 的页面说明下载 .zip 文件来使用。
该页面有进一步详细说明如何安装 Solidity 命令行编译器到你计算机上。如果你刚好要处理大型合约，或者需要更多的编译选项，那么你应该选择使用命令行编译器 solc。

译者注：Remix 还有一个 `Remix-ide <https://learnblockchain.cn/2018/06/07/remix-ide/>`_ 本地版本。

.. _solcjs:

npm / Node.js
=============

使用 ``npm`` 可以便捷地安装Solidity编译器solcjs。但该 ``solcjs`` 程序的功能相对于本页下面的所有其他选项都要少。在 :ref:`commandline-compiler` 一章中，我们假定你使用的是完整功能的编译器。 所以，如果你是从 `npm` 安装 `solcjs` ，就此打住，直接跳到 `solc-js  <https://github.com/ethereum/solc-js>`_ 去了解。


注意: solc-js 项目是利用 Emscripten 从 C++ 版的 solc 跨平台编译为 JavaScript 的，因此，可在 JavaScript 项目中使用 solcjs（如同 Remix）。
具体介绍请参考 solc-js 代码库。

.. code:: bash

    npm install -g solc

.. note::

    在命令行中，可执行文件命名为 `solcjs` 。

    `solcjs` 的命令行选项与 `solc` 和一些工具（如 `geth` )是不兼容的，因此不要期望 `solcjs`  `solc` 完全一样。

Docker
=======

Docker images of Solidity builds are available using the ``solc`` image from the ``ethereum`` organisation.
Use the ``stable`` tag for the latest released version, and ``nightly`` for potentially unstable changes in the develop branch.

The Docker image runs the compiler executable, so you can pass all compiler arguments to it.
For example, the command below pulls the stable version of the ``solc`` image (if you do not have it already),
and runs it in a new container, passing the ``--help`` argument.

.. code-block:: bash

    docker run ethereum/solc:stable --help

You can also specify release build versions in the tag, for example, for the 0.5.4 release.

.. code-block:: bash

    docker run ethereum/solc:0.5.4 --help

To use the Docker image to compile Solidity files on the host machine mount a
local folder for input and output, and specify the contract to compile. For example.

.. code-block:: bash

    docker run -v /local/path:/sources ethereum/solc:stable -o /sources/output --abi --bin /sources/Contract.sol

You can also use the standard JSON interface (which is recommended when using the compiler with tooling).
When using this interface it is not necessary to mount any directories as long as the JSON input is
self-contained (i.e. it does not refer to any external files that would have to be
:ref:`loaded by the import callback <initial-vfs-content-standard-json-with-import-callback>`).

.. code-block:: bash

    docker run ethereum/solc:stable --standard-json < input.json > output.json

Linux 包
===============

可在 `solidity/releases <https://github.com/ethereum/solidity/releases>`_ 下载 Solidity 的二进制安装包。

对于 Ubuntu ，我们也提供 PPAs 。通过以下命令，可获取最新的稳定版本：

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

Furthermore, some Linux distributions provide their own packages. These packages are not directly
maintained by us, but usually kept up-to-date by the respective package maintainers.

For example, Arch Linux has packages for the latest development version:

.. code-block:: bash

    pacman -S solidity

也提供在 `所有支持的Linux版本 <https://snapcraft.io/docs/core/install>`_ 下可安装的 `snap package <https://snapcraft.io/solc>`_ （不过当前未维护）。通过以下命令，可获取最新的稳定版本：

.. code:: bash

    sudo snap install solc

或者，如果你想测试 develop 分支下的最新变更，可通过如下方式安装开发者版本：

.. code:: bash

    sudo snap install solc --edge

.. note::

    The ``solc`` snap uses strict confinement. This is the most secure mode for snap packages
    but it comes with limitations, like accessing only the files in your ``/home`` and ``/media`` directories.
    For more information, go to `Demystifying Snap Confinement <https://snapcraft.io/blog/demystifying-snap-confinement>`_.


macOS Packages
==============

我们通过 Homebrew 从源码构建Solidity 编译器，预构建的bottles暂时还不支持。

.. code:: bash

    brew update
    brew upgrade
    brew tap ethereum/ethereum
    brew install solidity


如果你需要特定版本的 Solidity ，你需要从 Github 上安装一个 Homebrew formula。
你可查阅 `solidity.rb 在 Github的提交记录 <https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb>`_
的提交记录，去寻找包含 ``solidity.rb`` 文件改动的特殊提交。然后使用 ``brew`` 进行安装：

复制提交记录的你想要安装版本的提交记录 Hash ，下载（checkout）到本地。


.. code-block:: bash

    git clone https://github.com/ethereum/homebrew-ethereum.git
    cd homebrew-ethereum
    git checkout <提交记录 hash>

使用 ``brew`` 安装:

.. code:: bash

    brew unlink solidity
    # 例如安装 0.4.8
    brew install solidity.rb


冻结的二进制版本
================

We maintain a repository containing static builds of past and current compiler versions for all
supported platforms at `solc-bin`_. This is also the location where you can find the nightly builds.

The repository is not only a quick and easy way for end users to get binaries ready to be used
out-of-the-box but it is also meant to be friendly to third-party tools:

- The content is mirrored to https://binaries.soliditylang.org where it can be easily downloaded over
  HTTPS without any authentication, rate limiting or the need to use git.
- Content is served with correct `Content-Type` headers and lenient CORS configuration so that it
  can be directly loaded by tools running in the browser.
- Binaries do not require installation or unpacking (with the exception of older Windows builds
  bundled with necessary DLLs).
- We strive for a high level of backwards-compatibility. Files, once added, are not removed or moved
  without providing a symlink/redirect at the old location. They are also never modified
  in place and should always match the original checksum. The only exception would be broken or
  unusable files with a potential to cause more harm than good if left as is.
- Files are served over both HTTP and HTTPS. As long as you obtain the file list in a secure way
  (via git, HTTPS, IPFS or just have it cached locally) and verify hashes of the binaries
  after downloading them, you do not have to use HTTPS for the binaries themselves.

The same binaries are in most cases available on the `Solidity release page on Github`_. The
difference is that we do not generally update old releases on the Github release page. This means
that we do not rename them if the naming convention changes and we do not add builds for platforms
that were not supported at the time of release. This only happens in ``solc-bin``.

The ``solc-bin`` repository contains several top-level directories, each representing a single platform.
Each one contains a ``list.json`` file listing the available binaries. For example in
``emscripten-wasm32/list.json`` you will find the following information about version 0.7.4:

.. code-block:: json

    {
      "path": "solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js",
      "version": "0.7.4",
      "build": "commit.3f05b770",
      "longVersion": "0.7.4+commit.3f05b770",
      "keccak256": "0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3",
      "sha256": "0x2b55ed5fec4d9625b6c7b3ab1abd2b7fb7dd2a9c68543bf0323db2c7e2d55af2",
      "urls": [
        "bzzr://16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1",
        "dweb:/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS"
      ]
    }

This means that:

- You can find the binary in the same directory under the name
  `solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js <https://github.com/ethereum/solc-bin/blob/gh-pages/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js>`_.
  Note that the file might be a symlink, and you will need to resolve it yourself if you are not using
  git to download it or your file system does not support symlinks.
- The binary is also mirrored at https://binaries.soliditylang.org/emscripten-wasm32/solc-emscripten-wasm32-v0.7.4+commit.3f05b770.js.
  In this case git is not necessary and symlinks are resolved transparently, either by serving a copy
  of the file or returning a HTTP redirect.
- The file is also available on IPFS at `QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS`_.
- The file might in future be available on Swarm at `16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1`_.
- You can verify the integrity of the binary by comparing its keccak256 hash to
  ``0x300330ecd127756b824aa13e843cb1f43c473cb22eaf3750d5fb9c99279af8c3``.  The hash can be computed
  on the command line using ``keccak256sum`` utility provided by `sha3sum`_ or `keccak256() function
  from ethereumjs-util`_ in JavaScript.
- You can also verify the integrity of the binary by comparing its sha256 hash to
  ``0x2b55ed5fec4d9625b6c7b3ab1abd2b7fb7dd2a9c68543bf0323db2c7e2d55af2``.
  
.. warning::

   Due to the strong backwards compatibility requirement the repository contains some legacy elements
   but you should avoid using them when writing new tools:

   - Use ``emscripten-wasm32/`` (with a fallback to ``emscripten-asmjs/``) instead of ``bin/`` if
     you want the best performance. Until version 0.6.1 we only provided asm.js binaries.
     Starting with 0.6.2 we switched to `WebAssembly builds`_ with much better performance. We have
     rebuilt the older versions for wasm but the original asm.js files remain in ``bin/``.
     The new ones had to be placed in a separate directory to avoid name clashes.
   - Use ``emscripten-asmjs/`` and ``emscripten-wasm32/`` instead of ``bin/`` and ``wasm/`` directories
     if you want to be sure whether you are downloading a wasm or an asm.js binary.
   - Use ``list.json`` instead of ``list.js`` and ``list.txt``. The JSON list format contains all
     the information from the old ones and more.
   - Use https://binaries.soliditylang.org instead of https://solc-bin.ethereum.org. To keep things
     simple we moved almost everything related to the compiler under the new ``soliditylang.org``
     domain and this applies to ``solc-bin`` too. While the new domain is recommended, the old one
     is still fully supported and guaranteed to point at the same location.

.. warning::

    The binaries are also available at https://ethereum.github.io/solc-bin/ but this page
    stopped being updated just after the release of version 0.7.2, will not receive any new releases
    or nightly builds for any platform and does not serve the new directory structure, including
    non-emscripten builds.

    If you are using it, please switch to https://binaries.soliditylang.org, which is a drop-in
    replacement. This allows us to make changes to the underlying hosting in a transparent way and
    minimize disruption. Unlike the ``ethereum.github.io`` domain, which we do not have any control
    over, ``binaries.soliditylang.org`` is guaranteed to work and maintain the same URL structure
    in the long-term.

.. _IPFS: https://ipfs.io
.. _Swarm: https://swarm-gateways.net/bzz:/swarm.eth
.. _solc-bin: https://github.com/ethereum/solc-bin/
.. _Solidity release page on github: https://github.com/ethereum/solidity/releases
.. _sha3sum: https://github.com/maandree/sha3sum
.. _keccak256() function from ethereumjs-util: https://github.com/ethereumjs/ethereumjs-util/blob/master/docs/modules/_hash_.md#const-keccak256
.. _WebAssembly builds: https://emscripten.org/docs/compiling/WebAssembly.html
.. _QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS: https://gateway.ipfs.io/ipfs/QmTLs5MuLEWXQkths41HiACoXDiH8zxyqBHGFDRSzVE5CS
.. _16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1: https://swarm-gateways.net/bzz:/16c5f09109c793db99fe35f037c6092b061bd39260ee7a677c8a97f18c955ab1/


.. _building-from-source:

从源代码编译
====================

预先安装环境 - 所有平台
-------------------------------------

以下是所有编译Solidity的依赖关系:

+-----------------------------------+-------------------------------------------------------+
| 软件                              | 备注                                                  |
+===================================+=======================================================+
| `CMake`_ (version 3.13+)          | Cross-platform build file generator.                  |
+-----------------------------------+-------------------------------------------------------+
| `Boost`_ (version 1.77+ on        | C++ libraries.                                        |
| Windows, 1.65+ otherwise)         |                                                       |
+-----------------------------------+-------------------------------------------------------+
| `Git`_                            | 获取源代码的命令行工具                                |
+-----------------------------------+-------------------------------------------------------+
| `z3`_ (version 4.8+, Optional)    | For use with SMT checker.                             |
+-----------------------------------+-------------------------------------------------------+
| `cvc4`_ (Optional)                | For use with SMT checker.                             |
+-----------------------------------+-------------------------------------------------------+

.. _cvc4: https://cvc4.cs.stanford.edu/web/
.. _Git: https://git-scm.com/download
.. _Boost: https://www.boost.org
.. _CMake: https://cmake.org/download/
.. _z3: https://github.com/Z3Prover/z3

.. note::
    Solidity versions prior to 0.5.10 can fail to correctly link against Boost versions 1.70+.
    A possible workaround is to temporarily rename ``<Boost install path>/lib/cmake/Boost-1.70.0``
    prior to running the cmake command to configure solidity.

    Starting from 0.5.10 linking against Boost 1.70+ should work without manual intervention.


.. note::
    The default build configuration requires a specific Z3 version (the latest one at the time the
    code was last updated). Changes introduced between Z3 releases often result in slightly different
    (but still valid) results being returned. Our SMT tests do not account for these differences and
    will likely fail with a different version than the one they were written for. This does not mean
    that a build using a different version is faulty. If you pass ``-DSTRICT_Z3_VERSION=OFF`` option
    to CMake, you can build with any version that satisfies the requirement given in the table above.
    If you do this, however, please remember to pass the ``--no-smt`` option to ``scripts/tests.sh``
    to skip the SMT tests.

最低的编译器版本
^^^^^^^^^^^^^^^^^^^^^^^^^

The following C++ compilers and their minimum versions can build the Solidity codebase:

- `GCC <https://gcc.gnu.org>`_, version 8+
- `Clang <https://clang.llvm.org/>`_, version 7+
- `MSVC <https://visualstudio.microsoft.com/vs/>`_, version 2019+



环境依赖条件 - macOS
---------------------

在 macOS 中，需确保有安装最新版的
`Xcode <https://developer.apple.com/xcode/download/>`_，
Xcode 包含 `Clang C++ 编译器 <https://en.wikipedia.org/wiki/Clang>`_， 而
`Xcode IDE <https://en.wikipedia.org/wiki/Xcode>`_ 和其他苹果 OS X 下编译 C++ 应用所必须的开发工具。
如果你是第一次安装 Xcode 或者刚好更新了 Xcode 新版本，则在使用命令行构建前，需同意 Xcode 的使用协议：

.. code:: bash

    sudo xcodebuild -license accept

Solidity 在 OS X 下构建，必须 `安装 Homebrew <https://brew.sh>`_
包管理器来安装依赖。
如果你想从头开始，这里是 `卸载 Homebrew 的方法
<https://docs.brew.sh/FAQ#how-do-i-uninstall-homebrew>`_。


环境依赖条件 - Windows
-----------------------

在Windows下构建Solidity，需下载的依赖软件包：

+-----------------------------------+-------------------------------------------------------+
| 软件                              | 备注                                                  |
+===================================+=======================================================+
| `Visual Studio 2019 Build Tools`_ | C++ 编译器                                            |
+-----------------------------------+-------------------------------------------------------+
| `Visual Studio 2019`_  (Optional) | C++ 编译器和开发环境                                  |
+-----------------------------------+-------------------------------------------------------+
| `Boost`_ (version 1.77+)          | C++ libraries.                                        |
+-----------------------------------+-------------------------------------------------------+

如果你已经有了 IDE，仅需要编译器和相关的库，你可以安装 Visual Studio 2019 Build Tools。

Visual Studio 2019 提供了 IDE 以及必要的编译器和库。所以如果你还没有一个 IDE 并且想要开发 Solidity，那么 Visual Studio 2019 将是一个可以使你获得所有工具的简单选择。

这里是一个在 Visual Studio 2019 Build Tools 或 Visual Studio 2019 中应该安装的组件列表：

* Visual Studio C++ core features
* VC++ 2019 v141 toolset (x86,x64)
* Windows Universal CRT SDK
* Windows 8.1 SDK
* C++/CLI support

.. _Git for Windows: https://git-scm.com/download/win
.. _CMake: https://cmake.org/download/
.. _Visual Studio 2019: https://www.visualstudio.com/vs/
.. _Visual Studio 2019 Build Tools: https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2019


有一个脚本可以“一键”安装所需的外部依赖库:

.. code:: bat

    scripts\install_deps.ps1

命令将在 ``deps`` 子目录中安装  ``boost`` 和 ``cmake``


克隆代码库
--------------------

执行以下命令，克隆源代码：

.. code:: bash

    git clone --recursive https://github.com/ethereum/solidity.git
    cd solidity

如果你想参与 Solidity 的开发, 你可分叉 Solidity 源码库后，用你个人的分叉库作为第二远程源：

.. code:: bash

    git remote add personal git@github.com:[username]/solidity.git


.. note::
    This method will result in a prerelease build leading to e.g. a flag
    being set in each bytecode produced by such a compiler.
    If you want to re-build a released Solidity compiler, then
    please use the source tarball on the github release page:

    https://github.com/ethereum/solidity/releases/download/v0.X.Y/solidity_0.X.Y.tar.gz

    (not the "Source code" provided by github).


Solidity 有 Git 子模块，需确保完全加载它们：

.. code:: bash

    git submodule update --init --recursive


命令行构建
------------------

**确保你已安装外部依赖（见上面）**

Solidity 使用 CMake 来配置构建。你也许想要安装 `ccache`_ 来加速重复构建，CMake自动进行这个工作。
Linux、macOS 和其他 Unix系统上的构建方式都差不多：

.. _ccache: https://ccache.dev/

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
    cmake -G "Visual Studio 16 2019" ..


如果你想执行 ``scripts\install_deps.ps1`` 时使用你安装过的boost版本，可以添加参数 ``-DBoost_DIR="deps\boost\lib\cmake\Boost-*"`` 和 ``-DCMAKE_MSVC_RUNTIME_LIBRARY=MultiThreaded`` 去调用  ``cmake``.

这组指令的最后一句，会在 build 目录下创建一个 **solidity.sln** 文件，双击后，默认会使用 Visual Studio 打开。我们建议在VS上创建 **RelWithDebugInfo** 配置文件。

或者用命令创建：

.. code:: bash

    cmake --build . --config RelWithDebInfo

CMake参数
=============

如果你对 CMake 命令选项有兴趣，可执行 ``cmake .. -LH`` 进行查看。

.. _smt_solvers_build:

SMT Solvers
-----------
Solidity can be built against SMT solvers and will do so by default if
they are found in the system. Each solver can be disabled by a `cmake` option.

*Note: In some cases, this can also be a potential workaround for build failures.*


Inside the build folder you can disable them, since they are enabled by default:

.. code-block:: bash

    # disables only Z3 SMT Solver.
    cmake .. -DUSE_Z3=OFF

    # disables only CVC4 SMT Solver.
    cmake .. -DUSE_CVC4=OFF

    # disables both Z3 and CVC4
    cmake .. -DUSE_CVC4=OFF -DUSE_Z3=OFF


版本号字符串详解
============================

Solidity 版本名包含四部分：

- 版本号
- 预发布版本号，通常为 ``develop.YYYY.MM.DD`` 或者 ``nightly.YYYY.MM.DD``
- 以 ``commit.GITHASH`` 格式展示的提交号
- 由若干条平台、编译器详细信息构成的平台标识

如果本地有修改，则 commit 部分有后缀 ``.mod``。

这些部分按照 SemVer 的要求来组合， Solidity 预发布版本号等价于 SemVer 预发布版本号， Solidity 提交号和平台标识则组成 SemVer 的构建元数据。

发行版样例：``0.4.8+commit.60cc1668.Emscripten.clang`` .

预发布版样例： ``0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang``

版本信息详情
=====================================

在release版本发布之后，补丁版本号会增加，因为我们假定只有补丁级别的变更会在之后发生。当变更被合并后，版本应该根据 SemVer 和变更的剧烈程度进行调整。最后，发行版本总是与当前每日构建版本的版本号一致，但没有 ``prerelease`` 指示符。

例如：

1. 0.4.0 release 版本发布。
2. 从 0.4.1 版本开始，构建 nightly 版本。
3. 没有引入破坏性变更 —— 不改变版本号。
4. 引入破坏性变更 —— 版本跳跃到 0.5.0
5. 0.5.0 release版本发布

该方式与 :ref:`version pragma <version_pragma>` 一起运行良好。
