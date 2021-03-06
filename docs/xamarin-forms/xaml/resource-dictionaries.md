---
title: リソース ディクショナリ
description: XAML リソースは、共有し、Xamarin.Forms アプリケーション全体で再利用が可能なオブジェクトです。
ms.prod: xamarin
ms.assetid: DF103686-4A92-40FA-9CF1-A9376293B13C
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 08/15/2019
ms.custom: video
ms.openlocfilehash: 49ea5c2a8332e625710af8a9947e1cb0e0338040
ms.sourcegitcommit: 211fed94fb96127a3e158ae1ff5d7eb831a203d8
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/15/2020
ms.locfileid: "75955828"
---
# <a name="resource-dictionaries"></a>リソース ディクショナリ

[![サンプルのダウンロード](~/media/shared/download.png)サンプルのダウンロード](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/xaml-resourcedictionaries)

_XAML リソースは、Xamarin. フォームアプリケーション全体で共有および再利用できるオブジェクトの定義です。これらのリソースオブジェクトは、リソースディクショナリに格納されます。_

A [ `ResourceDictionary` ](xref:Xamarin.Forms.ResourceDictionary) Xamarin.Forms アプリケーションで使用されるリソースのリポジトリです。 一般的なリソースが含まれている、`ResourceDictionary`含める[スタイル](~/xamarin-forms/user-interface/styles/index.md)、[コントロール テンプレート](~/xamarin-forms/app-fundamentals/templates/control-template.md)、[データ テンプレート](~/xamarin-forms/app-fundamentals/templates/data-templates/index.md)色、およびコンバーター。

XAML に格納されているリソースで、`ResourceDictionary`取得して使用して要素に適用できるし、`StaticResource`マークアップ拡張機能。 C# の場合は、リソース定義することもに、`ResourceDictionary`を取得して、文字列ベースのインデクサーを使用して要素に適用します。 ただし、使用する利点のほとんどは、`ResourceDictionary`で c# の場合は、共有オブジェクトは単にフィールドまたはプロパティとして格納されているし、しない直接アクセスに最初から取得ディクショナリ。

## <a name="create-and-consume-a-resourcedictionary"></a>ResourceDictionary の作成と使用

リソースが定義されている、 [ `ResourceDictionary` ](xref:Xamarin.Forms.ResourceDictionary) 、次のいずれかに設定されている`Resources`プロパティ。

- [ `Resources` ](xref:Xamarin.Forms.Application.Resources)から派生したクラスのプロパティ [`Application`](xref:Xamarin.Forms.Application)
- [ `Resources` ](xref:Xamarin.Forms.VisualElement.Resources)から派生したクラスのプロパティ [`VisualElement`](xref:Xamarin.Forms.Application)

Xamarin.Forms のプログラムにはから派生したクラス 1 つだけにはが含まれています`Application`多くの場合を使用して、多くのクラスから派生するが、 `VisualElement`(ページ、レイアウト、コントロールなど)。 これらのオブジェクトのいずれかがその`Resources`プロパティに設定、`ResourceDictionary`します。 特定の配置場所を選択する`ResourceDictionary`影響は、リソースを使用できます。

- 内のリソースを`ResourceDictionary`など、ビューに接続される`Button`または`Label`のためこれは非常に便利なのみに、その特定のオブジェクトに適用できます。
- 内のリソースを`ResourceDictionary`などのレイアウトにアタッチされている`StackLayout`または`Grid`レイアウトとそのレイアウトのすべての子に適用できます。
- 内のリソースを`ResourceDictionary`定義されているページとそのすべての子ページにレベルは適用できます。
- 内のリソースを`ResourceDictionary`定義されているアプリケーションで、アプリケーション全体でレベルを適用できます。

次の XAML には、アプリケーション レベルで定義されているリソースが表示されます。`ResourceDictionary`で、 **App.xaml**標準 Xamarin.Forms プログラムの一部として作成されたファイル。

