.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

***********************
Structure of a Contract
***********************

Solidityのコントラクトはオブジェクト志向の言語のクラスに似ています。各コントラクトは :ref:`structure-state-variables`、:ref:`structure-functions`、:ref:`structure-function-modifiers`、:ref:`structure-events`、:ref:`structure-struct-types`、:ref:`structure-enum-types` の宣言を含んでいます。更に、コントラクトは他のコントラクトを継承できます。

:ref:`ライブラリ<libraries>` と :ref:`インターフェース<interfaces>` と呼ばれる特別な種類のコントラクトもあります。

:ref:`コントラクト<contracts>` に関するセクションはより詳細な説明があります。このセクションは簡単な全体像です。

.. _structure-state-variables:

State Variables
===============

状態変数とは値が永久的にコントラクトのストレージに保存される値です。

::

    pragma solidity >=0.4.0 <0.6.0;

    contract SimpleStorage {
        uint storedData; // State variable
        // ...
    }

有効な状態変数のタイプは :ref:`types` を、可視性の実行可能な選択肢については :ref:`visibility-and-getters` を参照ください。

.. _structure-functions:

Functions
=========

ファンクションはコントラクト内にある実行可能なコードの一式です。

::

    pragma solidity >=0.4.0 <0.6.0;

    contract SimpleAuction {
        function bid() public payable { // Function
            // ...
        }
    }

:ref:`function-calls` は内部でも外部でも行うことができ、他のコントラクトに対して異なったレベルの :ref:`visibility<visibility-and-getters>` を持ちます。パラメータや値を受け渡しするために、:ref:`Functions<functions>` は :ref:`parameters and return variables<function-parameters-return-variables>` を受け入れます。

.. _structure-function-modifiers:

Function Modifiers
==================

ファンクションModifierは宣言的な方法でファンクションのセマンティクスを修正することができます(コントラクトセクションの :ref:`modifiers` を参照ください)。

::

    pragma solidity >=0.4.22 <0.6.0;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // Modifier
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        function abort() public view onlySeller { // Modifier usage
            // ...
        }
    }

.. _structure-events:

Events
======

イベントはEVMのロギング機能で使われる便利なインターフェースです。

::

    pragma solidity >=0.4.21 <0.6.0;

    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // Event

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
        }
    }

DApp内でイベントがどの様に宣言され、使用されるかはコントラクトセクションの :ref:`events` を参照ください。

.. _structure-struct-types:

Struct Types
=============

構造体は複数の変数をグループ化できるカスタム定義されたタイプです (タイプセクションの :ref:`structs` を参照下さい)。

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Ballot {
        struct Voter { // Struct
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }

.. _structure-enum-types:

Enum Types
==========

Enumsは有限個の'constant values'を持つカスタムタイプを作成するのに使用されます(タイプセクションの :ref:`enums` を参照ください)。

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // Enum
    }
