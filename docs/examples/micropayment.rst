********************
Micropayment Channel
********************

このセクションではペイメントチャンネルの実行をどうやって行うかを学びます。ここでは同じ人の間で繰り返し行われるEtherの送金を安全、高速かつトランザクションの手数料なしに行うために暗号化された署名を使います。
この例では、どの様に署名、検証し、そしてどの様にペイメントチャンネルをセットアップするかを理解する必要があります。

Creating and verifying signatures
=================================

アリスがボブにある量のEtherを送りたいという状況を想像してください。

アリスは暗号学的に署名されたメッセージをオフチェーン（例えばEメール）でボブに送る必要があります。これはチェックを書く時に似ています。

アリスとボブはトランザクションを許可するために署名を使います。これはEthereum上のスマートコントラクトで可能です。
アリスはEtherを送るシンプルなスマートコントラクトを作りました。しかし、その送金を始めるためのファンクションの呼び出しを自分で行う代わりに、ボブにそれを行わせ、トランザクションの手数料を払わせます。

そのコントラクトは次の様に動作します:

    1. アリスは最終的に行われる支払いをカバーするのに十分な量のEtherを付与した上で ``ReceiverPays`` コントラクトをデプロイします。
    2. アリスは秘密鍵による署名により支払いを許可します。
    3. アリスは暗号学的に署名されたメッセージをボブに送ります。そのメッセージは隠される必要はありません（後で説明します）し、この送信メカニズムは重要ではありません。
    4. コントラクトに署名されたメッセージを渡すことでボブは支払いを要求します。コントラクトはメッセージの有効性を確認の上、コントラクト上の資金を解放します。

Creating the signature
----------------------

アリスはトランザクションに署名するのにEthereumネットワークに繋げる必要はありません。このプロセスは完全にオフラインで行われます。このチュートリアルでは `web3.js <https://github.com/ethereum/web3.js>`_ と `MetaMask <https://metamask.io>`_ を使ってブラウザ上で署名を行います。さらに `EIP-762 <https://github.com/ethereum/EIPs/pull/712>`_ の中のメソッドで、他のセキュリティに関する恩恵を色々得られるメソッドを使用します。


::

    /// Hashing first makes things easier
    var hash = web3.utils.sha3("message to sign");
    web3.eth.personal.sign(hash, web3.eth.defaultAccount, function () { console.log("Signed"); });


.. note::
  ``web3.eth.personal.sign`` は署名されたデータの長さを表しています。最初にハッシュ化されているため、メッセージは常に32バイトの長さで、この長さの接頭辞は常に同じです。

What to Sign
------------

支払いを行うコントラクトでは署名されたメッセージは下記を含んでいる必要があります。

    1. 受領者のアドレス
    2. 送金額
    3. リプレイアタックに対する防御策

リプレイアタックは署名されたメッセージが承認を要求するのに再び使われることです。これを回避するためにEthereumのトランザクションと同じ様に、アカウントから送信されたトランザクションの数、いわゆるnonceを使用します。スマートコントラクトはnonceが何度も使われていないか確認します。

別のタイプのリプレイアタックはあるオーナーが支払いを行い、その後コントラクトを破棄する ``ReceiverPays`` というスマートコントラクトをデプロイした時に起こり得ます。その後、オーナーが再び ``RecipientPays`` をデプロイする時、その新しいコントラクトは前のデプロイでのnonceを知らないので、攻撃者は古いメッセージを再使用することができます。

アリスはメッセージの中にコントラクトのアドレスを含めることによりこの攻撃を防ぐことができます。さらにこの中ではコントラクトのアドレスを含んだメッセージだけが許可されます。このセクションの最後にあるコントラクトの中の ``claimPayment()`` ファンクションの中の最初の2行でこの例を確認できます。

Packing arguments
-----------------

署名されたメッセージの中にどんな情報を含める必要があるか分かったので、メッセージをまとめ、ハッシュ化し、署名する準備ができました。
シンプルにするためにこのデータを連結させます。
`ethereumjs-abi <https://github.com/ethereumjs/ethereumjs-abi>`_ ライブラリは ``soliditySHA3`` というファンクションを持っています。これは ``abi.encodePacked`` を使ってエンコードされた引数に適用されたSolidityの ``keccak256`` と同じ振る舞いをします。以下は ``ReceiverPays`` で適切な署名を作成するJavaScriptのファンクションの例です。

