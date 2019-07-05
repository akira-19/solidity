#################
Solidity Assembly
#################

.. index:: ! assembly, ! asm, ! evmasm

SolidityはSolidityなしで使えるアセンブリ言語を定義しており、更にSolidityのソースコード内で"インラインアセンブリ"も使用可能です。このガイドではまずどの様にインラインアセンブリを使用するのか、スタンドアローンアセンブリとどう違うのか、そしてアセンブリ自体の説明をしていきます。

.. _inline-assembly:

Inline Assembly
===============

仮想マシンの様な言語のインラインアセンブリで、Solidityの宣言をインターリーブできます。これにより細かいコントールを得ることができます。特にライブラリを作って言語を便利にする時に有用です。

EVMはスタックマシンなので、正確なスタック位置の指定や正確なスタック位置のopcodeに引数を与えるのは難しいです。Solidityのインラインアセンブリはこれらの問題や、マニュアルアセンブリを書いている時に出てくる問題に対して役にたちます。

インラインアセンブリは下記の特徴があります:

* functional-style opcodes: ``mul(1, add(2, 3))``
* assembly-local variables: ``let x := add(2, 3)  let y := mload(0x40)  x := add(x, y)``
* 外部の変数にアクセス: ``function f(uint x) public { assembly { x := sub(x, 1) } }``
* ループ: ``for { let i := 0 } lt(i, x) { i := add(i, 1) } { y := mul(2, y) }``
* if文: ``if slt(x, 0) { x := sub(0, x) }``
* switch分: ``switch x case 0 { y := mul(x, 2) } default { y := 0 }``
* ファンクションコール: ``function f(x) -> y { switch x case 0 { y := 1 } default { y := mul(x, f(sub(x, 1))) }   }``

.. warning::
    インラインアセンブリはEVMに低レベルでアクセスする方法です。そのため、Solidityの大事な安全に関する機能やチェックをバイパスします。必要がある時のみ、またこの機能について知識がある時のみ使用する方が良いでしょう。

Syntax
------

アセンブリはコメント、リテラル、識別子をSolidityと同じ方法でパースします。そのため、普段使っている ``//`` と ``/* */`` は使用可能です。インラインアセンブリは ``assembly { ... }`` で書かれており、中括弧の中では下記が使用可能です（詳細は後述のセクションを参照ください）:

 - リテラル 例： ``0x123``、``42``、``"abc"`` (文字列は32文字まで)
 - ファンクション式のopcode 例: ``add(1, mlod(0))``
 - 変数の宣言 例: ``let x := 7``, ``let x := add(y, 3)`` or ``let x`` (空（0）の初期値は割り当て済み)
 - 識別子 (インラインアセンブリで使われたassembly-local 変数と外部のもの) 例: ``add(3, x)``, ``sstore(x_slot, 2)``
 - 値の割り当て 例: ``x := add(y, 3)``
 - ローカルの変数が内部でスコープされているブロック 例: ``{ let x := 3 { let y := add(x, 1) } }``

下記の機能はスタンドアローンのアセンブリでのみ使用可能です:

 - ``dup1``、``swap1``、、、を介した直接的なスタックのコントロール
 - ダイレクトなスタックの割り当て (in "instruction style") 例: ``3 =: x``
 - ラベル 例: ``name:``
 - ジャンプopcode

.. note::
  スタンドアローンのアセンブリは後方互換性をサポートしますが、ここではドキュメント化しません。

要求しない限り、``assembly { ... }`` ブロックの最後に、スタックはバランスしなければいけません。もしバランスしていなかったら、コンパイラは警告を発します。

Example
-------

下記の例は、別のコントラクトのコードにアクセスするライブラリのコード例で、``bytes`` 変数にそのコードを載せています。"生のSolidity" ではこれはできません。アセンブリライブラリはSolidityという言語をより良くするために使用されます。

.. code::

    pragma solidity >=0.4.0 <0.6.0;

    library GetCode {
        function at(address _addr) public view returns (bytes memory o_code) {
            assembly {
                // retrieve the size of the code, this needs assembly
                let size := extcodesize(_addr)
                // allocate output byte array - this could also be done without assembly
                // by using o_code = new bytes(size)
                o_code := mload(0x40)
                // new "memory end" including padding
                mstore(0x40, add(o_code, and(add(add(size, 0x20), 0x1f), not(0x1f))))
                // store length in memory
                mstore(o_code, size)
                // actually retrieve the code, this needs assembly
                extcodecopy(_addr, add(o_code, 0x20), 0, size)
            }
        }
    }

