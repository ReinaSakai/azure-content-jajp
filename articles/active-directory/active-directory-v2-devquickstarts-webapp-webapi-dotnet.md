<properties
	pageTitle="Azure AD v2.0 .NET Web アプリ | Microsoft Azure"
	description="サインインに Microsoft の個人および職場/学校アカウントを使用する Web サービスを呼び出す .NET MVC Web アプリを構築する方法。"
	services="active-directory"
	documentationCenter=".net"
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="dastrock"/>

# .NET Web アプリから Web API を呼び出す

v2.0 エンドポイントを使用すると、Microsoft の個人アカウントと職場/学校アカウントの両方に対応した Web アプリと Web API に認証をすばやく追加できます。ここでは、OpenID Connect と Microsoft の OWIN ミドルウェアを使用してユーザーのサインイン処理を行う MVC Web アプリを構築します。この Web アプリは、OAuth 2.0 によってセキュリティ保護された Web API 用の OAuth 2.0 アクセス トークンを取得し、指定したユーザーの "To-Do List" 項目の作成、読み取り、削除を可能にします。

> [AZURE.NOTE]
	Azure Active Directory のシナリオおよび機能のすべてが v2.0 エンドポイントでサポートされているわけではありません。v2.0 エンドポイントを使用する必要があるかどうかを判断するには、[v2.0 の制限事項](active-directory-v2-limitations.md)に関するページをお読みください。

