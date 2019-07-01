.. index:: ! value type, ! type;value
.. _value-types:

Value Types
===========

次の型も常に値として渡されるため値型と呼ばれています。引数や割り当て時にはこれらは常にコピーされます。

.. index:: ! bool, ! true, ! false

Booleans
--------

``bool``: 取りうる値は ``true`` と ``false``

Operators:

*  ``!`` (論理否定)
*  ``&&`` (論理積, "and")
*  ``||`` (論理和, "or")
*  ``==`` (等価)
*  ``!=`` (不等価)

``||`` と ``&&`` 演算子は一般的な短絡評価のルールに従います。つまり、``f(x) || g(y)`` という表現において、もし ``f(x)`` が``true`` と評価された場合、たとえ副次的な作用があったとしても ``g(y)`` は評価されません。

.. index:: ! uint, ! int, ! integer

Integers
--------

``int`` / ``uint``: 色々なサイズの符号付と符号なし整数です。``uint8`` から ``uint256`` まで8ずつ（符号なしの8から256ビットまで）と``int8`` から ``int256`` まで上がっていきます。``uint`` と ``int`` はそれぞれ ``uint256`` と ``int256`` のエイリアスです。

Operators:

* 比較: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (``bool`` で評価)
* ビット演算子: ``&``, ``|``, ``^`` (ビット排他論理和), ``~`` (ビット否定)
* シフト演算子: ``<<`` (左シフト), ``>>`` (右シフト)
* 算術演算子: ``+``, ``-``, unary ``-``, ``*``, ``/``, ``%`` (modulo), ``**`` (累乗)

.. warning::

  Solidityの整数はある範囲に制限されています。例えば、``uint32`` であれば ``0`` から最大 ``2**32 - 1`` までです。
  もし計算結果がこの範囲に収まらない場合には、切り捨てられます。この切り捨てによって起こる結果は :ref:`be 知っておくべきです<underflow-overflow>` 。

Comparisons
^^^^^^^^^^^

比較の値は整数値を比較することによって得られます。

Bit operations
^^^^^^^^^^^^^^

ビット演算子の計算は2の補数表現で行われます。例えば、 ``~int256(0) == int256(-1)`` です。

Shifts
^^^^^^

シフト演算の結果は左オペランドの型となります。``x << y`` という表現は ``x * 2**y`` と等価です。さらに正の整数に関しては ``x >> y`` と ``x / 2**y`` が等価です。負の ``x`` に対して ``x >> y`` は、``2`` のべき乗で除し、切り捨てしたもの（負の無限大で）と等価です。
負の数でシフトするとランタイム例外が投げられます。


.. warning::
    バージョン``0.5.0``の前までは、負の ``x`` に対して右シフト ``x >> y`` は ``x / 2**y`` と等価でした。例えば右シフトは負の無限大での切り捨ての代わりに、0の位での切り捨てを行なっていました。

Addition, Subtraction and Multiplication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

加算、減算、乗算は通常のセマンティクスです。
これらは2の補数表現で使用されます。つまり、例えば ``uint256(0) - uint256(1) == 2**256 - 1`` です。安全なスマートコントラクトを作成するときには、このオーバーフローを考慮する必要があります。

``T`` が ``x`` の型であるとき ``-x`` という表現は ``(T(0) - x)`` と等価です。つまり、``x`` の型が符号なし整数のときに ``-x`` は負の数にはなりません。また、``x`` が負の数であれば ``-x`` は正の数になりえます。さらに、別の2の補数表現による注意があります::

    int x = -2**255;
    assert(-x == x);

これが意味するのは、たとえある数字が負の数でも、それにマイナスをつけたものが正の数になるとは限らないということです。


Division
^^^^^^^^

ある演算の出力の型は常に演算対象の型と同じなので、整数の除算の結果は整数になります。Solidityでは、除算は1の位までの概算になります。つまり、``int256(-5) / int256(2) == int256(-2)`` となります。

一方で、:ref:`リテラル<rational_literals>` での除算は任意の精度での少数値が結果として出力されるということに注意して下さい。

.. note::
  ゼロでの除算はフェイルアサーションが発生します。

Modulo
^^^^^^

