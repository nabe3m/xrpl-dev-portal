---
html: transaction-common-fields.html
parent: transaction-formats.html
seo:
    description: これらの共通フィールドは、XRP Ledgerのあらゆるトランザクションで指定できます。
labels:
  - トランザクション送信
---
# トランザクションの共通フィールド

どのトランザクションについても、共通する一連のフィールドに加え、[トランザクションのタイプ](types/index.md)に応じた追加のフィールドがあります。フィールドの名前では、大文字と小文字が区別されます。すべてのトランザクションに共通するフィールドは、以下のとおりです。

| フィールド              | JSONの型              | [内部の型][]        | 必須?     | 説明        |
|:---------------------|:---------------------|:------------------|:----------|:------------|
| `Account`            | 文字列 - [アドレス][]   | AccountID         | はい       | トランザクションを開始した[アカウント](../../../concepts/accounts/index.md)。 |
| `TransactionType`    | 文字列                | UInt16            | はい       | トランザクションのタイプ。有効な[トランザクションのタイプ](types/index.md)には、`Payment`、`OfferCreate`、`TrustSet`など多数があります。 |
| `Fee`                | 文字列 - 数値          | Amount            | はい。[自動入力可能][] | 整数で表したXRPの額（drop単位）。このトランザクションを送信するためのコストとして消却されます。トランザクションのタイプによっては、最小要件が異なります。詳細は、[トランザクションコスト][]をご覧ください。 |
| `Sequence`           | 数値                  | UInt32            | はい。[自動入力可能][] | トランザクションを送信するアカウントの[シーケンス番号](../data-types/basic-data-types.md#アカウントシーケンス)。トランザクションが有効とみなされるのは、その`Sequence`番号が、同一のアカウントの直前トランザクションよりもちょうど1大きい場合のみです。特殊なケースとして、`0`はトランザクションが代わりに[Ticket](../../../concepts/accounts/tickets.md)を使用していることを意味します{% amendment-disclaimer name="TicketBatch" /%}。 |
| [`AccountTxnID`](#accounttxnid) | 文字列 - [ハッシュ][] | UInt256   | いいえ     | 別のトランザクションを識別するためのハッシュ値。このハッシュがある場合、このトランザクションが有効になるのは、送信側のアカウントの直前送信トランザクションがこのハッシュと一致しているときのみです。 |
| [`Delegate`](#delegate) | 文字列 - [アドレス][] | AccountID      | いいえ     | `Account`に代わってトランザクションを送信する委任先アカウント。{% amendment-disclaimer name="PermissionDelegationV1_1" /%} |
| [`Flags`](#flagsフィールド) | 数値            | UInt32            | いいえ     | このトランザクションのビットフラグのセット。 |
| `LastLedgerSequence` | 数値                  | UInt32            | いいえ。[自動入力可能][] | このトランザクションを登録できるレジャーインデックスの最大値。このフィールドを指定することにより、トランザクションが検証または拒否されるのを待たなければならない期間に厳密な上限を設定することができます。詳細は、[信頼できるトランザクションの送信](../../../concepts/transactions/reliable-transaction-submission.md)をご覧ください。 |
| [`Memos`](#memosフィールド) | オブジェクトの配列 | 配列             | いいえ     | このトランザクションに付加される任意の追加情報。 |
| [`NetworkID`](#networkidフィールド) | 数値   | UInt32            | _ネットワークによる_ | このトランザクションが対象とするチェーンのネットワークID。Mainnetおよび一部のテストネットワークでは**省略しなければなりません**。ネットワークIDが1025以上のチェーンでは**必須**です。 |
| [`Signers`](#signersフィールド) | 配列        | 配列              | いいえ     | このトランザクションを承認するための[マルチシグ](../../../concepts/accounts/multi-signing.md)を表すオブジェクトの配列。 |
| `Sponsor`            | 文字列 - [アドレス][]   | AccountID         | いいえ     | スポンサーとなるアカウントのアドレス。{% amendment-disclaimer name="Sponsor" /%} |
| [`SponsorFlags`](#sponsorflagsフィールド) | 数値 | UInt32           | いいえ     | スポンサーシップの種類を示すフラグ。指定する場合は、少なくとも1つのフラグを設定する必要があります。{% amendment-disclaimer name="Sponsor" /%} |
| [`SponsorSignature`](#sponsorsignatureフィールド) | オブジェクト | オブジェクト | いいえ | スポンサーシップの署名情報を保持します。{% amendment-disclaimer name="Sponsor" /%} |
| `SourceTag`          | 数値                  | UInt32            | いいえ     | この支払いの理由、またはこのトランザクションの実行元である送信者を識別するために使用される任意の整数。一般的に、返金については、最初の支払いの`SourceTag`を返金の`DestinationTag`として指定する必要があります。 |
| `SigningPubKey`      | 文字列 - 16進数        | Blob              | _署名時に自動追加_ | このトランザクションへの署名に使用される秘密鍵に対応する公開鍵。空文字列の場合は、代わりに`Signers`フィールドにマルチシグが保持されていることを示します。 |
| `TicketSequence`     | 数値                  | UInt32            | いいえ     | `Sequence`番号の代わりに使用する[チケット](../../../concepts/accounts/tickets.md)のシーケンス番号。このフィールドを指定する場合、`Sequence`は`0`でなければなりません。`AccountTxnID`と併用することはできません。 |
| `TxnSignature`       | 文字列 - 16進数        | Blob              | _署名時に自動追加_ | このトランザクションが、発信元であると主張しているアカウントから発信されたものであることを検証するための署名。 |

[自動入力可能]: #自動入力可能なフィールド

{% badge href="https://github.com/XRPLF/rippled/releases/tag/0.28.0" %}削除: rippled 0.28.0{% /badge %}: トランザクションの`PreviousTxnID`フィールドは、[`AccountTxnID`](#accounttxnid)フィールドに置き換えられました。この文字列/UInt256フィールドは、過去に発生したトランザクションの一部に記述されています。このフィールドは、一部の[レジャーオブジェクト](../ledger-data/index.md)にある`PreviousTxnID`という同じ名前のフィールドとは無関係です。


## AccountTxnID

<!-- SPELLING_IGNORE: AccountTxnID -->

`AccountTxnID`フィールドを使用すると、同じアカウントから送信された直前のトランザクションが特定の[トランザクションハッシュ][識別用ハッシュ]を持っていない限り、現在のトランザクションが有効にならないように、トランザクションどうしをチェーンにすることができます。

アカウントを _変更した_ 最後のトランザクション（送信者を問わない）を追跡する`PreviousTxnID`フィールドとは異なり、`AccountTxnID`はアカウントが _送信した_ 最後のトランザクションを追跡します。`AccountTxnID`を使用するには、アカウントの直前のトランザクションのIDがレジャーで追跡されるよう、最初に[`asfAccountTxnID`](types/accountset.md#accountsetのフラグ)フラグを有効にする必要があります（一方、`PreviousTxnID`は常に追跡されます）。

このフィールドが有用になるのは、例えば、トランザクション送信用のプライマリーシステムと受動的なバックアップシステムを運用している場合です。受動的なバックアップシステムがプライマリーから切断されたものの、プライマリーが完全に稼働停止となったわけではなく、両システムが同時に稼働を開始した場合は、トランザクションが2回送信される、あるいはまったく送信されないなど、深刻な問題が発生するおそれがあります。`AccountTxnID`を使用してトランザクションどうしをチェーンにすると、両方のシステムがアクティブになったときも、有効なトランザクションを送信できるのはいずれか一方のみとなります。

`AccountTxnID`フィールドは、[Ticket](../../../concepts/accounts/tickets.md)を使用するトランザクションでは使用できません。`AccountTxnID`を使用するトランザクションは、[トランザクションキュー](../../../concepts/transactions/transaction-queue.md)に入れることができません。


## 自動入力可能なフィールド <a id="auto-fillable-fields"></a>

一部のフィールドについては、トランザクションの署名前に、`xrpld`サーバまたは[クライアントライブラリ](../../client-libraries.md)によって値を自動入力できます。値を自動入力するには、最新の状態を取得するためのXRP Ledgerへのアクティブな接続が必要です。したがって、オフラインでは実行できません。詳細はライブラリによって異なりますが、自動入力では常に、少なくとも以下のフィールドに適切な値が提供されます。

* `Fee` - ネットワークに基づいて[トランザクションコスト][]を自動的に入力します。

    {% admonition type="info" name="注記" %}`xrpld`の[signメソッド][]を使用するときは、`fee_mult_max`パラメーターと`fee_div_max`パラメーターを使用して、自動入力値の上限を設定できます。{% /admonition %}

* `Sequence` - トランザクションを送信する側のアカウントの次のシーケンス番号を自動的に使用します。

本番システムについては、これらのフィールドの値がサーバによって入力される状態に _しない_ ことをお勧めします。例えば、ネットワークの負荷が一時的に急上昇したためにトランザクションコストが高騰した場合、トランザクションによっては、一時的な高額のコストを支払うよりも、コストが低下するのを待ってから送信したほうが好ましいことがあります。

[Paymentトランザクション][]タイプの[`Paths`フィールド](types/payment.md#パス)についても、値を自動入力できます。

## Delegate

`Delegate`フィールドが指定されている場合、このトランザクションは`Account`フィールドのアカウントに代わって、別のアカウントによって送信されています。`Account`フィールドのアカウントが _委任元アカウント_ であり、`Delegate`フィールドのアカウントが _委任先アカウント_ です。トランザクションは、以下の例外を除き、委任元アカウントが送信したものとして機能します。

- 署名は委任先アカウントにとって有効なものでなければなりません（委任先アカウントが承認しているマスターキー、レギュラーキー、またはマルチシグのリストで署名できます）。
- トランザクションコスト（`Fee`フィールド）は、委任先アカウントが支払います。

この方法でトランザクションを送信できるのは、委任元アカウントが委任先アカウントに適切なトランザクション権限を付与している場合のみです。詳細は、[権限の委任](../../../concepts/accounts/permission-delegation.md)をご覧ください。

{% amendment-disclaimer name="PermissionDelegationV1_1" /%}

## Flagsフィールド

`Flags`フィールドには、トランザクションの動作を調整する各種のオプションを設定できます。オプションは、ビット単位のOR演算と組み合わせることで複数のフラグを同時に設定できるバイナリー値として表現します。

トランザクションで所定のフラグが有効になっているかどうかを確認するには、ビット単位のAND演算子をフラグの値と`Flags`フィールドに対して使用します。結果が0の場合は無効になっていることを示し、結果がフラグ値と等しい場合は有効になっていることを示します（その他の結果の場合は、実行した操作に誤りがあることを示します）。

ほとんどのフラグは、特定のタイプのトランザクションに対してのみ意味を持ちます。複数のタイプのトランザクションに対して、同一のビット単位の値をフラグに再利用できるため、フラグの設定と読み取りでは`TransactionType`フィールドに留意することが重要です。

フラグとして定義されていないビットは、0にする必要があります（[fix1543 amendment][]では、一部のタイプのトランザクションについて、このルールが適用されます。デフォルトでは、ほとんどのタイプのトランザクションでこのルールが強制されます）。

### グローバルフラグ

すべてのトランザクションにグローバルに適用されるフラグは、以下のとおりです。

| フラグの名前            | 16進値        | 10進値        | 説明                      |
|:----------------------|:-------------|:--------------|:--------------------------|
| `tfFullyCanonicalSig` | `0x80000000` | 2147483648    | **非推奨** 効果はありません。（[RequireFullyCanonicalSig amendment][]が有効になっていない場合、このフラグは[完全に正規の署名](../../../concepts/transactions/finality-of-results/transaction-malleability.md#代替secp256k1署名)を強制します。） |
| `tfInnerBatchTxn`     | `0x40000000` | 1073741824    | このフラグは、トランザクションが[Batch][]トランザクションの内部トランザクションである場合にのみ使用されます。これは、トランザクションが署名されていないことを示します。このフラグを含む通常のトランザクションは拒否されます。 |

[signメソッド][]（または「署名と送信」モードの[submitメソッド][]）を使用すると、`xrpld`は、`Flags`フィールドがすでに存在している場合を除き、`tfFullyCanonicalSig`を有効にした状態で`Flags`フィールドを追加します。`tfFullyCanonicalSig`フラグは、`Flags`が明示的に指定されている場合、自動的には有効になりません。また、[sign_forメソッド][]を使用してマルチシグトランザクションに署名を追加する場合も、このフラグは自動的には有効になりません。

{% admonition type="info" name="注記" %}`tfFullyCanonicalSig`フラグは、2014年から2020年まで、従来の署名ソフトウェアとの互換性を維持しながら[トランザクションの展性](../../../concepts/transactions/finality-of-results/transaction-malleability.md)から保護するために使用されていました。[RequireFullyCanonicalSig amendment][]により、こうした従来のソフトウェアとの互換性は終了し、この保護がすべてのトランザクションのデフォルトになりました。RequireFullyCanonicalSigが有効になっていない[並列ネットワーク](../../../concepts/networks-and-servers/parallel-networks.md)を使用している場合は、トランザクションの展性から保護するために、常に`tfFullyCanonicalSig`フラグを有効にする必要があります。{% /admonition %}

### フラグの範囲

トランザクションの`Flags`フィールドには、さまざまなレベルや状況に適用されるフラグを設定できます。個々の状況に関するフラグは、以下の範囲に限定されます。

| 範囲の名前         | ビットマスク    | 説明                                        |
|:-----------------|:-------------|:-------------------------------------------|
| ユニバーサルフラグ   | `0xff000000` | すべてのタイプのトランザクションに対して一様に適用されるフラグ。 |
| タイプに基づくフラグ | `0x00ff0000` | フラグを使用する[トランザクションのタイプ](types/index.md)に応じて意味が異なるフラグ。 |
| 予約済みのフラグ    | `0x0000ffff` | 現時点では定義されていないフラグ。トランザクションが有効になるのは、これらのフラグが無効になっている場合のみです。 |

{% admonition type="info" name="注記" %}[AccountSetトランザクション][]タイプには、タイプに基づくフラグと似た目的を果たす[ビット単位ではない独自のフラグ](types/accountset.md#accountsetのフラグ)があります。[レジャーオブジェクト](../ledger-data/ledger-entry-types/index.md)にも、異なるビット単位のフラグが定義される`Flags`フィールドがあります。{% /admonition %}

## Memosフィールド

`Memos`フィールドは、トランザクションに関する任意のメッセージデータを保持します。このフィールドは、オブジェクトの配列として表現します。各オブジェクトには唯一のフィールド`Memo`があり、このフィールドは、以下のフィールドを*1つ以上*持つ別のオブジェクトを保持しています。

| フィールド       | 型     | [内部の型][] | 説明                               |
|:-------------|:-------|:------------|:-----------------------------------|
| `MemoData`   | 文字列 | Blob        | 通例、メモの内容を保持する任意の16進値。 |
| `MemoFormat` | 文字列 | Blob        | URLで使用できる文字を表現する16進値。通例、メモのエンコード方法に関する情報を保持しています（[MIMEタイプ](http://www.iana.org/assignments/media-types/media-types.xhtml)など）。 |
| `MemoType`   | 文字列 | Blob        | URLで使用できる文字を表現する16進値。通例、このメモのフォーマットを定義する一意の関係（[RFC 5988](http://tools.ietf.org/html/rfc5988#section-4)に準拠）。 |

`MemoType`フィールドと`MemoFormat`フィールドには、以下の文字のみを使用できます。 `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-._~:/?#[]@!$&'()*+,;=%`

`Memos`フィールドのサイズの上限は1KBです（バイナリーフォーマットでシリアル化されている場合）。

以下に、Memosフィールドが定義されているトランザクションの例を示します。

```json
{
    "TransactionType": "Payment",
    "Account": "rMmTCjGFRWPz8S2zAUUoNVSQHxtRQD4eCx",
    "Destination": "r3kmLJN5D28dHuH8vZNUZpMC43pEHpaocV",
    "Memos": [
        {
            "Memo": {
                "MemoType": "687474703a2f2f6578616d706c652e636f6d2f6d656d6f2f67656e65726963",
                "MemoData": "72656e74"
            }
        }
    ],
    "Amount": "1"
}
```

## NetworkIDフィールド <a id="networkid-field"></a>
{% badge href="https://github.com/XRPLF/rippled/releases/tag/1.11.0" %}新規: rippled 1.11.0{% /badge %}

`NetworkID`フィールドは「クロスチェーン」トランザクションのリプレイ攻撃に対する保護であり、同じトランザクションがコピーされ、意図していない[並列ネットワーク](../../../concepts/networks-and-servers/parallel-networks.md)で実行されることを防ぎます。既存のチェーンとの互換性のため、ネットワークIDが1024以下のネットワークでは`NetworkID`フィールドを省略する必要がありますが、ネットワークIDが1025以上のネットワークでは`NetworkID`フィールドを含める必要があります。以下の表は、さまざまな既知のネットワークのステータスと値を示しています。

| ネットワーク                      | ID    | `NetworkID`フィールド                                  |
| ------------------------------- | ----- | ----------------------------------------------------- |
| Mainnet                         | 0     | 使用不可                                               |
| Testnet                         | 1     | 使用不可                                               |
| Devnet                          | 2     | 使用不可                                               |
| Batch Testnet                   | 21336 | 必須                                                   |
| Xahau Mainnet                   | 21337 | 必須                                                   |
| Xahau Testnet                   | 21338 | 必須                                                   |
| JS Hooks Testnet                | 31338 | 必須                                                   |

トランザクションのリプレイ攻撃は理論的には可能ですが、2つ目のネットワークに特定の条件が必要です。次のすべてが真でなければなりません。

- トランザクションの送信者が2つ目のネットワーク上の資金供給済みアカウントである。
- 2つ目のネットワーク上の送信者の`Sequence`番号がトランザクションの`Sequence`と一致するか、トランザクションが2つ目のネットワークで利用可能な[Ticket](../../../concepts/accounts/tickets.md)を使用している。
- トランザクションが`LastLedgerSequence`フィールドを持っていないか、2つ目のレジャーの現在のレジャーインデックスよりも高い値を指定している。
    - Mainnetは一般的に、テストネットワークやサイドチェーンよりもレジャーインデックスが高いため、トランザクションが`LastLedgerSequence`を本来の意図通りに使用している場合、Mainnetのトランザクションをサイドチェーンやテストネットワークでリプレイするほうがその逆よりも容易です。
- 両方のネットワークのIDが1024以下であるか、両方のネットワークが同じIDを使用しているか、2つ目のネットワークが`NetworkID`フィールドを必要としないかのいずれか。


## Signersフィールド

`Signers`フィールドには、最大32のキーペアから取得された署名を保持し、トランザクションを承認するための[マルチシグ](../../../concepts/accounts/multi-signing.md)が含まれています。`Signers`リストはオブジェクトの配列であり、各オブジェクトが1つの`Signer`フィールドを保持しています。`Signer`フィールドには、以下の入れ子フィールドがあります。

| フィールド          | 型     | [内部の型][] | 説明                            |
|:----------------|:-------|:------------|:--------------------------------|
| `Account`       | 文字列 | AccountID   | 署名者リストに記述され、この署名に関連付けられているアドレス。 |
| `TxnSignature`  | 文字列 | Blob        | `SigningPubKey`を使用して検証できる、このトランザクションの署名。 |
| `SigningPubKey` | 文字列 | Blob        | この署名の作成に使用される公開鍵。 |

`SigningPubKey`は、`Account`アドレスに関連付けられているキーでなければなりません。参照されている`Account`が、レジャーにある資金供給済みアカウントである場合、`SigningPubKey`には、そのアカウントの現在のレギュラーキー（設定されている場合）を指定できます。また、[`lsfDisableMaster`](../ledger-data/ledger-entry-types/accountroot.md#accountrootのフラグ)フラグが有効になっている場合を除き、そのアカウントのマスターキーを指定することもできます。参照されている`Account`アドレスが、レジャーの資金供給済みのアカウントではない場合、`SigningPubKey`は、そのアドレスに関連付けられているマスターキーでなければなりません。

署名の検証は大量の演算能力を消費するタスクであるため、マルチシグトランザクションをネットワークに中継するには、追加のXRPがコストとしてかかります。マルチシグに含まれている署名ごとに、トランザクションに必要な[トランザクションコスト][]が増加します。例えば、トランザクションをネットワークに中継するための現在の最小トランザクションコストが`10000`dropである場合、`Signers`配列に3つのエントリが含まれているマルチシグトランザクションを中継するには、`Fee`の値を少なくとも`40000`dropにする必要があります。

## SponsorFlagsフィールド

`SponsorFlags`フィールドでは、参加したいスポンサーシップの種類を指定できます。{% amendment-disclaimer name="Sponsor" /%}

| フラグの名前           | 16進値        | 10進値  | 説明 |
| :------------------ | :----------- | :------ | :---------- |
| `spfSponsorFee`     | `0x00000001` | 1       | トランザクションの手数料をスポンサーします。 |
| `spfSponsorReserve` | `0x00000002` | 2       | トランザクションで作成されるオブジェクトの準備金をスポンサーします。 |

{% admonition type="info" name="注記" %}
両方のフラグを1つのトランザクションで同時に使用できます。`Sponsor`フィールドを含める場合は、少なくとも1つのフラグを設定する必要があります。
{% /admonition %}

[疑似トランザクション](pseudo-transaction-types/index.md)は、そもそも手数料も準備金も持たないため、いずれのスポンサーシップフラグもサポートしません。それ以外のすべてのトランザクションについては、以下のとおりです。

- **`spfSponsorFee`**フラグは、あらゆるトランザクションタイプで使用できます。
- **`spfSponsorReserve`**フラグは、以下のトランザクションでのみ使用できます。
    - [AccountSet](types/accountset.md)
    - [CheckCancel](types/checkcancel.md)
    - [CheckCash](types/checkcash.md)
    - [CheckCreate](types/checkcreate.md)
    - [Clawback](types/clawback.md)
    - [CredentialAccept](types/credentialaccept.md)
    - [CredentialCreate](types/credentialcreate.md)
    - [CredentialDelete](types/credentialdelete.md)
    - [DelegateSet](/docs/references/protocol/transactions/types/delegateset.md)
    - [DepositPreauth](types/depositpreauth.md)
    - [EscrowCancel](types/escrowcancel.md)
    - [EscrowCreate](types/escrowcreate.md)
    - [EscrowFinish](types/escrowfinish.md)
    - [MPTokenAuthorize](types/mptokenauthorize.md)
    - [MPTokenIssuanceCreate](types/mptokenissuancecreate.md)
    - [MPTokenIssuanceDestroy](types/mptokenissuancedestroy.md)
    - [MPTokenIssuanceSet](types/mptokenissuanceset.md)
    - [Payment](types/payment.md)
    - [PaymentChannelClaim](types/paymentchannelclaim.md)
    - [PaymentChannelCreate](types/paymentchannelcreate.md)
    - [PaymentChannelFund](types/paymentchannelfund.md)
    - [SetRegularKey](types/setregularkey.md)
    - [SignerListSet](types/signerlistset.md)
    - [SponsorshipTransfer](/docs/references/protocol/transactions/types/sponsorshiptransfer.md)
    - [TrustSet](types/trustset.md)

### Batchトランザクション

`SponsorFlags`は、[Batch][]の中で互いに逆のレベルに適用されます。外部のBatchトランザクションは`spfSponsorReserve`をサポートしないため、準備金のスポンサーシップは _内部_ トランザクションに指定します。手数料のスポンサーシップはその逆で、`spfSponsorFee`は _外部_ のBatchトランザクションでのみ有効であり、内部トランザクションでは無効です。

準備金のスポンサーが内部トランザクションに共同署名する場合、その内部トランザクションの`SponsorSignature`フィールドは存在していなければなりませんが、**空**でなければなりません。スポンサーは代わりに外部のBatchトランザクションに署名し、`BatchSigners`のエントリとして現れます。

### 委任されたトランザクション

[`Delegate`](#delegate)フィールドを含むトランザクションについては、以下の点に注意してください。

- 手数料のスポンサーシップは、トランザクションの手数料を支払う委任先アカウントに適用されます。
- 事前に資金提供された手数料のスポンサーシップを使用する場合、トランザクションを送信するアカウントではなく、 _委任先アカウント_ がスポンサーを受ける側（スポンシー）でなければなりません。
- 準備金のスポンサーシップは委任と組み合わせることができず、`temINVALID`で失敗します。

## SponsorSignatureフィールド

`SponsorSignature`フィールドは、スポンサーの署名情報を保持するオブジェクトです。{% amendment-disclaimer name="Sponsor" /%}

| フィールド          | JSONの型 | [内部の型][] | 必須?  | 説明  |
| :-------------- | :-------- | :---------- | :----- | :----------- |
| `SigningPubKey` | 文字列    | Blob        | いいえ | 単一署名の場合の、`Sponsor`の`SigningPubKey`。 |
| `TxnSignature`  | 文字列    | Blob        | いいえ | 単一署名の場合の、スポンサーの承認を示すトランザクションの署名。役割ごとに固有のハッシュプレフィックスを使用して生成する必要があります。標準のトランザクション署名は拒否されます。{% amendment-disclaimer name="fixCleanup3_4_0" mode="updated" /%} |
| `Signers`       | 配列      | 配列        | いいえ | スポンサーが[マルチシグ](../../../concepts/accounts/multi-signing.md)を使用する場合の、スポンサーの署名者による承認を示すトランザクションの署名の配列。役割ごとに固有のハッシュプレフィックスを使用して生成する必要があります。標準のトランザクション署名は拒否されます。{% amendment-disclaimer name="fixCleanup3_4_0" mode="updated" /%} |

これらのフィールドはトランザクションの署名には含まれませんが、保存されるトランザクションには含まれます。`TxnSignature`の使用に追加のトランザクション手数料はかかりません。

{% admonition type="info" name="注記" %}
スポンサーの署名が必要となるのは、事前に資金提供された`Sponsorship`レジャーエントリが存在しない場合、または[Sponsorshipレジャーエントリ](/docs/references/protocol/ledger-data/ledger-entry-types/sponsorship.md)で`lsfSponsorshipRequireSignForFee`フラグまたは`lsfSponsorshipRequireSignForReserve`フラグが有効になっている場合のみです。
{% /admonition %}

### トランザクション手数料の計算

`SponsorSignature.Signers`フィールドが必要な場合、処理すべき署名が増えるため、トランザクションの合計手数料が増加します。これはマルチシグの追加手数料と同様です。

署名に関する合計手数料の計算式は以下のとおりです。

```text
(1 + 署名者の数 + スポンサー側の署名者の数) × 基本手数料 (+ トランザクション固有の手数料)
```

{% raw-partial file="/@l10n/ja/docs/_snippets/common-links.md" /%}
