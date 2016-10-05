<properties
   pageTitle="Powershell コマンドレットを使用した VNet ピアリングの作成 | Microsoft Azure"
   description="Resource Manager で Azure ポータルを使用して仮想ネットワークを作成する方法を説明します。"
   services="virtual-network"
   documentationCenter=""
   authors="NarayanAnnamalai"
   manager="jefco"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="hero-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/14/2016"
   ms.author="narayanannamalai; annahar"/>

# Powershell コマンドレットを使用した VNet ピアリングの作成

[AZURE.INCLUDE [virtual-networks-create-vnet-selectors-arm-include](../../includes/virtual-networks-create-vnetpeering-selectors-arm-include.md)]

[AZURE.INCLUDE [virtual-networks-create-vnet-intro](../../includes/virtual-networks-create-vnetpeering-intro-include.md)]

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-basic-include](../../includes/virtual-networks-create-vnetpeering-scenario-basic-include.md)]

PowerShell を使用して VNet ピアリングを作成するには、次の手順に従います。

1. Azure PowerShell を初めて使用する場合は、[Azure PowerShell のインストールおよび構成方法](../powershell-install-configure.md)を参照し、このページにある手順をすべて最後まで実行し、Azure にサインインしてサブスクリプションを選択します。

        > [AZURE.NOTE] PowerShell cmdlet for managing VNet peering is shipped with [Azure PowerShell 1.6.](http://www.powershellgallery.com/packages/Azure/1.6.0)

2. 仮想ネットワーク オブジェクトを読み取ります。

        $vnet1 = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name vnet1
        $vnet2 = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name vnet2

3. VNET ピアリングを確立するには、2 つのリンク (各方向につき 1 つ) を作成する必要があります。まず、VNet1 から VNet2 への VNET ピアリング リンクを次の手順で作成します。

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet2 -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.id

    このコマンドの出力結果を次に示します。

        Name			: LinkToVNet2
        Id: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet1/virtualNetworkPeerings/LinkToVNet2
        Etag			: W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        ResourceGroupName	: vnet101
        VirtualNetworkName	: vnet1
        ProvisioningState		: Succeeded
        RemoteVirtualNetwork	: {
                                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet2"
                                        }
        AllowVirtualNetworkAccess	: True
        AllowForwardedTraffic	: False
        AllowGatewayTransit	: False
        UseRemoteGateways	: False
        RemoteGateways		: null
        RemoteVirtualNetworkAddressSpace : null

4. VNet2 から VNet1 への VNET ピアリング リンクを作成します。

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet1 -VirtualNetwork $vnet2 -RemoteVirtualNetworkId $vnet1.id

    このコマンドの出力結果を次に示します。

        Name			: LinkToVNet1
        Id				: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet2/virtualNetworkPeerings/LinkToVNet1
        Etag			: W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        ResourceGroupName	: vnet101
        VirtualNetworkName	: vnet2
        ProvisioningState		: Succeeded
        RemoteVirtualNetwork	: {
                                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet1"
                                        }
        AllowVirtualNetworkAccess	: True
        AllowForwardedTraffic	: False
        AllowGatewayTransit	: False
        UseRemoteGateways	: False
        RemoteGateways		: null
        RemoteVirtualNetworkAddressSpace : null

5. この VNET ピアリング リンクが作成されると、リンクの状態が次のように表示されます。

        Get-AzureRmVirtualNetworkPeering -VirtualNetworkName vnet1 -ResourceGroupName vnet101 -Name linktovnet2

    このコマンドの出力結果を次に示します。

		Name			: LinkToVNet2
		Id				: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet1/virtualNetworkPeerings/LinkToVNet2
		Etag			: W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
		ResourceGroupName	: vnet101
		VirtualNetworkName	: vnet1
		ProvisioningState		: Succeeded
		RemoteVirtualNetwork	: {
		                                     "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet2"
		                                }
		AllowVirtualNetworkAccess	: True
		AllowForwardedTraffic            : False
		AllowGatewayTransit              : False
		UseRemoteGateways                : False
		RemoteGateways                   : null
		RemoteVirtualNetworkAddressSpace : null

	VNet ピアリングには、構成可能なプロパティがいくつかあります。

	|オプション|Description|既定値|
	|:-----|:----------|:------|
	|AllowVirtualNetworkAccess|ピア VNet のアドレス空間を Virtual\_network タグの一部として含めるかどうかを選択します。|はい|
	|AllowForwardedTraffic|ピアリングされた VNet 以外の送信元のトラフィックを許可するか破棄するかを選択します。|いいえ|
	|AllowGatewayTransit|VNet ゲートウェイの使用をピア VNet に許可するかどうかを選択します。|いいえ|
	|UseRemoteGateways|ピアの VNet ゲートウェイを使用します。ピア VNET でゲートウェイが構成され、かつ AllowGatewayTransit が選択されている必要があります。ゲートウェイをローカルで構成した場合、このオプションは使用できません。|いいえ|

	上記の一連のプロパティは、VNET ピアリングの各リンクに存在します。たとえば、AllowVirtualNetworkAccess は、VNet1 から VNet2 への VNET ピアリング リンクの場合は True に、逆方向の VNET ピアリング リンクの場合は False に設定します。

        $LinktoVNet2 = Get-AzureRmVirtualNetworkPeering -VirtualNetworkName vnet1 -ResourceGroupName vnet101 -Name LinkToVNet2
        $LinktoVNet2.AllowForwardedTraffic = $true
        Set-AzureRmVirtualNetworkPeering -VirtualNetworkPeering $LinktoVNet2

    変更後、Get-AzureRmVirtualNetworkPeering を実行してプロパティの値をもう一度確認できます。上記のコマンドレットを実行した後、AllowForwardedTraffic が True に設定されていることが出力結果から確認できます。

        Name			: LinkToVNet2
        Id			: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet1/virtualNetworkPeerings/LinkToVNet2
        Etag			: W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        ResourceGroupName	: vnet101
        VirtualNetworkName	: vnet1
        ProvisioningState	: Succeeded
        RemoteVirtualNetwork	: {
                                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/vnet101/providers/Microsoft.Network/virtualNetworks/vnet2"
                                        }
        AllowVirtualNetworkAccess	: True
        AllowForwardedTraffic		: True
        AllowGatewayTransit		: False
        UseRemoteGateways		: False
        RemoteGateways		: null
        RemoteVirtualNetworkAddressSpace : null

	このシナリオでピアリングが確立された後は、双方の VNet の任意の仮想マシンから任意の仮想マシンへの接続を開始することができます。既定では AllowVirtualNetworkAccess が True に設定されているため、VNET 間の通信を許可する適切な ACL が VNET ピアリングによってプロビジョニングされます。ただし、ネットワーク セキュリティ グループ (NSG) ルールを適用して接続をブロックすることはできます。特定のサブネット間や仮想マシン間の接続をブロックすることで、2 つの仮想ネットワーク間のアクセスを細かく制御することができます。NSG ルールの作成の詳細については、こちらの[記事](virtual-networks-create-nsg-arm-ps.md)を参照してください。

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-crosssub-include](../../includes/virtual-networks-create-vnetpeering-scenario-crosssub-include.md)]

