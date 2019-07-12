###############
Common Patterns
###############

.. index:: withdrawal

.. _withdrawal_pattern:

*************************
Withdrawal from Contracts
*************************

発行の後にファンドを送る方法としてお勧めなのはwithdrawalパターンを使うことです。直接 ``transfer`` コールするのは直感的なEtherの送金方法ですが、潜在的にセキュリティリスクを実装することになるので、推奨しません。これに関しての詳しい情報は、:ref:`security_considerations` を読んでください。

次の例はあるコントラクトにおける実際のwithdrawalパターンです。このコントラクトの目的は"richest"（ `King of the Ether <https://www.kingoftheether.com/>`_ に触発されました）になるためにたくさんのお金をコントラクトに送ることです。

下記のコントラクトでは、richestの座を奪われたら、新しくrichestになった人のファンドを受け取れます。

::

    pragma solidity ^0.5.0;

    contract WithdrawalContract {
        address public richest;
        uint public mostSent;

        mapping (address => uint) pendingWithdrawals;

        constructor() public payable {
            richest = msg.sender;
            mostSent = msg.value;
        }

        function becomeRichest() public payable returns (bool) {
            if (msg.value > mostSent) {
                pendingWithdrawals[richest] += msg.value;
                richest = msg.sender;
                mostSent = msg.value;
                return true;
            } else {
                return false;
            }
        }

        function withdraw() public {
            uint amount = pendingWithdrawals[msg.sender];
            // Remember to zero the pending refund before
            // sending to prevent re-entrancy attacks
            pendingWithdrawals[msg.sender] = 0;
            msg.sender.transfer(amount);
        }
    }

次は逆にもっと直感的な送金パターンです。

::

    pragma solidity ^0.5.0;

    contract SendContract {
        address payable public richest;
        uint public mostSent;

        constructor() public payable {
            richest = msg.sender;
            mostSent = msg.value;
        }

        function becomeRichest() public payable returns (bool) {
            if (msg.value > mostSent) {
                // This line can cause problems (explained below).
                richest.transfer(msg.value);
                richest = msg.sender;
                mostSent = msg.value;
                return true;
            } else {
                return false;
            }
        }
    }

気づいて頂きたいのは、この例では攻撃者が、``richest`` に、失敗する（例えば、``revert()`` を使ったり、2300ガス以上を消費したりすることで）フォールバックファンクションを持ったコントラクトアドレスを入れることで、通常ではない状態をコントラクトに仕込めるということです。こういう方法によって、"毒された"コントラクトにファンドを送るために ``transfer`` が呼ばれると常に失敗し、その結果、``becomeRichest`` が失敗し、コントラクトは永遠に動かなくなります。

一方、もし最初の例の"withdraw"パターンを使うと、攻撃者は自分のwithdrawの失敗しか起こせないので、他のコントラクトの機能は損なわれません。

.. index:: access;restricting

******************
Restricting Access
******************

アクセス制限はコントラクトの共通パターンです。他の人やコンピューターがトランザクションやコントラクトのステートを読み取ることを制限することはできないということを覚えておいてください。暗号化で少し難しくはできますが、もしコントラクトがデータを読み込めるなら、他の人もできます。

**他のコントラクト** によるあなたのコントラクトのステートへの読み込みのアクセスは制限できます。状態変数を ``public`` で宣言しなければ、これはデフォルトでそうなっています。

さらに、誰があなたのコントラクトのステートを変更したり、ファンクションを呼び出したりできるか制限することができます。そしてこれがこのセクションで説明することです。

.. index:: function;modifier

**function modifiers** を使うことでこれらの制限をとても読みやすくします。

::

    pragma solidity >=0.4.22 <0.6.0;

    contract AccessRestriction {
        // These will be assigned at the construction
        // phase, where `msg.sender` is the account
        // creating this contract.
        address public owner = msg.sender;
        uint public creationTime = now;

        // Modifiers can be used to change
        // the body of a function.
        // If this modifier is used, it will
        // prepend a check that only passes
        // if the function is called from
        // a certain address.
        modifier onlyBy(address _account)
        {
            require(
                msg.sender == _account,
                "Sender not authorized."
            );
            // Do not forget the "_;"! It will
            // be replaced by the actual function
            // body when the modifier is used.
            _;
        }

        /// Make `_newOwner` the new owner of this
        /// contract.
        function changeOwner(address _newOwner)
            public
            onlyBy(owner)
        {
            owner = _newOwner;
        }

        modifier onlyAfter(uint _time) {
            require(
                now >= _time,
                "Function called too early."
            );
            _;
        }

        /// Erase ownership information.
        /// May only be called 6 weeks after
        /// the contract has been created.
        function disown()
            public
            onlyBy(owner)
            onlyAfter(creationTime + 6 weeks)
        {
            delete owner;
        }

        // This modifier requires a certain
        // fee being associated with a function call.
        // If the caller sent too much, he or she is
        // refunded, but only after the function body.
        // This was dangerous before Solidity version 0.4.0,
        // where it was possible to skip the part after `_;`.
        modifier costs(uint _amount) {
            require(
                msg.value >= _amount,
                "Not enough Ether provided."
            );
            _;
            if (msg.value > _amount)
                msg.sender.transfer(msg.value - _amount);
        }

        function forceOwnerChange(address _newOwner)
            public
            payable
            costs(200 ether)
        {
            owner = _newOwner;
            // just some example condition
            if (uint(owner) & 0 == 1)
                // This did not refund for Solidity
                // before version 0.4.0.
                return;
            // refund overpaid fees
        }
    }

