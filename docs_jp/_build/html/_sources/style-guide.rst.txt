.. index:: style, coding style

#############
Style Guide
#############

************
Introduction
************

このガイドの目的は、Solidityコードを書くためのコーディング規約を提供することです。
また、このガイドでは、みなさんが役に立つ規約が見つけられて、過去の規約が時代遅れとなれば、時間の経過とともに変化し進化するドキュメントとして考えてください。

多くのプロジェクトでは独自のスタイルガイドが実装されます。
つまり、コンフリクトが発生した場合は、プロジェクト固有のスタイルガイドが優先されるということです。

スタイルガイド内の構造と多くのレコメンデーションはpythonの `pep8 style guide <https://www.python.org/dev/peps/pep-0008/>`_. をもとにしています。

また、このガイドの目的は、堅実なコードを書くための正しい方法や最良の方法を提供することではありません。
ガイドのゴールは、Solidityにおける *consistency* について知ってもらうことです。pythonの `pep8 <https://www.python.org/dev/peps/pep-0008/#a-foolish-consistency-is-the-hobgoblin-of-little-minds>`_ からの引用により、この目的を達成できるようします。
    
    このスタイルガイドは一貫性(consistency)について述べます。このスタイルガイドのconsistencyは重要です。もっというと、プロジェクトのconsistencyはもっと重要です。
    さらには、1つのモジュールにおけるconsistency、1つの関数におけるconsistencyはより重要なものです。
    しかし最も重要なものとして: 一貫性のない(incosistentな)実装を行うほうが良い場合について知ることです。 -- 時にはスタイルガイドが適用されないことがあります。疑わしいときは、あなたの最善の判断に従ってください。他の例を見て、最も良く見えるものを選んでください。そしてそうすることに躊躇しないでください！

***********
Code Layout
***********


Indentation
===========

インデントには4つのスペースを使ってください。

Tabs or Spaces
==============

スペースはタブよりもインデントに適しています。
また、スペースとタブの混合は避けるべきです。

Blank Lines
===========

2行の空白行を持つSolidityソースでの最上位の宣言を囲むようにします。

Yes::

    pragma solidity >=0.4.0 <0.6.0;

    contract A {
        // ...
    }


    contract B {
        // ...
    }


    contract C {
        // ...
    }

No::

    pragma solidity >=0.4.0 <0.6.0;

    contract A {
        // ...
    }
    contract B {
        // ...
    }

    contract C {
        // ...
    }

コントラクト内では、関数宣言は1行の空白行で囲むようにします。
関連するワンライナーのグループ間で空白行を省略することができます(抽象コントラクトのスタブ関数など)。

Yes::

    pragma solidity >=0.4.0 <0.6.0;

    contract A {
        function spam() public pure;
        function ham() public pure;
    }


    contract B is A {
        function spam() public pure {
            // ...
        }

        function ham() public pure {
            // ...
        }
    }

No::

    pragma solidity >=0.4.0 <0.6.0;

    contract A {
        function spam() public pure {
            // ...
        }
        function ham() public pure {
            // ...
        }
    }

.. _maximum_line_length:

Maximum Line Length
===================

 `PEP 8 recommendation <https://www.python.org/dev/peps/pep-0008/#maximum-line-length>`_ に従い、行数は最大で79行(または99行)にとどめましょう。
これによりコードの可読性が向上します。

折り返し行は、次のガイドラインに従ってください。

1. 最初の引数は左括弧に付けないでください
2. 折り返し後のインデントは1つのみ使用してください
3. 各引数は1行で表現できるようにします
4. 最後の要素 :code:`);` は最終行に単独で置いてください

Function Calls

Yes::

    thisFunctionCallIsReallyLong(
        longArgument1,
        longArgument2,
        longArgument3
    );

No::

    thisFunctionCallIsReallyLong(longArgument1,
                                  longArgument2,
                                  longArgument3
    );

    thisFunctionCallIsReallyLong(longArgument1,
        longArgument2,
        longArgument3
    );

    thisFunctionCallIsReallyLong(
        longArgument1, longArgument2,
        longArgument3
    );

    thisFunctionCallIsReallyLong(
    longArgument1,
    longArgument2,
    longArgument3
    );

    thisFunctionCallIsReallyLong(
        longArgument1,
        longArgument2,
        longArgument3);

