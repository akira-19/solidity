******************
Using the compiler
******************

.. index:: ! commandline compiler, compiler;commandline, ! solc, ! linker

.. _commandline-compiler:

Using the Commandline Compiler
******************************

.. note::
    この章の内容は、コマンドラインモードも含めて  :ref:`solcjs <solcjs>` には使用することができません。

Solidityレポジトリのビルドターゲットの1つはsolidityコマンドラインコンパイラである ``solc`` です。
``solc --help`` を使ってオプションを表示できます。このコンパイラは、単純なバイナリから抽象構文木(構文解析ツリー)へのアセンブリからガス代の見積もりまで、さまざまな出力を生成することができます。
単一ファイルのみをコンパイルしたい場合、該当のファイルを ``solc --bin sourceFile.sol`` として実行すると、バイナリが出力されます。
``solc`` のより高度なアウトプットを取得したい場合は、 ``solc -o outputDirectory --bin --ast --asm sourceFile`` を使用してすべてを別々のファイルに出力するように指定したほうがよいでしょう。

また、コントラクトをデプロイする前に、 ``solc --optimize --bin sourceFile.sol`` を使用してコンパイルするときにオプティマイザを有効にしてください。
デフォルトでは、オプティマイザは、コントラクトがそのライフタイムにわたって200回呼び出されると仮定して、コントラクトを最適化します。
イニシャルコントラクトのデプロイのガス代をより安く、その後の関数実行時のガス代をより高くしたい場合は、``--optimize-runs=1`` と設定してください。
多くのトランザクションを想定していて、デフォルトより高いデプロイコストとアウトプットサイズを気にしないのであれば、 ``--optimize-runs`` を大きな数に設定してください。

コマンドラインコンパイラはファイルシステムからインポートされたファイルを自動的に読み込みますが、以下のように ``prefix=path`` を使ってリダイレクトするパスを明示することも可能です:

::

    solc github.com/ethereum/dapp-bin/=/usr/local/lib/dapp-bin/ file.sol

これはコンパイラに ``/usr/local/lib/dapp-bin`` 下の ``github.com/ethereum/dapp-bin/`` で始まるものを検索するように指示します。
``solc`` は再マッピングターゲットの外側にあるファイルシステムからファイルを読みません。ただし、 ``import "/etc/passwd";`` のように明示的に指定したディレクトリ外の再マッピングターゲットファイルが存在するときは ``/=/`` を追加すれば読み込むことができるようになります。

また、空の再マッピングプレフィックスは許可されていません。

再マッピングを行う対象が複数個ある場合は、最も長い共通プレフィックスを持つものが選択されます。

セキュリティ上の理由から、コンパイラがアクセスできるディレクトリには制限があります。
コマンドラインで指定されたソースファイルのパス(そのサブディレクトリを含む)と再マッピングで定義されたパスはimport文で使用できますが、それ以外はすべてリジェクトされます。追加のパス(そのサブディレクトリを含む)は ``--allow-paths /sample/path, /another/sample/path`` を使用することで許可することができます。

コントラクト内に :ref:`libraries <libraries>` を使用している場合、バイトコードが ``__$53aea86b7d70b31448b230b20ae141a537$__`` 形式の部分文字列を含んでいることに気付くはずです。これらは実際のライブラリアドレスのプレースホルダになります。
このプレースホルダは、完全修飾ライブラリ名のkeccak256ハッシュをhexエンコーディングした34文字のプレフィックスです。
また、バイトコードファイルの最後には、 ``// <プレースホルダ>  -> <完全修飾ライブラリ名>`` という形式の行も含まれます。
そして、プレースホルダがどのライブラリを表すかを識別するための完全修飾ライブラリ名ソースファイルのパスとライブラリ名を ``:`` で区切って指定します。
さらにリンカ(linker)として ``solc`` を使うことができます。これはリンカがある場所にライブラリアドレスを挿入することを意味します:

各ライブラリのアドレスを指定するか、ファイルごとに文字列(1行ごとに1つライブラリを指定してください)を格納するには、 ``solc`` で ``--libraries fileName`` オプションを使って ``--libraries "file.sol:Math:0x1234567890123456789012345678901234567890 file.sol:Heap:0xabCD567890123456789012345678901234567890"`` とコマンドに追加します。

オプション ``--link`` を付けて ``solc`` を呼び出すと、すべての入力ファイルは上記の ``__$53aea86b7d70b31448b230b20ae141a537$__`` 形式のリンクされていないバイナリ(hexエンコーディング)として解釈され、インプレースリンクとなります(入力が標準入力から読み取られる場合は、標準出力に書き込まれます)。この場合、 ``--libraries`` 以外のすべてのオプションは無視されます( ``-o`` を含みます)。

``solc`` が ``--standard-json`` オプション付きで呼び出された場合、(以下で説明するように)標準入力にJSONでの入力が求められ、標準出力にJSON出力が返されます。これは、より複雑で特に自動化された用途の場合に推奨されるインターフェースです。

.. note::
    ライブラリのプレースホルダは、以前はライブラリのハッシュではなく、ライブラリ自体の完全修飾名でした。
    このフォーマットはまだ ``solc --link`` によってサポートされてはいますが、コンパイラはもうその出力に対応していません。
    この変更は、完全修飾ライブラリ名の最初の36文字しか使用できないことから、ライブラリ間でのコンフリクトが発生する可能性を減らすために採用されました。

.. _evm-version:
.. index:: ! EVM version, compile target

Setting the EVM version to target
*********************************

コントラクトコードをコンパイルする際、特定の機能や動作を回避するために、コンパイルするEthereum virtual machine(EVM)のバージョンを指定できます。

.. warning::

   誤ったEVMバージョンでコンパイルすると、誤った、奇妙な、そして失敗するような振る舞いをするかもしれません。
   特にプライベートチェーンを実行している場合は、必ず一致するEVMバージョンを使用してください。

コマンドラインでは、次のようにEVMのバージョンを指定できます:

.. code-block:: shell

  solc --evm-version <VERSION> contract.sol

:ref:`standard JSON interface <compiler-api>` 内では、 ``"settings"`` フィールド内の ``"evmVersion"`` キーを使用してください:

.. code-block:: none

  {
    "sources": { ... },
    "settings": {
      "optimizer": { ... },
      "evmVersion": "<VERSION>"
    }
  }

Target options
--------------

以下は、ターゲットEVMのバージョンと、各バージョンで導入されたコンパイラ関連の変更のリストです。
各バージョン間の下位互換性は保証されていません。

- ``homestead`` (１番古いバージョン)
- ``tangerineWhistle``
   - 他のアカウントにアクセスするためのガス代が増加します。これはガス見積もりとオプティマイザに関連することにより起こります。
   - すべてのガスは、デフォルトでは外部コールとして送信されていましたが、以前は一定量を保持する必要がありました。
- ``spuriousDragon``
   - ``exp`` オペコードのガスコストが増加します。これはガス推定とオプティマイザに関連することにより起こります。
- ``byzantium`` (**default**)
   - オペコード ``returndatacopy`` 、 ``returndatasize`` 、 ``staticcall`` はアセンブリで利用可能です。
   - ``staticcall`` オペコードは、ライブラリ以外のviewやpureの修飾子が付いた関数を呼び出すときに使われます。これはEVMレベルで関数が状態を変更するのを防ぎます。つまり、無効な型変換を使ったときにも適用されます。
   - 関数呼び出しから返された動的データにアクセスすることが可能です。
   - ``revert`` オペコードが導入されました。 ``revert()`` がガスを無駄にしません。
- ``constantinople`` (開発中)
   - オペコード ``shl`` 、 ``shr`` および ``sar`` はアセンブリで利用できます。
   - シフト演算子はシフト演算コードを使用するため、必要なガスが少なくて済みます。