```xaml
<Application ...>
    <Application.Resources>
        <ResourceDictionary>
            <Color x:Key="PageBackgroundColor">Yellow</Color>
            <Color x:Key="HeadingTextColor">Black</Color>
            <Color x:Key="NormalTextColor">Blue</Color>
            <Style x:Key="LabelPageHeadingStyle" TargetType="Label">
                <Setter Property="FontAttributes" Value="Bold" />
                <Setter Property="HorizontalOptions" Value="Center" />
                <Setter Property="TextColor" Value="{StaticResource HeadingTextColor}" />
            </Style>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

これは、`ResourceDictionary`定義 3 [ `Color` ](xref:Xamarin.Forms.Color)リソースと[ `Style` ](xref:Xamarin.Forms.Style)リソース。 詳細については、`App`クラスを参照してください[App クラス](~/xamarin-forms/app-fundamentals/application-class.md)します。

明示的な Xamarin.Forms 3.0 以降では`ResourceDictionary`タグは必要ありません。 `ResourceDictionary`オブジェクトが自動的に作成され、間で直接、リソースを挿入することができます、`Resources`プロパティ要素タグ。

```xaml
<Application ...>
    <Application.Resources>
        <Color x:Key="PageBackgroundColor">Yellow</Color>
        <Color x:Key="HeadingTextColor">Black</Color>
        <Color x:Key="NormalTextColor">Blue</Color>
        <Style x:Key="LabelPageHeadingStyle" TargetType="Label">
            <Setter Property="FontAttributes" Value="Bold" />
            <Setter Property="HorizontalOptions" Value="Center" />
            <Setter Property="TextColor" Value="{StaticResource HeadingTextColor}" />
        </Style>
    </Application.Resources>
</Application>
```

各リソースを使用して指定されているキーには、`x:Key`属性には、ディクショナリ キーになりますが、`ResourceDictionary`します。 リソースを取得するキーが使用される、`ResourceDictionary`によって、 [ `StaticResource` ](xref:Xamarin.Forms.Xaml.StaticResourceExtension)マークアップ拡張機能、内で定義された追加のリソースを示す次の XAML コードの例のように、 `StackLayout`:

```xaml
<StackLayout Margin="0,20,0,0">
  <StackLayout.Resources>
    <ResourceDictionary>
      <Style x:Key="LabelNormalStyle" TargetType="Label">
        <Setter Property="TextColor" Value="{StaticResource NormalTextColor}" />
      </Style>
      <Style x:Key="MediumBoldText" TargetType="Button">
        <Setter Property="FontSize" Value="Medium" />
        <Setter Property="FontAttributes" Value="Bold" />
      </Style>
    </ResourceDictionary>
  </StackLayout.Resources>
  <Label Text="ResourceDictionary Demo" Style="{StaticResource LabelPageHeadingStyle}" />
    <Label Text="This app demonstrates consuming resources that have been defined in resource dictionaries."
           Margin="10,20,10,0"
           Style="{StaticResource LabelNormalStyle}" />
    <Button Text="Navigate"
            Clicked="OnNavigateButtonClicked"
            TextColor="{StaticResource NormalTextColor}"
            Margin="0,20,0,0"
            HorizontalOptions="Center"
            Style="{StaticResource MediumBoldText}" />
