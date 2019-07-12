.. index:: ! type;conversion, ! cast

.. _types-conversion-elementary-types:

Conversions between Elementary Types
====================================

Implicit Conversions
--------------------


もし演算子が違う型に使われた場合、コンパイラは暗黙のうちに演算対象の型を変換します（代入、割り当て時も同様です）。一般に、セマンティック的に同じ扱いができ、何の情報も失われないのであれば、暗黙の値型の変換は可能です: ``uint8`` は ``uint16``　に、``int128`` は ``int256`` に変換可能ですが、``int8`` は ``uint256`` に変換不可です（``uint256`` は例えば ``-1`` を持てないからです）。

詳細は型のセクションを参照してください。

Explicit Conversions
--------------------

もしコンパイラが暗黙の型変換を行わず、しかしあなたが何をしようとしているのか自分で理解しているのであれば、明示的な型変換は可能な時もあります。しかし、これは予想しない挙動や、コンパイラのセキュリティ的な機能をバイパスする可能性もあるので、結果が予想通りになるかテストを行ってください。以下の例は、負の ``int8`` を ``uint`` に変換しています。

::

    int8 y = -3;
    uint x = uint(y);

このコードスニペットの最後では ``x`` は ``0xfffff..fd`` (64文字の16進数)という値を持ち、これは256ビットの2の補数表現です。

もしある整数が明示的に小さい型に変換された場合、上側のビットがカットされます::

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // b will be 0x5678 now

もしある整数が大きい型に明示的に変換され場合は、左側に（上側のビットに）パディングされます。変換後の結果と元の整数を比較した場合、それらは等しいものとして扱われます::

    uint16 a = 0x1234;
    uint32 b = uint32(a); // b will be 0x00001234 now
    assert(a == b);

固定長のバイト型の変換は異なった挙動をします。個々のバイトのシーケンスとして扱われ、小さいサイズへの型変換時にはそのシーケンスがカットされます::

    bytes2 a = 0x1234;
    bytes1 b = bytes1(a); // b will be 0x12

もし固定サイズのバイト型が明示的に大きい型に変換された場合、右側がパディングされます。（もしインデックスが範囲内にあるのであれば）変換前後で同じインデックスで同じ値を返します::

    bytes2 a = 0x1234;
    bytes4 b = bytes4(a); // b will be 0x12340000
    assert(a[0] == b[0]);
    assert(a[1] == b[1]);

切り取りやパディングの際に、整数と固定サイズのバイト配列は異なった挙動を示すため、整数と固定サイズのバイト配列の明示的な変換は二つが同じサイズである場合のみ許可されます。もし異なるサイズの整数と固定サイズのバイト配列を変換したい時は、理想的な切り取りやパディングルールを明示的にする中継的な変換を使用する必要があります::

    bytes2 a = 0x1234;
    uint32 b = uint16(a); // b will be 0x00001234
    uint32 c = uint32(bytes4(a)); // c will be 0x12340000
    uint8 d = uint8(uint16(a)); // d will be 0x34
    uint8 e = uint8(bytes1(a)); // e will be 0x12

.. _types-conversion-literals:

Conversions between Literals and Elementary Types
=================================================

Integer Types
-------------

切り捨てなしに表せるほど十分大きければ10進数と16進数のリテラルは暗黙的にどの整数型にも変換可能です::

    uint8 a = 12; // fine
    uint32 b = 1234; // fine
    uint16 c = 0x123456; // fails, since it would have to truncate to 0x3456

Fixed-Size Byte Arrays
----------------------

10進数の数字リテラルは暗黙的に固定サイズのバイト配列に変換できません。16進数の数字リテラルはバイト型のサイズと桁数がぴったり合っている場合のみ変換可能です。10進数、16進数両者の唯一の例外として、値が0であればどの固定サイズのバイト型に変換できます::

    bytes2 a = 54321; // not allowed
    bytes2 b = 0x12; // not allowed
    bytes2 c = 0x123; // not allowed
    bytes2 d = 0x1234; // fine
    bytes2 e = 0x0012; // fine
    bytes4 f = 0; // fine
    bytes4 g = 0x0; // fine

文字数とバイト型のサイズが合っていれば、文字列リテラルと16進数文字列リテラルは暗黙的に固定サイズバイト配列に変換できます::

    bytes2 a = hex"1234"; // fine
    bytes2 b = "xy"; // fine
    bytes2 c = hex"12"; // not allowed
    bytes2 d = hex"123"; // not allowed
    bytes2 e = "x"; // not allowed
    bytes2 f = "xyz"; // not allowed

Addresses
---------

:ref:`address_literals` で説明したように、チェックサムテストが通る正しいサイズの16進数リテラルは ``アドレス`` 型です。他のリテラルは暗黙的に ``アドレス`` 型への変換はできません。

``bytes20`` もしくは他の整数型から ``address`` への明示的な変換を行うと ``address payable`` になります。
