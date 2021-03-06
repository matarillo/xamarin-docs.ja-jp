---
title: KitKat の機能
description: Android 4.4 (KitKat) には、ユーザーと開発者向けの生涯の機能が組み込まれています。 このガイドでは、これらの機能のいくつかについて説明し、KitKat を最大限に活用するのに役立つコード例と実装の詳細を示します。
ms.prod: xamarin
ms.assetid: D3FDEA1C-F076-406F-BCC3-2A55D2C6ADEE
ms.technology: xamarin-android
author: davidortinau
ms.author: daortin
ms.date: 03/01/2018
ms.openlocfilehash: 43061272f3d3486926f38af792ee3b9df0c53670
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73027242"
---
# <a name="kitkat-features"></a>KitKat の機能

_Android 4.4 (KitKat) には、ユーザーと開発者向けの生涯の機能が組み込まれています。このガイドでは、これらの機能のいくつかについて説明し、KitKat を最大限に活用するのに役立つコード例と実装の詳細を示します。_

## <a name="overview"></a>概要

Android 4.4 (API レベル 19) ("KitKat" とも呼ばれます) は、2013の遅延でリリースされました。 KitKat には、次のようなさまざまな新機能と機能強化が用意されています。

- [ユーザーエクスペリエンス](#user_experience)&ndash;、切り替え効果フレームワーク、半透明状態、ナビゲーションバー、全画面表示のイマーシブモードによって、ユーザーのエクスペリエンスを向上させることができます。

- ユーザーの[コンテンツ](#user_content)&ndash; ストレージアクセスフレームワークで簡略化されたユーザーファイルの管理印刷 Api を向上させることで、画像、web サイト、その他のコンテンツの印刷が簡単になります。

- [ハードウェア](#hardware)&ndash; Nfc ホストベースのカードエミュレーションを使用して、すべてのアプリを nfc カードに変換します。`SensorManager` で低電力センサーを実行します。

- [開発者ツール](#developer_tools)は、Android SDK の一部として利用できる、Android Debug Bridge クライアントで動作するスクリーンキャストアプリケーションを &ndash; します。

このガイドでは、既存の Xamarin Android アプリケーションを KitKat に移行するためのガイダンスと、Xamarin Android 開発者向けの KitKat の概要について説明します。

## <a name="requirements"></a>［要件］

KitKat を使用して Xamarin Android アプリケーションを開発するには、次のスクリーンショットに示すように、Android SDK Manager を使用して*Xamarin android 4.11.0*以降と android 4.4 (API レベル 19) がインストールされている必要があります。

[Android SDK マネージャーで Android 4.4 を選択![には](kitkat-images/api19.png)](kitkat-images/api19.png#lightbox)

<a name="Migrating_Your_App_to_KitKat" />

## <a name="migrating-your-app-to-kitkat"></a>KitKat へのアプリの移行

このセクションでは、既存のアプリケーションを Android 4.4 に移行するための最初の応答項目について説明します。

### <a name="check-system-version"></a>システムバージョンの確認

アプリケーションが以前のバージョンの Android と互換性を持っている必要がある場合は、以下のコードサンプルに示すように、KitKat 固有のコードをシステムのバージョンチェックに必ずラップしてください。

```csharp
if (Build.VERSION.SdkInt >= BuildVersionCodes.Kitkat) {
    //KitKat only code here
}
```

### <a name="alarm-batching"></a>アラームのバッチ処理

Android では、アラームサービスを使用して、指定した時間にバックグラウンドでアプリをスリープ解除します。 KitKat は、アラームをバッチ処理して電源を保持することで、これをさらに一歩進めます。 つまり、各アプリを正確なタイミングでスリープ解除する代わりに、KitKat は、同じ期間にウェイクアップするように登録されている複数のアプリケーションをグループ化し、同時にスリープ解除します。
指定された期間中にアプリをウェイクアップするよう Android に指示するには、 [`AlarmManager`](xref:Android.App.AlarmManager)で `SetWindow` を呼び出し、アプリがウェイクアップされるまでの経過時間 (ミリ秒単位) と、ウェイクアップ時に実行する操作を渡します。
次のコードは、ウィンドウが設定された時間から30分間にウェイクアップする必要があるアプリケーションの例を示しています。

```csharp
AlarmManager alarmManager = (AlarmManager)GetSystemService(AlarmService);
alarmManager.SetWindow (AlarmType.Rtc, AlarmManager.IntervalHalfHour, AlarmManager.IntervalHour, pendingIntent);
```

アプリのスリープを正確なタイミングで続行するには、`SetExact`を使用し、アプリをウェイクアップする正確な時間を渡し、次の操作を実行します。

```csharp
alarmManager.SetExact (AlarmType.Rtc, AlarmManager.IntervalDay, pendingIntent);
```

KitKat では、正確な繰り返しアラームを設定できなくなりました。 [`SetRepeating`](xref:Android.App.AlarmManager.SetRepeating*)を使用するアプリケーション
また、正確なアラームを使用するには、各アラームを手動でトリガーする必要があります。

### <a name="external-storage"></a>外部ストレージ

外部ストレージは、アプリケーションに固有のストレージと複数のアプリケーションによって共有されるデータの2種類に分類されるようになりました。 外部ストレージ上のアプリ固有の場所に対する読み取りと書き込みには、特別なアクセス許可は必要ありません。 共有ストレージ上のデータと対話するには、`READ_EXTERNAL_STORAGE` または `WRITE_EXTERNAL_STORAGE` のアクセス許可が必要になりました。 この2種類は、次のように分類できます。

- `Context` でメソッドを呼び出すことによってファイルまたはディレクトリのパスを取得している場合 (例: [`GetExternalFilesDir`](xref:Android.Content.Context.GetExternalFilesDir*)
  または[`GetExternalCacheDirs`](xref:Android.Content.Context.GetExternalCacheDirs)
  - アプリには追加のアクセス許可は必要ありません。

- プロパティにアクセスするか、`Environment` でメソッドを呼び出すことによってファイルまたはディレクトリのパスを取得する場合は ( [`GetExternalStorageDirectory`](xref:Android.OS.Environment.ExternalStorageDirectory)など)。
  または[`GetExternalStoragePublicDirectory`](xref:Android.OS.Environment.GetExternalStoragePublicDirectory*)
  アプリには、`READ_EXTERNAL_STORAGE` または `WRITE_EXTERNAL_STORAGE` のアクセス許可が必要です。

> [!NOTE]
> `WRITE_EXTERNAL_STORAGE` は `READ_EXTERNAL_STORAGE` のアクセス許可を意味するので、1つのアクセス許可のみを設定する必要があります。

### <a name="sms-consolidation"></a>SMS 統合

KitKat は、ユーザーによって選択された1つの既定のアプリケーションのすべての SMS コンテンツを集計することによって、ユーザーのメッセージングを簡略化します。 開発者は、アプリを既定のメッセージングアプリケーションとして選択可能にし、アプリケーションが選択されていない場合にコード内で適切に動作するようにします。 SMS アプリを KitKat に移行する方法の詳細については、Google の「 [Sms アプリを準備する KitKat](https://android-developers.blogspot.com/2013/10/getting-your-sms-apps-ready-for-kitkat.html) 」ガイドを参照してください。

### <a name="webview-apps"></a>WebView アプリ

[WebView](xref:Android.Webkit.WebView)が KitKat で作り直しを持っています。 最大の変更は、コンテンツを `WebView`に読み込むためのセキュリティを強化することです。 古い API バージョンを対象とするほとんどのアプリケーションは想定どおりに動作しますが、`WebView` クラスを使用するアプリケーションをテストすることを強くお勧めします。 影響を受ける WebView Api の詳細については、android 4.4 ドキュメントの「web ビューへの Android の[移行」](https://developer.android.com/guide/webapps/migrating.html)を参照してください。

<a name="user_experience" />

## <a name="user-experience"></a>ユーザー体験

KitKat には、ユーザーエクスペリエンスを向上させるための新しい Api がいくつか付属しています。これには、プロパティアニメーションを処理するための新しい遷移フレームワークや、テーマ用の半透明 UI オプションが含まれます。 これらの変更については、以下で説明します。

### <a name="transition-framework"></a>移行フレームワーク

移行フレームワークを使用すると、アニメーションの実装が容易になります。 KitKat を使用すると、1行のコードだけで簡単なプロパティアニメーションを実行したり、*シーン*を使用して切り替え効果をカスタマイズしたりできます。

#### <a name="simple-property-animation"></a>単純なプロパティアニメーション

新しい Android 遷移ライブラリは、プロパティアニメーションの分離コードを簡略化します。 フレームワークを使用すると、最小限のコードで単純なアニメーションを実行できます。 たとえば、次のコードサンプルではを使用して[`TransitionManager.BeginDelayedTransition`](xref:Android.Transitions.TransitionManager.BeginDelayedTransition*)
`TextView`の表示と非表示をアニメーション化するには:

```csharp
using Android.Transitions;

public class MainActivity : Activity
{
    LinearLayout linear;
    Button button;
    TextView text;

    protected override void OnCreate (Bundle bundle)
    {
        base.OnCreate (bundle);
        SetContentView (Resource.Layout.Main);

        linear = FindViewById<LinearLayout> (Resource.Id.linearLayout);
        button = FindViewById<Button> (Resource.Id.button);
        text = FindViewById<TextView> (Resource.Id.textView);

        button.Click += (o, e) => {

            TransitionManager.BeginDelayedTransition (linear);

            if(text.Visibility != ViewStates.Visible)
            {
                text.Visibility = ViewStates.Visible;
            }
            else
            {
                text.Visibility = ViewStates.Invisible;
            }
        };
    }
}
```

上の例では、遷移フレームワークを使用して、変化するプロパティ値の間に自動的に既定の遷移を作成します。 アニメーションは1行のコードによって処理されるため、`BeginDelayedTransition` の呼び出しをシステムのバージョンチェックでラップすることで、以前のバージョンの Android との互換性を簡単に実現できます。 詳細については、「 [KitKat へのアプリの移行](#Migrating_Your_App_to_KitKat)」を参照してください。

次のスクリーンショットは、アニメーションの前のアプリを示しています。

[アニメーションを開始する前にアプリのスクリーンショットを![](kitkat-images/trans-before.png)](kitkat-images/trans-before.png#lightbox)

次のスクリーンショットは、アニメーションの後のアプリを示しています。

[アニメーションが完了した後の![アプリのスクリーンショット](kitkat-images/trans-after.png)](kitkat-images/trans-after.png#lightbox)

次のセクションで説明するように、バックグラウンドで切り替え効果をより詳細に制御できます。

#### <a name="android-scenes"></a>Android シーン

[バックグラウンド](xref:Android.Transitions.Scene)は、開発者がアニメーションをより細かく制御できるように、移行フレームワークの一部として導入されました。 シーン: UI で動的な領域を作成します。コンテナー、いくつかのバージョン、または "シーン" をコンテナー内の XML コンテンツに対して指定します。 Android は、残りの作業を行って、バックグラウンド間の遷移をアニメーション化します。 Android シーンを使用すると、開発側での作業を最小限に抑えて複雑なアニメーションを作成できます。

動的コンテンツを格納する静的 UI 要素は、*コンテナー*または*シーンベース*と呼ばれるです。 次の例では、Android Designer を使用して、`container`という `RelativeLayout` を作成します。

[Android Designer を使用して RelativeLayout コンテナーを作成する![](kitkat-images/container.png)](kitkat-images/container.png#lightbox)

このサンプルレイアウトでは、`container`の下 `sceneButton` と呼ばれるボタンも定義されています。 このボタンをクリックすると、移行がトリガーされます。

コンテナー内の動的コンテンツには、2つの新しい Android レイアウトが必要です。 これらのレイアウトでは、コンテナー*内*のコードのみが指定されます。
次のコード例では、 *Scene1*という名前のレイアウトを定義しています。これには、"Kit" と "kat" の2つのテキストフィールドがそれぞれ含まれています。また、 *Scene2*と呼ばれる2番目のレイアウトで、同じテキストフィールドが反転されています。 XML は次のとおりです。

 **Scene1**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView
        android:id="@+id/textA"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Kit"
        android:textSize="35sp" />
    <TextView
        android:id="@+id/textB"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_toRightOf="@id/textA"
        android:text="Kat"
        android:textSize="35sp" />
</merge>
```

 **Scene2**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView
        android:id="@+id/textB"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Kat"
        android:textSize="35sp" />
    <TextView
        android:id="@+id/textA"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_toRightOf="@id/textB"
        android:text="Kit"
        android:textSize="35sp" />
</merge>
```

上の例では、`merge` を使用してビューコードを短くし、ビュー階層を簡略化しています。 `merge` レイアウトの詳細については、[こちら](https://android-developers.blogspot.com/2009/03/android-layout-tricks-3-optimize-by.html)を参照してください。

シーンを作成するには、次のコード例に示すように、 [`Scene.GetSceneForLayout`](xref:Android.Transitions.Scene.GetSceneForLayout*)を呼び出して、コンテナーオブジェクト、シーンのレイアウトファイルのリソース ID、現在の `Context`を渡します。

```csharp
RelativeLayout container = FindViewById<RelativeLayout> (Resource.Id.container);

Scene scene1 = Scene.GetSceneForLayout(container, Resource.Layout.Scene1, this);
Scene scene2 = Scene.GetSceneForLayout(container, Resource.Layout.Scene2, this);

scene1.Enter();
```

ボタンをクリックすると、次の2つのシーンが反転されます。 Android では、既定の遷移値がアニメーション化されます。

```csharp
sceneButton.Click += (o, e) => {
    Scene temp = scene2;
    scene2 = scene1;
    scene1 = temp;

    TransitionManager.Go (scene1);
};
```

次のスクリーンショットは、アニメーションの前のシーンを示しています。

[アニメーションを開始する前のアプリのスクリーンショットの![](kitkat-images/trans-after.png)](kitkat-images/trans-after.png#lightbox)

次のスクリーンショットは、アニメーションの後のシーンを示しています。

[アニメーションが完了した後のアプリのスクリーンショットの![](kitkat-images/scene.png)](kitkat-images/scene.png#lightbox)

> [!NOTE]
> Android の遷移ライブラリには[既知のバグ](https://code.google.com/p/android/issues/detail?id=62450)があります。これにより、ユーザーが2回目にアクティビティを移動したときに、`GetSceneForLayout` を使用して作成されたシーンが中断されます。 Java の回避策については、[こちら](http://www.doubleencore.com/2013/11/new-transitions-framework/)を参照してください。

##### <a name="custom-transitions-in-scenes"></a>バックグラウンドでのカスタム遷移

カスタム遷移は、次のスクリーンショットに示すように、`Resources`の下にある `transition` ディレクトリの xml リソースファイルで定義できます。

[![Resources/transition ディレクトリの下にある遷移の .xml ファイルの場所](kitkat-images/resources.png)](kitkat-images/resources.png#lightbox)

次のコードサンプルでは、5秒間アニメーション化し、[超え interpolator](https://developer.android.com/reference/android/views/animation/OvershootInterpolator.html)を使用する遷移を定義しています。

```xml
<changeBounds
  xmlns:android="http://schemas.android.com/apk/res/android"
  android:duration="5000"
  android:interpolator="@android:anim/overshoot_interpolator" />
```

遷移は、次のコードに示すように、 [TransitionInflater](xref:Android.Transitions.TransitionInflater)を使用してアクティビティ内に作成されます。

```csharp
Transition transition = TransitionInflater.From(this).InflateTransition(Resource.Transition.transition);
```

次に、アニメーションを開始する `Go` 呼び出しに新しい遷移が追加されます。

```csharp
TransitionManager.Go (scene1, transition);
```

### <a name="translucent-ui"></a>半透明の UI

KitKat を使用すると、オプションの半透明状態とナビゲーションバーを使用して、アプリのテーマをより細かく制御できます。 Android テーマを定義するために使用するのと同じ XML ファイル内のシステム UI 要素の透明度を変更できます。 KitKat には、次のプロパティが導入されています。

- `windowTranslucentStatus`-true に設定すると、最上位ステータスバーが半透明になります。

- `windowTranslucentNavigation`-true に設定すると、下部のナビゲーションバーが半透明になります。

- `fitsSystemWindows`-既定では、transcluent に上または下のバーを設定して、透明な UI 要素の下にコンテンツをシフトします。 このプロパティを `true` に設定することにより、コンテンツが半透明なシステム UI 要素と重複しないようにすることができます。

次のコードは、半透明のステータスとナビゲーションバーを持つテーマを定義します。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<resources>
    <style name="KitKatTheme" parent="android:Theme.Holo.Light">
        <item name="android:windowBackground">@color/xamgray</item>
        <item name="android:windowTranslucentStatus">true</item>
        <item name="android:windowTranslucentNavigation">true</item>
        <item name="android:fitsSystemWindows">true</item>
        <item name="android:actionBarStyle">@style/ActionBar.Solid.KitKat</item>
    </style>

    <style name="ActionBar.Solid.KitKat" parent="@android:style/Widget.Holo.Light.ActionBar.Solid">
        <item name="android:background">@color/xampurple</item>
    </style>
</resources>
```

次のスクリーンショットは、半透明のステータスとナビゲーションバーのあるテーマを示しています。

[![半透明のステータスとナビゲーションバーを持つアプリのスクリーンショットの例](kitkat-images/theme.png)](kitkat-images/theme.png#lightbox)

<a name="user_content" />

## <a name="user-content"></a>ユーザーのコンテンツ

### <a name="storage-access-framework"></a>ストレージアクセスフレームワーク

ストレージアクセスフレームワーク (SAF) は、画像、ビデオ、ドキュメントなどの保存されたコンテンツをユーザーが操作するための新しい方法です。 KitKat は、コンテンツを処理するアプリケーションを選択するためのダイアログを表示するのではなく、ユーザーが1つの集約された場所でデータにアクセスできるようにする新しい UI を開きます。 コンテンツが選択されると、ユーザーはコンテンツを要求したアプリケーションに戻り、アプリのエクスペリエンスは通常どおり続行されます。

この変更には、開発者側で2つのアクションが必要です。まず、プロバイダーのコンテンツを必要とするアプリを、コンテンツを要求する新しい方法に更新する必要があります。 2つ目の方法として、`ContentProvider` にデータを書き込むアプリケーションを、新しいフレームワークを使用するように変更する必要があります。 どちらのシナリオも、新しい[`DocumentsProvider`](xref:Android.Provider.DocumentsProvider)に依存します。
[API]。

#### <a name="documentsprovider"></a>ドキュメントプロバイダー

KitKat では、`ContentProviders` との相互作用は、`DocumentsProvider` クラスで抽象化されています。 つまり、SAF は、`DocumentsProvider` API を使用してアクセスできる限り、データの物理的な場所を気にすることはできません。 ローカルプロバイダー、クラウドサービス、および外部ストレージデバイスはすべて同じインターフェイスを使用し、同じ方法で処理されます。これにより、ユーザーと開発者は、ユーザーのコンテンツを1か所で操作できるようになります。

このセクションでは、ストレージアクセスフレームワークを使用してコンテンツを読み込んで保存する方法について説明します。

#### <a name="request-content-from-a-provider"></a>プロバイダーからのコンテンツの要求

`ActionOpenDocument` インテントと共に SAF UI を使用してコンテンツを選択するように指示できます。これは、デバイスで利用可能なすべてのコンテンツプロバイダーに接続することを意味します。 `CategoryOpenable`を指定することによって、この目的にフィルターを追加できます。これは、開くことができるコンテンツ (つまり、アクセス可能なコンテンツ) のみを返すことを意味します。 KitKat では、`MimeType`でコンテンツをフィルター処理することもできます。 たとえば、次のコードでは、イメージ `MimeType`を指定することによって、イメージの結果をフィルター処理します。

```csharp
Intent intent = new Intent (Intent.ActionOpenDocument);
intent.AddCategory (Intent.CategoryOpenable);
intent.SetType ("image/*");
StartActivityForResult (intent, save_request_code);
```

`StartActivityForResult` を呼び出すと、SAF UI が起動されます。ユーザーはこれを参照してイメージを選択できます。

[![イメージを参照するためにストレージアクセスフレームワークを使用するアプリのスクリーンショットの例](kitkat-images/saf-ui.png)](kitkat-images/saf-ui.png#lightbox)

ユーザーがイメージを選択すると、`OnActivityResult` は選択したファイルの `Android.Net.Uri` を返します。 次のコード例は、ユーザーのイメージ選択を示しています。

```csharp
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);

    if (resultCode == Result.Ok && data != null && requestCode == save_request_code) {
        imageView = FindViewById<ImageView> (Resource.Id.imageView);
        imageView.SetImageURI (data.Data);
    }
}
```

#### <a name="write-content-to-a-provider"></a>プロバイダーへのコンテンツの書き込み

KitKat では、SAF UI からコンテンツを読み込むだけでなく、`DocumentProvider` API を実装する任意の `ContentProvider` にコンテンツを保存することもできます。 コンテンツの保存では、`ActionCreateDocument`で `Intent` を使用します。

```csharp
Intent intentCreate = new Intent (Intent.ActionCreateDocument);
intentCreate.AddCategory (Intent.CategoryOpenable);
intentCreate.SetType ("text/plain");
intentCreate.PutExtra (Intent.ExtraTitle, "NewDoc");
StartActivityForResult (intentCreate, write_request_code);
```

上記のコードサンプルでは、SAF UI が読み込まれ、ユーザーはファイル名を変更し、新しいファイルを格納するディレクトリを選択できます。

[ユーザーのスクリーンショットを![ダウンロードディレクトリのファイル名を NewDoc に変更する](kitkat-images/saf-save.png)](kitkat-images/saf-save.png#lightbox)

ユーザーが **[保存]** を押すと、新しく作成されたファイルの `Android.Net.Uri` `OnActivityResult` 渡されます。このファイルには `data.Data`を使用してアクセスできます。 Uri は、新しいファイルにデータをストリーミングするために使用できます。

```csharp
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);

    if (resultCode == Result.Ok && data != null && requestCode == write_request_code) {
        using (Stream stream = ContentResolver.OpenOutputStream(data.Data)) {
            Encoding u8 = Encoding.UTF8;
            string content = "Hello, world!";
            stream.Write (u8.GetBytes(content), 0, content.Length);
        }
    }
}
```

[`ContentResolver.OpenOutputStream(Android.Net.Uri)`](xref:Android.Content.ContentResolver.OpenOutputStream*)
`System.IO.Stream`を返します。これにより、ストリーミングプロセス全体を .NET で記述できるようになります。

ストレージアクセスフレームワークを使用したコンテンツの読み込み、作成、および編集の詳細については、[ストレージアクセスフレームワークの Android ドキュメント](https://developer.android.com/guide/topics/providers/document-provider.html)を参照してください。

### <a name="printing"></a>印刷

[印刷サービス](xref:Android.PrintServices)と `PrintManager`の導入により、KitKat でのコンテンツの印刷が簡単になりました。 また、KitKat は、google [Cloud print アプリケーション](https://play.google.com/store/apps/details?id=com.google.android.apps.cloudprint)を使用して[Google の Cloud print service api](https://developers.google.com/cloud-print/)を完全に活用するための最初の API バージョンでもあります。
KitKat に付属しているほとんどのデバイスでは、最初に WiFi に接続するときに Google Cloud Print アプリと[HP Print Service プラグイン](https://play.google.com/store/apps/details?id=com.hp.android.printservice)が自動的にダウンロードされます。 ユーザーは、設定 **システム > 印刷** の順に移動して、自分のデバイスの印刷設定を確認でき > ます。

[![印刷の設定画面のスクリーンショットの例](kitkat-images/printing.png)](kitkat-images/printing.png#lightbox)

> [!NOTE]
> 既定では、印刷 Api は Google Cloud Print と連動するように設定されていますが、開発者は、新しい Api を使用して印刷コンテンツを準備し、それを他のアプリケーションに送信して印刷処理を行うことができます。

#### <a name="printing-html-content"></a>HTML コンテンツの印刷

KitKat を使用すると、web ビューの[`PrintDocumentAdapter`](xref:Android.Print.PrintDocumentAdapter)が `WebView.CreatePrintDocumentAdapter`で自動的に作成されます。 Web コンテンツの印刷は、HTML コンテンツが読み込まれるのを待機し、[オプション] メニューで [印刷] オプションを使用できるようにすることをアクティビティに知らせる[`WebViewClient`](xref:Android.Webkit.WebViewClient)と、ユーザーが [印刷] オプションと c を選択するのを待機するアクティビティの間の連携した作業です。`PrintManager`に `Print`を alls します。 このセクションでは、画面に表示される HTML コンテンツを印刷するために必要な基本的なセットアップについて説明します。

Web コンテンツの読み込みと印刷には、インターネットのアクセス許可が必要です。

[アプリオプションでインターネットアクセス許可を設定![には](kitkat-images/internet.png)](kitkat-images/internet.png#lightbox)

##### <a name="print-menu-item"></a>[印刷] メニュー項目

印刷オプションは、通常、活動の [[オプション] メニュー](https://developer.android.com/guide/topics/ui/menus.html#options-menu)に表示されます。
[オプション] メニューを使用すると、ユーザーはアクティビティに対してアクションを実行できます。 画面の右上隅にあり、次のように表示されます。

[![画面の右上隅に表示される [印刷] メニュー項目のスクリーンショット](kitkat-images/menu.png)](kitkat-images/menu.png#lightbox)

追加のメニュー項目は、[*リソース*] の下の*メニュー*ディレクトリで定義できます。 次のコードでは、 [Print](xref:Android.Print.PrintManager)という名前のサンプルメニュー項目を定義しています。

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/menu_print"
        android:title="Print"
        android:showAsAction="never" />
</menu>
```

アクティビティの [オプション] メニューとの対話は、`OnCreateOptionsMenu` および `OnOptionsItemSelected` メソッドを使用して行われます。
`OnCreateOptionsMenu` は、*メニュー*リソースディレクトリから、[印刷] オプションなどの新しいメニュー項目を追加する場所です。
`OnOptionsItemSelected` は、ユーザーがメニューから [印刷] オプションを選択するのを待ち、印刷を開始します。

```csharp
bool dataLoaded;

public override bool OnCreateOptionsMenu (IMenu menu)
{
    base.OnCreateOptionsMenu (menu);
    if (dataLoaded) {
        MenuInflater.Inflate (Resource.Menu.print, menu);
    }
    return true;
}

public override bool OnOptionsItemSelected (IMenuItem item)
{
    if (item.ItemId == Resource.Id.menu_print) {
        PrintPage ();
        return true;
    }
    return base.OnOptionsItemSelected (item);
}
```

また、上記のコードでは、HTML コンテンツのステータスを追跡するために `dataLoaded` と呼ばれる変数も定義されています。 `WebViewClient` は、すべてのコンテンツが読み込まれたときに、この変数を true に設定します。そのため、[オプション] メニューに [印刷] メニュー項目を追加することをアクティビティが認識します。

##### <a name="webviewclient"></a>WebViewClient

`WebViewClient` のジョブは、[印刷] オプションが `OnPageFinished` メソッドを使用して実行される前に、`WebView` のデータが完全に読み込まれるようにすることです。 `OnPageFinished` は、web コンテンツの読み込みが完了するのを待機し、`InvalidateOptionsMenu`を使用して [オプション] メニューを再作成するようにアクティビティに指示します。

```csharp
class MyWebViewClient : WebViewClient
{
    PrintHtmlActivity caller;

    public MyWebViewClient (PrintHtmlActivity caller)
    {
        this.caller = caller;
    }

    public override void OnPageFinished (WebView view, string url)
    {
        caller.dataLoaded = true;
        caller.InvalidateOptionsMenu ();
    }
}
```

また `OnPageFinished` `dataLoaded` の値を `true`に設定します。これにより、`OnCreateOptionsMenu` は [印刷] オプションを使用してメニューを再作成できます。

##### <a name="printmanager"></a>PrintManager

次のコード例では、`WebView`の内容を出力します。

```csharp
void PrintPage ()
{
    PrintManager printManager = (PrintManager)GetSystemService (Context.PrintService);
    PrintDocumentAdapter printDocumentAdapter = myWebView.CreatePrintDocumentAdapter ();
    printManager.Print ("MyWebPage", printDocumentAdapter, null);
}
```

`Print` は引数としてを受け取ります。印刷ジョブの名前 (この例では "MyWebPage ページ")、a [`PrintDocumentAdapter`](xref:Android.Print.PrintDocumentAdapter)
これにより、コンテンツから印刷ドキュメントが生成され、 [`PrintAttributes`](xref:Android.Print.PrintAttributes)
(上記の例では `null`)。 既定の属性はほとんどのシナリオを処理する必要がありますが、`PrintAttributes` を指定して、印刷されたページでコンテンツをレイアウトすることができます。

`Print` を呼び出すと、印刷ジョブのオプションが一覧表示された印刷 UI が読み込まれます。 UI では、次のスクリーンショットに示すように、HTML コンテンツを PDF に印刷または保存するオプションをユーザーに提供します。

[[印刷] メニューを表示する PrintHtmlActivity のスクリーンショット![](kitkat-images/print1.png)](kitkat-images/print1.png#lightbox)

[[PDF として保存] メニューを表示する PrintHtmlActivity のスクリーンショットを![](kitkat-images/print2.png)](kitkat-images/print2.png#lightbox)

<a name="hardware" />

## <a name="hardware"></a>ハードウェア

KitKat は、新しいデバイスの機能に対応するために、いくつかの Api を追加します。 最も注目すべき点は、ホストベースのカードエミュレーションと新しい `SensorManager`です。

### <a name="host-based-card-emulation-in-nfc"></a>NFC のホストベースのカードエミュレーション

ホストベースのカードエミュレーション (HCE) を使用すると、キャリアの専用安全要素に依存せずに、NFC カードや NFC カードリーダーのようにアプリケーションを動作させることができます。 HCE を設定する前に、`PackageManager.HasSystemFeature`を使用してデバイスで HCE を使用できることを確認します。

```csharp
bool hceSupport = PackageManager.HasSystemFeature(PackageManager.FeatureNfcHostCardEmulation);
```

HCE では、HCE 機能と `Nfc` アクセス許可の両方をアプリケーションの `AndroidManifest.xml`に登録する必要があります。

```xml
<uses-feature android:name="android.hardware.nfc.hce" />
```

[アプリオプションで NFC アクセス許可を設定![には](kitkat-images/nfc.png)](kitkat-images/nfc.png#lightbox)

作業するには、HCE をバックグラウンドで実行する必要があります。また、HCE を使用しているアプリケーションが実行されていない場合でも、ユーザーが NFC トランザクションを作成したときに開始する必要があります。 HCE コードを `Service`として記述することで、これを実現できます。 HCE サービスは、次のメソッドを実装する `HostApduService` インターフェイスを実装します。

- *Processcommandapdu* -アプリケーションプロトコルデータユニット (APDU) は、NFC リーダーと Hce サービスの間で送信されます。 このメソッドは、リーダーから ADPU を使用し、応答でデータ単位を返します。

- *Ondeactivated アクティブ*化-Hce サービスが NFC リーダーと通信しなくなったときに、`HostAdpuService` が非アクティブになります。

HCE サービスもアプリケーションのマニフェストに登録する必要があり、適切なアクセス許可、インテントフィルター、およびメタデータで修飾する必要があります。 次のコードは、`Service` 属性を使用して Android マニフェストに登録されている `HostApduService` の例を示しています (属性の詳細については、「Xamarin [Working With Android manifest guide (Android マニフェスト](~/android/platform/android-manifest.md)の概要)」を参照してください)。

```csharp
[Service(Exported=true, Permission="android.permissions.BIND_NFC_SERVICE"),
    IntentFilter(new[] {"android.nfc.cardemulation.HOST_APDU_SERVICE"}),
    MetaData("android.nfc.cardemulation.host.apdu_service",
    Resource="@xml/hceservice")]

class HceService : HostApduService
{
    public override byte[] ProcessCommandApdu(byte[] apdu, Bundle extras)
    {
        ...
    }

    public override void OnDeactivated (DeactivationReason reason)
    {
        ...
    }
}
```

上記のサービスでは、NFC リーダーがアプリケーションと対話する方法を提供していますが、NFC リーダーは、このサービスがスキャンする必要がある NFC カードをエミュレートしているかどうかを知ることはできません。 NFC リーダーがサービスを識別できるように、サービスに一意の*アプリケーション ID (支援)* を割り当てることができます。 `MetaData` 属性に登録されている xml リソースファイル (上記のコード例を参照) で、HCE サービスに関するその他のメタデータと共に補助を指定します。 このリソースファイルは、1つまたは複数の NFC リーダーデバイスの補助機能に対応する、1つ以上の補助フィルター (一意の識別子文字列) を16進数形式で指定します。

```xml
<host-apdu-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:description="@string/hce_service_description"
    android:requireDeviceUnlock="false"
    android:apduServiceBanner="@drawable/service_banner">
    <aid-group android:description="@string/aid_group_description"
                android:category="payment">
        <aid-filter android:name="1111111111111111"/>
        <aid-filter android:name="0123456789012345"/>
    </aid-group>
</host-apdu-service>
```

Xml リソースファイルは、フィルターを支援するだけでなく、HCE サービスのユーザー向けの説明も提供します。また、補助グループ (支払いアプリケーションと "その他") を指定し、支払いアプリケーションの場合は、ユーザーに表示する 260x96 dp バナーを指定します。

前に説明したセットアップは、NFC カードをエミュレートするアプリケーションの基本的な構成要素を提供します。 NFC 自体では、さらに多くの手順を実行し、さらにテストを構成する必要があります。 ホストベースのカードエミュレーションの詳細については、 [Android ドキュメントポータル](https://developer.android.com/guide/topics/connectivity/nfc/hce.html)を参照してください。
Xamarin での NFC の使用の詳細については、 [XAMARIN NFC のサンプル](https://github.com/xamarin/monodroid-samples/tree/master/NfcSample)を参照してください。

### <a name="sensors"></a>センサー

KitKat を使用すると、デバイスのセンサーに[`SensorManager`](xref:Android.Hardware.SensorManager)経由でアクセスできます。
`SensorManager` を使用すると、OS はアプリケーションへのセンサー情報の配信をバッチでスケジュールし、バッテリの寿命を維持することができます。

また、KitKat には、ユーザーの手順を追跡する2つの新しいセンサーの種類も付属しています。 これらは加速度計に基づいており、次のものが含まれます。

- *Stepdetector* -ユーザーがステップを実行したときにアプリに通知され、ウェイクアップされます。この検出機能によって、ステップが発生した時間の値が示されます。

- *Stepcounter* -センサーが*次回のデバイスの再起動まで*に登録された後にユーザーが行った手順の数を追跡します。

次のスクリーンショットは、動作中のステップカウンターを示しています。

[SensorsActivity アプリのスクリーンショットを![ステップカウンターを表示する](kitkat-images/stepcounter.png)](kitkat-images/stepcounter.png#lightbox)

`GetSystemService(SensorService)` を呼び出し、結果を `SensorManager`としてキャストすることで、`SensorManager` を作成できます。 ステップカウンターを使用するには、`SensorManager`で `GetDefaultSensor` を呼び出します。 センサーを登録し、ステップ数の変更をリッスンするには、 [`ISensorEventListener`](xref:Android.Hardware.ISensorEventListener)を参照してください。
インターフェイス。以下のコードサンプルを参照してください。

```csharp
public class MainActivity : Activity, ISensorEventListener
{
    float count = 0;

    protected override void OnCreate (Bundle bundle)
    {
        base.OnCreate (bundle);
        SetContentView (Resource.Layout.Main);

        SensorManager senMgr = (SensorManager) GetSystemService (SensorService);
        Sensor counter = senMgr.GetDefaultSensor (SensorType.StepCounter);
        if (counter != null) {
            senMgr.RegisterListener(this, counter, SensorDelay.Normal);
        }
    }

    public void OnAccuracyChanged (Sensor sensor, SensorStatus accuracy)
    {
        Log.Info ("SensorManager", "Sensor accuracy changed");
    }

    public void OnSensorChanged (SensorEvent e)
    {
        count = e.Values [0];
    }
}
```

`OnSensorChanged` は、アプリケーションがフォアグラウンドにある間にステップ数が更新された場合に呼び出されます。 アプリケーションがバックグラウンドで入力された場合、またはデバイスがスリープ状態の場合、`OnSensorChanged` は呼び出されません。ただし、`UnregisterListener` が呼び出されるまで、手順は引き続きカウントされます。

*ステップ数の値は、センサーを登録するすべてのアプリケーションで累積さ*れることに注意してください。 つまり、アプリケーションをアンインストールして再インストールし、アプリケーションの起動時に0で `count` 変数を初期化したとしても、センサーによって報告された値は、センサーが登録されている間に実行されたステップの合計数を保持します。アプリケーションまたはその他。 次のコードに示すように、`SensorManager`で `UnregisterListener` を呼び出すことによって、アプリケーションがステップカウンターに追加されないようにすることができます。

```csharp
protected override void OnPause()
{
    base.OnPause ();
    senMgr.UnregisterListener(this);
}
```

デバイスを再起動すると、ステップ数が0にリセットされます。 アプリは、センサーを使用する他のアプリケーションやデバイスの状態に関係なく、アプリケーションの正確な数を報告するために、追加のコードを必要とします。

> [!NOTE]
> ステップの検出とカウントの API には KitKat が付属していますが、すべての電話がセンサーに outfitted されるわけではありません。 `PackageManager.HasSystemFeature(PackageManager.FeatureSensorStepCounter);`を実行してセンサーが使用可能かどうかを確認するか、`GetDefaultSensor` の戻り値が `null`でないことを確認します。

<a name="developer_tools" />

## <a name="developer-tools"></a>開発者用ツール

### <a name="screen-recording"></a>画面記録

KitKat には、開発者が動作中のアプリケーションを記録できるように、新しい画面記録機能が用意されています。 画面記録は、 [Android Debug Bridge (ADB)](https://developer.android.com/tools/help/adb.html)クライアントを通じて利用できます。このクライアントは Android SDK の一部としてダウンロードできます。

画面を録画するには、デバイスを接続します。次に、Android SDK のインストールを見つけ、**プラットフォームツール**のディレクトリに移動して、 **adb**クライアントを実行します。

```shell
adb shell screenrecord /sdcard/screencast.mp4
```

上のコマンドでは、既定の3分間のビデオが 4 Mbps の既定の解像度で記録されます。 長さを編集するには、 *--time-limit*フラグを追加します。
解像度を変更するには、 *--bit レート*フラグを追加します。 次のコマンドは、1分間のビデオを8Mbps で記録します。

```shell
adb shell screenrecord --bit-rate 8000000 --time-limit 60 /sdcard/screencast.mp4
```

ビデオはデバイスにあります。録画が完了すると、ギャラリーに表示されます。

## <a name="other-kitkat-additions"></a>その他の KitKat の追加

KitKat では、上記の変更に加えて次のことができます。

- *全画面表示を使用する*-KitKat では、コンテンツを参照したり、ゲームを楽しんだ後に、全画面表示のエクスペリエンスの恩恵を受ける他のアプリケーションを実行したりするための新しい[イマーシブモード](https://developer.android.com/reference/android/view/View.html#setSystemUiVisibility(int))が導入されています。

- *通知をカスタマイズ*する- [`NotificationListenerService`](xref:Android.Service.Notification.NotificationListenerService)を使用してシステム通知に関する追加情報を取得する
  である必要があります。 これにより、アプリ内で別の方法で情報を表示できます。

- 作成される*リソースのミラー*化されるリソース-新しい[`autoMirrored`](https://developer.android.com/reference/android/R.attr.html#autoMirrored)
  左から右へのレイアウトの反転が必要なイメージのミラー化されたバージョンをシステムに通知する属性。

- *アニメーションの一時停止*- [`Animator`](xref:Android.Animation.Animator)で作成されたアニメーションを一時停止および再開します
  クラスの新しいインスタンスを初期化します。

- 新しいテキストを使用して動的に更新する UI の*テキストを動的に変更*する部分を、新しい[`accessibilityLiveRegion`](https://developer.android.com/reference/android/R.attr.html#accessibilityLiveRegion)で "ライブ領域" として読み取ります。
  属性を使用して、新しいテキストがユーザー補助モードで自動的に読み込まれるようにします。

- *オーディオエクスペリエンスの向上*- [`LoudnessEnhancer`](xref:Android.Media.Audiofx.LoudnessEnhancer)によってトラックが大きくなります
  では、 [`Visualizer`](xref:Android.Media.Audiofx.Visualizer.MeasurementModePeakRms)を使用してオーディオストリームのピークと RMS を検索します。
  オーディオビデオの同期に役立つように、[オーディオタイムスタンプ](xref:Android.Media.AudioTimestamp)から情報を取得します。

- *カスタム間隔での ContentResolver の同期*-KitKat は、同期要求の実行時にいくつかの可変性を追加します。 `ContentResolver.RequestSync` を呼び出して `SyncRequest`を渡すことによって、`ContentResolver` をカスタムの時刻または間隔で同期します。

- *コントローラーを区別*する-KitKat では、コントローラーには、デバイスの `ControllerNumber` プロパティを通じてアクセスできる一意の整数識別子が割り当てられます。 これにより、ゲームのプレーヤーを区別しやすくなります。

- *リモートコントロール*-ハードウェアとソフトウェア側の両方でいくつかの変更を加えることで、KitKat を使用して、IR トランスミッタを持つデバイス outfitted を `ConsumerIrService`を使用してリモートコントロールに変換し、新しい[`RemoteController`](xref:Android.Media.RemoteController)で周辺機器と対話することができます。
  Api.

上記の API 変更の詳細については、「Google [Android 4.4 api](https://developer.android.com/about/versions/android-4.4.html)の概要」を参照してください。

## <a name="summary"></a>まとめ

この記事では、Android 4.4 (API レベル 19) で利用できる新しい Api のいくつかを紹介し、アプリケーションを KitKat に移行する際のベストプラクティスについて説明しました。 *移行フレームワーク*や*テーマ*の新しいオプションなど、ユーザーエクスペリエンスに影響する api の変更について説明しています。 次に、*ストレージアクセスフレームワーク*と `DocumentsProvider` クラス、および新しい*印刷 api*が導入されました。 ここでは、 *NFC ホストベースのカードエミュレーション*と、ユーザーの手順を追跡する2つの新しいセンサーを含む*低電力センサー*の使用方法について説明しています。 最後に、*画面記録*を使用してアプリケーションのリアルタイムのデモをキャプチャし、KitKat API の変更と追加の詳細な一覧を示しました。

## <a name="related-links"></a>関連リンク

- [KitKat サンプル](https://docs.microsoft.com/samples/xamarin/monodroid-samples/kitkat)
- [Android 4.4 Api](https://developer.android.com/about/versions/android-4.4.html)
- [Android KitKat](https://developer.android.com/about/versions/kitkat.html)