.. _compiler-api:

Compiler Input and Output JSON Description
******************************************

Solidityコンパイラとインターフェースをとるための推奨される方法は、いわゆるJSON入出力インターフェースです。
これは特に複雑で自動化されたセットアップの際に有用です。
コンパイラのすべてのディストリビューションで同じインタフェースが提供されています。

そのフィールドは度々変更されることがあり、いくつかはオプションです(このドキュメントに書かれてある通りです)。
ただし、私達は後方互換性のある変更だけをします。

コンパイラAPIはJSON形式の入力を要求し、コンパイル結果をJSON形式の出力に出力します。

以下のサブセクションでは、例を通してその形式を説明します。
下にあるようなコメントはもちろん使うことはできません。ここでは説明の目的で使用されているだけです。

Input Description
-----------------

.. code-block:: none

    {
      // Required: ソースコードの言語。"Solidity"、 "Vyper"、 "lll"、 "assembly"など
      language: "Solidity",
      // Required
      sources:
      {
        // キーはソースファイルのグローバルネームです。
        // インポートは再マッピングを通して他のファイルを使うことができます(下記を参照してください)
        "myFile.sol":
        {
          // Optional: ソースファイルのkeccak256ハッシュ
          // URL経由でインポートした場合、取得したコンテンツを検証するために使用されます。
          "keccak256": "0x123...",
          // Required("content"が使用されていない限りRequiredです。下記を参照してください): ソースファイルのURLs
          // URLはこの順番でインポートされ、結果はkeccak256ハッシュ(利用可能な場合)に対してチェックされます。
          // ハッシュが一致しない場合、またはいずれのURLでも成功しなかった場合は、エラーが発生します。
          "urls":
          [
            "bzzr://56ab...",
            "ipfs://Qma...",
            // If files are used, their directories should be added to the command line via
            // `--allow-paths <path>`.
            "file:///tmp/path/to/file.sol"
          ]
        },
        "mortal":
        {
          // Optional: ソースファイルのkeccak256ハッシュ
          "keccak256": "0x234...",
          // Required("urls"が使用されていない限りRequiredです): ソースファイルのリテラル
          "content": "contract mortal is owned { function kill() { if (msg.sender == owner) selfdestruct(owner); } }"
        }
      },
      // Optional
      settings:
      {
        // Optional: 再マッピングのソート済みリスト
        remappings: [ ":g/dir" ],
        // Optional: オプティマイザー設定
        optimizer: {
          // デフォルトではdisabledです
          enabled: true,
          // コードを実行する回数を最適化します。
          // 値が小さいほどイニシャルデプロイコストが最適化され、値が高いと高頻度で使用するときにより最適化されます。
          runs: 200
        },
        // コンパイルするEVMのバージョン。型チェックとコード生成に影響します。
        // homestead、tangerineWhistle、spuriousDragon、byzantium、constantinopleのいずれかです。
        evmVersion: "byzantium",
        // メタデータ設定(optional)
        metadata: {
          // URLではなくリテラルのみを使用(デフォルトではfalseです)
          useLiteralContent: true
        },
        // ライブラリのアドレスです。ここにすべてのライブラリが指定されていないと、出力データがライブラリにリンクされていない異なるオブジェクトになる可能性があります。
        libraries: {
          // 最上位のキーは、ライブラリが使用されているソースファイルの名前です。
          // 再マッピングが使用されている場合、このソースファイルは再マッピングが適用された後のグローバルパスと一致する必要があります。
          // このキーが空の文字列の場合、グローバルレベルを参照します。
          "myFile.sol": {
            "MyLib": "0x123123..."
          }
        }
        // 以下は、ファイル名とコントラクト名に基づいて目的の出力を選択するために使用できます。
        // このフィールドを省略すると、コンパイラは型チェックをロードして実行しますが、エラー以外の出力は生成しません。
        // ファーストレベルのキーはファイル名、セカンドレベルのキーはコントラクト名です。
        // 空のコントラクト名は、コントラクトではなくASTのようなソースファイル全体に関連付けられている出力に使用されます。
        // コントラクト名としてのスター(*)は、ファイル内のすべての契約を表します。
        // 同様に、ファイル名としてのスター(*)はすべてのファイルに一致します。
        // コンパイラが生成する可能性のあるすべての出力を選択するには、"outputSelection：{" * "：{" * "：[" * "]、" "：[" * "]}}"を使用します。
        // ただし、これはコンパイルプロセスを不必要に遅くする可能性があることに留意してください。
        //
        // 利用できるアウトプットの型は以下の通りです:
        //
        // ファイルレベル(コントラクト名として空文字列が必要):
        //   ast - 全ソースファイルのAST
        //   legacyAST - 全ソースファイルのlegacy AST
        //
        // コントラクトレベル(コントラクト名か"*"が必要です):
        //   abi - ABI
        //   devdoc - 開発者向けドキュメント(natspec)
        //   userdoc - ユーザー向けドキュメント(natspec)
        //   metadata - メタデータ
        //   ir - desugaringする前の新しいアセンブリフォーマット
        //   evm.assembly - desugaringした後の新しいアセンブリフォーマット
        //   evm.legacyAssembly - JSONでの古いスタイルのアセンブリ
        //   evm.bytecode.object - バイトコードオブジェクト
        //   evm.bytecode.opcodes - オペコードのリスト
        //   evm.bytecode.sourceMap - ソースマッピング(デバッグに役立ちます)
        //   evm.bytecode.linkReferences - リンクリファレンス(もしunlinked objectであれば)
        //   evm.deployedBytecode* - デプロイ時のバイトコード(evm.bytecodeと同じオプションがあります)
        //   evm.methodIdentifiers - 関数ハッシュのリスト
        //   evm.gasEstimates - 関数のガス代の推定
        //   ewasm.wast - eWASM S-expressionsフォーマット(atmをサポートしていません)
        //   ewasm.wasm - eWASM binaryフォーマット(atmをサポートしていません)
        //
        // `evm` 、` evm.bytecode` 、 `ewasm` などを使用すると、その出力のすべてのターゲット部分が選択されます。
        // さらに、 `*` はすべてを要求するためのワイルドカードとして使うことができます。
        //
        outputSelection: {
          // 各コントラクトのメタデータとバイトコードの出力を有効にします。
          "*": {
            "*": [ "metadata", "evm.bytecode" ]
          },
          // defに定義されているMyContractのabiおよびオペコード出力を有効にします。
          "def": {
            "MyContract": [ "abi", "evm.bytecode.opcodes" ]
          },
          // 各コントラクトのソースマップ出力を有効にします。
          "*": {
            "*": [ "evm.bytecode.sourceMap" ]
          },
          // すべての単一ファイルのlegacy AST出力を有効にします。
          "*": {
            "": [ "legacyAST" ]
          }
        }
      }
    }


