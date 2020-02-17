---
ms.openlocfilehash: dffa9c202d42cfb12ed67e219b4e3155e4e554d0
ms.sourcegitcommit: 57f815bf0024b1afe9754c0e28054fc0a53ce302
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/06/2019
ms.locfileid: "70751922"
---
# <a name="contributing"></a>コントリビューション

Xamarin ドキュメントへの投稿に関心をお寄せいただきありがとうございます。

このページでは、[Xamarin ドキュメント](https://docs.microsoft.com/xamarin)のコンテンツを更新するための基本的なプロセスについて説明します。

- [共同作成者ライセンス条項](LICENSE)

## <a name="mt-engine-updates-on-international-docs"></a>Docs国際版の機械翻訳（MT）エンジンの更新

MTコンテンツの品質を改善するために、より良い翻訳を提供できると期待されるニューラルエンジンに移行中です。 MTコンテンツへの投稿は _一時的に_ 無効になります。[詳細はこちらをお読みください](MTEngineUpdate.md)。

## <a name="process-for-contributing"></a>投稿のプロセス

### <a name="small-changes--edits"></a>小規模な変更と編集

修正と小規模な更新を行うには、任意のページで **[編集]** ボタンをクリックし、GitHub Web サイトを使用して投稿するか、または次の手順に従います。

1. `MicrosoftDocs/xamarin-docs` リポジトリをフォークします。

2. 変更の`branch`を作成します。

3. コンテンツを記述します。 [テンプレート](contributing-guidelines/template.md)と[スタイル ガイド](contributing-guidelines/voice-tone.md)を参照してください。

4. ご自分のブランチから `MicrosoftDocs/xamarin-docs/live` に Pull Request (PR) を送信します。

5. PR を介してチームによって説明されたとおりに、ご自分のブランチに必要な更新を加えます。

6. フィードバックが適用され、変更が適切であるように見える場合は、保守管理者によって PR がマージされます。 これはまもなく docs.microsoft.com に表示されます。

> [!NOTE]
> PR が既存の問題に対処している場合は、コミット メッセージまたは PR の説明に `Fixes #Issue_Number` キーワードを追加すると、PR がマージされたときに問題が自動的に閉じられます。 詳細については、[コミット メッセージによって問題を閉じる](https://help.github.com/articles/closing-issues-via-commit-messages/)方法についての記事を参照してください。

### <a name="big-changes-or-new-content"></a>大規模な変更または新しいコンテンツ

大規模な投稿と新しいコンテンツの場合は、作成する記事と既存のコンテンツとの関連を説明する[問題を開きます](https://github.com/MicrosoftDocs/xamarin-docs/issues)。 docs フォルダー内の内容は、製品領域 (Android や iOS など) 別に編成されたセクションに編成されています。 新しいコンテンツ用の適切なフォルダーを特定します。 

**記述を開始する前に、問題を介して提案に対するフィードバックを得ます。**

新しいトピックの場合は、この[テンプレート ファイル](../contributing-guidelines/template.md)を開始点として使用できます。 このファイルには、作成のガイドラインが含まれ、作成者情報など、記事ごとに必要なメタデータについても説明されています。

イメージとその他の静的リソースについては、 **\<mypage>-images** という名前のサブフォルダーに追加します。 コンテンツ用の新しいフォルダーを作成する場合は、新しいフォルダーに images フォルダーを追加します。

#### <a name="example-structure"></a>構造の例

```
docs
    /android
        mypage.md
        /mypage-images
            some-image.png
```

適切なマークダウン構文に従ってください。 詳細については、[スタイル ガイド](../contributing-guidelines/template.md)を参照してください。

実際の送信手順は、小規模な変更の場合と同じです ([上記](#process-for-contributing))。

Xamarin チームが PR を確認し、変更が適切であるか、または変更を承認するために必要な他の更新や変更があるかどうかを (PR フィードバックを介して) お知らせします。

フィードバックが適用され、変更が適切であるように見える場合は、保守管理者によって PR がマージされます。

特定の周期で、マスター ブランチのすべてのコミットがライブ サイトにプッシュされるので、 https://docs.microsoft.com/xamarin/ でご自分の投稿を確認できます。

## <a name="dos-and-donts"></a>注意事項

.NET ドキュメントに投稿する際に注意する必要があるガイド ルールの簡単な一覧を次に示します。

- 突然に、巨大な pull requests を送信**しないでください**。 代わりに、問題を提出し、ディスカッションを開始して、大量の時間を費やす前に方向について同意が得られるようにしてください。
- [スタイル ガイド](contributing-guidelines/template.md)と[スタイルとトーン](contributing-guidelines/voice-tone.md)のガイドラインを**お読みください**。
- [テンプレート](contributing-guidelines/template.md) ファイルを作業の開始点として**使用してください**。
- 記事に取り掛かる前に、フォークに個別のブランチを**作成してください**。
- [GitHub フロー ワークフロー](https://guides.github.com/introduction/flow/)に**従ってください**。
- 投稿について頻繁にブログやツイート**を書いてください**。

> [!NOTE]
> 一部のトピックが現在、この記事や[スタイル ガイド](contributing-guidelines/template.md)に指定されたすべてのガイドラインに従っていないことに気付く場合があります。 Microsoft では、サイト全体で一貫性の達成に向けて取り組んでいます。 
