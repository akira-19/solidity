###############################
Introduction to Smart Contracts
###############################

.. _simple-smart-contract:

***********************
シンプルなスマートコントラクト
***********************

まずは値をセットし、他のコントラクトから呼び出せるような基本的なコントラクトの例から始めましょう。今は全てを理解する必要はありません。後ほど細かく説明します。

Storage
=======

::

    pragma solidity >=0.4.0 <0.6.0;

    contract SimpleStorage {
        uint storedData;

        function set(uint x) public {
            storedData = x;
        }

        function get() public view returns (uint) {
            return storedData;
        }
    }

最初の行はこのコードがSolidityバージョン0.4.0もしくはそれより新しいもので機能を損なわず動作するバージョンを簡潔に指定しています（0.6.0未満の最新バージョンまで）。これはこのコントラクトが新しいコンパイラーのバージョンで異なった動作（意図しない動作）が行われない様にするためです。いわゆるpragmaはソースコードをどの様に取り扱うかコンパイラーに指示するために一般的に使われるものです（例：`pragma once <https://en.wikipedia.org/wiki/Pragma_once>`_）。

Solidity上でのコントラクトというのはコード（その*function*たち）とEthereumブロックチェーン上の特定のアドレスに存在するデータ（その*状態*）の集合です。``uint storedData;``の行では``uint``（*u*\nsigned *int*\eger of *256* bits）型の``storedData``と呼ばれる状態変数を宣言しています。
データベース上で検索可能かつある機能を呼び出すことによって変更可能なデータの様なものだと思ってください。Ethereumの場合, コントラクトが常にそのデータを所有しています。今回の場合、``set`` と ``get``は値を修正もしくは取得する場合に使用されます。
状態変数にアクセスするのに他の言語でよく使われる ``this.``は必要ありません。
このコントラクトでは世界中誰でも数値を格納することが可能ということ以外には機能はありません。そしてあなたにはそれを防ぐ手段はありません。誰でも``set``を呼び出して好きな数字を上書きができます。しかし一度セットされた値はブロックチェーン上に残ります。あなただけが数値を変更できる様な制限を加える方法を後ほど説明します。

.. note::
    全ての識別子（コントラクト名、function名、変数名）はASCII文字に制限されています。UTF-8コードデータを文字列型に格納することは可能です。

.. warning::
    似た様な見た目のUnicodeテキストを使う際には注意してください。異なったバイト配列としてエンコードされてしまいます。

.. index:: ! subcurrency

Subcurrency Example
===================

以下のコントラクトは最も単純な仮想通貨を扱います。コインを0から生成することは可能ですが、コントラクトの作成者だけが実行可能です（異なる発行スキームの実行は簡単です）。さらにユーザー名やパスワードの登録無しに誰でもお互いにコインを送ることができます。必要なのはEthereumのキーペアだけです。

::

    pragma solidity ^0.5.0;
    contract Coin {
        // "public" というキーワードは値を
        // 外部から読み込み可能にさせます。
        address public minter;
        mapping (address => uint) public balances;
        // イベントは軽量クライアントが変更に対する反応を
        // 効率的に行うことを可能にします。
        event Sent(address from, address to, uint amount);
        // これはコントラクトが作られた時にだけ動作する
        // コンストラクタです。
        constructor() public {
            minter = msg.sender;
        }
        function mint(address receiver, uint amount) public {
            require(msg.sender == minter);
            require(amount < 1e60);
            balances[receiver] += amount;
        }
        function send(address receiver, uint amount) public {
            require(amount <= balances[msg.sender], "Insufficient balance.");
            balances[msg.sender] -= amount;
            balances[receiver] += amount;
            emit Sent(msg.sender, receiver, amount);
        }
    }

このコントラクトはいくつかの新しい機能が備わっていますので、一つずつ見ていきましょう。

``address public minter;``と書いてある行はパブリックにアクセス可能なアドレス型の変数を宣言しています。``address``型は160ビットの算術演算不可の値です。これはコントラクトのアドレスか外部の人間が持っているキーペアを保存するのに適しています。``public``というキーワードは自動的にコントラクトの外側から現在の状態変数の中身にアクセスできる様にする機能を生成します。
このキーワードなしでは他のコントラクトからはこの変数にアクセスできません。
コンパイラで生成されたこの機能は下記のコードとほぼイコールです（今は``external`` と ``view``は無視してください）::

    function minter() external view returns (address) { return minter; }


