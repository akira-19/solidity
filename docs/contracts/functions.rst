.. index:: ! functions

.. _functions:

*********
Functions
*********

.. _function-parameters-return-variables:

Function Parameters and Return Variables
========================================

JavaScriptに見られる様に、ファンクションは入力としてパラメータをとります。JavaScriptやCと違い、ファンクションは任意の数の値を出力できます。

Function Parameters
-------------------

ファンクションのパラメータは変数と同じ様に宣言されます。使われなかったパラメータは省略されます。

例えば、もし2つの整数でexternal callをしたい場合、次の様にできます::

    pragma solidity >=0.4.16 <0.6.0;

    contract Simple {
        uint sum;
        function taker(uint _a, uint _b) public {
            sum = _a + _b;
        }
    }

ファンクションパラメータはローカル変数としても使えますし、その値を割り当てすることもできます。

.. note::

  :ref:`external function<external-function-calls>` は入力パラメータとして多次元配列を受け入れません。この機能に関しては、ソースファイルに ``pragma experimental ABIEncoderV2;`` を追加して、新しい実験的な ``ABIEncoderV2`` 機能を有効にすれば使うことができます。

  :ref:`internal function<external-function-calls>` はこの機能を有効にしなくても多次元配列を使うことができます。

.. index:: return array, return string, array, string, array of strings, dynamic array, variably sized array, return struct, struct

Return Variables
----------------

ファンクションの返り値は ``returns`` キーワードの後、同じシンタックスで宣言されます。

例えば2つの結果が欲しい時: ファンクションパラメータとして渡された2つの整数の和と積が欲しい時、次の様に書けます::

    pragma solidity >=0.4.16 <0.6.0;

    contract Simple {
        function arithmetic(uint _a, uint _b)
            public
            pure
            returns (uint o_sum, uint o_product)
        {
            o_sum = _a + _b;
            o_product = _a * _b;
        }
    }

返り値の名前は省略可能です。返り値は他のローカル変数として使えます。返り値は :ref:`default value <default-value>` で初期化されており、明示的に値をセットしない限りその値を持ちます。

明示的に変数に値を割り当て、``return;`` を使うか、直接 ``return`` に返り値（1つもしくは :ref:`multiple ones<multi-return>` ）を入れるかいずれかが可能です::

    pragma solidity >=0.4.16 <0.6.0;

    contract Simple {
        function arithmetic(uint _a, uint _b)
            public
            pure
            returns (uint o_sum, uint o_product)
        {
            return (_a + _b, _a * _b);
        }
    }

これは返り値を変数に割り当てて ``return;`` を使って返す方法と結果は同じです。

.. note::
    internalでないファンクションからいくつかの型は返すことはできません。特に多次元動的配列と構造体です。ソースファイルに ``pragma experimental
    ABIEncoderV2;`` を加えて新しい実験的な ``ABIEncoderV2`` 機能を有効にすればもっと他の型も使える様になります。しかし、``mapping`` 型はそれでも1つのコントラクト内でしか扱うことができず、転送することはできません。

.. _multi-return:

Returning Multiple Values
-------------------------

ファンクションが複数の返り値の型を持つ時、``return (v0, v1, ..., vn)`` という宣言が複数の型を返すのに使用されます。
要素の数は返す値の数と同じでなければいけません。

.. index:: ! view function, function;view

.. _view-functions:

View Functions
==============

ステートを変えない場合ファンクションは ``view`` を宣言できます。

.. note::
  もしコンパイラのEVMターゲットかByzantiumより新しい場合、EVM実行の一部としてステートの変更をさせないopcode ``STATICCALL`` が ``view`` ファンクションに使われます。ライブラリの ``view`` ファンクションには ``DELEGATECALL`` が使用されます。なぜなら、``DELEGATECALL`` と ``STATICCALL`` が一緒になったものは存在しないからです。つまり、ライブラリの ``view`` ファンクションはステートの変更を妨げるランタイムチェックを持っていないということです。ライブラリのコードは普通はコンパイル時に既知であり、static checkerがコンパイル時にチェックするため、これはセキュリティ的に問題ないはずです。

下記のリストはステートを変更すると考えられます。

#. 状態変数を書く。
#. :ref:`イベントのemit <events>`。
#. :ref:`他のコントラクトを作る <creating-contracts>`。
#. ``selfdestruct`` を使う
#. callを通じてEtherを送る。
#. ``view`` や ``pure`` の付いていないファンクションを呼び出す。
#. 低レベルcallを使う。
#. あるopcodeの入ったインラインアセンブリを使う。

