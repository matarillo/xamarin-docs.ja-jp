---
title: グラフィックスとアニメーション
description: Android には、2D グラフィックスとアニメーションをサポートするための豊富で多様なフレームワークが用意されています。 このトピックでは、これらのフレームワークについて説明し、Xamarin Android アプリケーションで使用するカスタムグラフィックスとアニメーションを作成する方法について説明します。
ms.prod: xamarin
ms.assetid: 80086318-6FE4-4711-9A71-5C8F8C28C754
ms.technology: xamarin-android
author: davidortinau
ms.author: daortin
ms.date: 02/16/2018
ms.openlocfilehash: eeee9b7d694d9380c653fb87c24171bcaf79389d
ms.sourcegitcommit: 9ab907e053c57fc96419149f83187bc3e8983a6b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/04/2020
ms.locfileid: "75655294"
---
# <a name="android-graphics-and-animation"></a>Android のグラフィックスとアニメーション

_Android は、2D グラフィックスとアニメーションをサポートするための非常に豊富で多様なフレームワークを提供します。このトピックでは、これらのフレームワークについて説明し、Xamarin Android アプリケーションで使用するカスタムグラフィックスとアニメーションを作成する方法について説明します。_

## <a name="overview"></a>概要

多くの場合、性能が制限されているデバイスで実行されていても、最も高い評価を受けているモバイルアプリケーションでは高度なユーザーエクスペリエンス (UX) が用意されており、高品質のグラフィックスやアニメーションにより、直感的で応答性の高い動的な感覚が得られます。 モバイルアプリケーションは、より高度で洗練されているため、アプリケーションからさらに多くのことが求められるようになりました。

さいわいにも、最新のモバイルプラットフォームには、使いやすさを維持しながら、高度なアニメーションやカスタムグラフィックスを作成するための強力なフレームワークが用意されています。 これにより、開発者は非常に少ない労力で、豊富な対話機能を追加できます。

Android の UI API フレームワークは、ほぼ次の2つのカテゴリに分割できます。グラフィックスとアニメーション。

グラフィックスは、2D および3D グラフィックスを実行するためのさまざまなアプローチに分割されます。 3D グラフィックスは、OpenGL ES (モバイル固有バージョンの OpenGL) などのさまざまな組み込みフレームワーク、およびモノゲーム (XNA toolkit と互換性のあるクロスプラットフォームツールキット) などのサードパーティのフレームワークを介して利用できます。 3D グラフィックスはこの記事の範囲外ですが、ここでは、組み込みの2D 描画手法について説明します。

Android には、2D グラフィックスを作成するための2つの異なる API が用意されています。 1つは高レベルの宣言型のアプローチで、もう1つはプログラムによる低レベルの API です。

- 描画用の**リソース**&ndash; これらは、プログラムによって、または (通常は) XML ファイルに描画命令を埋め込むことによって、カスタムグラフィックスを作成するために使用されます。 通常、描画リソースは、Android が2D グラフィックをレンダリングするための命令またはアクションを含む XML ファイルとして定義されます。 

- **Canvas** &ndash; これは、基になるビットマップに直接描画することを必要とする低レベルの API です。 表示される内容を細かく制御できます。 

Android では、これらの2D グラフィックス技術に加えて、いくつかの異なる方法でアニメーションを作成できます。

- &ndash; Android の描画可能な**アニメーション**は、描画可能な*アニメーション*と呼ばれるフレーム単位のアニメーションもサポートしています。 これは最も単純なアニメーション API です。 Android では、(漫画とよく似ているように) 描画リソースを順番に読み込んで表示します。

- **アニメーションの表示 &ndash;** *表示アニメーション*は android の元のアニメーション API であり、android のすべてのバージョンで使用できます。 この API が制限されるのは、ビューオブジェクトでのみ機能し、これらのビューに対して単純な変換を実行できることだけです。
    ビューアニメーションは、通常、`/Resources/anim` フォルダーにある XML ファイルで定義されます。

