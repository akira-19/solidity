.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

.. _reference-types:

Reference Types
===============

参照型の値は複数の異なった名前で修正できます。値型の変数が使用される度に、独立したコピーをとった値型と比較してみて下さい。このことから参照型は値型より気をつけて扱う必要があります。現在、構造体、配列、マッピングは参照型です。もし参照型を使用しているのであれば、値が保存されるデータ領域を常に明示する必要があります: ``memory`` (ライフタイムはファンクションの呼び出し時のみに制限されます)、``storage`` （状態変数が保存されている場所）、もしくは ``calldata`` （特別なデータロケーションで、ファンクションの引数を含み、externalのファンクションコールのパラメータでのみ使用可能です）。

データロケーションを変える値の割り当てや型変換では常に自動でコピー操作が行われる一方で、
同じデータロケーション内での値の割り当てはただコピーするだけです（いくつかの例ではstorage型で）。

.. _data-location:

Data location
-------------

*配列*、*構造体* の様な全ての参照型は"data location"というそれが保存されている場所を表す追加のアノテーションを持っています。``memory``, ``storage`` and ``calldata`` という3つのデータロケーションがあります。Calldataは外部コントラクトのファンクションの参照型のパラメータとして要求される場合にのみ有効です。Calldataは修正不可で、ファンクションの引数が保存される非永続的なエリアで、基本的にはmemoryの様に振舞います。


.. note::
    バージョン0.5.0以前では、データロケーションは省略可能ですが、変数の種類やファンクションの種類などによってデフォルトで異なる場所に保存されます。しかし、現在は全ての複雑な型は明示的にデータロケーションを示す必要があります。

.. _data-location-assignment:

Data location and assignment behaviour
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

データロケーションはデータの持続性だけではなく、割り当てのセマンティクスにも関係しています:

* ``storage`` と ``memory`` (もしくは ``calldata`` から)の間での割り当ては常に独立したコピーを作成します。
* ``memory`` から ``memory`` への割り当てでは参照のみ作られます。つまり、memory変数への変化は同じデータを参照している他のmemory変数からも可視であるということです。
* ``storage`` からローカル変数へは参照のみ割り当てられます。
* 他の ``storage`` への割り当ては常に全てコピーとなります。この例としては、状態変数への割り当てや、たとえローカル変数がただの参照だったとしてもstorageの構造型の要素への割り当てはコピーになります。


::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint[] x; // the data location of x is storage

        // the data location of memoryArray is memory
        function f(uint[] memory memoryArray) public {
            x = memoryArray; // works, copies the whole array to storage
            uint[] storage y = x; // works, assigns a pointer, data location of y is storage
            y[7]; // fine, returns the 8th element
            y.length = 2; // fine, modifies x through y
            delete x; // fine, clears the array, also modifies y
            // The following does not work; it would need to create a new temporary /
            // unnamed array in storage, but storage is "statically" allocated:
            // y = memoryArray;
            // This does not work either, since it would "reset" the pointer, but there
            // is no sensible location it could point to.
            // delete y;
            g(x); // calls g, handing over a reference to x
            h(x); // calls h and creates an independent, temporary copy in memory
        }

        function g(uint[] storage) internal pure {}
        function h(uint[] memory) public pure {}
    }

.. index:: ! array

.. _arrays:

Arrays
------

配列はコンパイル時での固定サイズか動的サイズです。