サブスクリプション間の VNET ピアリングを PowerShell で作成するには、次の手順に従います。

1. サブスクリプション A の特権 User-A アカウントで Azure にサインインし、次のコマンドレットを実行します。

        New-AzureRmRoleAssignment -SignInName <UserB ID> -RoleDefinitionName "Network Contributor" -Scope /subscriptions/<Subscription-A-ID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/VirtualNetworks/VNet5

    これは必須ではありません。ユーザーが個々の VNET に対して個別にピアリング要求を行った場合でも、双方の要求が合致すればピアリングは確立されます。相手側 VNET の特権ユーザーをローカル VNET のユーザーとして追加すると、セットアップしやすくなります。

2. サブスクリプション B の特権 User-B アカウントで Azure にサインインし、次のコマンドレットを実行します。

        New-AzureRmRoleAssignment -SignInName <UserA ID> -RoleDefinitionName "Network Contributor" -Scope /subscriptions/<Subscription-B-ID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/VirtualNetworks/VNet3

3. User-A のログイン セッションで次のコマンドレットを実行します。

        $vnet3 = Get-AzureRmVirtualNetwork -ResourceGroupName hr-vnets -Name vnet3

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet5 -VirtualNetwork $vnet3 -RemoteVirtualNetworkId "/subscriptions/<Subscriptoin-B-Id>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/virtualNetworks/VNet5" -BlockVirtualNetworkAccess

4. User-B のログイン セッションで次のコマンドレットを実行します。

        $vnet5 = Get-AzureRmVirtualNetwork -ResourceGroupName vendor-vnets -Name vnet5

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet3 -VirtualNetwork $vnet5 -RemoteVirtualNetworkId "/subscriptions/<Subscriptoin-A-Id>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/virtualNetworks/VNet3" -BlockVirtualNetworkAccess

5. ピアリングが確立されると、VNet3 内の仮想マシンが VNet5 内の仮想マシンと通信できるようになります。

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-transit-include](../../includes/virtual-networks-create-vnetpeering-scenario-transit-include.md)]

