<properties 
	pageTitle="App Service 環境からバックエンド リソースへの安全な接続" 
	description="App Service 環境からバックエンド リソースに安全に接続する方法について説明します。" 
	services="app-service" 
	documentationCenter="" 
	authors="ccompy" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="07/11/2016" 
	ms.author="stefsch"/>

# App Service 環境からバックエンド リソースへの安全な接続 #

## 概要 ##
App Service 環境は常に Azure Resource Manager 仮想ネットワーク**または**クラシック デプロイ モデル[仮想ネットワーク][virtualnetwork]の**いずれか**に作成されるため、仮想ネットワーク経由でのみ、App Service 環境から他のバックエンド リソースへの送信接続を行うことができます。また、2016 年 6 月に行われた直近の変更で、パブリック アドレス範囲または RFC1918 アドレス空間 (つまりプライベート アドレス) のどちらかを使用した仮想ネットワークに ASE をデプロイできるようになりました。

たとえば、ポート 1433 がロックされている仮想マシンのクラスターで実行されている SQL Server がある場合があります。このエンドポイントは、同じ仮想ネットワークの他のリソースからのアクセスを許可する目的のみで使用されることがあります。

別の例として、機密性の高いエンドポイントがオンプレミスで実行されていて、[サイト対サイト][SiteToSite]の接続または [Azure ExpressRoute][ExpressRoute] 接続を使用して Azure に接続している場合があります。その結果、サイト対サイトまたは ExpressRoute トンネルに接続されている仮想ネットワーク内のリソースのみがオンプレミスのエンドポイントにアクセスできるようになります。

これらのすべてのシナリオで、App Service 環境で実行中のアプリが、さまざまなサーバーとリソースに安全に接続できます。App Service 環境で実行されているアプリから同じ仮想ネットワーク内の (または同じ仮想ネットワークに接続されている) プライベート エンドポイントへの送信トラフィックは、仮想ネットワーク経由でのみ行われます。プライベート エンドポイントへの送信トラフィックがパブリック インターネット経由で送信されることはありません。

