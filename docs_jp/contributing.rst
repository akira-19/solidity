############
貢献
############

貢献はいつでも歓迎します

始めるには、まず、Solidityのコンポーネントとビルドプロセスを理解するため、:ref:`building-from-source` を確認してください。
また、Solidityでスマートコントラクトを書くことに精通することも役立つでしょう。

以下の領域が特に手助けが必要です。

* ドキュメントを改善する
* `StackExchange <https://ethereum.stackexchange.com>`_ や `Solidity Gitter <https://gitter.im/ethereum/solidity>`_ に投稿されたユーザーからの質問に答える
* `Solidity's GitHub issues <https://github.com/ethereum/solidity/issues>`_ に投稿されたissueに回答し修正する。特にこのタグがついた `good first issue <https://github.com/ethereum/solidity/labels/good%20first%20issue>`_ ものは外部貢献者のための入門的なissueになります。

このプロジェクト(issues, pull requests, Gitter channels含む) は `Contributor Code of Conduct <https://raw.githubusercontent.com/ethereum/solidity/develop/CODE_OF_CONDUCT.md>`_ でリリースされていることに注意してください。
あなたはこの規約に同意することになります。

issueの報告方法
====================

issueの報告には、
`GitHub issues tracker <https://github.com/ethereum/solidity/issues>`_
を使って、以下の項目を含めるようにしてください。

* どのSolidityバージョンをつかっているのか
* どんなソースコードだったのか(可能であれば)
* どのプラットフォームで実行していたか
* 再現方法
* issueの結果はどのようなものか
* 期待する動作はどのようなものか

issueを再現するために最低限のソースコードにするのは有用で、誤解を明らかにします。

Pull Requestsのためのワークフロー
==========================

貢献するために、 ``develop`` ブランチをフォークし、あなたの変更をそこにいれてください。
コミットメッセージには、あなたの変更(小さな変更でなければ)の *what* に加えて *why* を詳細にいれるべきです。

もしあなたが変更を加えた後に ``develop`` からpullする必要がある場合(例えばコンフリクトのマージする)、
私たちがあなたの変更のレビューをするのを簡単にするため、
``git merge`` を使うことは避けて、 ``git rebase`` を使ってください。

また、追加の機能を実装している場合、適切なテストケースを ``test/`` に追加してください。(下記参照)

しかし、もし大きな変更をする場合、まず、
`Solidity Development Gitter channel
<https://gitter.im/ethereum/solidity-dev>`_
に相談してください。
(上記で参照したものとは異なり、こちらはコンパイラと言語開発用です)

新機能とバグ修正は ``Changelog.md`` に追加されるべきです。
前回のエントリーのスタイルにしたがってください。

最後に、このプロジェクトでは、`coding style
<https://github.com/ethereum/solidity/blob/develop/CODING_STYLE.md>`_
を尊重してください。
私たちがCIテストを実行してはいますが、pull requestsを送る前にはローカルでテストしビルドするようにしてください。

あなたの貢献を感謝します。

Running the compiler tests
コンパイラテストの実行
==========================

``./scripts/tests.sh`` スクリプトはほとんどのSolidityテストを実行し、``aleth`` を走らせます。
``aleth`` がパスに見つかる場合は実行しますが、ダウンロードは行いませんので、最初にインストールしておく必要があります。
詳細を確認してください。

The ``./scripts/tests.sh`` script executes most Solidity tests and
runs ``aleth`` automatically if it is in the path, but does not download it,
so you need to install it first. Please read on for the details.

Solidityは様々なテストを含んでおり、ほとんどが ``soltest`` アプリケーションに同包されています。
それらのいくつかはテストモードで　``aleth`` クライアントを必要とし、``libz3`` を必要とするものもあります。
Solidity includes different types of tests, most of them bundled into the ``soltest``
application. Some of them require the ``aleth`` client in testing mode, others require ``libz3``.

``aleth`` も ``libz3`` も必要としない基本的なテストセットを実行するには、
``./scripts/soltest.sh --no-ipc --no-smt`` を実行します。
このスクリプトは ``./build/test/soltest`` を内部的に走らせます。
To run a basic set of tests that require neither ``aleth`` nor ``libz3``, run
``./scripts/soltest.sh --no-ipc --no-smt``. This script runs ``./build/test/soltest``
internally.