- Android 3.0 &ndash; の**プロパティアニメーション**では、*プロパティアニメーション*と呼ばれる新しい一連のアニメーション API が導入されました。 これらの新しい API には、表示オブジェクトだけでなく、任意のオブジェクトのプロパティをアニメーション化するために使用できる、拡張可能で柔軟なシステムが導入されました。 この柔軟性により、アニメーションを個別のクラスにカプセル化して、コードを簡単に共有できるようになります。

アニメーションの表示は、以前の Android 3.0 API (API レベル 11) をサポートする必要があるアプリケーションに適しています。 それ以外の場合は、前述の理由により、アプリケーションは新しいプロパティアニメーション API を使用する必要があります。

これらのフレームワークはすべて実行可能なオプションですが、可能な限り、プロパティアニメーションに設定することをお勧めします。これは、より柔軟な API であるためです。 プロパティアニメーションを使用すると、アニメーションロジックを個別のクラスにカプセル化できるため、コードの共有が容易になり、コードの保守が簡単になります。

## <a name="accessibility"></a>ユーザー補助

グラフィックスとアニメーションは、Android アプリを魅力的で使いやすくするのに役立ちます。ただし、スクリーンリーダー、代替入力デバイス、または支援ズームによって、一部の対話が行われることに注意することが重要です。
また、オーディオ機能を使用しないと、一部の対話が行われる場合もあります。

このような状況では、ユーザーインターフェイスにヒントとナビゲーションアシスタンスを提供し、UI の要素を視覚的に表現するテキストコンテンツや説明があることを確認することで、アプリの方が便利です。

