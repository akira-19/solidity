.. index:: !mapping
.. _mapping-types:

Mapping Types
=============

マッピング型は ``mapping(_KeyType => _ValueType)`` という構文で宣言します。``_KeyType`` はどの基本型でも入ります。つまりどのビルトインの型に加え、``bytes`` と ``string`` が使えます。ユーザー定義、もしくはコントラクト型、enum、マッピング、構造体、``bytes`` と ``string`` を除いた配列は使用できません。``_ValueType`` はマッピングを含めて、どの型でもとることができます。

マッピングはどんなキーも存在し、そのバイト表現は全てゼロ（型の :ref:`デフォルト値 <default-value>`）で実質的に初期化されている `hash tables <https://en.wikipedia.org/wiki/Hash_table>`_ の様に考えることができます。ただ似ているのはそれだけで、キーデータはマッピングには保存されず、その ``keccak256`` ハッシュだけが値を検索するのに使用されます。

そのため、マッピングは長さを持たず、キーの概念やセットする値などはありません。

マッピングは ``storage`` にのみデータを置けるため、ファンクション内のstorage参照型、もしくはライブラリファンクションのパラメータとしての状態変数として使用されます。パラメータやパブリックにアクセスできるコントラクトファンクションの返り値としては使用できません。

マッピングタイプに ``public`` 修飾子をつけることができます。Solidityはそれにより :ref:`getter <visibility-and-getters>` を生成します。``_KeyType`` はゲッターのパラメータになります。もし ``_ValueType`` が値型か構造体の場合、ゲッターは ``_ValueType`` を返します。もし ``_ValueType`` が配列かマッピングであれば、ゲッターは各 ``_KeyType`` に対して一つのパラメータを再帰的に持ちます。以下マッピングの例です:

::

    pragma solidity >=0.4.0 <0.6.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingUser {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(address(this));
        }
    }


.. note::
  マッピングはiterableではありませんが、マッピング上のデータ構造を実装することは可能です。具体例は `iterable mapping <https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol>`_ を参照ください。
