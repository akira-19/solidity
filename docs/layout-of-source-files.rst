********************************
Layout of a Solidity Source File
********************************

ソースファイルは任意の数の :ref:`コントラクトの定義<contract_structure>`、import_ の指示、:ref:`pragmaの指示<pragma>` を含むことができます。

.. index:: ! pragma

.. _pragma:

Pragmas
=======

``pragma`` というキーワードは特定のコンパイラの機能を使用可能にするもしくはコンパイラのバージョンをチェックするのに使用することができます。pragma指示は常にローカルのソースファイルにあるので、プロジェクト全体で使用可能にしたい場合にはpragmaを全てのファイルに追加する必要があります。もし他のファイルを :ref:`インポート<import>` してもそのファイルから得たpragmaは自動的にはインポートしているファイルには適用されません。

.. index:: ! pragma, version

.. _version_pragma:

Version Pragma
--------------

互換性がない可能性がある未来のコンパイラでのコンパイルを避けるために、ソースファイルはいわゆるバージョンPragmaの注記をつけることができます（そしてつけるべきです）。私たちはこの様な変更を最小限にする努力をしていますし、特にセマンティクス上の変更がシンタックスの変更も必要とする様な変更は通知しています。しかし、もちろん常にできる訳ではないので、少なくともブレーキングチェンジを含む様な大きな変更があるときは変更のログを見ることは良いことです。この様な大きな変更がある場合にはバージョンは ``0.x.0`` もしくは ``x.0.0`` の様になります。

バージョンPragmaは下記の様に使用されます::

  pragma solidity ^0.5.2;

この様なソースファイルはバージョン0.5.2より低いバージョンのコンパイラでコンパイルすることはありませんし、0.6.0以上のコンパイラでコンパイルすることもありません（後者の条件は ``^`` を使うことで追加しています）。このアイデアの根本はバージョン ``0.6.0`` まで大きな変更（ブレーキングチェンジ）がなく、コードが意図した通りにコンパイルされるということを保証されるというものです。コンパイラのバージョンを固定しないので、bugfixされたバージョンも使用可能です。

もっと複雑なコンパイラのバージョンのルールを指定することもできます。`npm <https://docs.npmjs.com/misc/semver>`_ を使った方法に準拠します。

.. note::
  version pragmaを使ってもコンパイラのバージョンを変えることはできません。また、コンパイラの機能を有効にしたり無効にしたりもできません。ただコンパイラにpragmaで要求されたバージョンと合っているかチェックさせるだけです。合っていなければコンパイラはエラーを出します。

.. index:: ! pragma, experimental

.. _experimental_pragma:

Experimental Pragma
-------------------

2つ目のpragmaは実験的なpragmaです。これはコンパイラや言語のまだデフォルトで有効になっていない機能を有効にするのに使うことができます。下記の実験的なpragmaは現在サポートされています。


ABIEncoderV2
~~~~~~~~~~~~

新しいABI encoderは任意にネストされた配列と構造体をエンコード、デコードできます。これはあまり最適化されていないコードを生成します。（この部分のオプティマイザは未だ開発中です。）そして、古いエンコーダほどテストがされていません。``pragma experimental ABIEncoderV2;`` を使うことでこれを有効化できます。

.. _smt_checker:

SMTChecker
~~~~~~~~~~

このコンポーネントはSolidityのコンパイラが組まれているときに有効でなければならない。そのためSolidityのバイナリでは利用できない。:ref:`build instructions<smt_solvers_build>` はどの様にこのオプションを有効にしているか説明しています。
これはほとんどのバージョンのUbuntu PPAのリリースのために有効化されるが、solc-js、Dockerイメージ、 Windowsバイナリやstatically-built Linuxバイナリのためではありません。

もし ``pragma experimental SMTChecker;`` を使うなら、SMT solverにクエリすることで取得される追加の安全警告を受け取ります。このコンポーネントはまだ全てのSolidityの機能をサポートしていないので、たくさんの警告を発する可能性が高いです。もしサポートされていない機能がレポートされた場合でも、その分析は完璧ではないかもしれません。

.. index:: source file, ! import

.. _import:

Importing other Source Files
============================

Syntax and Semantics
--------------------

"default export"はありませんが、SolidityはJavascript（ES6）の様なimportの宣言をサポートしています。

グローバルのレベルで、下記の様なimportの宣言ができます。

::

  import "filename";

この宣言は全てのグローバルな記号を"filename" (とそこにインポートされた記号)から現在のグローバルスコープ（ES6とは違いますがSolidityに後方互換性があります）にインポートします。 この単純な使い方は推奨されません。なぜなら、名前空間を予期せぬ方法で汚してしまうからです。もし"filename"内でトップレベルのアイテムを追加したら、"filename"からインポートした全てのファイルで自動的にそのアイテムが現れます。特定の記号だけを明示的にインポートした方が良いです。

下記の例では全ての要素が ``"filename"`` から来たグローバルな記号である新しいグローバルな記号 ``symbolName`` が作られます。

::

  import * as symbolName from "filename";

もし名前の重複があった場合には、インポートの際に名前を変えることができます。次のコードでは新しいグローバルな記号 ``alias`` と ``symbol2`` を作ります。それぞれ ``"filename"`` の中の ``symbol1`` と ``symbol2`` を参照しています。

::

  import {symbol1 as alias, symbol2} from "filename";



次の例はES6の一部ではありませんが、おそらく便利でしょう。

::

  import "filename" as symbolName;

これは ``import * as symbolName from "filename";`` と等価です。

.. note::
  もし `import "filename.sol" as moduleName;` を使うのであれば、`"filename.sol"` の中から `moduleName.C` として `C` と呼ばれるコントラクトにアクセスしてください。`C` は直接使わないでください。