オプティマイザが効率的なコードを生成するのに失敗した場合にもインラインアセンブリは有用です。例えば:

.. code::

    pragma solidity >=0.4.16 <0.6.0;

    library VectorSum {
        // This function is less efficient because the optimizer currently fails to
        // remove the bounds checks in array access.
        function sumSolidity(uint[] memory _data) public pure returns (uint o_sum) {
            for (uint i = 0; i < _data.length; ++i)
                o_sum += _data[i];
        }

        // We know that we only access the array in bounds, so we can avoid the check.
        // 0x20 needs to be added to an array because the first slot contains the
        // array length.
        function sumAsm(uint[] memory _data) public pure returns (uint o_sum) {
            for (uint i = 0; i < _data.length; ++i) {
                assembly {
                    o_sum := add(o_sum, mload(add(add(_data, 0x20), mul(i, 0x20))))
                }
            }
        }

        // Same as above, but accomplish the entire code within inline assembly.
        function sumPureAsm(uint[] memory _data) public pure returns (uint o_sum) {
            assembly {
               // Load the length (first 32 bytes)
               let len := mload(_data)

               // Skip over the length field.
               //
               // Keep temporary variable so it can be incremented in place.
               //
               // NOTE: incrementing _data would result in an unusable
               //       _data variable after this assembly block
               let data := add(_data, 0x20)

               // Iterate until the bound is not met.
               for
                   { let end := add(data, mul(len, 0x20)) }
                   lt(data, end)
                   { data := add(data, 0x20) }
               {
                   o_sum := add(o_sum, mload(data))
               }
            }
        }
    }


.. _opcodes:

Opcodes
-------

このドキュメントではEthereum Virtual Machineについて完全には説明しませんが、下記のリストはEVMのopcodeのリファレンスとして使用できます。

opcodeが引数をとる場合（常にスタックの上からとります）、括弧の中に引数が入ります。
引数の順番はnon-functional styleでは逆さまに入っています（後述します）。
``-`` がついているopcodeは何もスタック上にプッシュしません（結果も返しません）。``*`` がついているopcodeは特別で、他のopcodeは1つだけスタック上にプッシュします（"返り値"）。
``F``、``H``、``B``、``C`` がついているopcodeはそれぞれFrontier、Homestead、Byzantium、Constantinople から導入されました。Constantinopleは未だプラニングの段階ですので、そのマークがついているインストラクションは無効なインストラクションの例外を投げます。

下記で、``mem[a...b)`` は 位置 ``a`` から始まって、``b`` で終わる（bは含まない）メモリのバイトを表しており、``storage[p]`` は 位置 ``p`` でのストレージの内容を表しています。

``pushi`` と ``jumpdest`` のopcodeは直接は使用できません。

グラマー上、opcodeは事前に定義された識別子として表されます。