</StackLayout>
```

最初の[ `Label` ](xref:Xamarin.Forms.Label)インスタンスを取得し、使用、`LabelPageHeadingStyle`アプリケーション レベルで定義されているリソース`ResourceDictionary`、2 つ目`Label`インスタンスの取得および使用、 `LabelNormalStyle`制御レベルで定義されているリソース`ResourceDictionary`します。 同様に、 [ `Button` ](xref:Xamarin.Forms.Button)インスタンスを取得し、使用、`NormalTextColor`アプリケーション レベルで定義されているリソース`ResourceDictionary`、および`MediumBoldText`制御レベルで定義されているリソース`ResourceDictionary`します。 この結果、次のスクリーンショットに示すような外観が表示されます。

[![ResourceDictionary リソースを消費している](resource-dictionaries-images/screenshots-sml.png)](resource-dictionaries-images/screenshots.png#lightbox)

> [!NOTE]
> 1 つのページに固有のリソースは、アプリケーション レベルのリソース ディクショナリをそのため、ページで必要なときに、リソースの代わりに、アプリケーションの起動時に解析されますに含めることはできません。 詳細については、次を参照してください。[アプリケーション リソース ディクショナリのサイズを減らす](~/xamarin-forms/deploy-test/performance.md)します。

## <a name="override-resources"></a>リソースの上書き

ときに`ResourceDictionary`のリソースを共有`x:Key`属性値は、ビューの階層内で下に定義されているリソースは優先を以上定義されています。 たとえば、設定、`PageBackgroundColor`リソースを`Blue`アプリケーションでレベルがページ レベルで上書きされている`PageBackgroundColor`リソースに設定`Yellow`。 ページ レベルでは同様に、`PageBackgroundColor`制御レベルによってリソースが上書きされます`PageBackgroundColor`リソース。 この優先順位は次の XAML コード例について説明します。

```xaml
<ContentPage ... BackgroundColor="{StaticResource PageBackgroundColor}">
    <ContentPage.Resources>
        <ResourceDictionary>
            <Color x:Key="PageBackgroundColor">Blue</Color>
            <Color x:Key="NormalTextColor">Yellow</Color>
        </ResourceDictionary>
    </ContentPage.Resources>
    <StackLayout Margin="0,20,0,0">
        ...
        <Label Text="ResourceDictionary Demo" Style="{StaticResource LabelPageHeadingStyle}" />
        <Label Text="This app demonstrates consuming resources that have been defined in resource dictionaries."
               Margin="10,20,10,0"
               Style="{StaticResource LabelNormalStyle}" />
        <Button Text="Navigate"
                Clicked="OnNavigateButtonClicked"
                TextColor="{StaticResource NormalTextColor}"
                Margin="0,20,0,0"
                HorizontalOptions="Center"
                Style="{StaticResource MediumBoldText}" />
    </StackLayout>
</ContentPage>
```

元の`PageBackgroundColor`と`NormalTextColor`によって、アプリケーション レベルで定義されている、インスタンスが上書きされます、`PageBackgroundColor`と`NormalTextColor`ページ レベルで定義されているインスタンス。 したがって、ページの背景色が青と、次のスクリーン ショットに示すよう、ページ上のテキストが、黄色になります。

[![ResourceDictionary リソースのオーバーライド](resource-dictionaries-images/overridding-screenshots-sml.png)](resource-dictionaries-images/overridding-screenshots.png#lightbox)

ただし、注意のバック グラウンド バー、 [ `NavigationPage` ](xref:Xamarin.Forms.NavigationPage)黄色のまま、ため、 [ `BarBackgroundColor` ](xref:Xamarin.Forms.NavigationPage.BarBackgroundColor)の値に設定されて、`PageBackgroundColor`アプリケーションで定義されているリソースレベル`ResourceDictionary`します。

について検討する別の方法を次に示します`ResourceDictionary`優先順位: ときに、XAML パーサーが検出した、 `StaticResource`、ビジュアル ツリーを介したを結ぶこと、一致するキーを検索、見つかった最初の一致を使用します。 XAML パーサーが検索ページでこの検索を終了し、キーもまだ検出された場合、`ResourceDictionary`にアタッチされている、`App`オブジェクト。 でも、キーが存在しない場合は、例外が発生します。

## <a name="stand-alone-resource-dictionaries"></a>スタンドアロンリソースディクショナリ

派生したクラス`ResourceDictionary`別のスタンドアロン ファイルにもできます。 (正確には、`ResourceDictionary` から派生したクラスには、通常、ファイルの_ペア_が必要です。これは、リソースが XAML ファイルで定義されていても、`InitializeComponent` 呼び出しを伴う分離コードファイルが必要なためです)。結果ファイルは、アプリケーション間で共有できます。

このようなファイルを作成するには、追加、新しい**コンテンツ ビュー**または**コンテンツ ページ**項目をプロジェクト (ではなく、**コンテンツ ビュー**または**コンテンツ ページ**でc# ファイルのみ)。 XAML ファイルと c# ファイルの両方でから基底クラスの名前を変更`ContentView`または`ContentPage`に`ResourceDictionary`します。 XAML ファイルでは、基底クラスの名前は、最上位要素です。

次の XAML の例は、 [ `ResourceDictionary` ](xref:Xamarin.Forms.ResourceDictionary)という`MyResourceDictionary`:

```xaml
<ResourceDictionary xmlns="http://xamarin.com/schemas/2014/forms"
                    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                    x:Class="ResourceDictionaryDemo.MyResourceDictionary">
    <DataTemplate x:Key="PersonDataTemplate">
        <ViewCell>
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="0.5*" />
                    <ColumnDefinition Width="0.2*" />
                    <ColumnDefinition Width="0.3*" />
                </Grid.ColumnDefinitions>
                <Label Text="{Binding Name}" TextColor="{StaticResource NormalTextColor}" FontAttributes="Bold" />
                <Label Grid.Column="1" Text="{Binding Age}" TextColor="{StaticResource NormalTextColor}" />
                <Label Grid.Column="2" Text="{Binding Location}" TextColor="{StaticResource NormalTextColor}" HorizontalTextAlignment="End" />
            </Grid>
        </ViewCell>
    </DataTemplate>
