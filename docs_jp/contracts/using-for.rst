.. index:: ! using for, library

.. _using-for:

*********
Using For
*********

``using A for B;`` という指示はライブラリのファンクション（ ``A`` というライブラリから）をどんな型(``B``)にも加えるのに使うことができます。
このファンクションは最初のパラメータとして（Pythonの ``self`` の様に）、そのファンクションを呼び出したオブジェクトを受け取ります。

``using A for *;`` というのは、ライブラリ ``A`` のファンクションが *どんな* 型にも追加されるということです。

いずれの場合でも、最初のパラメータの型がオブジェクトの型と合わなくても、ライブラリ中の *全ての* ファンクションが追加されます。ファンクションが呼ばれた時に型はチェックされ、ファンクションのoverload resolutionが実行されます。

``using A for B;`` は現在の全てのファンクション内も含むコントラクト内でのみ有効です。そのコントラクトが使用されている外部のコントラクトでは使えません。この指示はファンクション内でなく、コントラクト内でのみ実行可能です。

ライブラリを含めることにより、ライブラリのファンクションを含むデータタイプが追加のコードなしで使える様になります。

:ref:`libraries` の例を書き換えてみます::

    pragma solidity >=0.4.16 <0.6.0;

    // This is the same code as before, just without comments
    library Set {
      struct Data { mapping(uint => bool) flags; }

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
        using Set for Set.Data; // this is the crucial change
        Set.Data knownValues;

        function register(uint value) public {
            // Here, all variables of type Set.Data have
            // corresponding member functions.
            // The following function call is identical to
            // `Set.insert(knownValues, value)`
            require(knownValues.insert(value));
        }
    }

この様に基本型を拡張することも可能です::

    pragma solidity >=0.4.16 <0.6.0;

    library Search {
        function indexOf(uint[] storage self, uint value)
            public
            view
            returns (uint)
        {
            for (uint i = 0; i < self.length; i++)
                if (self[i] == value) return i;
            return uint(-1);
        }
    }

    contract C {
        using Search for uint[];
        uint[] data;

        function append(uint value) public {
            data.push(value);
        }

        function replace(uint _old, uint _new) public {
            // This performs the library function call
            uint index = data.indexOf(_old);
            if (index == uint(-1))
                data.push(_new);
            else
                data[index] = _new;
        }
    }

全てのライブラリのコールは実際のEVMのファンクションコールであるということを覚えておいてください。つまり、もしメモリもしくは値型を渡す時には、``self`` 変数であってもコピーが実行されます。ストレージ参照の変数が使われた時だけ、コピーは実行されません。