どのファンクションコールへのアクセスが制限できるかというもっと特殊な方法は次の例で説明します。

.. index:: state machine

*************
State Machine
*************

コントラクトはよくステートマシンとして振舞います。つまり、異なる挙動をする、もしくは異なるファンクションがコールされるいくつかの **ステージ** を持っています。ファンクションコールはしばしばステージを終わらせ、コントラクトを次のステージへ以降させます（特にコントラクトに **相互作用** があった場合）。いくつかのステージは自動的にあるポイントに **そのうちに** なっていることもよくあります。

この例としてブラインドオークションのコントラクトがあります。このコントラクトでは"ブラインドビッドの承認"ステージから始まり、"オークションの結果"で終わる"ビッドの公開"に移行します。

.. index:: function;modifier

このシチュエーションで、ステートを作って、コントラクトの間違った使用法から守るため、ファンクションmodifierを使うことができます。

Example
=======

次の例では、modifier ``atStage`` はあるステージの時にのみファンクションが呼ばれるようにすることができます。

自動でのある時間での変遷はmodifier ``timeTransitions`` でコントロールすることができ、全てのファンクションで使えます。

.. note::
    **modifierの順番は重要です。**
    atStageがtimedTransitionsと一緒にする場合、timedTransitionsの後にatStageを付けてください。そうすることで新しいステージが考慮されます。


最後に、modifier ``transitionNext`` はファンクションが終わった時に自動的に次のステージに行くために使用できます。

.. note::
    **modifierはスキップされるかもしれません。**
    これはSolidityバージョン0.4.0より前にのみ適用されます。modifierはファンクションコールを使うのではなく、単純にコードを別の所に書くことで適用されているので、もしファンクションがreturnを使っていたら、transitionNext内のコードはスキップされるかもしれません。もしスキップさせたくないのであれば、ファンクション内マニュアルでnextStageを呼び出してください。
    バージョン0.4.0からはファンクションが明示的にreturnしていても、modifierコードは実行されます。

::

    pragma solidity >=0.4.22 <0.6.0;

    contract StateMachine {
        enum Stages {
            AcceptingBlindedBids,
            RevealBids,
            AnotherStage,
            AreWeDoneYet,
            Finished
        }

        // This is the current stage.
        Stages public stage = Stages.AcceptingBlindedBids;

        uint public creationTime = now;

        modifier atStage(Stages _stage) {
            require(
                stage == _stage,
                "Function cannot be called at this time."
            );
            _;
        }

        function nextStage() internal {
            stage = Stages(uint(stage) + 1);
        }

        // Perform timed transitions. Be sure to mention
        // this modifier first, otherwise the guards
        // will not take the new stage into account.
        modifier timedTransitions() {
            if (stage == Stages.AcceptingBlindedBids &&
                        now >= creationTime + 10 days)
                nextStage();
            if (stage == Stages.RevealBids &&
                    now >= creationTime + 12 days)
                nextStage();
            // The other stages transition by transaction
            _;
        }

        // Order of the modifiers matters here!
        function bid()
            public
            payable
            timedTransitions
            atStage(Stages.AcceptingBlindedBids)
        {
            // We will not implement that here
        }

        function reveal()
            public
            timedTransitions
            atStage(Stages.RevealBids)
        {
        }

        // This modifier goes to the next stage
        // after the function is done.
        modifier transitionNext()
        {
            _;
            nextStage();
        }

        function g()
            public
            timedTransitions
            atStage(Stages.AnotherStage)
            transitionNext
        {
        }

        function h()
            public
            timedTransitions
            atStage(Stages.AreWeDoneYet)
            transitionNext
        {
        }

        function i()
            public
            timedTransitions
            atStage(Stages.Finished)
        {
        }
    }
