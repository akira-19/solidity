.. index:: ! library, callcode, delegatecall

.. _libraries:

*********
Libraries
*********

ライブラリはコントラクトに似ていますが、ある特定のアドレスに一度だけデプロイされ、``DELEGATECALL`` (Homesteadまでは ``CALLCODE``)を使って、そのコードを再利用するのが目的です。つまり、ライブラリのファンクションが呼び出されたら、呼び出したコントラクトのコンテキストで実行されます。``this`` は呼び出したコントラクトを指しますし、特に呼び出し元のコントラクトのストレージもアクセス可能です。ライブラリは独立したソースコードなので、状態変数が明示的に渡された場合、呼び出したコントラクトの状態変数にアクセスできます（そうでないと名前をつけられません）。
ライブラリのファンクションはステートを変更しない場合（ ``view`` もしくは ``pure`` の場合）、直接呼び出すことしかできません（つまり ``DELEGATECALL`` を使わない）。なぜなら、ライブラリはステートレスと想定されているからです。つまり、ライブラリを破棄することはできません。

.. note::
    バージョン0.4.20までは、Solidityのタイプシステムを迂回することで、ライブラリの破棄が可能でした。
    そのバージョンから、ライブラリがステートを変更するファンクションを直接呼び出す :ref:`mechanism<call-protection>` を導入しました（つまり ``DELEGATECALL`` を使わない）。


ライブラリはコントラクトが使う暗示的なベースコントラクトと見做すことができます。ライブラリは継承の階層の中で明示的に可視ではないですが、ライブラリのファンクションを呼ぶのは、明示的なベースファンクションを呼ぶ様なものです（ ``L`` がライブラリの名前の時に ``L.f()`` ）。さらに、ライブラリの ``internal`` ファンクションはまるでライブラリがベースコントラクトであるかの様に、全てのコントラクトからアクセスできます。もちろんinternalのファンクションを呼び出すにはinternalの呼び出しのルールに従う必要があります。それは全てのinternalタイプは受け渡されて、:ref:`メモリに保存 <data-location>` された型は参照として渡され、コピーされないということです。
EVMでこれを実現するために、internalのライブラリファンクションのコードとその中から呼ばれるファンクションはコンパイル時に呼び出し元のコントラクトに入り、``DELEGATECALL`` の代わりに通常の ``JUMP`` コールが使用されます。

.. index:: using for, set

次の例ではどの様にライブラリを使うかを説明します（マニュアルメソッドに関してはより詳細な例 :ref:`using for <using-for>` を参照ください）。

::

    pragma solidity >=0.4.22 <0.6.0;

    library Set {
      // We define a new struct datatype that will be used to
      // hold its data in the calling contract.
      struct Data { mapping(uint => bool) flags; }

      // Note that the first parameter is of type "storage
      // reference" and thus only its storage address and not
      // its contents is passed as part of the call.  This is a
      // special feature of library functions.  It is idiomatic
      // to call the first parameter `self`, if the function can
      // be seen as a method of that object.
      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
              return false; // already there
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // not there
          self.flags[value] = false;
          return true;
      }

      function contains(Data storage self, uint value)
          public
          view
          returns (bool)
      {
          return self.flags[value];
      }
    }

    contract C {
        Set.Data knownValues;

        function register(uint value) public {
            // The library functions can be called without a
            // specific instance of the library, since the
            // "instance" will be the current contract.
            require(Set.insert(knownValues, value));
        }
        // In this contract, we can also directly access knownValues.flags, if we want.
    }

もちろん、ライブラリを使うのにこの方法に従う必要はありません。構造体型を定義しなくても使えます。ファンクションはストレージの参照型を使わなくても動きますし、どこでも複数のストレージの参照型を持つことができます。

``Set.contains``、``Set.insert``、``Set.remove`` の呼び出しは外部のコントラクト、ライブラリの呼び出し(``DELEGATECALL``)としてコンパイルされます。もしライブラリを使うなら、1つの実際のexternalのファンクションコールが実行されることを覚えておいてください。``msg.sender``、``msg.value``、``this`` はその値をこのコール中に保持します（ ``CALLCODE``、``msg.sender``、``msg.value`` の使用方法が変わったため、Homestead以前で有効です）。