Paths
-----

上では、``filename`` は常にディレクトリのセパレータとしての ``/``、現在のディレクトリとしての ``.``、親ディレクトリとしての ``..`` と一緒にパスとして使われていました。``.`` と ``..`` は ``/`` の後に続かなければ、現在もしくは親ディレクトリとしては扱われません。全てのパスは ``.`` もしくは ``..`` で始まらなければ絶対パスとして扱われます。

現在のファイルと同じディレクトリにあるファイル ``x`` をインポートするためには、``import "./x" as x;`` を使ってください。

実際にどの様にパスが読み込まれるかはコンパイラによります（下記参照）。一般的に、ディレクトリ構造はローカルに限定されません。例えばipfs、httpやgitを通じて得たリソースを指定することも可能です。

.. note::
    常に ``import "./filename.sol";`` の様な相対パスを使ってください。また、パスを指定するのに ``..`` を使うのは避けてください。後のケースではおそらくグローバルパスを使い、下記で説明するリマッピングをセットアップするのが良いでしょう。

Use in Actual Compilers
-----------------------

コンパイラを呼び出す時に、パスの最初の要素とパスのプレフィックスのリマッピングをどの様に指定するか決めることができます。例えば、あるリマッピングをセットアップしたら、仮のディレクトリ ``github.com/ethereum/dapp-bin/library`` からインポートしたもの全てが実際にはローカルのディレクトリ ``/usr/local/dapp-bin/library`` から読み込まれているといった様なことができます。
もし複数のリマッピングを使うと、一番長いキーをもつリマッピングが最初に適用されます。
空のプレフィックスは使えません。リマッピングはコンテキストに依存します。そのため、例えば同じ名前の異なるバージョンのライブラリをインポートするためにパッケージを設計できます。

**solc**:

solc（コマンドラインコンパイラ）に、``context:prefix=target`` 属性としてパスのリマッピングを渡します。``context:`` と ``=target`` のパートはオプションです（この場合 ``target`` が ``prefix`` のデフォルトとなります）。通常ファイルの全てのリマッピング値はコンパイルされます（それらの依存関係も含めて）。

このメカニズムは後方互換性をもち（ファイル名に ``=`` もしくは ``:`` を含んでいない限り）、そのためブレーキングチェンジにはなりません。``prefix`` で始まるファイルをインポートする ``context`` ディレクトリの中もしくは以下にある全てのファイルは ``prefix`` を ``target`` に変更することでリダイレクトされます。

例えば、もし ``github.com/ethereum/dapp-bin/`` をローカルの ``/usr/local/dapp-bin`` にコピーしたら、ソースファイル内で以下が使える様になります。

::

  import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;

そしてコンパイラを使用してください:

.. code-block:: bash

  solc github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ source.sol

もっと複雑な例として、もし ``/usr/local/dapp-bin_old`` を参照している古いバージョンのdapp-binを使っているモジュールを使っているとしたら、次のコードを使用することができます。

.. code-block:: bash

  solc module1:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ \
       module2:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin_old/ \
       source.sol

上記は ``module2`` の中の全てのインポートは古いバージョンで使われるが、``module1`` は新しいバージョンで使われるという意味です。

.. note::

  ``solc`` は特定のディレクトリからのファイルを含めるのを許可するだけです。そのファイルははっきりと明示されたソースファイルの1つ、もしくはリマッピングのターゲットのディレクトリ（もしくはサブディレクトリ）の中にある必要があります。もし直接的に含めたい場合にはリマッピングに ``/=/`` を追加して下さい。

もし有効なファイルを参照する複数のリマッピングがあった場合には、一番長い共通のプレフィックスがついているリマッピングが選択されます。

**Remix**:

`Remix <https://remix.ethereum.org/>`_ は自動的にGithubにリマッピングし、自動的にネットワークを通じてファイルを引っ張ってきます。上記の様な繰り返し可能なマッピングをインポートできます。例えば、

::
  import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;

Remixはおそらく将来的に他のソースコードプロバイダを追加するかもしれません。

.. index:: ! comment, natspec

Comments
========

1行コメント(``//``)と複数行コメント(``/*...*/``)が使用可能です。

::

  // これは1行コメントです。

  /*
  これは複数行
  コメントです。
  */

.. note::
  1行コメントはutf8エンコードにおいてどのunicode方式のラインブレーク(LF, VF, FF, CR, NEL, LS, PS)でも終了します。ラインブレークはコメントの後でもソースコードの一部となっているため、ascii記号（NEL, LS, PS）でない場合にはパーサーエラーを起こします。

さらに別のタイプのnatspecコメントというコメントがあります。これは :ref:`style guide<natspec>` で詳細を確認できます。これはトリプルスラッシュ(``///``)かダブルアスタリスクブロック(``/** ... */``)で書かれ、ファンクションの宣言の直前に書かれます。

ファンクションを付記したり、形式を検証するための条件を注記したり、ユーザーがファンクションを実行する時に表示される **confirmation text** を追加するために、このコメントの中の `Doxygen <https://en.wikipedia.org/wiki/Doxygen>`_-styleタグを使うことができます。

次の例の中ではコントラクトのタイトル、2つのファンクションのパラメータの説明、2つの返り値が示されています。

::

    pragma solidity >=0.4.0 <0.6.0;

    /** @title Shape calculator. */
    contract ShapeCalculator {
        /** @dev Calculates a rectangle's surface and perimeter.
          * @param w Width of the rectangle.
          * @param h Height of the rectangle.
          * @return s The calculated surface.
          * @return p The calculated perimeter.
          */
        function rectangle(uint w, uint h) public pure returns (uint s, uint p) {
            s = w * h;
            p = 2 * (w + h);
        }
    }