Output Description
------------------

.. code-block:: none

    {
      // Optional: エラー/警告が発生しなかった場合は存在しません
      errors: [
        {
          // Optional: ソースファイル内の位置
          sourceLocation: {
            file: "sourceFile.sol",
            start: 0,
            end: 100
          ],
          // Mandatory: エラータイプ。"TypeError"、"InternalCompilerError"、"Exception"など。
          // すべての型のリストは以下を参照してください。
          type: "TypeError",
          // Mandatory: エラーが発生したコンポーネント。"general"、"ewasm"など。
          component: "general",
          // Mandatory ("error"もしくは"warning")
          severity: "error",
          // Mandatory
          message: "Invalid keyword"
          // Optional: ソースの場所でフォーマットされたメッセージ
          formattedMessage: "sourceFile.sol:100: Invalid keyword"
        }
      ],
      // これはファイルレベルの出力を含むものです。outputSelectionの設定によって制限/フィルタリングできます。
      sources: {
        "sourceFile.sol": {
          // ソースの識別子（ソースマップで使用）
          id: 1,
          // ASTオブジェクト
          ast: {},
          // legacy ASTオブジェクト
          legacyAST: {}
        }
      },
      // これはコントラクトレベルの出力を含むものです。outputSelectionの設定によって制限/フィルタリングできます。
      contracts: {
        "sourceFile.sol": {
          // 使用されている言語にコントラクト名がない場合、このフィールドは空の文字列と等しくなります。
          "ContractName": {
            // Ethereum Contract ABIです。もし空なら、空配列として表されます。
            // https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABIを参照してください。
            abi: [],
            // Metadata Outputのドキュメント(serialised JSON string)を参照してください。
            metadata: "{...}",
            // ユーザードキュメント(natspec)
            userdoc: {},
            // 開発者向けドキュメント(natspec)
            devdoc: {},
            // 中間表現(string)
            ir: "",
            // EVMに関連する出力
            evm: {
              // Assembly (string)
              assembly: "",
              // 過去の形式のアセンブリ(object)
              legacyAssembly: {},
              // バイトコードと関連する詳細
              bytecode: {
                // hex stringとしてのバイトコード
                object: "00fe",
                // オペコードのリスト(string)
                opcodes: "",
                // 文字列としてのソースマッピング。ソースマッピング定義を参照してください。
                sourceMap: "",
                // もし存在する場合、これはunlinked objectとなります。
                linkReferences: {
                  "libraryFile.sol": {
                    // バイトコードへのバイトオフセット。リンクすると、その位置の20バイトが置き換えられます。
                    "Library1": [
                      { start: 0, length: 20 },
                      { start: 200, length: 20 }
                    ]
                  }
                }
              },
              // 上記と同じレイアウトです。
              deployedBytecode: { },
              // 関数ハッシュのリスト
              methodIdentifiers: {
                "delegate(address)": "5c19a95c"
              },
              // ガス代の推定を行う関数
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
            // 出力に関連するeWASM
            ewasm: {
              // S-expressionsフォーマット
              wast: "",
              // バイナリ形式(hex string)
              wasm: ""
            }
          }
        }
      }
    }


Error types
~~~~~~~~~~~

1. ``JSONError``: JSON入力が必要なフォーマットに準拠していません。入力がJSONオブジェクトではない、言語がサポートされていない、など。
2. ``IOError``: IOとimportプロセスエラーです。利用できないURLや提供されたソースのハッシュの不一致など。
3. ``ParserError``: ソースコードが言語の規則に準拠していません。
4. ``DocstringParsingError``: コメントブロック内のNatSpecタグは解析できません。
5. ``SyntaxError``: シンタックスエラーです。``continue`` のような構文エラーは ``for`` ループの外側で使われます。
6. ``DeclarationError``: 無効な、解決できない、またはコンフリクトする識別子名です。例えば ``Identifier not found`` など。
7. ``TypeError``: 型システム内のエラーです。invalid type conversions や invalid assignmentsなど。
8. ``UnimplementedFeatureError``: この機能はコンパイラではサポートされていませんが、将来のバージョンでサポートされる予定です。
9. ``InternalCompilerError``: 内部バグがコンパイラで引き起こされました。 -これは問題として報告されるべきです。
10. ``Exception``: コンパイル中にUnknown failureが発生しました。 - これは問題として報告されるべきです。
11. ``CompilerError``: コンパイラスタックの無効な使用が起こりました。 - これは問題として報告されるべきです。
12. ``FatalError``: Fatal errorにより正しく処理されませんでした。 - これは問題として報告されるべきです。
13. ``Warning``: Warningです。コンパイルは止めませんでしたが、可能であれば対処するべきです。
