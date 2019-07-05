********************************
Solidity v0.5.0 Breaking Changes
********************************

このセクションではSolidityバージョン0.5.0で導入された主要な変更を、変更の理由と影響あるコードをアップデートする方法と共に紹介します。
フルリストは、`the release changelog <https://github.com/ethereum/solidity/releases/tag/v0.5.0>`_ を参照ください。

.. note::
   Solidityのバージョン0.5.0でコンパイルされたコントラクトは古いバージョンでコンパイルされたコントラクトやライブラリを再コンパイルもしくは再デプロイなしに使えます。
   データロケーション、可視性、ミュータビリティを含めるようにインターフェースを変えれば十分です。下記の :ref:`Interoperability With Older Contracts <interoperability>` を参照ください。


Semantic Only Changes
=====================

このセクションはセマンティクス上の変更だけのリストです。そのため、潜在的には現状のコードで、新しいもしくは異なる動作をしてしまうかもしれません。

* 符号付右シフトは現在適切な算術シフトを使っています。つまり、0への丸めの代わりに負の無限大への丸めとなります。符号付、符号なしのシフトはConstantinopleで専用のopcodeになり、その間Solidityによってエミュレートされます。

* ``do...while`` ループの中の ``continue`` は現在条件文にjumpします。そのような場合になる一般的な挙動です。以前はループの本体にjumpしていました。もし条件文がfalseだった場合、ループは終了します。

* ``bytes`` が1つ与えられた時に、ファンクションの ``.call()``、``.delegatecall()``、``.staticcall()`` はもうパディングしません。

* pureとviewファンクションはEVMバージョンがByzantium以降であれば、``CALL`` の代わりに ``STATICCALL`` を使って呼ばれます。このopcodeではEVMレベルで、状態変更ができません。

* 外部のファンクションコールと ``abi.encode`` で使われた時、ABIエンコーダは現在calldata（ ``msg.data`` と外部のファンクションパラメータ）のバイト配列と文字列を適切にパディングします。パディングされないエンコードしたい時は、``abi.encodePacked`` を使用してください。

* ABIデコーダは渡されたcalldataが短すぎるもしくは範囲外を指し示していたら、ファンクションの始めと ``abi.decode()`` で状態を全て元に戻します。
汚れた高次ビットは単純に無視されるということを覚えておいてください。

* Tangerine Whistleからは外部のファンクションコールで使用可能な全てのガスを転送します。

Semantic and Syntactic Changes
==============================

このセクションではシンタックスとセマンティクスに関連する変更を紹介します。

