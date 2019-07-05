##################################
Expressions and Control Structures
##################################

.. index:: ! parameter, parameter;input, parameter;output, function parameter, parameter;function, return variable, variable;return, return


.. index:: if, else, while, do/while, for, break, continue, return, switch, goto

Control Structures
===================

Curly-braces languageで使われる様な制御構造のほとんどはSolidityでも使用可能です:

CやJavaScriptで知られる通常のセマンティクスに加えて、``if``、``else``、``while``、``do``、``for``、``break``、``continue``、``return`` があります。

条件文では括弧は省略できませんが、単文の周りの波括弧は省略可能です。

CやJavaScriptの様に非ブーリアンをブーリアンに変換することはできません。つまり ``if (1) { ... }`` はSolidityでは有効ではありません。

.. index:: ! function;call, function;internal, function;external

.. _function-calls:

Function Calls
==============

.. _internal-function-calls:

Internal Function Calls
-----------------------

現在のコントラクトのファンクションは直接（"内部的に"）呼び出すことができます。また、下記の無意味な例の様に再帰的に呼び出すこともできます。::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function g(uint a) public pure returns (uint ret) { return a + f(); }
        function f() internal pure returns (uint ret) { return g(7) + f(); }
    }

これらのファンクションの呼び出しはEVM内部で単純なジャンプとして変換されます。これには現在のメモリがクリアされないという影響があります。例えば、メモリの参照を内部で呼ばれたファンクションに渡すことは非常に効率的です。同じコントラクトのファンクションのみが内部的に呼び出すことが可能です。

全てのinternalファンクションの呼び出しは少なくとも1つのスタックのスロットを使い、最大で1024個しかそのスロットは使えないので、余計な再帰処理は避けたほうが良いでしょう。

.. _external-function-calls:

External Function Calls
-----------------------

``this.g(8);`` と ``c.g(2);`` という表現（``c`` はコントラクトのインスタンス）も有効なファンクションの呼び出しですが、今回はファンクションは直接ジャンプするのではなく、メッセージコールを通じて "外部的に" 呼び出されます。
実際のコントラクトは未だ作られていないため、``this`` でのファンクションの呼び出しはコンストラクタでは使用することができないということを覚えておいてください。

他のコントラクトのファンクションは外部的に呼び出す必要があります。外部コールに関して、全てのファンクションの引数はmemoryにコピーされなければいけません。

.. note::
    あるコントラクトから別のコントラクトへのファンクションコールはトランザクションを生成しません。トランザクション全体の一部としてのメッセージコールとなります。

他のコントラクトのファンクションを呼び出すときに、送るWeiの量やガスを特別なオプション ``.value()`` と ``.gas()`` を使ってそれぞれ決めることができます。送ったWeiは全てコントラクトのトータルバランスに追加されます。


::

    pragma solidity >=0.4.0 <0.6.0;

    contract InfoFeed {
        function info() public payable returns (uint ret) { return 42; }
    }

    contract Consumer {
        InfoFeed feed;
        function setFeed(InfoFeed addr) public { feed = addr; }
        function callFeed() public { feed.info.value(10).gas(800)(); }
    }

``info`` ファンクションではmodifierの ``payable`` を使う必要があります。使わない場合、``.value()`` オプションが使えません。

.. warning::
  ``feed.info.value(10).gas(800)`` は ``value`` とファンクションコールと一緒に送信される ``gas`` 量をローカルにセットするだけです。そして、最後の括弧が実際にファンクションをコールしています。そのため、このケースではファンクションは呼ばれていません。

呼ばれたコントラクトが存在しない場合（アカウントがコードを含んでいない場合）、もしくは呼ばれたコントラクト自体がエラーを投げた場合、ガス不足の場合にはファンクションが呼ばれると例外が発生します。

.. warning::
    他のコントラクトのやり取りは、潜在的な危険をはらんでいます。特に、事前にそのコントラクトのソースコードを知らなかった場合には危険です。現在のコントラクトは呼ばれたコントラクトにコントロールを渡すと、潜在的には何でもできてしまうかもしれません。たとえ呼ばれたコントラクトが既知の親コントラクトを継承していたとしても、継承したコントラクトは正しいインターフェースを持てば良いだけです。しかし、そのコントラクトの実行は完全に任意ですので、危険をもたらします。また、そのコントラクトがあなたのシステムの他のコントラクトを呼び出したり、初めの呼び出しが帰って来る前に呼び出し元のコントラクトを呼び返す様な事態に備えてください。つまり呼ばれたコントラクトはファンクションを通じて呼び出し元のコントラクトの状態変数を変えることができます。例えば、コントラクト内の状態変数の変更の後に外部のファンクションを呼び出すような形でファンクションを作ってください。そうすれば、あなたのコントラクトはリエントラントの悪用に対して攻撃されにくくなるでしょう。