もちろんfunction名と状態変数が同じ名前のためこの様なfunctionを追加しても動きませんが、コンパイラがこの様に解釈するということを理解して頂けると幸いです。

.. index:: mapping

次の行の``mapping (address => uint) public balances;``は同様にパブリックな状態変数を生成しますが、もう少し複雑なデータタイプです。
これはaddressに符号無しのinteger型を割り当てます。Mappingは`hash table <https://en.wikipedia.org/wiki/Hash_table>`_として扱うことができます。そしてそれは事実上初期化され、そのため全てのpossible keyは最初から存在し、その値はバイト表現で0となります。しかし全く同じではありません。mappingではキーや値のリストを取得することはできません。そのため、何をmappingに追加したか覚えておいてください（もしくはリストを保存するか他の高度なデータタイプを使ってください）。もしくはそんなことをしなくて済む様な場合において使用して下さい。

今回の場合``public``で作られた:ref:`getter function<getter-functions>`はもう少し複雑でおおまかには下記の様になります::
    function balances(address _account) external view returns (uint) {
        return balances[_account];
    }

見ての通り、あるアカウントの残高をクエリするのにこのfunctionが利用できます。

.. index:: event

``event Sent(address from, address to, uint amount);``の行は``send``functionの最終行でemitされています、いわゆる"event"を宣言しています。ユーザーインターフェース（ともちろんサーバーサイドのアプリケーション）は多くのコストを支払わずにブロックチェーン上でemitされたそれらのイベントをリッスンすることができます。emitされるとすぐにlistenerは``from``、``to`` そして``amount``を引数として受け取り、トランザクションをトラックするのに役立ちます。このイベントをリッスンするために下記のJavaScriptコードを使います（``Coin`` はweb3.jsもしくは似た様なモジュールを用いて作られたコントラクトオブジェクトです。）::

Coin.Sent().watch({}, '', function(error, result) {
    if (!error) {
        console.log("Coin transfer: " + result.args.amount +
            " coins were sent from " + result.args.from +
            " to " + result.args.to + ".");
        console.log("Balances now:\n" +
            "Sender: " + Coin.balances.call(result.args.from) +
            "Receiver: " + Coin.balances.call(result.args.to));
    }
})

``balances`` functionがユーザーインターフェースから自動的にどの様に呼ばれるか確認してください。

.. index:: coin

コンストラクタはコントラクトが作成される時に1回だけ呼ばれる特別なfunctionです。このコンストラクタではコントラクトを作った人のアドレスを永久的に保存しています。``msg`` （``tx``と``block``と同じ様に）は特別なグローバル変数で、ブロックチェーンにアクセスできるいくつかのプロパティを含んでいます。``msg.sender``は外部からfunctionが呼んだアカウントのアドレスを常に返します。

コントラクトの最後にあり、ユーザもしくはコントラクトによって呼び出される``mint``と``send``です。
もし``mint`がコントラクトを作ったアカウント以外の誰かに呼ばれても何も起きません。これは特別なfunction``require``によって保証されています。これは引数がfalseだった場合に全ての変更を元に戻す機能を持っています。
2つ目の``require``は後にオーバーフローを起こす様な大量のコインがないことを保証しています。

一方で、``send``は誰にでも（コインを持っていれば）コインを誰かに送ることができます。送るのに十分なコインを持っていなかった場合、``require``はプロセスを中止し、適切なエラーメッセージの文字列を返します。

.. note::
    もしあなたがコインをどこかに送るためにこのコントラクトを使うのであれば、ブロックチェーンエクスプローラ上のアドレスを見た時に何も見ることができません。これはあなたがコインを送り、残高が変わったという事実はこの特定のコインコントラクトのデータストレージにのみ保存されるためです。イベントを使うことで比較的簡単にトランザクションと残高ををトラックする"ブロックチェーンエクスプローラ"を作成することが可能ですが、コインオーナーではなく、コントラクト作成者のあなたがコインコントラクトを検査する必要があります。

.. _blockchain-basics:

*****************
Blockchain Basics
*****************

ブロックチェーンのコンセプトを理解することはプログラマーにとってさほど難しいことではありません。その理由はほとんどの複雑な機能（mining, `hashing <https://en.wikipedia.org/wiki/Cryptographic_hash_function>`_, `elliptic-curve cryptography <https://en.wikipedia.org/wiki/Elliptic_curve_cryptography>`_, `peer-to-peer networks <https://en.wikipedia.org/wiki/Peer-to-peer>`_, etc.）はただプラットフォームに特徴と約束を与えているだけだからです。これらの特徴をそういうものとして受け入れれば、内部のテクノロジーについて心配する必要はありません。（AmazonのAWSを使うのに内部でどの様に動作しているか知る必要ありますか？）

