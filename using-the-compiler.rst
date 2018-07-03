******************
使用编译器
******************

.. index:: ! commandline compiler, compiler;commandline, ! solc, ! linker

.. _commandline-compiler:

使用命令行编译器
******************************

.. note:: 这一节并不适用于 :ref:`solcjs <solcjs>`


``solc`` 是 Solidity 源码库的构建目标之一，它是 Solidity 的命令行编译器。你可使用 ``solc --help`` 命令来查看它的所有选项的解释。该编译器可以生成各种输出，范围从简单的二进制文件、汇编文件到用于估计“gas”使用情况的抽象语法树（解析树）。如果你只想编译一个文件，你可以运行 ``solc --bin sourceFile.sol`` 来生成二进制文件。如果你想通过 ``solc`` 获得一些更高级的输出信息，可以通过 ``solc -o outputDirectory --bin --ast --asm sourceFile.sol`` 命令将所有的输出都保存到一个单独的文件夹中。

Before you deploy your contract, activate the optimizer while compiling using ``solc --optimize --bin sourceFile.sol``. By default, the optimizer will optimize the contract for 200 runs. If you want to optimize for initial contract deployment and get the smallest output, set it to ``--runs=1``. If you expect many transactions and don't care for higher deployment cost and output size, set ``--runs`` to a high number.

命令行编译器会自动从文件系统中读取并导入的文件，但同时，它也支持通过 ``prefix=path`` 选项将路径重定向。比如：

::

    solc github.com/ethereum/dapp-bin/=/usr/local/lib/dapp-bin/ =/usr/local/lib/fallback file.sol

这实质上是告诉编译器去搜索 ``/usr/local/lib/dapp-bin`` 目录下的所有以 ``github.com/ethereum/dapp-bin/`` 开头的文件，如果编译器找不到这样的文件，它会接着读取 ``/usr/local/lib/fallback`` 目录下的所有文件（空前缀意味着始终匹配）。``solc`` 不会从位于重定向目标之外和显式指定的源文件所在目录之外的文件系统读取文件，所以，类似 ``import "/etc/passwd";`` 这样的语句，编译器只会在你添加了 ``=/`` 选项之后，才会尝试到根目录下加载 ``/etc/passwd`` 文件。

如果重定向路径下存在多个匹配，则选择具有最长公共前缀的那个匹配。

出于安全原因，编译器限制了它可以访问的目录。在命令行中指定的源文件的路径（及其子目录）和通过重定向定义的路径可用于 ``import`` 语句，其他的则会被拒绝。额外路径（及其子目录）可以通过  ``--allow-paths /sample/path,/another/sample/path`` 进行配置。

如果您的合约使用 :ref:`libraries <libraries>` ，您会注意到在编译后的十六进制字节码中会包含形如 ``__LibraryName____`` 的字符串。当您将 ``solc`` 作为链接器使用时，它会在下列情况中为你插入库的地址：要么在命令行中添加 ``--libraries "Math:0x12345678901234567890 Heap:0xabcdef0123456"`` 来为每个库提供地址，或者将这些字符串保存到一个文件中（每行一个库），并使用 ``--libraries fileName`` 参数。

如果在调用 ``solc`` 命令时使用了 ``--link`` 选项，则所有的输入文件会被解析为上面提到过的  ``__LibraryName____`` 格式的未链接的二进制数据（十六进制编码），并且就地链接。（如果输入是从stdin读取的，则生成的数据会被写入stdout）。在这种情况下，除了 ``--libraries`` 外的其他选项（包括 ``-o`` ）都会被忽略。

如果在调用 ``solc`` 命令时使用了 ``--standard-json`` 选项，它将会按JSON格式解析标准输入上的输入，并在标准输出上返回JSON格式的输出。

.. _compiler-api:

编译器输入输出JSON描述
******************************************


下面展示的这些JSON格式是编译器API使用的，当然，在 ``solc`` 上也是可用的。有些字段是可选的（参见注释），并且它们可能会发生变化，但所有的变化都应该是后向兼容的。

编译器API需要JSON格式的输入，并以JSON格式输出编译结果。

注释是不允许的，这里仅用于解释目的。

输入说明
-----------------

