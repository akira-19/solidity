.. index:: ! contract;creation, constructor

******************
Creating Contracts
******************

コントラクトはEthreumトランザクションを通じて"外部から"、もしくはSolidityコントラクトの内側から作成可能です。

`Remix <https://remix.ethereum.org/>`_ の様なIDEはUIの要素を使って、シームレスに生成プロセスを作ります。

Ethreum上にプログラムでコントラクトを作成するにはJavaScript API `web3.js <https://github.com/ethereum/web3.js>`_ を使うのがベストです。
web3.jsは簡単にコントラクトを作るための `web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_ というファンクションを持っています。

コントラクトが作られた時、:ref:`constructor <constructor>`  ( ``constructor`` キーワードで宣言されるファンクション)が一度だけ実行されます。

コンストラクタはオプションです。1つのコンストラクタだけ使えます。つまりオーバーロードはサポートされていません。

コンストラクタが実行された後、最終的なコントラクトのコードがブロックチェーン上にデプロイされます。このコードは全てのpublic、externalのファンクションを含んでおり、全てのファンクションはファンクションコールを通じてそのコードからアクセスできます。デプロイされたコードはコンストラクタもしくはコンストラクタによって呼ばれたinternalのファンクションだけ含んでいません。

.. index:: constructor;arguments

内部的にはコントラクトのコードの後、コンストラクタの引数は :ref:`ABI encoded <ABI>` されて渡されますが、もし ``web3.js`` を使っているのであればきにする必要はありません。

もしコントラクトが別のコントラクトを作りたい場合、作成されたコントラクトのソースコード（とそのバイナリ）は作成者によって把握されている必要があります。
つまり、cyclic creation dependenciesは出来ないということです。

::

    pragma solidity >=0.4.22 <0.6.0;

    contract OwnedToken {
        // `TokenCreator` is a contract type that is defined below.
        // It is fine to reference it as long as it is not used
        // to create a new contract.
        TokenCreator creator;
        address owner;
        bytes32 name;

        // This is the constructor which registers the
        // creator and the assigned name.
        constructor(bytes32 _name) public {
            // State variables are accessed via their name
            // and not via e.g. `this.owner`. Functions can
            // be accessed directly or through `this.f`,
            // but the latter provides an external view
            // to the function. Especially in the constructor,
            // you should not access functions externally,
            // because the function does not exist yet.
            // See the next section for details.
            owner = msg.sender;

            // We do an explicit type conversion from `address`
            // to `TokenCreator` and assume that the type of
            // the calling contract is `TokenCreator`, there is
            // no real way to check that.
            creator = TokenCreator(msg.sender);
            name = _name;
        }

        function changeName(bytes32 newName) public {
            // Only the creator can alter the name --
            // the comparison is possible since contracts
            // are explicitly convertible to addresses.
            if (msg.sender == address(creator))
                name = newName;
        }

        function transfer(address newOwner) public {
            // Only the current owner can transfer the token.
            if (msg.sender != owner) return;

            // We ask the creator contract if the transfer
            // should proceed by using a function of the
            // `TokenCreator` contract defined below. If
            // the call fails (e.g. due to out-of-gas),
            // the execution also fails here.
            if (creator.isTokenTransferOK(owner, newOwner))
                owner = newOwner;
        }
    }

    contract TokenCreator {
        function createToken(bytes32 name)
           public
           returns (OwnedToken tokenAddress)
        {
            // Create a new `Token` contract and return its address.
            // From the JavaScript side, the return type is
            // `address`, as this is the closest type available in
            // the ABI.
            return new OwnedToken(name);
        }

        function changeName(OwnedToken tokenAddress, bytes32 name) public {
            // Again, the external type of `tokenAddress` is
            // simply `address`.
            tokenAddress.changeName(name);
        }

        // Perform checks to determine if transferring a token to the
        // `OwnedToken` contract should proceed
        function isTokenTransferOK(address currentOwner, address newOwner)
            public
            pure
            returns (bool ok)
        {
            // Check an arbitrary condition to see if transfer should proceed
            return keccak256(abi.encodePacked(currentOwner, newOwner))[0] == 0x7f;
        }
    }