.. note ::

    Windows環境のこれらの作業で、上記の基本的なテストセットをGit Bash上でalethやlibz3なしで実行したい場合、
    ``./build/test/Release/soltest.exe -- --no-ipc --no-smt`` このコマンドを実行する必要があるでしょう。
    Those working in a Windows environment wanting to run the above basic sets without aleth or libz3 in Git Bash, you would have to do: ``./build/test/Release/soltest.exe -- --no-ipc --no-smt``.
    もし標準のコマンドプロンプトで実行する場合は、 ``.\build\test\Release\soltest.exe -- --no-ipc --no-smt`` を実行します。
    If you're running this in plain Command Prompt, use ``.\build\test\Release\soltest.exe -- --no-ipc --no-smt``.

``--no-smt`` オプションは ``libz3`` を必要とするテストを無効にし、 ``--no-ipc`` オプションは ``aleth`` を必要とするテストを無効にします。
The option ``--no-smt`` disables the tests that require ``libz3`` and
``--no-ipc`` disables those that require ``aleth``.

ipcテスト(生成されたコードのセマンティックをテストするもの)を実行したい場合、
`aleth <https://github.com/ethereum/aleth/releases/download/v1.5.0-alpha.7/aleth-1.5.0-alpha.7-linux-x86_64.tar.gz>`_
をインストールします。
その後、テストモードで実行します。 
``aleth --db memorydb --test -d /tmp/testeth``
If you want to run the ipc tests (that test the semantics of the generated code),
you need to install `aleth <https://github.com/ethereum/aleth/releases/download/v1.5.0-alpha.7/aleth-1.5.0-alpha.7-linux-x86_64.tar.gz>`_ 
and run it in testing mode: ``aleth --db memorydb --test -d /tmp/testeth``.

実際のテストを実行するには、 ``./scripts/soltest.sh --ipcpath /tmp/testeth/geth.ipc`` を使います。
To run the actual tests, use: ``./scripts/soltest.sh --ipcpath /tmp/testeth/geth.ipc``.

テストの一部を実行する場合は、フィルタを使います。
``./scripts/soltest.sh -t TestSuite/TestName --ipcpath /tmp/testeth/geth.ipc``
``TestName`` はワイルドカードを指定できます。 ``*`` 
To run a subset of tests, you can use filters:
``./scripts/soltest.sh -t TestSuite/TestName --ipcpath /tmp/testeth/geth.ipc``,
where ``TestName`` can be a wildcard ``*``.

例えば、実行したテストがあるとして、
``./scripts/soltest.sh -t "yulOptimizerTests/disambiguator/*" --no-ipc --no-smt``
を実行すると、disambiguatorの全てのテストが実行されます。
For example, here's an example test you might run;
``./scripts/soltest.sh -t "yulOptimizerTests/disambiguator/*" --no-ipc --no-smt``.
This will test all the tests for the disambiguator.

全てテストを確認するには、
``./build/test/soltest --list_content=HRF -- --ipcpath /tmp/irrelevant``
を使います。
To get a list of all tests, use
``./build/test/soltest --list_content=HRF -- --ipcpath /tmp/irrelevant``.

もしGDBでデバッグしたい場合は、通常とは異なる方法でビルドしていることが必要です。
以下のコマンドを ``build`` フォルダで実行します。
If you want to debug using GDB, make sure you build differently than the "usual".
For example, you could run the following command in your ``build`` folder:
::

   cmake -DCMAKE_BUILD_TYPE=Debug ..
   make


これは、あなたが ``--debug`` フラグを使いテストをデバッグする際のシンボルを生成します。
これで、関数や変数をbreakしたりprintで表示したりすることができるようになります。
This will create symbols such that when you debug a test using the ``--debug`` 
flag, you will have access to functions and variables in which you can break 
or print with.

``./scripts/tests.sh`` スクリプトも、``soltest`` で見つかったテストに加えて、コマンドラインテストを実行しテストをコンパイルします。
The script ``./scripts/tests.sh`` also runs commandline tests and compilation tests
in addition to those found in ``soltest``.

CIはEmscriptenのコンパイルを必要とする追加のテスト( ``solc-js`` やサードパーティのSolidityフレームワークのテストを含みます)を実行します。
The CI runs additional tests 
(including ``solc-js`` and testing third party Solidity frameworks) 
that require compiling the Emscripten target.

.. note ::

    いくつかの ``aleth`` のバージョンはテストに使うことはできません。
    SolidityのCIで使っているものと同じバージョンを使うことを推奨します。
    現在CIはバージョン ``aleth`` の ``1.5.0-alpha.7`` を使っています。
    Some versions of ``aleth`` can not be used for testing. We suggest using
    the same version that the Solidity continuous integration tests use.
    Currently the CI uses version ``1.5.0-alpha.7`` of ``aleth``.

