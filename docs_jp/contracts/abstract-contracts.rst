.. index:: ! contract;abstract, ! abstract contract

.. _abstract-contract:

******************
Abstract Contracts
******************

アブストラクトとなるコントラクトは少なくとも1つのファンクションは下記の様に実行部がないものを持っています（ファンクションのヘッダーが ``;`` で終わっていることを確認して下さい）::

    pragma solidity >=0.4.0 <0.6.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

その様なコントラクトはコンパイルできません（実行できるファンクションが一緒にあったとしても）が、ベースコントラクトとして使用することができます::

    pragma solidity >=0.4.0 <0.6.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }

もしあるコントラクトがアブストラクトを継承し、全ての非実行のファンクションをオーバーライドした上で、実行しなかった場合、そのコントラクト自体がアブストラクトになります。

シンタックスはとても似ていますが、実行しないファンクションは :ref:`Function Type <function_types>` とは違うということに注意してください。

実行しないファンクションの例（ファンクションの宣言）です::

    function foo(address) external returns (address);

ファンクション型の例です（変数の型が ``function`` である変数の宣言）::

    function(address) external returns (address) foo;

アブストラクトコントラクトは、より良い拡張性を持つ実装や、自己文書化、`Template method <https://en.wikipedia.org/wiki/Template_method_pattern>`_ の様なファシリテーションパターン、コード複製の削除を、コントラクトの定義から切り離します。
アブストラクトコントラクトはインターフェースに置いてメソッド定義が便利なのと同じ様に便利です。これがアブストラクトコントラクトの作成者が「自分の子は全てこのメソッドを実行する」と言うための方法です。
