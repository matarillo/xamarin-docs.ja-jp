---
title: Xamarin. iOS の CloudKit
description: このドキュメントでは、Xamarin の CloudKit を使用する方法について説明します。 ここでは、CloudKit の概要を説明し、cloudkit の便利な API、スケーラビリティ、ユーザーアカウント、および開発環境と運用環境を有効にする方法について説明します。
ms.prod: xamarin
ms.assetid: 66B207F2-FAA0-4551-B43B-3DB9F620C397
ms.technology: xamarin-ios
author: davidortinau
ms.author: daortin
ms.date: 05/11/2016
ms.openlocfilehash: 29ccb919f68a45212bff3b66b4bc3fbdebd24faf
ms.sourcegitcommit: bad1ab3f78d7f94d48511666626b54f8ba155689
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/04/2020
ms.locfileid: "75663461"
---
# <a name="cloudkit-in-xamarinios"></a>Xamarin. iOS の CloudKit

CloudKit フレームワークを使用すると、iCloud にアクセスするアプリケーションの開発が効率化されます。 これには、アプリケーションのデータと資産の権限の取得だけでなく、アプリケーションの情報を安全に格納できることも含まれます。 このキットは、個人情報を共有せずに iCloud Id を使用してアプリケーションにアクセスできるようにすることで、ユーザーに対して匿名のレイヤーを提供します。

開発者はクライアント側のアプリケーションに集中でき、iCloud によってサーバー側のアプリケーションロジックを作成する必要がなくなります。 CloudKit は、認証、プライベートおよびパブリックデータベース、構造化データおよびアセットストレージサービスを提供します。