+-------------------------+-----+---+-----------------------------------------------------------------+
| Instruction             |     |   | Explanation                                                     |
+=========================+=====+===+=================================================================+
| stop                    + `-` | F | stop execution, identical to return(0,0)                        |
+-------------------------+-----+---+-----------------------------------------------------------------+
| add(x, y)               |     | F | x + y                                                           |
+-------------------------+-----+---+-----------------------------------------------------------------+
| sub(x, y)               |     | F | x - y                                                           |
+-------------------------+-----+---+-----------------------------------------------------------------+
| mul(x, y)               |     | F | x * y                                                           |
+-------------------------+-----+---+-----------------------------------------------------------------+
| div(x, y)               |     | F | x / y                                                           |
+-------------------------+-----+---+-----------------------------------------------------------------+
| sdiv(x, y)              |     | F | x / y, for signed numbers in two's complement                   |
+-------------------------+-----+---+-----------------------------------------------------------------+
| mod(x, y)               |     | F | x % y                                                           |
+-------------------------+-----+---+-----------------------------------------------------------------+
| smod(x, y)              |     | F | x % y, for signed numbers in two's complement                   |
+-------------------------+-----+---+-----------------------------------------------------------------+
| exp(x, y)               |     | F | x to the power of y                                             |
+-------------------------+-----+---+-----------------------------------------------------------------+
| not(x)                  |     | F | ~x, every bit of x is negated                                   |
+-------------------------+-----+---+-----------------------------------------------------------------+
| lt(x, y)                |     | F | 1 if x < y, 0 otherwise                                         |
+-------------------------+-----+---+-----------------------------------------------------------------+
| gt(x, y)                |     | F | 1 if x > y, 0 otherwise                                         |
+-------------------------+-----+---+-----------------------------------------------------------------+
| slt(x, y)               |     | F | 1 if x < y, 0 otherwise, for signed numbers in two's complement |
+-------------------------+-----+---+-----------------------------------------------------------------+
| sgt(x, y)               |     | F | 1 if x > y, 0 otherwise, for signed numbers in two's complement |
+-------------------------+-----+---+-----------------------------------------------------------------+
| eq(x, y)                |     | F | 1 if x == y, 0 otherwise                                        |
+-------------------------+-----+---+-----------------------------------------------------------------+
| iszero(x)               |     | F | 1 if x == 0, 0 otherwise                                        |
+-------------------------+-----+---+-----------------------------------------------------------------+
| and(x, y)               |     | F | bitwise and of x and y                                          |
+-------------------------+-----+---+-----------------------------------------------------------------+
| or(x, y)                |     | F | bitwise or of x and y                                           |
+-------------------------+-----+---+-----------------------------------------------------------------+
| xor(x, y)               |     | F | bitwise xor of x and y                                          |
+-------------------------+-----+---+-----------------------------------------------------------------+
| byte(n, x)              |     | F | nth byte of x, where the most significant byte is the 0th byte  |
+-------------------------+-----+---+-----------------------------------------------------------------+
| shl(x, y)               |     | C | logical shift left y by x bits                                  |
+-------------------------+-----+---+-----------------------------------------------------------------+
| shr(x, y)               |     | C | logical shift right y by x bits                                 |
+-------------------------+-----+---+-----------------------------------------------------------------+
| sar(x, y)               |     | C | arithmetic shift right y by x bits                              |
+-------------------------+-----+---+-----------------------------------------------------------------+
| addmod(x, y, m)         |     | F | (x + y) % m with arbitrary precision arithmetic                 |
+-------------------------+-----+---+-----------------------------------------------------------------+
| mulmod(x, y, m)         |     | F | (x * y) % m with arbitrary precision arithmetic                 |
+-------------------------+-----+---+-----------------------------------------------------------------+
| signextend(i, x)        |     | F | sign extend from (i*8+7)th bit counting from least significant  |
+-------------------------+-----+---+-----------------------------------------------------------------+
| keccak256(p, n)         |     | F | keccak(mem[p...(p+n)))                                          |
+-------------------------+-----+---+-----------------------------------------------------------------+
| jump(label)             | `-` | F | jump to label / code position                                   |
+-------------------------+-----+---+-----------------------------------------------------------------+
| jumpi(label, cond)      | `-` | F | jump to label if cond is nonzero                                |
+-------------------------+-----+---+-----------------------------------------------------------------+
| pc                      |     | F | current position in code                                        |
+-------------------------+-----+---+-----------------------------------------------------------------+
| pop(x)                  | `-` | F | remove the element pushed by x                                  |
+-------------------------+-----+---+-----------------------------------------------------------------+
| dup1 ... dup16          |     | F | copy nth stack slot to the top (counting from top)              |
+-------------------------+-----+---+-----------------------------------------------------------------+
| swap1 ... swap16        | `*` | F | swap topmost and nth stack slot below it                        |
+-------------------------+-----+---+-----------------------------------------------------------------+
| mload(p)                |     | F | mem[p...(p+32))                                                 |
+-------------------------+-----+---+-----------------------------------------------------------------+
| mstore(p, v)            | `-` | F | mem[p...(p+32)) := v                                            |
+-------------------------+-----+---+-----------------------------------------------------------------+
| mstore8(p, v)           | `-` | F | mem[p] := v & 0xff (only modifies a single byte)                |
+-------------------------+-----+---+-----------------------------------------------------------------+
| sload(p)                |     | F | storage[p]                                                      |
+-------------------------+-----+---+-----------------------------------------------------------------+
| sstore(p, v)            | `-` | F | storage[p] := v                                                 |
+-------------------------+-----+---+-----------------------------------------------------------------+
| msize                   |     | F | size of memory, i.e. largest accessed memory index              |
+-------------------------+-----+---+-----------------------------------------------------------------+
| gas                     |     | F | gas still available to execution                                |
+-------------------------+-----+---+-----------------------------------------------------------------+
| address                 |     | F | address of the current contract / execution context             |
+-------------------------+-----+---+-----------------------------------------------------------------+
| balance(a)              |     | F | wei balance at address a                                        |
+-------------------------+-----+---+-----------------------------------------------------------------+
| caller                  |     | F | call sender (excluding ``delegatecall``)                        |
+-------------------------+-----+---+-----------------------------------------------------------------+
| callvalue               |     | F | wei sent together with the current call                         |
+-------------------------+-----+---+-----------------------------------------------------------------+
| calldataload(p)         |     | F | call data starting from position p (32 bytes)                   |
+-------------------------+-----+---+-----------------------------------------------------------------+
| calldatasize            |     | F | size of call data in bytes                                      |
+-------------------------+-----+---+-----------------------------------------------------------------+
| calldatacopy(t, f, s)   | `-` | F | copy s bytes from calldata at position f to mem at position t   |
+-------------------------+-----+---+-----------------------------------------------------------------+
| codesize                |     | F | size of the code of the current contract / execution context    |
+-------------------------+-----+---+-----------------------------------------------------------------+
| codecopy(t, f, s)       | `-` | F | copy s bytes from code at position f to mem at position t       |
+-------------------------+-----+---+-----------------------------------------------------------------+
| extcodesize(a)          |     | F | size of the code at address a                                   |
+-------------------------+-----+---+-----------------------------------------------------------------+
| extcodecopy(a, t, f, s) | `-` | F | like codecopy(t, f, s) but take code at address a               |
+-------------------------+-----+---+-----------------------------------------------------------------+
| returndatasize          |     | B | size of the last returndata                                     |
+-------------------------+-----+---+-----------------------------------------------------------------+
| returndatacopy(t, f, s) | `-` | B | copy s bytes from returndata at position f to mem at position t |
+-------------------------+-----+---+-----------------------------------------------------------------+
| extcodehash(a)          |     | C | code hash of address a                                          |
+-------------------------+-----+---+-----------------------------------------------------------------+
| create(v, p, n)         |     | F | create new contract with code mem[p...(p+n)) and send v wei     |
|                         |     |   | and return the new address                                      |
+-------------------------+-----+---+-----------------------------------------------------------------+
| create2(v, p, n, s)     |     | C | create new contract with code mem[p...(p+n)) at address         |
|                         |     |   | keccak256(0xff . this . s . keccak256(mem[p...(p+n)))           |
|                         |     |   | and send v wei and return the new address, where ``0xff`` is a  |
|                         |     |   | 8 byte value, ``this`` is the current contract's address        |
|                         |     |   | as a 20 byte value and ``s`` is a big-endian 256-bit value      |
+-------------------------+-----+---+-----------------------------------------------------------------+
| call(g, a, v, in,       |     | F | call contract at address a with input mem[in...(in+insize))     |
| insize, out, outsize)   |     |   | providing g gas and v wei and output area                       |
|                         |     |   | mem[out...(out+outsize)) returning 0 on error (eg. out of gas)  |
|                         |     |   | and 1 on success                                                |
+-------------------------+-----+---+-----------------------------------------------------------------+
| callcode(g, a, v, in,   |     | F | identical to ``call`` but only use the code from a and stay     |
| insize, out, outsize)   |     |   | in the context of the current contract otherwise                |
+-------------------------+-----+---+-----------------------------------------------------------------+
| delegatecall(g, a, in,  |     | H | identical to ``callcode`` but also keep ``caller``              |
| insize, out, outsize)   |     |   | and ``callvalue``                                               |
+-------------------------+-----+---+-----------------------------------------------------------------+
| staticcall(g, a, in,    |     | B | identical to ``call(g, a, 0, in, insize, out, outsize)`` but do |
| insize, out, outsize)   |     |   | not allow state modifications                                   |
+-------------------------+-----+---+-----------------------------------------------------------------+
| return(p, s)            | `-` | F | end execution, return data mem[p...(p+s))                       |
+-------------------------+-----+---+-----------------------------------------------------------------+
| revert(p, s)            | `-` | B | end execution, revert state changes, return data mem[p...(p+s)) |
+-------------------------+-----+---+-----------------------------------------------------------------+
| selfdestruct(a)         | `-` | F | end execution, destroy current contract and send funds to a     |
+-------------------------+-----+---+-----------------------------------------------------------------+
| invalid                 | `-` | F | end execution with invalid instruction                          |
+-------------------------+-----+---+-----------------------------------------------------------------+
| log0(p, s)              | `-` | F | log without topics and data mem[p...(p+s))                      |
+-------------------------+-----+---+-----------------------------------------------------------------+
| log1(p, s, t1)          | `-` | F | log with topic t1 and data mem[p...(p+s))                       |
+-------------------------+-----+---+-----------------------------------------------------------------+
| log2(p, s, t1, t2)      | `-` | F | log with topics t1, t2 and data mem[p...(p+s))                  |
+-------------------------+-----+---+-----------------------------------------------------------------+
| log3(p, s, t1, t2, t3)  | `-` | F | log with topics t1, t2, t3 and data mem[p...(p+s))              |
+-------------------------+-----+---+-----------------------------------------------------------------+
| log4(p, s, t1, t2, t3,  | `-` | F | log with topics t1, t2, t3, t4 and data mem[p...(p+s))          |
| t4)                     |     |   |                                                                 |
+-------------------------+-----+---+-----------------------------------------------------------------+
| origin                  |     | F | transaction sender                                              |
+-------------------------+-----+---+-----------------------------------------------------------------+
| gasprice                |     | F | gas price of the transaction                                    |
+-------------------------+-----+---+-----------------------------------------------------------------+
| blockhash(b)            |     | F | hash of block nr b - only for last 256 blocks excluding current |
+-------------------------+-----+---+-----------------------------------------------------------------+
| coinbase                |     | F | current mining beneficiary                                      |
+-------------------------+-----+---+-----------------------------------------------------------------+
| timestamp               |     | F | timestamp of the current block in seconds since the epoch       |
+-------------------------+-----+---+-----------------------------------------------------------------+
| number                  |     | F | current block number                                            |
+-------------------------+-----+---+-----------------------------------------------------------------+
| difficulty              |     | F | difficulty of the current block                                 |
+-------------------------+-----+---+-----------------------------------------------------------------+
| gaslimit                |     | F | block gas limit of the current block                            |
+-------------------------+-----+---+-----------------------------------------------------------------+

Literals
--------

10進数か16進数の表記をつけることにより整数の定数を使用することができます。そして、適切な ``PUSHi`` インストラクションが自動的に生成されます。下記のコードは2と3を足して5になり、文字列"abc"とのビット積をとります。
最終的な値はローカル変数の ``x`` に割り当てられます。
文字列は左詰めで保存され、32バイト以下でなければいけません。

.. code::

    assembly { let x := and("abc", add(3, 2)) }


Functional Style
-----------------

opcodeのシーケンスでは、あるopcodeの実際の引数が何であるか見るのが大変なことがままあります。下記の例では、位置 ``0x80`` にあるメモリの内容に ``3`` が足されます。

.. code::

    3 0x80 mload add 0x80 mstore

Solidityのインラインアセンブリは下記のように書かれる"ファンクショナルスタイル"の表記を持っています:

.. code::

    mstore(0x80, add(mload(0x80), 3))

右から左に読むと、完全に同じ定数とopcodeのシーケンスとなりますが、とても読みやすいです。

もしスタックのレイアウトが気になるなら、覚えておいて欲しいのは、シンタックス的にファンクションもしくはopcodeの最初の引数はスタックの一番上に置かれるということです。

Access to External Variables, Functions and Libraries
-----------------------------------------------------

Solidityの変数や他の識別子にはその名前でアクセスすることができます。
メモリに保存されている変数に関してはスタックに値ではなくアドレスをプッシュします。ストレージに保存されている変数はこれとは異なります。全てのストレージのスロットをおそらく占有しないので、"アドレス"はスロットとスロット内のバイトオフセットで構成されています。変数 ``x`` でポイントされているスロットを読み出すには ``x_slot`` を使用してください。バイトオフセットを読み出すには、``x_offset`` を使用してください。

Solidityのローカル変数は割り当て可能です。例えば:

.. code::

    pragma solidity >=0.4.11 <0.6.0;

    contract C {
        uint b;
        function f(uint x) public view returns (uint r) {
            assembly {
                r := mul(x, sload(b_slot)) // ignore the offset, we know it is zero
            }
        }
    }

.. warning::
    256ビット未満の型の変数（例えば、``uint64``、``address``、``bytes16``、``byte``）にアクセスする場合、型のエンコーディングの一部ではないビットに対してどんな想定もできません。特にゼロと想定してはいけません。
    安全のために、コンテキスト内で使う前に常にデータを適切にクリアしてください:
    ``uint32 x = f(); assembly { x := and(x, 0xffffffff) /* now use x */ }``
    符号付の型をクリアするのに ``signextend`` opcodeが使用できます。

Labels
------

ラベルのサポートはSolidity 0.5.0版で削除されました。
ファンクション、ループ、if文、switch文を代わりに使用してください。

Declaring Assembly-Local Variables
----------------------------------

変数を宣言するのに、``let`` キーワードを使用することができます。letはインラインアセンブリ内でのみ、実際には現在の ``{...}``-block内でのみのスコープです。
``let`` は新しい変数のためのスタックのスロットを作り、ブロックの最後に達したら自動的に削除されます。
その変数には初期値を与える必要があります。ただの ``0`` でも良いですが、複雑なファンクショナルスタイルの式でも構いません。

.. code::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function f(uint x) public view returns (uint b) {
            assembly {
                let v := add(x, 1)
                mstore(0x80, v)
                {
                    let y := add(sload(v), 1)
                    b := y
                } // y is "deallocated" here
                b := add(b, v)
            } // v is "deallocated" here
        }
    }