剰余演算 ``a % n`` は ``a`` を ``n`` で割ったときの 余り ``r`` を結果として返します（``q = int(a / n)`` で ``r = a - (n * q)`` です）。つまり、剰余演算の答えは左オペランドと同じ符号（もしくはゼロ）で、負の数 ``a`` に対して ``a % n == -(abs(a) % n)`` となります:

 * ``int256(5) % int256(2) == int256(1)``
 * ``int256(5) % int256(-2) == int256(1)``
 * ``int256(-5) % int256(2) == int256(-1)``
 * ``int256(-5) % int256(-2) == int256(-1)``

.. note::
  0での剰余演算はフェイルアサーションが発生します。

Exponentiation
^^^^^^^^^^^^^^

指数演算は符号なしの型でのみ使用可能です。使っている型が指数演算の結果を包括するのに、また将来的に起こりうるラッピングに対して十分な大きさであることを確認してください。

.. note::
  EVMでは ``0**0`` は ``1`` と定義されています。

.. index:: ! ufixed, ! fixed, ! fixed point number

Fixed Point Numbers
-------------------

.. warning::
    固定小数点はまだSolidityでは完全にサポートされていません。宣言はできますが、値を割り当てたりはできません。

``fixed`` / ``ufixed``: いくつかのサイズがある符号付、符号なし固定小数点です。``ufixedMxN`` と ``fixedMxN`` で ``M`` はその型で取れるビットの数で、``N`` は、何桁の10進数少数点が取れるかを表しています。``M`` は8で割り切れる数で、8から256ビットまでとれます。``N`` は0から80までとることができます。``ufixed`` と ``fixed`` はそれぞれ ``ufixed128x18`` と ``fixed128x18`` のエイリアスです。

Operators:

* Comparisons: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (evaluate to ``bool``)
* Arithmetic operators: ``+``, ``-``, unary ``-``, ``*``, ``/``, ``%`` (modulo)

.. note::
    浮動小数と固定小数の主な違いですが、前者では整数部分と小数部分（多くの言語では ``float`` と ``double`` です。より詳細な情報はIEEE 754で確認してください）の桁数がフレキシブルで、後者では厳密に決められていることです。一般的に、浮動小数点数はほぼ全てのスペースをその数を表すのに使用し、少しのビットで小数点の長さを表します。

.. index:: address, balance, send, call, callcode, delegatecall, staticcall, transfer

.. _address:

Address
-------

アドレス型は広義的には同じである2つの種類があります:

 - ``address``: 20バイトの値 (Ethereumアドレスの大きさ)です。
 - ``address payable``: ``address`` と同じですが、追加のメンバ ``transfer`` と ``send`` が使えます。

この特徴が意味するのは、``address payable`` にはEtherを送ることができますが、ただの ``address`` にはできません。


Type conversions:

``address payable`` から ``address`` への暗黙的な変換は可能ですが、``address`` から ``address payable`` にはできません（唯一 ``uint160`` を中継することで変換可能です）。

:ref:`アドレスリテラル<address_literals>` は暗黙的に ``address payable`` に変換可能です。

``address`` への、もしくは ``address`` からの明示的な変換は整数、整数リテラル、``bytes20``、コントラクト型で可能ですが下記の注意事項を参照ください:
``address payable(x)`` という形での変換はできません。代わりに、``address(x)`` という変換結果が ``address payable`` もしくは、もし ``x`` が整数もしくは固定サイズのバイト型であれば、リテラルかpayableのフォールバックファンクションを持つコントラクトになります。
もし ``x`` がpayableのフォールバックファンクションを持たないコントラクトであれば、``address(x)`` は ``address`` になります。
外部のファンクションの署名では、``address`` は ``address`` 型、``address payable`` 型両方で使用されます。

.. note::
    おそらく、``address``
    と ``address payable`` の違いを気にする必要はあまりなく、どこでも``address`` を使うことになるでしょう。例えば、もし、:ref:`withdrawal pattern<withdrawal_pattern>` を使うと、``address payable`` である ``msg.sender`` で ``transfer`` を使うので、``address`` としてアドレスを保存できます（するべきです）。

Operators:

* ``<=``, ``<``, ``==``, ``!=``, ``>=`` and ``>``

.. warning::
    もし``address`` 型よりも大きなサイズの型、例えば、``bytes32``、から変換しようとすると、``address`` は切り詰められます。
    変換時の曖昧さを減らすためにバージョン0.4.24からは変換時にコンパイラは切り捨てを明示的にすることを要求します。例えば、``0x111122223333444455556666777788889999AAAABBBBCCCCDDDDEEEEFFFFCCCC``　です。

    ``0x111122223333444455556666777788889999aAaa`` となる ``address(uint160(bytes20(b)))`` もしくは、``0x777788889999AaAAbBbbCcccddDdeeeEfFFfCcCc`` となる ``address(uint160(uint256(b)))`` が使えます。