::

    pragma solidity ^0.5.0;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + now;
        }
    }

.. note::
  ファンクションで ``constant`` は ``view`` のエイリアスとして使われていましたが、バージョン0.5.0でドロップされました。

.. note::
  getterメソッドは自動的に ``view`` がつきます。

.. note::
  バージョン0.5.0以前ではコンパイラは ``view`` ファンクションに ``STATICCALL`` を使っていませんでした。
  これは、無効な明示的型変換を使うことで、``view`` ファンクションでのステートの変更を可能にしていました。
  ``STATICCALL`` を ``view`` ファンクションに使うことで、EVM上ではステートの変更を行うことができなくなりました。

.. index:: ! pure function, function;pure

.. _pure-functions:

Pure Functions
==============

何も読まない、ステートも変更しない場合、ファンクションで ``pure`` を宣言できます。

.. note::
  コンパイラのEVMターゲットがByzantium以降であれば、opcode ``STATICCALL`` が使えます。ステートが読まれないかは保証しませんが、少なくともステートが変更されていないことは保証されます。


ステートを変更する上記のリストに加えて、下記はステートを読み込むとされる処理のリストです。

#. 状態変数から読み込む。
#. ``address(this).balance`` もしくは ``<address>.balance`` にアクセスする。
#. ``block``、``tx``、``msg`` ( ``msg.sig`` と ``msg.data`` は除く)のどれかにアクセスする。
#. ``pure`` が付いていないファンクションを呼び出す。
#. あるopcodeの入ったインラインアセンブリを使う。

::

    pragma solidity ^0.5.0;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }

Pureファンクションは :ref:`エラーが起きた <assert-and-require>` 時に、潜在的なステートの変更を元に戻すため `revert()` と `require()` を使えます。

ステートを元に戻すのは"ステートの変更"とは見なされません。
なぜなら、``view`` もしくは ``pure`` がないコードで以前に作られたステートへの変更だけがrevertされていたためです。さらに、そのコードは ``revert`` をキャッチし、受け渡ししないオプションがあります。

この挙動は ``STATICCALL`` opcodeにも合っています。

.. warning::
  EVMレベルではファンクションがステートを読み込むことを止めることはできません。できるのは書き込みを止めることだけです（ ``view`` がEVMレベルで強制されますが、``pure`` はされません）。

.. note::
  バージョン0.5.0以前ではコンパイラは ``pure`` ファンクションに ``STATICCALL`` を使っていませんでした。
  これは、無効な明示的型変換を使うことで、``view`` ファンクションでのステートの変更を可能にしていました。
  ``STATICCALL`` を ``pure`` ファンクションに使うことで、EVM上ではステートの変更を行うことができなくなりました。

.. note::
  バージョン0.4.17以前では、コンパイラは ``pure`` にステートを読まなせないということを強制していませんでした。
  コンパイル時の型チェックで、コントラクト型間での無効な明示的変換を避けることができます。なぜなら、コンパイラがそのタイプのコントラクトはステートを変える操作をしないと証明するからです。ただ、ランタイム時に呼ばれるコントラクトに関しては実際にそのタイプかどうかはチェックしません。

.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fallback Function
=================

コントラクトは1つだけ名前の付いていないファンクションを持つことができます。そのファンクションは引数を持てず、何も返せません。そして可視性は ``external`` である必要があります。
もし、他のファンクションが与えられたファンクションの識別子になかった場合（もしくは何のデータも渡されなかった場合）、そのコントラクトが呼ばれた時に実行されます。

さらに、このファンクションはコントラクトが（データなしの）Etherを受け取った時は実行されます。Etherを受け取って、コントラクトのトータルバランスにそれを追加するにはフォールバックファンクションは ``payable`` でなければいけません。もしそのようなファンクションがない場合、コントラクトは通常のトランザクションを通じてEtherを受け取れず、例外を投げます。

最悪の場合、フォールバックファンクションは2300ガスだけを利用可能（例えば `send` か `transfer` を使うのに）とします。基本的なログの機能以外に他の演算のための余力をほとんど残しません。下記の演算は固定で2300ガス以上使う演算です。

- ストレージに書き込む
- コントラクトを作る
- ガスを多く使う外部ファンクションを呼び出す
- Etherを送る

他のファンクションのように、フォールバックファンクションは十分なガスが渡される限り、複雑な演算も実行可能です。

.. note::
    フォールバックファンクションは引数を持てませんが、呼び出しで供給されたペイロードを引き出すのに、``msg.data`` を使うことができます。