Assignments
-----------

値の割り当てはアセンブリのローカル変数とファンクションのローカル変数へ可能です。メモリやストレージの変数を割り当てる時に注意したいのは、それはポインタを変えるだけで、データを変えているわけではないということです。

1つの値に落ち着く式しか変数に割り当てることはできません。
もし複数の値を返すファンクションから受け取った値を変数に割り当てたいときは、複数の変数を用意しなければいけません。

.. code::

    {
        let v := 0
        let g := add(v, 2)
        function f() -> a, b { }
        let c, d := f()
    }

If
--

if文は条件分岐で使用できます。
"else"部分はありません。複数の条件分岐があるのであれば、"switch"の使用を検討してください（下記参照）。

.. code::

    {
        if eq(value, 0) { revert(0, 0) }
    }

ボディには波括弧が必要です。

Switch
------

基本的な"if/else"文として、switch文が使用可能です。
ある式の値をとって、それをいくつかの定数と比較します。
条件に合う定数の分岐が選ばれます。
間違いを起こしやすいいくつかのプログラミング言語とは異なり、操作フローはある条件から次の条件へコンテニューしません。フォールバックもしくは ``default`` というデフォルトの条件が使えます。

.. code::

    {
        let x := 0
        switch calldataload(4)
        case 0 {
            x := calldataload(0x24)
        }
        default {
            x := calldataload(0x44)
        }
        sstore(0, div(x, 2))
    }


