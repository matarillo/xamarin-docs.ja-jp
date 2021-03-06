---
title: 指紋認証ガイダンス
ms.prod: xamarin
ms.assetid: B40332CC-8123-4150-B47E-996214388842
ms.technology: xamarin-android
author: davidortinau
ms.author: daortin
ms.date: 02/16/2018
ms.openlocfilehash: e955d4f96724bd5682e7d0e6db2c36fa1b7810f4
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73027428"
---
# <a name="fingerprint-authentication-guidance"></a>指紋認証ガイダンス

## <a name="fingerprint-authentication-guidance"></a>指紋認証ガイダンス

これで、Android 6.0 フィンガープリント認証に関する概念と Api が見られました。次は、指紋 Api の使用に関する一般的なアドバイスについて説明します。

1. **Android サポートライブラリ V4 互換性 api を使用する**&ndash; これにより、コードから API チェックを削除し、アプリケーションが可能な限り多くのデバイスを対象にすることによって、アプリケーションコードを簡略化できます。
2. 指紋認証の代替手段を**提供**&ndash; 指紋認証は、アプリケーションがユーザーを認証するための優れた迅速な方法ですが、常に機能しているか、使用可能であると想定することはできません。 指紋スキャナーが故障した場合、レンズが汚れている場合、ユーザーが指紋認証を使用するようにデバイスを構成していない場合、または指紋が紛失したことが原因である可能性があります。 また、ユーザーがアプリケーションで指紋認証を使用しないようにすることもできます。 このような理由から、Android アプリケーションでは、ユーザー名やパスワードなどの代替認証プロセスを提供する必要があります。
3. **Google の指紋アイコンを使用**して &ndash; すべてのアプリケーションで、google によって提供されるものと同じ指紋アイコンを使用する必要があります。 標準アイコンを使用すると、アプリ内の指紋認証が使用されている場所を Android ユーザーが簡単に認識できるようになります。 
    
    ![Android の指紋アイコン](summary-images/ic-fp-40px.png)
    
4. **ユーザーに通知**する &ndash; アプリケーションは、指紋スキャナーがアクティブであり、タッチまたはスワイプを待機していることをユーザーに通知します。 

## <a name="summary"></a>まとめ

指紋認証は、ユーザーがアプリ内購入などの重要な機能を簡単に操作できるようにするために、Xamarin Android アプリケーションでユーザーを迅速に検証するための優れた方法です。 このガイドでは、Android 6.0 指紋 API を Xamarin Android アプリケーションに組み込むために必要な概念とコードについて説明しました。

最初に、指紋 API 自体、`FingerprintManager` (および `FingerprintManagerCompat`) について説明しました。 ここでは、`FingerprintManager.AuthenticationCallbacks` 抽象クラスをアプリケーションによって拡張する必要があること、および指紋ハードウェアとアプリケーション自体の仲介役として使用する方法について説明します。 次に、Java `Cipher` オブジェクトを使用して、フィンガープリントスキャナーの結果の整合性を確認する方法について説明します。 最後に、デバイスに指紋を登録し、 **adb**を使用してエミュレーターで指紋のスワイプをシミュレートする方法を説明することによって、テストについて少し触れます。 

まだ行っていない場合は、このガイドに付属する[サンプルアプリケーション](https://github.com/xamarin/monodroid-samples/tree/master/FingerprintGuide)を参照してください。 [フィンガープリントダイアログのサンプル](https://docs.microsoft.com/samples/xamarin/monodroid-samples/android-m-fingerprintdialog)は Java から Xamarin android に移植されており、android アプリケーションに指紋認証を追加する方法についてもう1つの例を示しています。

## <a name="related-links"></a>関連リンク

- [指紋ガイドのサンプルアプリ](https://github.com/xamarin/monodroid-samples/tree/master/FingerprintGuide)
- [フィンガープリントダイアログのサンプル](https://docs.microsoft.com/samples/xamarin/monodroid-samples/android-m-fingerprintdialog)
- [フィンガープリントアイコン](https://raw.githubusercontent.com/xamarin/monodroid-samples/master/FingerprintGuide/FingerprintSampleApp/Resources/drawable-hdpi/ic_fp_40px.png)
