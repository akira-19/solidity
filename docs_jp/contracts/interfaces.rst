.. index:: ! contract;interface, ! interface contract

.. _interfaces:

**********
Interfaces
**********

インターフェースはアブストラクトコントラクトに似ていますが、どんなファンクションも実行することはできません。他の制限がこちらです。

- 他のコントラクやインターフェースを継承できません。
- 全ての宣言されたファンクションはexternalでなければいけません。
- コンストラクタは使えません。
- 状態変数は宣言できません。

将来的にはこの制限のいくつかは撤廃されるかもしれません。

インターフェースは基本的にコントラクトABIが表せるものに制限され、ABIとインターフェース間の変換は、どんな情報の喪失もなく行われるはずです。

インターフェースはinterfaceというキーワードで表されます。

::

    pragma solidity ^0.5.0;

    interface Token {
        enum TokenType { Fungible, NonFungible }
        struct Coin { string obverse; string reverse; }
        function transfer(address recipient, uint amount) external;
    }

コントラクトは他のコントラクトを継承するようにインターフェースを継承できます。

インターフェース内で定義された型と他のコントラクトの様な構造は他のコントラクトからアクセスできます: ``Token.TokenType`` もしくは ``Token.Coin``