Named Calls and Anonymous Function Parameters
---------------------------------------------

ファンクションよ呼ぶ際の引数は ``{ }`` で囲えば下記の例のように任意の順番で、名前を与えることができます。引数の名前はファンクションの宣言の際のパラメータと一致する必要がありますが、順番は任意です。

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        mapping(uint => uint) data;

        function f() public {
            set({value: 2, key: 3});
        }

        function set(uint key, uint value) public {
            data[key] = value;
        }

    }

Omitted Function Parameter Names
--------------------------------

使われなかったパラメータの名前（特に返り値）は省略することができます。
そのパラメータはスタック上に残りますが、アクセスできません。

::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        // omitted name for parameter
        function func(uint k, uint) public pure returns(uint) {
            return k;
        }
    }


.. index:: ! new, contracts;creating

.. _creating-contracts:

Creating Contracts via ``new``
==============================

コントラクトは ``new`` というキーワードを使って他のコントラクトを作ることができます。コントラクトを作るコントラクトがコンパイルされる時に、作られるコントラクトのフルコードは既知である必要があります。そうすれば、再帰的なcreation-dependenciesは起きません。

::

    pragma solidity ^0.5.0;

    contract D {
        uint public x;
        constructor(uint a) public payable {
            x = a;
        }
    }

    contract C {
        D d = new D(4); // will be executed as part of C's constructor

        function createD(uint arg) public {
            D newD = new D(arg);
            newD.x();
        }

        function createAndEndowD(uint arg, uint amount) public payable {
            // Send ether along with the creation
            D newD = (new D).value(amount)(arg);
            newD.x();
        }
    }

上記の例の様に、``.value()`` オプションを使って、``D`` のインスタンスを作る時に、Etherを送ることができます。しかし、ガス量を制限することはできません。もし生成が失敗（スタックの制限、バランスの不足等で）したら、例外が投げられます。

Order of Evaluation of Expressions
==================================

式の評価の順番は決まっていません（正確には、式ツリーの中の1つのノードの子が評価される順番は決まっていませんが、そのノードが評価される前にはもちろん評価されています）。唯一保証されているのは、宣言は順番に実行され、boolean式に対する短絡評価もなされます。詳細は :ref:`order` を参照ください。

.. index:: ! assignment

Assignment
==========

.. index:: ! assignment;destructuring

Destructuring Assignments and Returning Multiple Values
-------------------------------------------------------

Solidityは内部ではタプル型を許可しています。例えば、コンパイル時に定数を持つ潜在的に異なるタイプのオブジェクトのリストです。
このタプルは同時に複数の値を返す時に使うことができます。これらは新しく宣言された変数や既に存在している変数（もしくは一般的に左辺値）に割り当てることができます。

タプルはSolidityでは適切な型ではありません。式のシンタックスのグルーピングを作るためだけに使用することができます。

::

    pragma solidity >0.4.23 <0.6.0;

    contract C {
        uint[] data;

        function f() public pure returns (uint, bool, uint) {
            return (7, true, 2);
        }

        function g() public {
            // Variables declared with type and assigned from the returned tuple,
            // not all elements have to be specified (but the number must match).
            (uint x, , uint y) = f();
            // Common trick to swap values -- does not work for non-value storage types.
            (x, y) = (y, x);
            // Components can be left out (also for variable declarations).
            (data.length, , ) = f(); // Sets the length to 7
        }
    }

変数の宣言と、宣言なしの値の割り当てを混ぜることはできません。例えば、次の式は無効です: ``(x, uint y) = (1, 2);``

.. note::
    バージョン0.5.0以前では、片側が小さいサイズで、左辺か右辺（空の方）を補完する様なタプルを割り当てることができました。
    現在これは禁止です。つまり両辺とも同じ数の要素を持っている必要があります。

.. warning::
    参照型を含む複数の変数を同時に割り当てる際には気をつけてください。予期しないコピーを行う可能性があります。

Complications for Arrays and Structs
------------------------------------

配列や構造体の様な非値型に対するアサインのセマンティクスは少し複雑です。
状態変数に対する割り当ては常に独立したコピーを生成します。一方で、ローカル変数への割り当ては基本型（例えば32バイトに収まる静的タイプ）の時のみ独立したコピーを生成します。もし構造体か配列（``bytes`` や を含んでいる）が状態変数からローカル変数に割り当てられた場合、ローカル変数はオリジナルの状態変数への参照を持ちます。次にローカル変数へ割り当てても、ステートは変わらず、参照先のみ変更します。ローカル変数のメンバ（もしくは要素）への割り当てはステートを *変更します*。

下記の例で ``g(x)`` の呼び出しは ``x`` に何の影響も与えません。それはmemoryに独立したstorageの値を生成するからです。しかし、``h(x)`` は ``x`` を変更します。それはコピーではなく参照が渡されているからです。