</ResourceDictionary>
```

これは、`ResourceDictionary`型のオブジェクトは、1 つのリソースを含む`DataTemplate`します。

インスタンスを作成できる`MyResourceDictionary`のペアの間に配置して`Resources`プロパティ要素タグをたとえばで、 `ContentPage`:

```xaml
<ContentPage ...>
    <ContentPage.Resources>
        <local:MyResourceDictionary />
    </ContentPage.Resources>
    ...
</ContentPage>
```

インスタンス`MyResourceDictionary`に設定されている、`Resources`のプロパティ、`ContentPage`オブジェクト。

ただし、このアプローチにはいくつかの制限:`Resources`のプロパティ、`ContentPage`参照のみ`ResourceDictionary`します。 などの他のオプションが必要なほとんどの場合、`ResourceDictionary`インスタンスとおそらく他のリソースもします。

このタスクでは、マージされたリソース ディクショナリが必要です。

## <a name="merged-resource-dictionaries"></a>結合されたリソース ディクショナリ

マージされたリソースディクショナリは、1つ以上の[`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary)オブジェクトを別の `ResourceDictionary`に結合します。

### <a name="merge-local-resource-dictionaries"></a>ローカルリソースディクショナリのマージ

ローカル[`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary)を別の `ResourceDictionary` にマージするには、 [`Source`](xref:Xamarin.Forms.ResourceDictionary.Source)プロパティに、リソースを含む XAML ファイルのファイル名を設定します。

```xaml
<ContentPage ...>
    <ContentPage.Resources>
        <!-- Add more resources here -->
        <ResourceDictionary Source="MyResourceDictionary.xaml" />
        <!-- Add more resources here -->
    </ContentPage.Resources>
    ...
</ContentPage>
```

この構文では、`MyResourceDictionary` クラスはインスタンス化されません。 代わりに、XAML ファイルを参照します。 このため、 [`Source`](xref:Xamarin.Forms.ResourceDictionary.Source)プロパティを設定するときに、分離コードファイル (**MyResourceDictionary.xaml.cs**) は必要なく、`x:Class` 属性は**myresourcedictionary .xaml**ファイルのルートタグから削除できます。 また、この方法を使用してリソースディクショナリをマージする場合、Xamarin. Forms は[`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary)を自動的にインスタンス化するため、外側の `ResourceDictionary` タグは必要ありません。