条件のリストでは波括弧は必要ありませんが、ボディでは必要となります。

Loops
-----

アセンブリは単純なforループをサポートしています。forループは初期化部分、条件、イテレーション後のパートを含むヘッダを持っています。条件はファンクショナルスタイルの式である必要がありますが、他の2つはブロックです。初期化部分で変数を宣言した場合、この変数のスコープは本体（条件、イテレーション後のパートを含む）に拡張されます。

次の例ではメモリ内のある領域の合計を計算しています。

.. code::

    {
        let x := 0
        for { let i := 0 } lt(i, 0x100) { i := add(i, 0x20) } {
            x := add(x, mload(i))
        }
    }

　forループはwhileループの様に書くこともできます:単純に初期化部分とイテレーション後のパートを空にします。

.. code::

    {
        let x := 0
        let i := 0
        for { } lt(i, 0x100) { } {     // while(i < 0x100)
            x := add(x, mload(i))
            i := add(i, 0x20)
        }
    }

Functions
---------

アセンブリは低レベルファンクションの定義もできます。
このファンクションは引数（とリターンPC）をスタックからとってきます。また、スタックに結果をプットします。ファンクションの呼び出しはファンクショナルスタイルの実行のopcodeと同様な方法に見えます。

ファンクションはどこでも定義することができて、宣言されたブロック内がスコープになります。ファンクション内では、ファンクション外で宣言されたローカル変数にはアクセスできません。明示的な ``return`` の宣言はありません。

