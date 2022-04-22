******************
使用编译器
******************

.. index:: ! commandline compiler, compiler;commandline, ! solc

.. _commandline-compiler:

使用命令行编译器
******************************

.. note:: 
  这一节不适用于 :ref:`solcjs <solcjs>` , 即使它是在命令行模式下使用的也不行。


基础用法
-----------

``solc`` 是 Solidity 源码库的构建目标之一，它是 Solidity 的命令行编译器。你可使用 ``solc --help`` 命令来查看它的所有选项的解释。该编译器可以生成各种输出，范围从简单的二进制文件、汇编文件到用于估计“gas”使用情况的抽象语法树（解析树）。如果你只想编译一个文件，你可以运行 ``solc --bin sourceFile.sol`` 来生成二进制文件。
如果你想通过 ``solc`` 获得一些更高级的输出信息，可以通过 ``solc -o outputDirectory --bin --ast-compact-json --asm sourceFile.sol`` 命令将所有的输出都保存到一个单独的文件夹中。


优化器选项
-----------------

Before you deploy your contract, activate the optimizer while compiling using ``solc --optimize --bin sourceFile.sol``.
By default, the optimizer will optimize the contract for 200 runs. 
If you want to optimize for initial contract deployment and get the smallest output, set it to ``--runs=1``. 
If you expect many transactions and don't care for higher deployment cost and output size, set ``--runs`` to a high number.

This parameter has effects on the following (this might change in the future):

 - the size of the binary search in the function dispatch routine
 - the way constants like large numbers or strings are stored

.. index:: allowed paths, --allow-paths, base path, --base-path, include paths, --include-path

Base Path and Import Remapping
------------------------------------------

命令行编译器会自动从文件系统中读取并导入的文件，但同时，它也支持通过 :ref:`path redirects <import-remapping>` 使用 ``prefix=path`` 选项将路径重定向。比如：

.. code-block:: bash

    solc github.com/ethereum/dapp-bin/=/usr/local/lib/dapp-bin/ file.sol

这实质上是告诉编译器去搜索 ``/usr/local/lib/dapp-bin`` 目录下的所有以 ``github.com/ethereum/dapp-bin/`` 开头的文件。

When accessing the filesystem to search for imports, :ref:`paths that do not start with ./
or ../ <direct-imports>` are treated as relative to the directories specified using ``--base-path`` and ``--include-path`` options (or the current working directory if base path is not specified).
Furthermore, the part of the path added via these options will not appear in the contract metadata.

For security reasons the compiler has :ref:`restrictions on what directories it can access <allowed-paths>`.
Directories of source files specified on the command line and target paths of remappings are automatically allowed to be accessed by the file reader, but everything else is rejected by default.
Additional paths (and their subdirectories) can be allowed via the ``--allow-paths /sample/path,/another/sample/path`` switch.

Everything inside the path specified via ``--base-path`` is always allowed.

The above is only a simplification of how the compiler handles import paths.
For a detailed explanation with examples and discussion of corner cases please refer to the section on :ref:`path resolution <path-resolution>`.

.. index:: ! linker, ! --link, ! --libraries

.. _library-linking:

Library Linking
---------------


If your contracts use :ref:`libraries <libraries>`, you will notice that the bytecode contains substrings of the form ``__$53aea86b7d70b31448b230b20ae141a537$__``. These are placeholders for the actual library addresses.
The placeholder is a 34 character prefix of the hex encoding of the keccak256 hash of the fully qualified library name.
The bytecode file will also contain lines of the form ``// <placeholder> -> <fq library name>`` at the end to help
identify which libraries the placeholders represent. Note that the fully qualified library name
is the path of its source file and the library name separated by ``:``.
You can use ``solc`` as a linker meaning that it will insert the library addresses for you at those points:


Either add ``--libraries "file.sol:Math=0x1234567890123456789012345678901234567890 file.sol:Heap=0xabCD567890123456789012345678901234567890"`` to your command to provide an address for each library (use commas or spaces as separators) or store the string in a file (one library per line) and run ``solc`` using ``--libraries fileName``.