Writing and running syntax tests
構文テストの実装と実行
--------------------------------

構文テストは、コンパイラが無効なコードに正しいエラーメッセージを生成し、適切に有効なコードを受け入れることをチェックします。
それらは ``tests/libsolidity/syntaxTests`` フォルダ内へ個別のファイルに格納されます。
それらのファイルは、個別のテストケースの正しい結果とアノテーションを含んでいます。
テストスイートは、正しい結果に対してチェックしコンパイルします。
Syntax tests check that the compiler generates the correct error 
messages for invalid code
and properly accepts valid code.
They are stored in individual files inside the ``tests/libsolidity/syntaxTests`` folder.
These files must contain annotations, stating the expected result(s) of the respective test.
The test suite compiles and checks them against the given expectations.

例えば、 ``./test/libsolidity/syntaxTests/double_stateVariable_declaration.sol`` では、
For example: ``./test/libsolidity/syntaxTests/double_stateVariable_declaration.sol``

::

    contract test {
        uint256 variable;
        uint128 variable;
    }
    // ----
    // DeclarationError: (36-52): Identifier already declared.

構文テストは、少なくともセパレータ ``// ----`` に続くテスト自身のコントラクトを含まなければいけません。
A syntax test must contain at least the contract under test itself, 
followed by the separator ``// ----``. 
セパレータに続くコメントは、正しいコンパイラエラーやワーニングを記述するのに使われます。
The comments that follow the separator are used to describe the
expected compiler errors or warnings. 
数字の範囲は、エラーが発生したソースコードの場所を指定しています。
The number range denotes the location in the source where the error occurred.
コントラクトにエラーやワーニングなしでコンパイルしたい場合、セパレータとコメントを削除することができます。
If you want the contract to compile without any errors or warning you can leave
out the separator and the comments that follow it.

上記の例だと、``variable``変数は２度宣言されてます。
これは、すでに宣言されていますという識別子の ``DeclarationError`` となります。
In the above example, the state variable ``variable`` was declared twice, 
which is not allowed. 
This results in a ``DeclarationError`` stating 
that the identifier was already declared.

The ``isoltest`` tool is used for these tests and you can find it under ``./build/test/tools/``. It is an interactive tool which allows
editing of failing contracts using your preferred text editor. 
Let's try to break this test by removing the second declaration of ``variable``:

::

    contract test {
        uint256 variable;
    }
    // ----
    // DeclarationError: (36-52): Identifier already declared.

Running ``./build/test/isoltest`` again results in a test failure:

::

    syntaxTests/double_stateVariable_declaration.sol: FAIL
        Contract:
            contract test {
                uint256 variable;
            }

        Expected result:
            DeclarationError: (36-52): Identifier already declared.
        Obtained result:
            Success


``isoltest`` prints the expected result next to the obtained result, and also
provides a way to edit, update or skip the current contract file, or quit the application.

It offers several options for failing tests:

- ``edit``: ``isoltest`` tries to open the contract in an editor so you can adjust it. It either uses the editor given on the command line (as ``isoltest --editor /path/to/editor``), in the environment variable ``EDITOR`` or just ``/usr/bin/editor`` (in that order).
- ``update``: Updates the expectations for contract under test. This updates the annotations by removing unmet expectations and adding missing expectations. The test is then run again.
- ``skip``: Skips the execution of this particular test.
- ``quit``: Quits ``isoltest``.

All of these options apply to the current contract, expect ``quit`` which stops the entire testing process.

Automatically updating the test above changes it to

::

    contract test {
        uint256 variable;
    }
    // ----

and re-run the test. It now passes again:

::

    Re-running test case...
    syntaxTests/double_stateVariable_declaration.sol: OK


.. note::

    Choose a name for the contract file that explains what it tests, e.g. ``double_variable_declaration.sol``.
    Do not put more than one contract into a single file, unless you are testing inheritance or cross-contract calls.
    Each file should test one aspect of your new feature.


Running the Fuzzer via AFL
==========================

Fuzzing is a technique that runs programs on more or less random inputs to find exceptional execution
states (segmentation faults, exceptions, etc). Modern fuzzers are clever and run a directed search
inside the input. We have a specialized binary called ``solfuzzer`` which takes source code as input
and fails whenever it encounters an internal compiler error, segmentation fault or similar, but
does not fail if e.g., the code contains an error. This way, fuzzing tools can find internal problems in the compiler.