.. index:: transaction

Transactions
============

ブロックチェーンはグローバルにシェアされたトランザクションのデータベースです。
つまり誰でもネットワークに接続するだけでこのデータベース上の項目を読み込むことができます。もしデータベース上の何かを変えたいときはいわゆるトランザクションを発行し、他の全員の同意を得る必要があります。トランザクションという単語はあなたがしたい変更が（例えばあなたが2つの値を同時に変えたいとすると）その両方ともが変わらないか、両方とも変更されることを意味しています。さらに、あなたのトランザクションがデータベースに登録されている最中に他のトランザクションはそのトランザクションを変更することができません。

例として、ある電子通貨の残高リストのテーブルを想像してください。もしあるアカウントから別のアカウントへの送金がリクエストされた際に、データベースのトランザクションの基本として、もしあるアカウントの残高から送金分が引かれたら、別のアカウントの残高には送金分が常に追加されなければいけません。何かの理由でその別のアカウント残高に送金分が追加されないのであれば、送金元のアカウントの残高も元のままでなければいけません。

更にトランザクションは常に送信者（作成者）によって暗号学的に署名されます。これによりデータベースのある種の改ざんを防ぐことができます。電子通貨の例で言えば、単純なチェックでキーを持っている人だけがお金を送ることができます。

.. index:: ! block

Blocks
======

解決しなければならない大きな問題の一つとして（Bitcoinの用語で）"二重支払い攻撃"があります。もしあるアカウントを空にする様な2つのトランザクションが同時に存在していたらどうなるでしょうか。基本的には最初に承認された最初のトランザクションのみが有効です。しかし問題は"最初の"というのはpeer-to-peerネットワークにおいて客観的ではないのです。

この問題に対する抽象的な答えは、気にする必要がないです。グローバルに承認された順番のトランザクションが選ばれ、このコンフリクトが解消します。いくつかのトランザクションはブロックと言われるもので一まとめにされ全ての参加しているノードの間で処理されます。
もし2つの矛盾したトランザクションがあった場合には、2つ目のトランザクションはリジェクトされブロックの一部として組み込まれることはありません。

これらのブロックは一つのシーケンスを作るためブロックチェーンという名前がつけられました。ブロックは定期的に追加され、Ethereumでは約17秒ごとに追加されます。

順序選択メカニズム（マイニング）では、ブロックが取り消されることもあります。しかしこれはチェーンの先端でだけで起こり、ブロックが追加されるごとに取り消される可能性が減ります。ですのであなたのトランザクションは取り消されるもしくは削除される可能性もありますが、長く待てば待つほどその可能性は低くなります。

.. note::
    トランザクションは次のブロックやある特定の未来のブロックに組み込まれる保証はありません。これはトランザクションを送った人にではなく、マイナーにどのトランザクションをブロックに組み込むかの権限があるためです。

    もしあなたのコントラクトである未来の時間でコールしたい場合には`alarm clock <http://www.ethereum-alarm-clock.com/>`_もしくは似た様なoracleのサービスが使用可能です。

.. _the-ethereum-virtual-machine:

.. index:: !evm, ! ethereum virtual machine

****************************
The Ethereum Virtual Machine
****************************

Overview
========

Ethereum Virtual Machine（EVM）はEthereum上のスマートこコントラクトのためのruntime環境です。サンドボックス化されているだけでなく、実際には完全に独立しています。つまりEVM内部のコードはネットワークやファイルシステム、または他のプロセスにアクセスしません。
スマートコントラクトですら他のスマートコントラクトへのアクセスは制限されています。

.. index:: ! account, address, storage, balance

Accounts
========

Ethereumには2種類のアカウントがあります。両方とも同じアドレスを共有しています。**外部アカウント**は公開・秘密鍵のペアで管理されており、**コントラクトアカウント**はアカウントと一緒に保存されたコードによってコントロールされています。

外部アカウントのアドレスは公開鍵から決まる一方で、コントラクトのアドレスはコントラクトが作られた時に決まります。（コントラクトの作成者のアドレスと送られたトランザクションの数いわゆる"nonce"によって決まります。）

アカウントがコードを保存するかどうかに関わらず、EVMはこの2つのタイプを同様に扱います。

全てのアカウントは**storage**という256ビットのワードで256ビットのワードを保存するkey-value store mappingを持っている。