.. note::
    ``address`` と ``address payable`` の違いはバージョン0.5.0で導入されました。
    また、コントラクトはアドレス型からは生成されませんが、もしpayableのフォールバックファンクションを持っていれば、``address`` もしくは ``address payable`` に明示的に変換できるという機能がバージョン0.5.0から導入されました。

.. _members-of-addresses:

Members of Addresses
^^^^^^^^^^^^^^^^^^^^

アドレス型の全てのメンバのクイックリファレンスは  :ref:`address_related` を参照ください。

* ``balance`` and ``transfer``

``balance`` プロパティであるアドレスのバランスを確認できます。また、``transfer`` でpayableなアドレスにEther（単位はwei）を送ることができます:

::

    address payable x = address(0x123);
    address myAddress = address(this);
    if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);

現在のコントラクトのバランスが十分大きくないか、受け取り側のアカウントでEtherの送金が拒否された場合、``transfer``ファンクションは失敗し、送金前の状態に戻ります。

.. note::
    もし ``x`` がコントラクトアドレスだった場合、そのコード（具体的には、もしあれば :ref:`fallback-function`）は ``transfer`` と一緒に実行されます（これはEVMの特徴で、止めることはできません）。もしこの実行時にガス不足になったり、他の理由でフェイルした場合は、Etherの送金はキャンセル、元の状態に戻り、現在のコントラクトは例外と共にストップします。

* ``send``

Sendは ``transfer`` の低レベルバージョンです。もし実行が失敗したら、現在のコントラクトは例外と共にストップしない代わりに、``send`` が ``false`` を返します。

.. warning::
    ``send`` を使うといくつかの危険が伴います:
    送金はコールスタックの深さが1024でフェイルします（これは常に呼び出し元によって行われます）。そして、もし受領者がガスを使い切ってもフェイルします。そのため、安全なEtherの送金のために、常に ``send`` の返り値を確認する、もしくは ``transfer`` を使ってください。さらに良いのは:
    受領者がお金を引き出す時の様式を使用することです。


* ``call``, ``delegatecall`` and ``staticcall``

ABIに従わないコントラクトと繋げるために、もしくはエンコードに対してもっと直接的なコントロールを得るために、``call``、``delegatecall``、``staticcall`` ファンクションを使うことができます。
これらは全て一つの ``bytes memory`` パラメータを引数とし、（``bool`` で）成否と ``bytes memory`` の返ってきたデータを返します。
``abi.encode``、``abi.encodePacked``、``abi.encodeWithSelector``、``abi.encodeWithSignature`` は体系的なデータをエンコードするために使用することができます。

Example::

    bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
    (bool success, bytes memory returnData) = address(nameReg).call(payload);
    require(success);

.. warning::
    これら全てのファンクションは低級のファンクションで、使う際には注意が必要です。特に、未知のコントラクトは悪意を持っている可能性があり、もしそのコントラクトを呼び出すと、あなたのコントラクトに次々とコールバックを投げるコントラクトにコントロールを渡してしまうかもしれません。そのため、その呼び出しが返ってきたときに、状態変数の変化に対して準備をしておいてください。他のコントラクトと繋がる一般的な方法はコントラクトオブジェクト(``x.f()``)上でファンクションを呼び出すことです。

.. note::
    以前のバージョンではこれらのファンクションで任意の引数を取ることができ、``bytes4`` 型の第一引数を異なる方法で処理することができました。そのようなエッジケースはバージョン0.5.0で削除されました。

供給されたガスを``.gas()``
It is possible to adjust the supplied gas with the ``.gas()`` modifierで調整することができます::

    address(nameReg).call.gas(1000000)(abi.encodeWithSignature("register(string)", "MyName"));

同様に、供給されたEtherの値もコントロールすることができます::

    address(nameReg).call.value(1 ether)(abi.encodeWithSignature("register(string)", "MyName"));

最後に、これらのmodifierは結合させることができます。順番は関係ありません::

    address(nameReg).call.gas(1000000).value(1 ether)(abi.encodeWithSignature("register(string)", "MyName"));