We mainly use `AFL <http://lcamtuf.coredump.cx/afl/>`_ for fuzzing. You need to download and
install the AFL packages from your repositories (afl, afl-clang) or build them manually.
Next, build Solidity (or just the ``solfuzzer`` binary) with AFL as your compiler:

::

    cd build
    # if needed
    make clean
    cmake .. -DCMAKE_C_COMPILER=path/to/afl-gcc -DCMAKE_CXX_COMPILER=path/to/afl-g++
    make solfuzzer

At this stage you should be able to see a message similar to the following:

::

    Scanning dependencies of target solfuzzer
    [ 98%] Building CXX object test/tools/CMakeFiles/solfuzzer.dir/fuzzer.cpp.o
    afl-cc 2.52b by <lcamtuf@google.com>
    afl-as 2.52b by <lcamtuf@google.com>
    [+] Instrumented 1949 locations (64-bit, non-hardened mode, ratio 100%).
    [100%] Linking CXX executable solfuzzer

If the instrumentation messages did not appear, try switching the cmake flags pointing to AFL's clang binaries:

::

    # if previously failed
    make clean
    cmake .. -DCMAKE_C_COMPILER=path/to/afl-clang -DCMAKE_CXX_COMPILER=path/to/afl-clang++
    make solfuzzer

Otherwise, upon execution the fuzzer halts with an error saying binary is not instrumented:

::

    afl-fuzz 2.52b by <lcamtuf@google.com>
    ... (truncated messages)
    [*] Validating target binary...

    [-] Looks like the target binary is not instrumented! The fuzzer depends on
        compile-time instrumentation to isolate interesting test cases while
        mutating the input data. For more information, and for tips on how to
        instrument binaries, please see /usr/share/doc/afl-doc/docs/README.

        When source code is not available, you may be able to leverage QEMU
        mode support. Consult the README for tips on how to enable this.
        (It is also possible to use afl-fuzz as a traditional, "dumb" fuzzer.
        For that, you can use the -n option - but expect much worse results.)

    [-] PROGRAM ABORT : No instrumentation detected
             Location : check_binary(), afl-fuzz.c:6920


Next, you need some example source files. This makes it much easier for the fuzzer
to find errors. You can either copy some files from the syntax tests or extract test files
from the documentation or the other tests:

::

    mkdir /tmp/test_cases
    cd /tmp/test_cases
    # extract from tests:
    path/to/solidity/scripts/isolate_tests.py path/to/solidity/test/libsolidity/SolidityEndToEndTest.cpp
    # extract from documentation:
    path/to/solidity/scripts/isolate_tests.py path/to/solidity/docs docs

The AFL documentation states that the corpus (the initial input files) should not be
too large. The files themselves should not be larger than 1 kB and there should be
at most one input file per functionality, so better start with a small number of.
There is also a tool called ``afl-cmin`` that can trim input files
that result in similar behaviour of the binary.

Now run the fuzzer (the ``-m`` extends the size of memory to 60 MB):

::

    afl-fuzz -m 60 -i /tmp/test_cases -o /tmp/fuzzer_reports -- /path/to/solfuzzer

The fuzzer creates source files that lead to failures in ``/tmp/fuzzer_reports``.
Often it finds many similar source files that produce the same error. You can
use the tool ``scripts/uniqueErrors.sh`` to filter out the unique errors.

Whiskers
========

*Whiskers* is a string templating system similar to `Mustache <https://mustache.github.io>`_. It is used by the
compiler in various places to aid readability, and thus maintainability and verifiability, of the code.

The syntax comes with a substantial difference to Mustache. The template markers ``{{`` and ``}}`` are
replaced by ``<`` and ``>`` in order to aid parsing and avoid conflicts with :ref:`inline-assembly`
(The symbols ``<`` and ``>`` are invalid in inline assembly, while ``{`` and ``}`` are used to delimit blocks).
Another limitation is that lists are only resolved one depth and they do not recurse. This may change in the future.

A rough specification is the following:

Any occurrence of ``<name>`` is replaced by the string-value of the supplied variable ``name`` without any
escaping and without iterated replacements. An area can be delimited by ``<#name>...</name>``. It is replaced
by as many concatenations of its contents as there were sets of variables supplied to the template system,
each time replacing any ``<inner>`` items by their respective value. Top-level variables can also be used
inside such areas.