::

    // recipient is the address that should be paid.
    // amount, in wei, specifies how much ether should be sent.
    // nonce can be any unique number to prevent replay attacks
    // contractAddress is used to prevent cross-contract replay attacks
    function signPayment(recipient, amount, nonce, contractAddress, callback) {
        var hash = "0x" + abi.soliditySHA3(
            ["address", "uint256", "uint256", "address"],
            [recipient, amount, nonce, contractAddress]
        ).toString("hex");

        web3.eth.personal.sign(hash, web3.eth.defaultAccount, callback);
    }

Recovering the Message Signer in Solidity
-----------------------------------------

一般的にECDSA署名は ``r`` と ``s`` という2つのパラメータで構成されています。Ethereum上の署名は3つ目のパラメータ ``v`` を含んでいます。これは署名のどのアカウントの秘密鍵が使われたか、トランザクションの送信者が誰か検証するのに使われます。

Solidityは ``r``、``s`` そして ``v`` パラメータと一緒にメッセージを受け入れるビルトインファンクションの `ecrecover <mathematical-and-cryptographic-functions>`_ を持っています。さらにこのファンクションはメッセージに署名したアドレスを返します。

Extracting the Signature Parameters
-----------------------------------

web3.jsでされた署名は ``r``、``s``、``v`` が連結されたものなので、最初のステップはこれをそれぞれに分けることです。これはクライアント側でもできますが、スマートコントラクト内ですれば1つのパラメータだけで済みます。バイト配列を要素ごとに分けるのは見た目があまり良くないので、``splitSignature`` ファンクション（セクションの最後のフルコントラクトの3番目のファンクション）内でこの操作を行うために `inline assembly <assembly>`_ を使用します。




Computing the Message Hash
--------------------------

スマートコントラクトはどのパラメータがサインされたか知る必要あるので、スマートコントラクトはパラメータをメッセージから再作成し、それを署名の認証に使わなければなりません。``prefixed`` と ``recoverSigner`` ファンクションは ``claimPayment`` ファンクションの中でこれを行います。

The full contract
-----------------

