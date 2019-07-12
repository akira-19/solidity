###
Yul
###

.. _yul:

.. index:: ! assembly, ! asm, ! evmasm, ! yul, julia, iulia

Yul(以前JULIAやIULIAと呼ばれていたものです)は複数の異なるバックエンド(EVM1.0やEVM1.5、eWASM)でコンパイルできる中間言語です。
そのため、Yulは3つのプラットフォームすべての使用可能な共通分母になるように設計されています。
YulはすでにSolidity内の"inline assembly"としても使用されており、Solidityコンパイラの将来のバージョンではYulを中間言語として使用できるようになります。
また、Yul用の高水準オプティマイザステージを構築するのは簡単です。

.. note::

    "inline assembly"として使用されるときに型を持つわけではありません(すべて ``u256`` になります)。
    そして、組み込み関数はEVMオペコードと同じです。詳細についてはinline assemblyのドキュメントを参照してください。

Yulのコアコンポーネントは関数、ブロック、変数、リテラル、forループ、if文、switch文、式と変数への代入です。

Yulは型付けされ、変数とリテラルの両方が後置記法で型を指定しなければなりません。
サポートしている型は、 ``bool``、 ``u8``、 ``s8``、 ``u32``、 ``s32``、 ``u64``、 ``s64``、 ``u128``、 ``s128``、 ``u256``、 ``s256`` です。

Yul自体は、演算子すら持ちません。もしあるEVMが指定された場合、オペコードは組み込み関数として利用可能になりますが、バックエンドが変更された場合は再実装することができます。
マンダトリーなビルドイン関数のリストは下のセクションを参照ください。

次のプログラムの例では、EVMのオペコード ``mul``、 ``div``、および ``mod`` がネイティブでも関数としても利用可能であると仮定して、べき乗を計算します。

.. code::

    {
        function power(base:u256, exponent:u256) -> result:u256
        {
            switch exponent
            case 0:u256 { result := 1:u256 }
            case 1:u256 { result := base }
            default
            {
                result := power(mul(base, base), div(exponent, 2:u256))
                switch mod(exponent, 2:u256)
                    case 1:u256 { result := mul(base, result) }
            }
        }
    }


また、再帰ではなくforループを使用して同じ関数を実装することも可能です。
ここで、利用可能なEVMオペコードの ``lt`` (less-than)と ``add`` が必要になります。

.. code::

    {
        function power(base:u256, exponent:u256) -> result:u256
        {
            result := 1:u256
            for { let i := 0:u256 } lt(i, exponent) { i := add(i, 1:u256) }
            {
                result := mul(result, base)
            }
        }
    }

Specification of Yul
====================

この章では、Yulのコードを記述します。通常、Yulオブジェクトの中に置かれるものです。これについては次の章で説明します。

