.. index:: ! function;modifier

.. _modifiers:

******************
Function Modifiers
******************

modifierはファンクションの挙動を簡単に変えるのに使うことができます。例えば、ファンクションの実行前に自動的にある条件をチェックできます。modifierは継承可能なコントラクトのプロパティで、継承されたコントラクトによってオーバーライドされるかもしれません。

::

    pragma solidity ^0.5.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;

        // This contract only defines a modifier but does not use
        // it: it will be used in derived contracts.
        // The function body is inserted where the special symbol
        // `_;` in the definition of a modifier appears.
        // This means that if the owner calls this function, the
        // function is executed and otherwise, an exception is
        // thrown.
        modifier onlyOwner {
            require(
                msg.sender == owner,
                "Only owner can call this function."
            );
            _;
        }
    }

    contract mortal is owned {
        // This contract inherits the `onlyOwner` modifier from
        // `owned` and applies it to the `close` function, which
        // causes that calls to `close` only have an effect if
        // they are made by the stored owner.
        function close() public onlyOwner {
            selfdestruct(owner);
        }
    }

    contract priced {
        // Modifiers can receive arguments:
        modifier costs(uint price) {
            if (msg.value >= price) {
                _;
            }
        }
    }

    contract Register is priced, owned {
        mapping (address => bool) registeredAddresses;
        uint price;

        constructor(uint initialPrice) public { price = initialPrice; }

        // It is important to also provide the
        // `payable` keyword here, otherwise the function will
        // automatically reject all Ether sent to it.
        function register() public payable costs(price) {
            registeredAddresses[msg.sender] = true;
        }

        function changePrice(uint _price) public onlyOwner {
            price = _price;
        }
    }

    contract Mutex {
        bool locked;
        modifier noReentrancy() {
            require(
                !locked,
                "Reentrant call."
            );
            locked = true;
            _;
            locked = false;
        }

        /// This function is protected by a mutex, which means that
        /// reentrant calls from within `msg.sender.call` cannot call `f` again.
        /// The `return 7` statement assigns 7 to the return value but still
        /// executes the statement `locked = false` in the modifier.
        function f() public noReentrancy returns (uint) {
            (bool success,) = msg.sender.call("");
            require(success);
            return 7;
        }
    }

複数のmodifierはホワイトスペースで分けられたリストで明記されあるファンクションに適用されます。書かれた順番に評価されます。

.. warning::
    以前のSolidityでは ``return`` はmodifierを持つファンクションの中では異なった挙動をしていました。

modifierもしくはファンクション本体からの明示的なreturnは現在のmodifierもしくはファンクション本体からしか出ません。返ってきた変数は割り当てられ、制御フローは先に処理されたmodifierの"_"の後に続きます。

modifierの引数に任意の式が使えます。このコンテキストに置いて、ファンクションから可視の全ての記号はmodifierでも可視です。modifierで処理される記号はファンクションからは可視ではありません（オーバーライドで変わってしまう可能性があるため）。
