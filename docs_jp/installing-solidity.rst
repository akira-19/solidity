.. index:: ! installing

.. _installing-solidity:

################################
Installing the Solidity Compiler
################################

Versioning
==========

Solidityのバージョンは `semantic versioning <https://semver.org>`_ に準じており、リリースに加え、**nightly development builds** も使用可能となっています。ドキュメントに含んでいないものや大きな変更を含んだ成果が組み込まれているものの、nightly buildsは動作を保証されていません。
最新バージョンを使用することをお薦めします。下記のパッケージインストーラでは最新バージョンを使用しています。

Remix
=====

*小規模のコントラクトやさっとSolidityを学ぶのにRemixをお薦めしています。*

`Remix onlineにアクセスして下さい <https://remix.ethereum.org/>`_。何もインストールする必要はありません。
もしインターネット接続無しで使用したい場合は https://github.com/ethereum/remix-live/tree/gh-pages にアクセスし、指示に従って ``.zip`` ファイルをダウンロードして下さい。

追加オプションとしてこのページではコマンドラインのSolidityコンパイラのインストール方法を紹介しています。大規模なコントラクトやもっと複雑なコンパイルオプションが必要な場合にはコマンドラインコンパイラを使用してください。

.. _solcjs:

npm / Node.js
=============

Solidityコンパイラである `solcjs` の便利で簡単なインストール方法として `npm` を使用してください。`solcjs` はこの後紹介するコンパイラへのアクセス方法に比べて機能は少ないです。:ref:`commandline-compiler` のドキュメンテーションでは全ての機能が備わっている `solc` というコンパイラを使用します。`solcjs` の使用方法はその `レポジトリ <https://github.com/ethereum/solc-js>`_ にドキュメント化されています。

注意: solc-jsプロジェクトはEmscriptenを使うことでC++ `solc` から派生されています。つまり両方とも同じコンパイラのソースコードを使用しているということです。`solc-js` はJavascriptのプロジェクトで（Remixの様に）直接使用可能です。使い方はsolc-jsレポジトリを参照して下さい。

.. code-block:: bash

    npm install -g solc

.. note::

    コマンドラインの実行ファイル名は `solcjs` です。

    `solcjs` のコマンドラインオプションは `solc` と互換性がなく、`solc` では動く様なツール（例えば `geth` ）は `solcjs` では動作しません。

Docker
======

コンパイラ用に最新のdockerビルドが提供されています。``stable`` レポジトリはリリースされたバージョンを含み、``nightly`` レポジトリはdevelopブランチに潜在的に不安定になりうる変更を含んでいます。

.. code-block:: bash

    docker run ethereum/solc:stable --version

現在、dockerイメージはコンパイラ実行ファイルだけを含んでいます。そのためソースとアウトプットディレクトリをリンクする作業が必要です。

Binary Packages
===============

Solidityのバイナリパッケージは `solidity/releases <https://github.com/ethereum/solidity/releases>`_ で利用可能です。

更にUbuntu用にPPA（Personal Package Archive）もあるので、下記のコマンドで最新の安定バージョンを取得可能です:

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install solc

nightlyバージョンは下記のコマンドでインストール可能です:

.. code-block:: bash

    sudo add-apt-repository ppa:ethereum/ethereum
    sudo add-apt-repository ppa:ethereum/ethereum-dev
    sudo apt-get update
    sudo apt-get install solc

`snap package <https://snapcraft.io/>`_ もリリースしています。これは全ての `supported Linuxディストリビューション <https://snapcraft.io/docs/core/install>`_ でインストール可能です。solcの最新の安定バージョンは下記でインストールできます。

.. code-block:: bash

    sudo snap install solc

もし最新のSolidityのdevelopmentバージョンの最近の変更のテストを手伝って頂けるのであれば、下記を使用してください:

.. code-block:: bash

    sudo snap install solc --edge

最新developmentバージョンに限られますが、Arch Linuxもパッケージがあります:

.. code-block:: bash

    pacman -S solidity

Homebrewでbuild-from-sourceとしてSolidityのコンパイラを提供しています。Pre-built bottlesは現在サポートされていません。