.. warning::
    呼び出し元が利用不可なファンクションを呼び出した時にもフォールバックファンクションは実行されます。Etherを受け取るためだけにフォールバックファンクションを実行したい場合、不正な呼び出しを防ぐために ``require(msg.data.length == 0)`` を追加した方が良いでしょう。

.. warning::
    Etherを直接受け取る（ファンクションコール、つまり ``send`` か ``transfer`` を伴わない）が、フォールバックファンクションを定義しないコントラクトは例外を投げ、Etherを送り返します（Solidityバージョン0.4.0以前では違いました）。そのため、コントラクトでEtherを受け取りたい場合、payableのフォールバックファンクションを実装しなければいけません。

.. warning::
    payableフォールバックファンクションがないコントラクトは `coinbase transaction` (もしくは `miner block reward`)として、もしくは ``selfdestruct`` の送り先としてEtherを受け取ることができます。

    コントラクトはそのようなEtherの送金に関して何の反応もできないため、それを拒否することもできません。
    これはEVMの設計であるため、Solidityではどうにもできません。

    これは、``address(this).balance`` がコントラクト内で処理された手動の会計処理の合計より高くなりうることを意味しています（フォールバックファンクションでアプデートされるカウンタを持っているということです）。

::

    pragma solidity ^0.5.0;

    contract Test {
        // This function is called for all messages sent to
        // this contract (there is no other function).
        // Sending Ether to this contract will cause an exception,
        // because the fallback function does not have the `payable`
        // modifier.
        function() external { x = 1; }
        uint x;
    }


    // This contract keeps all Ether sent to it with no way
    // to get it back.
    contract Sink {
        function() external payable { }
    }

    contract Caller {
        function callTest(Test test) public returns (bool) {
            (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            // results in test.x becoming == 1.

            // address(test) will not allow to call ``send`` directly, since ``test`` has no payable
            // fallback function. It has to be converted to the ``address payable`` type via an
            // intermediate conversion to ``uint160`` to even allow calling ``send`` on it.
            address payable testPayable = address(uint160(address(test)));

            // If someone sends ether to that contract,
            // the transfer will fail, i.e. this returns false here.
            return testPayable.send(2 ether);
        }
    }

.. index:: ! overload

.. _overload-function:

Function Overloading
====================

コントラクトは異なるパラメータを持つ、同じ名前のファンクションを複数持つことができます。
このプロセスは"オーバーロード"といわれ、継承されたファンクションにも適用されます。
コントラクト ``A`` のスコープに入っているファンクション ``f`` のオーバーロードの例を下記に示します。

::

    pragma solidity >=0.4.16 <0.6.0;

    contract A {
        function f(uint _in) public pure returns (uint out) {
            out = _in;
        }

        function f(uint _in, bool _really) public pure returns (uint out) {
            if (_really)
                out = _in;
        }
    }

オーバーロードされたファンクションは外部インターフェースの中にもあります。もし2つの外部的に可視なファンクションが外部としての型ではなく、Solidityの型として異なる場合エラーになります。

::

    pragma solidity >=0.4.16 <0.6.0;

    // This will not compile
    contract A {
        function f(B _in) public pure returns (B out) {
            out = _in;
        }

        function f(address _in) public pure returns (address out) {
            out = _in;
        }
    }

    contract B {
    }

両方の ``f`` ファンクションはABIとしてはアドレス型を受け入れてオーバーロードしますが、Solidity内では違うものとして考えられます。

Overload resolution and Argument matching
-----------------------------------------

オーバーロードされたファンクションは、現在のスコープ内のファンクションの宣言をファンクションコール内で渡された引数に合わせることによって選択されます。
全ての引数が暗示的に期待する型に変換できる場合、ファンクションはオーバーロードの候補として選ばれます。もし候補がなければ、そのresolutionは失敗します。

.. note::
    返ってくるパラメータはオーバーロードresolutionに考慮されません。

::

    pragma solidity >=0.4.16 <0.6.0;

    contract A {
        function f(uint8 _in) public pure returns (uint8 out) {
            out = _in;
        }

        function f(uint256 _in) public pure returns (uint256 out) {
            out = _in;
        }
    }

``50`` ``uint8`` と ``uint256`` どちらにも暗示的に変換できるため、``f(50)`` の呼び出しは型エラーを生成します。
一方で、``256`` は暗示的に ``uint8`` に変換できないため ``f(256)`` は ``f(uint256)`` オーバーロードします。
