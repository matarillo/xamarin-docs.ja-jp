---
title: Xamarin 用の Apple でのサインインの使用
description: Xamarin. Forms モバイルアプリケーションで Apple とのサインインを実装する方法について説明します。
ms.prod: xamarin
ms.assetid: 2E47E7F2-93D4-4CA3-9E66-247466D25E4D
ms.technology: xamarin-forms
author: davidortinau
ms.author: daortin
ms.date: 09/10/2019
ms.openlocfilehash: df011a6eb72b6eb30af0a197d4be48b0f2494384
ms.sourcegitcommit: fc689c1a6b641c124378dedc1bd157d96fc759a7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/27/2019
ms.locfileid: "71319522"
---
# <a name="use-sign-in-with-apple-in-xamarinforms"></a>Xamarin での Apple でのサインインの使用

[![サンプルのダウンロード](~/media/shared/download.png)サンプルをダウンロードします。](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/signinwithapple/)

Apple でのサインインは、サードパーティの認証サービスを使用する iOS 13 のすべての新しいアプリケーションを対象としています。 IOS と Android の間の実装の詳細は大きく異なります。 このガイドでは、Xamarin. フォームでこの操作を行う方法について説明します。

このガイドとサンプルでは、Apple でのサインインを処理するために、特定のプラットフォームサービスが使用されています。

- OpenID/OpenAuth と Azure Functions する汎用 web サービスを使用した Android
- ios では、ios 13 での認証にネイティブ API を使用し、iOS 12 以下の汎用 web サービスにフォールバックします。

## <a name="a-sample-apple-sign-in-flow"></a>Apple サインインフローのサンプル

このサンプルは、Xamarin. Forms アプリで機能するために Apple サインインを取得するためのこだわり実装を提供します。

認証フローに役立つ2つの Azure Functions を使用します。

1. `applesignin_auth`-Apple サインイン承認 URL を生成し、それにリダイレクトします。  これは、モバイルアプリではなくサーバー側で実行します。これにより、 `state` Apple のサーバーがコールバックを送信するときに、をキャッシュして検証することができます。
2. `applesignin_callback`-Apple からのポストコールバックを処理し、アクセストークンと ID トークンの認証コードを安全に交換します。  最後に、アプリの URI スキームにリダイレクトし、URL フラグメント内のトークンを返します。

モバイルアプリは、選択されたカスタム URI スキーム (この場合`xamarinformsapplesignin://`は) を処理するように自身を登録します。これにより、関数は`applesignin_callback`トークンを再びリレーできるようになります。

ユーザーが認証を開始すると、次の手順が実行されます。

1. モバイルアプリによって`nonce`と`state`の値が生成され`applesignin_auth` 、Azure 関数に渡されます。
2. Azure `applesignin_auth`関数は、(指定さ`state`れたと`nonce`を使用して) Apple サインイン承認 URL を生成し、モバイルアプリブラウザーをそれにリダイレクトします。
3. ユーザーは、Apple のサーバーでホストされている Apple サインイン承認ページで資格情報を安全に入力します。
4. Apple のサーバーで apple サインインフローが完了する`redirect_uri` `applesignin_callback`と、apple は Azure 関数になるにリダイレクトします。
5. `applesignin_callback`関数に送信される Apple からの要求は、正しい`state`が返されること、および ID トークンの要求が有効であることを確認するために検証されます。
6. Azure `applesignin_callback`関数は、Apple `code`によって投稿されたを、_アクセストークン_、_更新トークン_、および_id トークン_(ユーザー ID、名前、および電子メールに関する要求を含む) に対して交換します。
7. 最後`applesignin_callback`に、Azure 関数は、uri フラグメントをトークンと共`xamarinformsapplesignin://`に追加して、アプリの uri スキーム () `xamarinformsapplesignin://#access_token=...&refresh_token=...&id_token=...`にリダイレクトします (例:)。
8. モバイルアプリは URI フラグメント`AppleAccount`をに解析し、受信した要求が`nonce`フローの`nonce`開始時に生成されたと一致することを検証します。
9. モバイルアプリは認証されました。

## <a name="azure-functions"></a>Azure Functions