.. note::
    Starting Solidity 0.8.1 accepts ``=`` as separator between library and address, and ``:`` as a separator is deprecated. It will be removed in the future. Currently ``--libraries "file.sol:Math:0x1234567890123456789012345678901234567890 file.sol:Heap:0xabCD567890123456789012345678901234567890"`` will work too.

.. index:: --standard-json, --base-path

If ``solc`` is called with the option ``--standard-json``, it will expect a JSON input (as explained below) on the standard input, and return a JSON output on the standard output. This is the recommended interface for more complex and especially automated uses. The process will always terminate in a "success" state and report any errors via the JSON output.
The option ``--base-path`` is also processed in standard-json mode.

If ``solc`` is called with the option ``--link``, all input files are interpreted to be unlinked binaries (hex-encoded) in the ``__$53aea86b7d70b31448b230b20ae141a537$__``-format given above and are linked in-place (if the input is read from stdin, it is written to stdout). All options except ``--libraries`` are ignored (including ``-o``) in this case.

.. warning::
    Manually linking libraries on the generated bytecode is discouraged because it does not update
    contract metadata. Since metadata contains a list of libraries specified at the time of
    compilation and bytecode contains a metadata hash, you will get different binaries, depending
    on when linking is performed.

    You should ask the compiler to link the libraries at the time a contract is compiled by either
    using the ``--libraries`` option of ``solc`` or the ``libraries`` key if you use the
    standard-JSON interface to the compiler.

.. note::
    The library placeholder used to be the fully qualified name of the library itself
    instead of the hash of it. This format is still supported by ``solc --link`` but
    the compiler will no longer output it. This change was made to reduce
    the likelihood of a collision between libraries, since only the first 36 characters
    of the fully qualified library name could be used.

.. _evm-version:
.. index:: ! EVM version, compile target

Setting the EVM Version to Target
*********************************

When you compile your contract code you can specify the Ethereum virtual machine
version to compile for to avoid particular features or behaviours.

.. warning::

   Compiling for the wrong EVM version can result in wrong, strange and failing
   behaviour. Please ensure, especially if running a private chain, that you
   use matching EVM versions.

On the command line, you can select the EVM version as follows:

.. code-block:: shell

  solc --evm-version <VERSION> contract.sol

In the :ref:`standard JSON interface <compiler-api>`, use the ``"evmVersion"``
key in the ``"settings"`` field:

.. code-block:: javascript

    {
      "sources": {/* ... */},
      "settings": {
        "optimizer": {/* ... */},
        "evmVersion": "<VERSION>"
      }
    }

Target Options
--------------

Below is a list of target EVM versions and the compiler-relevant changes introduced
at each version. Backward compatibility is not guaranteed between each version.

- ``homestead``
   - (oldest version)
- ``tangerineWhistle``
   - Gas cost for access to other accounts increased, relevant for gas estimation and the optimizer.
   - All gas sent by default for external calls, previously a certain amount had to be retained.
- ``spuriousDragon``
   - Gas cost for the ``exp`` opcode increased, relevant for gas estimation and the optimizer.
- ``byzantium``
   - Opcodes ``returndatacopy``, ``returndatasize`` and ``staticcall`` are available in assembly.
   - The ``staticcall`` opcode is used when calling non-library view or pure functions, which prevents the functions from modifying state at the EVM level, i.e., even applies when you use invalid type conversions.
   - It is possible to access dynamic data returned from function calls.
   - ``revert`` opcode introduced, which means that ``revert()`` will not waste gas.
- ``constantinople``
   - Opcodes ``create2`, ``extcodehash``, ``shl``, ``shr`` and ``sar`` are available in assembly.
   - Shifting operators use shifting opcodes and thus need less gas.
- ``petersburg``
   - The compiler behaves the same way as with constantinople.
- ``istanbul``
   - Opcodes ``chainid`` and ``selfbalance`` are available in assembly.
- ``berlin``
   - Gas costs for ``SLOAD``, ``*CALL``, ``BALANCE``, ``EXT*`` and ``SELFDESTRUCT`` increased. The
     compiler assumes cold gas costs for such operations. This is relevant for gas estimation and
     the optimizer.
- ``london`` (**default**)
   - The block's base fee (`EIP-3198 <https://eips.ethereum.org/EIPS/eip-3198>`_ and `EIP-1559 <https://eips.ethereum.org/EIPS/eip-1559>`_) can be accessed via the global ``block.basefee`` or ``basefee()`` in inline assembly.