> [!IMPORTANT]
> [`Source`](xref:Xamarin.Forms.ResourceDictionary.Source)プロパティは、XAML からのみ設定できます。

### <a name="merge-resource-dictionaries-from-other-assemblies"></a>他のアセンブリからリソースディクショナリをマージする

[`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary)は、`ResourceDictionary`の[`MergedDictionaries`](xref:Xamarin.Forms.ResourceDictionary.MergedDictionaries)プロパティに追加することで、別の `ResourceDictionary` にマージすることもできます。 この手法では、リソースディクショナリが配置されているアセンブリに関係なく、リソースディクショナリをマージできます。

> [!IMPORTANT]
> [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary)クラスは、 [`MergedWith`](xref:Xamarin.Forms.ResourceDictionary.MergedWith)プロパティも定義します。 ただし、このプロパティは非推奨とされているため、使用できなくなりました。

次のコード例は、ページレベル[`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary)の[`MergedDictionaries`](xref:Xamarin.Forms.ResourceDictionary.MergedDictionaries)コレクションに追加される `MyResourceDictionary` を示しています。

```xaml
<ContentPage ...
             xmlns:local="clr-namespace:ResourceDictionaryDemo">
    <ContentPage.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <local:MyResourceDictionary />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </ContentPage.Resources>
    ...
</ContentPage>
```

次の例では、同じアセンブリに存在する `MyResourceDictionary`のインスタンスを[`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary)に追加しています。 さらに、他のアセンブリ、 [`MergedDictionaries`](xref:Xamarin.Forms.ResourceDictionary.MergedDictionaries)プロパティ要素タグ内の他の `ResourceDictionary` オブジェクト、およびこれらのタグの外部にある他のリソースからリソースディクショナリを追加することもできます。

```xaml
<ContentPage ...
             xmlns:local="clr-namespace:ResourceDictionaryDemo"
             xmlns:theme="clr-namespace:MyThemes;assembly=MyThemes">
    <ContentPage.Resources>
        <ResourceDictionary>
            <!-- Add more resources here -->
            <ResourceDictionary.MergedDictionaries>
                <!-- Add more resource dictionaries here -->
                <local:MyResourceDictionary />
                <theme:LightTheme />
                <!-- Add more resource dictionaries here -->
            </ResourceDictionary.MergedDictionaries>
            <!-- Add more resources here -->
        </ResourceDictionary>
    </ContentPage.Resources>
    ...
</ContentPage>
```

> [!IMPORTANT]
> [`ResourceDictionary`](xref:Xamarin.Forms.ResourceDictionary)には `MergedDictionaries` プロパティ要素タグが1つしか存在できませんが、必要な数の `ResourceDictionary` オブジェクトを含めることができます。

マージする際に[ `ResourceDictionary` ](xref:Xamarin.Forms.ResourceDictionary)リソースが同じ共有`x:Key`属性値は、Xamarin.Forms は、次のリソースの優先順位を使用します。

1. リソース ディクショナリのローカル リソース。
1. 使用してマージされたリソース ディクショナリに含まれるリソース、`MergedDictionaries`に登録されている逆の順序で、コレクション、`MergedDictionaries`プロパティ。

> [!NOTE]
> アプリケーションが複数含まれている場合に、負荷の高いタスク リソース ディクショナリを検索することができますサイズの大きいリソース ディクショナリ。 そのため、不要な検索を避けるため、アプリケーション内の各ページだけが、ページに適切なリソース ディクショナリを使用するを確認する必要があります。

## <a name="related-links"></a>関連リンク

- [リソース ディクショナリ (サンプル)](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/xaml-resourcedictionaries)
- [スタイル](~/xamarin-forms/user-interface/styles/index.md)
- [ResourceDictionary](xref:Xamarin.Forms.ResourceDictionary)

## <a name="related-video"></a>関連ビデオ

> [!Video https://channel9.msdn.com/Shows/XamarinShow/XamarinForms-101-Application-Resources/player]

[!include[](~/essentials/includes/xamarin-show-essentials.md)]