似たような方法で、``delegatecall`` は使用されます: 違いは与えられたアドレスのコードだけ使われ、他の要素（storage、balance等）は現在のコントラクトから使われます。``delegatecall`` の目的は別のコントラクトに保存されているライブラリを使用することです。ユーザはstorageの構造がどちらのコントラクトでもdelegatecallを使用するのに適切であることを確認しなければいけません。

.. note::
    homestead以前は、``callcode`` という制限のある変形型のみが使用可能でしたが、オリジナルの ``msg.sender`` と ``msg.value`` にアクセス不可でした。このファンクションはバージョン0.5.0で削除されました。

Byzantiumから ``staticcall`` も使うことができます。基本的には ``call`` と同じですが、もし呼ばれたファンクションがステートを変更したらリバートします。

``call``、``delegatecall``、``staticcall`` の3つのファンクションは全てとても低級のファンクションで、Solidityの型安全性を破るため、*最終手段* として使用してください。

``.gas()`` オプションは全てのメソッドで使用可能ですが、``.value()`` は ``delegatecall`` ではサポートされません。

.. note::
    全てのコントラクトは ``address`` に変換できるため、``address(this).balance`` で現在のコントラクトにそのバランスをクエリすることができます。

.. index:: ! contract type, ! type; contract

.. _contract_types:

Contract Types
--------------

全ての :ref:`contract<contracts>` は自分自身の型を定義します。
あるコントラクトからそのコントラクトが継承しているコントラクトへ暗黙的に変換することができます。
コントラクトは他のコントラクト型から、もしくは他のコントラクト型へ明示的に変換することができます。さらに ``address`` 型への変換も可能です。

コントラクト型がpayableのフォールバックファンクションを持っている時のみ、``address payable`` 型から、もしくは ``address payable`` 型への明示的な変換が可能です。
その変換は ``address(x)`` では行われますが、``address payable(x)`` では行われません。詳細な情報は　:ref:`address type<address>` を参照ください。

.. note::
    バージョン0.5.0以前では、コントラクトは直接アドレス型から得られており、``address`` と ``address payable`` に違いはありませんでした。

もしコントラクト型（`MyContract c`）のローカル変数を宣言した場合、そのコントラクト上でファンクションを呼び出すことができます。同じコントラクト型からその変数を割り当てる様にして下さい。

コントラクトのインスタンスも作成可能です（つまり新しくそのコントラクトが作られるということです）。詳細は :ref:`'Contracts via new'<creating-contracts>` を参照ください。

コントラクトのデータ表現は ``address`` 型のデータ表現と同じで、この型は :ref:`ABI<ABI>` でも使われています。

コントラクト型はどんな演算子もサポートしません。

コントラクト型のメンバはpublicの状態変数を含んだそのコントラクトのexternalのファンクションです。

あるコントラクト ``C`` に対して、そのコントラクトの :ref:`type information<meta-type>` にアクセスするために ``type(C)`` を使うことができます。

.. index:: byte array, bytes32

Fixed-size byte arrays
----------------------

値型である``bytes1``、``bytes2``、``bytes3`` ... ``bytes32`` は1から32までバイト列を保持しています。
``byte`` は ``byte1`` のエイリアスです。

Operators:

* Comparisons: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (``bool`` を返します)
* Bit operators: ``&``, ``|``, ``^`` (bitwise exclusive or), ``~`` (bitwise negation)
* Shift operators: ``<<`` (left shift), ``>>`` (right shift)
* Index access: If ``x`` is of type ``bytesI``, then ``x[k]`` for ``0 <= k < I`` returns the ``k`` th byte (read-only).

* 比較: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (``bool`` で評価)
* ビット演算子: ``&``, ``|``, ``^`` (ビット排他論理和), ``~`` (ビット否定)
* シフト演算子: ``<<`` (左シフト), ``>>`` (右シフト)
* インデックスアクセス: もし ``x`` が ``bytesI`` 型なら ``0 <= k < I`` の元で ``x[k]`` は ``k`` 番目のバイトを返します（読み取り専用）。

どれだけのビット数をシフトさせるか決める右オペランドがどの整数型でもシフト演算子は動作します（結果は左オペランドの型で返ります）。
負の数でのシフトはランタイムの例外を生成します。

Members:

* ``.length`` は固定長さのバイト配列を返します（読み取り専用）。