.. index:: ! standard JSON, ! --standard-json

.. _compiler-api:

编译器输入输出JSON描述
******************************************


下面展示的这些JSON格式是编译器API使用的，当然，在 ``solc`` 上也是可用的。有些字段是可选的（参见注释），并且它们可能会发生变化，但所有的变化都应该是后向兼容的。

编译器API需要JSON格式的输入，并以JSON格式输出编译结果。

注释是不允许的，这里仅用于解释目的。

输入说明
-----------------

.. code-block:: javascript

    {
      // 必选: 源代码语言，比如“Solidity”，“serpent”，“lll”，“assembly”等
      language: "Solidity",
      // 必选
      sources:
      {
        // 这里的键值是源文件的“全局”名称，可以通过remappings引入其他文件（参考下文）
        "myFile.sol":
        {
          // 可选: 源文件的kaccak256哈希值，可用于校验通过URL加载的内容。
          "keccak256": "0x123...",
          // 必选（除非声明了 "content" 字段）: 指向源文件的URL。
          // URL(s) 会按顺序加载，并且结果会通过keccak256哈希值进行检查（如果有keccak256的话）
          // 如果哈希值不匹配，或者没有URL返回成功，则抛出一个异常。
          "urls":
          [
            "bzzr://56ab...",
            "ipfs://Qma...",
            "file:///tmp/path/to/file.sol"
          ]
        },
        "mortal":
        {
          // 可选: 该文件的keccak256哈希值
          "keccak256": "0x234...",
          // 必选（除非声明了 "urls" 字段）: 源文件的字面内容
          "content": "contract mortal is owned { function kill() { if (msg.sender == owner) selfdestruct(owner); } }"
        }
      },
      // 可选
      settings:
      {
        // 可选: Stop compilation after the given stage. Currently only "parsing" is valid here
        "stopAfter": "parsing",
        // 可选: 重定向参数的排序列表
        remappings: [ ":g/dir" ],
        // 可选: 优化器配置
        optimizer: {
          // 默认为 disabled
          // NOTE: enabled=false still leaves some optimizations on. See comments below.
          // WARNING: Before version 0.8.6 omitting the 'enabled' key was not equivalent to setting
          // it to false and would actually disable all the optimizations.
          enabled: true,
          // 基于你希望运行多少次代码来进行优化。
          // 较小的值可以使初始部署的费用得到更多优化，较大的值可以使高频率的使用得到优化。
          runs: 200,
          // Switch optimizer components on or off in detail.
          // The "enabled" switch above provides two defaults which can be
          // tweaked here. If "details" is given, "enabled" can be omitted.
          "details": {
            // The peephole optimizer is always on if no details are given,
            // use details to switch it off.
            "peephole": true,
            // The inliner is always on if no details are given,
            // use details to switch it off.
            "inliner": true,
            // The unused jumpdest remover is always on if no details are given,
            // use details to switch it off.
            "jumpdestRemover": true,
            // Sometimes re-orders literals in commutative operations.
            "orderLiterals": false,
            // Removes duplicate code blocks
            "deduplicate": false,
            // Common subexpression elimination, this is the most complicated step but
            // can also provide the largest gain.
            "cse": false,
            // Optimize representation of literal numbers and strings in code.
            "constantOptimizer": false,
            // The new Yul optimizer. Mostly operates on the code of ABI coder v2
            // and inline assembly.
            // It is activated together with the global optimizer setting
            // and can be deactivated here.
            // Before Solidity 0.6.0 it had to be activated through this switch.
            "yul": false,
            // Tuning options for the Yul optimizer.
            "yulDetails": {
              // Improve allocation of stack slots for variables, can free up stack slots early.
              // Activated by default if the Yul optimizer is activated.
              "stackAllocation": true,
              // Select optimization steps to be applied.
              // Optional, the optimizer will use the default sequence if omitted.
              "optimizerSteps": "dhfoDgvulfnTUtnIf..."
            }
        },
        // 指定需编译的EVM的版本。会影响代码的生成和类型检查。可用的版本为：homestead，tangerineWhistle，spuriousDragon，byzantium，constantinople
        evmVersion: "byzantium",
        // Optional: Change compilation pipeline to go through the Yul intermediate representation.
        // This is false by default.
        "viaIR": true,
        // Optional: Debugging settings
        "debug": {
          // How to treat revert (and require) reason strings. Settings are
          // "default", "strip", "debug" and "verboseDebug".
          // "default" does not inject compiler-generated revert strings and keeps user-supplied ones.
          // "strip" removes all revert strings (if possible, i.e. if literals are used) keeping side-effects
          // "debug" injects strings for compiler-generated internal reverts, implemented for ABI encoders V1 and V2 for now.
          // "verboseDebug" even appends further information to user-supplied revert strings (not yet implemented)
          "revertStrings": "default",
          // Optional: How much extra debug information to include in comments in the produced EVM
          // assembly and Yul code. Available components are:
          // - `location`: Annotations of the form `@src <index>:<start>:<end>` indicating the
          //    location of the corresponding element in the original Solidity file, where:
          //     - `<index>` is the file index matching the `@use-src` annotation,
          //     - `<start>` is the index of the first byte at that location,
          //     - `<end>` is the index of the first byte after that location.
          // - `snippet`: A single-line code snippet from the location indicated by `@src`.
          //     The snippet is quoted and follows the corresponding `@src` annotation.
          // - `*`: Wildcard value that can be used to request everything.
          "debugInfo": ["location", "snippet"]
        },
        // 可选: 元数据配置
        metadata: {
          // 只可使用字面内容，不可用URLs （默认设为 false）
          useLiteralContent: true,
          // Use the given hash method for the metadata hash that is appended to the bytecode.
          // The metadata hash can be removed from the bytecode via option "none".
          // The other options are "ipfs" and "bzzr1".
          // If the option is omitted, "ipfs" is used by default.
          "bytecodeHash": "ipfs"
        },
        // 库的地址。如果这里没有把所有需要的库都给出，会导致生成输出数据不同的未链接对象
        libraries: {
          // 最外层的 key 是使用这些库的源文件的名字。
          // 如果使用了重定向， 在重定向之后，这些源文件应该能匹配全局路径
          // 如果源文件的名字为空，则所有的库为全局引用
          "myFile.sol": {
            "MyLib": "0x123123..."
          }
        },
        // 以下内容可以用于选择所需的输出。
        // 如果这个字段被忽略，那么编译器会加载并进行类型检查，但除了错误之外不会产生任何输出。
        // 第一级的key是文件名，第二级是合约名称，如果合约名为空，则针对文件本身（进行输出）。
        // 若使用通配符*，则表示所有合约。
        //
        // 可用的输出类型如下所示：
        //   abi - ABI
        //   ast - 所有源文件的AST
        //   devdoc - 开发者文档（natspec）
        //   userdoc - 用户文档（natspec）
        //   metadata - 元数据
        //   ir - 去除语法糖（desugaring）之前的新汇编格式
        //   irOptimized - Intermediate representation after optimization
        //   storageLayout - Slots, offsets and types of the contract's state variables.
        //   evm.assembly - 去除语法糖（desugaring）之后的新汇编格式
        //   evm.legacyAssembly - JSON的旧样式汇编格式
        //   evm.bytecode.functionDebugData - Debugging information at function level
        //   evm.bytecode.object - 字节码对象
        //   evm.bytecode.opcodes - 操作码列表
        //   evm.bytecode.sourceMap - 源码映射（用于调试）
        //   evm.bytecode.linkReferences - 链接引用（如果是未链接的对象）
        //   evm.bytecode.generatedSources - Sources generated by the compiler
        //   evm.deployedBytecode* - 部署的字节码（具有evm.bytecode所有的选项）
        //   evm.deployedBytecode.immutableReferences - Map from AST ids to bytecode ranges that reference immutables
        //   evm.methodIdentifiers - 函数哈希值列表
        //   evm.gasEstimates - 函数的gas预估量
        //   ewasm.wast - Ewasm in WebAssembly S-expressions 格式
        //   ewasm.wasm - Ewasm in WebAssembly 二进制格式
        //
        // 请注意，如果使用 `evm` ，`evm.bytecode` ，`ewasm` 等选项，会选择其所有的子项作为输出。 另外，`*`可以用作通配符来请求所有内容。
        //
        outputSelection: {
          // 为每个合约生成元数据和字节码输出。
          "*": {
            "*": [ "metadata"，"evm.bytecode" ]
          },
          // 启用“def”文件中定义的“MyContract”合约的abi和opcodes输出。
          "def": {
            "MyContract": [ "abi"，"evm.bytecode.opcodes" ]
          },
      },
      // The modelChecker object is experimental and subject to changes.
        "modelChecker":
        {
          // Chose which contracts should be analyzed as the deployed one.
          "contracts":
          {
            "source1.sol": ["contract1"],
            "source2.sol": ["contract2", "contract3"]
          },
          // Choose how division and modulo operations should be encoded.
          // When using `false` they are replaced by multiplication with slack
          // variables. This is the default.
          // Using `true` here is recommended if you are using the CHC engine
          // and not using Spacer as the Horn solver (using Eldarica, for example).
          // See the Formal Verification section for a more detailed explanation of this option.
          "divModNoSlacks": false,
          // Choose which model checker engine to use: all (default), bmc, chc, none.
          "engine": "chc",
          // Choose which types of invariants should be reported to the user: contract, reentrancy.
          "invariants": ["contract", "reentrancy"],
          // Choose whether to output all unproved targets. The default is `false`.
          "showUnproved": true,
          // Choose which solvers should be used, if available.
          // See the Formal Verification section for the solvers description.
          "solvers": ["cvc4", "smtlib2", "z3"],
          // Choose which targets should be checked: constantCondition,
          // underflow, overflow, divByZero, balance, assert, popEmptyArray, outOfBounds.
          // If the option is not given all targets are checked by default,
          // except underflow/overflow for Solidity >=0.8.7.
          // See the Formal Verification section for the targets description.
          "targets": ["underflow", "overflow", "assert"],
          // Timeout for each SMT query in milliseconds.
          // If this option is not given, the SMTChecker will use a deterministic
          // resource limit by default.
          // A given timeout of 0 means no resource/time restrictions for any query.
          "timeout": 20000
        }
      }
    }


输出说明
------------------

.. code-block:: javascript

    {
      // 可选：如果没有遇到错误/警告/提示，则不出现
      errors: [
        {
          // 可选：源文件中的位置
          sourceLocation: {
            file: "sourceFile.sol",
            start: 0,
            end: 100
          },
        // Optional: Further locations (e.g. places of conflicting declarations)
          secondarySourceLocations: [
            {
              "file": "sourceFile.sol",
              "start": 64,
              "end": 92,
              "message": "Other declaration is here:"
            }
          ],
          // 强制: 错误类型，例如 “TypeError”， “InternalCompilerError”， “Exception”等.
          // 可在文末查看完整的错误类型列表
          type: "TypeError",
          // 强制: 发生错误的组件，例如“general”，“ewasm”等
          component: "general",
          // 强制：错误的严重级别（“error” , “warning” 或 "info"）, 将来也许会扩展
          severity: "error",
          // 可选: 引起错误的唯一编码
          "errorCode": "3141",
          // 强制
          message: "Invalid keyword",
          // 可选: 带错误源位置的格式化消息
          formattedMessage: "sourceFile.sol:100: Invalid keyword"
        }
      ],
      // 这里包含了文件级别的输出。可以通过outputSelection来设置限制/过滤。
      sources: {
        "sourceFile.sol": {
          // 标识符（用于源码映射）
          id: 1,
          // AST对象
          ast: {},
        }
      },
      // 这里包含了合约级别的输出。 可以通过outputSelection来设置限制/过滤。
      contracts: {
        "sourceFile.sol": {
          // 如果使用的语言没有合约名称，则该字段应该留空。
          "ContractName": {
            // 以太坊合约的应用二进制接口（ABI）。如果为空，则表示为空数组。
            // 请参阅 https://docs.soliditylang.org/en/develop/abi-spec.html
            abi: [],
            // 请参阅元数据输出文档（序列化的JSON字符串）
            metadata: "{/* ... */}",
            // 用户文档（natspec）
            userdoc: {},
            // 开发人员文档（natspec）
            devdoc: {},
            // 中间表示形式 (string)
            ir: "",
            // See the Storage Layout documentation.
            "storageLayout": {"storage": [/* ... */], "types": {/* ... */} },
            // EVM相关输出
            evm: {
              // 汇编 (string)
              assembly: "",
              // 旧风格的汇编 (object)
              legacyAssembly: {},
              // 字节码和相关细节
              bytecode: {
                // Debugging data at the level of functions.
                "functionDebugData": {
                  // Now follows a set of functions including compiler-internal and
                  // user-defined function. The set does not have to be complete.
                  "@mint_13": { // Internal name of the function
                    "entryPoint": 128, // Byte offset into the bytecode where the function starts (optional)
                    "id": 13, // AST ID of the function definition or null for compiler-internal functions (optional)
                    "parameterSlots": 2, // Number of EVM stack slots for the function parameters (optional)
                    "returnSlots": 1 // Number of EVM stack slots for the return values (optional)
                  }
                },
                // 十六进制字符串的字节码
                object: "00fe",
                // 操作码列表 (string)
                opcodes: "",
                // 源码映射的字符串。 请参阅源码映射的定义
                sourceMap: "",
                // Array of sources generated by the compiler. Currently only
                // contains a single Yul file.
                "generatedSources": [{
                  // Yul AST
                  "ast": { /* ...*/ },
                  // Source file in its text form (may contain comments)
                  "contents":"{ function abi_decode(start, end) -> data { data := calldataload(start) } }",
                  // Source file ID, used for source references, same "namespace" as the Solidity source files
                  "id": 2,
                  "language": "Yul",
                  "name": "#utility.yul"
                }],
                // 如果这里给出了信息，则表示这是一个未链接的对象
                linkReferences: {
                  "libraryFile.sol": {
                    // 字节码中的字节偏移；链接时，从指定的位置替换20个字节
                    "Library1": [
                      { start: 0，length: 20 },
                      { start: 200，length: 20 }
                    ]
                  }
                }
              },
             
              deployedBytecode: {
                /* ..., */  // 与上面相同的布局
                "immutableReferences": {
                  // There are two references to the immutable with AST ID 3, both 32 bytes long. One is
                  // at bytecode offset 42, the other at bytecode offset 80.
                  "3": [{ "start": 42, "length": 32 }, { "start": 80, "length": 32 }]
                }
              },
              // 函数哈希的列表
              methodIdentifiers: {
                "delegate(address)": "5c19a95c"
              },
              // 函数的gas预估量
              gasEstimates: {
                creation: {
                  codeDepositCost: "420000",
                  executionCost: "infinite",
                  totalCost: "infinite"
                },
                external: {
                  "delegate(address)": "25000"
                },
                internal: {
                  "heavyLifting()": "infinite"
                }
              }
            },
            // Ewasm相关的输出
            ewasm: {
              // S-expressions格式
              wast: "",
              // 二进制格式（十六进制字符串）
              wasm: ""
            }
          }
        }
      }
    }


错误类型
~~~~~~~~~~~

1. ``JSONError``: JSON输入不符合所需格式，例如，输入不是JSON对象，不支持的语言等。
2. ``IOError``: IO和导入处理错误，例如，在提供的源里包含无法解析的URL或哈希值不匹配。
3. ``ParserError``: 源代码不符合语言规则。
4. ``DocstringParsingError``: 注释块中的NatSpec标签无法解析。
5. ``SyntaxError``: 语法错误，例如 ``continue`` 在 ``for`` 循环外部使用。
6. ``DeclarationError``: 无效的，无法解析的或冲突的标识符名称 比如 ``Identifier not found``。
7. ``TypeError``: 类型系统内的错误，例如无效类型转换，无效赋值等。
8. ``UnimplementedFeatureError``: 编译器当前不支持该功能，但预计将在未来的版本中支持。
9. ``InternalCompilerError``: 在编译器中触发的内部错误——应将此报告为一个issue。
10. ``Exception``: 编译期间的未知失败——应将此报告为一个issue。
11. ``CompilerError``: 编译器堆栈的无效使用——应将此报告为一个issue。
12. ``FatalError``: 未正确处理致命错误——应将此报告为一个issue。
13. ``Warning``: 警告，不会停止编译，但应尽可能处理。
14. ``Info``: 编译器认为用户可能会发现有用的信息，但并不危险，不一定需要处理。


.. _compiler-tools:

Compiler tools
**************

solidity-upgrade
----------------

``solidity-upgrade`` can help you to semi-automatically upgrade your contracts
to breaking language changes. While it does not and cannot implement all
required changes for every breaking release, it still supports the ones, that
would need plenty of repetitive manual adjustments otherwise.

.. note::

    ``solidity-upgrade`` carries out a large part of the work, but your
    contracts will most likely need further manual adjustments. We recommend
    using a version control system for your files. This helps reviewing and
    eventually rolling back the changes made.

.. warning::

    ``solidity-upgrade`` is not considered to be complete or free from bugs, so
    please use with care.

How it Works
~~~~~~~~~~~~

You can pass (a) Solidity source file(s) to ``solidity-upgrade [files]``. If these make use of ``import`` statement which refer to files outside the
current source file's directory, you need to specify directories that are allowed to read and import files from, by passing ``--allow-paths [directory]``. You can ignore missing files by passing ``--ignore-missing``.

``solidity-upgrade`` is based on ``libsolidity`` and can parse, compile and
analyse your source files, and might find applicable source upgrades in them.

Source upgrades are considered to be small textual changes to your source code.
They are applied to an in-memory representation of the source files
given. The corresponding source file is updated by default, but you can pass
``--dry-run`` to simulate to whole upgrade process without writing to any file.

The upgrade process itself has two phases. In the first phase source files are
parsed, and since it is not possible to upgrade source code on that level,
errors are collected and can be logged by passing ``--verbose``. No source
upgrades available at this point.

In the second phase, all sources are compiled and all activated upgrade analysis
modules are run alongside compilation. By default, all available modules are
activated. Please read the documentation on
:ref:`available modules <upgrade-modules>` for further details.


This can result in compilation errors that may
be fixed by source upgrades. If no errors occur, no source upgrades are being
reported and you're done.
If errors occur and some upgrade module reported a source upgrade, the first
reported one gets applied and compilation is triggered again for all given
source files. The previous step is repeated as long as source upgrades are
reported. If errors still occur, you can log them by passing ``--verbose``.
If no errors occur, your contracts are up to date and can be compiled with
the latest version of the compiler.

.. _upgrade-modules:

Available upgrade modules
~~~~~~~~~~~~~~~~~~~~~~~~~~
+----------------------------+---------+--------------------------------------------------+
| Module                     | Version | Description                                      |
+============================+=========+==================================================+
| ``constructor``            | 0.5.0   | Constructors must now be defined using the       |
|                            |         | ``constructor`` keyword.                         |
+----------------------------+---------+--------------------------------------------------+
| ``visibility``             | 0.5.0   | Explicit function visibility is now mandatory,   |
|                            |         | defaults to ``public``.                          |
+----------------------------+---------+--------------------------------------------------+
| ``abstract``               | 0.6.0   | The keyword ``abstract`` has to be used if a     |
|                            |         | contract does not implement all its functions.   |
+----------------------------+---------+--------------------------------------------------+
| ``virtual``                | 0.6.0   | Functions without implementation outside an      |
|                            |         | interface have to be marked ``virtual``.         |
+----------------------------+---------+--------------------------------------------------+
| ``override``               | 0.6.0   | When overriding a function or modifier, the new  |
|                            |         | keyword ``override`` must be used.               |
+----------------------------+---------+--------------------------------------------------+
| ``dotsyntax``              | 0.7.0   | The following syntax is deprecated:              |
|                            |         | ``f.gas(...)()``, ``f.value(...)()`` and         |
|                            |         | ``(new C).value(...)()``. Replace these calls by |
|                            |         | ``f{gas: ..., value: ...}()`` and                |
|                            |         | ``(new C){value: ...}()``.                       |
+----------------------------+---------+--------------------------------------------------+
| ``now``                    | 0.7.0   | The ``now`` keyword is deprecated. Use           |
|                            |         | ``block.timestamp`` instead.                     |
+----------------------------+---------+--------------------------------------------------+
| ``constructor-visibility`` | 0.7.0   | Removes visibility of constructors.              |
|                            |         |                                                  |
+----------------------------+---------+--------------------------------------------------+

Please read :doc:`0.5.0 release notes <050-breaking-changes>`,
:doc:`0.6.0 release notes <060-breaking-changes>` and
:doc:`0.7.0 release notes <070-breaking-changes>`  and :doc:`0.8.0 release notes <080-breaking-changes>` for further details.

Synopsis
~~~~~~~~

.. code-block:: none

    Usage: solidity-upgrade [options] contract.sol

    Allowed options:
        --help               Show help message and exit.
        --version            Show version and exit.
        --allow-paths path(s)
                             Allow a given path for imports. A list of paths can be
                             supplied by separating them with a comma.
        --ignore-missing     Ignore missing files.
        --modules module(s)  Only activate a specific upgrade module. A list of
                             modules can be supplied by separating them with a comma.
        --dry-run            Apply changes in-memory only and don't write to input
                             file.
        --verbose            Print logs, errors and changes. Shortens output of
                             upgrade patches.
        --unsafe             Accept *unsafe* changes.



Bug Reports / Feature Requests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you found a bug or if you have a feature request, please
`file an issue <https://github.com/ethereum/solidity/issues/new/choose>`_ on Github.


Example
~~~~~~~

Assume that you have the following contract in ``Source.sol``:

.. code-block:: Solidity

    pragma solidity >=0.6.0 <0.6.4;
    // This will not compile after 0.7.0
    // SPDX-License-Identifier: GPL-3.0
    contract C {
        // FIXME: remove constructor visibility and make the contract abstract
        constructor() internal {}
    }

    contract D {
        uint time;

        function f() public payable {
            // FIXME: change now to block.timestamp
            time = now;
        }
    }

    contract E {
        D d;

        // FIXME: remove constructor visibility
        constructor() public {}

        function g() public {
            // FIXME: change .value(5) =>  {value: 5}
            d.f.value(5)();
        }
    }



Required Changes
^^^^^^^^^^^^^^^^

The above contract will not compile starting from 0.7.0. To bring the contract up to date with the
current Solidity version, the following upgrade modules have to be executed:
``constructor-visibility``, ``now`` and ``dotsyntax``. Please read the documentation on
:ref:`available modules <upgrade-modules>` for further details.


Running the upgrade
^^^^^^^^^^^^^^^^^^^

It is recommended to explicitly specify the upgrade modules by using ``--modules`` argument.

.. code-block:: bash

    solidity-upgrade --modules constructor-visibility,now,dotsyntax Source.sol

The command above applies all changes as shown below. Please review them carefully (the pragmas will
have to be updated manually.)

.. code-block:: Solidity

    pragma solidity ^0.7.0;
    // SPDX-License-Identifier: GPL-3.0
    abstract contract C {
        // FIXME: remove constructor visibility and make the contract abstract
        constructor() {}
    }

    contract D {
        uint time;

        function f() public payable {
            // FIXME: change now to block.timestamp
            time = block.timestamp;
        }
    }

    contract E {
        D d;

        // FIXME: remove constructor visibility
        constructor() {}

        function g() public {
            // FIXME: change .value(5) =>  {value: 5}
            d.f{value: 5}();
        }
    }