Grammar::

    Block = '{' Statement* '}'
    Statement =
        Block |
        FunctionDefinition |
        VariableDeclaration |
        Assignment |
        If |
        Expression |
        Switch |
        ForLoop |
        BreakContinue
    FunctionDefinition =
        'function' Identifier '(' TypedIdentifierList? ')'
        ( '->' TypedIdentifierList )? Block
    VariableDeclaration =
        'let' TypedIdentifierList ( ':=' Expression )?
    Assignment =
        IdentifierList ':=' Expression
    Expression =
        FunctionCall | Identifier | Literal
    If =
        'if' Expression Block
    Switch =
        'switch' Expression ( Case+ Default? | Default )
    Case =
        'case' Literal Block
    Default =
        'default' Block
    ForLoop =
        'for' Block Expression Block Block
    BreakContinue =
        'break' | 'continue'
    FunctionCall =
        Identifier '(' ( Expression ( ',' Expression )* )? ')'
    Identifier = [a-zA-Z_$] [a-zA-Z_$0-9]*
    IdentifierList = Identifier ( ',' Identifier)*
    TypeName = Identifier | BuiltinTypeName
    BuiltinTypeName = 'bool' | [us] ( '8' | '32' | '64' | '128' | '256' )
    TypedIdentifierList = Identifier ':' TypeName ( ',' Identifier ':' TypeName )*
    Literal =
        (NumberLiteral | StringLiteral | HexLiteral | TrueLiteral | FalseLiteral) ':' TypeName
    NumberLiteral = HexNumber | DecimalNumber
    HexLiteral = 'hex' ('"' ([0-9a-fA-F]{2})* '"' | '\'' ([0-9a-fA-F]{2})* '\'')
    StringLiteral = '"' ([^"\r\n\\] | '\\' .)* '"'
    TrueLiteral = 'true'
    FalseLiteral = 'false'
    HexNumber = '0x' [0-9a-fA-F]+
    DecimalNumber = [0-9]+

Restrictions on the Grammar
---------------------------

Switch文はdefault caseを含む最低でも1つのcaseを持ちます。
もし式のすべての可能な値がカバーされている場合、defaultのcaseは許容するべきではありません(すなわち、真と偽の両方のケースを持つ ``bool`` を持つSwitch文)。
また、すべてのcaseの値は同じ型である必要があります。

すべての式はゼロ以上の値に評価されます。識別子とリテラルは厳密に1つの値であると評価され、関数呼び出しは呼び出された関数の戻り値の数と等しい数の値であると評価されます。

変数宣言時や代入時、右辺の式は、左辺の変数の数と等しい数の値であると評価される必要があります。
これが、複数の値に評価される式が許可される唯一の状況です。

ステートメント(すなわち、ブロックレベル)でもある式は、0として評価される必要があります。
そして、その他のすべてのシチュエーションにおいて、式は単一の値として評価する必要があります。

 ``continue`` や ``break`` 文はループ文中でのみ使用でき、ループ文と同じ関数内になければなりません(もしくは両方ともトップレベルになければなりません)。

for文の条件は、単一の値で評価される必要があります。

リテラルは、その型以上に大きくなることはありません。最大値の型は、256ビット長であることが決められています。

Scoping Rules
-------------

Yulのスコープはブロックと結びついており(関数とforループは例外です)、すべての宣言( ``FunctionDefinition``、 ``VariableDeclaration``)はこれらのスコープに新たな識別子をもたらします。

識別子は、定義されたブロック内で表示することができます(すべてのサブノートやサブブロックを含みます)。
例外として、forループの"init"部分(最初のブロック)で定義された識別子は、forループの他のすべての部分で表示することができます(ただし、ループの外側では表示できません)。
関数のパラメータと戻り値は関数本体に表示され、それらの名前は同じものを使用することはできません。

変数は宣言後に参照することができます。特に、変数はそれ自身の変数宣言の右側では参照できません。
関数は宣言前にすでに参照できます(可視性である場合に限ります)。

シャドーイングは使用できません。つまり、たとえアクセスできない場合でも、同じ名前の別の識別子も表示されている場所で識別子を宣言することはできません。

また、関数の外側で宣言された変数にアクセスすることはできません。

Formal Specification
--------------------

私たちは、ASTのさまざまなノードにオーバーロードされた評価関数Eを提供して、Yulを形式的に指定します。
どの関数にも副作用がある可能性があるため、評価関数Eは2つのステートオブジェクトとASTノードを取り、2つの新しいステートオブジェクトと可変数の他の値を返します。
2つの状態オブジェクトとはグローバルステートオブジェクト(EVMのコンテキストではブロックチェーンのメモリ、ストレージ、およびステート)とローカルステートオブジェクト(ローカル変数のステート、つまりEVM内のスタックのセグメント)です。

もしASTノードがステートメントである場合、評価関数Eは2つのステートオブジェクトと、 ``break`` と ``continue`` に使用される "mode"を返します。
もしASTノードが式である場合、評価関数Eは2つのステートオブジェクトと式が評価する同じ数の値を返します。

グローバルステートの正確な性質は、この上位レベルの説明では規定されていません。
ローカルステート ``L`` は、識別子 ``i`` から値 ``v`` へのマッピングで、``L[i] = v`` と表されます。

識別子 ``v`` の場合、 ``$v`` を識別子の名前とします。

また、ASTノードには分割表記を使用します。

.. code::

    E(G, L, <{St1, ..., Stn}>: Block) =
        let G1, L1, mode = E(G, L, St1, ..., Stn)
        let L2 be a restriction of L1 to the identifiers of L
        G1, L2, mode
    E(G, L, St1, ..., Stn: Statement) =
        if n is zero:
            G, L, regular
        else:
            let G1, L1, mode = E(G, L, St1)
            if mode is regular then
                E(G1, L1, St2, ..., Stn)
            otherwise
                G1, L1, mode
    E(G, L, FunctionDefinition) =
        G, L, regular
    E(G, L, <let var1, ..., varn := rhs>: VariableDeclaration) =
        E(G, L, <var1, ..., varn := rhs>: Assignment)
    E(G, L, <let var1, ..., varn>: VariableDeclaration) =
        let L1 be a copy of L where L1[$vari] = 0 for i = 1, ..., n
        G, L1, regular
    E(G, L, <var1, ..., varn := rhs>: Assignment) =
        let G1, L1, v1, ..., vn = E(G, L, rhs)
        let L2 be a copy of L1 where L2[$vari] = vi for i = 1, ..., n
        G, L2, regular
    E(G, L, <for { i1, ..., in } condition post body>: ForLoop) =
        if n >= 1:
            let G1, L1, mode = E(G, L, i1, ..., in)
            // 構文上の制限のため、modeは規則的でなければなりません
            let G2, L2, mode = E(G1, L1, for {} condition post body)
            // 構文上の制限のため、modeは規則的でなければなりません
            let L3 be the restriction of L2 to only variables of L
            G2, L3, regular
        else:
            let G1, L1, v = E(G, L, condition)
            if v is false:
                G1, L1, regular
            else:
                let G2, L2, mode = E(G1, L, body)
                if mode is break:
                    G2, L2, regular
                else:
                    G3, L3, mode = E(G2, L2, post)
                    E(G3, L3, for {} condition post body)
    E(G, L, break: BreakContinue) =
        G, L, break
    E(G, L, continue: BreakContinue) =
        G, L, continue
    E(G, L, <if condition body>: If) =
        let G0, L0, v = E(G, L, condition)
        if v is true:
            E(G0, L0, body)
        else:
            G0, L0, regular
    E(G, L, <switch condition case l1:t1 st1 ... case ln:tn stn>: Switch) =
        E(G, L, switch condition case l1:t1 st1 ... case ln:tn stn default {})
    E(G, L, <switch condition case l1:t1 st1 ... case ln:tn stn default st'>: Switch) =
        let G0, L0, v = E(G, L, condition)
        // i = 1 .. n
        // コンテキストに関係なくリテラルを評価します
        let _, _, v1 = E(G0, L0, l1)
        ...
        let _, _, vn = E(G0, L0, ln)
        if there exists smallest i such that vi = v:
            E(G0, L0, sti)
        else:
            E(G0, L0, st')

    E(G, L, <name>: Identifier) =
        G, L, L[$name]
    E(G, L, <fname(arg1, ..., argn)>: FunctionCall) =
        G1, L1, vn = E(G, L, argn)
        ...
        G(n-1), L(n-1), v2 = E(G(n-2), L(n-2), arg2)
        Gn, Ln, v1 = E(G(n-1), L(n-1), arg1)
        Let <function fname (param1, ..., paramn) -> ret1, ..., retm block>
        be the function of name $fname visible at the point of the call.
        Let L' be a new local state such that
        L'[$parami] = vi and L'[$reti] = 0 for all i.
        Let G'', L'', mode = E(Gn, L', block)
        G'', Ln, L''[$ret1], ..., L''[$retm]
    E(G, L, l: HexLiteral) = G, L, hexString(l),
        where hexString decodes l from hex and left-aligns it into 32 bytes
    E(G, L, l: StringLiteral) = G, L, utf8EncodeLeftAligned(l),
        where utf8EncodeLeftAligned performs a utf8 encoding of l
        and aligns it left into 32 bytes
    E(G, L, n: HexNumber) = G, L, hex(n)
        where hex is the hexadecimal decoding function
    E(G, L, n: DecimalNumber) = G, L, dec(n),
        where dec is the decimal decoding function

Type Conversion Functions
-------------------------

Yulは暗黙的型変換をサポートしていないため、明示的変換を提供するための関数が存在します。
大きな型からより小さな型へ変換するとき、オーバーフローの場合にruntime exceptionが発生する可能性があります。

以下の型間での変換の切り捨てがサポートされています:
 - ``bool``
 - ``u32``
 - ``u64``
 - ``u256``
 - ``s256``

これらのそれぞれに対して、型変換関数は、 ``u32tobool(x:u32) -> y:bool``、 ``u256tou32(x:u256) -> y:u32`` や ``s256tou256(x:s256) -> y:u256`` などといった ``<input_type>to<output_type>(x:<input_type>) -> y:<output_type>`` 形式のプロトタイプを持ちます。

.. note::

    ``u32tobool(x:u32) -> y:bool`` は ``y := not(iszerou256(x))`` として実行され、
    ``booltou32(x:bool) -> y:u32`` は ``switch x case true:bool { y := 1:u32 } case false:bool { y := 0:u32 }`` として実行されます。

Low-level Functions
-------------------

以下の関数が利用可能でなければなりません:

+---------------------------------------------------------------------------------------------------------------+
| *Logic*                                                                                                       |
+---------------------------------------------+-----------------------------------------------------------------+
| not(x:bool) -> z:bool                       | logical not                                                     |
+---------------------------------------------+-----------------------------------------------------------------+
| and(x:bool, y:bool) -> z:bool               | logical and                                                     |
+---------------------------------------------+-----------------------------------------------------------------+
| or(x:bool, y:bool) -> z:bool                | logical or                                                      |
+---------------------------------------------+-----------------------------------------------------------------+
| xor(x:bool, y:bool) -> z:bool               | xor                                                             |
+---------------------------------------------+-----------------------------------------------------------------+
| *Arithmetic*                                                                                                  |
+---------------------------------------------+-----------------------------------------------------------------+
| addu256(x:u256, y:u256) -> z:u256           | x + y                                                           |
+---------------------------------------------+-----------------------------------------------------------------+
| subu256(x:u256, y:u256) -> z:u256           | x - y                                                           |
+---------------------------------------------+-----------------------------------------------------------------+
| mulu256(x:u256, y:u256) -> z:u256           | x * y                                                           |
+---------------------------------------------+-----------------------------------------------------------------+
| divu256(x:u256, y:u256) -> z:u256           | x / y                                                           |
+---------------------------------------------+-----------------------------------------------------------------+
| divs256(x:s256, y:s256) -> z:s256           | x / y, for signed numbers in two's complement                   |
+---------------------------------------------+-----------------------------------------------------------------+
| modu256(x:u256, y:u256) -> z:u256           | x % y                                                           |
+---------------------------------------------+-----------------------------------------------------------------+
| mods256(x:s256, y:s256) -> z:s256           | x % y, for signed numbers in two's complement                   |
+---------------------------------------------+-----------------------------------------------------------------+
| signextendu256(i:u256, x:u256) -> z:u256    | sign extend from (i*8+7)th bit counting from least significant  |
+---------------------------------------------+-----------------------------------------------------------------+
| expu256(x:u256, y:u256) -> z:u256           | x to the power of y                                             |
+---------------------------------------------+-----------------------------------------------------------------+
| addmodu256(x:u256, y:u256, m:u256) -> z:u256| (x + y) % m with arbitrary precision arithmetic                 |
+---------------------------------------------+-----------------------------------------------------------------+
| mulmodu256(x:u256, y:u256, m:u256) -> z:u256| (x * y) % m with arbitrary precision arithmetic                 |
+---------------------------------------------+-----------------------------------------------------------------+
| ltu256(x:u256, y:u256) -> z:bool            | true if x < y, false otherwise                                  |
+---------------------------------------------+-----------------------------------------------------------------+
| gtu256(x:u256, y:u256) -> z:bool            | true if x > y, false otherwise                                  |
+---------------------------------------------+-----------------------------------------------------------------+
| lts256(x:s256, y:s256) -> z:bool            | true if x < y, false otherwise                                  |
|                                             | (for signed numbers in two's complement)                        |
+---------------------------------------------+-----------------------------------------------------------------+
| gts256(x:s256, y:s256) -> z:bool            | true if x > y, false otherwise                                  |
|                                             | (for signed numbers in two's complement)                        |
+---------------------------------------------+-----------------------------------------------------------------+
| equ256(x:u256, y:u256) -> z:bool            | true if x == y, false otherwise                                 |
+---------------------------------------------+-----------------------------------------------------------------+
| iszerou256(x:u256) -> z:bool                | true if x == 0, false otherwise                                 |
+---------------------------------------------+-----------------------------------------------------------------+
| notu256(x:u256) -> z:u256                   | ~x, every bit of x is negated                                   |
+---------------------------------------------+-----------------------------------------------------------------+
| andu256(x:u256, y:u256) -> z:u256           | bitwise and of x and y                                          |
+---------------------------------------------+-----------------------------------------------------------------+
| oru256(x:u256, y:u256) -> z:u256            | bitwise or of x and y                                           |
+---------------------------------------------+-----------------------------------------------------------------+
| xoru256(x:u256, y:u256) -> z:u256           | bitwise xor of x and y                                          |
+---------------------------------------------+-----------------------------------------------------------------+
| shlu256(x:u256, y:u256) -> z:u256           | logical left shift of x by y                                    |
+---------------------------------------------+-----------------------------------------------------------------+
| shru256(x:u256, y:u256) -> z:u256           | logical right shift of x by y                                   |
+---------------------------------------------+-----------------------------------------------------------------+
| sars256(x:s256, y:u256) -> z:u256           | arithmetic right shift of x by y                                |
+---------------------------------------------+-----------------------------------------------------------------+
| byte(n:u256, x:u256) -> v:u256              | nth byte of x, where the most significant byte is the 0th byte  |
|                                             | Cannot this be just replaced by and256(shr256(n, x), 0xff) and  |
|                                             | let it be optimised out by the EVM backend?                     |
+---------------------------------------------+-----------------------------------------------------------------+
| *Memory and storage*                                                                                          |
+---------------------------------------------+-----------------------------------------------------------------+
| mload(p:u256) -> v:u256                     | mem[p..(p+32))                                                  |
+---------------------------------------------+-----------------------------------------------------------------+
| mstore(p:u256, v:u256)                      | mem[p..(p+32)) := v                                             |
+---------------------------------------------+-----------------------------------------------------------------+
| mstore8(p:u256, v:u256)                     | mem[p] := v & 0xff    - only modifies a single byte             |
+---------------------------------------------+-----------------------------------------------------------------+
| sload(p:u256) -> v:u256                     | storage[p]                                                      |
+---------------------------------------------+-----------------------------------------------------------------+
| sstore(p:u256, v:u256)                      | storage[p] := v                                                 |
+---------------------------------------------+-----------------------------------------------------------------+
| msize() -> size:u256                        | size of memory, i.e. largest accessed memory index, albeit due  |
|                                             | due to the memory extension function, which extends by words,   |
|                                             | this will always be a multiple of 32 bytes                      |
+---------------------------------------------+-----------------------------------------------------------------+
| *Execution control*                                                                                           |
+---------------------------------------------+-----------------------------------------------------------------+
| create(v:u256, p:u256, n:u256)              | create new contract with code mem[p..(p+n)) and send v wei      |
|                                             | and return the new address                                      |
+---------------------------------------------+-----------------------------------------------------------------+
| create2(v:u256, p:u256, n:u256, s:u256)     | create new contract with code mem[p...(p+n)) at address         |
|                                             | keccak256(0xff . this . s . keccak256(mem[p...(p+n)))           |
|                                             | and send v wei and return the new address, where ``0xff`` is a  |
|                                             | 8 byte value, ``this`` is the current contract's address        |
|                                             | as a 20 byte value and ``s`` is a big-endian 256-bit value      |
+---------------------------------------------+-----------------------------------------------------------------+
| call(g:u256, a:u256, v:u256, in:u256,       | call contract at address a with input mem[in..(in+insize))      |
| insize:u256, out:u256,                      | providing g gas and v wei and output area                       |
| outsize:u256)                               | mem[out..(out+outsize)) returning 0 on error (eg. out of gas)   |
| -> r:u256                                   | and 1 on success                                                |
+---------------------------------------------+-----------------------------------------------------------------+
| callcode(g:u256, a:u256, v:u256, in:u256,   | identical to ``call`` but only use the code from a              |
| insize:u256, out:u256,                      | and stay in the context of the                                  |
| outsize:u256) -> r:u256                     | current contract otherwise                                      |
+---------------------------------------------+-----------------------------------------------------------------+
| delegatecall(g:u256, a:u256, in:u256,       | identical to ``callcode``,                                      |
| insize:u256, out:u256,                      | but also keep ``caller``                                        |
| outsize:u256) -> r:u256                     | and ``callvalue``                                               |
+---------------------------------------------+-----------------------------------------------------------------+
| abort()                                     | abort (equals to invalid instruction on EVM)                    |
+---------------------------------------------+-----------------------------------------------------------------+
| return(p:u256, s:u256)                      | end execution, return data mem[p..(p+s))                        |
+---------------------------------------------+-----------------------------------------------------------------+
| revert(p:u256, s:u256)                      | end execution, revert state changes, return data mem[p..(p+s))  |
+---------------------------------------------+-----------------------------------------------------------------+
| selfdestruct(a:u256)                        | end execution, destroy current contract and send funds to a     |
+---------------------------------------------+-----------------------------------------------------------------+
| log0(p:u256, s:u256)                        | log without topics and data mem[p..(p+s))                       |
+---------------------------------------------+-----------------------------------------------------------------+
| log1(p:u256, s:u256, t1:u256)               | log with topic t1 and data mem[p..(p+s))                        |
+---------------------------------------------+-----------------------------------------------------------------+
| log2(p:u256, s:u256, t1:u256, t2:u256)      | log with topics t1, t2 and data mem[p..(p+s))                   |
+---------------------------------------------+-----------------------------------------------------------------+
| log3(p:u256, s:u256, t1:u256, t2:u256,      | log with topics t, t2, t3 and data mem[p..(p+s))                |
| t3:u256)                                    |                                                                 |
+---------------------------------------------+-----------------------------------------------------------------+
| log4(p:u256, s:u256, t1:u256, t2:u256,      | log with topics t1, t2, t3, t4 and data mem[p..(p+s))           |
| t3:u256, t4:u256)                           |                                                                 |
+---------------------------------------------+-----------------------------------------------------------------+
| *State queries*                                                                                               |
+---------------------------------------------+-----------------------------------------------------------------+
| blockcoinbase() -> address:u256             | current mining beneficiary                                      |
+---------------------------------------------+-----------------------------------------------------------------+
| blockdifficulty() -> difficulty:u256        | difficulty of the current block                                 |
+---------------------------------------------+-----------------------------------------------------------------+
| blockgaslimit() -> limit:u256               | block gas limit of the current block                            |
+---------------------------------------------+-----------------------------------------------------------------+
| blockhash(b:u256) -> hash:u256              | hash of block nr b - only for last 256 blocks excluding current |
+---------------------------------------------+-----------------------------------------------------------------+
| blocknumber() -> block:u256                 | current block number                                            |
+---------------------------------------------+-----------------------------------------------------------------+
| blocktimestamp() -> timestamp:u256          | timestamp of the current block in seconds since the epoch       |
+---------------------------------------------+-----------------------------------------------------------------+
| txorigin() -> address:u256                  | transaction sender                                              |
+---------------------------------------------+-----------------------------------------------------------------+
| txgasprice() -> price:u256                  | gas price of the transaction                                    |
+---------------------------------------------+-----------------------------------------------------------------+
| gasleft() -> gas:u256                       | gas still available to execution                                |
+---------------------------------------------+-----------------------------------------------------------------+
| balance(a:u256) -> v:u256                   | wei balance at address a                                        |
+---------------------------------------------+-----------------------------------------------------------------+
| this() -> address:u256                      | address of the current contract / execution context             |
+---------------------------------------------+-----------------------------------------------------------------+
| caller() -> address:u256                    | call sender (excluding delegatecall)                            |
+---------------------------------------------+-----------------------------------------------------------------+
| callvalue() -> v:u256                       | wei sent together with the current call                         |
+---------------------------------------------+-----------------------------------------------------------------+
| calldataload(p:u256) -> v:u256              | call data starting from position p (32 bytes)                   |
+---------------------------------------------+-----------------------------------------------------------------+
| calldatasize() -> v:u256                    | size of call data in bytes                                      |
+---------------------------------------------+-----------------------------------------------------------------+
| calldatacopy(t:u256, f:u256, s:u256)        | copy s bytes from calldata at position f to mem at position t   |
+---------------------------------------------+-----------------------------------------------------------------+
| codesize() -> size:u256                     | size of the code of the current contract / execution context    |
+---------------------------------------------+-----------------------------------------------------------------+
| codecopy(t:u256, f:u256, s:u256)            | copy s bytes from code at position f to mem at position t       |
+---------------------------------------------+-----------------------------------------------------------------+
| extcodesize(a:u256) -> size:u256            | size of the code at address a                                   |
+---------------------------------------------+-----------------------------------------------------------------+
| extcodecopy(a:u256, t:u256, f:u256, s:u256) | like codecopy(t, f, s) but take code at address a               |
+---------------------------------------------+-----------------------------------------------------------------+
| extcodehash(a:u256)                         | code hash of address a                                          |
+---------------------------------------------+-----------------------------------------------------------------+
| *Others*                                                                                                      |
+---------------------------------------------+-----------------------------------------------------------------+
| discard(unused:bool)                        | discard value                                                   |
+---------------------------------------------+-----------------------------------------------------------------+
| discardu256(unused:u256)                    | discard value                                                   |
+---------------------------------------------+-----------------------------------------------------------------+
| splitu256tou64(x:u256) -> (x1:u64, x2:u64,  | split u256 to four u64's                                        |
| x3:u64, x4:u64)                             |                                                                 |
+---------------------------------------------+-----------------------------------------------------------------+
| combineu64tou256(x1:u64, x2:u64, x3:u64,    | combine four u64's into a single u256                           |
| x4:u64) -> (x:u256)                         |                                                                 |
+---------------------------------------------+-----------------------------------------------------------------+
| keccak256(p:u256, s:u256) -> v:u256         | keccak(mem[p...(p+s)))                                          |
+---------------------------------------------+-----------------------------------------------------------------+
| *Object access*                             |                                                                 |
+---------------------------------------------+-----------------------------------------------------------------+
| datasize(name:string) -> size:u256          | size of the data object in bytes, name has to be string literal |
+---------------------------------------------+-----------------------------------------------------------------+
| dataoffset(name:string) -> offset:u256      | offset of the data object inside the data area in bytes,        |
|                                             | name has to be string literal                                   |
+---------------------------------------------+-----------------------------------------------------------------+
| datacopy(dst:u256, src:u256, len:u256)      | copy len bytes from the data area starting at offset src bytes  |
|                                             | to memory at position dst                                       |
+---------------------------------------------+-----------------------------------------------------------------+

Backends
--------

バックエンドやターゲットはYulから特定のバイトコードへのtranslatorとなります。各バックエンドは、バックエンドの名前を接頭辞として持つ関数を公開することができます。
また、バックエンドのために ``evm`` と ``ewasm`` の接頭辞を予約語としています。

Backend: EVM
------------

EVMターゲットは ``evm_`` 接頭辞で公開されているすべての基底のEVMオペコードを持ちます。

Backend: "EVM 1.5"
------------------

TBD

Backend: eWASM
--------------

TBD

Specification of Yul Object
===========================

Yulオブジェクトは、名前付きコードとデータセクションをグループ化するために使用されます。
関数 ``datasize`` 、 ``dataoffset`` および ``datacopy`` は、コード内からこれらのセクションにアクセスするために使用できます。
また、16進数エンコーディングでデータを指定し、ネイティブエンコーディングで通常の文字列を指定するために16進数の文字列を使用できます。
コードの場合、 ``datacopy`` はassembled binary representationにアクセスします。

Grammar::

    Object = 'object' StringLiteral '{' Code ( Object | Data )* '}'
    Code = 'code' Block
    Data = 'data' StringLiteral ( HexLiteral | StringLiteral )
    HexLiteral = 'hex' ('"' ([0-9a-fA-F]{2})* '"' | '\'' ([0-9a-fA-F]{2})* '\'')
    StringLiteral = '"' ([^"\r\n\\] | '\\' .)* '"'

以上において、``Block`` は前の章で説明したYulコード文法の ``Block`` を表します。

Yulオブジェクトの例を以下に示します:

.. code::

    // コードは単一のオブジェクトで構成されています。単一の "code"ノードはオブジェクトのコードです。
    // すべての（他の）名前付きオブジェクトまたはデータセクションはシリアライズされ、
    // 特別な組み込み関数 datacopy/dataoffset/datasize にアクセスできるようになります。
    // ネストされたオブジェクトへのアクセスは"."を使って名前を結合することによって可能です。
    // 現在のオブジェクト、現在のオブジェクト内のサブオブジェクト、およびデータ項目は、ネストアクセスなしでスコープ内にあります。
    object "Contract1" {
        code {
            // 最初に "runtime.Contract2" を作成します
            let size = datasize("runtime.Contract2")
            let offset = allocate(size)
            // 以下はeWASMの場合はmemory-> memory、
            // EVMの場合は "runtime.Contract2"のコードコピーになります
            datacopy(offset, dataoffset("runtime.Contract2"), size)
            // コンストラクタパラメータは単一値0x1234になります
            mstore(add(offset, size), 0x1234)
            create(offset, add(size, 32))

            // これでランタイムオブジェクトを返すようになりました（これはコンストラクタコードです）
            size := datasize("runtime")
            offset := allocate(size)
            // 以下はeWASMの場合はmemory-> memory、EVMの場合はcodecopyになります。
            datacopy(offset, dataoffset("runtime"), size)
            return(offset, size)
        }

        data "Table2" hex"4123"

        object "runtime" {
            code {
                // ランタイムコード

                let size = datasize("Contract2")
                let offset = allocate(size)
                // 以下はeWASMの場合はmemory-> memory、EVMの場合はcodecopyになります。
                datacopy(offset, dataoffset("Contract2"), size)
                // コンストラクタパラメータは単一値0x1234になります
                mstore(add(offset, size), 0x1234)
                create(offset, add(size, 32))
            }

            // 埋め込みオブジェクト。ユースケースとして、オブジェクト外においてファクトリコントラクトであり、
            // Contract2がファクトリによって作成されるコードです。
            object "Contract2" {
                code {
                    // コード...
                }

                object "runtime" {
                    code {
                        // コード...
                    }
                 }

                 data "Table1" hex"4123"
            }
        }
    }
