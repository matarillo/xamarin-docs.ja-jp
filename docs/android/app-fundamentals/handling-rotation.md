---
title: 回転の処理
description: このトピックでは、Xamarin Android でのデバイスの向きの変更を処理する方法について説明します。 Android リソースシステムを使用して、特定のデバイスの向きに関するリソースを自動的に読み込み、方向の変更をプログラムで処理する方法についても説明します。
ms.prod: xamarin
ms.assetid: 6D33ADF7-ED81-0256-479D-D9E3787A76B0
ms.technology: xamarin-android
author: davidortinau
ms.author: daortin
ms.date: 02/16/2018
ms.openlocfilehash: 3277dd5eb7600500a5f60b2bbb13621aa237a235
ms.sourcegitcommit: 2fbe4932a319af4ebc829f65eb1fb1816ba305d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73019268"
---
# <a name="handling-rotation"></a>回転の処理

_このトピックでは、Xamarin Android でのデバイスの向きの変更を処理する方法について説明します。Android リソースシステムを使用して、特定のデバイスの向きに関するリソースを自動的に読み込み、方向の変更をプログラムで処理する方法についても説明します。_

## <a name="overview"></a>概要

モバイルデバイスは簡単にローテーションできるため、組み込みのローテーションは mobile Os の標準機能です。 Android は、アプリケーション内でのローテーションを扱うための高度なフレームワークを提供します。ユーザーインターフェイスは、宣言によって XML で作成されるか、コードでプログラムによって作成されます。 ローテーションされたデバイスでの宣言型のレイアウトの変更を自動的に処理する場合、アプリケーションは Android リソースシステムとの緊密な統合の恩恵を受けることができます。 プログラムによるレイアウトでは、変更を手動で処理する必要があります。 これにより、実行時により細かい制御が可能になりますが、開発者にとってはより多くの作業を行うことができます。 アプリケーションでは、アクティビティの再起動を無効にしたり、向きの変更を手動で制御したりすることもできます。

このガイドでは、次の方向のトピックについて説明します。

- **宣言型のレイアウトのローテーション**&ndash;、Android リソースシステムを使用して、特定の向きに合わせてレイアウトと引き出しの両方を読み込む方法など、方向を認識するアプリケーションを構築する方法を説明します。

- プログラムに**よるレイアウトのローテーション**&ndash;、プログラムによってコントロールを追加する方法と、向きの変更を手動で処理する方法を説明します。

## <a name="handling-rotation-declaratively-with-layouts"></a>レイアウトを使用した、宣言によるローテーションの処理

名前付け規則に従ったフォルダーにファイルを含めることにより、Android は、向きが変更されたときに適切なファイルを自動的に読み込みます。
これには、次のサポートが含まれます。

- 各向きに対してどのレイアウトファイルを拡大するかを指定する*レイアウトリソース*&ndash; ます。

- 各向きに対して読み込まれる drawables 指定するための、描画*リソース*&ndash; ます。

### <a name="layout-resources"></a>リソースのレイアウト

既定では、 **Resources/layout**フォルダーに含まれる Android XML (axml) ファイルは、アクティビティのビューを表示するために使用されます。 このフォルダーのリソースは、横向きに特化したレイアウトリソースが提供されていない場合に、縦向きと横方向の両方で使用されます。 既定のプロジェクトテンプレートによって作成されたプロジェクトの構造を検討します。

