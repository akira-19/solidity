**************************************
Units and Globally Available Variables
**************************************

.. index:: wei, finney, szabo, ether

Ether Units
===========

リテラルの数値はEhterの貨幣の単位として ``wei``、``finney``、``szabo`` もしくは ``ether`` を接尾辞として取れます。接尾辞がないEtherはWeiであるとみなされます。

::

    assert(1 wei == 1);
    assert(1 szabo == 1e12);
    assert(1 finney == 1e15);
    assert(1 ether == 1e18);

貨幣の単位を表す接尾辞は10の累乗を乗ずるだけです。

.. index:: time, seconds, minutes, hours, days, weeks, years

Time Units
==========

リテラルの数値の後の ``seconds``、``minutes``、``hours``、``days``、``weeks`` の様な接尾辞は時間の単位を指定するのに使用されます。秒を基準として単位は単純に下記の様に決められます。

 * ``1 == 1 seconds``
 * ``1 minutes == 60 seconds``
 * ``1 hours == 60 minutes``
 * ``1 days == 24 hours``
 * ``1 weeks == 7 days``

これらの単位を使ってカレンダーの計算をするのであれば気を付けてください。`うるう秒 <https://en.wikipedia.org/wiki/Leap_second>`_ のせいで、1年は毎年365日というわけではなく、1日も必ずしも24時間ではありません。うるう秒は予想できないので、カレンダーライブラリは外部のオラクルによってアップデートされる必要があります。

.. note::
    上記の理由により ``years`` という接尾辞はバージョン0.5.0で削除されました。

これらの接尾辞は変数には使えません。例えばもしファンクションのパラメータをdaysにしたい場合には下記の方法で変換できます::

    function f(uint start, uint daysAfter) public {
        if (now >= start + daysAfter * 1 days) {
          // ...
        }
    }

Special Variables and Functions
===============================

グローバルな名前空間にあり、主にブロックチェーンに関する情報や一般的な用途で使うユーティリティのファンクションを提供する特別な変数とファンクションがあります。

.. index:: abi, block, coinbase, difficulty, encode, number, block;number, timestamp, block;timestamp, msg, data, gas, sender, value, now, gas price, origin


Block and Transaction Properties
--------------------------------

- ``blockhash(uint blockNumber) returns (bytes32)``: 与えられたブロックのハッシュ - 現在のブロックを除いた直近256個のブロックのみで有効
- ``block.coinbase`` (``address payable``): 現在のブロックのマイナーのアドレス
- ``block.difficulty`` (``uint``): 現在のブロックのdifficulty
- ``block.gaslimit`` (``uint``): 現在のブロックのガスリミット
- ``block.number`` (``uint``): 現在のブロックナンバー
- ``block.timestamp`` (``uint``): 現在のunixのタイムスタンプ（秒）
- ``gasleft() returns (uint256)``: 残ガス
- ``msg.data`` (``bytes calldata``): コールデータ
- ``msg.sender`` (``address payable``): メッセージの送信者
- ``msg.sig`` (``bytes4``): コールデータの始め4byte（例：ファンクションの識別子）
- ``msg.value`` (``uint``): メッセージと一緒に送られたweiの量
- ``now`` (``uint``): 現在のブロックのタイムスタンプ ( ``block.timestamp`` のエイリアス)
- ``tx.gasprice`` (``uint``): トランザクションのガスプライス
- ``tx.origin`` (``address payable``): トランザクションの送信者 (フルコールチェーン)

.. note::
    ``msg.sender`` と ``msg.value`` を含んだ ``msg`` の値は **external** のファンクションのコールごとに変えることができます。これはライブラリのファンクションでも同様です。

.. note::
    自分のコードで何をしているか把握していない限り、``block.timestamp``、``now`` と ``blockhash`` を乱数のソースとして信用しないでください。

    タイムスタンプとブロックハッシュはある程度マイナーによって影響されます。悪意を持ったマイナーは例えばあるハッシュでカジノの支払いファンクションを呼び出し、もしお金を受け取れなかったら、また別のハッシュでそのファンクションを呼び出すことができます。

    現在のブロックのタイムスタンプは最後のブロックより確実に大きい必要がありますが、保証されているのはタイムスタンプは2つの連続する標準ブロックの間であるということだけです。

.. note::
    ブロックハッシュはスケーラビリティの理由から全てのブロックに対して利用可能という訳ではありません。最新256ブロックのハッシュにのみアクセス可能で、それ以前の値はゼロになります。

.. note::
    ``blockhash`` ファンクションは以前は ``block.blockhash`` でしたが、バージョン0.4.22で非推奨になり、バージョン0.5.0で削除されました。