::

    pragma solidity >=0.4.16 <0.6.0;

     contract C {
        uint[20] x;

         function f() public {
            g(x);
            h(x);
        }

         function g(uint[20] memory y) internal pure {
            y[2] = 3;
        }

         function h(uint[20] storage y) internal {
            y[3] = 4;
        }
    }

.. index:: ! scoping, declarations, default value

.. _default-value:

Scoping and Declarations
========================

宣言された変数はバイト表現が全てゼロのデフォルト値を与えられます。変数の "デフォルト値" はどの型であれ一般的な "zero-state" です。例えば、``bool`` のデフォルト値は ``false`` です。``uint`` もしくは ``int`` 型のデフォルト値は ``0`` です。静的サイズの配列や ``bytes1`` から ``bytes32`` では、個々の要素がその型に応じたデフォルト値で初期化されます。最後に、動的サイズの配列や ``bytes``、``string`` に関してはデフォルト値は空の配列もしくは空の文字列です。

Solidityでのスコープは広く使われているC99のルールに従います: 変数は宣言直後からその宣言を含む最小の ``{ }`` ブロックの最後までvisibleです。このルールの例外として、forループ内の初期化内で宣言された変数はそのforループが終わるまでの間でだけvisibleです。

コードブロック外で変数や他の宣言されたもの、例えばファンクション、コントラクト、ユーザー定義型などは宣言される前からvisibleです。つまり、宣言前から状態変数を使うことができ、ファンクションを再帰的に呼び出すことができます。

その結果、2つの変数が同じ名前を持っていますが、バラバラのスコープを持つため、下記の例は警告なしでコンパイルされます。

::

    pragma solidity ^0.5.0;
    contract C {
        function minimalScoping() pure public {
            {
                uint same;
                same = 1;
            }

            {
                uint same;
                same = 3;
            }
        }
    }

特別なC99のスコーピングルールの例として、下記の最初の ``x`` への割り当ては、実際はインナーの変数ではなく、アウターに割り当てられています。どの様な場合でもアウターの変数がシャドーイングされた時に警告が出ます。

::

    pragma solidity ^0.5.0;
    // This will report a warning
    contract C {
        function f() pure public returns (uint) {
            uint x = 1;
            {
                x = 2; // this will assign to the outer variable
                uint x;
            }
            return x; // x has value 2
        }
    }

.. warning::
    バージョン0.5.0以前のSolidityではJavaScriptと同じスコーピングのルールに従っていました。そのルールとは、ファンクション内で宣言された変数のスコープは、どこで宣言されたか関係なく、そのファンクション全体というものです。下記の例は、以前はコンパイルされていましたが、バージョン0.5.0以降はエラーとなります。

 ::

    pragma solidity ^0.5.0;
    // This will not compile
    contract C {
        function f() pure public returns (uint) {
            x = 2;
            uint x;
            return x;
        }
    }

.. index:: ! exception, ! throw, ! assert, ! require, ! revert, ! errors

.. _assert-and-require:

Error handling: Assert, Require, Revert and Exceptions
======================================================

Solidityはエラーを処理するのにstate-revertingの例外を使っています。現在のコール（とそのサブコール全て）内で変更されたものを全て元に戻し、さらに呼び出し元に対してエラーのフラグをたてます。便利なファンクション ``assert`` と ``require`` はもし条件が満たされていない場合に例外を投げるために使用されます。
``assert`` は内部エラーのテストと不変条件のチェックのためだけに使用するべきです。
``require`` は有効な条件を保証するために使用するのが良いでしょう。例えば入力、コントラクトの状態変数が合致しているか、もしくは外部コントラクトから返ってきた値のバリデーションに使うことができます。
正しく使えば、分析ツールはコントラクトのコンディションとフェイル ``assert`` になるファンクションコールを見極めるためにコントラクトの評価を行うことができます。正しく実装されたコードはフェイルアサートが出ないはずです。もしそれでもアサートが出た場合、それは直さなくてはいけないバグです。

例外を引き起こす方法が2つあります: ``revert`` ファンクションはエラーにフラグを立て、現在のコールをrevertするのに使用することができます。呼び出し元に対して、詳細を含むエラーメッセージを返すことができます。

.. note::
    以前は ``throw`` という ``revert()`` と同じセマンティクスのファンクションがありました。バージョン0.4.13で非推奨になり、バージョン0.5.0で削除されました。

サブコールで例外が発生した場合、その例外は自動的にどんどん出てきます（例外が再度投げられます）。このルールに対する例外は、``send`` と低レベルファンクションの ``call``、``delegatecall``、``staticcall`` で、これらは例外がどんどん出てこない代わりに、``false`` を返り値として最初に返します。