[![既定のプロジェクトテンプレートの構造](handling-rotation-images/00.png)](handling-rotation-images/00.png#lightbox)

このプロジェクトでは、 **Resources/layout**フォルダー内に1つの**メインの axml**ファイルが作成されます。 アクティビティの `OnCreate` メソッドが呼び出されると、**増えで定義**されているビューが、次の xml に示すようにボタンを宣言します。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:orientation="vertical"
  android:layout_width="fill_parent"
  android:layout_height="fill_parent">
<Button  
  android:id="@+id/myButton"
  android:layout_width="fill_parent" 
  android:layout_height="wrap_content" 
  android:text="@string/hello"/>
</LinearLayout>
```

デバイスが横向きに回転している場合は、次のスクリーンショットに示すように、アクティビティの `OnCreate` メソッドが再度呼び出され、同じ**メインの axml**ファイルが大きくなります。

[同じ画面を![横方向](handling-rotation-images/01-sml.png)](handling-rotation-images/01.png#lightbox)

#### <a name="orientation-specific-layouts"></a>向きに固有のレイアウト

レイアウトフォルダー (既定では縦) に加えて、`layout-land`という名前のフォルダーを含めることによって明示的に*レイアウトポート*を指定することもできます。アプリケーションでは、コードを変更することなく、横に必要なビューを定義できます。

たとえば、**メインの axml**ファイルに次の xml が含まれているとします。

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="fill_parent"
  android:layout_height="fill_parent">
  <TextView
    android:text="This is portrait"
    android:layout_height="wrap_content"
    android:layout_width="fill_parent" />
</RelativeLayout>
```

追加の**メインの**拡大ファイルを含む layout という名前のフォルダーがプロジェクトに追加された場合、横長のときにレイアウトが**新しく追加さ**れたを読み込むようになります。 次のコードが含まれている、**メインの .xml**ファイルのランドスケープバージョンを考えてみます (わかりやすくするために、この XML は、コードの既定の縦長バージョンに似ていますが、`TextView`では異なる文字列を使用します)。

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="fill_parent"
  android:layout_height="fill_parent">
  <TextView
    android:text="This is landscape"
    android:layout_height="wrap_content"
    android:layout_width="fill_parent" />
</RelativeLayout>
```

次に示すように、このコードを実行して、デバイスを縦から横に回転すると、新しい XML の読み込みが示されます。

[縦と横のスクリーンショット![縦モードで印刷する](handling-rotation-images/02.png)](handling-rotation-images/02.png#lightbox)

### <a name="drawable-resources"></a>描画リソース

ローテーション中、Android は、描画されたリソースをレイアウトリソースと同様に扱います。 この場合、システムは、**リソース/** 描画と**リソース/** 描画フォルダーから、それぞれのシステムによって発生した drawables 取得できます。

たとえば、次のように XML の `ImageView` から参照先が参照されている**Resources/** 構成ファイルフォルダーに、プロジェクトには、"サル .png .png" という名前のイメージが含まれているとします。

```xml
<ImageView
  android:layout_height="wrap_content"
  android:layout_width="wrap_content"
  android:src="@drawable/monkey"
  android:layout_centerVertical="true"
  android:layout_centerHorizontal="true" />
```

さらに、 **[リソース]** の下に、別のバージョンの **.png**が含まれているとします。 レイアウトファイルと同様に、デバイスを回転させると、次に示すように、指定した向きの描画が変更されます。

[![縦モードと横モードで表示されているさまざまなバージョンのサル .png](handling-rotation-images/03.png)](handling-rotation-images/03.png#lightbox)

## <a name="handling-rotation-programmatically"></a>プログラムによるローテーションの処理

コードでレイアウトを定義することがあります。 これは、技術的な制限、開発者の設定など、さまざまな理由で発生する可能性があります。プログラムを使用してコントロールを追加する場合、アプリケーションはデバイスの向きを手動で考慮する必要があります。これは、XML リソースを使用するときに自動的に処理されます。

### <a name="adding-controls-in-code"></a>追加 (コードにコントロールを)

プログラムによってコントロールを追加するには、アプリケーションで次の手順を実行する必要があります。

- レイアウトを作成します。
- レイアウトパラメーターを設定します。
- コントロールを作成します。
- コントロールレイアウトパラメーターを設定します。
- レイアウトにコントロールを追加します。
- レイアウトをコンテンツビューとして設定します。

たとえば、次のコードに示すように、`RelativeLayout`に追加された単一の `TextView` コントロールで構成されるユーザーインターフェイスを考えてみます。

```csharp
protected override void OnCreate (Bundle bundle)
{
  base.OnCreate (bundle);
                        
  // create a layout
  var rl = new RelativeLayout (this);

  // set layout parameters
  var layoutParams = new RelativeLayout.LayoutParams (ViewGroup.LayoutParams.FillParent, ViewGroup.LayoutParams.FillParent);
  rl.LayoutParameters = layoutParams;
        
  // create TextView control
  var tv = new TextView (this);

  // set TextView's LayoutParameters
  tv.LayoutParameters = layoutParams;
  tv.Text = "Programmatic layout";

  // add TextView to the layout
  rl.AddView (tv);
        
  // set the layout as the content view
  SetContentView (rl);
}
```

このコードは `RelativeLayout` クラスのインスタンスを作成し、その `LayoutParameters` プロパティを設定します。 `LayoutParams` クラスは、再利用可能な方法でコントロールを配置する方法を Android の方法でカプセル化します。 レイアウトのインスタンスが作成されると、コントロールを作成して追加できます。 コントロールには、この例の `TextView` などの `LayoutParameters`もあります。 `TextView` が作成された後 `RelativeLayout` に追加し、`RelativeLayout` をコンテンツビューとして設定すると、次のように、アプリケーションによって `TextView` が表示されます。

[縦モードと横モードの両方で表示される![インクリメントカウンターボタン](handling-rotation-images/04.png)](handling-rotation-images/04.png#lightbox)

### <a name="detecting-orientation-in-code"></a>検出 (コード内の方向を)

`OnCreate` が呼び出されたときに、アプリケーションが各向きに対して異なるユーザーインターフェイスを読み込もうとした場合 (これはデバイスがローテーションされるたびに発生します)、向きを検出し、目的のユーザーインターフェイスコードを読み込む必要があります。 Android には `WindowManager`というクラスがあり、次に示すように、`WindowManager.DefaultDisplay.Rotation` プロパティを使用して現在のデバイスのローテーションを決定するために使用できます。

```csharp
protected override void OnCreate (Bundle bundle)
{
  base.OnCreate (bundle);
                        
  // create a layout
  var rl = new RelativeLayout (this);

  // set layout parameters
  var layoutParams = new RelativeLayout.LayoutParams (ViewGroup.LayoutParams.FillParent, ViewGroup.LayoutParams.FillParent);
  rl.LayoutParameters = layoutParams;
                        
  // get the initial orientation
  var surfaceOrientation = WindowManager.DefaultDisplay.Rotation;
  // create layout based upon orientation
  RelativeLayout.LayoutParams tvLayoutParams;
                
  if (surfaceOrientation == SurfaceOrientation.Rotation0 || surfaceOrientation == SurfaceOrientation.Rotation180) {
    tvLayoutParams = new RelativeLayout.LayoutParams (ViewGroup.LayoutParams.FillParent, ViewGroup.LayoutParams.WrapContent);
  } else {
    tvLayoutParams = new RelativeLayout.LayoutParams (ViewGroup.LayoutParams.FillParent, ViewGroup.LayoutParams.WrapContent);
    tvLayoutParams.LeftMargin = 100;
    tvLayoutParams.TopMargin = 100;
  }
                        
  // create TextView control
  var tv = new TextView (this);
  tv.LayoutParameters = tvLayoutParams;
  tv.Text = "Programmatic layout";
        
  // add TextView to the layout
  rl.AddView (tv);
        
  // set the layout as the content view
  SetContentView (rl);
}
```

次に示すように、このコードでは、画面の左上から100ピクセルの位置に、新しいレイアウトに自動的にアニメーション化されるように `TextView` を設定します。

[![ビューステートは、縦モードと横モードで保持されます。](handling-rotation-images/05.png)](handling-rotation-images/05.png#lightbox)

### <a name="preventing-activity-restart"></a>アクティビティの再開の防止

アプリケーションでは、`OnCreate`のすべてを処理するだけでなく、次のように `ActivityAttribute` で `ConfigurationChanges` を設定することによって、向きが変化したときにアクティビティが再起動されないようにすることもできます。

```csharp
[Activity (Label = "CodeLayoutActivity", ConfigurationChanges=Android.Content.PM.ConfigChanges.Orientation | Android.Content.PM.ConfigChanges.ScreenSize)]
```

デバイスがローテーションされると、アクティビティは再起動されません。 この場合、方向の変更を手動で処理するために、アクティビティは、次のアクティビティの新しい実装のように、`OnConfigurationChanged` メソッドをオーバーライドし、渡された `Configuration` オブジェクトから向きを決定できます。

```csharp
[Activity (Label = "CodeLayoutActivity", ConfigurationChanges=Android.Content.PM.ConfigChanges.Orientation | Android.Content.PM.ConfigChanges.ScreenSize)]
public class CodeLayoutActivity : Activity
{
  TextView _tv;
  RelativeLayout.LayoutParams _layoutParamsPortrait;
  RelativeLayout.LayoutParams _layoutParamsLandscape;
                
  protected override void OnCreate (Bundle bundle)
  {
    // create a layout
    // set layout parameters
    // get the initial orientation

    // create portrait and landscape layout for the TextView
    _layoutParamsPortrait = new RelativeLayout.LayoutParams (ViewGroup.LayoutParams.FillParent, ViewGroup.LayoutParams.WrapContent);
                
    _layoutParamsLandscape = new RelativeLayout.LayoutParams (ViewGroup.LayoutParams.FillParent, ViewGroup.LayoutParams.WrapContent);
    _layoutParamsLandscape.LeftMargin = 100;
    _layoutParamsLandscape.TopMargin = 100;
                        
    _tv = new TextView (this);
                        
    if (surfaceOrientation == SurfaceOrientation.Rotation0 || surfaceOrientation == SurfaceOrientation.Rotation180) {
      _tv.LayoutParameters = _layoutParamsPortrait;
    } else {
      _tv.LayoutParameters = _layoutParamsLandscape;
    }
                        
    _tv.Text = "Programmatic layout";
    rl.AddView (_tv);
    SetContentView (rl);
  }
                
  public override void OnConfigurationChanged (Android.Content.Res.Configuration newConfig)
  {
    base.OnConfigurationChanged (newConfig);
                        
    if (newConfig.Orientation == Android.Content.Res.Orientation.Portrait) {
      _tv.LayoutParameters = _layoutParamsPortrait;
      _tv.Text = "Changed to portrait";
    } else if (newConfig.Orientation == Android.Content.Res.Orientation.Landscape) {
      _tv.LayoutParameters = _layoutParamsLandscape;
      _tv.Text = "Changed to landscape";
    }
  }
}
```

ここでは、`TextView's` のレイアウトパラメーターは、横方向と縦方向の両方に初期化されます。 クラス変数は、`TextView` 自体と共にパラメーターを保持します。これは、方向の変更時にアクティビティが再作成されないためです。 コードは引き続き `OnCreate` の `surfaceOrientartion` を使用して、`TextView`の初期レイアウトを設定します。 その後、`OnConfigurationChanged` は、後続のすべてのレイアウト変更を処理します。

アプリケーションを実行すると、Android はデバイスのローテーションが発生したときにユーザーインターフェイスの変更を読み込み、アクティビティを再起動しません。

## <a name="preventing-activity-restart-for-declarative-layouts"></a>宣言型レイアウトでのアクティビティの再起動の防止

XML でレイアウトを定義すると、デバイスのローテーションによるアクティビティの再起動を防ぐこともできます。 たとえば、アクティビティを再起動しないようにする場合 (パフォーマンス上の理由から)、さまざまな方向に新しいリソースを読み込む必要がない場合に、この方法を使用できます。

これを行うには、プログラムによるレイアウトで使用するのと同じ手順に従います。 先ほど `CodeLayoutActivity` で行ったように、`ActivityAttribute`内の `ConfigurationChanges` を設定するだけです。 方向の変更に対して実行する必要があるすべてのコードは、`OnConfigurationChanged` メソッドに実装することもできます。

## <a name="maintaining-state-during-orientation-changes"></a>向きの変更中の状態の保持

ローテーションを宣言またはプログラムによって処理するかどうかにかかわらず、すべての Android アプリケーションは、デバイスの向きが変更されたときの状態管理と同じ手法を実装します。 Android デバイスがローテーションされると、実行中のアクティビティがシステムによって再起動されるため、状態の管理が重要になります。 Android はこれを行うことで、特定の向き専用に設計されたレイアウトや drawables 実現性など、代替リソースを簡単に読み込むことができます。 このアクティビティを再開すると、ローカルクラス変数に格納されている可能性がある一時的な状態が失われます。 したがって、アクティビティが状態に依存している場合は、そのアクティビティの状態をアプリケーションレベルで永続化する必要があります。 アプリケーションは、向きの変化にわたって保持する必要があるアプリケーションの状態の保存と復元を処理する必要があります。

Android での状態の永続化の詳細については、「[アクティビティライフサイクル](~/android/app-fundamentals/activity-lifecycle/index.md)ガイド」を参照してください。

## <a name="summary"></a>まとめ

この記事では、Android の組み込み機能を使用してローテーションを操作する方法について説明します。 まず、Android リソースシステムを使用して、方向を認識するアプリケーションを作成する方法について説明しました。 次に、コード内にコントロールを追加する方法と、向きの変更を手動で処理する方法について説明します。

## <a name="related-links"></a>関連リンク

- [回転のデモ (サンプル)](https://docs.microsoft.com/samples/xamarin/monodroid-samples/applicationfundamentals-rotationdemo)
- [アクティビティのライフサイクル](~/android/app-fundamentals/activity-lifecycle/index.md)
- [実行時の変更の処理](https://developer.android.com/guide/topics/resources/runtime-changes.html)
- [画面の向きの変更 (高速)](https://android-developers.blogspot.com/2009/02/faster-screen-orientation-change.html)