このサンプルでは、Azure Functions を使用します。 また、ASP.NET Core コントローラーや同様の web サーバーソリューションでも、同じ機能を提供できます。

### <a name="configuration"></a>構成

Azure Functions を使用する場合は、いくつかのアプリ設定を構成する必要があります。

- `APPLE_SIGNIN_KEY_ID`-これは`KeyId`以前のバージョンです。
- `APPLE_SIGNIN_TEAM_ID`-これは通常、[メンバーシッププロファイル](https://developer.apple.com/account/#/membership)にある_チーム ID_です
- `APPLE_SIGNIN_SERVER_ID`:これは、 `ServerId`以前のです。  アプリ_バンドル id_で*はなく、* 作成した*サービス ID*の*識別子*です。
- `APPLE_SIGNIN_APP_CALLBACK_URI`-これは、アプリケーションにリダイレクトするカスタム URI スキームです。  このサンプル`xamarinformsapplesignin://`では、を使用します。
- `APPLE_SIGNIN_REDIRECT_URI`-*サービス ID*を作成するときに設定した*リダイレクト URL*が、 *Apple サインイン*構成セクションに表示されます。  テストするには、次のようになります。`http://local.test:7071/api/applesignin_callback`
- `APPLE_SIGNIN_P8_KEY`- `.p8`ファイルのテキストの内容。 `\n`すべての改行が削除され、1つの長い文字列になります。

### <a name="security-considerations"></a>セキュリティの考慮事項

アプリケーションコード内に P8 キーを格納**しない**でください。 アプリケーションコードは、簡単にダウンロードして逆アセンブルできます。 

また、を`WebView`使用して認証フローをホストし、URL ナビゲーションイベントをインターセプトして認証コードを取得することは、不適切な方法であると考えられます。 現時点では、トークン交換を処理するためにサーバーでコードをホストすることなく、iOS13 + 以外のデバイスでのサインインを完全に安全に処理することはできません。 サーバーで認証 url 生成コードをホストすることをお勧めします。これにより、Apple がサーバーへのポストコールバックを発行したときに、状態をキャッシュして検証することができます。

## <a name="a-cross-platform-sign-in-service"></a>クロスプラットフォームサインインサービス

Xamarin DependencyService を使用すると、iOS のプラットフォームサービスを使用する別個の認証サービスと、共有インターフェイスに基づく Android およびその他の iOS 以外のプラットフォーム用の汎用 web サービスを作成できます。

```csharp
public interface IAppleSignInService
{
    bool Callback(string url);

    Task<AppleAccount> SignInAsync();
}
```

IOS では、ネイティブ Api が使用されます。

```csharp
public class AppleSignInServiceiOS : IAppleSignInService
{
#if __IOS__13
    AuthManager authManager;
#endif

    bool Is13 => UIDevice.CurrentDevice.CheckSystemVersion(13, 0);
    WebAppleSignInService webSignInService;

    public AppleSignInServiceiOS()
    {
        if (!Is13)
            webSignInService = new WebAppleSignInService();
    }

    public async Task<AppleAccount> SignInAsync()
    {
        // Fallback to web for older iOS versions
        if (!Is13)
            return await webSignInService.SignInAsync();

        AppleAccount appleAccount = default;

#if __IOS__13
        var provider = new ASAuthorizationAppleIdProvider();
        var req = provider.CreateRequest();

        authManager = new AuthManager(UIApplication.SharedApplication.KeyWindow);

        req.RequestedScopes = new[] { ASAuthorizationScope.FullName, ASAuthorizationScope.Email };
        var controller = new ASAuthorizationController(new[] { req });

        controller.Delegate = authManager;
        controller.PresentationContextProvider = authManager;

        controller.PerformRequests();

        var creds = await authManager.Credentials;

        if (creds == null)
            return null;

        appleAccount = new AppleAccount();
        appleAccount.IdToken = JwtToken.Decode(new NSString(creds.IdentityToken, NSStringEncoding.UTF8).ToString());
        appleAccount.Email = creds.Email;
        appleAccount.UserId = creds.User;
        appleAccount.Name = NSPersonNameComponentsFormatter.GetLocalizedString(creds.FullName, NSPersonNameComponentsFormatterStyle.Default, NSPersonNameComponentsFormatterOptions.Phonetic);
        appleAccount.RealUserStatus = creds.RealUserStatus.ToString();
#endif

        return appleAccount;
    }

    public bool Callback(string url) => true;
}

#if __IOS__13
class AuthManager : NSObject, IASAuthorizationControllerDelegate, IASAuthorizationControllerPresentationContextProviding
{
    public Task<ASAuthorizationAppleIdCredential> Credentials
        => tcsCredential?.Task;

    TaskCompletionSource<ASAuthorizationAppleIdCredential> tcsCredential;

    UIWindow presentingAnchor;

    public AuthManager(UIWindow presentingWindow)
    {
        tcsCredential = new TaskCompletionSource<ASAuthorizationAppleIdCredential>();
        presentingAnchor = presentingWindow;
    }

    public UIWindow GetPresentationAnchor(ASAuthorizationController controller)
        => presentingAnchor;

    [Export("authorizationController:didCompleteWithAuthorization:")]
    public void DidComplete(ASAuthorizationController controller, ASAuthorization authorization)
    {
        var creds = authorization.GetCredential<ASAuthorizationAppleIdCredential>();
        tcsCredential?.TrySetResult(creds);
    }

    [Export("authorizationController:didCompleteWithError:")]
    public void DidComplete(ASAuthorizationController controller, NSError error)
        => tcsCredential?.TrySetException(new Exception(error.LocalizedDescription));
}
#endif
```

コンパイルフラグ`__IOS__13`は、汎用 web サービスにフォールバックする iOS 13 および従来のバージョンのサポートを提供するために使用されます。

Android では、Azure Functions を使用した汎用 web サービスが使用されます。

```csharp
public class WebAppleSignInService : IAppleSignInService
{
    // IMPORTANT: This is what you register each native platform's url handler to be
    public const string CallbackUriScheme = "xamarinformsapplesignin";
    public const string InitialAuthUrl = "http://local.test:7071/api/applesignin_auth";

    string currentState;
    string currentNonce;

    TaskCompletionSource<AppleAccount> tcsAccount = null;

    public bool Callback(string url)
    {
        // Only handle the url with our callback uri scheme
        if (!url.StartsWith(CallbackUriScheme + "://"))
            return false;

        // Ensure we have a task waiting
        if (tcsAccount != null && !tcsAccount.Task.IsCompleted)
        {
            try
            {
                // Parse the account from the url the app opened with
                var account = AppleAccount.FromUrl(url);

                // IMPORTANT: Validate the nonce returned is the same as our originating request!!
                if (!account.IdToken.Nonce.Equals(currentNonce))
                    tcsAccount.TrySetException(new InvalidOperationException("Invalid or non-matching nonce returned"));

                // Set our account result
                tcsAccount.TrySetResult(account);
            }
            catch (Exception ex)
            {
                tcsAccount.TrySetException(ex);
            }
        }

        tcsAccount.TrySetResult(null);
        return false;
    }

    public async Task<AppleAccount> SignInAsync()
    {
        tcsAccount = new TaskCompletionSource<AppleAccount>();

        // Generate state and nonce which the server will use to initial the auth
        // with Apple.  The nonce should flow all the way back to us when our function
        // redirects to our app
        currentState = Util.GenerateState();
        currentNonce = Util.GenerateNonce();

        // Start the auth request on our function (which will redirect to apple)
        // inside a browser (either SFSafariViewController, Chrome Custom Tabs, or native browser)
        await Xamarin.Essentials.Browser.OpenAsync($"{InitialAuthUrl}?&state={currentState}&nonce={currentNonce}",
            Xamarin.Essentials.BrowserLaunchMode.SystemPreferred);

        return await tcsAccount.Task;
    }
}
```

## <a name="summary"></a>まとめ

この記事では、Xamarin アプリケーションで使用するために Apple でサインインを設定するために必要な手順について説明しました。

## <a name="related-links"></a>関連リンク

- [XamarinFormsAppleSignIn (サンプル)](https://github.com/Redth/Xamarin.AppleSignIn.Sample)
- [Apple でのサインインに関するガイドライン](https://developer.apple.com/design/human-interface-guidelines/sign-in-with-apple/overview/)