Assignment Statements

Yes::

    thisIsALongNestedMapping[being][set][to_some_value] = someFunction(
        argument1,
        argument2,
        argument3,
        argument4
    );

No::

    thisIsALongNestedMapping[being][set][to_some_value] = someFunction(argument1,
                                                                       argument2,
                                                                       argument3,
                                                                       argument4);

Event Definitions and Event Emitters

Yes::

    event LongAndLotsOfArgs(
        address sender,
        address recipient,
        uint256 publicKey,
        uint256 amount,
        bytes32[] options
    );

    LongAndLotsOfArgs(
        sender,
        recipient,
        publicKey,
        amount,
        options
    );

No::

    event LongAndLotsOfArgs(address sender,
                            address recipient,
                            uint256 publicKey,
                            uint256 amount,
                            bytes32[] options);

    LongAndLotsOfArgs(sender,
                      recipient,
                      publicKey,
                      amount,
                      options);

Source File Encoding
====================

UTF-8とASCIIを使いましょう。

Imports
=======

Import文は常にファイルの先頭に置きましょう。

Yes::

    pragma solidity >=0.4.0 <0.6.0;

    import "./Owned.sol";

    contract A {
        // ...
    }

    contract B is Owned {
        // ...
    }

No::

    pragma solidity >=0.4.0 <0.6.0;

    contract A {
        // ...
    }


    import "./Owned.sol";


    contract B is Owned {
        // ...
    }

Order of Functions
==================

関数レベルの順序付けは、読者がどの関数を呼び出すことができるかを識別することを簡単にします。
これにより、コンストラクタとフォールバック定義を簡単に見つけるのに役立ちます。

関数は可視性(可視性修飾子を含む)に従って分類され、順序付けされるべきです:

- constructor
- fallback function(もしあるなら)
- external
- public
- internal
- private

グループ内では、関数の最後に ``view`` や ``pure`` を付けましょう。

Yes::

    pragma solidity >=0.4.0 <0.6.0;

    contract A {
        constructor() public {
            // ...
        }

        function() external {
            // ...
        }

        // External functions
        // ...

        // External functions that are view
        // ...

        // External functions that are pure
        // ...

        // Public functions
        // ...

        // Internal functions
        // ...

        // Private functions
        // ...
    }

No::

    pragma solidity >=0.4.0 <0.6.0;

    contract A {

        // External functions
        // ...

        function() external {
            // ...
        }

        // Private functions
        // ...

        // Public functions
        // ...

        constructor() public {
            // ...
        }

        // Internal functions
        // ...
    }

Whitespace in Expressions
=========================

次の場合は、余分なスペースを避けましょう:

単一行の関数宣言以外の、括弧、大括弧または大括弧のすぐ内側:

Yes::

    spam(ham[1], Coin({name: "ham"}));

No::

    spam( ham[ 1 ], Coin( { name: "ham" } ) );

Exception::

    function singleLine() public { spam(); }

コンマ、セミコロンの直前:

Yes::

    function spam(uint i, Coin coin) public;

No::

    function spam(uint i , Coin coin) public ;

代入または他の演算子の周囲に、他のものと位置を合わせるためのスペースが複数ある場合:

Yes::

    x = 1;
    y = 2;
    long_variable = 3;

No::

    x             = 1;
    y             = 2;
    long_variable = 3;

フォールバック関数内にはホワイトスペースを含めないでください:

Yes::

    function() external {
        ...
    }

No::

    function () external {
        ...
    }

Control Structures
==================

コントラクトボディ、ライブラリ、関数、および構造体を示す波括弧は、次のようになります。

* 宣言と同じ行で開きます
* 宣言の始めと同じインデントレベルで、それぞれの行を閉じます
* 波括弧を開くときは、シングルスペースに続きます

Yes::

    pragma solidity >=0.4.0 <0.6.0;

    contract Coin {
        struct Bank {
            address owner;
            uint balance;
        }
    }

No::

    pragma solidity >=0.4.0 <0.6.0;

    contract Coin
    {
        struct Bank {
            address owner;
            uint balance;
        }
    }