.. note::
    ``byte[]`` はバイトの配列ですがパディングのため、各要素の間の31バイトを無駄にしています（storage以外）。代わりに、``bytes`` を使用する方が良いでしょう。

Dynamically-sized byte array
----------------------------

``bytes``:
    動的サイズのバイト配列です。:ref:`arrays` を参照ください。値型ではありません。
``string``:
    動的サイズのUTF-8でエンコードされた文字列です。:ref:`arrays` を参照ください。値型ではありません。

.. index:: address, literal;address

.. _address_literals:

Address Literals
----------------

アドレスのチェックサムをパスする16進数のリテラル、例えば ``0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF`` は ``address payable`` 型です。
39から41文字で、チェックサムにパスしない16進数リテラルは警告を発し、通常の有理数リテラルとして扱われます。

.. note::
    The mixed-case address checksum format is defined in `EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_.

.. index:: literal, literal;rational

.. _rational_literals:

Rational and Integer Literals
-----------------------------

整数リテラルは0から9までの数字から形成され、10進数として認識されます。例えば、``69`` は六十九のことです。

10進数の小数リテラルは ``.`` を使って形成され、少なくとも1文字片側に数字があります。例えば、``1.``、``.1``、``1.3`` です。

指数表記もサポートされています。基数は小数を取れますが、指数はできません。
例えば、``2e10``、``-2e10``、``2e-10``、``2.5e1``です。

アンダースコアは数字リテラルの読みやすさを改善するために数字を分けるのに使用することができます。
例えば、10進数 ``123_000``、16進数の ``0x2eff_abde``、10進数の指数表記 ``1_2e345_678`` は全て有効です。
アンダースコアは2つの数字の間でのみ有効で、1つのアンダースコアしか使うことができません（2つ連続でアンダースコアを使うことはできません）。セマンティクス的な意味は何もありません。アンダースコアを含む数字リテラルでアンダースコアは無視されます。

数字リテラルは非リテラル型に（例えば非リテラル型と一緒に使うか、明示的な変換によって）変換されるまで任意の精度を持ちます。
つまり数字リテラル表現では、計算してもオーバーフローしませんし、除算では切り捨ては起きません。

例えば、中間の計算結果は機械語のサイズに収まっていませんが、``(2**800 + 1) - 2**800`` の結果は（``uint8`` 型の）定数 ``1`` になります。さらに（非整数が使われていますが）``.5 * 8`` の結果は整数 ``4`` になります。

計算対象が整数で有る限り、整数に使える演算子は全て数字リテラルで使用することができます。
もし片方でも小数を含んでいた場合、ビット演算子は使うことはできません。また、指数部分に小数は使えません（結果が非有理数になる可能性があるため）。

.. note::
    Solidityは各有理数に対して数字リテラル型が使えます。整数リテラルと有理数リテラルは数字リテラル型に属します。
    さらに、全ての数字リテラル表現（数字リテラルと演算子のみを含む表現）は数字リテラル型に属します。そのため、数字リテラル表現の ``1 + 2`` と ``2 + 1`` の結果である有理数の3は両方とも同じ数字リテラル型に属します。

.. warning::
    バージョン0.4.0以前では整数リテラルの除算の結果は切り捨てされていましたが、現在は有理数に変換されます。例えば、``5 / 2`` は ``2`` ではなく、``2.5`` です。

.. note::
    数字リテラル表現は非数字リテラル表現が使われたタイミングで非数字リテラル型に変換されます。
    型を無視し、下記の ``b`` に割り当てられている式の値は整数となります。``a`` は ``uint128`` 型であるため、``2.5 + a`` はある適切な型を持っていなければいけませんが、``2.5`` の型と ``uint128`` 型に共通した型が存在しないため、Solidityのコンパイラはこのコードを処理しません。

::

    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5;

.. index:: literal, literal;string, string
.. _string_literals:

String Literals and Types
-------------------------

文字列リテラルはダブルもしくはシングルクオテーション(``"foo"`` もしくは ``'bar'``)で記述されます。これらはC言語の様な後置ゼロにはなりません。``"foo"`` は3バイトを表します。4バイトではありません。整数リテラルは複数の型をとりうりますが、``bytes1`` ... ``bytes32`` に暗黙的に変換可能です。もしサイズが合えば``bytes`` と ``string`` にも変換可能です。

例えば、``bytes32 samevar = "stringliteral"`` では、文字列リテラルは ``bytes32`` 型に割り当てられる時に、その生のバイト構造で解釈されます。

文字列リテラルは以下のエスケープキャラクターをサポートします:

 - ``\<newline>`` (escapes an actual newline)
 - ``\\`` (backslash)
 - ``\'`` (single quote)
 - ``\"`` (double quote)
 - ``\b`` (backspace)
 - ``\f`` (form feed)
 - ``\n`` (newline)
 - ``\r`` (carriage return)
 - ``\t`` (tab)
 - ``\v`` (vertical tab)
 - ``\xNN`` (hex escape, see below)
 - ``\uNNNN`` (unicode escape, see below)

``\xNN`` は16進数をとり、適切なバイトを挿入します。一方で、``\uNNNN`` はUnicodeのコードポイントをとり、UTF-8のシーケンスを挿入します。

以下の例の中の文字列は10バイトの長さを持ちます。
新しい行で始まり、ダブルクオート、シングルクオート、バックスラッシュと続き、（区切りなく）``abcdef`` という文字が続きます。

::

    "\n\"\'\\abc\
    def"

改行コード(例えばLF、VF、FF、CR、NEL、LS、PS)でないどんなユニコードのラインターミネータは文字列リテラルを終了させると考えられています。もし ``\`` で処理されていないのであれば、改行は文字列リテラルを終了させるだけです。