固定サイズ ``k`` で要素の型が ``T`` の配列は ``T[k]` の様に記述されます。そして動的サイズの配列は ``T[]`` の様に書けます。

例えば、5個の ``uint`` 動的配列の配列は ``uint[][5]`` の様に書けます。この記法は他の記法でも使われています。Solidityでは、たとえ ``X`` 自体が配列だとしても ``X[3]`` は常に3つの要素を含んだ ``X`` 型となります。これは例えばC言語の様な他の言語とは異なっています。

インデックスはゼロベースで、アクセスする際には宣言とは逆方向でアクセスします。

例えば、もし ``uint[][5] x memory`` を持っていたら、3つ目の動的配列に入っている2つ目の ``uint`` には ``x[2][1]`` でアクセスできます。また、3つ目の動的配列自体には ``x[2]`` でアクセスします。繰り返しですが、``T[5] a`` という配列（``T`` 自体は配列でも良い）を持っていたら、``a[2]`` というのは常に ``T`` という型になります。

配列の要素はマッピングや構造体含めてどの型でも構いません。全般的な制限として、マッピングは ``storage`` にのみ保存され、publicなファンクションは :ref:`ABI types <ABI>` であるパラメータが必要になります。

配列に ``public`` をつけることでSolidityが :ref:`getter <visibility-and-getters>`を生成します。
インデックスがgetterのパラメータになります。

そのサイズ以上の配列要素にアクセスしようとするとフェイルアサーションが発生します。新しい要素を配列の最後に追加するために ``.push()`` が、新しいサイズを指定するのに ``.length`` :ref:`member <array-members>` が使用できます（下記補足を参照ください）。


``bytes`` and ``strings`` as Arrays
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``bytes`` と ``string`` 型の変数は特別な配列となります。``bytes`` は ``byte[]`` と似ていますが、calldataとmemoryに保存されています。``string`` は ``bytes`` と等価ですが、lengthとインデックスによるアクセスができません。

SolidityはStringを操作するファンクションがありませんが、同じ機能を使うための暗黙の変換が使えます。例えば、2つのstringを比較するためには ``keccak256(abi.encode(s1)) == keccak256(abi.encode(s2))``、エンコードされた2つのstringを連結させるには ``abi.encodePacked(s1, s2);`` を使うことができます。

``byte[]`` は要素間を埋めるのに31バイト追加するので、``byte[]`` よりその分安い ``bytes`` を使用する方が良いでしょう。全般的なルールとして、``bytes`` は任意の長さの生のバイトデータを、``string`` を任意の長さのstring (UTF-8)データを使用するために使ってください。もしバイト長に制限を咥えられるのであれば、非常に低コストに抑えられるため、常に``bytes1`` から ``bytes32`` までのいずれかを使用してください。


.. note::
    もしバイト表現のある文字列 ``s`` にアクセスしたい場合は、``bytes(s).length`` / ``bytes(s)[7] = 'x';`` を使ってください。この際、低レベルのUTF-8表現にアクセスしているのであって、個々の文字にアクセスしている訳ではないということを覚えておいてください。

.. index:: ! array;allocating, new

Allocating Memory Arrays
^^^^^^^^^^^^^^^^^^^^^^^^

メモリー内のランタイム依存の長さを持つ配列を作成するには ``new`` というキーワードを使う必要があります。storageの配列とは逆で、（``.length`` を使ったりして）memoryの配列の長さを変えることはできません。事前に長さを計算しておくか、新しいmemoryの配列を作成して全ての要素をコピーする必要があります。

::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);
            assert(a.length == 7);
            assert(b.length == len);
            a[6] = 8;
        }
    }

.. index:: ! array;literals, ! inline;arrays

Array Literals
^^^^^^^^^^^^^^

配列リテラルは角括弧 (``[...]``)で囲まれ、カンマで区切られた1つ以上のリストを持っています（例えば ``[1, a, f(3)]``）。全ての要素が暗黙的に変換できる共通の型が存在しなければなりません。これはその配列の基本型になります。

配列リテラルは常に静的サイズのmemoryの配列となります。

下記の例で、``[1, 2, 3]`` の型は ``uint8[3] memory`` です。各値の型が ``uint8`` ですので、もし ``uint[3] memory`` の結果が欲しい場合には、最初の要素を ``uint`` 型に変換する必要があります。


::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function f() public pure {
            g([uint(1), 2, 3]);
        }
        function g(uint[3] memory) public pure {
            // ...
        }
    }

固定サイズのmemoryの配列は可変サイズのmemoryの配列に割り当てることはできません。例えば、次の例の様なことはできません:

::

    pragma solidity >=0.4.0 <0.6.0;

    // This will not compile.
    contract C {
        function f() public {
            // The next line creates a type error because uint[3] memory
            // cannot be converted to uint[] memory.
            uint[] memory x = [uint(1), 3, 4];
        }
    }

この制約は将来的には削除する予定ですが、ABI内で配列の受け渡し方で複雑になってしまいます。

.. index:: ! array;length, length, push, pop, !array;push, !array;pop

.. _array-members:

Array Members
^^^^^^^^^^^^^

**length**:
    配列はその要素の長さを含む ``length`` というメンバを持っています。
    memory配列の長さは作成時に固定されます（動的配列はランタイムのパラメータによります）。
    動的配列（storageでのみ使用可）に関して、このメンバは配列のサイズを変えるのに使用できます。
    そのサイズ以上の配列要素にアクセスしようとすると、自動でサイズを変更するのではなく、フェイルアサーションが発生します。
    長さを大きくすると、ゼロ初期化された要素が配列に加わります。長さを減らすと、削除された各要素に対して ``delete`` を暗黙的に行います。もしstorage出ない非動的配列のリサイズを行おうとすると、``Value must be an lvalue`` というエラーが発生します。
**push**:
    動的storage配列と ``bytes``（``string`` ではなく）は ``push`` というファンクションを持ち、配列の最後に要素を追加することができます。その要素はゼロ初期化されます。ファンクションは新しい長さを返します。
**pop**:
    動的storage配列と ``bytes``（``string`` ではなく）は ``pop`` というファンクションを持ち、配列の最後から一つの要素を削除することができます。これも暗黙的に削除する要素に対して ``delete`` を呼び出しています。

.. warning::
    もし ``.length--`` を空の配列に対して使うのと、アンダーフローし、長さが ``2**256-1`` となってしまいます。

.. note::
    storageの配列の長さを増やすことで一定のガスがかかります。これはstorageがゼロ初期化されているとみなされているためです。一方で、長さを減らすのは少なくとも比例関数的にコストがかかります（しかしほとんどの場合比例関数より高くなります）。これは、``delete`` を呼び出す様に明示的に削除した要素をクリアする工程を含んでいるためです。

.. note::
    まだ配列の配列をexternalのファンクションで使用することはできません（ただし、publicのファンクションではサポートされています）。

.. note::
    Byzantiumの前のEVMのバージョンではファンクションコールからの返り値として動的配列にはアクセスできませんでした。もし動的配列を返すファンクションんを呼び出すときは、ByzantiumモードがセットされているEVMを使っていることを確認して下さい。

::

    pragma solidity >=0.4.16 <0.6.0;

    contract ArrayContract {
        uint[2**20] m_aLotOfIntegers;
        // Note that the following is not a pair of dynamic arrays but a
        // dynamic array of pairs (i.e. of fixed size arrays of length two).
        // Because of that, T[] is always a dynamic array of T, even if T
        // itself is an array.
        // Data location for all state variables is storage.
        bool[2][] m_pairsOfFlags;

        // newPairs is stored in memory - the only possibility
        // for public contract function arguments
        function setAllFlagPairs(bool[2][] memory newPairs) public {
            // assignment to a storage array performs a copy of ``newPairs`` and
            // replaces the complete array ``m_pairsOfFlags``.
            m_pairsOfFlags = newPairs;
        }

        struct StructType {
            uint[] contents;
            uint moreInfo;
        }
        StructType s;

        function f(uint[] memory c) public {
            // stores a reference to ``s`` in ``g``
            StructType storage g = s;
            // also changes ``s.moreInfo``.
            g.moreInfo = 2;
            // assigns a copy because ``g.contents``
            // is not a local variable, but a member of
            // a local variable.
            g.contents = c;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // access to a non-existing index will throw an exception
            m_pairsOfFlags[index][0] = flagA;
            m_pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // if the new size is smaller, removed array elements will be cleared
            m_pairsOfFlags.length = newSize;
        }

        function clear() public {
            // these clear the arrays completely
            delete m_pairsOfFlags;
            delete m_aLotOfIntegers;
            // identical effect here
            m_pairsOfFlags.length = 0;
        }

        bytes m_byteData;

        function byteArrays(bytes memory data) public {
            // byte arrays ("bytes") are different as they are stored without padding,
            // but can be treated identical to "uint8[]"
            m_byteData = data;
            m_byteData.length += 7;
            m_byteData[3] = 0x08;
            delete m_byteData[2];
        }

        function addFlag(bool[2] memory flag) public returns (uint) {
            return m_pairsOfFlags.push(flag);
        }

        function createMemoryArray(uint size) public pure returns (bytes memory) {
            // Dynamic memory arrays are created using `new`:
            uint[2][] memory arrayOfPairs = new uint[2][](size);

            // Inline arrays are always statically-sized and if you only
            // use literals, you have to provide at least one type.
            arrayOfPairs[0] = [uint(1), 2];

            // Create a dynamic byte array:
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = byte(uint8(i));
            return b;
        }
    }


.. index:: ! struct, ! type;struct

.. _structs:

Structs
-------

Solidityでは次の例の様に構造体として新しい型を定義する方法があります:

::

    pragma solidity >=0.4.11 <0.6.0;

    contract CrowdFunding {
        // Defines a new type with two fields.
        struct Funder {
            address addr;
            uint amount;
        }

        struct Campaign {
            address payable beneficiary;
            uint fundingGoal;
            uint numFunders;
            uint amount;
            mapping (uint => Funder) funders;
        }

        uint numCampaigns;
        mapping (uint => Campaign) campaigns;

        function newCampaign(address payable beneficiary, uint goal) public returns (uint campaignID) {
            campaignID = numCampaigns++; // campaignID is return variable
            // Creates new struct in memory and copies it to storage.
            // We leave out the mapping type, because it is not valid in memory.
            // If structs are copied (even from storage to storage), mapping types
            // are always omitted, because they cannot be enumerated.
            campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // Creates a new temporary memory struct, initialised with the given values
            // and copies it over to storage.
            // Note that you can also use Funder(msg.sender, msg.value) to initialise.
            c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
            c.amount += msg.value;
        }

        function checkGoalReached(uint campaignID) public returns (bool reached) {
            Campaign storage c = campaigns[campaignID];
            if (c.amount < c.fundingGoal)
                return false;
            uint amount = c.amount;
            c.amount = 0;
            c.beneficiary.transfer(amount);
            return true;
        }
    }

このコントラクトはクラウドファンディングの全機能を備えている訳ではありませんが、構造体を理解するために必要な基本的な概念を含んでいます。構造体型はマッピングや配列の中でも使えますし、構造体の中にマッピングや配列を含むこともできます。

構造体自身はマッピングの要素の値型になったり、自分自身の型の動的配列を含むことはできますが、構造体の中に自分自身の構造体型を含めることはできません。構造体のサイズが有限である様にするためにこの制限が必要となっています。

これらの全てのファンクションの中で、構造体型がどの様にデータの保存場所である ``storage`` が付いているローカル変数に割り当てられているか注意してください。構造体をコピーしているのではなく、参照先を保存しているだけなので、ローカル変数への割り当ては実際は状態のみを記述しています。

もちろん、``campaigns[campaignID].amount = 0`` の様にローカル変数への割り当てをせずに直接構造体へアクセスすることもできます。