複数の値を返すファンクションをコールする場合、``a, b := f(x)`` or ``let a, b := f(x)`` を使って、その値をタプルに割り当てなければいけません。

下記の例では二乗と乗算を使った累乗の計算をしています。

.. code::

    {
        function power(base, exponent) -> result {
            switch exponent
            case 0 { result := 1 }
            case 1 { result := base }
            default {
                result := power(mul(base, base), div(exponent, 2))
                switch mod(exponent, 2)
                    case 1 { result := mul(base, result) }
            }
        }
    }

Things to Avoid
---------------

インラインアセンブリはかなり高いレベルに見えるかもしれませんが、実際は非常に低レベルです。ファンクションコール、ループ、if文、switch文は単純なルールの書き換えで変換されます。その後アセンブラがするのはファンクショナルスタイルのopcodeを再度並び替え、変数アクセスのためのスタックハイトを数え、ブロックの最後になったらアセンブリローカル変数のためのスタックのスロットを削除するということだけです。

Conventions in Solidity
-----------------------

EVMアセンブリと異なり、Solidityは ``uint24`` の様な256ビットより小さい型を識別できます。効率的にするために、ほとんどの算術演算はその型を256ビットの値として扱い、はみ出た分のビットは必要な時に処理します。例えば、メモリに書き込まれる直前や、比較演算がされる時です。つまり、もしインラインアセンブリ内の変数にアクセスする時には、まずマニュアルでそのはみ出たビットを処理する必要があるかもしれません。