Android のユーザー補助 Api を利用する方法の詳細については、 [Google のアクセシビリティガイド](https://developer.android.com/guide/topics/ui/accessibility/)を参照してください。

## <a name="2d-graphics"></a>2D グラフィックス

描画リソースは、Android アプリケーションで一般的な手法です。 他のリソースと同様に、描画リソースは、XML ファイルで定義されている &ndash; 宣言型のリソースです。 この方法により、リソースからコードを明確に分離できます。 これにより、Android アプリケーションのグラフィックスを更新または変更するためにコードを変更する必要がないため、開発と保守が簡単になります。 ただし、合成されたリソースは、多くの単純で一般的なグラフィック要件に役立ちますが、キャンバス API のパワーと制御はありません。

[Canvas](xref:Android.Graphics.Canvas)オブジェクトを使用したもう1つの手法は、他の従来の API フレームワーク (例、Drawing や IOS のコア描画) によく似ています。 Canvas オブジェクトを使用すると、2D グラフィックスの作成方法を最も細かく制御できます。 これは、描画可能なリソースが動作しない場合や、操作が困難な場合に適しています。 たとえば、スライダーの値に関連する計算に基づいて外観が変化するカスタムスライダーコントロールを描画する必要がある場合があります。

まず、事前に用意されたリソースを調べてみましょう。 これらは単純で、最も一般的なカスタム描画ケースをカバーしています。

### <a name="drawable-resources"></a>描画リソース

描画リソースは、ディレクトリ `/Resources/drawable`の XML ファイルで定義されます。 PNG や JPEG の埋め込みとは異なり、密度固有のリソースのバージョンを提供する必要はありません。
実行時に、Android アプリケーションはこれらのリソースを読み込み、これらの XML ファイルに含まれている命令を使用して2D グラフィックスを作成します。
Android では、いくつかの異なる種類の描画リソースを定義しています。

- [図形の描画用 &ndash; これ](https://developer.android.com/guide/topics/resources/drawable-resource.html#Shape)は、プリミティブジオメトリック形状を描画し、その図形に対して限られたグラフィカル効果のセットを適用する、描画できるオブジェクトです。 これらは、ボタンをカスタマイズしたり、テキストビューの背景を設定したりする場合に非常に便利です。 `ShapeDrawable` を使用する方法の例については、この記事の後半で説明します。

- [Statelistdrawable ル](https://developer.android.com/guide/topics/resources/drawable-resource.html#StateList)&ndash; これは、ウィジェット/コントロールの状態に基づいて外観を変更する、描画できるリソースです。 たとえば、ボタンが押されているかどうかによって、ボタンの外観が変わる場合があります。

- &ndash;[レイヤー](https://developer.android.com/guide/topics/resources/drawable-resource.html#LayerList)の描画を行うと、他のいくつかの引き出しが1つ上にあると考えられる他のいくつかの drawables を積み重ねることができます。 次のスクリーンショットでは、*レイヤー*描画の例を示しています。

    ![レイヤー描画の例](graphics-and-animation-images/image1.png)

- この[&ndash; は](https://developer.android.com/guide/topics/resources/drawable-resource.html#Transition)、*レイヤー*の描画ができますが、違いは1つあります。 1*つのレイヤー*を別のレイヤー上に表示するようにアニメーション化することができます。

- [Levellistdrawable ル](https://developer.android.com/guide/topics/resources/drawable-resource.html#LevelList)&ndash; これは、特定の条件に基づいてイメージを表示するという点で、 *statelistdrawable ル*と非常によく似ています。 ただし、 *Statelistdrawable ル*とは異なり、 *levellistdrawable ル*は、整数値に基づいてイメージを表示します。 *Levellistdrawable ル*の例として、WiFi 信号の強さを表示する方法があります。 WiFi 信号の強度が変化すると、表示される描画用の設定が変わります。

- [ScaleDrawable](https://developer.android.com/guide/topics/resources/drawable-resource.html#Scale)/[ClipDrawable](https://developer.android.com/guide/topics/resources/drawable-resource.html#Clip) &ndash;を使用すると、その名前が示すように、これらのdrawablesスケーリングとクリッピングの両方の機能を提供します。 *Scaledrawable*描画可能な場合は、別の描画可能なスケーリングが行われますが、 *clipdrawable*ルは別の描画可能なクリップをクリップします。

- [Insetdrawable](https://developer.android.com/guide/topics/resources/drawable-resource.html#Inset) &ndash; この描画を行うと、別の描画リソースの側にインセットが適用されます。 ビューの実際の境界よりも小さい背景がビューに必要な場合に使用されます。

- XML [Bitmapdrawable ル](https://developer.android.com/guide/topics/resources/drawable-resource.html#Bitmap)&ndash; このファイルは、実際のビットマップに対して実行される一連の命令 (xml) です。 Android が実行できるアクションには、タイル、ディザリング、アンチエイリアシングなどがあります。 この一般的な用途の1つは、レイアウトの背景全体にビットマップを並べて表示することです。

#### <a name="drawable-example"></a>描画の例

`ShapeDrawable`を使用して2D グラフィックを作成する方法の簡単な例を見てみましょう。 `ShapeDrawable` では、四角形、楕円、線、およびリングという4つの基本図形のいずれかを定義できます。 また、グラデーション、色、サイズなどの基本的な効果を適用することもできます。 次の XML は、(ファイル `Resources/drawable/shape_rounded_blue_rect.xml`内の)*アニメーションの Sdemo*コンパニオンプロジェクトに含まれる `ShapeDrawable` です。
これは、紫色のグラデーションの背景と丸い角を持つ四角形を定義します。

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
<!-- Specify a gradient for the background -->
<gradient android:angle="45"
          android:startColor="#55000066"
          android:centerColor="#00000000"
          android:endColor="#00000000"
          android:centerX="0.75" />

<padding android:left="5dp"
          android:right="5dp"
          android:top="5dp"
          android:bottom="5dp" />

<corners android:topLeftRadius="10dp"
          android:topRightRadius="10dp"
          android:bottomLeftRadius="10dp"
          android:bottomRightRadius="10dp" />
</shape>
```

この描画可能なリソースは、次の XML に示すように、レイアウトまたは他の組み込み先で宣言によって参照できます。

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:background="#33000000">
    <TextView android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_centerInParent="true"
              android:background="@drawable/shape_rounded_blue_rect"
              android:text="@string/message_shapedrawable" />
</RelativeLayout>
```

描画可能リソースは、プログラムによって適用することもできます。 次のコードスニペットは、プログラムで TextView の背景を設定する方法を示しています。

```csharp
TextView tv = FindViewById<TextView>(Resource.Id.shapeDrawableTextView);
tv.SetBackgroundResource(Resource.Drawable.shape_rounded_blue_rect);
```

これがどのように表示されるかを確認するには、[*アニメーション*] プロジェクトを実行し、メインメニューから [図形の構築] 項目を選択します。 次のスクリーンショットのような内容が表示されます。

[![カスタムの背景を持つ Textview を します。グラデーションと丸い角で描画されます](graphics-and-animation-images/image2-sml.png)](graphics-and-animation-images/image2.png#lightbox)

XML 要素および描画リソースの構文の詳細については、 [Google のドキュメント](https://developer.android.com/guide/topics/resources/drawable-resource.html#Shape)を参照してください。

### <a name="using-the-canvas-drawing-api"></a>キャンバス描画 API の使用

Drawables 強力ですが、制限があります。 特定の項目が不可能または非常に複雑な場合 (たとえば、デバイスのカメラによって撮影された画像にフィルターを適用する場合)。 描画可能なリソースを使用すると、赤目の縮小を適用することが非常に困難になります。
代わりに、Canvas API を使用すると、アプリケーションは、画像の特定の部分で色を選択的に変更することが非常にきめ細かく制御できます。

キャンバスで一般的に使用されるクラスの1つに、[描画](xref:Android.Graphics.Paint)クラスがあります。 このクラスは、描画方法に関する色とスタイルの情報を保持します。 色や透明度などを提供するために使用されます。

Canvas API は、*塗装のモデル*を使用して2d グラフィックスを描画します。
操作は、相互に連続するレイヤーで適用されます。 各操作は、基になるビットマップの領域をカバーします。 領域が前に描画された領域と重なると、新しい描画は部分的または完全に不透明になります。 これは、他の多くの描画 Api (たとえば、Drawing や iOS のコアグラフィックス) が動作するのと同じ方法です。

`Canvas` オブジェクトを取得するには、次の2つの方法があります。 最初の方法では、 [Bitmap](xref:Android.Graphics.Bitmap)オブジェクトを定義し、それを使用して `Canvas` オブジェクトをインスタンス化します。 たとえば、次のコードスニペットは、基になるビットマップを使用して新しいキャンバスを作成します。

```csharp
Bitmap bitmap = Bitmap.CreateBitmap(100, 100, Bitmap.Config.Argb8888);
Canvas canvas = new Canvas(b);
```

`Canvas` オブジェクトを取得するもう1つの方法は、[ビュー](xref:Android.Views.View)の基底クラスを提供する[OnDraw](xref:Android.Views.View.OnDraw*)コールバックメソッドです。 Android は、ビューがそれ自体を描画する必要があると判断したときにこのメソッドを呼び出し、ビューが操作するために `Canvas` オブジェクトに渡す必要があります。

Canvas クラスは、描画命令をプログラムによって提供するメソッドを公開します。 次に例を示します。

- [Canvas 描画](xref:Android.Graphics.Canvas.DrawPaint*)&ndash; 指定した描画を使用してキャンバスのビットマップ全体を塗りつぶします。

- 指定した描画を使用して、指定した幾何学図形を描画するための &ndash; し[ます。](xref:Android.Graphics.Canvas.DrawPath*)

- [Canvas.DrawText](xref:Android.Graphics.Canvas.DrawText*) &ndash; は、指定された色でキャンバス上のテキストを描画します。 テキストは `x,y` の位置に描画されます。

#### <a name="drawing-with-the-canvas-api"></a>Canvas API を使用した描画

次に、キャンバス API の動作例を示します。 次のコードスニペットは、ビューを描画する方法を示しています。

```csharp
public class MyView : View
{
    protected override void OnDraw(Canvas canvas)
    {
        base.OnDraw(canvas);
        Paint green = new Paint {
            AntiAlias = true,
            Color = Color.Rgb(0x99, 0xcc, 0),
        };
        green.SetStyle(Paint.Style.FillAndStroke);

        Paint red = new Paint {
            AntiAlias = true,
            Color = Color.Rgb(0xff, 0x44, 0x44)
        };
        red.SetStyle(Paint.Style.FillAndStroke);

        float middle = canvas.Width * 0.25f;
        canvas.DrawPaint(red);
        canvas.DrawRect(0, 0, middle, canvas.Height, green);
    }
}
```

このコードでは、最初に赤のペイントオブジェクトと緑色の描画オブジェクトを作成します。 キャンバスの内容を赤で塗りつぶし、キャンバスの幅の25% である緑の四角形を描画するようにキャンバスに指示します。 この例については、この記事のソースコードに含まれている `AnimationsDemo` プロジェクトを参照してください。 アプリケーションを起動し、メインメニューから描画項目を選択すると、次のような画面が表示されます。

[![赤の描画オブジェクトと緑の描画オブジェクトを含む 画面](graphics-and-animation-images/image3-sml.png)](graphics-and-animation-images/image3.png#lightbox)

## <a name="animation"></a>アニメーション

アプリケーション内を移動するユーザーのようなものです。 アニメーションは、アプリケーションのユーザーエクスペリエンスを向上させ、それを支援するための優れた方法です。最適なアニメーションは、自然であることが理由でユーザーに通知されないものです。 Android には、アニメーション用に次の3つの API が用意されています。

- **アニメーションを表示**&ndash; これは元の API です。 これらのアニメーションは特定のビューに関連付けられており、ビューのコンテンツに対して単純な変換を実行できます。 この API は単純であるため、アルファアニメーションや回転などにも役立ちます。

- プロパティ**アニメーション &ndash; プロパティアニメーションは**、Android 3.0 で導入されました。 これにより、アプリケーションはほぼすべてをアニメーション化できるようになります。 プロパティアニメーションは、オブジェクトが画面に表示されていない場合でも、任意のオブジェクトのプロパティを変更するために使用できます。

- 描画用の**アニメーション**では、この特殊な描画用リソース &ndash;、レイアウトに非常に単純なアニメーション効果を適用するために使用されます。

一般に、プロパティアニメーションはより柔軟性が高く、より多くの機能を提供するため、使用する優先システムです。

### <a name="view-animations"></a>アニメーションの表示

ビューアニメーションはビューに限定され、開始点、終了点、サイズ、回転、透明度などの値に対してのみアニメーションを実行できます。
これらの種類のアニメーションは、通常、*トゥイーンアニメーション*と呼ばれます。 ビューアニメーションは、コード内で、または XML ファイルを使用して、プログラムによって &ndash; する2つの方法で定義できます。 XML ファイルは、より読みやすく、保守が簡単なため、ビューアニメーションを宣言するために推奨される方法です。

アニメーション XML ファイルは、Xamarin. Android プロジェクトの `/Resources/anim` ディレクトリに格納されます。 このファイルには、ルート要素として次のいずれかの要素が含まれている必要があります。

- フェードインまたはフェードアウトアニメーションを `alpha` &ndash; ます。

- 回転アニメーションを `rotate` &ndash; ます。

- サイズ変更アニメーション &ndash; `scale` ます。

- 水平方向または垂直方向のモーション &ndash; `translate` ます。

- 1つ以上の他のアニメーション要素を保持できるコンテナーを `set` &ndash; ます。

既定では、XML ファイル内のすべてのアニメーションが同時に適用されます。 アニメーションを順番に実行するには、上で定義した要素の1つに `android:startOffset` 属性を設定します。

*Interpolator*を使用すると、アニメーションの変化率に影響を与える可能性があります。 Interpolator を使用すると、アニメーション効果を加速、繰り返し、または decelerated にすることができます。 Android framework には、いくつかの interpolators が用意されています (ただし、これらに限定されません)。

- これらの interpolators を `AccelerateInterpolator` / &ndash; `DecelerateInterpolator` は、アニメーションの変化率を増減させます。

- 変更が最後にバウンスされる &ndash; `BounceInterpolator` ます。

- `LinearInterpolator` &ndash; 変化率は一定です。

次の XML は、これらの要素の一部を組み合わせたアニメーションファイルの例を示しています。

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android=http://schemas.android.com/apk/res/android
     android:shareInterpolator="false">

    <scale android:interpolator="@android:anim/accelerate_decelerate_interpolator"
           android:fromXScale="1.0"
           android:toXScale="1.4"
           android:fromYScale="1.0"
           android:toYScale="0.6"
           android:pivotX="50%"
           android:pivotY="50%"
           android:fillEnabled="true"
           android:fillAfter="false"
           android:duration="700" />

    <set android:interpolator="@android:anim/accelerate_interpolator">
        <scale android:fromXScale="1.4"
               android:toXScale="0.0"
               android:fromYScale="0.6"
               android:toYScale="0.0"
               android:pivotX="50%"
               android:pivotY="50%"
               android:fillEnabled="true"
               android:fillBefore="false"
               android:fillAfter="true"
               android:startOffset="700"
               android:duration="400" />

        <rotate android:fromDegrees="0"
                android:toDegrees="-45"
                android:toYScale="0.0"
                android:pivotX="50%"
                android:pivotY="50%"
                android:fillEnabled="true"
                android:fillBefore="false"
                android:fillAfter="true"
                android:startOffset="700"
                android:duration="400" />
    </set>
</set>
```

このアニメーションは、すべてのアニメーションを同時に実行します。 最初のスケールアニメーションでは、イメージが水平方向に拡大され、垂直方向に縮小されます。その後、イメージは、45°の反時計回りに回転し、画面から消えます。

アニメーションは、アニメーションを拡大し、ビューに適用することで、プログラムによってビューに適用できます。 Android には、アニメーションリソースを拡大し、`Android.Views.Animations.Animation`のインスタンスを返すヘルパークラス `Android.Views.Animations.AnimationUtils` が用意されています。 このオブジェクトは、`StartAnimation` を呼び出し、`Animation` オブジェクトを渡すことによって、ビューに適用されます。 次のコードスニペットは、この例を示しています。

```csharp
Animation myAnimation = AnimationUtils.LoadAnimation(Resource.Animation.MyAnimation);
ImageView myImage = FindViewById<ImageView>(Resource.Id.imageView1);
myImage.StartAnimation(myAnimation);
```

これで、ビューアニメーションのしくみについての基本的な理解が得られたので、プロパティアニメーションに移動できるようになりました。

### <a name="property-animations"></a>プロパティアニメーション

プロパティアニメーターは、Android 3.0 で導入された新しい API です。
これらは、任意のオブジェクトの任意のプロパティをアニメーション化するために使用できる、より拡張可能な API を提供します。

すべてのプロパティアニメーションは、[アニメーター](xref:Android.Animation.Animator)サブクラスのインスタンスによって作成されます。 アプリケーションは、このクラスを直接使用するのではなく、次のいずれかのサブクラスを使用します。

- [Valueanimator](xref:Android.Animation.ValueAnimator) &ndash; このクラスは、プロパティアニメーション API 全体で最も重要なクラスです。 変更する必要があるプロパティの値を計算します。 `ViewAnimator` は、これらの値を直接更新しません。代わりに、アニメーションオブジェクトを更新するために使用できるイベントを発生させます。

- [Objectanimator](xref:Android.Animation.ObjectAnimator) &ndash; このクラスは `ValueAnimator` のサブクラスです。 ターゲットオブジェクトと更新するプロパティを受け入れることによって、オブジェクトをアニメーション化するプロセスを簡略化することを目的としています。

- アニメーションの[設定](xref:Android.Animation.AnimatorSet)&ndash; このクラスは、アニメーションを互いに相対的に実行する方法を調整します。 アニメーションは、連続して、または指定された間隔で同時に実行できます。

*エバリュエーター*は、アニメーション中に新しい値を計算するためにアニメーターによって使用される特殊なクラスです。 既定では、Android には次のエバリュエーターが用意されています。

- [&ndash; は](xref:Android.Animation.IntEvaluator)、整数プロパティの値を計算します。

- [FloatEvaluator](xref:Android.Animation.FloatEvaluator) &ndash; は、float プロパティの値を計算します。

- [Argbevaluator](xref:Android.Animation.ArgbEvaluator) &ndash; は、色のプロパティの値を計算します。

アニメーション化されているプロパティが `float`、`int` またはカラーではない場合、アプリケーションは `ITypeEvaluator` インターフェイスを実装することによって独自のエバリュエーターを作成できます。 (カスタムエバリュエーターの実装については、このトピックでは扱いません)。

#### <a name="using-the-valueanimator"></a>ValueAnimator の使用

アニメーションには2つの部分があります。アニメーション化された値を計算し、いくつかのオブジェクトのプロパティに値を設定します。 
[Valueanimator](xref:Android.Animation.ValueAnimator)は値のみを計算しますが、オブジェクトに直接は作用しません。 代わりに、アニメーションの有効期間中に呼び出されるイベントハンドラー内でオブジェクトが更新されます。 この設計では、アニメーション化された1つの値から複数のプロパティを更新できます。

`ValueAnimator` のインスタンスを取得するには、次のいずれかのファクトリメソッドを呼び出します。

- `ValueAnimator.OfInt`
- `ValueAnimator.OfFloat`
- `ValueAnimator.OfObject`

この処理が完了すると、`ValueAnimator` インスタンスの期間が設定されている必要があります。その後、インスタンスを開始できます。 次の例は、1000ミリ秒の範囲内で0から1の値をアニメーション化する方法を示しています。

```csharp
ValueAnimator animator = ValueAnimator.OfInt(0, 100);
animator.SetDuration(1000);
animator.Start();
```

ただし、上記のコードスニペットは、アニメーターが実行される &ndash; はあまり役に立ちませんが、更新された値のターゲットはありません。 `Animator` クラスは、リスナーに新しい値を通知する必要があると判断した場合に、更新イベントを発生させます。 アプリケーションでは、次のコードスニペットに示すように、このイベントに応答するためのイベントハンドラーを提供できます。

```csharp
MyCustomObject myObj = new MyCustomObject();
myObj.SomeIntegerValue = -1;

animator.Update += (object sender, ValueAnimator.AnimatorUpdateEventArgs e) =>
{
    int newValue = (int) e.Animation.AnimatedValue;
    // Apply this new value to the object being animated.
    myObj.SomeIntegerValue = newValue;
};
```

`ValueAnimator`について理解できたので、`ObjectAnimator`の詳細について説明します。

#### <a name="using-the-objectanimator"></a>ObjectAnimator の使用

[Objectanimator](xref:Android.Animation.ObjectAnimator)は `ViewAnimator` のサブクラスであり、`ValueAnimator` のタイミングエンジンと値の計算を、イベントハンドラーを接続するために必要なロジックと結合します。 `ValueAnimator` では、アプリケーションでイベントハンドラーを明示的に接続する必要があります。これにより、`ObjectAnimator` はこの手順を自動的に実行 &ndash; ます。

`ObjectAnimator` 用の API は、`ViewAnimator`の API と非常によく似ていますが、オブジェクトと更新するプロパティの名前を指定する必要があります。 次の例は、`ObjectAnimator`の使用例を示しています。

```csharp
MyCustomObject myObj = new MyCustomObject();
myObj.SomeIntegerValue = -1;

ObjectAnimator animator = ObjectAnimator.OfFloat(myObj, "SomeIntegerValue", 0, 100);
animator.SetDuration(1000);
animator.Start();
```

前のコードスニペットからわかるように、`ObjectAnimator` を使用すると、オブジェクトをアニメーション化するために必要なコードを減らすことができます。

### <a name="drawable-animations"></a>描画効果のあるアニメーション

最終的なアニメーション API は、描画用のアニメーション API です。 描画されたアニメーションは、一連の描画リソースをもう一方の後に1つずつ読み込み、反転 it の漫画と同様に順番に表示します。

描画リソースは、ルート要素として `<animation-list>` 要素を持ち、アニメーションの各フレームを定義する一連の `<item>` 要素を持つ XML ファイルで定義されます。 この XML ファイルは、アプリケーションの `/Resource/drawable` フォルダーに格納されています。 次の XML は、描画されたアニメーションの一例です。

```xml
<animation-list xmlns:android="http://schemas.android.com/apk/res/android">
  <item android:drawable="@drawable/asteroid01" android:duration="100" />
  <item android:drawable="@drawable/asteroid02" android:duration="100" />
  <item android:drawable="@drawable/asteroid03" android:duration="100" />
  <item android:drawable="@drawable/asteroid04" android:duration="100" />
  <item android:drawable="@drawable/asteroid05" android:duration="100" />
  <item android:drawable="@drawable/asteroid06" android:duration="100" />
</animation-list>
```

このアニメーションは6つのフレームを通じて実行されます。 `android:duration` 属性は、各フレームを表示する期間を宣言します。 次のコードスニペットは、描画されたアニメーションを作成し、ユーザーが画面上のボタンをクリックしたときに開始する例を示しています。

```csharp
AnimationDrawable _asteroidDrawable;

protected override void OnCreate(Bundle bundle)
{
    base.OnCreate(bundle);
    SetContentView(Resource.Layout.Main);

    _asteroidDrawable = (Android.Graphics.Drawables.AnimationDrawable)
    Resources.GetDrawable(Resource.Drawable.spinning_asteroid);

    ImageView asteroidImage = FindViewById<ImageView>(Resource.Id.imageView2);
    asteroidImage.SetImageDrawable((Android.Graphics.Drawables.Drawable) _asteroidDrawable);

    Button asteroidButton = FindViewById<Button>(Resource.Id.spinAsteroid);
    asteroidButton.Click += (sender, e) =>
    {
        _asteroidDrawable.Start();
    };
}
```

この時点で、Android アプリケーションで使用できるアニメーション Api の基礎について説明しました。

## <a name="summary"></a>まとめ

この記事では、Android アプリケーションにいくつかのグラフィックスを追加するのに役立つ、多数の新しい概念と API を紹介しました。 まず、さまざまな2D グラフィックス API について説明し、Android が Canvas オブジェクトを使用してアプリケーションを画面に直接描画できるようにする方法を示しました。 また、XML ファイルを使用して、グラフィックスを宣言によって作成できる代替技法もいくつか紹介しました。 次に、Android でアニメーションを作成するための古い API と新しい API について説明しました。

## <a name="related-links"></a>関連リンク

- [アニメーションのデモ (サンプル)](https://docs.microsoft.com/samples/xamarin/monodroid-samples/animationdemo)
- [アニメーションとグラフィックス](https://developer.android.com/guide/topics/graphics/index.html)
- [アニメーションを使用して Mobile Apps を有効にする](https://youtu.be/ikSk_ILg3d0)
- [AnimationDrawable](xref:Android.Graphics.Drawables.AnimationDrawable)
- [Canvas](xref:Android.Graphics.Canvas)
- [オブジェクトのアニメーター](xref:Android.Animation.ObjectAnimator)
- [値のアニメーター](xref:Android.Animation.ValueAnimator)
