<properties 
   pageTitle="Azure DNS での .NET SDK を使用した DNS ゾーンとレコード セットの作成 | Microsoft Azure" 
   description="Azure DNS で、.NET SDK を使用して DNS ゾーンとレコード セットを作成する方法について説明します。" 
   services="dns" 
   documentationCenter="na" 
   authors="jtuliani" 
   manager="carmonm" 
   editor=""/>

<tags
   ms.service="dns"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services" 
   ms.date="09/19/2016"
   ms.author="jtuliani"/>


# .NET SDK を使用した DNS ゾーンとレコード セットの作成

DNS ゾーン、レコードセット、レコードを作成、削除、更新する操作は、DNS SDK と .NET DNS 管理ライブラリを使用して自動化できます。Visual Studio プロジェクト全体は、[こちら](https://www.microsoft.com/ja-JP/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True)から入手できます。

## サービス プリンシパル アカウントの作成

通常、Azure リソースへのプログラムによるアクセスは、ユーザー自身の資格情報ではなく、専用アカウント経由で許可されます。これらの専用アカウントは、"サービス プリンシパル" アカウントと呼ばれます。Azure DNS SDK のサンプル プロジェクトを使用するには、まずサービス プリンシパル アカウントを作成し、作成したアカウントに適切なアクセス許可を割り当てる必要があります。

1. [こちらの手順](../resource-group-authenticate-service-principal.md)に従い、サービス プリンシパル アカウントを作成します (Azure DNS SDK のサンプル プロジェクトではパスワードベースの認証を前提としています)。

2. リソース グループを作成 ([手順はこちら](../azure-portal/resource-group-portal.md)) します。

3. Azure RBAC を使用して、サービス プリンシパル アカウント "DNS Zone Contributor" にリソース グループへのアクセス許可を付与します ([手順はこちら](../active-directory/role-based-access-control-configure.md))。

4. Azure DNS SDK のサンプル プロジェクトを使用する場合、"program.cs" ファイルを次のように編集します。
	* 手順 1. で使用した、tenantId、clientId (アカウント ID とも呼ばれます)、シークレット (サービス プリンシパル アカウントのパスワード)、および subscriptionId の適切な値を入力します。
	* 手順 2. で選択したリソース グループ名を入力します。
	* 任意の DNS ゾーンの名前を入力します。

## NuGet パッケージと名前空間宣言

Azure DNS .NET SDK を使用するには、**Azure DNS 管理ライブラリ** NuGet パッケージとその他の必要な Azure パッケージをインストールする必要があります。
 
1. **Visual Studio** で、プロジェクトを開くか、新規作成します。

2. **[ツール]**、**[NuGet パッケージ マネージャー]**、 **[ソリューションの NuGet パッケージの管理...]** の順に移動します。

3. **[参照]** をクリックし、**[プレリリースを含める]** チェック ボックスをオンにして、検索ボックスに「**Microsoft.Azure.Management.Dns**」と入力します。

4. パッケージを選択し、**[インストール]** をクリックして Visual Studio プロジェクトに追加します。
 
5. 上記の手順を繰り返して、**Microsoft.Rest.ClientRuntime.Azure.Authentication** パッケージと **Microsoft.Azure.Management.ResourceManager** パッケージもインストールします。

## 名前空間宣言の追加

次の名前空間宣言を追加します。

	using Microsoft.Rest.Azure.Authentication;
	using Microsoft.Azure.Management.Dns;
	using Microsoft.Azure.Management.Dns.Models;

## DNS 管理クライアントの初期化

*DnsManagementClient* には、DNS ゾーンとレコードセットの管理に必要なメソッドとプロパティが含まれます。次のコードにより、サービス プリンシパル アカウントにログインし、DnsManagementClient オブジェクトを作成します。

	// Build the service credentials and DNS management client
	var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, secret);
	var dnsClient = new DnsManagementClient(serviceCreds);
	dnsClient.SubscriptionId = subscriptionId;

## DNS ゾーンを作成または更新する