これらのレコメンデーションは``if`` や ``else`` 、 ``while`` 、 ``for`` といった制御構造にも適用されます。

さらに、制御構造 ``if`` 、 ``while`` 、 ``for`` と、条件を表す括弧ブロックの間、
および条件括弧ブロックと括弧の間には、シングルスペースがあるべきです。

Yes::

    if (...) {
        ...
    }

    for (...) {
        ...
    }

No::

    if (...)
    {
        ...
    }

    while(...){
    }

    for (...) {
        ...;}

コントラクトボディに単一のステートメントが含まれる制御構造の場合、波括弧は省略しても構いません。
ただし、ステートメントが1行で表せられる場合に限ります。

Yes::

    if (x < 10)
        x += 1;

No::

    if (x < 10)
        someArray.push(Coin({
            name: 'spam',
            value: 42
        }));

``else`` または ``else if`` 節を持つ ``if`` ブロックの場合、 ``else``は ``if`` の閉じ括弧と同じ行に配置する必要があります。
これは他のブロックのような構造の規則と比較して例外です。

Yes::

    if (x < 3) {
        x += 1;
    } else if (x > 7) {
        x -= 1;
    } else {
        x = 5;
    }


    if (x < 3)
        x += 1;
    else
        x -= 1;

No::

    if (x < 3) {
        x += 1;
    }
    else {
        x -= 1;
    }

Function Declaration
====================

短い関数の宣言時は、関数内の左波括弧を関数宣言と同じ行に置くことを推奨します。

また、右波括弧は、関数宣言と同じインデントレベルになければなりません。

さらに、左波括弧の前には、単一のスペースを入れます。

Yes::

    function increment(uint x) public pure returns (uint) {
        return x + 1;
    }

    function increment(uint x) public pure onlyowner returns (uint) {
        return x + 1;
    }

No::

    function increment(uint x) public pure returns (uint)
    {
        return x + 1;
    }

    function increment(uint x) public pure returns (uint){
        return x + 1;
    }

    function increment(uint x) public pure returns (uint) {
        return x + 1;
        }

    function increment(uint x) public pure returns (uint) {
        return x + 1;}

コンストラクタを含むすべての関数の可視性に明示的にラベルを付ける必要があります。

Yes::

    function explicitlyPublic(uint val) public {
        doSomething();
    }

No::

    function implicitlyPublic(uint val) {
        doSomething();
    }

関数の可視性修飾子は、どのカスタム修飾子よりも先に来る必要があります。

Yes::

    function kill() public onlyowner {
        selfdestruct(owner);
    }

No::

    function kill() onlyowner public {
        selfdestruct(owner);
    }

長い関数の宣言時は、関数本体と同じインデントレベルで、各引数を独自の行に配置することを推奨します。
また、右括弧と左角括弧は、関数宣言と同じインデントレベルで、新しい行に単一に配置する必要があります。

Yes::

    function thisFunctionHasLotsOfArguments(
        address a,
        address b,
        address c,
        address d,
        address e,
        address f
    )
        public
    {
        doSomething();
    }

No::

    function thisFunctionHasLotsOfArguments(address a, address b, address c,
        address d, address e, address f) public {
        doSomething();
    }

    function thisFunctionHasLotsOfArguments(address a,
                                            address b,
                                            address c,
                                            address d,
                                            address e,
                                            address f) public {
        doSomething();
    }

    function thisFunctionHasLotsOfArguments(
        address a,
        address b,
        address c,
        address d,
        address e,
        address f) public {
        doSomething();
    }

もしこの長い関数の宣言時に関数が修飾子を持つ場合、各修飾子は新しい行に単一に配置する必要があります。