.. code-block:: bash

    brew update
    brew upgrade
    brew tap ethereum/ethereum
    brew install solidity

もしSolidityの特定バージョンが必要であればHomebrew formulaをGithubから直接インストールできます。

`solidity.rb commits on Github <https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb>`_ を確認して下さい。

``solidity.rb`` の特定のコミットのraw file linkを持つまでは過去のリンクを参照して下さい。

``brew`` を使用してインストールして下さい:

.. code-block:: bash

    brew unlink solidity
    # Install 0.4.8
    brew install https://raw.githubusercontent.com/ethereum/homebrew-ethereum/77cce03da9f289e5a3ffe579840d3c5dc0a62717/solidity.rb

Gentoo Linuxも ``emerge`` を使用してインストール可能なSolidityのパッケージを提供しています:

.. code-block:: bash

    emerge dev-lang/solidity

.. _building-from-source:

Building from Source
====================

Prerequisites - Linux
---------------------

SolidityのLinux buildのために下記のdependenciesをインストールする必要があります:

+-----------------------------------+-------------------------------------------------------+
| Software                          | Notes                                                 |
+===================================+=======================================================+
| `Git for Linux`_                  | Command-line tool for retrieving source from Github.  |
+-----------------------------------+-------------------------------------------------------+

.. _Git for Linux: https://git-scm.com/download/linux

Prerequisites - macOS
---------------------

macOS用に最新バージョンの `Xcodeがインストールされている <https://developer.apple.com/xcode/download/>`_ ことを確認して下さい。これには `Clang C++ compiler <https://en.wikipedia.org/wiki/Clang>`_ と `Xcode IDE <https://en.wikipedia.org/wiki/Xcode>`_、それに他のOS XでC++アプリを開発するのに必要なAppleのdevelopmentツールが含まれています。もしXcodeをインストールするのが初めて、もしくは新しいバージョンをインストールしたばかりなのであれば、コマンドラインbuildsをする前にライセンスに同意する必要があります:

.. code-block:: bash

    sudo xcodebuild -license accept

私たちのOS X buildsは外部のdependenciesをインストールするのに `Homebrew　package managerのインストール <http://brew.sh>`_ を要求しています。もし始めから行いたい場合は、こちらが `Homebrewのアンインストール
<https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/FAQ.md#how-do-i-uninstall-homebrew>`_ 方法です。


Prerequisites - Windows
-----------------------

SolidityのWindows buildsに下記のdependenciesのインストールが必要です:

+-----------------------------------+-------------------------------------------------------+
| Software                          | Notes                                                 |
+===================================+=======================================================+
| `Git for Windows`_                | Command-line tool for retrieving source from Github.  |
+-----------------------------------+-------------------------------------------------------+
| `CMake`_                          | Cross-platform build file generator.                  |
+-----------------------------------+-------------------------------------------------------+
| `Visual Studio 2017 Build Tools`_ | C++ compiler                                          |
+-----------------------------------+-------------------------------------------------------+
| `Visual Studio 2017`_  (Optional) | C++ compiler and dev environment.                     |
+-----------------------------------+-------------------------------------------------------+

もし既にIDEを持っており、コンパイラとライブラリだけが必要な場合には、Visual Studio 2017ビルドツールをインストールできます。

Visual Studio 2017はIDEと必要なコンパイラとライブラリを提供しています。そのためもしIDEを持っておらずSolidityの開発を行いたい場合にはVisual Studio 2017は全てを簡単にセットアップするための選択肢かもしれません。

こちらがVisual Studio 2017 Build ToolsもしくはVisual Studio 2017でインストールされるコンポーネントのリストです。

* Visual Studio C++ core features
* VC++ 2017 v141 toolset (x86,x64)
* Windows Universal CRT SDK
* Windows 8.1 SDK
* C++/CLI support

.. _Git for Windows: https://git-scm.com/download/win
.. _CMake: https://cmake.org/download/
.. _Visual Studio 2017: https://www.visualstudio.com/vs/
.. _Visual Studio 2017 Build Tools: https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2017

Clone the Repository
--------------------

ソースコードをクローンするのに下記のコマンドを実行して下さい:

.. code-block:: bash

    git clone --recursive https://github.com/ethereum/solidity.git
    cd solidity