App Service 環境から仮想ネットワーク内のエンドポイントへの送信トラフィックには、注意すべき点が 1 つあります。App Service 環境から、App Service 環境と**同じ**サブネットにある仮想マシンのエンドポイントに到達することはできません。App Service 環境が、App Service 環境専用として予約されているサブネットにデプロイされていれば、通常、これは問題にはなりません。

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## 発信接続と DNS の要件 ##
App Service Environment が正常に機能するためには、さまざまなエンドポイントへの発信アクセスが必要となります。[ExpressRoute を使用した環境のネットワーク構成](app-service-app-service-environment-network-configuration-expressroute.md#required-network-connectivity)の記事の「必要なネットワーク接続」というセクションに、App Service Environment で使用されるすべての外部エンドポイントが掲載されています。

App Service 環境では、仮想ネットワーク用に構成された有効な DNS インフラストラクチャが必要です。何らかの理由で、App Service Environment の作成後に DNS 構成が変わった場合、開発者は強制的に App Service Environment から新しい DNS 構成を選択することができます。ポータルの App Service 環境管理ブレードの上部にある [再起動] アイコンを使用して、ローリングする環境の再起動をトリガーすると、新しい DNS 構成が自動的に選択されます。

また、App Service 環境を作成する前に、vnet 上のカスタム DNS サーバーをセットアップしておくことをお勧めします。App Service 環境の作成中に仮想ネットワークの DNS 構成が変更された場合、App Service 環境の作成プロセスは失敗します。同様に、VPN ゲートウェイの他端にカスタム DNS サーバーが存在していて、その DNS サーバーが到達不能または使用できない場合、App Service 環境の作成プロセスも失敗します。

## SQL Server への接続
SQL Server の構成には一般的に、ポート 1433 でリッスンしているエンドポイントがあります。

![SQL Server エンドポイント][SqlServerEndpoint]

このエンドポイントへのトラフィックを制限する方法は 2 つあります。


- [ネットワーク アクセス制御リスト][NetworkAccessControlLists] \(ネットワーク ACL)

- [ネットワーク セキュリティ グループ][NetworkSecurityGroups]


## ネットワーク ACL でのアクセスの制限

ポート 1433 は、ネットワーク アクセス制御リストを使用して保護できます。次の例は、仮想ネットワーク内から送信されたクライアント アドレスをホワイトリストに登録し、その他のすべてのクライアントへのアクセスをブロックします。

![ネットワーク アクセス制御リストの例][NetworkAccessControlListExample]

SQL Server と同じ仮想ネットワークの App Service 環境で実行されているすべてのアプリケーションが、SQL Server 仮想マシンの **VNet 内部** IP アドレスを使用して SQL Server インスタンスに接続できるようになります。

次の例の接続文字列は、プライベート IP アドレスを使用して SQL Server を参照します。

    Server=tcp:10.0.1.6;Database=MyDatabase;User ID=MyUser;Password=PasswordHere;provider=System.Data.SqlClient

仮想マシンにはパブリック エンドポイントもありますが、ネットワーク ACL によって、パブリック IP アドレスを使用した接続の試行は拒否されます。

## ネットワーク セキュリティ グループを使用したアクセスの制限
アクセスを保護する方法にはほかに、ネットワーク セキュリティ グループを使用する方法があります。ネットワーク セキュリティ グループは、個々の仮想マシン、または仮想マシンを含むサブネットに適用できます。

まず、ネットワーク セキュリティ グループを作成する必要があります。

    New-AzureNetworkSecurityGroup -Name "testNSGexample" -Location "South Central US" -Label "Example network security group for an app service environment"

VNet 間の内部トラフィックのみにアクセスを制限すると、ネットワーク セキュリティ グループをごく単純に使用できます。ネットワーク セキュリティ グループの既定の規則では、同じ仮想ネットワーク内の他のネットワーク クライアントからのアクセスのみが許可されます。

その結果、SQL Server へのアクセスのロックは、SQL Server を実行している仮想マシンまたはその仮想マシンを含むサブネットに対する既定の規則を使用してネットワーク セキュリティ グループを適用するくらい単純になります。

次の例では、ネットワーク セキュリティ グループを、このグループを含むサブネットに適用します。

    Get-AzureNetworkSecurityGroup -Name "testNSGExample" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-1'
    
最終的に、VNet 内部のアクセスを許可し、外部アクセスをブロックするセキュリティの規則の組み合わせになります。

![既定のネットワーク セキュリティ グループ][DefaultNetworkSecurityRules]


## 使用の開始
App Service 環境に関するすべての記事と作業方法は [App Service 環境の README](../app-service/app-service-app-service-environments-readme.md) を参照してください。

App Service 環境の使用を開始するには、「[App Service 環境の概要][IntroToAppServiceEnvironment]」を参照してください。

App Service 環境への着信トラフィックを制御する方法の詳細については、[App Service 環境への着信トラフィックの制御][ControlInboundASE]に関するページを参照してください。

Azure App Service プラットフォームの詳細については、[Azure App Service][AzureAppService] に関するページを参照してください。

[AZURE.INCLUDE [app-service-web-whats-changed](../../includes/app-service-web-whats-changed.md)]

[AZURE.INCLUDE [app-service-web-try-app-service](../../includes/app-service-web-try-app-service.md)]
 

<!-- LINKS -->
[virtualnetwork]: https://azure.microsoft.com/documentation/articles/virtual-networks-faq/
[ControlInboundTraffic]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-control-inbound-traffic/
[SiteToSite]: https://azure.microsoft.com/documentation/articles/vpn-gateway-site-to-site-create/
[ExpressRoute]: http://azure.microsoft.com/services/expressroute/
[NetworkAccessControlLists]: https://azure.microsoft.com/documentation/articles/virtual-networks-acl/
[NetworkSecurityGroups]: https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[IntroToAppServiceEnvironment]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-intro/
[AzureAppService]: http://azure.microsoft.com/documentation/articles/app-service-value-prop-what-is/
[ControlInboundASE]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-control-inbound-traffic/

<!-- IMAGES -->
[SqlServerEndpoint]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/SqlServerEndpoint01.png
[NetworkAccessControlListExample]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/NetworkAcl01.png
[DefaultNetworkSecurityRules]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/DefaultNetworkSecurityRules01.png

<!---HONumber=AcomDC_0713_2016-->