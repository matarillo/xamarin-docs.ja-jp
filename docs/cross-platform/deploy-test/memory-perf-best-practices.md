---
title: "クロスプラットフォームのパフォーマンス"
description: "Xamarin プラットフォームでビルドされたアプリケーションのパフォーマンスを高めるための手法は多数あります。 これらの手法をすべて使用することで、CPU で実行される作業量や、アプリケーションで消費されるメモリ量を大幅に減らすことができます。 この記事では、これらの方法について説明します。"
ms.topic: article
ms.prod: xamarin
ms.assetid: 9ce61f18-22ac-4b93-91be-5b499677d661
ms.technology: xamarin-cross-platform
author: asb3993
ms.author: amburns
ms.date: 03/24/2017
ms.openlocfilehash: 56d868f64de009d01930ec34ee2cb436276006ef
ms.sourcegitcommit: 6cd40d190abe38edd50fc74331be15324a845a28
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/27/2018
---
# <a name="cross-platform-performance"></a>クロスプラットフォームのパフォーマンス

_Xamarin プラットフォームでビルドされたアプリケーションのパフォーマンスを高めるための手法は多数あります。これらの手法をすべて使用することで、CPU で実行される作業量や、アプリケーションで消費されるメモリ量を大幅に減らすことができます。この記事では、これらの方法について説明します。_

低いアプリケーション パフォーマンスは、さまざまな方法で示されます。 たとえば、アプリケーションが応答しない、スクロールが遅くなった、電池の寿命が減っている可能性がある、などです。 ただし、パフォーマンスを最適化するには、単に効率的なコードを実装するだけでは済みません。 アプリケーション パフォーマンスのユーザー エクスペリエンスも考慮する必要があります。 たとえば、操作の実行によって、ユーザーが他の操作を実行できない状況にならないようにすることで、ユーザー エクスペリエンスを改善できます。

Xamarin プラットフォームでビルドされたアプリケーションのパフォーマンスとユーザーの体感パフォーマンスを高めるための手法は多数あります。 Windows コモン コントロールには以下が含まれます。