> [!IMPORTANT]
> Apple からは、開発者が欧州連合の一般データ保護規則 (GDPR) を適切に処理するための[ツールが提供](https://developer.apple.com/support/allowing-users-to-manage-data/)されています。

## <a name="requirements"></a>要件

この記事に記載されている手順を完了するには、次のものが必要です。

- **Xcode と IOS SDK** – Apple の Xcode と Ios 8 api は、開発者のコンピューターにインストールして構成する必要があります。
- **Visual Studio for Mac** –ユーザーデバイスで Visual Studio for Mac の最新バージョンをインストールして構成する必要があります。
- **ios 8 デバイス**-テスト用に ios 8 の最新バージョンを実行している ios デバイス。

## <a name="what-is-cloudkit"></a>CloudKit とは

CloudKit は、開発者が iCloud サーバーにアクセスできるようにするための手段です。 ICloud ドライブと iCloud フォトライブラリの両方の基盤を提供します。 CloudKit は、macOS デバイスと iOS デバイスの両方でサポートされています。

[macOS と iOS の両方のデバイスで CloudKit がどのようにサポートされるかを ![します。](intro-to-cloudkit-images/image1.png)](intro-to-cloudkit-images/image1.png#lightbox)

CloudKit は iCloud アカウントインフラストラクチャを使用します。 デバイスで iCloud アカウントにログインしているユーザーがいる場合、CloudKit はその ID を使用してユーザーを識別します。 使用可能なアカウントがない場合は、制限付きの読み取り専用アクセスが提供されます。

CloudKit は、パブリックデータベースとプライベートデータベースの両方の概念をサポートしています。 パブリックデータベースは、ユーザーがアクセスできるすべてのデータの "スープ" を提供します。 プライベートデータベースは、特定のユーザーにバインドされたプライベートデータを格納することを目的としています。

CloudKit では、構造化データと一括データの両方がサポートされています。 大きなファイル転送をシームレスに処理できます。 CloudKit は、バックグラウンドで iCloud サーバーとの間で大きなファイルを効率的に転送し、開発者が他のタスクに専念するのを防ぐことができます。

> [!NOTE]
> CloudKit は_トランスポートテクノロジ_であることに注意する必要があります。 永続化は提供されません。これにより、アプリケーションはサーバーから情報を効率的に送受信することができます。

このドキュメントの執筆時点で、Apple は最初に、帯域幅とストレージ容量の両方に上限がある CloudKit を無料で提供しています。 大規模なユーザーベースを持つ大規模なプロジェクトまたはアプリケーションの場合、Apple では手頃な価格スケールが提供されることがヒントになりました。

## <a name="enabling-cloudkit-in-a-xamarin-application"></a>Xamarin アプリケーションでの CloudKit の有効化

Xamarin アプリケーションで CloudKit フレームワークを利用するには、「[機能の使用](~/ios/deploy-test/provisioning/capabilities/icloud-capabilities.md)」および「[権利の](~/ios/deploy-test/provisioning/entitlements.md)使用」ガイドの説明に従って、アプリケーションが正しくプロビジョニングされている必要があります。

CloudKit にアクセスするには、**有効**にする必要があります iCloud ファイル、**キー値のストレージ**、および**cloudkit**のアクセス許可。

### <a name="sample-app"></a>サンプル アプリ

[CloudKitAtlas サンプル](https://docs.microsoft.com/samples/xamarin/ios-samples/ios8-cloudkitatlas)は、Xamarin で cloudkit を使用する方法を示しています。 次の手順では、サンプルを構成する方法を示します。 CloudKit だけではなく、追加の設定が必要になります。

1. Visual Studio for Mac または Visual Studio でプロジェクトを開きます。
2. **ソリューションエクスプローラー**で、**情報 plist**ファイルを開き、**バンドル識別子**が、プロビジョニングセットアップの一部として作成された**アプリ ID**で定義されているものと一致していることを確認します。
3. **情報 plist**ファイルの一番下までスクロールし、 **[有効なバックグラウンドモード]** 、 **[場所の更新]** 、および **[リモート通知]** を選択します。
4. ソリューションで iOS プロジェクトを右クリックし、 **[オプション]** を選択します。
5. 選択**iOS バンドル署名**を選択、**開発者 Identity**と**プロビジョニング プロファイル**上記で作成しました。
6. **資格**情報の plist に、 **iCloud**、**キー値のストレージ**、および**cloudkit**の有効化が含まれていることを確認します。
7. アプリケーションの**ユビキタスコンテナー**が存在することを確認します。 例 : `iCloud.com.your-company.CloudKitAtlas`
8. 変更内容をファイルに保存します。

これらの設定が適用されたので、サンプルアプリは、CloudKit フレームワーク Api に加えて、バックグラウンド、場所、および通知サービスにもアクセスできるようになりました。

## <a name="cloudkit-api-overview"></a>CloudKit API の概要

Xamarin iOS アプリケーションで CloudKit を実装する前に、この記事では、CloudKit フレームワークの基礎について説明します。これには、次のトピックが含まれます。

1. **コンテナー** -iCloud 通信の分離されたサイロ。
2. **データベース**–パブリックとプライベートはアプリケーションで使用できます。
3. **レコード**– cloudkit との間で構造化データを移動するメカニズム。
4. **レコードゾーン**–レコードのグループです。
5. **レコード識別子**: は完全に正規化され、レコードの特定の場所を表します。
6. **Reference** –特定のデータベース内の関連するレコード間の親子リレーションシップを提供します。
7. Asset **–大**規模な非構造化データのファイルを iCloud にアップロードし、特定のレコードに関連付けることができます。

### <a name="containers"></a>コンテナー

IOS デバイスで実行されている特定のアプリケーションは、そのデバイス上の他のアプリケーションやサービスと同時に実行されます。 クライアントデバイスでは、何らかの方法でアプリケーションが孤立またはサンドボックス化されます。 場合によっては、これがリテラルサンドボックスであり、それ以外の場合は、アプリケーションが専用のメモリ領域で実行されることもあります。

クライアントアプリケーションを使用して他のクライアントから分離して実行するという概念は非常に強力であり、次のような利点があります。

1. **セキュリティ**–1つのアプリケーションが他のクライアントアプリや OS 自体と干渉することはできません。
1. **安定性**–クライアントアプリケーションがクラッシュした場合、OS の他のアプリを実行することはできません。
1. **プライバシー** –各クライアントアプリケーションは、デバイス内に格納されている個人情報へのアクセスを制限されています。

CloudKit は上記と同じ利点を提供するように設計されており、クラウドベースの情報を操作するために適用されます。

 [![](intro-to-cloudkit-images/image31.png "CloudKit apps communicate using containers")](intro-to-cloudkit-images/image31.png#lightbox)

デバイス上で実行されているアプリケーションと同じように、アプリケーションは iCloud との通信を行います。 これらのさまざまな通信サイロは、コンテナーと呼ばれます。

コンテナーは、CloudKit フレームワークで `CKContainer` クラスを介して公開されます。 既定では、1つのアプリケーションが1つのコンテナーと通信し、このコンテナーはそのアプリケーションのデータを分離します。 これは、複数のアプリケーションが同じ iCloud アカウントに情報を格納できることを意味しますが、この情報は混在しません。

ICloud データのコンテナー化により、CloudKit はユーザー情報をカプセル化することもできます。 この方法では、ユーザーのプライバシーとセキュリティを保護しながら、iCloud アカウントと、に格納されているユーザー情報へのアクセスが制限されます。

コンテナーは、WWDR ポータルを使用して、アプリケーションの開発者によって完全に管理されます。 コンテナーの名前空間は、すべての Apple 開発者にとってグローバルであるため、コンテナーは、特定の開発者のアプリケーションだけでなく、すべての Apple 開発者およびアプリケーションに対して一意である必要があります。

Apple では、アプリケーションコンテナーの名前空間を作成するときに、逆引き DNS 表記を使用することを提案しています。 例 : `iCloud.com.company-name.application-name`

コンテナーは、既定では、特定のアプリケーションに対して1対1でバインドされますが、アプリケーション間で共有できます。 そのため、複数のアプリケーションで1つのコンテナーを調整できます。 1つのアプリケーションが複数のコンテナーと通信することもできます。

### <a name="databases"></a>データベース

CloudKit の主な機能の1つは、アプリケーションのデータモデルと、iCloud サーバーまでのレプリケーションを実行することです。 一部の情報は、それを作成したユーザーを対象としています。その他の情報は、ユーザーが一般に使用するために作成できるパブリックデータ (レストランのレビューなど)、または開発者がアプリケーションに発行した情報である可能性があります。 どちらの場合も、対象ユーザーは単なる1人のユーザーではなく、ユーザーのコミュニティです。

 [![](intro-to-cloudkit-images/image32.png "CloudKit Container Diagram")](intro-to-cloudkit-images/image32.png#lightbox)

コンテナー内では、まずパブリックデータベースがあります。 ここで、すべてのパブリック情報が存在し、共同 mingles ます。 また、アプリケーションのユーザーごとに個別のプライベートデータベースがいくつかあります。

IOS デバイスで実行されている場合、アプリケーションは、現在ログオンしている iCloud ユーザーの情報にのみアクセスできます。 そのため、コンテナーのアプリケーションのビューは次のようになります。

 [![](intro-to-cloudkit-images/image33.png "The applications view of the container")](intro-to-cloudkit-images/image33.png#lightbox)

パブリックデータベースと、現在ログオンしている iCloud ユーザーに関連付けられているプライベートデータベースのみが表示されます。

データベースは、CloudKit フレームワークで `CKDatabase` クラスを介して公開されます。 すべてのアプリケーションは、パブリックデータベースとプライベートデータベースの2つのデータベースにアクセスできます。

コンテナーは、CloudKit への最初のエントリポイントです。 次のコードを使用して、アプリケーションの既定のコンテナーからパブリックおよびプライベートデータベースにアクセスできます。

```csharp
using CloudKit;
//...

public CKDatabase PublicDatabase { get; set; }
public CKDatabase PrivateDatabase { get; set; }
//...

// Get the default public and private databases for
// the application
PublicDatabase = CKContainer.DefaultContainer.PublicCloudDatabase;
PrivateDatabase = CKContainer.DefaultContainer.PrivateCloudDatabase;
```

データベースの種類の違いは次のとおりです。

||パブリック データ|プライベートデータベース|
|---|--- |--- |
|**データ型**|共有データ|現在のユーザーのデータ|
|**[クォータ]**|開発者のクォータについて考慮されます。|ユーザーのクォータについて考慮されます。|
|**既定のアクセス許可**|世界での読み取り可能|ユーザーが読み取り可能|
|**アクセス許可の編集**|レコードクラスレベルを使用した iCloud ダッシュボードロール|N/A|

### <a name="records"></a>レコード

コンテナーはデータベースを保持し、データベース内にはレコードです。 レコードは、CloudKit との間で構造化データを移動するためのメカニズムです。

 [![](intro-to-cloudkit-images/image34.png "Containers hold databases, and inside databases are records")](intro-to-cloudkit-images/image34.png#lightbox)

レコードは、キーと値のペアをラップする `CKRecord` クラスを使用して、CloudKit フレームワークで公開されます。 アプリケーション内のオブジェクトのインスタンスは、CloudKit の `CKRecord` に相当します。 また、各 `CKRecord` は、オブジェクトのクラスに相当するレコード型を所有しています。

レコードにはジャストインタイムスキーマがあるため、処理のために渡される前に、データが CloudKit に記述されます。 その時点から、CloudKit は情報を解釈し、レコードの格納と取得の物流を処理します。

`CKRecord` クラスは、さまざまなメタデータもサポートします。 たとえば、レコードには、作成された日時とそれを作成したユーザーに関する情報が含まれます。 レコードには、最後に変更された日時とそれを変更したユーザーに関する情報も含まれます。

レコードには、変更タグの概念が含まれています。 これは、特定のレコードの以前のバージョンのリビジョンです。 Change タグは、クライアントとサーバーの特定のレコードのバージョンが同じであるかどうかを確認するための軽量な方法として使用されます。

前述のように、キーと値のペアをラップする `CKRecords`、次の種類のデータをレコードに格納できます。

1. `NSString`
1. `NSNumber`
1. `NSData`
1. `NSDate`
1. `CLLocation`
1. `CKReferences`
1. `CKAssets`

1つの値型に加えて、レコードには上記の型のいずれかの同種の配列を含めることができます。

次のコードを使用すると、新しいレコードを作成してデータベースに格納できます。

```csharp
using CloudKit;
//...

private const string ReferenceItemRecordName = "ReferenceItems";
//...

var newRecord = new CKRecord (ReferenceItemRecordName);
newRecord ["name"] = (NSString)nameTextField.Text;
await CloudManager.SaveAsync (newRecord);
```

### <a name="record-zones"></a>レコードゾーン

レコードは、特定のデータベース内には存在しません。レコードのグループは、レコードゾーン内にまとめられます。 レコードゾーンは、従来のリレーショナルデータベースのテーブルと考えることができます。

 [![](intro-to-cloudkit-images/image35.png "Groups of records exist together inside a Record Zone")](intro-to-cloudkit-images/image35.png#lightbox)

特定のレコードゾーン内に複数のレコードを指定し、特定のデータベース内に複数のレコードゾーンを配置することができます。 各データベースには、既定のレコードゾーンが含まれます。

 [![](intro-to-cloudkit-images/image36.png "Every database contains a Default Record Zone and Custom Zone")](intro-to-cloudkit-images/image36.png#lightbox)

ここで、レコードは既定で格納されます。 さらに、カスタムレコードゾーンを作成することもできます。 レコードゾーンは、アトミックコミットと Change Tracking が実行される基本粒度を表します。

## <a name="record-identifiers"></a>レコード識別子

レコード識別子は、クライアントが指定したレコード名と、レコードが存在するゾーンの両方を含むタプルとして表されます。 レコード識別子には次の特性があります。

- これらはクライアントアプリケーションによって作成されます。
- これらは完全に正規化され、レコードの特定の場所を表します。
- 外部データベース内のレコードの一意の ID をレコード名に割り当てることにより、CloudKit 内に格納されていないローカルデータベースをブリッジするために使用できます。

開発者は、新しいレコードを作成するときに、レコード識別子を渡すことを選択できます。 レコード識別子が指定されていない場合は、UUID が自動的に作成され、レコードに割り当てられます。

開発者は、新しいレコード識別子を作成するときに、各レコードが属するレコードゾーンを指定できます。 何も指定されていない場合は、既定のレコードゾーンが使用されます。

レコード識別子は、CloudKit フレームワークで `CKRecordID` クラスを介して公開されます。 次のコードを使用して、新しいレコード識別子を作成できます。

```csharp
var recordID =  new CKRecordID("My Record");
```

### <a name="references"></a>参照

参照は、特定のデータベース内の関連するレコード間のリレーションシップを提供します。

 [![](intro-to-cloudkit-images/image37.png "References provide relationships between related Records within a given Database")](intro-to-cloudkit-images/image37.png#lightbox)

上の例では、親子が親レコードの子レコードであるように、親が子を所有しています。 リレーションシップは、子レコードから親レコードに移動し、*後方参照*と呼ばれます。

参照は、CloudKit フレームワークで `CKReference` クラスを介して公開されます。 これは、iCloud サーバーがレコード間の関係を理解できるようにするための手段です。

参照は、連鎖削除の背後にあるメカニズムを提供します。 親レコードがデータベースから削除されると、リレーションシップに指定されている子レコードも、データベースから自動的に削除されます。

> [!NOTE]
> 未解決のポインターは、CloudKit を使用する場合に発生する可能性があります。 たとえば、アプリケーションがレコードポインターのリストをフェッチし、レコードを選択して、そのレコードを要求したときに、レコードがデータベースに存在しなくなった可能性があります。 このような状況を適切に処理するには、アプリケーションをコーディングする必要があります。

必須ではありませんが、CloudKit フレームワークを使用する場合は、後方参照を使用することをお勧めします。 Apple は、これを最も効率的な種類の参照になるようにシステムを微調整しました。

開発者は、参照を作成するときに、既にメモリ内にあるレコードを提供するか、レコード識別子への参照を作成できます。 レコード識別子を使用していて、指定された参照がデータベース内に存在しない場合は、未解決のポインターが作成されます。

既知のレコードに対して参照を作成する例を次に示します。

```csharp
var reference = new CKReference(newRecord, new CKReferenceAction());
```

### <a name="assets"></a>アセット

資産を使用すると、大規模な非構造化データのファイルを iCloud にアップロードし、特定のレコードに関連付けることができます。

 [![](intro-to-cloudkit-images/image38.png "Assets allow for a file of large, unstructured data to be uploaded to iCloud and associated with a given Record")](intro-to-cloudkit-images/image38.png#lightbox)

クライアントでは、iCloud サーバーにアップロードされるファイルを記述する `CKRecord` が作成されます。 ファイルを格納する `CKAsset` が作成され、それを記述するレコードにリンクされます。

ファイルがサーバーにアップロードされると、レコードがデータベースに配置され、そのファイルが特別な一括ストレージデータベースにコピーされます。 レコードポインターとアップロードされたファイルの間にリンクが作成されます。

アセットは、CloudKit フレームワークで `CKAsset` クラスを介して公開され、大規模な非構造化データを格納するために使用されます。 開発者は、大量の非構造化データをメモリ内に保持する必要がないため、ディスク上のファイルを使用して資産が実装されます。

資産はレコードによって所有されます。これにより、レコードをポインターとして使用して iCloud から資産を取得できます。 このようにすることで、資産を所有するレコードが削除されたときに、サーバーがアセットをガベージコレクトできます。

`CKAssets` は大規模なデータファイルを処理するため、Apple は CloudKit を使用してアセットを効率的にアップロードしてダウンロードします。

次のコードを使用すると、資産を作成してレコードに関連付けることができます。

```csharp
var fileUrl = new NSUrl("LargeFile.mov");
var asset = new CKAsset(fileUrl);
newRecord ["name"] = asset;
```

CloudKit 内のすべての基本的なオブジェクトについて説明しました。 コンテナーはアプリケーションに関連付けられ、データベースが含まれます。 データベースにはレコードゾーンにグループ化され、レコード識別子によってポイントされるレコードが含まれます。 親子リレーションシップは、参照を使用してレコード間で定義されます。 最後に、大きなファイルをアップロードして、アセットを使用してレコードに関連付けることができます。

## <a name="cloudkit-convenience-api"></a>CloudKit の便利な API

Apple では、CloudKit を操作するための2つの異なる API セットを提供しています。

- **運用 API** – cloudkit のすべての機能を提供します。 より複雑なアプリケーションの場合、この API は CloudKit をきめ細かく制御します。
- **便宜上 API** – cloudkit 機能の共通の事前構成済みのサブセットを提供します。 これは、iOS アプリケーションに CloudKit 機能を含めるための便利で簡単なアクセスソリューションを提供します。

通常、便利な API は、ほとんどの iOS アプリケーションに最適な選択であり、Apple での使用が推奨されます。 このセクションの残りの部分では、次の便利な API のトピックについて説明します。

- レコードを保存しています。
- レコードをフェッチしています。
- レコードを更新しています。

### <a name="common-setup-code"></a>一般的なセットアップコード

CloudKit の便宜的な API を始める前に、標準セットアップコードが必要です。 まず、アプリケーションの `AppDelegate.cs` ファイルを変更し、次のようにします。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Foundation;
using UIKit;
using CloudKit;

namespace CloudKitAtlas
{
    [Register ("AppDelegate")]
    public partial class AppDelegate : UIApplicationDelegate
    {
        public override UIWindow Window { get; set;}
        public CKDatabase PublicDatabase { get; set; }
        public CKDatabase PrivateDatabase { get; set; }

        public override bool FinishedLaunching (UIApplication application, NSDictionary launchOptions)
        {
            application.RegisterForRemoteNotifications ();

            // Get the default public and private databases for
            // the application
            PublicDatabase = CKContainer.DefaultContainer.PublicCloudDatabase;
            PrivateDatabase = CKContainer.DefaultContainer.PrivateCloudDatabase;

            return true;
        }

        public override void RegisteredForRemoteNotifications (UIApplication application, NSData deviceToken)
        {
            Console.WriteLine ("Registered for Push notifications with token: {0}", deviceToken);
        }

        public override void FailedToRegisterForRemoteNotifications (UIApplication application, NSError error)
        {
            Console.WriteLine ("Push subscription failed");
        }

        public override void ReceivedRemoteNotification (UIApplication application, NSDictionary userInfo)
        {
            Console.WriteLine ("Push received");
        }
    }
}
```

上記のコードでは、アプリケーションの他の部分で操作しやすいように、パブリックおよびプライベートの CloudKit データベースをショートカットとして公開しています。

次に、CloudKit を使用するビューまたはビューコンテナーに次のコードを追加します。

```csharp
using CloudKit;
//...

public AppDelegate ThisApp {
    get { return (AppDelegate)UIApplication.SharedApplication.Delegate; }
}
```

これにより、`AppDelegate` にアクセスするショートカットが追加され、上で作成したパブリックデータベースとプライベートデータベースのショートカットにアクセスできるようになります。

このコードを使用して、Xamarin iOS 8 アプリケーションでの CloudKit の便利な API の実装について見ていきましょう。

### <a name="saving-a-record"></a>レコードの保存

次のコードでは、レコードを説明するときに上記のパターンを使用して、新しいレコードを作成し、便利な API を使用してそれをパブリックデータベースに保存します。

```csharp
private const string ReferenceItemRecordName = "ReferenceItems";
...

// Create a new record
var newRecord = new CKRecord (ReferenceItemRecordName);
newRecord ["name"] = (NSString)nameTextField.Text;

// Save it to the database
ThisApp.PublicDatabase.SaveRecord(newRecord, (record, err) => {
    // Was there an error?
    if (err != null) {
        ...
    }
});
```

上記のコードについては、次の3つの点に注意してください。

1. 開発者は、`PublicDatabase`の `SaveRecord` メソッドを呼び出すことによって、データの送信方法、書き込み先のゾーンなどを指定する必要がありません。便宜上 API は、これらの詳細をすべて処理します。
1. 呼び出しは非同期であり、呼び出しが完了すると、成功または失敗のいずれかでコールバックルーチンが提供されます。 呼び出しが失敗した場合は、エラーメッセージが表示されます。
1. CloudKit はローカルストレージ/永続化を提供しません。転送メディアのみです。 そのため、レコードを保存する要求が行われると、その要求は iCloud サーバーに直ちに送信されます。

> [!NOTE]
> 接続が常に削除または中断されているモバイルネットワーク通信の "損失" の性質により、開発者が CloudKit を使用するときに最初に行う必要がある考慮事項の1つは、エラー処理です。

### <a name="fetching-a-record"></a>レコードのフェッチ

レコードを作成して iCloud サーバーに正常に保存したら、次のコードを使用してレコードを取得します。

```csharp
// Create a record ID and fetch the record from the database
var recordID = new CKRecordID("MyRecordName");
ThisApp.PublicDatabase.FetchRecord(recordID, (record, err) => {
    // Was there an error?
    if (err != null) {
        ...
    }
});
```

レコードを保存するのと同様に、上記のコードは非同期で単純であり、優れたエラー処理が必要です。

### <a name="updating-a-record"></a>レコードを更新する

ICloud サーバーからレコードがフェッチされた後、次のコードを使用してレコードを変更し、変更内容をデータベースに保存することができます。

```csharp
// Create a record ID and fetch the record from the database
var recordID = new CKRecordID("MyRecordName");
ThisApp.PublicDatabase.FetchRecord(recordID, (record, err) => {
    // Was there an error?
    if (err != null) {

    } else {
        // Modify the record
        record["name"] = (NSString)"New Name";

        // Save changes to database
        ThisApp.PublicDatabase.SaveRecord(record, (r, e) => {
            // Was there an error?
            if (e != null) {
                 ...
            }
        });
    }
});
```

呼び出しが成功した場合、`PublicDatabase` の `FetchRecord` メソッドは `CKRecord` を返します。 その後、アプリケーションはレコードを変更し、`SaveRecord` を再度呼び出して、変更をデータベースに書き戻します。

このセクションでは、CloudKit の便利な API を使用するときにアプリケーションが使用する一般的なサイクルについて説明しました。 アプリケーションは iCloud にレコードを保存し、icloud からそれらのレコードを取得し、レコードを変更して、それらの変更を iCloud に保存します。

## <a name="designing-for-scalability"></a>スケーラビリティのための設計

この記事では、アプリケーションのオブジェクトモデル全体を iCloud サーバーから格納して取得する方法について説明しました。 このアプローチは、少量のデータと非常に小さなユーザーベースでは適切に機能しますが、情報やユーザーベースの量が増えると、適切に拡張できません。

### <a name="big-data-tiny-device"></a>ビッグデータ、小さなデバイス

アプリケーションが普及すると、データベース内のデータが増え、そのデータ全体をデバイスにキャッシュできるようになります。 この問題を解決するには、次の方法を使用できます。

- **クラウドで大規模なデータを保持**する– cloudkit は、大規模なデータを効率的に処理するように設計されています。
- **クライアントは、そのデータのスライスのみを表示する必要があり**ます。特定の時点で任意のタスクを処理するために必要な最小限のデータを表示します。
- **クライアントビューが変更**される場合があります。各ユーザーは異なる設定を使用しているため、表示されているデータのスライスがユーザーごとに変化し、特定のスライスのユーザーの個々のビューが異なる場合があります。
- **クライアントはクエリを使用してビューポイントにフォーカスを設定し**ます。クエリを使用すると、クラウド内に存在する大規模なデータセットの小さなサブセットを表示できます。

### <a name="queries"></a>クエリ

前述のように、クエリを使用すると、開発者はクラウド内に存在する大規模なデータセットの小さなサブセットを選択できます。 クエリは、CloudKit フレームワークで `CKQuery` クラスを介して公開されます。

クエリでは、レコードの種類 (`RecordType`)、述語 (`NSPredicate`)、および必要に応じて並べ替え記述子 (`NSSortDescriptors`) という3つの異なる項目が結合されます。 CloudKit では、ほとんどの `NSPredicate`がサポートされています。

#### <a name="supported-predicates"></a>サポートされている述語

CloudKit では、クエリを操作するときに次の種類の `NSPredicates` がサポートされています。

1. 名前が変数に格納されている値と等しいレコードを照合します。

    ```csharp
    NSPredicate.FromFormat(string.Format("name = '{0}'", recordName))
    ```

2. では、コンパイル時にキーが認識される必要がないように、動的キー値に基づいて照合を行うことができます。

    ```csharp
    NSPredicate.FromFormat(string.Format("{0} = '{1}'", key, value))
    ```

3. レコードの値が指定された値を超えるレコードの照合:

    ```csharp
    NSPredicate.FromFormat(string.Format("start > {0}", (NSDate)date))
    ```

4. レコードの場所が指定された場所の100メートル内にある場合は、次のように一致します。

    ```csharp
    var location = new CLLocation(37.783,-122.404);
    var predicate = NSPredicate.FromFormat(string.Format("distanceToLocation:fromLocation(Location,{0}) < 100", location));
    ```

5. CloudKit はトークン化された検索をサポートしています。 この呼び出しにより、2つのトークンが作成されます。1つは `after` 用で、もう1つは `session`用です。 次の2つのトークンを含むレコードが返されます。

    ```csharp
    NSPredicate.FromFormat(string.Format("ALL tokenize({0}, 'Cdl') IN allTokens", "after session"))
    ```

6. CloudKit は、`AND` 演算子を使用して結合された複合述語をサポートしています。

    ```csharp
    NSPredicate.FromFormat(string.Format("start > {0} AND name = '{1}'", (NSDate)date, recordName))
    ```

#### <a name="creating-queries"></a>クエリの作成

次のコードを使用して、Xamarin iOS 8 アプリケーションで `CKQuery` を作成できます。

```csharp
var recordName = "MyRec";
var predicate = NSPredicate.FromFormat(string.Format("name = '{0}'", recordName));
var query = new CKQuery("CloudRecords", predicate);
```

まず、指定された名前に一致するレコードのみを選択する述語を作成します。 次に、述語に一致する指定されたレコードの種類のレコードを選択するクエリを作成します。

#### <a name="performing-a-query"></a>クエリの実行

クエリが作成されたら、次のコードを使用してクエリを実行し、返されたレコードを処理します。

```csharp
var recordName = "MyRec";
var predicate = NSPredicate.FromFormat(string.Format("name = {0}", recordName));
var query = new CKQuery("CloudRecords", predicate);

ThisApp.PublicDatabase.PerformQuery(query, CKRecordZone.DefaultRecordZone().ZoneId, (NSArray results, NSError err) => {
    // Was there an error?
    if (err != null) {
       ...
    } else {
        // Process the returned records
        for(nint i = 0; i < results.Count; ++i) {
            var record = (CKRecord)results[i];
        }
    }
});
```

上記のコードは、上記で作成したクエリを受け取り、パブリックデータベースに対して実行します。 レコードゾーンが指定されていないため、すべてのゾーンが検索されます。 エラーが発生しなかった場合は、クエリのパラメーターに一致する `CKRecords` の配列が返されます。

クエリについて考える方法は、それらがポーリングであり、大規模なデータセットを使用してスライスする場合に最適です。 ただし、クエリは、次の理由により、大部分の静的データセットには適していません。

- デバイスのバッテリ寿命には問題があります。
- ネットワークトラフィックには問題ありません。
- 表示される情報は、アプリケーションがデータベースをポーリングする頻度によって制限されるため、ユーザーエクスペリエンスには不適切です。 今日のユーザーは、何らかの変更があったときにプッシュ通知を受け取ることになります

### <a name="subscriptions"></a>サブスクリプション

大規模で静的なデータセットを処理する場合、クライアントデバイスでクエリを実行する必要はありません。クライアントの代わりにサーバー上で実行する必要があります。 クエリはバックグラウンドで実行する必要があり、現在のデバイスまたは別のデバイスによって同じデータベースに接しているかどうかにかかわらず、単一のレコードを保存するたびに実行する必要があります。

最後に、サーバー側のクエリを実行するときに、データベースにアタッチされているすべてのデバイスにプッシュ通知を送信する必要があります。

サブスクリプションは、CloudKit フレームワークで `CKSubscription` クラスを介して公開されます。 レコードの種類 (`RecordType`)、述語 (`NSPredicate`)、Apple Push Notification (`Push`) を結合します。

> [!NOTE]
> CloudKit のプッシュは、プッシュが発生する原因など、CloudKit 固有の情報を含むペイロードが含まれているため、若干強化されています。

#### <a name="how-subscriptions-work"></a>サブスクリプションのしくみ

コードでC#サブスクリプションを実装する前に、サブスクリプションの動作の概要を簡単に説明します。

 [![](intro-to-cloudkit-images/image39.png "An overview of how subscriptions work")](intro-to-cloudkit-images/image39.png#lightbox)

上のグラフは、次のような一般的なサブスクリプションプロセスを示しています。

1. クライアントデバイスは、サブスクリプションをトリガーする一連の条件と、トリガーが発生したときに送信されるプッシュ通知を含む新しいサブスクリプションを作成します。
2. サブスクリプションは、既存のサブスクリプションのコレクションに追加されるデータベースに送信されます。
3. 2番目のデバイスでは、新しいレコードを作成し、そのレコードをデータベースに保存します。
4. データベースは、サブスクリプションの一覧を検索して、新しいレコードがいずれかの条件に一致するかどうかを確認します。
5. 一致が見つかった場合、プッシュ通知は、サブスクリプションを登録したデバイスに、トリガーされたレコードに関する情報と共に送信されます。

この知識があれば、Xamarin iOS 8 アプリケーションでサブスクリプションを作成する方法を見てみましょう。

#### <a name="creating-subscriptions"></a>サブスクリプションの作成

次のコードを使用して、サブスクリプションを作成できます。

```csharp
// Create a new subscription
DateTime date;
var predicate = NSPredicate.FromFormat(string.Format("start > {0}", (NSDate)date));
var subscription = new CKSubscription("RecordType", predicate, CKSubscriptionOptions.FiresOnRecordCreation);

// Describe the type of notification
var notificationInfo = new CKNotificationInfo();
notificationInfo.AlertLocalizationKey = "LOCAL_NOTIFICATION_KEY";
notificationInfo.SoundName = "ping.aiff";
notificationInfo.ShouldBadge = true;

// Attach the notification info to the subscription
subscription.NotificationInfo = notificationInfo;
```

まず、サブスクリプションをトリガーする条件を提供する述語を作成します。 次に、特定のレコードの種類に対してサブスクリプションを作成し、トリガーをテストするときのオプションを設定します。 最後に、サブスクリプションがトリガーされたときに発生する通知の種類を定義し、サブスクリプションにアタッチします。

### <a name="saving-subscriptions"></a>サブスクリプションの保存

サブスクリプションが作成されると、次のコードによってデータベースに保存されます。

```csharp
// Save the subscription to the database
ThisApp.PublicDatabase.SaveSubscription(subscription, (s, err) => {
    // Was there an error?
    if (err != null) {

    }
});
```

この呼び出しは、便利な API を使用して非同期的に行うことができ、簡単なエラー処理を提供します。

#### <a name="handling-push-notifications"></a>プッシュ通知の処理

開発者が以前に Apple Push notification (APS) を使用していた場合は、CloudKit によって生成された通知を処理するプロセスを理解しておく必要があります。

`AppDelegate.cs`で、次のように `ReceivedRemoteNotification` クラスをオーバーライドします。

```csharp
public override void ReceivedRemoteNotification (UIApplication application, NSDictionary userInfo)
{
    // Parse the notification into a CloudKit Notification
    var notification = CKNotification.FromRemoteNotificationDictionary (userInfo);

    // Get the body of the message
    var alertBody = notification.AlertBody;

    // Was this a query?
    if (notification.NotificationType == CKNotificationType.Query) {
        // Yes, convert to a query notification and get the record ID
        var query = notification as CKQueryNotification;
        var recordID = query.RecordId;
    }
}
```

上記のコードでは、CloudKit に対して、CloudKit 通知に userInfo を解析するように要求しています。 次に、アラートに関する情報が抽出されます。 最後に、通知の種類がテストされ、それに応じて通知が処理されます。

このセクションでは、クエリとサブスクリプションを使用して、上で示したビッグデータの小さなデバイスの問題を解決する方法について説明しました。 アプリケーションは大きなデータをクラウドに残し、これらのテクノロジを使用してこのデータセットにビューを提供します。

## <a name="cloudkit-user-accounts"></a>CloudKit のユーザーアカウント

この記事の冒頭で説明したように、CloudKit は既存の iCloud インフラストラクチャの上に構築されています。 次のセクションでは、CloudKit API を使用して開発者にアカウントを公開する方法について詳しく説明します。

### <a name="authentication"></a>認証

ユーザーアカウントを扱う場合の最初の考慮事項は認証です。 CloudKit は、デバイス上の現在ログインしている iCloud ユーザーによる認証をサポートしています。 認証はバックグラウンドで行われ、iOS によって処理されます。 このようにして、開発者は認証の実装の詳細について心配する必要がありません。 ユーザーがログオンしているかどうかをテストするだけです。

### <a name="user-account-information"></a>ユーザー アカウント情報

CloudKit は、開発者に次のユーザー情報を提供します。

- **Id** –ユーザーを一意に識別する方法。
- **メタデータ**–ユーザーに関する情報を保存および取得する権限です。
- **プライバシー** -すべての情報は、プライバシーを意識した、またはによって処理されます。 ユーザーが同意していない限り、何も公開されません。
- **[検出]** –ユーザーは、同じアプリケーションを使用している友人を見つけることができます。

次に、これらのトピックについて詳しく説明します。

#### <a name="identity"></a>ID と

前述のように、CloudKit は、アプリケーションが特定のユーザーを一意に識別する方法を提供します。

 [![](intro-to-cloudkit-images/image40.png "Uniquely identifing a given user")](intro-to-cloudkit-images/image40.png#lightbox)

ユーザーのデバイスで実行されているクライアントアプリケーションと、CloudKit コンテナー内の特定のユーザープライベートデータベースがあります。 クライアントアプリケーションは、これらの特定のユーザーのいずれかにリンクされます。 これは、デバイスのローカルに iCloud にログインしているユーザーに基づいています。

これは iCloud から送信されるため、ユーザー情報の豊富なバッキングストアがあります。 また、iCloud は実際にはコンテナーをホストしているため、ユーザーを関連付けることができます。 上の図では、iCloud アカウント `user@icloud.com` が現在のクライアントにリンクされています。

コンテナーごとに、ランダムに生成された一意のユーザー ID が作成され、ユーザーの iCloud アカウント (電子メールアドレス) に関連付けられます。 このユーザー ID は、アプリケーションに返されます。これは、開発者が適合する任意の方法で使用できます。

> [!NOTE]
> 同じデバイスで同じ iCloud ユーザー用に実行されている異なるアプリケーションは、異なる CloudKit コンテナーに接続されているため、異なるユーザー Id を持つことになります。

次のコードは、デバイスに現在ログインしている iCloud ユーザーの CloudKit ユーザー ID を取得します。

```csharp
public CKRecordID UserID { get; set; }
...

// Get the CloudKit User ID
CKContainer.DefaultContainer.FetchUserRecordId ((recordID, err) => {
    // Was there an error?
    if (err!=null) {
        Console.WriteLine("Error: {0}", err.LocalizedDescription);
    } else {
        // Save user ID
        UserID = recordID;
    }
});
```

上記のコードは、現在ログインしているユーザーの ID を提供するように CloudKit コンテナーに要求しています。 この情報は iCloud サーバーから送信されるため、呼び出しは非同期であり、エラー処理が必要です。

#### <a name="metadata"></a>メタデータ

CloudKit の各ユーザーには、それらを説明する特定のメタデータがあります。 このメタデータは、CloudKit レコードとして表されます。

 [![](intro-to-cloudkit-images/image41.png "Each user in CloudKit has specific Metadata that describes them")](intro-to-cloudkit-images/image41.png#lightbox)

プライベートデータベース内でコンテナーの特定のユーザーを検索するには、そのユーザーを定義する1つのレコードがあります。 パブリックデータベース内には、コンテナーのユーザーごとに1つずつ、多くのユーザーレコードがあります。 このうちの1つは、現在ログオンしているユーザーのレコード ID と一致するレコード ID を持つことになります。

パブリックデータベースのユーザーレコードは、世界中で読むことができます。 ほとんどの場合、通常のレコードとして扱われ、`CKRecordTypeUserRecord`の種類があります。 これらのレコードはシステムによって予約されており、クエリでは使用できません。

ユーザーレコードにアクセスするには、次のコードを使用します。

```csharp
public CKRecord UserRecord { get; set; }
...

// Get the user's record
PublicDatabase.FetchRecord(UserID, (record ,er) => {
    //was there an error?
    if (er != null) {
        Console.WriteLine("Error: {0}", er.LocalizedDescription);
    } else {
        // Save the user record
        UserRecord = record;
    }
});
```

上記のコードは、前にアクセスした ID を持つユーザーのユーザーレコードを返すようにパブリックデータベースに要求しています。 この情報は iCloud サーバーから送信されるため、呼び出しは非同期であり、エラー処理が必要です。

#### <a name="privacy"></a>プライバシー

CloudKit は、現在ログオンしているユーザーのプライバシーを保護するために、既定で設計されています。 既定では、ユーザーに関する個人を特定する情報は公開されません。 アプリケーションがユーザーに関する限られた情報を必要とする場合もあります。

このような場合、アプリケーションは、ユーザーがこの情報を開示するように要求できます。 ユーザーに対して、アカウント情報を公開するかどうかを確認するダイアログボックスが表示されます。

#### <a name="discovery"></a>の探索

ユーザーがユーザーアカウント情報へのアクセスが制限されていることをユーザーがオプトインしていると仮定すると、アプリケーションの他のユーザーが検出できるようになります。

 [![](intro-to-cloudkit-images/image42.png "A user can be discoverable to other users of the application")](intro-to-cloudkit-images/image42.png#lightbox)

クライアントアプリケーションはコンテナーと通信し、コンテナーは iCloud と通信してユーザー情報にアクセスします。 ユーザーは電子メールアドレスを指定できます。探索を使用して、ユーザーに関する情報を取得できます。 必要に応じて、ユーザー ID を使用して、ユーザーに関する情報を探索することもできます。

また、CloudKit では、アドレス帳全体に対してクエリを実行することで、現在ログオンしている iCloud ユーザーの友人である可能性のあるユーザーに関する情報を探索することもできます。 CloudKit プロセスは、ユーザーの連絡先ブックをプルし、電子メールアドレスを使用して、それらのアドレスに一致するアプリケーションの他のユーザーが検索できるかどうかを確認します。

これにより、アプリケーションは、アクセスを提供したり、連絡先へのアクセスを承認するようにユーザーに依頼したりせずに、ユーザーの連絡先を利用できます。 アプリケーションが連絡先情報を利用できるようになった時点で、CloudKit プロセスのみがアクセス権を持っています。

要約すると、ユーザーの探索に使用できる入力には3種類あります。

- **ユーザーレコード ID** –現在ログインしている cloudkit ユーザーのユーザー ID に対して、検出を実行できます。
- **ユーザーの電子メールアドレス**–ユーザーは電子メールアドレスを入力し、検出に使用できます。
- **連絡先ブック**–ユーザーのアドレス帳を使用して、連絡先に記載されているものと同じ電子メールアドレスを持つアプリケーションのユーザーを検出できます。

ユーザー探索では、次の情報が返されます。

- **ユーザーレコード ID** -パブリックデータベース内のユーザーの一意の id。
- 姓**と名**-パブリックデータベースに格納されます。

この情報は、オプトインが検出されたユーザーに対してのみ返されます。

次のコードでは、デバイス上の iCloud に現在ログインしているユーザーに関する情報が検出されます。

```csharp
public CKDiscoveredUserInfo UserInfo { get; set; }
//...

// Get the user's metadata
CKContainer.DefaultContainer.DiscoverUserInfo(UserID, (info, e) => {
    // Was there an error?
    if (e != null) {
        Console.WriteLine("Error: {0}", e.LocalizedDescription);
    } else {
        // Save the user info
        UserInfo = info;
    }
});
```

次のコードを使用して、連絡先ブック内のすべてのユーザーを照会します。

```csharp
// Ask CloudKit for all of the user's friends information
CKContainer.DefaultContainer.DiscoverAllContactUserInfos((info, er) => {
    // Was there an error
    if (er != null) {
        Console.WriteLine("Error: {0}", er.LocalizedDescription);
    } else {
        // Process all returned records
        for(int i = 0; i < info.Count(); ++i) {
            // Grab a user
            var userInfo = info[i];
        }
    }
});
```

このセクションでは、CloudKit がアプリケーションに提供できるユーザーアカウントにアクセスする4つの主な領域について説明しました。 ユーザーの Id とメタデータを取得して、CloudKit に組み込まれているプライバシーポリシーと最後に、アプリケーションの他のユーザーを検出する機能。

## <a name="the-development-and-production-environments"></a>開発環境と運用環境

CloudKit は、アプリケーションのレコードの種類とデータに対して個別の開発環境と運用環境を提供します。 開発環境は、開発チームのメンバーだけが使用できる、より柔軟な環境です。 アプリケーションによってレコードに新しいフィールドが追加され、そのレコードが開発環境に保存されると、サーバーによってスキーマ情報が自動的に更新されます。

開発者はこの機能を使用して、開発時にスキーマを変更できます。これにより、時間が節約されます。 注意点として、レコードにフィールドを追加した後で、そのフィールドに関連付けられているデータ型をプログラムで変更することはできません。 フィールドの型を変更するには、開発者が[Cloudkit ダッシュボード](https://icloud.developer.apple.com/dashboard/)のフィールドを削除し、新しい型を使用してもう一度追加する必要があります。

開発者は、アプリケーションをデプロイする前に、 **Cloudkit ダッシュボード**を使用して、スキーマとデータを運用環境に移行できます。 運用環境に対して実行する場合、サーバーは、アプリケーションがプログラムによってスキーマを変更するのを防ぎます。 開発者は引き続き**Cloudkit ダッシュボード**を変更できますが、運用環境のレコードにフィールドを追加しようとすると、エラーが発生します。

> [!NOTE]
> IOS シミュレーターは、**開発環境**でのみ動作します。 開発者が**運用環境**でアプリケーションをテストする準備ができたら、物理的な iOS デバイスが必要です。

## <a name="shipping-a-cloudkit-enabled-app"></a>CloudKit が有効になっているアプリの発送

CloudKit を使用するアプリケーションを出荷する前に、**運用環境の Cloudkit 環境**を対象とするように構成する必要があります。そうしないと、アプリケーションは Apple によって拒否されます。

次を実行します。

1. Visual Studio for Ma で、 **Release** > **iOS デバイス**用にアプリケーションをコンパイルします。

    [![](intro-to-cloudkit-images/shipping01.png "Compile the application for Release")](intro-to-cloudkit-images/shipping01.png#lightbox)

2. **[ビルド]** メニューの **[アーカイブ]** を選択します。

    [![](intro-to-cloudkit-images/shipping02.png "Select Archive")](intro-to-cloudkit-images/shipping02.png#lightbox)

3. **アーカイブ**が作成され、Visual Studio for Mac に表示されます。

    [![](intro-to-cloudkit-images/shipping03.png "The Archive will be created and displayed")](intro-to-cloudkit-images/shipping03.png#lightbox)

4. **Xcode** を起動します。
5. **[ウィンドウ]** メニューの **[オーガナイザー]** を選択します。

    [![](intro-to-cloudkit-images/shipping04.png "Select Organizer")](intro-to-cloudkit-images/shipping04.png#lightbox)

6. アプリケーションのアーカイブを選択し、 **[エクスポート]** ボタンをクリックします。

    [![](intro-to-cloudkit-images/shipping05.png "The application's archive")](intro-to-cloudkit-images/shipping05.png#lightbox)

7. エクスポートするメソッドを選択し、 **[次へ]** ボタンをクリックします。

    [![](intro-to-cloudkit-images/shipping06.png "Select a method for export")](intro-to-cloudkit-images/shipping06.png#lightbox)

8. ドロップダウンリストから**開発チーム**を選択し、 **[選択]** ボタンをクリックします。

    [![](intro-to-cloudkit-images/shipping07.png "Select the Development Team from the dropdown list")](intro-to-cloudkit-images/shipping07.png#lightbox)

9. ドロップダウンリストから **[Production]** を選択し、 **[次へ]** ボタンをクリックします。

    [![](intro-to-cloudkit-images/shipping08.png "Select Production from the dropdown list")](intro-to-cloudkit-images/shipping08.png#lightbox)

10. 設定を確認し、 **[エクスポート]** ボタンをクリックします。

    [![](intro-to-cloudkit-images/shipping09.png "Review the setting")](intro-to-cloudkit-images/shipping09.png#lightbox)

11. 生成されたアプリケーション `.ipa` ファイルを生成する場所を選択します。

ITunes Connect に直接アプリケーションを送信するために似ていますクリックするだけに、**送信しています。** オーガナイザー ウィンドウで、アーカイブを選択した後に... エクスポートではなくボタンをクリックします。

## <a name="when-to-use-cloudkit"></a>CloudKit を使用する場合

この記事で説明したように、CloudKit を使用すると、アプリケーションで iCloud サーバーから情報を格納および取得するための簡単な方法が提供されます。 これは、CloudKit が廃止されたり、既存のツールやフレームワークを廃止したりすることはありません。

### <a name="use-cases"></a>ユース ケース

次のユースケースは、開発者が特定の iCloud フレームワークまたはテクノロジを使用するタイミングを決定するのに役立ちます。

- **iCloud のキーと値のストア**–少量のデータを非同期的に最新の状態に保ち、アプリケーションの設定を操作するのに適しています。 ただし、非常に少量の情報に制限されています。
- **Icloud ドライブ**-既存の Icloud ドキュメント api の上に構築され、ファイルシステムから非構造化データを同期する単純な api を提供します。 Mac OS X の完全なオフラインキャッシュが用意されており、ドキュメント中心のアプリケーションに適しています。
- **ICloud コアデータ**-すべてのユーザーのデバイス間でデータをレプリケートできるようにします。 データはシングルユーザーであり、プライベートな構造化データの同期を維持するのに最適です。
- **Cloudkit** –構造と一括の両方のパブリックデータを提供し、大規模なデータセットと大規模な非構造化ファイルの両方を処理できます。 ユーザーの iCloud アカウントに関連付けられ、クライアント向けのデータ転送を提供します。

開発者は、このようなユースケースを念頭に置いて、現在必要なアプリケーション機能を提供し、将来の成長に優れたスケーラビリティを提供するために、適切な iCloud テクノロジを選択する必要があります。

## <a name="summary"></a>要約

この記事では、CloudKit API の簡単な概要について説明しました。 CloudKit を使用するように Xamarin iOS アプリケーションをプロビジョニングして構成する方法について説明しました。 CloudKit の便利な API の機能について説明しました。 ここでは、クエリとサブスクリプションを使用して、CloudKit が有効になっているアプリケーションのスケーラビリティを設計する方法を説明しました。 最後に、CloudKit によってアプリケーションに公開されるユーザーアカウント情報が表示されます。

## <a name="related-links"></a>関連リンク

- [CloudKit (Apple)](https://developer.apple.com/icloud/cloudkit/)
- [CloudKitAtlas (サンプル)](https://docs.microsoft.com/samples/xamarin/ios-samples/ios8-cloudkitatlas)
- [プロビジョニングプロファイルの作成](~/ios/get-started/installation/device-provisioning/index.md)