.. index:: literal, bytes

Hexadecimal Literals
--------------------

16進数リテラルは ``hex`` という接頭辞をつけて、ダブルかシングルクオートで囲まれます（``hex"001122FF"``）。中身は16進数の文字列でなければなりません。そしてその値は2進数表現になります。

16進数リテラルは :ref:`string literals <string_literals>` の様に振る舞い、変換に関して同じ制限を持っています。

.. index:: enum

.. _enums:

Enums
-----

列挙型はSolidityでのユーザー定義型の1つです。明示的に整数型から、もしくは整数型に変換可能ですが、暗黙的には変換できません。整数型からの明示的な変換では実行時にその値が列挙型の範囲内に収まっているかチェックし、範囲外である場合にはフェイルアサーションを起こします。
列挙型は少なくとも1つの要素が必要です。

そのデータ表現はC言語の列挙型と同じです。オプションは ``0`` で始まる符号なし整数値によって表されます。


::

    pragma solidity >=0.4.16 <0.6.0;

    contract test {
        enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
        ActionChoices choice;
        ActionChoices constant defaultChoice = ActionChoices.GoStraight;

        function setGoStraight() public {
            choice = ActionChoices.GoStraight;
        }

        // Since enum types are not part of the ABI, the signature of "getChoice"
        // will automatically be changed to "getChoice() returns (uint8)"
        // for all matters external to Solidity. The integer type used is just
        // large enough to hold all enum values, i.e. if you have more than 256 values,
        // `uint16` will be used and so on.
        function getChoice() public view returns (ActionChoices) {
            return choice;
        }

        function getDefaultChoice() public pure returns (uint) {
            return uint(defaultChoice);
        }
    }

.. index:: ! function type, ! type; function

.. _function_types:

Function Types
--------------

ファンクション型はファンクションの型です。
ファンクション型の変数はファンクションから割り当てられ、ファンクションコールにファンクションを渡す、またはファンクションコールからファンクションをリターンするためにファンクション型のパラメータは使用されます。
ファンクション型は2種類あります - *internal* と *external* ファンクションです:

現在のコントラクトの外からは実行することができないため、internalファンクションは現在のコントラクト内でのみ呼び出すことができます（具体的には、internalのライブラリファンクションや継承したファンクションも含むコード内）。internalのファンクションは、現在のコントラクト内部でファンクションを呼び出す様に、そのファンクションのエントリポイントにジャンプすることによって実行されます。

Externalファンクションはアドレスとファンクションの署名によって構成され、外部からのファンクションコールを通し、返ってきます。

ファンクションの種類は下記の様に表されます::

    function (<parameter types>) {internal|external} [pure|view|payable] [returns (<return types>)]

parameter typesと異なり、return typesは空ではいけません。もしファンクションが何も返さないのであれば、``returns (<return types>)`` 部分は除外しなければなりません。

デフォルトでは、ファンクション型はinternalで、``internal`` というキーワードは削除できます。これはファンクション型でのみ可能です。コントラクト内で定義されたファンクションはデフォルトで定義されておらず、可視性を明示しなければいけません。

Conversions:

externalファンクション型の値は明示的に ``address`` に変換可能で、そのファンクションのコントラクトのアドレスになります。

もしパラメータの型、返り値の型が同じであり、internal/externalのプロパティも同じ、さらに ``A`` のミュータビリティの制限が ``B`` に比べて厳しくない場合あるファンクション型 ``A`` は 別のファンクション型 ``B`` に暗黙的に変換可能です。特に:

 - ``pure`` ファンクションは ``view`` と ``non-payable`` ファンクションに変換可能
 - ``view`` ファンクションは ``non-payable`` ファンクションに変換可能
 - ``payable`` は ``non-payable`` ファンクションに変換可能

他のファンクション型間の変換はできません。

``payable`` と ``non-payable`` 間のルールは少し分かりづらいかもしれません。しかし、大事なことは、``payable`` ファンクションは0 etherの支払いを容認し、同様に ``non-payable`` ファンクションもそれを容認することです。一方で、``non-payable`` はEtherの受け取りを拒否するため、``non-payable`` ファンクションは ``payable`` ファンクションに変換できません。

ファンクション型の変数が初期化されていない場合、その変数を呼び出してもフェイルアサーションとなります。``delete`` をその変数に対して使った後に呼び出した場合も同じことが起きます。

もしexternalのファンクション型がSolidityのコンテキスト外で使用された場合、ファンクション型として扱われます。そして、それはそのファンクションの識別子とその後のアドレスを一緒に1つの ``bytes24`` 型にエンコードします。

現在のコントラクトのpublicのファンクションはinternalファンクションとしてもexternalファンクションとしても使用可能です。``f`` をinternalファンクションとして使用したい場合には、単純に ``f`` を、もしexternalファンクションとして使用した場合には、``this.f`` を使用してください。

Members:

Public（もしくはexternal）のファンクションは ``selector`` という特別なメンバも持っています。これは :ref:`ABI function selector <abi_function_selector>` を返します::

    pragma solidity >=0.4.16 <0.6.0;

    contract Selector {
      function f() public pure returns (bytes4) {
        return this.f.selector;
      }
    }

internalのファンクション型の使用例です::

    pragma solidity >=0.4.16 <0.6.0;

    library ArrayUtils {
      // internal functions can be used in internal library functions because
      // they will be part of the same code context
      function map(uint[] memory self, function (uint) pure returns (uint) f)
        internal
        pure
        returns (uint[] memory r)
      {
        r = new uint[](self.length);
        for (uint i = 0; i < self.length; i++) {
          r[i] = f(self[i]);
        }
      }
      function reduce(
        uint[] memory self,
        function (uint, uint) pure returns (uint) f
      )
        internal
        pure
        returns (uint r)
      {
        r = self[0];
        for (uint i = 1; i < self.length; i++) {
          r = f(r, self[i]);
        }
      }
      function range(uint length) internal pure returns (uint[] memory r) {
        r = new uint[](length);
        for (uint i = 0; i < r.length; i++) {
          r[i] = i;
        }
      }
    }

    contract Pyramid {
      using ArrayUtils for *;
      function pyramid(uint l) public pure returns (uint) {
        return ArrayUtils.range(l).map(square).reduce(sum);
      }
      function square(uint x) internal pure returns (uint) {
        return x * x;
      }
      function sum(uint x, uint y) internal pure returns (uint) {
        return x + y;
      }
    }

externalファンクション型の別の使用例です::

    pragma solidity >=0.4.22 <0.6.0;

    contract Oracle {
      struct Request {
        bytes data;
        function(uint) external callback;
      }
      Request[] requests;
      event NewRequest(uint);
      function query(bytes memory data, function(uint) external callback) public {
        requests.push(Request(data, callback));
        emit NewRequest(requests.length - 1);
      }
      function reply(uint requestID, uint response) public {
        // Here goes the check that the reply comes from a trusted source
        requests[requestID].callback(response);
      }
    }

    contract OracleUser {
      Oracle constant oracle = Oracle(0x1234567); // known contract
      uint exchangeRate;
      function buySomething() public {
        oracle.query("USD", this.oracleResponse);
      }
      function oracleResponse(uint response) public {
        require(
            msg.sender == address(oracle),
            "Only oracle can call this."
        );
        exchangeRate = response;
      }
    }

.. note::
    ラムダ式もしくはインラインファンクションの導入が予定されていますが、まだサポートされていません。