Solidityはメモリはとてもシンプルな方法で管理しています。
メモリ内の ``0x40`` に"free memory pointer"があります。メモリを振り分ける時に、このポインタが示しているメモリから始めて、順にアップデートしていくだけです。
そのメモリが以前に使われていたかどうかの保証はないので、その中身がゼロであると仮定はできません。
メモリを解放したり、好きなところにメモリを割り当てる内蔵機能は付いていません。
下記はメモリの割り当てのアセンブリの例です::

    function allocate(length) -> pos {
      pos := mload(0x40)
      mstore(0x40, add(pos, length))
    }

メモリの最初の64バイトは短期間の割り当てのための"scratch space"として使うことができます。フリーメモリポインタ（ ``0x60`` から始まる様な）の32バイト後はずっと0であるはずで、空の動的メモリ配列の初期値として使われます。
つまり、割り当て可能なメモリは ``0x80`` から始まり、それはフリーメモリポインタの初期値です。

Solidityのメモリ配列の要素は常に32バイトの倍数を占めています（これは ``byte[]`` では同じですが、``bytes`` と ``string`` では違います）。多次元メモリ配列はメモリ配列のポインタです。動的配列の長さは配列の最初のスロットに保存され、その他に配列の要素が続きます。

.. warning::
    静的サイズのメモリ配列は長さの領域を持っていません。しかし、今後静的配列と動的配列の互換性のために追加されるかもしれません。


Standalone Assembly
===================

上記でインラインアセンブリとして紹介したアセンブリ言語はスタンドアローンとしても使用可能で、実はSolidityのコンパイラとの中間言語として使用する予定です。ここでは、いくつか達成したいことがあります:

1. その中で書かれたプログラムはSolidityからコンパイラを通じて生成されたとしても読んで理解できるものであるべきです。
2. アセンブリからバイトコードへの変換は可能な限り"サプライズ"を含まない様にするべきです。
3. 制御フローは形式的検証や最適化をしやすくするために、簡単に検知されるべきです。

最初と最後の目標を達成するためにアセンブリでは ``for`` ループ、``if``、``switch`` 文の様な高レベルの概念やファンクションコールが使えます。明示的な ``SWAP``、``DUP``、``JUMP``、``JUMPI`` を使わないアセンブリのプログラムを書くことが可能であるべきです。なぜなら初めの2つはデータフローを分かりにくくし、後の2つは制御フローを分かりにくくするからです。
さらに、``mul(add(x, y), 7)`` という形のファンクショナルステートメントは ``7 y x add mul`` の様なピュアなopcodeより好ましいです。なぜならば、最初の式の方がどっちのオペランドにどっちのopcodeを使うのか分かりやすいからです。

2つ目の目標に関しては、とても標準的な方法でより高いレベルの概念からバイトコードにコンパイルすることで達成します。
アセンブラによって行われる唯一の非ローカル演算は、ユーザー定義の識別子（ファンクション、変数など）の名前検索です。その演算ではとてもシンプルで標準的なスコープのルールに則り、また標準的な方法でスタックからのローカル変数の削除します。



Scoping: 宣言された識別子（ラベル、変数、ファンクション、アセンブリ）は宣言されたブロック内（現在のブロックの内側のネストされたブロックも含む）のみがスコープとなります。たとえスコープ内だったとしてもファンクションの垣根を超えてローカル変数にはアクセスできません。シャドーイングは使えません。
ローカル変数は宣言前には使えませんが、ファンクションやアセンブリは使用可能です。アセンブリはランタイムコードを返したり、コントラクトの作成に使われる特別なブロックです。アウターのアセンブリの識別子はサブアセンブリの中では使えません。

もし制御フローが最後のブロックを通ったら、ブロック内で宣言されたローカル変数の数だけポップの命令が挿入されます。
ローカル変数がリファレンスされる時は、コードジェネレータはスタック内でのその変数の相対的な位置を把握している必要があり、そのためいわゆるブロックハイトをトラックしています。ブロックの最後に全てのローカル変数は削除されるので、スタックハイトはブロックの前後で同じのはずです。もし同じでない場合、コンパイルは失敗します。

