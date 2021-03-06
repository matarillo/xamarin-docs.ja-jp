---
title: チュートリアル-Xamarin のバックグラウンドの場所
description: このドキュメントでは、backgrounded アプリケーションで位置情報を使用する方法についてのチュートリアルを提供します。 必要なセットアップ、ユーザーインターフェイス、およびアプリケーションの状態について説明します。
ms.prod: xamarin
ms.assetid: F8EEA0FD-5614-47FE-ADAC-80A5BCA6EB5F
ms.technology: xamarin-ios
author: davidortinau
ms.author: daortin
ms.date: 03/18/2017
ms.openlocfilehash: b357a6e1531c8f587c8fc0f86e2b56d6fecc2923
ms.sourcegitcommit: d0e6436edbf7c52d760027d5e0ccaba2531d9fef
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/25/2019
ms.locfileid: "75488323"
---
# <a name="walkthrough---background-location-in-xamarinios"></a>チュートリアル-Xamarin のバックグラウンドの場所

この例では、現在の場所に関する情報 (緯度、経度、およびその他のパラメーター) を画面に出力する iOS ロケーションアプリケーションを作成します。 このアプリケーションでは、アプリケーションがアクティブまたは Backgrounded の間に位置情報の更新を適切に実行する方法を示します。

このチュートリアルでは、バックグラウンドで必要なアプリケーションとしてのアプリの登録、アプリの backgrounded 時の UI 更新の中断、`WillEnterBackground` `WillEnterForeground` および `AppDelegate` メソッドの操作など、いくつかの重要なバックグラウンド処理の概念について説明します。

## <a name="application-set-up"></a>アプリケーションのセットアップ

1. 最初に、**単一ビューアプリケーション > 新しい iOS > アプリC#** を作成します ()。 その_場所_を呼び出し、IPad と iPhone の両方が選択されていることを確認します。