.. note::
    ``gasleft`` ファンクションは以前は ``msg.gas`` でしたが、バージョン0.4.21で非推奨になり、バージョン0.5.0で削除されました。

.. index:: abi, encoding, packed

ABI Encoding and Decoding Functions
-----------------------------------

- ``abi.decode(bytes memory encodedData, (...)) returns (...)``: 与えられたデータをABIデコードする。第二引数として型を括弧付きで与えます。例: ``(uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))``
- ``abi.encode(...) returns (bytes memory)``: 引数をABIエンコードします。
- ``abi.encodePacked(...) returns (bytes memory)``: 与えられた引数で :ref:`packed encoding <abi_packed_mode>` を行います。
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory)``: 与えられた引数を二番目からABIエンコードし、その前に与えられた4バイトのセレクタを追加します。
- ``abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)``: ``abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), ...)``` と同じです。

.. note::
    これらのエンコーディングのファンクションは実際に外部のファンクションを呼ぶことなく外部のファンクション用のデータを作るために使われます。さらに、``keccak256(abi.encodePacked(a, b))`` は構造化されたデータのハッシュを計算する方法です。（異なるファンクションのパラメータの型を使ってもハッシュ衝突を起こす可能性があることに気をつけてください。）

エンコーディングの詳細は公式ドキュメントの :ref:`ABI <ABI>` と
:ref:`tightly packed encoding <abi_packed_mode>` を参照ください。

.. index:: assert, revert, require

Error Handling
--------------

エラーハンドリングに対する細かな詳細と、どのファンクションをいつ使うに関しては :ref:`assert  require<assert-and-require>` にあるそれらに特化したセクションを参照ください。

``assert(bool condition)``:
    条件を満たしていないと、invalid opcodeを発生させ、その結果state change reversionが起きます - 内部エラーに使用されます。
``require(bool condition)``:
    条件を満たしていないと、revertします - 入力か外部要素に対してのエラーに使用されます。
``require(bool condition, string memory message)``:
    条件を満たしていないと、revertします - 入力か外部要素に対してのエラーに使用されます。加えてエラーメッセージも出力されます。
``revert()``:
    実行を中断し、stateの変化を元に戻します。
``revert(string memory reason)``:
    説明付きで実行を中断し、stateの変化を元に戻します。

.. index:: keccak256, ripemd160, sha256, ecrecover, addmod, mulmod, cryptography,

Mathematical and Cryptographic Functions
----------------------------------------

``addmod(uint x, uint y, uint k) returns (uint)``:
    任意の精度で ``(x + y) % k`` の加算を行い、``2**256`` でラップアラウンドしません。バージョン0.5.0からは ``k != 0`` のアサーションを行います。
``mulmod(uint x, uint y, uint k) returns (uint)``:
    任意の精度で ``(x * y) % k`` の加算を行い、``2**256`` でラップアラウンドしません。バージョン0.5.0からは ``k != 0`` のアサーションを行います。
``keccak256(bytes memory) returns (bytes32)``:
    入力に対してKeccak-256のハッシュを計算します。
``sha256(bytes memory) returns (bytes32)``:
    入力に対してSHA-256のハッシュを計算します。
``ripemd160(bytes memory) returns (bytes20)``:
    入力に対してRIPEMD-160のハッシュを計算します。
``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``:
    楕円曲線の署名から公開鍵に関連したアドレスを復元する、もしくはエラーでゼロを返します。(`使用例 <https://ethereum.stackexchange.com/q/1777/222>`_)

.. note::
   ``ecrecover`` は ``address`` を返し、``address
   payable`` は返しません。復元されたアドレスで送金を行いたい場合には、変換するために :ref:`address payable<address>` を参照ください。

*プライベートなブロックチェーン* 上では ``sha256``、``ripemd160`` もしくは ``ecrecover`` でガス不足になるかもしれません。理由としては、これらはいわゆるプレコンパイルされたコントラクトとして実行され、そのコントラクトが本当に存在するのは、最初のメッセージを受け取った後だからです（そのコントラクトはハードコードですが）。存在しないコントラクトへのメッセージは高価なため、ガス不足になります。この問題の回避策としては例えば実際のコントラクトでそのファンクションを使う前に最初に1Weiをそのコントラクトに送ることです。メインネットやテストネットではこの問題は起こりません。

.. note::
    ``sha3`` と呼ばれる ``keccak256`` のエイリアスがありましたが、バージョン0.5.0で削除されました。

.. index:: balance, send, transfer, call, callcode, delegatecall, staticcall
.. _address_related:

Members of Address Types
------------------------

``<address>.balance`` (``uint256``):
    :ref:`address` のバランス（Wei）