このチュートリアルでは主に、ADAL を使用して Web アプリでアクセス トークンを取得および使用する方法を説明します。完全な詳細については、[こちら](active-directory-v2-flows.md#web-apps)を参照してください。前提条件としてまず、[基本的なサインインを Web アプリに追加](active-directory-v2-devquickstarts-dotnet-web.md)する方法、または、[Web API を適切に保護](active-directory-v2-devquickstarts-dotnet-api.md)する方法を知っておくと便利です。

## サンプル コードのダウンロード

このチュートリアルのコードは、[GitHub](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet) で管理されています。追加の参考資料として、[アプリのスケルトン (.zip) をダウンロード](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet/archive/skeleton.zip)したり、スケルトンを複製したりすることができます:

```git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet.git```

また、[完成したアプリを zip ファイルとしてダウンロード](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet/archive/complete.zip)することも、完成したアプリを複製することもできます。

```git clone --branch complete https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet.git```

## アプリを登録します
[apps.dev.microsoft.com](https://apps.dev.microsoft.com) で新しいアプリを作成するか、この[詳細な手順](active-directory-v2-app-registration.md)に従います。次のことを確認します。

- アプリに割り当てられた**アプリケーション ID** をメモしておきます。これは後で必要になります。
- **[パスワード]** の **[アプリケーション シークレット]** を作成し、後で使用するために値を書き留めます
- アプリの **Web** プラットフォームを追加します。
- 適切な**リダイレクト URI** を入力します。リダイレクト URI は、認証の応答が送られる Azure AD を示します。このチュートリアルの既定値は `https://localhost:44326/` です。


## OWIN をインストールする
パッケージ マネージャー コンソールを使用して、`TodoList-WebApp` プロジェクトに OWIN ミドルウェア NuGet パッケージを追加します。OWIN ミドルウェアは、サインイン要求またはサインアウト要求の発行、ユーザー セッションの管理、ユーザーに関する情報の取得などを行うために使用されます。

```
PM> Install-Package Microsoft.Owin.Security.OpenIdConnect -ProjectName TodoList-WebApp
PM> Install-Package Microsoft.Owin.Security.Cookies -ProjectName TodoList-WebApp
PM> Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName TodoList-WebApp
```

## ユーザーのサインイン
[OpenID Connect 認証プロトコル](active-directory-v2-protocols.md#openid-connect-sign-in-flow)を使用するように、OWIN ミドルウェアを構成します。

-	`TodoList-WebApp` プロジェクトのルートにある `web.config` ファイルを開いて、アプリの構成値を `<appSettings>` セクションに入力します。
    -	`ida:ClientId` は、登録ポータルでアプリに割り当てられた**アプリケーション ID** です。
	- `ida:ClientSecret` は、登録ポータルで作成した**アプリケーション シークレット**です。
    -	`ida:RedirectUri` は、ポータルで入力した**リダイレクト URI** です。
- `TodoList-Service` プロジェクトのルートにある `web.config` ファイルを開き、`ida:Audience` を上記と同じ**アプリケーション ID** に置き換えます。


- `App_Start\Startup.Auth.cs` ファイルを開き、前記のライブラリの `using` ステートメントを追加します。
- 同じファイルに、`ConfigureAuth(...)` メソッドを実装します。`OpenIDConnectAuthenticationOptions` で提供されたパラメーターは、アプリが Azure AD と通信するための調整役として機能します。

```C#
public void ConfigureAuth(IAppBuilder app)
{
    app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

    app.UseCookieAuthentication(new CookieAuthenticationOptions());

    app.UseOpenIdConnectAuthentication(
        new OpenIdConnectAuthenticationOptions
        {

					// The `Authority` represents the v2.0 endpoint - https://login.microsoftonline.com/common/v2.0 
					// The `Scope` describes the permissions that your app will need.  See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/
					// In a real application you could use issuer validation for additional checks, like making sure the user's organization has signed up for your app, for instance.

					ClientId = clientId,
					Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0 "),
					Scope = "openid email profile offline_access",
					RedirectUri = redirectUri,
					PostLogoutRedirectUri = redirectUri,
					TokenValidationParameters = new TokenValidationParameters
					{
						ValidateIssuer = false,
					},

					// The `AuthorizationCodeReceived` notification is used to capture and redeem the authorization_code that the v2.0 endpoint returns to your app.

					Notifications = new OpenIdConnectAuthenticationNotifications
					{
						AuthenticationFailed = OnAuthenticationFailed,
						AuthorizationCodeReceived = OnAuthorizationCodeReceived,
					}

    	});
}
...
```

## ADAL を使用してアクセス トークンを取得する
`AuthorizationCodeReceived` 通知で、[OpenID Connect と共に OAuth 2.0](active-directory-v2-protocols.md#openid-connect-with-oauth-code-flow) を使用して、To-Do List サービスへのアクセス トークンに authorization\_code を使います。ADAL を使用すると、これを簡単に行えるようになります。

- まず、ADAL のプレビュー バージョンをインストールします。

```PM> Install-Package Microsoft.Experimental.IdentityModel.Clients.ActiveDirectory -ProjectName TodoList-WebApp -IncludePrerelease```
- ADAL 用の `App_Start\Startup.Auth.cs` ファイルにもう 1 つの `using` ステートメントを追加します。
- 新しいメソッドとして `OnAuthorizationCodeReceived` イベント ハンドラーを追加します。このハンドラーは、ADAL を使用して To Do List API へのアクセス トークンを取得し、後で使用できるように ADAL のトークン キャッシュに格納します。

```C#
private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification notification)
{
		string userObjectId = notification.AuthenticationTicket.Identity.FindFirst("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier").Value;
		string tenantID = notification.AuthenticationTicket.Identity.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid").Value;
		string authority = String.Format(CultureInfo.InvariantCulture, aadInstance, tenantID, string.Empty);
		ClientCredential cred = new ClientCredential(clientId, clientSecret);

		// Here you ask for a token using the web app's clientId as the scope, since the web app and service share the same clientId.
		var authContext = new Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext(authority, new NaiveSessionCache(userObjectId));
		var authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(notification.Code, new Uri(redirectUri), cred, new string[] { clientId });
}
...
```

- Web アプリの ADAL には、トークンの格納に使用できる拡張可能なトークン キャッシュが用意されています。このサンプルでは、http セッション ストレージを使用してトークンをキャッシュする `NaiveSessionCache` を実装します。

<!-- TODO: Token Cache article -->


## 4\.Web API の呼び出し
次に、手順 3. で取得した access\_token を実際に使用します。Web アプリの `Controllers\TodoListController.cs` ファイルを開きます。このファイルは、To-Do List API に対するすべての CRUD 要求を実行します。

- ここで再度 ADAL を使用して、ADAL キャッシュから access\_tokens を取得できます。最初に、ADAL の `using` ステートメントをこのファイルに追加します。

    `using Microsoft.Experimental.IdentityModel.Clients.ActiveDirectory;`

- `Index` アクションで ADAL の `AcquireTokenSilentAsync` メソッドを使用して access\_token を取得します。これは、To-Do List サービスからデータを読み取るのに使用できます。

```C#
...
string userObjectID = ClaimsPrincipal.Current.FindFirst("http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier").Value;
string tenantID = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid").Value;
string authority = String.Format(CultureInfo.InvariantCulture, Startup.aadInstance, tenantID, string.Empty);
ClientCredential credential = new ClientCredential(Startup.clientId, Startup.clientSecret);

// Here you ask for a token using the web app's clientId as the scope, since the web app and service share the same clientId.
AuthenticationContext authContext = new AuthenticationContext(authority, new NaiveSessionCache(userObjectID));
result = await authContext.AcquireTokenSilentAsync(new string[] { Startup.clientId }, credential, UserIdentifier.AnyUser);
...
```

- 次に、作成されたトークンを `Authorization` ヘッダーとして HTTP GET 要求に追加します。To-Do List サービスはこれを使って要求を認証します。
- To-Do List サービスから `401 Unauthorized` 応答が返された場合、ADAL の access\_tokens が何らかの理由で無効になっています。この場合、ADAL キャッシュから access\_tokens をドロップし、サインインし直す必要があることを示すメッセージをユーザーに表示する必要があります。これにより、トークンの取得フローが再起動されます。

```C#
...
// If the call failed with access denied, then drop the current access token from the cache,
// and show the user an error indicating they might need to sign-in again.
if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
{
		var todoTokens = authContext.TokenCache.ReadItems().Where(a => a.Scope.Contains(Startup.clientId));
		foreach (TokenCacheItem tci in todoTokens)
				authContext.TokenCache.DeleteItem(tci);

		return new RedirectResult("/Error?message=Error: " + response.ReasonPhrase + " You might need to sign in again.");
}
...
```

- 同様に、ADAL が何らかの理由で access\_token を返せない場合、ユーザーにサインインし直すよう指示する必要があります。これは、`AdalException` のキャッチと同様に簡単です。

```C#
...
catch (AdalException ee)
{
		// If ADAL could not get a token silently, show the user an error indicating they might need to sign in again.
		return new RedirectResult("/Error?message=An Error Occurred Reading To Do List: " + ee.Message + " You might need to log out and log back in.");
}
...
```

- まったく同じ `AcquireTokenSilentAsync` 呼び出しが、`Create` アクションと `Delete` アクションに実装されています。Web アプリでは、この ADAL メソッドを使用して、アプリで必要になったときにいつでも access\_tokens を取得できます。ADAL によりトークンが自動的に取得、キャッシュ、および更新されます。

最後に、アプリを構築して実行します。 Microsoft アカウントまたは Azure AD アカウントでサインインすると、ユーザーの ID が上部のナビゲーション バーにどのように反映されるかがわかります。ユーザーの To-Do List からいくつかの項目を追加、削除して、OAuth 2.0 の保護された API 呼び出しの動作を確認します。これで、Web アプリと Web API のいずれも業界標準のプロトコルで保護され、個人および職場/学校アカウントの両方でユーザーを認証できるようになりました。

参照用の完全なサンプル (構成値を除く) が、[ここ](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-WebAPI-OpenIdConnect-DotNet/archive/complete.zip)で提供されています。

## 次のステップ

その他のリソースについては、以下を参照してください。
- [v2.0 開発者向けガイド >>](active-directory-appmodel-v2-overview.md)
- [StackOverflow "adal" タグ >>](http://stackoverflow.com/questions/tagged/adal)

## マイクロソフト製品のセキュリティ更新プログラムを取得する

セキュリティの問題が発生したときに通知を受け取ることをお勧めします。そのためには、[このページ](https://technet.microsoft.com/security/dd252948)にアクセスし、セキュリティ アドバイザリ通知を受信登録してください。

<!---HONumber=AcomDC_0921_2016-->