- [プロファイラーを使用する](#profiler)
- [IDisposable のリソースを解放する](#idisposable)
- [イベントのサブスクリプションを解除する](#events)
- [弱い参照を使用して不変オブジェクトを回避する](#weakreferences)
- [オブジェクトの作成コストの発生を遅らせる](#lazy)
- [非同期操作を実装する](#async)
- [SGen ガベージ コレクターを使用する](#sgen)
- [アプリケーションのサイズを縮小する](#linker)
- [イメージ リソースを最適化する](#optimizeimages)
- [アプリケーションのアクティブ化期間を短くする](#activationperiod)
- [Web サービス通信を減らす](#webservicecommunication)

この無料の [Xamarin University のビデオ](https://university.xamarin.com/guestlectures/avoiding-common-pitfalls-in-xamarin-apps)にも Xamarin アプリの設計に役立つヒントが含まれています。

[ ![](memory-perf-best-practices-images/clancey-sml.png "一般的な落とし穴の回避に関する無料の Xamarin University ビデオ")](https://university.xamarin.com/guestlectures/avoiding-common-pitfalls-in-xamarin-apps)

<a name="profiler" />

## <a name="use-the-profiler"></a>プロファイラーを使用する

アプリケーションを開発する場合、プロファイリング後にコードの最適化を試みることが重要です。 プロファイリングは、パフォーマンスの問題を減らすのにコードの最適化が最も大きな効果がある場所を特定するための手法です。 プロファイラーは、アプリケーションのメモリ使用量を追跡し、アプリケーションでのメソッドの実行時間を記録します。 このデータは、最適化の絶好の機会を見つけられるように、アプリケーションの実行パスと、コードの実行コストをナビゲートする場合に役立ちます。

Xamarin Profiler では、アプリケーションでのパラメーターに関する問題の測定、評価、および検出が可能です。 Visual Studio for Mac または Visual Studio 内から Xamarin.iOS および Xamarin.Android アプリケーションのプロファイリングを行う場合に使用できます。 Xamarin Profiler の詳細については、「[Xamarin Profiler の概要](~/tools/profiler/index.md)」を参照してください。

次のベスト プラクティスは、アプリのプロファイリングを行う場合に推奨されます。

- シミュレーターはアプリケーションのパフォーマンスを損なう可能性があるため、シミュレーターではアプリケーションのプロファイリングは行わないでください。
- 1 つのデバイスでパフォーマンスを測定する際に、必ずしも他のデバイスのパフォーマンス特性が示されるわけではないため、さまざまなデバイスでプロファイリングを実行するのが理想的です。 ただし、少なくとも、性能が最も低いと思われるデバイスでプロファイリングを行う必要があります。
- 他のすべてのアプリケーションを終了し、他のアプリケーションではなく、プロファイリングを行うアプリケーションのあらゆる影響を測定するようにしてください。

<a name="idisposable" />

## <a name="release-idisposable-resources"></a>IDisposable のリソースを解放する

`IDisposable` インターフェイスは、リソースを解放するためのメカニズムを提供します。 リソースを明示的に解放するために実装する必要がある `Dispose` メソッドを提供します。 `IDisposable` はデストラクターではありません。以下の状況でのみ実装する必要があります。

- クラスがアンマネージ リソースを所有している場合。 インクルード ファイル、ストリーム、およびネットワーク接続の解放が必要な一般的なアンマネージ リソース。
- クラスがマネージ `IDisposable` リソースを所有している場合。

インスタンスが必要でなくなったときに、型コンシューマーは `IDisposable.Dispose` 実装を呼び出してリソースを解放できます。 これには、次の 2 つのアプローチがあります。

- `using` ステートメントで `IDisposable` オブジェクトをラップする。
- `try`/`finally` ブロックで `IDisposable.Dispose` 呼び出しをラップする。

### <a name="wrapping-the-idisposable-object-in-a-using-statement"></a>using ステートメントでの IDisposable オブジェクトのラップ

以下のコード例は、`using` ステートメントでの `IDisposable` オブジェクトのラップ方法を示しています。

```csharp
public void ReadText (string filename)
{
  ...
  string text;
  using (StreamReader reader = new StreamReader (filename)) {
    text = reader.ReadToEnd ();
  }
  ...
}
```

`StreamReader` クラスは `IDisposable` を実装し、`using` ステートメントは、スコープを外れる前に `StreamReader` オブジェクトで `StreamReader.Dispose` メソッドを呼び出す便利な構文を提供します。 `StreamReader` オブジェクトは、`using` ブロック内では読み取り専用です。再割り当てすることはできません。 また、コンパイラは `try`/`finally` ブロックの中間言語 (IL) を実装するため、例外が発生した場合でも、`using` ステートメントで確実に `Dispose` メソッドを呼び出すことができます。

### <a name="wrapping-the-call-to-idisposabledispose-in-a-tryfinally-block"></a>Try/Finally ブロックでの IDisposable.Dispose 呼び出しのラップ

以下のコード例は、`try`/`finally` ブロックでの `IDisposable.Dispose` 呼び出しのラップ方法を示しています。

```csharp
public void ReadText (string filename)
{
  ...
  string text;
  StreamReader reader = null;
  try {
    reader = new StreamReader (filename);
    text = reader.ReadToEnd ();
  } finally {
    if (reader != null) {
      reader.Dispose ();
    }
  }
  ...
}
```

`StreamReader` クラスは `IDisposable` を実装し、`finally` ブロックはリソースを解放するために `StreamReader.Dispose` メソッドを呼び出します。

詳細については、[IDisposable インターフェイス](https://developer.xamarin.com/api/type/System.IDisposable/)に関するページを参照してください。

<a name="events" />

## <a name="unsubscribe-from-events"></a>イベントのサブスクリプションを解除する

メモリ リークを防ぐため、サブスクライバー オブジェクトが破棄される前に、イベントのサブスクリプションを解除する必要があります。 イベントのサブスクリプションを解除するまで、パブリッシュ側オブジェクトでイベントのデリゲートは、サブスクライバーのイベント ハンドラーをカプセル化するデリゲートへの参照を保持しています。 パブリッシュ側オブジェクトがこの参照を保持している限り、ガベージ コレクションはサブスクライバー オブジェクトのメモリを再利用しません。

以下のコード例は、イベントのサブスクリプションの解除方法を示しています。

```csharp
public class Publisher
{
  public event EventHandler MyEvent;

  public void OnMyEventFires ()
  {
    if (MyEvent != null) {
      MyEvent (this, EventArgs.Empty);
    }
  }
}

public class Subscriber : IDisposable
{
  readonly Publisher publisher;

  public Subscriber (Publisher publish)
  {
    publisher = publish;
    publisher.MyEvent += OnMyEventFires;
  }

  void OnMyEventFires (object sender, EventArgs e)
  {
    Debug.WriteLine ("The publisher notified the subscriber of an event");
  }

  public void Dispose ()
  {
    publisher.MyEvent -= OnMyEventFires;
  }
}
```

`Subscriber` クラスは、その `Dispose` メソッドでイベントのサブスクリプションを解除します。

ラムダ式ではオブジェクトを参照して保持できるため、イベント ハンドラーとラムダ構文を使用する場合、参照循環も発生します。 したがって、匿名メソッドへの参照をフィールドに格納し、次のコード例のように、イベントのサブスクリプションの解除で使用できます。

```csharp
public class Subscriber : IDisposable
{
  readonly Publisher publisher;
  EventHandler handler;

  public Subscriber (Publisher publish)
  {
    publisher = publish;
    handler = (sender, e) => {
      Debug.WriteLine ("The publisher notified the subscriber of an event");
    };
    publisher.MyEvent += handler;
  }

  public void Dispose ()
  {
    publisher.MyEvent -= handler;
  }
}
```

`handler` フィールドは匿名メソッドへの参照を保持し、イベントのサブスクリプションとサブスクリプション解除に使用されます。

<a name="weakreferences" />

## <a name="use-weak-references-to-prevent-immortal-objects"></a>弱い参照を使用して不変オブジェクトを回避する

> [!NOTE]
> iOS 開発者は、アプリでメモリを効率的に使用できるように、[iOS での循環参照の回避](~/ios/deploy-test/performance.md#avoidcircularreferences)に関するドキュメントを確認する必要があります。

<a name="lazy" />

## <a name="delay-the-cost-of-creating-objects"></a>オブジェクトの作成コストの発生を遅らせる

遅延初期化を使用して、最初に使用されるまでオブジェクトの作成を遅らせることができます。 この手法は主に、パフォーマンスの改善、計算の回避、メモリ要件の縮小を目的として利用されます。


以下の 2 つのシナリオの場合は作成コストのかかるオブジェクトに対して遅延初期化を使用することを検討してください。

- アプリケーションでオブジェクトを使用しない可能性がある。
- オブジェクトが作成される前に、コストのかかる他の操作を完了する必要がある。

以下のコード例に示すように、遅延初期化の型を定義する場合は、`Lazy<T>` クラスを使用します。

```csharp
void ProcessData(bool dataRequired = false)
{
  Lazy<double> data = new Lazy<double>(() =>
  {
    return ParallelEnumerable.Range(0, 1000)
                 .Select(d => Compute(d))
                 .Aggregate((x, y) => x + y);
  });

  if (dataRequired)
  {
    if (data.Value > 90)
    {
      ...
    }
  }
}

double Compute(double x)
{
  ...
}
```

遅延初期化は、`Lazy<T>.Value` プロパティへの初回のアクセス時に発生します。 初回アクセス時にラップされた型が作成され、返されて、今後のアクセスのために保存されます。

遅延初期化の詳細については、「[限定的な初期化](https://msdn.microsoft.com/en-us/library/dd997286(v=vs.110).aspx)」を参照してください。

<a name="async" />

## <a name="implement-asynchronous-operations"></a>非同期操作を実装する

.NET は、非同期バージョンの多くの API を提供します。 同期 API とは異なり、非同期 API では、アクティブな実行スレッドによって、呼び出しスレッドが非常に長時間ブロックされることはありません。 したがって、UI スレッドから API を呼び出すときは、非同期 API (使用可能な場合) を使用します。 これにより、UI スレッドの非ブロック状態が保たれ、アプリケーションのユーザー エクスペリエンスの向上に役立ちます。

さらに、UI スレッドがブロックされないようにするために、長時間実行される操作はバックグラウンド スレッドで実行する必要があります。 .NET では `async` および `await` キーワードが提供されます。これにより、バックグラウンド スレッドで長時間実行される操作を行い、完了時に結果にアクセスする非同期コードを書き込むことができます。 ただし、長時間実行される操作を `await` キーワードで非同期に実行することはできますが、バックグラウンド スレッドでの操作の実行は保証されません。 代わりに、以下のコード例のように、長時間実行される操作を `Task.Run` に渡して、これを行うことができます。

```csharp
public class FaceDetection
{
  ...
  async void RecognizeFaceButtonClick(object sender, EventArgs e)
  {
    await Task.Run(() => RecognizeFace ());
    ...
  }

  async Task RecognizeFace()
  {
    ...
  }
}
```

`RecognizeFace` メソッドはバックグラウンド スレッドで実行され、`RecognizeFaceButtonClick` メソッドが `RecognizeFace` メソッドの完了を待ってから続行されます。

長時間実行される操作ではキャンセルをサポートする必要もあります。 たとえば、ユーザーがアプリケーション内を移動する場合、長時間実行される操作の続行は不要になる可能性があります。 キャンセルを実装するためのパターンは次のとおりです。

- `CancellationTokenSource` インスタンスを作成します。 このインスタンスはキャンセル通知を管理して送信します。
- `CancellationTokenSource.Token` プロパティ値を、キャンセル可能である必要がある各タスクに渡します。
- それぞれのタスクに対し、キャンセルに応答するメカニズムを提供します。
- キャンセル通知を提供する `CancellationTokenSource.Cancel` メソッドを呼び出します。

> [!IMPORTANT]
> `CancellationTokenSource` クラスは `IDisposable` インターフェイスを実装します。そのため、`CancellationTokenSource` インスタンスが終了したら `CancellationTokenSource.Dispose` メソッドを呼び出す必要があります。



詳細については、「[非同期サポートの概要](~/cross-platform/platform/async.md)」を参照してください。

<a name="sgen" />

## <a name="use-the-sgen-garbage-collector"></a>SGen ガベージ コレクターを使用する

使用されなくなったオブジェクトに割り当てられているメモリを再利用するために、C# などのマネージ言語ではガーベジ コレクションが使用されます。 Xamarin プラットフォームで使用される 2 つのガベージ コレクターは次のとおりです。

- [**SGen** ](http://www.mono-project.com/docs/advanced/garbage-collector/sgen/) – これは世代別ガベージ コレクターであり、Xamarin のプラットフォームの既定のガベージ コレクターです。
- [**Boehm** ](http://www.hboehm.info/gc/) – これは、保守的な、非世代別ガベージ コレクターです。 Classic API を使用する Xamarin.iOS アプリケーションで使用される既定のガベージ コレクターです。

SGen では、オブジェクトにスペースを割り当てる際に、次の 3 つのヒープのいずれかが利用されます。

-  **新世代** – 新しい小さなオブジェクトが割り当てられます。 新世代のスペースが不足すると、マイナー ガベージ コレクションが発生します。 ライブ オブジェクトはすべてメジャー ヒープに移動されます。
-  **メジャー ヒープ** – 長時間実行されるオブジェクトが保持されます。 メジャー ヒープに十分なメモリがない場合は、メジャー ガベージ コレクションが発生します。 メジャー ガベージ コレクションで十分なメモリを解放できない場合は、SGen がシステムにメモリの追加を要求します。
-  **ラージ オブジェクト スペース** – 8000 を超えるバイトが必要なオブジェクトが保持されます。 ラージ オブジェクトは新世代では開始されませんが、代わりにこのヒープで割り当てられます。

SGen の利点の 1 つは、マイナー ガベージ コレクションの実行にかかる時間が、最後のマイナー ガベージ コレクション以降に作成された新しいライブ オブジェクトの数に比例することです。 したがって、これらのマイナー ガベージ コレクションの実行にかかる時間がメジャー ガベージ コレクションより短くなるため、アプリケーションのパフォーマンスへのガベージ コレクションの影響が軽減します。 メジャー ガベージ コレクションは引き続き発生しますが、頻度は低くなります。

### <a name="reducing-pressure-on-the-garbage-collector"></a>ガベージ コレクターの負荷の軽減

SGen がガベージ コレクションを開始すると、メモリの再利用中は、アプリケーションのスレッドが停止します。 メモリが再利用されている間、アプリケーションはしばらく一時停止するか、UI の途切れが生じる可能性があります。 この一時停止がどう認識されるかは、次の 2 つの要因によって決まります。

1. **頻度** – ガベージ コレクションの発生頻度。 コレクション間でのメモリの割り当て量が多いほど、ガベージ コレクションの頻度が増えます。
1. **期間** – 個々のガベージ コレクションにかかる時間。 これは、収集されるライブ オブジェクトの数にほぼ比例します。

つまり、多くのオブジェクトが割り当てられても、保持されなければ、多数の短いガベージ コレクションが発生します。 逆に、新しいオブジェクトが徐々に割り当てられ、保持されれば、少数の長いガベージ コレクションが発生します。

ガベージ コレクターの負荷を減らすには、次のガイドラインに従います。

- オブジェクト プールを使用して、短いループでのガベージ コレクションを回避します。 これは特に、事前に大部分のオブジェクトを作成する必要がある、ゲームに適しています。
- ストリーム、ネットワーク接続、大量のメモリ、およびファイルなどのリソースを必要でなくなった時点で明示的に解放します。 詳細については、「[IDisposable のリソースを解放する](#idisposable)」を参照してください。
- オブジェクトを収集できるように、不要になった時点でイベント ハンドラーの登録を解除します。 詳細については、「[イベントのサブスクリプションを解除する](#events)」を参照してください。

<a name="linker" />

## <a name="reduce-the-size-of-the-application"></a>アプリケーションのサイズを縮小する

アプリケーション実行可能ファイルのサイズの取得元を把握するために、各プラットフォームでのコンパイル処理を理解することが重要です。

- iOS アプリケーションは、ARM アセンブリ言語に AOT (Ahead-of-Time) コンパイルされます。 .NET Framework が組み込まれ、その未使用のクラスは、適切なリンカー オプションが有効になっている場合にのみ削除されます。
- Android アプリケーションは中間言語 (IL) にコンパイルされ、MonoVM と Just-In-Time (JIT) コンパイルでパッケージ化されます。 未使用のフレームワーク クラスは、適切なリンカー オプションが有効になっている場合にのみ削除されます。
- Windows Phone アプリケーションは IL にコンパイルされ、組み込みラインタイムによって実行されます。

さらに、アプリケーションがジェネリックを広く利用する場合、ネイティブにコンパイルされたジェネリックを含む可能性があるため、最終的な実行可能ファイルのサイズはさらに大きくなります。

アプリケーションのサイズを容易に縮小できるように、Xamarin プラットフォームにはビルド ツールの一部としてリンカーが含まれています。 既定では、リンカーは無効になっているため、アプリケーションのプロジェクト オプションで有効にする必要があります。 ビルド時に、アプリケーションで実際に使用される型、およびメンバーを特定するために、アプリケーションのスタティック分析を実行します。 その後、アプリケーションから未使用の型とメソッドを削除します。

次のスクリーンショットは、Xamarin.iOS プロジェクト用の Visual Studio for Mac のリンカー オプションを示しています。

![](memory-perf-best-practices-images/linker-options-ios.png)

次のスクリーンショットは、Xamarin.Android プロジェクト用の Visual Studio for Mac のリンカー オプションを示しています。

![](memory-perf-best-practices-images/linker-options-droid.png)

リンカーでは、その動作を制御するための次の 3 つの設定が提供されます。

-  **リンクしない** – 未使用の型とメソッドはリンカーによって削除されます。 パフォーマンス上の理由から、デバッグ ビルドではこれが既定の設定になります。
-  **フレームワーク SDK のみをリンクする/SDK アセンブリのみをリンクする** – この設定では、Xamarin に付属しているこれらのアセンブリのサイズのみが縮小されます。 ユーザー コードには影響しません。
-  **すべてのアセンブリをリンクする** – これは、SDK アセンブリとユーザー コードをターゲットとする、よりアグレッシブな最適化です。 バインディングの場合、未使用のバッキング フィールドが削除され、各インスタンス (またはバインド オブジェクト) が軽くなるため、メモリ消費量が少なくなります。

予期しない方法でアプリケーションが壊れる可能性がある場合に、*[すべてのアセンブリをリンクする]* を使用する必要があります。 リンカーで実行されるスタティック分析では、必要なコードの一部が正しく認識されない場合があり、その結果、コンパイルされたアプリケーションからコードが過剰に削除されます。 アプリケーションがクラッシュした場合の実行時にのみこのような状態になります。 そのため、リンカーの動作を変更した後で、アプリケーションを十分テストすることが重要です。

テストを行った際に、リンカーがクラスまたはメソッドを正しく削除しなかったことがわかった場合は、以下の属性のいずれかを使用して、静的に参照されていないが、アプリケーションでは必要な型またはメソッドにマークを付けることができます。

-  `Xamarin.iOS.Foundation.PreserveAttribute` – この属性は、Xamarin.iOS プロジェクト用です。
-  `Android.Runtime.PreserveAttribute` – この属性は、Xamarin.Android プロジェクト用です。

たとえば、動的にインスタンス化されている型の既定のコンストラクターを保持する必要がある場合があります。 また、XML シリアル化を使用するには、型のプロパティを維持する必要がある場合があります。

詳細については、[iOS のリンカー](~/ios/deploy-test/linker.md)と [Android のリンカー](~/android/deploy-test/linker.md)に関するページを参照してください。

### <a name="additional-size-reduction-techniques"></a>その他のサイズ縮小の手法

モバイル デバイスに電源を供給するさまざまな CPU アーキテクチャがあります。 そのため、Xamarin.iOS および Xamarin.Android では *FAT バイナリ*が生成されます。これには、各 CPU アーキテクチャのコンパイル済みバージョンのアプリケーションが含まれます。 これにより、CPU アーキテクチャに関係なく、デバイスでモバイル アプリケーションを確実に実行することができます。

次の手順を使用して、アプリケーションの実行可能ファイルのサイズをさらに縮小することができます。

- リリース ビルドが生成されていることを確認します。
- FAT バイナリが生成されないように、アプリケーションがビルドされるアーキテクチャの数を減らします。
- より最適化された実行可能ファイルを生成するために、LLVM コンパイラが使用されていることを確認します。
- アプリケーションのマネージ コード サイズを減らします。 これは、すべてのアセンブリでリンカーを有効にすることで行うことができます (iOS プロジェクトの場合は *[すべてリンク]*、Android プロジェクトの場合は *[すべてのアセンブリをリンクする]*)。

Android アプリは、ABI ("アーキテクチャ") ごとに別の APK に分割することもできます。
詳細については、このブログの投稿「[How To Keep Your Android App Size Down](http://motzcod.es/post/112072508362/how-to-keep-your-android-app-size-down)」 (Android アプリのサイズを小さくしておく方法) を参照してください。

<a name="optimizeimages" />

## <a name="optimize-image-resources"></a>画像リソースを最適化する

アプリケーションが使用するリソースのうち最もコストが高いものとして画像があります。多くの場合、画像は高解像度でキャプチャされます。 画像は細かい部分まで鮮明になりますが、そのような画像を表示するアプリケーションでは通常、画像をデコードするためにより多くの CPU を使用する必要があり、また、デコードされた画像を格納するためにより多くのメモリが必要になります。 表示サイズを小さくするためにスケールダウンする場合、メモリ内の高解像度画像をデコードするのは無駄です。 代わりに、予測された表示サイズに近い、格納された画像の多重解像度バージョンを作成して、CPU 使用量とメモリの占有領域を減らします。 たとえば、リスト ビューに表示される画像は、全画面で表示される画像よりも解像度が低くなる可能性が最も高くなります。 さらに、メモリへの影響を最小限に抑えて効率的に画像を表示するために、高解像度の画像のスケールダウン バージョンを読み込むことができます。 詳細については、「[Load Large Bitmaps Efficiently](https://developer.xamarin.com/recipes/android/resources/general/load_large_bitmaps_efficiently/)」 (大きなビットマップを効率的に読み込む) を参照してください。

画像の解像度に関係なく、画像リソースを表示すると、アプリのメモリの占有領域が大幅に増える場合があります。 そのため、必要な場合にのみ作成し、アプリケーションで不要になったらすぐに解放する必要があります。

<a name="activationperiod" />

## <a name="reduce-the-application-activation-period"></a>アプリケーションのアクティブ化期間を短くする

すべてのアプリケーションに*アクティブ化期間*があります。これは、アプリケーションが開始されてから、使用できるようになるまでの期間です。 このアクティブ化期間に、ユーザーにアプリケーションの第一印象を与えることになります。したがって、ユーザーにアプリケーションに対する好意的な第一印象を与えるためには、アクティブ化期間を減らし、ユーザーの認識を和らげることが重要です。

アプリケーションで初期 UI が表示される前に、スプラッシュ スクリーンを提供し、アプリケーションを開始していることをユーザーに示す必要があります。 アプリケーションで初期 UI をすぐに表示できない場合は、アプリケーションがハングしていないことを確認させるために、スプラッシュ スクリーンを使用して、アクティブ化期間での進行状況をユーザーに知らせる必要があります。 この確認は進行状況バー、または同様のコントロールで行うことができます。

アクティブ化期間中に、アプリケーションはアクティブ化ロジックを実行します。これには、多くの場合、リソースの読み込みと処理が含まれます。 リモートで取得されるのではなく、アプリ内に必要なリソースがパッケージ化されるようにすることで、アクティブ化期間を減らすことができます。 たとえば、ある状況では、アクティブ化期間中にローカルに格納されたプレースホルダー データを読み込むことが適切な場合があります。 その後、初期 UI が表示された時点で、ユーザーはアプリと対話でき、プレースホルダー データをリモート ソースから段階的に置き換えることができます。 さらに、アプリのアクティブ化ロジックでは、ユーザーにアプリケーションの使用を開始させるために必要な作業のみを実行する必要があります。 アセンブリは初回使用時に読み込まれるため、これは追加のアセンブリの読み込みを遅らせる場合に役立ちます。

<a name="webservicecommunication" />

## <a name="reduce-web-service-communication"></a>Web サービス通信を減らす

アプリケーションから Web サービスに接続すると、アプリケーションのパフォーマンスに影響を与える可能性があります。 たとえば、ネットワーク帯域幅の使用量が増えると、デバイスのバッテリ使用量が増えます。 さらに、ユーザーは、帯域幅に制限のある環境でアプリケーションを使用する可能性があります。 したがって、アプリケーションと Web サービス間の帯域幅使用率を制限するのが賢明です。

アプリケーションの帯域幅使用率を減らす 1 つの手法は、ネットワーク経由で転送する前にデータを圧縮することです。 ただし、圧縮プロセスにより CPU 使用量が増えると、バッテリ使用量も増えます。 そのため、ネットワーク経由で圧縮データを移動するかどうかを判断する前に、このトレードオフを慎重に評価する必要があります。

考慮すべきもう 1 つの問題は、アプリケーションと Web サービス間を移動するデータの形式です。 主な 2 つの形式は、拡張マークアップ言語 (XML) と JavaScript Object Notation (JSON) です。 XML は、多数の書式設定文字を含むため、比較的大きなデータ ペイロードを生成するテキスト ベースのデータ交換形式です。 JSON は、データの送受信時に帯域幅要件を減らせる、コンパクトなデータ ペイロードを生成するテキスト ベースのデータ交換形式です。 そのため、多くの場合、モバイル アプリケーションでは JSON 形式が推奨されます。

アプリケーションと Web サービス間でデータを転送する場合は、データ転送オブジェクト (DTO) を使用することをお勧めします。 DTO には、ネットワーク経由で転送するためのデータ セットが含まれています。 DTO を利用することで、1 つのリモート呼び出しでより多くのデータを送信でき、アプリケーションによるリモート呼び出しの数を減らすことができます。 一般に、大きなデータ ペイロードを渡すリモート呼び出しには、小さなデータ ペイロードのみを渡す呼び出しと同様の時間がかかります。

Web サービスから取得されたデータはローカルでキャッシュする必要があります。Web サービスから繰り返し取得されるのではなく、そのキャッシュされたデータが利用されます。 ただし、この手法を採用する場合は、Web サービスで変更されたときにローカル キャッシュのデータを更新するために適切なキャッシュ戦略を実装する必要もあります。

## <a name="summary"></a>まとめ

この記事では、Xamarin プラットフォームを使用してビルドされたアプリケーションのパフォーマンスを高めるための手法について説明しました。 これらの手法をすべて使用することで、CPU で実行される作業量や、アプリケーションで消費されるメモリ量を大幅に減らすことができます。

## <a name="related-links"></a>関連リンク

- [Xamarin.iOS のパフォーマンス](~/ios/deploy-test/performance.md)
- [Xamarin.Android パフォーマンス](~/android/deploy-test/performance.md)
- [Xamarin Profiler の概要](~/tools/profiler/index.md)
- [Xamarin.Forms のパフォーマンス](~/xamarin-forms/deploy-test/performance.md)
- [非同期サポートの概要](~/cross-platform/platform/async.md)
- [IDisposable](https://developer.xamarin.com/api/type/System.IDisposable/)
- [Xamarin アプリのよくある落とし穴の回避 (動画)](https://university.xamarin.com/guestlectures/avoiding-common-pitfalls-in-xamarin-apps)