.. warning::
    低レベルファンクションの ``call``、``delegatecall``、``staticcall`` は、もし呼ばれたアカウントが存在しない場合、EVMの設計上、最初の返り値として ``true`` を返します。もし必要なら、呼び出す前に存在確認をしなければいけません。

例外のキャッチはまだ可能ではありません。

下記の例では、``require`` が入力に対するコンディションチェックを簡単に行なっているか見ることができます。また、``assert`` がどの様に内部エラーのチェックに使用されうるのか確認できます。覚えて欲しいのは、``require`` ではオプションでエラーメッセージを与えることができます。しかし ``assert`` ではできません。

::

    pragma solidity ^0.5.0;

    contract Sharer {
        function sendHalf(address payable addr) public payable returns (uint balance) {
            require(msg.value % 2 == 0, "Even value required.");
            uint balanceBeforeTransfer = address(this).balance;
            addr.transfer(msg.value / 2);
            // Since transfer throws an exception on failure and
            // cannot call back here, there should be no way for us to
            // still have half of the money.
            assert(address(this).balance == balanceBeforeTransfer - msg.value / 2);
            return address(this).balance;
        }
    }

``assert``-スタイルの例外は下記のシチュエーションで発生します:

#. 大きすぎるもしくは負のインデックスで配列にアクセスした場合(例： ``x[i]`` において ``i >= x.length`` もしくは ``i < 0``)。
#. 大きすぎるもしくは負のインデックスで固定長 ``bytesN`` にアクセスした場合。
#. 0で除算、剰余算を行なった場合(例: ``5 / 0`` もしくは ``23 % 0``)。
#. 負の数でシフトを行なった場合。
#. とても大きい、もしくは負の数を列挙型に変換した場合。
#. インターバルファンクション型のゼロ初期化された変数を呼び出した場合。
#. falseとなる引数で ``assert`` を呼び出した場合。

``require``-スタイルの例外は下記のシチュエーションで発生します:

#. falseとなる引数で ``require`` を呼び出した場合。
#. メッセージコールを通じてファンクションを呼び出したが、正常に終わらなかった場合（例えば、ガス不足、マッチするファンクションがなかった場合、それ自体が例外を投げた場合）、ただし低レベルのファンクション、``call``、``send``、``delegatecall``、``callcode`` もしくは ``staticcall`` が使用された場合を除きます。低レベルファンクションは例外を投げませんが、``false`` を返すことで失敗を示します。
#. ``new`` を使ってコントラクトを作ったものの、そのコントラクト作成が正常に終わらなかった場合（"正常に終わらない"の定義は上記参照）。
#. 何もコードを含まないコントラクトを外から呼び出した場合。
#. コントラクトが ``payable`` modifierが付いていないpublicのファンクションを通じてEtherを受け取った場合（コンストラクタ、フォールバックファンクションを含む）。
#. コントラクトがpublicのgetterファンクションを通じてEtherを受け取った場合。
#. ``.transfer()`` が失敗した場合。

内部では、Solidityは ``require``-スタイルの例外に対してrevert操作（``0xfd`` 命令）を行います。また、``require``-スタイルの例外を投げるために、無効な操作を実行します。どちらのケースにおいても、ステートに対してなされた変更の全てをEVMがrevertする様にします。revertする理由は、期待した効果が実際には起きず、命令の実行を続ける安全な方法がないためです。トランザクションの原子性（一連のやりとりが全て実行されるか、1つも実行されない状態になること）を保ちたいので、最も安全な方法は全ての変更をrevertし、トランザクション全体（もしくは少なくとも呼び出し）の結果を無効にすることです。``assert``-スタイルの例外は全ての使用可能なガスをその呼び出しに使ってしまう一方で、``require``-スタイルの例外はガスを使いません。これはMetropolisがリリースされた時からスタートしました。

以下の例は、どの様にrevertとrequireでエラーメッセージが使われているかを示しています:

::

    pragma solidity ^0.5.0;

    contract VendingMachine {
        function buy(uint amount) public payable {
            if (amount > msg.value / 2 ether)
                revert("Not enough Ether provided.");
            // Alternative way to do it:
            require(
                amount <= msg.value / 2 ether,
                "Not enough Ether provided."
            );
            // Perform the purchase.
        }
    }

与えられた文字列は、:ref:`abi-encoded <ABI>` となります。そしてそれはまるで ``Error(string)`` ファンクションを呼び出している様なものです。
上記の例では、``revert("Not enough Ether provided.");`` は下記のエラーの返り値としてセットされた16進数データを生成します:

.. code::

    0x08c379a0                                                         // Function selector for Error(string)
    0x0000000000000000000000000000000000000000000000000000000000000020 // Data offset
    0x000000000000000000000000000000000000000000000000000000000000001a // String length
    0x4e6f7420656e6f7567682045746865722070726f76696465642e000000000000 // String data