::

    pragma solidity >=0.4.24 <0.6.0;

    contract ReceiverPays {
        address owner = msg.sender;

        mapping(uint256 => bool) usedNonces;

        constructor() public payable {}

        function claimPayment(uint256 amount, uint256 nonce, bytes memory signature) public {
            require(!usedNonces[nonce]);
            usedNonces[nonce] = true;

            // this recreates the message that was signed on the client
            bytes32 message = prefixed(keccak256(abi.encodePacked(msg.sender, amount, nonce, this)));

            require(recoverSigner(message, signature) == owner);

            msg.sender.transfer(amount);
        }

        /// destroy the contract and reclaim the leftover funds.
        function kill() public {
            require(msg.sender == owner);
            selfdestruct(msg.sender);
        }

        /// signature methods.
        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // first 32 bytes, after the length prefix.
                r := mload(add(sig, 32))
                // second 32 bytes.
                s := mload(add(sig, 64))
                // final byte (first byte of the next 32 bytes).
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// builds a prefixed hash to mimic the behavior of eth_sign.
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


Writing a Simple Payment Channel
================================

アリスは今、シンプルですが完全なペイメントチャンネルを作っています。ペイメントチャンネルは繰り返されるEtherのやり取りを安全、即時、かつトランザクション手数料なしで行うため、暗号学的な署名を使用しています。アリスとボブによるシンプルな間接的ペイメントチャンネルを考えてみましょう。


What is a Payment Channel?
--------------------------

ペイメントチャンネルは参加者にトランザクションを使用しないで何度もEtherのやり取りをできる様にしています。つまりトランザクションに関わる遅れや手数料が発生しないということです。


    1. アリスはあるコントラクトにEtherでお金を入れました。これでペイメントチャンネルが"開きます"。
    2. アリスは何Etherが受領者に受け渡されるか書いてあるメッセージに署名しました。このステップは支払いごとに繰り返されます。
    3. ボブは支払われたEtherを引き出し、残りを送金者に返しペイメントチャンネルを"閉じました"。

.. note::
  ステップ1と3だけトランザクションが必要で、ステップ2では送金者が受領者にオフチェーンの方法（例えばEmail）で署名されたメッセージを送っているということです。つまりたった2つのトランザクションだけでいくらでも送金が行えるということです。

  スマートコントラクトがエスクロー（第三者信託）としてEtherを扱い、そして有効に署名されたメッセージを引き受けているため、ボブはファンドされたお金を受け取れることが保証されています。スマートコントラクトは二人のペイメントチャンネルのタイムアウトを行うこともできるので、アリスは受領者がチャンネルのクローズを拒否してもお金が戻ってくることが保証されています。どのくらいペイメントチャンネルを開いておくかは参加者が決めることができます。短い期間のトランザクションでは例えばインターネットカフェで分ごとに課金される仕組みであったり、もっと長いもので言えば、時給で働く従業員への支払いに使えますし、ペイメントチャンネルは何ヶ月、何年もオープンにしておくことができます。

Opening the Payment Channel
---------------------------

ペイメントチャンネルを開くためにアリスはスマートコントラクトをデプロイしました。そのスマートコントラクトにはエスクローされるEtherを渡し、受領者とチャンネルの最大存続期間を決めました。これはこのセクションの最後にあるコントラストの中の ``SimplePaymentChannel`` ファンクションに入っています。

Making Payments
---------------

アリスはボブに署名されたメッセージを送ることで支払いを行います。このステップは完全にEthereumネットワークの外側で行われます。
メッセージは送信者により暗号化された署名が行われ、受領者に直接送られます。

それぞれのメッセージは以下の情報を含んでいます。
    * スマートコントラクトのアドレス（クロスコントラクト攻撃を防ぐため）
    * 現状受領者が受け取っているEtherの総額

ペイメントチャンネルは幾度と行われる送金の最後に一度だけクローズされます。
このため送られたメッセージの内、1つだけが履行されます。これが各マイクロペイメントの額ではなく累積額をメッセージにのせている理由です。最新のメッセージが最高額が書いてあるので、受領者は自然にそのメッセージを履行します。
スマートコントラクトは1つのメッセージだけを受け入れるため、メッセージごとのナンスはもう必要ありません。意図していた以外の他のチャンネルによって使われない様に、このスマートコントラクトのアドレスは使われたままです。

以下にメッセージに暗号学的に署名した前回のセクションから修正したJavaScriptを示します。

::

    function constructPaymentMessage(contractAddress, amount) {
        return abi.soliditySHA3(
            ["address", "uint256"],
            [contractAddress, amount]
        );
    }

    function signMessage(message, callback) {
        web3.eth.personal.sign(
            "0x" + message.toString("hex"),
            web3.eth.defaultAccount,
            callback
        );
    }

    // contractAddress is used to prevent cross-contract replay attacks.
    // amount, in wei, specifies how much Ether should be sent.

    function signPayment(contractAddress, amount, callback) {
        var message = constructPaymentMessage(contractAddress, amount);
        signMessage(message, callback);
    }


Closing the Payment Channel
---------------------------

ボブがチャンネルにあるお金を受け取る準備ができた時、スマートコントラクト内の ``close`` ファンクションを呼び出し、チャンネルをクローズする時間です。
チャンネルを閉じるときに、受領者に彼らがチャンネルに渡したEtherが支払われ、コントラクトは破棄されます。残っているEtherはアリスに返却されます。チャンネルを閉じるために、ボブはアリスによってサインされたメッセージを提供する必要があります。

スマートコントラクトはそのメッセージに送信者からの有効な署名がなされているか検証しなければなりません。この検証プロセスの目的は受領者が使うプロセスと同じです。このセクションの最後にあるSolidityの ``isValidSignature`` と ``recoverSigner`` ファンクションは ``ReceiverPays`` コントラクトから借りてきたファンクションと共に、これらに対応する前セクションのJavascriptのファンクションと同様な動きをします。

一番最近のペイメントのメッセージを送ったペイメントチャンネルの受領者のみが ``close`` ファンクションを呼ぶことができます。なぜならそのメッセージが一番高い合計の金額を持っているからです。もし送信者がこのファンクションを呼ぶ権限を持っていると、その送信者が低い総額のメッセージを作って、受領者が受け取るべき金額を改ざんできてしまいます。

そのファンクションは署名されたメッセージが与えられたパラメータと合っているか検証します。全ての検証が通ったら、受領者は取り分のEtherを受け取り、送信者が ``selfdestruct`` を通じて残りを受け取ります。フルコントラクト内で ``close`` ファンクションは見ることができます。


Channel Expiration
-------------------

ボブはペイメントチャンネルをいつでも閉じることができます。しかしチャンネルが閉じられなかった場合、アリスは第三者預託されたお金を回収する方法が必要になります。コントラクトのデプロイ時にコントラクトの失効期日がセットされます。その日になると、アリスはお金を回収するために ``claimTimeout`` を呼ぶことができます。``claimTimeout`` ファンクションはフルコントラクト内で確認できます。


The full contract
-----------------

::

    pragma solidity >=0.4.24 <0.6.0;

    contract SimplePaymentChannel {
        address payable public sender;      // The account sending payments.
        address payable public recipient;   // The account receiving the payments.
        uint256 public expiration;  // Timeout in case the recipient never closes.

        constructor (address payable _recipient, uint256 duration)
            public
            payable
        {
            sender = msg.sender;
            recipient = _recipient;
            expiration = now + duration;
        }

        function isValidSignature(uint256 amount, bytes memory signature)
            internal
            view
            returns (bool)
        {
            bytes32 message = prefixed(keccak256(abi.encodePacked(this, amount)));

            // check that the signature is from the payment sender
            return recoverSigner(message, signature) == sender;
        }

        /// the recipient can close the channel at any time by presenting a
        /// signed amount from the sender. the recipient will be sent that amount,
        /// and the remainder will go back to the sender
        function close(uint256 amount, bytes memory signature) public {
            require(msg.sender == recipient);
            require(isValidSignature(amount, signature));

            recipient.transfer(amount);
            selfdestruct(sender);
        }

        /// the sender can extend the expiration at any time
        function extend(uint256 newExpiration) public {
            require(msg.sender == sender);
            require(newExpiration > expiration);

            expiration = newExpiration;
        }

        /// if the timeout is reached without the recipient closing the channel,
        /// then the Ether is released back to the sender.
        function claimTimeout() public {
            require(now >= expiration);
            selfdestruct(sender);
        }

        /// All functions below this are just taken from the chapter
        /// 'creating and verifying signatures' chapter.

        function splitSignature(bytes memory sig)
            internal
            pure
            returns (uint8 v, bytes32 r, bytes32 s)
        {
            require(sig.length == 65);

            assembly {
                // first 32 bytes, after the length prefix
                r := mload(add(sig, 32))
                // second 32 bytes
                s := mload(add(sig, 64))
                // final byte (first byte of the next 32 bytes)
                v := byte(0, mload(add(sig, 96)))
            }

            return (v, r, s);
        }

        function recoverSigner(bytes32 message, bytes memory sig)
            internal
            pure
            returns (address)
        {
            (uint8 v, bytes32 r, bytes32 s) = splitSignature(sig);

            return ecrecover(message, v, r, s);
        }

        /// builds a prefixed hash to mimic the behavior of eth_sign.
        function prefixed(bytes32 hash) internal pure returns (bytes32) {
            return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
        }
    }


.. note::
  ``splitSignature`` ファンクションは全てのセキュリティチェックを使いません。本当の実行時にはもっと厳しくテストされたopenzepplin's `version  <https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/ECRecovery.sol>`_ の様なライブラリを使用するべきです。

Verifying Payments
------------------

前のセクションとは違い、ペイメントチャンネル内のメッセージはすぐには履行されません。受領者が最新のメッセージを確認し続け、ペイメントチャンネルを閉じるときにそのメッセージを履行します。つまり受領者が各メッセージの検証をすることが重要ということです。
そうしないと、受領者が最終的にお金を受け取れる保証がされなくなります。

受領者は下記のプロセスで各メッセージを検証すべきです。

    1. メッセージの中のコントラクトアドレスがペイメントチャンネルと合っているか検証してください。
    2. 新しい総額が予定していたものと同じか検証してください。
    3. 新しい総額が第三者預託されたEtherを超えていないか検証してください。
    4. 署名が有効か、そしてペイメントチャンネルの送信者からのものか検証してください。

この検証を記載するために `ethereumjs-util <https://github.com/ethereumjs/ethereumjs-util>`_ ライブラリを使用します。最終ステップは色々な方法で行うことができますが、ここではJavaScriptを使用します。次のコードは上記の **JavaScriptコード** の署名から `constructMessage` ファンクションを借りています。

::

    // this mimics the prefixing behavior of the eth_sign JSON-RPC method.
    function prefixed(hash) {
        return ethereumjs.ABI.soliditySHA3(
            ["string", "bytes32"],
            ["\x19Ethereum Signed Message:\n32", hash]
        );
    }

    function recoverSigner(message, signature) {
        var split = ethereumjs.Util.fromRpcSig(signature);
        var publicKey = ethereumjs.Util.ecrecover(message, split.v, split.r, split.s);
        var signer = ethereumjs.Util.pubToAddress(publicKey).toString("hex");
        return signer;
    }

    function isValidSignature(contractAddress, amount, signature, expectedSigner) {
        var message = prefixed(constructPaymentMessage(contractAddress, amount));
        var signer = recoverSigner(message, signature);
        return signer.toLowerCase() ==
            ethereumjs.Util.stripHexPrefix(expectedSigner).toLowerCase();
    }