1. 場所アプリケーションは、iOS のバックグラウンドで必要なアプリケーションとして認定されます。 プロジェクトの**情報の plist**ファイルを編集して、アプリケーションを場所アプリケーションとして登録します。

    [ソリューションエクスプローラー] の下で、 **[ファイル]** をダブルクリックして開き、一覧の一番下までスクロールします。 **[バックグラウンドモードを有効にする]** チェックボックスと **[場所の更新]** チェックボックスの両方をオンにします。

    Visual Studio for Mac では、次のようになります。

    [![](location-walkthrough-images/image7.png "Place a check by both the Enable Background Modes and the Location Updates checkboxes")](location-walkthrough-images/image7.png#lightbox)

    Visual Studio では、次のキーと値のペアを追加して、手動で**情報**を更新する必要があります。

    ```xml
    <key>UIBackgroundModes</key>
    <array>
      <string>location</string>
    </array>
    ```

1. これで、アプリケーションが登録されたので、デバイスから場所データを取得できます。 IOS では、`CLLocationManager` クラスを使用して位置情報にアクセスし、場所の更新を提供するイベントを発生させることができます。

1. コードで、`LocationManager` という名前の新しいクラスを作成します。これにより、場所の更新をサブスクライブするための、さまざまな画面やコードのための単一の場所が提供されます。 `LocationManager` クラスで、`LocMgr`という `CLLocationManager` のインスタンスを作成します。

    ```csharp
    public class LocationManager
    {
        protected CLLocationManager locMgr;

        public LocationManager () {
            this.locMgr = new CLLocationManager();
            this.locMgr.PausesLocationUpdatesAutomatically = false;

            // iOS 8 has additional permissions requirements
            if (UIDevice.CurrentDevice.CheckSystemVersion (8, 0)) {
                locMgr.RequestAlwaysAuthorization (); // works in background
                //locMgr.RequestWhenInUseAuthorization (); // only in foreground
            }

            if (UIDevice.CurrentDevice.CheckSystemVersion (9, 0)) {
                locMgr.AllowsBackgroundLocationUpdates = true;
            }
        }

        public CLLocationManager LocMgr {
            get { return this.locMgr; }
        }
    }
    ```

    上記のコードでは、 [Cllocationmanager](xref:CoreLocation.CLLocationManager)クラスに対するいくつかのプロパティと権限を設定しています。

    - `PausesLocationUpdatesAutomatically` –システムが位置情報の更新を一時停止できるかどうかに応じて設定できるブール値です。 一部のデバイスでは、既定で `true`に設定されています。これにより、約15分後にデバイスがバックグラウンドの場所の更新プログラムの取得を停止する可能性があります。
    - `RequestAlwaysAuthorization`-このメソッドを渡す必要があります。これにより、アプリのユーザーに対して、バックグラウンドで場所へのアクセスを許可するオプションを与える必要があります。 アプリがフォアグラウンドにあるときにのみ場所へのアクセスを許可するオプションをユーザーに付与する場合は、`RequestWhenInUseAuthorization` を渡すこともできます。
    - `AllowsBackgroundLocationUpdates` –これは、iOS 9 で導入されたブール型のプロパティであり、中断されたときにアプリが位置情報の更新を受け取ることができるように設定できます。

    > [!IMPORTANT]
    > iOS 8 (およびそれ以降) では、ユーザーを承認要求の一部として表示するために、**情報 plist**ファイルのエントリも必要です。

1. アプリが必要とするアクセス許可の種類 (`NSLocationAlwaysUsageDescription`、`NSLocationWhenInUseUsageDescription`、`NSLocationAlwaysAndWhenInUseUsageDescription`) の情報を入力**します。** このキーは、場所データへのアクセスを要求するアラートにユーザーに表示される文字列と共に追加されます。

1. iOS 9 では、`AllowsBackgroundLocationUpdates` を使用する場合、`location`値を持つキー `UIBackgroundModes`**が含まれ**ている必要があります。 このチュートリアルの手順2を完了している場合は、既に情報 plist ファイルに含まれています。

1. `LocationManager` クラス内で、次のコードを使用して `StartLocationUpdates` というメソッドを作成します。 このコードは、`CLLocationManager`から場所の更新の受信を開始する方法を示しています。

    ```csharp
    if (CLLocationManager.LocationServicesEnabled) {
        //set the desired accuracy, in meters
        LocMgr.DesiredAccuracy = 1;
        LocMgr.LocationsUpdated += (object sender, CLLocationsUpdatedEventArgs e) =>
        {
            // fire our custom Location Updated event
            LocationUpdated (this, new LocationUpdatedEventArgs (e.Locations [e.Locations.Length - 1]));
        };
        LocMgr.StartUpdatingLocation();
    }
    ```

    この方法では、いくつかの重要な点があります。 まず、アプリケーションがデバイス上の場所データにアクセスできるかどうかを確認します。 このことを確認するには、`CLLocationManager`で `LocationServicesEnabled` を呼び出します。 ユーザーが場所情報へのアプリケーションのアクセスを拒否した場合、このメソッドは**false**を返します。

1. 次に、ロケーションマネージャーに更新頻度を通知します。 `CLLocationManager` には、更新の頻度など、場所データのフィルター処理や構成に関する多くのオプションが用意されています。 この例では、メーターによって場所が変更されるたびに、`DesiredAccuracy` を更新するように設定します。 場所の更新頻度とその他の設定の構成の詳細については、Apple のドキュメントの「 [Cllocationmanager クラスリファレンス](https://developer.apple.com/library/ios/#documentation/CoreLocation/Reference/CLLocationManager_Class/CLLocationManager/CLLocationManager.html)」を参照してください。

1. 最後に、`CLLocationManager` インスタンスで `StartUpdatingLocation` を呼び出します。 これにより、ロケーションマネージャーは、現在の場所で最初の修正を取得し、更新の送信を開始するように指示します。

ここまでで、ロケーションマネージャーが作成され、受信するデータの種類で構成され、初期の場所が決定されています。 ここで、コードで位置データをユーザーインターフェイスに表示する必要があります。 これは、引数として `CLLocation` を受け取るカスタムイベントで実行できます。

```csharp
// event for the location changing
public event EventHandler<LocationUpdatedEventArgs>LocationUpdated = delegate { };
```

次の手順では、`CLLocationManager`から位置情報の更新をサブスクライブし、新しい場所のデータが使用可能になったときにカスタムの `LocationUpdated` イベントを発生させて、その場所を引数として渡します。 これを行うには、新しいクラス**LocationUpdateEventArgs.cs**を作成します。 このコードはメインアプリケーション内でアクセスでき、イベントが発生したときにデバイスの場所を返します。

```csharp
public class LocationUpdatedEventArgs : EventArgs
{
    CLLocation location;

    public LocationUpdatedEventArgs(CLLocation location)
    {
       this.location = location;
    }

    public CLLocation Location
    {
       get { return location; }
    }
}
```

## <a name="user-interface"></a>ユーザー インターフェイス

1. IOS designer を使用して、場所の情報を表示する画面を作成します。 開始する**メインのストーリーボード**ファイルをダブルクリックします。

    ストーリーボードで、場所情報のプレースホルダーとして機能する複数のラベルを画面にドラッグします。 この例では、緯度、経度、標高、コース、および速度のラベルがあります。

    レイアウトは次のようになります。

    ![](location-walkthrough-images/image8.png "An example UI layout in the iOS Designer")

1. Solution Pad で、`ViewController.cs` ファイルをダブルクリックして編集し、LocationManager の新しいインスタンスを作成して、それに対して `StartLocationUpdates`を呼び出します。
  コードを次のように変更します。

    ```csharp
    #region Computed Properties
    public static bool UserInterfaceIdiomIsPhone {
        get { return UIDevice.CurrentDevice.UserInterfaceIdiom == UIUserInterfaceIdiom.Phone; }
    }

    public static LocationManager Manager { get; set;}
    #endregion

    #region Constructors
    public ViewController (IntPtr handle) : base (handle)
    {
    // As soon as the app is done launching, begin generating location updates in the location manager
        Manager = new LocationManager();
        Manager.StartLocationUpdates();
    }

    #endregion
    ```

    これにより、アプリケーションの起動時に位置情報の更新が開始されます。ただし、データは表示されません。

1. 場所の更新が受信されたので、場所情報で画面を更新します。 次のメソッドは、`LocationUpdated` イベントから場所を取得し、UI に表示します。

    ```csharp
    #region Public Methods
    public void HandleLocationChanged (object sender, LocationUpdatedEventArgs e)
    {
        // Handle foreground updates
        CLLocation location = e.Location;

        LblAltitude.Text = location.Altitude + " meters";
        LblLongitude.Text = location.Coordinate.Longitude.ToString ();
        LblLatitude.Text = location.Coordinate.Latitude.ToString ();
        LblCourse.Text = location.Course.ToString ();
        LblSpeed.Text = location.Speed.ToString ();

        Console.WriteLine ("foreground updated");
    }
    #endregion
    ```

引き続き、AppDelegate で `LocationUpdated` イベントをサブスクライブし、新しいメソッドを呼び出して UI を更新する必要があります。 `StartLocationUpdates` 呼び出しの直後に `ViewDidLoad,` 次のコードを追加します。

```csharp
public override void ViewDidLoad ()
{
    base.ViewDidLoad ();

    // It is better to handle this with notifications, so that the UI updates
    // resume when the application re-enters the foreground!
    Manager.LocationUpdated += HandleLocationChanged;

}
```

アプリケーションを実行すると、次のようになります。

[![](location-walkthrough-images/image5.png "An example app run")](location-walkthrough-images/image5.png#lightbox)

## <a name="handling-active-and-background-states"></a>アクティブ状態とバックグラウンド状態の処理

1. アプリケーションは、フォアグラウンドでアクティブなときに場所の更新を出力しています。 アプリがバックグラウンドに入ったときの動作を示すために、アプリケーションの状態の変更を追跡する `AppDelegate` メソッドをオーバーライドして、アプリケーションがフォアグラウンドとバックグラウンドの間を移行するときにコンソールに書き込むようにします。

    ```csharp
    public override void DidEnterBackground (UIApplication application)
    {
        Console.WriteLine ("App entering background state.");
    }

    public override void WillEnterForeground (UIApplication application)
    {
        Console.WriteLine ("App will enter foreground");
    }
    ```

    次のコードを `LocationManager` に追加して、更新された場所のデータをアプリケーション出力に継続的に出力し、場所情報がまだバックグラウンドで使用可能であることを確認します。

    ```csharp
    public class LocationManager
    {
        public LocationManager ()
        {
        ...
        LocationUpdated += PrintLocation;
        }
        ...

        //This will keep going in the background and the foreground
        public void PrintLocation (object sender, LocationUpdatedEventArgs e) {
        CLLocation location = e.Location;
        Console.WriteLine ("Altitude: " + location.Altitude + " meters");
        Console.WriteLine ("Longitude: " + location.Coordinate.Longitude);
        Console.WriteLine ("Latitude: " + location.Coordinate.Latitude);
        Console.WriteLine ("Course: " + location.Course);
        Console.WriteLine ("Speed: " + location.Speed);
        }
    }
    ```

1. コードにもう1つ問題があります。アプリが backgrounded されたときに UI を更新しようとすると、iOS によって終了されます。 アプリがバックグラウンドになると、コードは場所の更新の登録を解除し、UI の更新を停止する必要があります。

    iOS では、アプリが別のアプリケーション状態に移行しようとしているときに通知を提供します。 この場合、`ObserveDidEnterBackground` 通知をサブスクライブできます。

    次のコードスニペットは、通知を使用して、UI の更新をいつ停止するかをビューに表示させる方法を示しています。 これは `ViewDidLoad`にあります。

    ```csharp
    UIApplication.Notifications.ObserveDidEnterBackground ((sender, args) => {
        Manager.LocationUpdated -= HandleLocationChanged;
    });
    ```

    アプリが実行されている場合、出力は次のようになります。

    ![](location-walkthrough-images/image6.png "Example of the location output in the console")

1. アプリケーションは、フォアグラウンドで動作しているときに場所の更新を画面に出力し、バックグラウンドで動作している間、アプリケーションの出力ウィンドウにデータを出力し続けます。

未解決の問題は1つしか残っていません。アプリが最初に読み込まれたときに画面が UI 更新を開始しますが、アプリがフォアグラウンドに再入力されたタイミングを知る方法はありません。 Backgrounded アプリケーションがフォアグラウンドに戻ると、UI の更新は再開されません。

この問題を解決するには、別の通知内で UI 更新を開始する呼び出しを入れ子にします。これは、アプリケーションがアクティブ状態になったときに起動します。

```csharp
UIApplication.Notifications.ObserveDidBecomeActive ((sender, args) => {
  Manager.LocationUpdated += HandleLocationChanged;
});
```

これで、アプリケーションの初回起動時に UI が更新を開始し、アプリがフォアグラウンドに戻るたびに更新が再開されます。

このチュートリアルでは、画面とアプリケーションの両方の出力ウィンドウに位置データを出力する、適切に動作するバックグラウンド対応の iOS アプリケーションを構築しています。

## <a name="related-links"></a>関連リンク

- [場所 (パート 4) (サンプル)](https://docs.microsoft.com/samples/xamarin/ios-samples/location)
- [コアロケーションフレームワークリファレンス](https://developer.apple.com/library/ios/documentation/CoreLocation/Reference/CoreLocation_Framework/_index.html)