以下の例では、externalファンクションコールのオーバーヘッドなしでカスタム型を実行するために、:ref:`メモリに保存される型 <data-location>` とライブラリのinternalファンクションをどの様に使用するか示しています。

::

    pragma solidity >=0.4.16 <0.6.0;

    library BigInt {
        struct bigint {
            uint[] limbs;
        }

        function fromUint(uint x) internal pure returns (bigint memory r) {
            r.limbs = new uint[](1);
            r.limbs[0] = x;
        }

        function add(bigint memory _a, bigint memory _b) internal pure returns (bigint memory r) {
            r.limbs = new uint[](max(_a.limbs.length, _b.limbs.length));
            uint carry = 0;
            for (uint i = 0; i < r.limbs.length; ++i) {
                uint a = limb(_a, i);
                uint b = limb(_b, i);
                r.limbs[i] = a + b + carry;
                if (a + b < a || (a + b == uint(-1) && carry > 0))
                    carry = 1;
                else
                    carry = 0;
            }
            if (carry > 0) {
                // too bad, we have to add a limb
                uint[] memory newLimbs = new uint[](r.limbs.length + 1);
                uint i;
                for (i = 0; i < r.limbs.length; ++i)
                    newLimbs[i] = r.limbs[i];
                newLimbs[i] = carry;
                r.limbs = newLimbs;
            }
        }

        function limb(bigint memory _a, uint _limb) internal pure returns (uint) {
            return _limb < _a.limbs.length ? _a.limbs[_limb] : 0;
        }

        function max(uint a, uint b) private pure returns (uint) {
            return a > b ? a : b;
        }
    }

    contract C {
        using BigInt for BigInt.bigint;

        function f() public pure {
            BigInt.bigint memory x = BigInt.fromUint(7);
            BigInt.bigint memory y = BigInt.fromUint(uint(-1));
            BigInt.bigint memory z = x.add(y);
            assert(z.limb(1) > 0);
        }
    }

ライブラリがどこにデプロイされるかコンパイラは分からないので、アドレスはlinkerで最後のバイトコードに入れられる必要があります（リンキングのためのコマンドラインコンパイラの使い方は :ref:`commandline-compiler` を参照下さい）。もしアドレスが引数としてコンパイラに渡されない場合、コンパイラの16進数コードは ``__Set______`` (
``Set`` はライブラリの名前) のプレースホルダを含みます。ライブラリコントラクトのアドレスの16進数エンコーディングによって、アドレスは手動で40個の記号で置き換えられ、埋められます。

.. note::
    生成されたバイトコード上でのライブラリの手動のリンキングは推奨されません。なぜなら、36文字に制限されているからです。コントラクトが ``solc`` の ``--libraries`` オプションか、コンパイラにstandard-JSONインターフェースを使っているなら、``libraries`` キーを使ってコンパイルされている時に、コンパイラにライブラリをリンクすることをお願いした方が良いです。

コントラクトと比べた時のライブラリの制限は:

- 状態変数がありません
- 継承する、継承されることはありません
- Etherを受け取ることができません

(これらはいつか撤廃されるかもしれません)

.. _call-protection:

Call Protection For Libraries
=============================

イントロダクションで言及した通り、ライブラリのコードが ``DELEGATECALL`` もしくは ``CALLCODE`` の代わりに ``CALL`` で実行された時、``view`` か ``pure`` でなければrevertします。

EVMではコントラクトが ``CALL`` で呼ばれたかどうか直接検知する方法はありませんが、そのコードが現在"どこ"で動作しているか調べる ``ADDRESS`` opcodeは使えます。生成されたコードは呼び出しの種類を決めるために、このアドレスとコード生成時に使われたアドレスを比較します。

特に、ライブラリのラインタイムコードはpushから始まります。それはコンパイル時に20バイトの0で構成されています。デプロイコードがまわっている時、この定数は現在のアドレスでメモリ内で置き換えられ、修正されたコードがコントラクトに保存されます。ランタイム時に、デプロイ時のアドレスが最初の定数となって、スタック上にプッシュされ、ディスパッチャーコードが現在のアドレスとその定数をviewでもpureでもないファンクションのために比較します。