DNS ゾーンを作成するには、まず、DNS ゾーン パラメーターを含む "Zone" オブジェクトを作成します。DNS ゾーンは特定のリージョンにリンクされないため、場所は "global" に設定されます。この例では、[Azure Resource Manager の "タグ"](https://azure.microsoft.com/updates/organize-your-azure-resources-with-tags/) も、ゾーンに追加されます。

Azure DNS でゾーンを実際に作成または更新するには、ゾーン パラメーターを含むゾーン オブジェクトを *DnsManagementClient.Zones.CreateOrUpdateAsyc* メソッドに渡します。

>[AZURE.NOTE] DnsManagementClient は、同期 ("CreateOrUpdate")、非同期 ("CreateOrUpdateAsync")、および HTTP 応答へのアクセスを使用した非同期 ("CreateOrUpdateWithHttpMessagesAsync") の 3 つの操作モードをサポートします。アプリケーションのニーズに応じていずれかのモードを選択できます。

Azure DNS では、[Etag](dns-getstarted-create-dnszone.md) と呼ばれるオプティミスティック同時実行制御がサポートされます。この例では、"If-None-Match" ヘッダーに "*" を指定して、DNS ゾーンが存在しない場合は DNS ゾーンを作成することを Azure DNS に通知します。指定したリソース グループ内に指定した名前を持つゾーンが既に存在する場合、呼び出しは失敗します。

	// Create zone parameters
	var dnsZoneParams = new Zone("global"); // All DNS zones must have location = "global"
	
	// Create a Azure Resource Manager 'tag'.  This is optional.  You can add multiple tags
	dnsZoneParams.Tags = new Dictionary<string, string>();
	dnsZoneParams.Tags.Add("dept", "finance");
	
	// Create the actual zone.
	// Note: Uses 'If-None-Match *' ETAG check, so will fail if the zone exists already.
	// Note: For non-async usage, call dnsClient.Zones.CreateOrUpdate(resourceGroupName, zoneName, dnsZoneParams, null, "*")
	// Note: For getting the http response, call dnsClient.Zones.CreateOrUpdateWithHttpMessagesAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*")
	var dnsZone = await dnsClient.Zones.CreateOrUpdateAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*");

## DNS レコード セットおよびレコードの作成

DNS レコードはレコード セットとして管理されます。レコード セットは、ゾーン内で名前とレコードの種類が同じレコードのセットです。レコード セットの名前は、完全修飾 DNS 名ではなく、ゾーンの名前に対して相対的です。

レコード セットを作成または更新するには、"RecordSet" パラメーター オブジェクトを作成し、*DnsManagementClient.RecordSets.CreateOrUpdateAsync* に渡します。DNS ゾーンと同様に、同期 ("CreateOrUpdate")、非同期 ("CreateOrUpdateAsync")、および HTTP 応答へのアクセスを使用した非同期 ("CreateOrUpdateWithHttpMessagesAsync") の 3 つの操作モードがあります。

DNS ゾーンと同様に、レコード セットでの操作では、オプティミスティック同時実行制御がサポートされます。この例では、"If-Match" も "If-None-Match" も指定されていないので、レコード セットは常に作成されます。この呼び出しは、この DNS ゾーン内で同じ名前とレコード タイプを持つ既存のレコード セットを上書きします。

	// Create record set parameters
	var recordSetParams = new RecordSet();
	recordSetParams.TTL = 3600;

	// Add records to the record set parameter object.  In this case, we'll add a record of type 'A'
	recordSetParams.ARecords = new List<ARecord>();
	recordSetParams.ARecords.Add(new ARecord("1.2.3.4"));

	// Add metadata to the record set.  Similar to Azure Resource Manager tags, this is optional and you can add multiple metadata name/value pairs
	recordSetParams.Metadata = new Dictionary<string, string>();
	recordSetParams.Metadata.Add("user", "Mary");

	// Create the actual record set in Azure DNS
	// Note: no ETAG checks specified, will overwrite existing record set if one exists
	var recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSetParams);

## ゾーンとレコード セットの取得

*DnsManagementClient.Zones.Get* メソッドと *DnsManagementClient.RecordSets.Get* メソッドは、個別のゾーンおよび個別のレコード セットをそれぞれ取得します。RecordSets は、その種類、名前、それが存在するゾーンとリソース グループで識別されます。ゾーンは、その名前と、それが存在するリソース グループで識別されます。

	var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);
	
## 既存のレコード セットの更新

既存の DNS レコード セットを更新するには、まずレコード セットを取得し、レコード セットのコンテンツを更新してから、変更を送信します。この例では、"If-Match" パラメーターで取得したレコード セットの "Etag" を指定します。同時実行の操作によって更新中にレコード セットが変更された場合、この呼び出しは失敗します。

	var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);

	// Add a new record to the local object.  Note that records in a record set must be unique/distinct
	recordSet.ARecords.Add(new ARecord("5.6.7.8"));

	// Update the record set in Azure DNS
	// Note: ETAG check specified, update will be rejected if the record set has changed in the meantime
	recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSet, recordSet.Etag);

## ゾーンとレコード セットの一覧表示

ゾーンを一覧表示するには、*DnsManagementClient.Zones.List...* メソッドを使用します。このメソッドを使用すると、指定したリソース グループ内のすべてのゾーンまたは指定した Azure サブスクリプション内 (リソース グループ全体) のすべてのゾーンを一覧表示することができます。 レコード セットを一覧表示するには、*DnsManagementClient.RecordSets.List...* メソッドを使用します。このメソッドを使用すると、指定したゾーン内のすべてのレコード セットを一覧表示することも、指定した種類のレコード セットのみを一覧表示することもできます。

ゾーンとレコード セットを一覧表示する場合、結果がページ分割される可能性がある点に注意してください。次の例では、結果のページを反復処理する方法を示します(意図的に "2" という小さなページ サイズを使用して、強制的にページングしています。実際にはこのパラメーターは削除し、既定のページ サイズを使用する必要があります)。

	// Note: in this demo, we'll use a very small page size (2 record sets) to demonstrate paging
	// In practice, to improve performance you would use a large page size or just use the system default
	int recordSets = 0;
	var page = await dnsClient.RecordSets.ListAllInResourceGroupAsync(resourceGroupName, zoneName, "2");
	recordSets += page.Count();

	while (page.NextPageLink != null)
	{
		page = await dnsClient.RecordSets.ListAllInResourceGroupNextAsync(page.NextPageLink);
		recordSets += page.Count();
	}

## 次のステップ

[Azure DNS .NET SDK サンプル プロジェクト](https://www.microsoft.com/ja-JP/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True)をダウンロードします。このプロジェクトには、他の DNS レコードの種類の例など、より多くの Azure DNS .NET SDK の使用方法の例が含まれています。

<!---HONumber=AcomDC_0921_2016-->