.. code-block:: none

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
        // 可选: 重定向参数的排序列表
        remappings: [ ":g/dir" ],
        // 可选: 优化器配置
        optimizer: {
          // 默认为 disabled
          enabled: true,
          // 基于你希望运行多少次代码来进行优化。
          // 较小的值可以使初始部署的费用得到更多优化，较大的值可以使高频率的使用得到优化。
          runs: 200
        },
        // 指定需编译的EVM的版本。会影响代码的生成和类型检查。可用的版本为：homestead，tangerineWhistle，spuriousDragon，byzantium，constantinople
        evmVersion: "byzantium",
        // 可选: 元数据配置
        metadata: {
          // 只可使用字面内容，不可用URLs （默认设为 false）
          useLiteralContent: true
        },
        // 库的地址。如果这里没有把所有需要的库都给出，会导致生成输出数据不同的未链接对象
        libraries: {
          // 最外层的 key 是使用这些库的源文件的名字。
          // 如果使用了重定向， 在重定向之后，这些源文件应该能匹配全局路径
          // 如果源文件的名字为空，则所有的库为全局引用
          "myFile.sol": {
            "MyLib": "0x123123..."
          }
        }
        // 以下内容可以用于选择所需的输出。
        // 如果这个字段被忽略，那么编译器会加载并进行类型检查，但除了错误之外不会产生任何输出。
        // 第一级的key是文件名，第二级是合约名称，如果合约名为空，则针对文件本身（进行输出）。
        // 若使用通配符*，则表示所有合约。
        //
        // 可用的输出类型如下所示：
        //   abi - ABI
        //   ast - 所有源文件的AST
        //   legacyAST - 所有源文件的legacy AST
        //   devdoc - 开发者文档（natspec）
        //   userdoc - 用户文档（natspec）
        //   metadata - 元数据
        //   ir - 去除语法糖（desugaring）之前的新汇编格式
        //   evm.assembly - 去除语法糖（desugaring）之后的新汇编格式
        //   evm.legacyAssembly - JSON的旧样式汇编格式
        //   evm.bytecode.object - 字节码对象
        //   evm.bytecode.opcodes - 操作码列表
        //   evm.bytecode.sourceMap - 源码映射（用于调试）
        //   evm.bytecode.linkReferences - 链接引用（如果是未链接的对象）
        //   evm.deployedBytecode* - 部署的字节码（与evm.bytecode具有相同的选项）
        //   evm.methodIdentifiers - 函数哈希值列表
        //   evm.gasEstimates - 函数的gas预估量
        //   ewasm.wast - eWASM S-expressions 格式（不支持atm）
        //   ewasm.wasm - eWASM二进制格式（不支持atm）
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
          // 为每个合约生成源码映射输出
          "*": {
            "*": [ "evm.bytecode.sourceMap" ]
          },
          // 每个文件生成legacy AST输出
          "*": {
            "": [ "legacyAST" ]
          }
        }
      }
    }


输出说明
------------------

.. code-block:: none

    {
      // 可选：如果没有遇到错误/警告，则不出现
      errors: [
        {
          // 可选：源文件中的位置
          sourceLocation: {
            file: "sourceFile.sol",
            start: 0,
            end: 100
          ],
          // 强制: 错误类型，例如 “TypeError”， “InternalCompilerError”， “Exception”等.
          // 可在文末查看完整的错误类型列表
          type: "TypeError",
          // 强制: 发生错误的组件，例如“general”，“ewasm”等
          component: "general",
          // 强制：错误的严重级别（“error”或“warning”）
          severity: "error",
          // 强制
          message: "Invalid keyword"
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
          // legacy AST 对象
          legacyAST: {}
        }
      },
      // 这里包含了合约级别的输出。 可以通过outputSelection来设置限制/过滤。
      contracts: {
        "sourceFile.sol": {
          // 如果使用的语言没有合约名称，则该字段应该留空。
          "ContractName": {
            // 以太坊合约的应用二进制接口（ABI）。如果为空，则表示为空数组。
            // 请参阅 https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI
            abi: [],
            // 请参阅元数据输出文档（序列化的JSON字符串）
            metadata: "{...}",
            // 用户文档（natspec）
            userdoc: {},
            // 开发人员文档（natspec）
            devdoc: {},
            // 中间表示形式 (string)
            ir: "",
            // EVM相关输出
            evm: {
              // 汇编 (string)
              assembly: "",
              // 旧风格的汇编 (object)
              legacyAssembly: {},
              // 字节码和相关细节
              bytecode: {
                // 十六进制字符串的字节码
                object: "00fe",
                // 操作码列表 (string)
                opcodes: "",
                // 源码映射的字符串。 请参阅源码映射的定义
                sourceMap: "",
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
              // 与上面相同的布局
              deployedBytecode: { },
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
            // eWASM相关的输出
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