さらに、全てのアカウントは**balance**をEther（"Wei"でいうと`1 ether`は`10**18 wei`です）で持っており、Etherを含んだトランザクションを送ることでこの値を変えることができます。

.. index:: ! transaction

Transactions
============

トランザクションはあるアカウントから別のアカウント（これは同じアカウントもしくは空のアカウントの場合もある。下記をご参照ください）へのメッセージです。これはバイナリーデータ（"payload"と呼ばれます）とEtherを含んでいます。

送信先のアカウントがコードを含んでいた場合、そのコードは実行され、payloadはインプットデータとして提供されます。

もし送信先のアカウントがセットされていなかったら（トランザクションがrecipientを持っていないか、recipientが``null``だった場合には）、トランザクションは**new contract**を生成します。先にも言及した通り、コントラクトのアドレスはゼロアドレスではなく送信者やトランザクションの数（nonce）によって決まります。 The payload
of such a contract creation transaction is taken to be
EVM bytecode and executed. この実行のアウトプットデータはコントラクトのコードとして永久的に保存されます。
これが意味するのはコントラクトを生成するために実際のコントラクトのコードを送るのではなく、コードが実行された時にそのコードを返すコードを送っています。

.. note::
    コントラクトが作られている間、そのコードはまだ空です。このため、you should not call back into the
    contract under construction until its constructor has
    finished executing.

.. index:: ! gas, ! gas price

Gas
===

トランザクションの生成にあたり、各トランザクションはある量の**gas**を要求します。この目的は必要な処理の量を制限し、この処理に対しての報酬を同時に行うためです。EVMがトランザクションを実行している間、gasは段々とあるルールに則り、depleteしていきます。

**gas price**とはトランザクションの作成者によってセットされる値であり、この作成者は``gas_price * gas``を送信するアカウントから支払う必要があります。もしトランザクションの実行後にgasが残っていたら、作成者に返金されます。

もしgasはある値より多く使われたら（負の値になりえます）、gas不足の例外が投げられ、現在の呼び出されたフレーム内での変更は全てrevertされます。

.. index:: ! storage, ! memory, ! stack

Storage, Memory and the Stack
=============================

Ethereum Virtual Machineはデータを保存できる場所が3つあります。それはstorage、memory、stackです。以下で説明していきます。

各アカウントは**storage**と呼ばれるデータエリアを持っており、which is persistent between function calls
and transactions.
Storage is a key-value store that maps 256-bit words to 256-bit words.
It is not possible to enumerate storage from within a contract and it is
comparatively costly to read, and even more to modify storage.
A contract can neither read nor write to any storage apart from its own.

2つ目のデータエリアは**memory**と呼ばれ、コントラクトは各message callに対するクリアされたインスタンスを取得します。memoryはlinearであり、バイトレベルでaddressされますが、読み取りは256bitに制限され、書き込みは8もしくは256bitに制限されます。過去に変更がないmemoryの単語（例えばany offset
within a word）にアクセスした際にmemoryは256-bitの単語に拡張されます。拡張の際にはgasは支払われます。memoryは成長すればするほど高くなリマス。（(it scales quadratically)）

EVMは登録機械ではなくstack machineです。そのため全ての計算は**stack**と呼ばれるデータエリアで行われます。最大1024要素であり、256-bitの単語を含見ます。stackへのアクセスは下記のようにtop endに制限されます。
It is possible to copy one of
the topmost 16 elements to the top of the stack or swap the
topmost element with one of the 16 elements below it.
All other operations take the topmost two (or one, or more, depending on
the operation) elements from the stack and push the result onto the stack.
Of course it is possible to move stack elements to storage or memory
in order to get deeper access to the stack,
but it is not possible to just access arbitrary elements deeper in the stack
without first removing the top of the stack.

.. index:: ! instruction

Instruction Set
===============

EVMのinstruction setは間違った、もしくはinconsistentなコンセンサス問題を起こしうる実行を避けるために最小限に保たれています。
全てのinstructionは基本的なデータタイプ、256-bitのワードもしくはmemory（もしくは他のバイト配列）の元で成り立っています。

基本的な算術、ビット、論理、比較計算は存在しています。Conditional and unconditional jumps are possible。更にコントラクトはブロック番号やタイムスタンプの様な関連するpropertyにアクセスできます。

For a complete list, please see the :ref:`list of opcodes <opcodes>` as part of the inline
assembly documentation.

.. index:: ! message call, function;call

Message Calls
=============

