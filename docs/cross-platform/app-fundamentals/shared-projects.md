---
title: "共有プロジェクト"
description: "共有プロジェクトでは、別のアプリケーション プロジェクトの数によって参照されている共通のコードを記述できます。 コードでは、各参照元のプロジェクトの一部としてコンパイルされ、共有コード ベースにプラットフォーム固有の機能を組み込むためのコンパイラ ディレクティブを含めることができます。"
ms.topic: article
ms.prod: xamarin
ms.assetid: 191c71fb-44a4-4e6c-af4b-7b1107dce6af
ms.technology: xamarin-cross-platform
author: asb3993
ms.author: amburns
ms.date: 03/23/2017
ms.openlocfilehash: 0ab1daa9ce76900067f374cda58040354688c7be
ms.sourcegitcommit: 6cd40d190abe38edd50fc74331be15324a845a28
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/27/2018
---
# <a name="shared-projects"></a>共有プロジェクト

_共有プロジェクトでは、別のアプリケーション プロジェクトの数によって参照されている共通のコードを記述できます。コードでは、各参照元のプロジェクトの一部としてコンパイルされ、共有コード ベースにプラットフォーム固有の機能を組み込むためのコンパイラ ディレクティブを含めることができます。_

共有プロジェクト (共有アセット プロジェクトと呼ばれることもあります) を使用して、Xamarin アプリケーションを含む複数のターゲット プロジェクト間で共有されるコードを記述できます。

共有プロジェクトを参照しているプロジェクトのサブセットにコンパイルされるプラットフォーム固有のコードを条件付きで含めることができるようにコンパイラ ディレクティブをサポートしています。 コンパイラ ディレクティブを管理し、各アプリケーションでコードがどのように表示されるかを視覚化するために IDE のサポートもあります。

を使用した過去ファイル リンク プロジェクト間でコードを共有する場合、共有プロジェクトは IDE サポートが大幅に向上が、同様の方法で動作します。


# <a name="requirements"></a>必要条件

Xamarin Studio 5 および Visual Studio 2013 Update 2 (注を参照してください) のサポートが追加されたプロジェクトを共有します。

