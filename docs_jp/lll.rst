###
LLL
###

.. _lll:

LLLはS式構文に対応したEVMのための低水準言語です。

SolidityリポジトリはLLLコンパイラを含んでおり、Solidityとアセンブラサブシステムを共有します。 
しかし、コンパイルされていることを除けば、これ以上改善させることはありません。

以下のように特別に指定しない限り、ビルドに含まれることはありません。

.. code-block:: bash

    $ cmake -DLLL=ON ..
    $ cmake --build .

.. warning::

    LLLのコードは非推奨扱いであり、将来Solidityリポジトリから削除されるでしょう。