コントラクトは他のコントラクトを呼び出したり、Etherをコントラクトアカウントではないアカウントにmessage callを使って送ることができます。message callはソース、送信先、data payload、Ether、gas、return dataがある点でトランザクションに似ています。実際に全てのトランザクションはtop-level message call which in turn can create further message calls.

コントラクトは残っている**gas**をどれだけ送るか、そしてどのくらい残すかを内部のmessage callで決めることができます。内部呼び出しでgas不足の例外（もしくは他の例外）が発生したら、スタックに追加されることによりエラーが伝えられます。この場合、呼び出しと一緒に送られたgasのみが使用されます。その様な状況においてSolidityではデフォルトでコントラクトの呼び出しは手動の例外を起こし、例外は呼び出しのスタックから呼び出されます。so that exceptions "bubble up" the call stack.

既に議論した様に、呼び出されたコントラクト（呼び出し元と同じになる場合もあります）はクリアされたmemoryのインスタンスを受け取り、**calldata**と呼ばれる別のエリアにあるcall payloadにアクセスできます。
このコントラクト実行後に、このコントラクトは呼び出し元が事前に割り振ったmemoryの場所に保存されていたデータを返します。
これら全ての呼び出しは同時に起きます。

呼び出しは1024の深さに**制限**される。これが意味するのはもっと複雑な運用においてループ処理は再帰的な呼び出しより好まれます。更に、63/64番目のgasだけはmessage callの中に送られるため実際には深さは1000より少し小さくなります。

.. index:: delegatecall, callcode, library

Delegatecall / Callcode and Libraries
=====================================

**delegatecall**と呼ばれる特別なmessage callのvariantがあります。これはmessage callと同じですが、送信先のアドレスのコードが呼び出し元のコントラクトのcontextで実行されるということと``msg.sender``と``msg.value``はその値を変えません。

つまりコントラクトは動的に違うアドレスからコードをロードできるということです。Storage、つまり現在のアドレスとバランスはまだ呼び出し元のコントラクトを参照していますが、コードだけが呼び出されたアドレスから取得されています。

これはSolidityにおいて"library"機能を実装可能としています。例えば複雑なデータ構造を実行するために、再利用可能なlibraryのコードをコントラクトのstorageに保存できます。

.. index:: log

Logs
====

特別にインデックスされ、ブロックレベルで全てマッピングされたデータ構造の中にデータを保存することができます。この**logs**と呼ばれるfeatureは:ref:`events <events>`を実行するためにSolidityによって使用されています。コントラクトは作成後はログデータにアクセスできませんがブロックチェーンの外側から効率的にアクセスできます。いくつかのログデータは`bloom filters <https://en.wikipedia.org/wiki/Bloom_filter>`_に保存されるため、このデータは効率的かつcryptographicallyに安全な方法で検索できます。そのためブロックチェーン全てをダウンロードしていないネットワーク上のpeer（いわゆる"light clients"）でもこれらのログを見つけることがきます。

.. index:: contract creation

Create
======

コントラクトは特別なopcodeを使って他のコントラクトを作ることもできます（i.e.
they do not simply call the zero address as a transaction would）。これら**create calls**と通常のmessage callの唯一の違いはpayloadデータが実行され、結果がコードとして保存され、呼び出し元と作成者がスタック上にある新しいコントラクトのアドレスを受け取ります。

.. index:: selfdestruct, self-destruct, deactivate

Deactivate and Self-destruct
============================

ブロックチェーン からコードを削除する唯一の手段はコントラクトが``selfdestruct``を実行する時のみです。そのアドレスに残っているEtherが設定されていた送信先に送られた時にstorageとコードは削除されます。コントラクトの削除は理論上は良いアイデアの様に聞こえますが、潜在的に危険を孕んでいます。誰かがコントラクトを削除するためにEtherを送り、そのEtherは永遠に失われるかの様、、、、、、、

.. note::
    もしコントラクトのコードが``selfdestruct``を含んでいなかったとしても、``delegatecall``もしくは``callcode``を使うことで実行可能です。

もしコントラクトを無効化したいのであれば、代わりに全てのfunctionsを元に戻させる内部の状態（機能）を変更することでコントラクトを無効化すべきです。これによりコントラクトがEtherを返すとすぐにそのコントラクトを使えなくします。

.. warning::
    もしコントラクトを"selfdestruct"で削除したとしても、ブロックチェーン上の履歴には残りますし、きっとほぼ全てのノードにより保持されます。つまり"selfdestruct"はハードディスクからデータを消すのとは違うということです。