* ``.call()``、``.delegatecall()``、``staticcall()``、``keccak256()``、``sha256()``、``ripemd160()`` は1つの ``bytes`` 引数だけを受け入れます。さらに引数はパディングされません。これは引数がどのように連結させるかをわかりやすくするためです。
全ての ``.call()`` (and family) を ``.call("")`` に代えてください。``.call(signature, a,
  b, c)`` の代わりに ``.call(abi.encodeWithSignature(signature, a, b, c))`` (最後のものは値型にしか使えません)を使ってください。また、``keccak256(a, b, c)`` を ``keccak256(abi.encodePacked(a, b, c))`` に代えてください。ブレーキングチェンジではありませんが、``x.call(bytes4(keccak256("f(uint256)"), a, b)`` から ``x.call(abi.encodeWithSignature("f(uint256)", a, b))`` に代えることをお勧めします。

* ``.call()``、``.delegatecall()``、``.staticcall()`` は返ってきたデータにアクセスできるように現在 ``(bool, bytes memory)`` を返します。``bool success = otherContract.call("f")`` を ``(bool success, bytes memory
data) = otherContract.call("f")`` に変更して下さい。

* Solidityは現在ファンクションのローカル変数に対してC99-styleのスコーピングのルールを使っています。その中で、変数は宣言された後でのみ有効で、同じもしくはネストされたスコープでのみ使えます。``for`` ループの初期化ブロックの中で宣言された変数はループ内のどこでも有効です。

Explicitness Requirements
=========================

このセクションはコードをもっと明示的にする様な変更のリストです。
ほとんどの場合、コンパイラが忠告を出してくれます。

* 明示的なファンクションの可視性は現在必須です。可視性を特に指示していないものに関して、ファンクションとコンストラクタには ``public`` を追加し、fallbackとインターフェースには ``external`` を追加して下さい。

* 構造体、配列、マッピングに関して明示的なデータロケーションは現在必須です。これはファンクションのパラメータや返り値にも適用されます。例えば、``uint[] x = m_x`` は ``uint[] storage x =
m_x`` に、``function f(uint[][] x)`` は ``function f(uint[][] memory x)`` に代えてください。
ここで、``memory`` はデータロケーションで、場合によって ``storage`` や ``calldata`` に置き換えられます。``external`` のファンクションはデータロケーションとして　``calldata`` が必要なことを覚えておいてください。

* 名前空間を分けるために、コントラクト型は ``address`` メンバをもう持っていません。そのため、``address`` メンバを使う前に明示的にコントラクト型の値をアドレスに変換する必要があります。例： ``c`` がコントラクトの場合、``c.transfer(...)`` から ``address(c).transfer(...)`` に、``c.balance`` は ``address(c).balance`` に代えてください。

* 関係ないコントラクト型同士の明示的な変換は現在できません。あるコントラクト型から、そのベースもしくは親（祖先）のコントラクト型への変換のみ可能です。もし継承していないのにも関わらず、あるコントラクトから別のコントラクトへの変換ができると考えているのであれば、まず ``address`` へ変換することで対処できます。
例: ``A`` と ``B`` がコントラクト型で``B`` が ``A`` を継承しておらず、``b`` が ``B`` 型のコントラクトである場合、``A(address(b))`` を使うことで ``b`` を ``A`` 型に変換できます。
下記で説明されている通り、payable fallbackファンクションのマッチングに気をつける必要があります。

* ``address`` 型は ``address`` と ``address payable`` に分けられ、``address payable`` だけが ``transfer`` ファンクションを使えます。``address payable`` は直接 ``address`` に変換可能ですが、逆はできません。``address`` から ``address payable`` への変換は ``uint160`` を通じて行うことで可能です。``c`` がコントラクトで、payable fallbackファンクションを持っていた場合、``address(c)`` は ``address payable`` になります。もし :ref:`withdraw pattern<withdrawal_pattern>` を使っているのであれば、コードを変える必要はきっとないでしょう。なぜなら、``transfer`` は 保存されているアドレスの代わりに ``msg.sender`` にのみ使われ、``msg.sender`` は ``address payable`` だからです。

* 異なるサイズの ``bytesX`` と ``uintY`` 間の変換は現在できません。なぜなら、``bytesX`` は右パディングで ``uintY`` は左パディングのため、予期しない変換結果を生じる可能性があるためです。変換前に型内でサイズは調整される必要があります。例えば、始めに ``bytes4`` を ``bytes8`` に変換してから ``uint64`` に変換することで ``bytes4`` (4 bytes) を ``uint64`` (8 bytes) に変換することができます。``uint32`` を通じて変換すると逆側のパディングをすることになります。

* payableではないファンクションで ``msg.value`` を使うこと（もしくはmodifierを使って実行する）はセキュリティの機能によりできません。
``payable`` のファンクションに変換するか、``msg.value`` を使う新しいinternalのファンクションを作ってください。

* 分かりやすくするため、標準入力がソースとして使用される場合、コマンドラインインターフェースに ``-`` が必要です。

Deprecated Elements
===================

このセクションでは以前の機能やシンタックスで非推奨に変更になったリストを紹介します。これらの多くは既に ``v0.5.0`` のexperimental modeでは使用できません。

Command Line and JSON Interfaces
--------------------------------

* コマンドラインオプションの ``--formal`` （形式的検証のため以前はWhy3のアウトプットを生成していた）は非推奨隣、現在は削除されました。新しい形式的検証のモジュールのSMTCheckerは ``pragma
experimental SMTChecker;`` で使うことができます。

* 中間言語の名前が ``Julia`` から ``Yul`` へ変更になったためコマンドラインオプションの ``--julia`` は ``--yul`` へ名前が変更されました。

* コマンドラインオプションの ``--clone-bin`` と ``--combined-json clone-bin`` は削除されました。

* 空の接頭辞での理マッピングはできません。

* JSON ASTフィールドの``constant`` と ``payable`` は削除されました。その情報は現在 ``stateMutability`` フィールドにあります。

* JSON ASTフィールドの ``FunctionDefinition`` ノード
の ``isConstructor`` は ``kind`` というフィールドに置き換えられ、``"constructor"``、``"fallback"``、``"function"` という値を持つことができます。

* リンクしてないbinary hexファイルでは、ライブラリアドレスのプレースホルダは正規のライブラリの名前全体のkeccak256ハッシュの最初の16進数の36文字で、``$...$`` に囲われています。
以前は正規のライブラリの名前だけ使われていました。これにより、特にパスが長い時は名前が衝突する可能性が減ります。バイナリファイルは現在、このプレースホルダから正規の名前全体のマッピングのリストを含んでいます。

Constructors
------------

* コンストラクトは現在、``constructor`` を使って定義しなければいけません。

* 括弧なしのベースのコンストラクトを呼び出すことは現在できません。

* 同じ継承の階層でベースのコンストラクトの引数をな何度も決めることは現在できません。

* 間違った数の引数でコンストラクタを呼ぶことはできません。もし引数を渡さないで継承の関係を指定したい場合は、括弧をつけないで下さい。

Functions
---------

* ``callcode`` ファンクションは現在使えません（）。
Function ``callcode`` is now disallowed ( ``delegatecall`` が使われます)。インラインアセンブリを使えば使うことができます。

* ``suicide`` は現在使えません( ``selfdestruct`` が使われます).

* ``sha3`` は現在使えません( ``keccak256`` が使われます).

* ``throw`` は現在使えません( ``revert``、``require``、``assert`` が使われます)。

Conversions
-----------

* 明示的にも暗示的にも10進数リテラルの ``bytesXX`` 型への変換は現在使えません。

* 明示的にも暗示的にも16進数リテラルの異なるサイズの ``bytesXX`` 型への変換は現在使えません。

Literals and Suffixes
---------------------

* 閏年で混乱してしまうため ``years`` という単位名は現在使えません。数字の後に

* 数字の続かない末尾のドットは使えません。

* 16進数の数字とuintの単位名(例: ``0x1e wei``) は一緒に使えません。

* 16進数に接頭辞 ``0X`` は使えません。``0x`` だけ使えます。

Variables
---------

* 分かりやすさのため、空の構造体の宣言は現在できません。

* 分かりやすさのため、現在 ``var`` キーワードは使えません。

* タプルへの異なった要素数の割り当ては現在できません。

* コンパイル時に定数でない定数値は使えません。

* 値の数と変数の数が合わない複数変数の宣言は現在使えません。

* 初期化されていないストレージ変数は現在使えません。

* 空のタプル要素は現在使えません。

* 変数と構造体の循環参照の検知の繰り返しは256回までと制限されています。

* 長さが0の固定長さ配列は現在使えません。

Syntax
------

* ファンクションのstate mutability modifierとしての ``constant`` は現在使えません。

* 真偽式は算術演算を行えません。

* 単項の ``+`` 演算子は使えません。

* リテラルは以前の明示的な型への変換なしでは ``abi.encodePacked`` を使えません。

* 返り値があるファンクションでの空のリターンは現在使えません。

* "loose assembly"シンタックスは現在完全に使えません。つまりjump label、jump、非ファンクショナルな指示はもう使えません。新しい ``while``、``switch``、``if`` を代わりに使ってください。

* 実行されないファンクションではmodifierは使えません。

* 名前のついた返り値のファンクション型は使えません。

* if/while/forのブロックでないボディ内での単文での変数の宣言はできません。

* New keywords: ``calldata`` and ``constructor``.

* New reserved keywords: ``alias``, ``apply``, ``auto``, ``copyof``,
  ``define``, ``immutable``, ``implements``, ``macro``, ``mutable``,
  ``override``, ``partial``, ``promise``, ``reference``, ``sealed``,
  ``sizeof``, ``supports``, ``typedef`` and ``unchecked``.

.. _interoperability:

Interoperability With Older Contracts
=====================================

インターフェースを定義すれば、Solidityバージョン0.5.0以前（もしくは以降）で書かれたコントラクトで使うことができます。
0.5.0以前のバージョンで作った以下のコントラクトを見てください。

::

   // This will not compile with the current version of the compiler
   pragma solidity ^0.4.25;
   contract OldContract {
      function someOldFunction(uint8 a) {
         //...
      }
      function anotherOldFunction() constant returns (bool) {
         //...
      }
      // ...
   }

これは0.5.0ではコンパイルできませんが、互換性のあるインターフェースは定義できます。

::

   pragma solidity ^0.5.0;
   interface OldContract {
      function someOldFunction(uint8 a) external;
      function anotherOldFunction() external returns (bool);
   }


オリジナルのコントラクトでは ``constant`` と宣言されているのにも関わらず、``anotherOldFunction`` を ``view`` で宣言していないことに注目して下さい。これはSolidity v0.5.0から ``staticcall`` が ``view`` ファンクションを呼ぶのに使われ始めたからです。
v0.5.0以前では ``constant`` キーワードは強制ではありませんでしたので、``constant`` と宣言されたファンクションを ``staticcall`` で呼び出してもrevertする可能性がありました。なぜなら、``constant`` ファンクションはストレージを修正しようとする可能性があったためです。
結論としては、古いコントラクト用のインターフェースを定義するときには、``staticcall`` で確実にファンクションが動作する様に、``constant`` の代わりに ``view`` を使った方が良いでしょう。

上記で定義されたインターフェースがあれば、もう簡単に0.5.0以前のコントラクトをデプロイできます。

::

   pragma solidity ^0.5.0;

   interface OldContract {
      function someOldFunction(uint8 a) external;
      function anotherOldFunction() external returns (bool);
   }

   contract NewContract {
      function doSomething(OldContract a) public returns (bool) {
         a.someOldFunction(0x42);
         return a.anotherOldFunction();
      }
   }

同様に、0.5.0以前のライブラリは実行せず、リンク中にそのライブラリのアドレスを渡さなくても、そのライブラリのファンクションを定義することで、使用することができます（リンキングにどうやってコマンドラインコンパイラを使うかは :ref:`commandline-compiler` を参照ください）。

::

   pragma solidity ^0.5.0;

   library OldLibrary {
      function someFunction(uint8 a) public returns(bool);
   }

   contract NewContract {
      function f(uint8 a) public returns (bool) {
         return OldLibrary.someFunction(a);
      }
   }


Example
=======

下記の例ではあるコントラクトとこのセクションで紹介したいくつかの変更を加えたSolidityバージョン0.5.0のコントラクトを紹介します。

Old version:

::

   // This will not compile
   pragma solidity ^0.4.25;

   contract OtherContract {
      uint x;
      function f(uint y) external {
         x = y;
      }
      function() payable external {}
   }

   contract Old {
      OtherContract other;
      uint myNumber;

      // Function mutability not provided, not an error.
      function someInteger() internal returns (uint) { return 2; }

      // Function visibility not provided, not an error.
      // Function mutability not provided, not an error.
      function f(uint x) returns (bytes) {
         // Var is fine in this version.
         var z = someInteger();
         x += z;
         // Throw is fine in this version.
         if (x > 100)
            throw;
         bytes b = new bytes(x);
         y = -3 >> 1;
         // y == -1 (wrong, should be -2)
         do {
            x += 1;
            if (x > 10) continue;
            // 'Continue' causes an infinite loop.
         } while (x < 11);
         // Call returns only a Bool.
         bool success = address(other).call("f");
         if (!success)
            revert();
         else {
            // Local variables could be declared after their use.
            int y;
         }
         return b;
      }

      // No need for an explicit data location for 'arr'
      function g(uint[] arr, bytes8 x, OtherContract otherContract) public {
         otherContract.transfer(1 ether);

         // Since uint32 (4 bytes) is smaller than bytes8 (8 bytes),
         // the first 4 bytes of x will be lost. This might lead to
         // unexpected behavior since bytesX are right padded.
         uint32 y = uint32(x);
         myNumber += y + msg.value;
      }
   }

New version:

::

   pragma solidity ^0.5.0;

   contract OtherContract {
      uint x;
      function f(uint y) external {
         x = y;
      }
      function() payable external {}
   }

   contract New {
      OtherContract other;
      uint myNumber;

      // Function mutability must be specified.
      function someInteger() internal pure returns (uint) { return 2; }

      // Function visibility must be specified.
      // Function mutability must be specified.
      function f(uint x) public returns (bytes memory) {
         // The type must now be explicitly given.
         uint z = someInteger();
         x += z;
         // Throw is now disallowed.
         require(x > 100);
         int y = -3 >> 1;
         // y == -2 (correct)
         do {
            x += 1;
            if (x > 10) continue;
            // 'Continue' jumps to the condition below.
         } while (x < 11);

         // Call returns (bool, bytes).
         // Data location must be specified.
         (bool success, bytes memory data) = address(other).call("f");
         if (!success)
            revert();
         return data;
      }

      using address_make_payable for address;
      // Data location for 'arr' must be specified
      function g(uint[] memory arr, bytes8 x, OtherContract otherContract, address unknownContract) public payable {
         // 'otherContract.transfer' is not provided.
         // Since the code of 'OtherContract' is known and has the fallback
         // function, address(otherContract) has type 'address payable'.
         address(otherContract).transfer(1 ether);

         // 'unknownContract.transfer' is not provided.
         // 'address(unknownContract).transfer' is not provided
         // since 'address(unknownContract)' is not 'address payable'.
         // If the function takes an 'address' which you want to send
         // funds to, you can convert it to 'address payable' via 'uint160'.
         // Note: This is not recommended and the explicit type
         // 'address payable' should be used whenever possible.
         // To increase clarity, we suggest the use of a library for
         // the conversion (provided after the contract in this example).
         address payable addr = unknownContract.make_payable();
         require(addr.send(1 ether));

         // Since uint32 (4 bytes) is smaller than bytes8 (8 bytes),
         // the conversion is not allowed.
         // We need to convert to a common size first:
         bytes4 x4 = bytes4(x); // Padding happens on the right
         uint32 y = uint32(x4); // Conversion is consistent
         // 'msg.value' cannot be used in a 'non-payable' function.
         // We need to make the function payable
         myNumber += y + msg.value;
      }
   }

   // We can define a library for explicitly converting ``address``
   // to ``address payable`` as a workaround.
   library address_make_payable {
      function make_payable(address x) internal pure returns (address payable) {
         return address(uint160(x));
      }
   }