もしSolidityの開発に助力頂けるのであればSolidityをforkしてセカンドリモートとしてあなたの個人的なforkを追加して下さい:

.. code-block:: bash

    git remote add personal git@github.com:[username]/solidity.git

External Dependencies
---------------------

macOS、Windows、多数のLinuxディストリビューションで必要な全ての外部dependenciesをインストールするヘルパースクリプトがあります。

.. code-block:: bash

    ./scripts/install_deps.sh

もしくはWindows上では:

.. code-block:: bat

    scripts\install_deps.bat


Command-Line Build
------------------

**開発を始める前に外部dependenciesをインストールするのを忘れないでください（上記参照）。**

Solidityプロジェクトはビルドを設定するためにCMakeを使っています。繰り返して行うbuildを高速化するためにccacheをインストールした方が良いでしょう。CMakeは自動的にccacheをピックアップします。
SolidityのビルドはLinux、macOSや他のUniX上ではほぼ同じです。

.. code-block:: bash

    mkdir build
    cd build
    cmake .. && make

もしくはもっと簡単に:

.. code-block:: bash

    #note: これはbinaries solcとsoltestをusr/local/bin
    ./scripts/build.sh上にインストールします。

そしてWindowsでは:

.. code-block:: bash

    mkdir build
    cd build
    cmake -G "Visual Studio 15 2017 Win64" ..

後半のやり方ではbuildディレクトリに **solidity.sln** を作成します。これをダブルクリックするとVisual Studioが起動するはずです。**Release**  configurationをビルドすることをお薦めしますが、他は全て動作します。

他の方法として、Windowsのコマンドラインでもビルドできます:

.. code-block:: bash

    cmake --build . --config Release

CMake options
=============

もし何のCMakeオプションが使用可能か興味があるのであれば ``cmake .. -LH`` を動かしてください。

.. _smt_solvers_build:

SMT Solvers
-----------
Solidityはデフォルトでシステム内でSMT solversがあれば、それを使ってビルドすることができます（デフォルトで使用します）。`cmake` オプションで全てのsolverは無効にできます。

*注意: いくつかの例においては潜在的にビルドの失敗を引き起こす場合があります。*


デフォルトで有効になっていますが、buildフォルダ内ではsolverは無効にできます:

.. code-block:: bash

    # disables only Z3 SMT Solver.
    cmake .. -DUSE_Z3=OFF

    # disables only CVC4 SMT Solver.
    cmake .. -DUSE_CVC4=OFF

    # disables both Z3 and CVC4
    cmake .. -DUSE_CVC4=OFF -DUSE_Z3=OFF

The version string in detail
============================

Solidityのバージョン文字列は4つの要素を含んでいます:

- バージョンナンバー
- プレリリースタグ、通常 ``develop.YYYY.MM.DD`` もしくは ``nightly.YYYY.MM.DD``
- ``commit.GITHASH`` のフォーマットでコミット
- 任意の数のアイテムを持ったプラットフォームで、そのプラットフォームとコンパイラの詳細が記述されている

もしローカルな修正があった場合にはコミットは ``.mod`` という接尾辞がつきます。

これらの要素はSemverの要求通りに結合され、SolidityのプレリリースタグはSemverのプレリリースに相当します。そしてSolidityのコミットとプラットフォームの2つでSemver buildのメタデータを作ります。

リリースの例: ``0.4.8+commit.60cc1668.Emscripten.clang``.

プレリリースの例: ``0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang``

Important information about versioning
======================================

リリース後はパッチバージョンのレベルは上がっていきます。これはパッチレベルの変更だけは進んでいくと考えているからです。変更がマージされたらバージョンはsemverと変更の重要度により
上がっていきます。最後に、リリースは常に現在のnightly buildのバージョンでされますが、``prerelease`` の指定はありません。

例:

0. 0.4.0版がリリース
1. nightly buildの0.4.1版ができる
2. 大きな変更がないので、バージョンの変更はなし
3. 大きな変更があるので、バージョンが0.5.0に上がる
4. 0.5.0がリリースされる

このバージョニングは :ref:`version pragma <version_pragma>` でちゃんと動作します。
