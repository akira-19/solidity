.. index:: ! inheritance, ! base class, ! contract;base, ! deriving

***********
Inheritance
***********

Solidityはポリモフィズムを含めた多重継承をサポートしています。

全てのファンクションコールは仮想です。つまりコントラクト名が明示的に与えられた場合、``super`` が使われた場合を除いて、最後に派生されたファンクションが呼ばれます。

あるコントラクトがある他のコントラクトを継承するとき、1つのコントラクトだけがブロックチェーン上に生成されます。全てのベースコントラクトからのコードは生成したコントラクトにコンパイルされます。

継承のシステム全般は `Python <https://docs.python.org/3/tutorial/classes.html#inheritance>`_ にとても似ています。特に多重継承は似ていますが、いくつかの :ref:`違い <multi-inheritance>` もあります。

詳細は下記の例で示します。

::

    pragma solidity ^0.5.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    // Use `is` to derive from another contract. Derived
    // contracts can access all non-private members including
    // internal functions and state variables. These cannot be
    // accessed externally via `this`, though.
    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // These abstract contracts are only provided to make the
    // interface known to the compiler. Note the function
    // without body. If a contract does not implement all
    // functions it can only be used as an interface.
    contract Config {
        function lookup(uint id) public returns (address adr);
    }

    contract NameReg {
        function register(bytes32 name) public;
        function unregister() public;
     }

    // Multiple inheritance is possible. Note that `owned` is
    // also a base class of `mortal`, yet there is only a single
    // instance of `owned` (as for virtual inheritance in C++).
    contract named is owned, mortal {
        constructor(bytes32 name) public {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // Functions can be overridden by another function with the same name and
        // the same number/types of inputs.  If the overriding function has different
        // types of output parameters, that causes an error.
        // Both local and message-based function calls take these overrides
        // into account.
        function kill() public {
            if (msg.sender == owner) {
                Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
                NameReg(config.lookup(1)).unregister();
                // It is still possible to call a specific
                // overridden function.
                mortal.kill();
            }
        }
    }

    // If a constructor takes an argument, it needs to be
    // provided in the header (or modifier-invocation-style at
    // the constructor of the derived contract (see below)).
    contract PriceFeed is owned, mortal, named("GoldFeed") {
       function updateInfo(uint newInfo) public {
          if (msg.sender == owner) info = newInfo;
       }

       function get() public view returns(uint r) { return info; }

       uint info;
    }

上記注目して頂きたいのですが、破壊の要求を"転送"するのに ``mortal.kill()`` を呼んでいます。これは下記の例で見られる様に問題があります::

    pragma solidity >=0.4.22 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* do cleanup 1 */ mortal.kill(); }
    }

    contract Base2 is mortal {
        function kill() public { /* do cleanup 2 */ mortal.kill(); }
    }

    contract Final is Base1, Base2 {
    }

``Final.kill()`` のコールは、最後にオーバーライドされたものとして ``Base2.kill`` を呼び出し、このファンクションは ``Base1.kill`` をバイパスします。なぜなら、そのファンクションは ``Base1`` を把握していないからです。これの回避策は ``super`` を使うことです::

    pragma solidity >=0.4.22 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* do cleanup 1 */ super.kill(); }
    }


    contract Base2 is mortal {
        function kill() public { /* do cleanup 2 */ super.kill(); }
    }

    contract Final is Base1, Base2 {
    }

もし ``Base2`` が ``super`` のファンクションを呼び出しても、単純にベースコントラクトの内の1つのこのファンクションを呼び出しません。最終的な継承図の中のベースコントラクトの次のコントラクトのファンクションを呼び出します。そのため、``Base1.kill()`` を呼び出します（最終的な継承の順番は、最後に継承されたコントラクトから始まります: Final、Base2、Base1、mortal、owned）。
superを使った時に呼び出される実際のファンクションは、型が分かっていても、そのクラスのコンテキストの中ではわかりません。これは一般的な仮想メソッドの検索に似ています。

.. index:: ! constructor

.. _constructor:

Constructors
============

コンストラクタは ``constructor`` キーワードで宣言され、コントラクト生成時に実行される任意のファンクションで、コントラクトの初期化コードを実行できます。

コンストラクタが実行される前に、インラインで初期化していれば状態変数はその値で初期化され、していなければ0になります。

