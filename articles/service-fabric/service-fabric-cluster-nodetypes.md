<properties
   pageTitle="Service Fabric ノードの種類と VM スケール セット | Microsoft Azure"
   description="Service Fabric のノードの種類を VM スケール セットに関連付ける方法と、VM スケール セットのインスタンスまたはクラスター ノードにリモート接続する方法について説明します。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/09/2016"
   ms.author="chackdan"/>


# Service Fabric ノードの種類と仮想マシン スケール セットの関係

仮想マシン スケール セットは、セットとして仮想マシンのコレクションをデプロイおよび管理するために使用できる Azure 計算リソースです。Service Fabric クラスターで定義されているすべてのノードの種類は、個別の VM スケール セットとしてセットアップされます。各ノードの種類は、個別にスケール アップまたはスケール ダウンすることができ、さまざまなセットのポートを開き、異なる容量のメトリックスを持つことができます。

次のスクリーン ショットでは、ノードが 2 種類あるクラスターを示します (FrontEnd と BackEnd)。各ノードの種類に、5 つのノードがあります。

![ノードが 2 種類あるクラスターのスクリーン ショット][NodeTypes]

## VM スケール セットのインスタンスをノードにマッピング

上述のように、VM スケール セットのインスタンスは、インスタンス 0 から始まり増えていきます。番号設定は名前に反映されます。たとえば、BackEnd_0 は、BackEnd VM スケール セットのインスタンス 0 です。この特定の VM スケール セットには、BackEnd_0、BackEnd1、BackEnd2、BackEnd3、BackEnd4 という名前の 5 つのインスタンスがあります。

VM スケール セットをスケール アップすると、新しいインスタンスが作成されます。通常、新しい VM スケール セットのインスタンス名は、VM スケール セットの名前 + 次のインスタンス番号になります。この例では、BackEnd\_5 です。


## VM スケール セットのロード バランサーを各ノードの種類/VM スケール セットにマッピングする

ポータルからクラスターをデプロイした場合、または提供しているサンプル Resource Manager テンプレートを使用した場合、リソース グループ下のすべてのリソースの一覧を取得すると、VM スケール セットまたはノードの種類ごとにロード バランサーが表示されます。

名前は次のようになります: **LB-&lt;NodeType name&gt;**。たとえば、次のスクリーンショットで示すように、LB-sfcluster4doc-0 などです。


![リソース][Resources]


## VM スケール セットのインスタンスまたはクラスター ノードへのリモート接続
クラスターで定義されているすべてのノードの種類は、個別の VM スケール セットとしてセットアップされます。つまり、ノードの種類は個別にスケール アップまたはスケール ダウンすることができ、異なる VM SKU で作ることができます。単一のインスタンス VM とは異なり、VM スケール セットのインスタンスは、独自の仮想 IP アドレスを取得しません。そのため、特定のインスタンスにリモート接続するために使用できる IP アドレスとポートを検索するときに、少し難しい場合があります。

IP アドレスとポートを検出する手順を次に示します。

### 手順 1: ノードの種類に仮想 IP アドレスを検索し、RDP に受信 NAT 規則を検索する

これを取得するには、**Microsoft.Network/loadBalancers** のリソース定義の一部として定義された受信 NAT 規則の値を取得する必要があります。

ポータルで、[ロード バランサー] ブレード、**[設定]** の順に移動します。

![LBBlade][LBBlade]


**[設定]** で、**[受信 NAT 規則]** をクリックします。ここでは、最初の VM スケール セットのインスタンスにリモート接続するために使用できる IP アドレスとポートを提供します。次のスクリーンショットでは、**104.42.106.156** と **3389** です

![NATRules][NATRules]

### 手順 2: 特定の VM スケール セットのインスタンス/ノードにリモート接続するために使用できるポートを検索する

このドキュメントの前半では、VM スケール セットのインスタンスをノードにマップする方法について説明しました。正確なポートを見つけるためにこのマッピングを使用します。

ポートは、VM スケール セットのインスタンスの昇順で割り当てられます。そのため、FrontEnd のノードの種類の例では、5 つのインスタンスの各ポートは次のようになります。ここでは、VM スケール セットのインスタンスと同じマッピングを行う必要があります。

|**VM スケール セットのインスタンス**|**ポート**|
|-----------------------|--------------------------|
|FrontEnd\_0|3389|
|FrontEnd\_1|3390|
|FrontEnd\_2|3391|
|FrontEnd\_3|3392|
|FrontEnd\_4|3393|
|FrontEnd\_5|3394|


### 手順 3: 特定の VM スケール セットのインスタンスにリモート接続する

次のスクリーンショットでは、リモート デスクトップ接続を使用して、FrontEnd\_1 に接続します。

![RDP][RDP]

## RDP ポート範囲の値を変更する方法

### クラスター デプロイの前

Resource Manager テンプレートを使用してクラスターを設定している場合、**inboundNatPools** の範囲を指定できます。

**Microsoft.Network/loadBalancers** のリソース定義に移動します。その下に、**inboundNatPools** の説明があります。*frontendPortRangeStart* と *frontendPortRangeEnd* の値を置き換えます。

![InboundNatPools][InboundNatPools]


### クラスター デプロイの後
これは少し複雑で、VM がリサイクルされる可能性があります。Azure PowerShell を使用して、新しい値を設定する必要があります。Azure PowerShell 1.0 以降がコンピューターにインストールされていることを確認します。まだインストールされていない場合は、「[Azure PowerShell のインストールおよび構成方法](../powershell-install-configure.md)」の手順を実行することを強くお勧めします。

Azure アカウントにサインインします。この PowerShell コマンドが何らかの理由で失敗する場合、Azure PowerShell が正しくインストールされているかどうかを確認することをお勧めします。

```
Login-AzureRmAccount
```

次を実行して、ロード バランサーに関する詳細を取得すると、**inboundNatPools** の説明の値が表示されます。

```
Get-AzureRmResource -ResourceGroupName <RGname> -ResourceType Microsoft.Network/loadBalancers -ResourceName <load balancer name>
```

*frontendPortRangeEnd* と *frontendPortRangeStart* を必要な値に設定します。

```
$PropertiesObject = @{
	#Property = value;
}
Set-AzureRmResource -PropertyObject $PropertiesObject -ResourceGroupName <RG name> -ResourceType Microsoft.Network/loadBalancers -ResourceName <load Balancer name> -ApiVersion <use the API version that get returned> -Force
```


## 次のステップ

- ["任意の場所にデプロイ" 機能の概要と Azure で管理されるクラスターとの比較](service-fabric-deploy-anywhere.md)
- [クラスターのセキュリティ](service-fabric-cluster-security.md)
- [Service Fabric SDK と概要](service-fabric-get-started.md)


<!--Image references-->
[NodeTypes]: ./media/service-fabric-cluster-nodetypes/NodeTypes.png
[Resources]: ./media/service-fabric-cluster-nodetypes/Resources.png
[InboundNatPools]: ./media/service-fabric-cluster-nodetypes/InboundNatPools.png
[LBBlade]: ./media/service-fabric-cluster-nodetypes/LBBlade.png
[NATRules]: ./media/service-fabric-cluster-nodetypes/NATRules.png
[RDP]: ./media/service-fabric-cluster-nodetypes/RDP.png

<!---HONumber=AcomDC_0921_2016-->