Yes::

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public
        onlyowner
        priced
        returns (address)
    {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(
        address x,
        address y,
        address z,
    )
        public
        onlyowner
        priced
        returns (address)
    {
        doSomething();
    }

No::

    function thisFunctionNameIsReallyLong(address x, address y, address z)
                                          public
                                          onlyowner
                                          priced
                                          returns (address) {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public onlyowner priced returns (address)
    {
        doSomething();
    }

    function thisFunctionNameIsReallyLong(address x, address y, address z)
        public
        onlyowner
        priced
        returns (address) {
        doSomething();
    }

複数のアウトプットパラメータとそのreturn文は :ref:`Maximum Line Length <maximum_line_length>` セクションにある複数行をラップする場合と同じように行います。

Yes::

    function thisFunctionNameIsReallyLong(
        address a,
        address b,
        address c
    )
        public
        returns (
            address someAddressName,
            uint256 LongArgument,
            uint256 Argument
        )
    {
        doSomething()

        return (
            veryLongReturnArg1,
            veryLongReturnArg2,
            veryLongReturnArg3
        );
    }

No::

    function thisFunctionNameIsReallyLong(
        address a,
        address b,
        address c
    )
        public
        returns (address someAddressName,
                 uint256 LongArgument,
                 uint256 Argument)
    {
        doSomething()

        return (veryLongReturnArg1,
                veryLongReturnArg1,
                veryLongReturnArg1);
    }

引数が必要なコントラクトを継承するコンストラクタ関数において、関数宣言が長い場合や読みにくい場合は、
修飾子と同じ方法でベースコンストラクタを新しい行に配置することを推奨しています。

Yes::

    pragma solidity >=0.4.0 <0.6.0;

    // Base contracts just to make this compile
    contract B {
        constructor(uint) public {
        }
    }
    contract C {
        constructor(uint, uint) public {
        }
    }
    contract D {
        constructor(uint) public {
        }
    }

    contract A is B, C, D {
        uint x;

        constructor(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4)
            public
        {
            // do something with param5
            x = param5;
        }
    }

No::

    pragma solidity >=0.4.0 <0.6.0;

    // Base contracts just to make this compile
    contract B {
        constructor(uint) public {
        }
    }
    contract C {
        constructor(uint, uint) public {
        }
    }
    contract D {
        constructor(uint) public {
        }
    }

    contract A is B, C, D {
        uint x;

        constructor(uint param1, uint param2, uint param3, uint param4, uint param5)
        B(param1)
        C(param2, param3)
        D(param4)
        public
        {
            x = param5;
        }
    }

    contract X is B, C, D {
        uint x;

        constructor(uint param1, uint param2, uint param3, uint param4, uint param5)
            B(param1)
            C(param2, param3)
            D(param4)
            public {
            x = param5;
        }
    }

短い関数を1つのステートメントで宣言するときは、1行で宣言してもかまいません。

Permissible::

    function shortFunction() public { doSomething(); }

関数宣言に関するこれらのガイドラインは、可読性を向上させることを目的としています。
このガイドは関数宣言時のすべてのケースを網羅しようとするものではないので、コードを書く場合はその都度最善の判断に従うべきでしょう。

Mappings
========

変数の宣言時には、キーワード ``mapping`` をその型とスペースで区切らないでください。
また、ネストされた `` mapping``キーワードをそのタイプとスペースで区切らないでください。

Yes::

    mapping(uint => uint) map;
    mapping(address => bool) registeredAddresses;
    mapping(uint => mapping(bool => Data[])) public data;
    mapping(uint => mapping(uint => s)) data;

No::

    mapping (uint => uint) map;
    mapping( address => bool ) registeredAddresses;
    mapping (uint => mapping (bool => Data[])) public data;
    mapping(uint => mapping (uint => s)) data;

Variable Declarations
=====================

配列変数の宣言時には、型と角括弧の間にスペースを入れないでください。

Yes::

    uint[] x;

No::

    uint [] x;


Other Recommendations
=====================

* 文字列は、シングルクオーテーションではなくダブルクオーテーションで囲む必要があります:

Yes::

    str = "foo";
    str = "Hamlet says, 'To be or not to be...'";

No::

    str = 'bar';
    str = '"Be yourself; everyone else is already taken." -Oscar Wilde';

* 演算子はシングルスペースで囲ってください:

Yes::

    x = 3;
    x = 100 / 10;
    x += 3 + 4;
    x |= y && z;

No::

    x=3;
    x = 100/10;
    x += 3+4;
    x |= y&&z;

* 高い優先順位を持つ演算子は、優先順位を示すために周囲のスペースを除外することができます。
  これは複雑な代入文の可読性を向上させることを目的としています。
  また、演算子の両側には常に同じ量の空白を使用してください:

Yes::

    x = 2**3 + 5;
    x = 2*y + 3*z;
    x = (a+b) * (a-b);

No::

    x = 2** 3 + 5;
    x = y+z;
    x +=1;

***************
Order of Layout
***************

コントラクト内にある要素のレイアウトは次の順序に従ってください: 

1. Pragma statements
2. Import statements
3. Interfaces
4. Libraries
5. Contracts

各コントラクトやライブラリ、インターフェース内においては次の順序に従ってください: 

1. 型宣言
2. 状態変数
3. イベント
4. 関数

.. note::
    
    イベントや状態変数での使用に近い型を宣言する方が明確な場合があります。

******************
Naming Conventions
******************

命名規則は、採用され広く使用されている場合に強力なものとなります。
異なる規則を使用すると、利用できない重要な *meta* 情報を伝えることができます。

ここで与えられた命名規則は可読性を改善することを目的としています。
そのため、それらは規則というより、むしろ名前を通してほとんどの情報を伝えられるという点に着目します。

最後に、コードベース内における一貫性は、この文書で説明されている規約よりも常に優先されるべきです。

Naming Styles
=============

混乱を避けるために、以下の名前はさまざまな命名スタイルを指すために使用されます。

* ``b`` (単一の小文字)
* ``B`` (単一の大文字)
* ``lowercase``
* ``lower_case_with_underscores``
* ``UPPERCASE``
* ``UPPER_CASE_WITH_UNDERSCORES``
* ``CapitalizedWords`` (もしくはCapWords)
* ``mixedCase`` (頭文字が小文字である点がCapitalizedWordsと異なります！)
* ``Capitalized_Words_With_Underscores``

.. note:: CapWordsでイニシャリズムを使用するときは、イニシャリズムのすべての文字を大文字にします。そのため、HTTPServerError は HttpServerError よりも良い命名といえます。イニシャルを使用する場合は、mixedCaseを使用します。名前の頭文字である場合は最初の1文字を小文字にする以外は、イニシャルのすべての文字を大文字にします。そのため、xmlHTTPRequest は XMLHTTPRequest よりも良い命名です。

Names to Avoid
==============

* ``l`` - 小文字の el
* ``O`` - 大文字の oh
* ``I`` - 大文字の eye

1文字の変数名にこれらのいずれも使用しないでください。数字の1と0と区別がつかないケースがよくあります。

Contract and Library Names
==========================

* コントラクトとライブラリは、CapWordsスタイルを使用して命名する必要があります。 例: ``SimpleToken`` 、 ``SmartBank`` 、 ``CertificateHashRepository`` 、 ``Player`` 、 ``Congress`` 、 ``Owned`` など。
* コントラクトとライブラリの名前もそれらのファイル名と一致する必要があります。
* コントラクトファイルに複数のコントラクトやライブラリが含まれている場合、ファイル名は *core contract* と一致させる必要があります。ただしこの構造はお勧めできないため、でいるだけ回避しましょう。


以下の例に示すように、コントラクト名が `Congress` でライブラリ名が `Owned` の場合、それらに関連するファイル名は `Congress.sol` と ` Owned.sol` になります。

Yes::

    pragma solidity >=0.4.0 <0.6.0;

    // Owned.sol
    contract Owned {
         address public owner;

         constructor() public {
             owner = msg.sender;
         }

         modifier onlyOwner {
             require(msg.sender == owner);
             _;
         }

         function transferOwnership(address newOwner) public onlyOwner {
             owner = newOwner;
         }
    }

    // Congress.sol
    import "./Owned.sol";

    contract Congress is Owned, TokenRecipient {
        //...
    }

No::

    pragma solidity >=0.4.0 <0.6.0;

    // owned.sol
    contract owned {
         address public owner;

         constructor() public {
             owner = msg.sender;
         }

         modifier onlyOwner {
             require(msg.sender == owner);
             _;
         }

         function transferOwnership(address newOwner) public onlyOwner {
             owner = newOwner;
         }
    }

    // Congress.sol
    import "./owned.sol";

    contract Congress is owned, tokenRecipient {
        //...
    }


Struct Names
==========================

構造体はCapWordsスタイルを使用して命名する必要があります。例: ``MyCoin`` 、 ``Position`` 、 ``PositionXY`` など。


Event Names
===========

イベントはCapWordsスタイルを使って命名されるべきです。例: ``Deposit`` 、 ``Transfer`` 、 ``Approval`` 、 ``BeforeTransfer`` 、 ``AfterTransfer`` など。


Function Names
==============

コンストラクタ以外の関数はmixedCaseを使用するべきです。例: ``getBalance`` 、 ``transfer`` 、 ``verifyOwner`` 、 ``addMember`` 、 ``changeOwner`` など。


Function Argument Names
=======================

関数の引数はmixedCaseを使用するべきです。例: ``initialSupply`` 、 ``account`` 、 ``recipientAddress`` 、 ``senderAddress`` 、 ``newOwner`` など。

カスタム構造体を操作するライブラリ関数を書くときは、その構造体を最初の引数にして、常に ``self`` という名前にします。


Local and State Variable Names
==============================

mixedCaseを使用してください。 例: ``totalSupply`` 、 ``remainingSupply`` 、 ``balancesOf`` 、 ``creatorAddress`` 、 ``isPreSale`` 、 ``tokenExchangeRate`` など。


Constants
=========

定数は単語を区切るアンダースコア付きのすべて大文字で名前を付ける必要があります。例: ``MAX_BLOCKS`` 、 ``TOKEN_NAME`` 、 ``TOKEN_TICKER`` 、 ``CONTRACT_VERSION`` など。



Modifier Names
==============

mixedCaseを使用してください。 例: ``onlyBy`` 、 ``onlyAfter`` 、 ``onlyDuringThePreSale`` など。


Enums
=====

列挙型は、単純な型宣言において、CapWordsスタイルを使用して命名する必要があります。例: ``TokenGroup`` 、 ``Frame`` 、 ``HashStyle`` 、 ``CharacterLocation`` など。


Avoiding Naming Collisions
==========================

* ``single_trailing_underscore_``

この規則は、命名する名前が組み込み名または予約語と競合する場合に推奨されます。


General Recommendations
=======================

TODO

.. _natspec:

*******
NatSpec
*******

Solidityコントラクトは、Ethereum Natural Language Specification Format のベースとなっている形式のコメントを付けることができます。

`///` またはaで始まる1行または複数行の `doxygen <http://www.doxygen.nl>`_ 表記に続く関数または規約の上にコメントを追加します。

例えば、コメントが追加された `a simple smart contract <simple-smart-contract>`_ のコントラクトは以下のようになります::

    pragma solidity >=0.4.0 <0.6.0;

    /// @author The Solidity Team
    /// @title A simple storage example
    contract SimpleStorage {
        uint storedData;

        /// Store `x`.
        /// @param x the new value to store
        /// @dev stores the number in the state variable `storedData`
        function set(uint x) public {
            storedData = x;
        }

        /// Return the stored value.
        /// @dev retrieves the value of the state variable `storedData`
        /// @return the stored value
        function get() public view returns (uint) {
            return storedData;
        }
    }

Natspecは特別な意味を持つdoxygenスタイルのタグを使います。
タグが使用されていない場合、コメントは ``@notice`` として適用されます。
``@notice`` タグはNatSpecのメインタグで、読み手はソースコードを読んだことのないコントラクトユーザーを想定しています。
そのため、できるだけ内部の詳細についての推測させないようにするべきです。
また、すべてのタグはオプショナルです。

+-------------+-------------------------------------------+-------------------------------+
| Tag         | Description                               | Context                       |
+=============+===========================================+===============================+
| ``@title``  | A title that describes the contract       | contract, interface           |
+-------------+-------------------------------------------+-------------------------------+
| ``@author`` | The name of the author                    | contract, interface, function |
+-------------+-------------------------------------------+-------------------------------+
| ``@notice`` | Explanation of functionality              | contract, interface, function |
+-------------+-------------------------------------------+-------------------------------+
| ``@dev``    | Any extra details                         | contract, interface, function |
+-------------+-------------------------------------------+-------------------------------+
| ``@param``  | Parameter type followed by parameter name | function                      |
+-------------+-------------------------------------------+-------------------------------+
| ``@return`` | The return value of a contract's function | function                      |
+-------------+-------------------------------------------+-------------------------------+