コンストラクタが実行された後、コントラクトの最終的なコードがブロックチェーンにデプロイされます。コードのデプロイはコードの長さに比例して追加のガスコストがかかります。
このコードはpublicインターフェースの一部でありファンクション全てと、ファンクションコールを通じてアクセスできるファンクションを含んでいます。
このコードはコンストラクタのコードと、コンストラクタからのみ呼ばれるinternalのファンクションは含んでいません。

コンストラクタは ``public`` か ``internal`` です。もし、コンストラクタがなかったら、コントラクトはデフォルトのコンストラクタ（ ``constructor() public {}`` と等価の）を想定します。例えば:

::

    pragma solidity ^0.5.0;

    contract A {
        uint public a;

        constructor(uint _a) internal {
            a = _a;
        }
    }

    contract B is A(1) {
        constructor() public {}
    }

コンストラクタを ``internal`` でセットすると、そのコントラクトは :ref:`abstract <abstract-contract>` になります。

.. warning ::
    0.4.22以前ではコンストラクタはコントラクトと同じ名前のファンクションとして定義されていました。このシンタックスは非推奨となり、バージョン0.5.0で使えなくなりました。

.. index:: ! base;constructor

Arguments for Base Constructors
===============================

全てのベースコントラクトのコンストラクタは下記で説明される線形ルールに則り呼び出されます。もしベースコンストラクタが引数を持っていたら、継承したコントラクトはその全てを指定する必要があります。
2通りの方法でできます::

    pragma solidity >=0.4.22 <0.6.0;

    contract Base {
        uint x;
        constructor(uint _x) public { x = _x; }
    }

    // Either directly specify in the inheritance list...
    contract Derived1 is Base(7) {
        constructor() public {}
    }

    // or through a "modifier" of the derived constructor.
    contract Derived2 is Base {
        constructor(uint _y) Base(_y * _y) public {}
    }

1つ目の方法は直接継承のリストに入れることです(``is Base(7)``)。もう1つの方法は継承したコンストラクタの一部としてmodifierを呼び出します(``Base(_y * _y)``)。もしコンストラクタの引数が定数で、コントラクトの挙動を決めるもしくは表現するものである場合、最初の方法の方が便利です。もしベースのコンストラクタの引数が継承したコントラクトによるのであれば、2つ目の方法を使う必要があります。引数は継承のリスト、もしくは継承したコンストラクタのmodifierスタイルで与えられる必要があります。
両方で引数を指定するとエラーとなります。

もし継承したコントラクトがベースコントラクトのコンストラクタに対する引数を決めなかった場合、そのコントラクトは抽象コントラクトになります。

.. index:: ! inheritance;multiple, ! linearization, ! C3 linearization

.. _multi-inheritance:

Multiple Inheritance and Linearization
======================================

多重継承ができる言語はいくつかの問題を扱わなければいけません。1つは `Diamond Problem <https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem>`_ です。SolidityはPythonに似ていて、"`C3 Linearization <https://en.wikipedia.org/wiki/C3_linearization>`_"を使っており、ベースクラスのdirected acyclic graph (DAG)で特定の順番にさせています。これはmonotonicityの理想的な性質を実現していますが、いくつかの継承図を許可していません。
特に、``is`` で与えられたベースクラスの順番が重要です。直のベースコントラクトを"一番ベースになるもの"から"最後に継承されるもの"の順で並べなければいけません。
これはPythonとは逆の順序であることに気をつけて下さい。

これを説明する別のシンプルな方法は、異なるコントラクトで何度か定義されたファンクションが呼ばれる時、ベースコントラクトは縦型探索で右から左に調べて（Pythonだと左から右）、最初にマッチしたところで止まります。もしすでにベースコントラクトが検索済みだった場合、それはスキップされます。

下記のコードでは、Solidityは"Linearization of inheritance graph impossible"というエラーを出します。

::

    pragma solidity >=0.4.0 <0.6.0;

    contract X {}
    contract A is X {}
    // This will not compile
    contract C is A, X {}

この理由は、``A`` をオーバーライドするのに ``C`` は ``X`` をリクエストしましたが、``A``　自体は ``X`` をオーバーライドしようとしたので、矛盾が生まれたことです。

Inheriting Different Kinds of Members of the Same Name
======================================================

継承の結果、同じ名前のファンクションとmodifierがあるコントラクトになった場合、エラーになります。
このエラーは同じイベント名とmodifier名、同じイベント名とファンクション名でも起きます。
例外として、状態変数のgetterはpublicファンクションをオーバーライドできます。