1. このシナリオでは、以下の PowerShell コマンドレットを実行することによって、VNET ピアリングを確立できます。VNET1 から HubVnet 方向のリンクに関して、AllowForwardedTraffic プロパティを True に設定する必要があります。こうすることで、ピアリング VNET のアドレス空間外からの受信トラフィックが許可されます。

        $hubVNet = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name HubVNet
        $vnet1 = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name vnet1

        Add-AzureRmVirtualNetworkPeering -name LinkToHub -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $HubVNet.id -AllowForwardedTraffic

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet1 -VirtualNetwork $HubVNet -RemoteVirtualNetworkId $vnet1.id

2. ピアリングが確立されたら、こちらの[記事](virtual-network-create-udr-arm-ps.md)を参照してください。VNet1 トラフィックを仮想アプライアンス経由でリダイレクトするようにユーザー定義ルート (UDR) を設定することで、その機能を利用することができます。ルートの次ホップ アドレスを指定するときは、ピア VNET (HubVNet) に存在する仮想アプライアンスの IP アドレスを設定します。以下に例を示します。

        $route = New-AzureRmRouteConfig -Name TestNVA -AddressPrefix 10.3.0.0/16 -NextHopType VirtualAppliance -NextHopIpAddress 192.0.1.5

        $routeTable = New-AzureRmRouteTable -ResourceGroupName VNet101 -Location brazilsouth -Name TestRT -Route $route

        $vnet1 = Get-AzureRmVirtualNetwork -ResourceGroupName VNet101 -Name VNet1

        Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet1 -Name subnet-1 -AddressPrefix 10.1.1.0/24 -RouteTable $routeTable

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet1

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-asmtoarm-include](../../includes/virtual-networks-create-vnetpeering-scenario-asmtoarm-include.md)]

クラシック仮想ネットワークと Azure Resource Manager 仮想ネットワークの間の VNet ピアリングを PowerShell で作成するには、次の手順に従います。

1. 次のようにして、Azure Resource Manager 仮想ネットワーク **VNET1** の仮想ネットワーク オブジェクトを読み取ります。$vnet1 = Get-AzureRmVirtualNetwork -ResourceGroupName vnet101 -Name vnet1

2. このシナリオで VNet ピアリングを確立するには、リンクを 1 つだけ作成します。具体的には、**VNET1** から **VNET2** にリンクを作成します。この手順では、クラシック VNet のリソース ID を知っておく必要があります。リソース グループ ID の形式は次のとおりです。/subscriptions/SubscriptionID/resourceGroups/ResourceGroupName/providers/Microsoft.ClassicNetwork/virtualNetworks/VirtualNetworkName

    SubscriptionID、ResourceGroupName、および VirtualNetworkName は適切な名前に置き換えてください。

    これを行うには、次のコマンドを実行します。

        Add-AzureRmVirtualNetworkPeering -name LinkToVNet2 -VirtualNetwork $vnet1 -RemoteVirtualNetworkId /subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.ClassicNetwork/virtualNetworks/VNET2

3. この VNet ピアリング リンクが作成されると、出力結果にリンクの状態が次のように表示されます。

        Name                             : LinkToVNet2
        Id                               : /subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.Network/virtualNetworks/VNET1/virtualNetworkPeerings/LinkToVNet2
        Etag                             : W/"acecbd0f-766c-46be-aa7e-d03e41c46b16"
        ResourceGroupName                : MyResourceGroup
        VirtualNetworkName               : VNET1
        PeeringState                     : Connected
        ProvisioningState                : Succeeded
        RemoteVirtualNetwork             : {
                                         "Id": "/subscriptions/xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.ClassicNetwork/virtualNetworks/VNET2"
                                       }
        AllowVirtualNetworkAccess        : True
        AllowForwardedTraffic            : False
        AllowGatewayTransit              : False
        UseRemoteGateways                : False
        RemoteGateways                   : null
        RemoteVirtualNetworkAddressSpace : null

## VNet ピアリングの削除

1.	VNET ピアリングを削除するには、次のコマンドレットを実行する必要があります。

        Remove-AzureRmVirtualNetworkPeering  

        remove both links, as shown below:

        Remove-AzureRmVirtualNetworkPeering -ResourceGroupName vnet101 -VirtualNetworkName vnet1 -Name linktovnet2
        Remove-AzureRmVirtualNetworkPeering -ResourceGroupName vnet101 -VirtualNetworkName vnet1 -Name linktovnet2

2. VNET ピアリングのいずれかのリンクを削除すると、ピアのリンク状態が "切断" に変わります。この状態になると、ピア リンク状態が "開始済み" に変化するまではリンクを再作成できません。両方のリンクを削除してから、VNET ピアリングを作成し直すことをお勧めします。

<!---HONumber=AcomDC_0928_2016-->