``<address payable>.transfer(uint256 amount)``:
    与えられたWeiを :ref:`address` に送ります。失敗するとリバートし、固定で2300ガスを送ります。 （変更不可です。）
``<address payable>.send(uint256 amount) returns (bool)``:
    与えられたWeiを :ref:`address` に送ります。失敗すると ``false`` を返し、固定で2300ガスを送ります。 （変更不可です。）
``<address>.call(bytes memory) returns (bool, bytes memory)``:
    低レベルの ``CALL`` を、与えられたペイロードと共に発行し、成否とデータを返し、使用可能なガスを全て送ります。（変更可能です。）
``<address>.delegatecall(bytes memory) returns (bool, bytes memory)``:
    低レベルの ``DELEGATECALL`` を、与えられたペイロードと共に発行し、成否とデータを返し、使用可能なガスを全て送ります。（変更可能です。）
``<address>.staticcall(bytes memory) returns (bool, bytes memory)``:
    低レベルの ``STATICCALL`` を、与えられたペイロードと共に発行し、成否とデータを返し、使用可能なガスを全て送ります。（変更可能です。）

詳細は :ref:`address` を参照ください。

.. warning::
    タイプチェックやファンクションの存在チェック、引数のパッキングをバイパスするため、他のコントラクトのファンクションを実行する際には極力 ``.call()`` の使用を避けてください。

.. warning::
    ``send`` を使うことにはいくつかの危険があります: コールスタックの深さが1024で送金は失敗します（これは呼び出し元によっていつも強制されます）。そして、受領者のガスが不足した際にも送金は失敗します。そのため、安全にEtherを送るために、常に ``send`` の返り値を確認する、もしくは ``transfer`` を使用してください。もっと良い手段は受領者がお金を引き出す時のパターンを使用することです。

.. note::
   バージョン0.5.0以前では、例えば ``this.balance`` の様にコントラクトインスタンスからアドレス型のメンバにアクセス可能でした。現在ではこれは禁止されており、明示的にアドレス型に変換する必要があります：``address(this).balance``。

.. note::
   もし状態変数が低レベルdelegatecallを通じてアクセスされた場合、呼び出されたコントラクトが呼び出したコントラクトのストレージ変数に名前で正しくアクセスできる様に2つのコントラクトのストレージの配置は揃ってなければいけません。
   これはもちろんストレージのポインタがファンクションの引数で渡される場合ではなく、高レベルのライブラリの場合です。

.. note::
    バージョン0.5.0以前では, ``.call``、``.delegatecall``、``.staticcall`` は成否だけ返し、データは返しません。

.. note::
    バージョン0.5.0以前では, ``callcode`` と呼ばれる ``delegatecall`` に似ていますが、微妙に異なるメンバがあります。


.. index:: this, selfdestruct

Contract Related
----------------

``this`` (現在のコントラクトの型):
    現在のコントラクト、明示的に :ref:`address` と変換可能です。

``selfdestruct(address payable recipient)``:
    現在のコントラクトを破壊し、与えられた :ref:`address` にファンドを送ります。

さらに、現在のコントラクトの全てのファンクションは現在のファンクションを含めて直接呼ぶことができます。

.. note::
    バージョン0.5.0以前では、``selfdestruct`` と同じ意味の ``suicide`` というファンクションがあります。

.. index:: type, creationCode, runtimeCode

.. _meta-type:

Type Information
----------------

``type(X)`` という表現は ``X`` 型についての情報を引き出すのに使用可能です。現在、この機能について限定的なサポートしかありませんが、将来拡張されるかもしれません。以下のプロパティはコントラクト型 ``C`` で使用可能です。


``type(C).creationCode``:
    コントラクトのバイトコードの生成を含んでいるメモリーバイト配列
    カスタムクリエーションルーティンを作るためにインラインアッセンブリで使用できます（特に ``create2`` opcodeを使って）。
    このプロパティはコントラクト自体、もしくは継承元のコントラクトからは呼び出すことができません。そのため、呼び出し元のバイトコードにこのバイトコードが含まれ、その結果循環参照が不可能になります。

``type(C).runtimeCode``:
    コントラクトのランタイムバイトコードを含んでいるメモリーバイト配列
    これは通常、``C`` のコンストラクタによってデプロイされるコードです。もし``C``がインラインアセンブリを使うコンストラクタを持っていたら、実際のデプロイされるバイトコードとは異なるかもしれません。通常の呼び出しから保護するために、デプロイ時にライブラリがランタイムバイトコードを修正するということを覚えておいてください。
    同じ制限が ``.creationCode`` と同様にこのプロパティに適用されます。