> [!IMPORTANT]
>  Microsoft では、この新しいプロジェクトの種類をリリースしました。**共有プロジェクト ([Visual Studio 拡張機能のプレビューをダウンロードして](http://visualstudiogallery.msdn.microsoft.com/315c13a7-2787-4f57-bdf7-adae6ed54450))** - Visual Studio 2013 Update 2 (April 2014)。 Microsoft を参照してください[Windows Phone 8.1](http://blogs.msdn.com/b/visualstudio/archive/2014/04/08/building-windows-phone-8-1-apps-in-html.aspx)と[Microsoft ストア](http://msdn.microsoft.com/en-us/library/windows/apps/dn609832.aspx#CrossPlatform)これらのプラットフォームでの動作についての詳細についてはドキュメントです。




 <a name="Walkthrough" />


# <a name="what-is-a-shared-project"></a>共有プロジェクトとは何ですか。

その他のほとんどのプロジェクトの種類とは異なり、共有プロジェクトが出力 (DLL 形式)、それを参照する各プロジェクトにコードをコンパイルする代わりにします。 次の図に示す - 概念的には、共有プロジェクトの内容全体は各参照元のプロジェクトに「コピー」と、それらの一部のようにコンパイルします。

 ![](shared-projects-images/sharedassetproject.png "共有プロジェクトのアーキテクチャ")

共有プロジェクトのコードは、コンパイラ ディレクティブを有効または無効にすると、アプリケーションによってプロジェクトは、ダイアグラムで色分けされたプラットフォーム ボックスによって提示されると、コードを使用してコードのセクションが指定を含めることができます。

共有プロジェクトは自身にコンパイルされていない、他のプロジェクトに含めることができるソース コード ファイルのグループとしてのみ存在しています。 コードとして効果的にコンパイルされた他のプロジェクトによって参照された場合、*一部*そのプロジェクトのです。 共有プロジェクトには、その他のプロジェクトの種類 (他の共有プロジェクトを含む) を参照できません。

Android アプリケーション プロジェクトが他の Android アプリケーション プロジェクトを参照できません - たとえば、Android の単体テスト プロジェクトが Android アプリケーション プロジェクトを参照できないことに注意してください。 この制限の詳細については、これを参照して[フォーラムのディスカッション](http://forums.xamarin.com/discussion/comment/98092/)です。

# <a name="visual-studio-for-mactabvsmac"></a>[Visual Studio for Mac](#tab/vsmac)



<a name="Xamarin_Studio_Walkthrough" />

# <a name="visual-studio-for-mac-walkthrough"></a>Visual Studio for Mac のチュートリアル


このセクションで作成および共有使用してプロジェクトを Visual Studio for mac を使用する方法について説明します。 参照してくださいに[共有プロジェクトの使用例](#Shared_Project_Example)コード例全体についてのセクションでします。


## <a name="creating-a-shared-project"></a>共有プロジェクトを作成します。


移動する、新しい共有プロジェクトを作成する**ファイル > 新しいソリューションをしています.**名前を選択します。


![](shared-projects-images/xs-newsolution.png "新しいソリューション")


ソリューション ファイルを右クリックして] を選択して、ソリューションに新しい共有プロジェクトを追加することも**追加 > [新しいプロジェクトの追加.**.新しい共有プロジェクトは、次に示すように注意してください参照されていないか、コンポーネント ノードです。これらは、共有プロジェクトのサポートされていません。


![](shared-projects-images/xs-empty.png "空の共有プロジェクト")


共有プロジェクトを使用する (iOS または Android のアプリケーションまたはライブラリを PCL プロジェクトなど) には、少なくとも 1 つのビルドできないプロジェクトによって参照されていることが必要です。 共有プロジェクトは取得ときはコンパイルされませんが、または参照するための構文 (その他の) は関係ありませんエラーが強調表示されませんが、他のものによって参照されてまでです。



共有プロジェクトへの参照を追加すると、通常のライブラリ プロジェクトを参照するいると同じ方法は実行されます。 このスクリーン ショットは、共有プロジェクトを参照する Xamarin.iOS プロジェクトを示します。


![](shared-projects-images/xs-reference.png "共有プロジェクトにプロジェクト参照")


共有プロジェクトが別のライブラリまたはアプリケーションによって参照されているとは、ソリューションをビルドし、コードですべてのエラーを表示します。 によって、共有プロジェクトを参照するときに_2 またはそれ以上_他のプロジェクトでは、メニューが表示されます、ソース コード エディターの上部左に表示は、プロジェクトは、このファイルを参照を選択します。



## <a name="shared-project-options"></a>共有プロジェクトのオプション


共有プロジェクトを右クリックして選択して**オプション**以下の設定よりも他のプロジェクトの種類があります。 (自分で) 共有プロジェクトはコンパイルされていないために、出力、またはコンパイラ オプション、プロジェクトの構成、アセンブリの署名、またはカスタムのコマンドを設定できません。 共有プロジェクトのコードは、すべてが参照しているからこれらの値を効果的に継承します。



**オプション**画面を次のプロジェクトに**名前**と**Namespace の既定の**は一般に、変更する 2 つだけ設定します。


![](shared-projects-images/xs-sharedprojectoptions.png "共有プロジェクトのオプション")



# <a name="visual-studiotabvswin"></a>[Visual Studio](#tab/vswin)



<a name="Visual_Studio_Walkthrough" />

# <a name="visual-studio-walkthrough"></a>Visual Studio チュートリアル


このセクションで作成して Visual Studio を使用して、共有プロジェクトを使用する方法について説明します。 参照してくださいに[共有プロジェクト例](#Shared_Project_Example)完全な実装についてのセクションです。


## <a name="creating-a-shared-project"></a>共有プロジェクトを作成します。


移動する、新しい共有プロジェクトを作成する**ファイル > 新しいソリューションをしています.**プロジェクトおよびソリューションの名前を選択します。


![](shared-projects-images/vs-newsolution.png "新しいソリューション")


ソリューション ファイルを右クリックして] を選択して、ソリューションに新しい共有プロジェクトを追加することも**追加 > [新しいプロジェクト.**.新しい共有プロジェクトの検索 (クラス ファイルを追加した) 後、次のように - 参照されていないことを確認またはコンポーネント ノードです。これらは、共有プロジェクトのサポートされていません。


![](shared-projects-images/vs-empty.png "空の共有プロジェクト")


共有プロジェクトを使用する (iOS または Android のアプリケーションまたはライブラリを PCL プロジェクトなど) には、少なくとも 1 つのビルドできないプロジェクトによって参照されていることが必要です。 共有プロジェクトは取得ときはコンパイルされませんが、または参照するための構文 (その他の) は関係ありませんエラーが強調表示されませんが、他のものによって参照されてまでです。



共有プロジェクトへの参照を追加すると、通常のライブラリ プロジェクトを参照するいると同じ方法は実行されます。 このスクリーン ショットは、共有プロジェクトを参照する Xamarin.iOS プロジェクトを示します。


![](shared-projects-images/vs-reference.png "共有プロジェクトにプロジェクト参照")


共有プロジェクトが別のライブラリまたはアプリケーションによって参照されているとは、ソリューションをビルドし、コードですべてのエラーを表示します。 によって、共有プロジェクトを参照するときに_2 またはそれ以上_先となるプロジェクトが現在のコード ファイルを参照してソース コード エディターの左上の他のプロジェクトでは、メニューが表示されます。


## <a name="shared-project-properties"></a>共有プロジェクトのプロパティ


選択すると、共有プロジェクトがある設定は少なくなってその他のプロジェクトの種類よりプロパティ パネルで。 (自分で) 共有プロジェクトはコンパイルされていないために、出力、またはコンパイラ オプション、プロジェクトの構成、アセンブリの署名、またはカスタムのコマンドを設定できません。 共有プロジェクトのコードは、すべてが参照しているからこれらの値を効果的に継承します。



**プロパティ**パネルは、次に示す、**ルート Namespace**唯一の設定を変更できます。


![](shared-projects-images/vs-sharedprojectproperties.png "共有プロジェクトのプロパティ")



-----

 <a name="Shared_Project_Example" />


# <a name="shared-project-example"></a>共有プロジェクトの使用例

[Tasky](https://github.com/xamarin/mobile-samples/tree/master/Tasky)例の両方、iOS、Android、Windows Phone アプリケーションで使用される一般的なコードを含むプロジェクトの共有を使用します。 両方の`SQLite.cs`と`TaskRepository.cs`ソース コード ファイルを使えるコンパイラ ディレクティブ (たとえばです。 `#if __ANDROID__`) を参照するアプリケーションの各別の出力を生成します。

完全なソリューションの構造を次に示します (Mac と Visual Studio の Visual Studio でそれぞれ)。

# <a name="visual-studio-for-mactabvsmac"></a>[Visual Studio for Mac](#tab/vsmac)

 ![](shared-projects-images/xs-examplesolution.png "Mac ソリューション用の visual Studio")

# <a name="visual-studiotabvswin"></a>[Visual Studio](#tab/vswin)

 ![](shared-projects-images/vs-examplesolution.png "Visual Studio のソリューション")

-----

Windows Phone プロジェクトから誘導できます Visual Studio 内で、Mac の場合でも、そのプロジェクトの種類は for mac の Visual Studio でのコンパイルでサポートされていません

実行中のアプリケーションは、以下に示します。

 ![](shared-projects-images/example.png "iOS、Android、Windows Phone の例")

 <a name="Summary" />


# <a name="summary"></a>まとめ

このドキュメントでは、共有プロジェクトの動作方法、作成して Visual Studio for Mac と Visual Studio の両方で使用を説明し、共有プロジェクトの動作を示す簡単なサンプル アプリケーションが導入されました。

## <a name="related-links"></a>関連リンク

- [Tasky サンプル アプリ](https://github.com/xamarin/mobile-samples/tree/master/Tasky)
- [ポータブル クラス ライブラリ (サンプル)](~/cross-platform/app-fundamentals/pcl.md)
- [コード共有のオプション (サンプル)](~/cross-platform/app-fundamentals/code-sharing.md)