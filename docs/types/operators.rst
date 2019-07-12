.. index:: assignment, ! delete, lvalue

Operators Involving LValues
===========================


もし ``a`` がLValue（変数もしくは何か割り当てが行えるもの）であれば、以下の演算子が短縮記法として使えます:

``a += e`` は ``a = a + e`` と等価です。 演算子 ``-=``, ``*=``, ``/=``, ``%=``, ``|=``, ``&=``, ``^=`` も同様に定義されます。``a++`` と ``a--`` は ``a += 1`` / ``a -= 1`` と等価ですがこの式自体は計算前の値 ``a`` です。一方、``--a`` と ``++a`` は ``a`` に対して同じ計算を行いますが、この式自体は計算後の値を持ちます。

delete
------

``delete a`` は ``a`` に初期値を割り当てます。例えば、整数型であれば ``a = 0`` に相当します。しかし、長さゼロの動的配列や全ての要素が初期値で同じ長さの静的配列も使用可能です。``delete a[x]`` はインデックス ``x`` の配列の要素を削除し、他のは残し配列の長さは変えません。つまり配列に空白を残します。もし要素を削除する予定であれば、おそらくマッピングの方がよいでしょう。

構造体に対しては全ての要素をリセットした上で初期値を割り当てます。言い換えると、``delete a``　の後は ``a`` があたかも値の代入されていないただ宣言されただけの ``a`` と同じ扱いができます。ただし、以下の様に注意が必要です:

``delete`` はマッピングには効きません（マッピングのキーはおそらく任意であり、一般に未知のためです）。そのため、もし構造体を削除したら、マッピングではない全ての要素はリセットされ、マッピング以外の要素は再帰的に処理されます。しかし、個々のキーと、マッピングしたものは削除することができます: ``a`` がマッピングであれば、``delete a[x]`` で ``x`` に保存されている値は削除されます。

覚えておきたいのは ``delete a`` は ``a`` への値の代入の様に振舞うことです。``a`` に新しいオブジェクトを保存します。この特徴は ``a`` が参照型の場合分かりやすいです: ``a`` そのものだけリセットし、元々参照していた値はリセットしません。


::

    pragma solidity >=0.4.0 <0.6.0;

    contract DeleteExample {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // sets x to 0, does not affect data
            delete data; // sets data to 0, does not affect x
            uint[] storage y = dataArray;
            delete dataArray; // this sets dataArray.length to zero, but as uint[] is a complex object, also
            // y is affected which is an alias to the storage object
            // On the other hand: "delete y" is not valid, as assignments to local variables
            // referencing storage objects can only be made from existing storage objects.
            assert(y.length == 0);
        }
    }