``switch``、``for``、ファンクションを使えば、手動で ``jump`` もしくは ``jumpi`` を使わずに複雑なコードが書けるはずです。これにより制御フローの分析がとても簡単になり、形式的検証や最適化が改善されます。

さらに、手動でのジャンプができると、スタックハイトの計算はむしろ複雑になります。全てのスタック上のローカル変数の位置は既知である必要あり、そうでなければ、ローカル変数への参照やブロックの最後に自動でスタックからローカル変数を削除する機能はちゃんと動作しません。

Example:

Solidityからアセンブリへのコンパイル例を見てみましょう。
下記のラインタイムバイトコードを考えます::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
      function f(uint x) public pure returns (uint y) {
        y = 1;
        for (uint i = 0; i < x; i++)
          y = 2 * y;
      }
    }

以下のアセンブリが生成されます::

    {
      mstore(0x40, 0x80) // store the "free memory pointer"
      // function dispatcher
      switch div(calldataload(0), exp(2, 226))
      case 0xb3de648b {
        let r := f(calldataload(4))
        let ret := $allocate(0x20)
        mstore(ret, r)
        return(ret, 0x20)
      }
      default { revert(0, 0) }
      // memory allocator
      function $allocate(size) -> pos {
        pos := mload(0x40)
        mstore(0x40, add(pos, size))
      }
      // the contract function
      function f(x) -> y {
        y := 1
        for { let i := 0 } lt(i, x) { i := add(i, 1) } {
          y := mul(2, y)
        }
      }
    }


Assembly Grammar
----------------

パーサのタスクは下記の通りです:

- バイトストリームをトークンストリームにし、C++スタイルのコメントを破棄します（参照ソースには特別なコメントがありますが、ここでは割愛します）。
- 下記のグラマーに従って、トークンストリームをASTにします。
- 識別子をその識別子が定義されたブロックと一緒に登録します（ASTノードへの注記）。どこから変数にアクセスできるか注意してください。

アセンブリの字句解析器はSolidityで定義されたものに従います。

ホワイトスペース（空白文字、タブ、改行）はトークンの範囲を決めるのに使用されます。コメントは標準のJavaScript/C++のコメントで、ホワイトスペースと同様に変換されます。

Grammar::

    AssemblyBlock = '{' AssemblyItem* '}'
    AssemblyItem =
        Identifier |
        AssemblyBlock |
        AssemblyExpression |
        AssemblyLocalDefinition |
        AssemblyAssignment |
        AssemblyStackAssignment |
        LabelDefinition |
        AssemblyIf |
        AssemblySwitch |
        AssemblyFunctionDefinition |
        AssemblyFor |
        'break' |
        'continue' |
        SubAssembly
    AssemblyExpression = AssemblyCall | Identifier | AssemblyLiteral
    AssemblyLiteral = NumberLiteral | StringLiteral | HexLiteral
    Identifier = [a-zA-Z_$] [a-zA-Z_0-9]*
    AssemblyCall = Identifier '(' ( AssemblyExpression ( ',' AssemblyExpression )* )? ')'
    AssemblyLocalDefinition = 'let' IdentifierOrList ( ':=' AssemblyExpression )?
    AssemblyAssignment = IdentifierOrList ':=' AssemblyExpression
    IdentifierOrList = Identifier | '(' IdentifierList ')'
    IdentifierList = Identifier ( ',' Identifier)*
    AssemblyStackAssignment = '=:' Identifier
    LabelDefinition = Identifier ':'
    AssemblyIf = 'if' AssemblyExpression AssemblyBlock
    AssemblySwitch = 'switch' AssemblyExpression AssemblyCase*
        ( 'default' AssemblyBlock )?
    AssemblyCase = 'case' AssemblyExpression AssemblyBlock
    AssemblyFunctionDefinition = 'function' Identifier '(' IdentifierList? ')'
        ( '->' '(' IdentifierList ')' )? AssemblyBlock
    AssemblyFor = 'for' ( AssemblyBlock | AssemblyExpression )
        AssemblyExpression ( AssemblyBlock | AssemblyExpression ) AssemblyBlock
    SubAssembly = 'assembly' Identifier AssemblyBlock
    NumberLiteral = HexNumber | DecimalNumber
    HexLiteral = 'hex' ('"' ([0-9a-fA-F]{2})* '"' | '\'' ([0-9a-fA-F]{2})* '\'')
    StringLiteral = '"' ([^"\r\n\\] | '\\' .)* '"'
    HexNumber = '0x' [0-9a-fA-F]+
    DecimalNumber = [0-9]+
