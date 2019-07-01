.. index:: ! contract

.. _contracts:

##########
Contracts
##########

Solidityに置けるコントラクトはオブジェクト指向の言語におけるクラスに似ています。コントラクトは永続的な状態変数とそれを変更するファンクションを保持しています。異なるコントラクト上（インスタンス上）のファンクションコールはEVMのファンクションコールとなり、その結果コンテクストが変わり、状態変数にアクセスできなくなります。コントラクトとそのファンクションは呼び出さないと動きません。Ethereumには"cron"の様に特定のイベントで自動的にファンクションを呼び出す機能はありません。

.. include:: contracts/creating-contracts.rst

.. include:: contracts/visibility-and-getters.rst

.. include:: contracts/function-modifiers.rst

.. include:: contracts/constant-state-variables.rst
.. include:: contracts/functions.rst

.. include:: contracts/events.rst

.. include:: contracts/inheritance.rst

.. include:: contracts/abstract-contracts.rst
.. include:: contracts/interfaces.rst

.. include:: contracts/libraries.rst

.. include:: contracts/using-for.rst
