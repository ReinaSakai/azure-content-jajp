<properties
   pageTitle="Service Fabric クラスターでサポートされているセキュリティ保護された接続を構成する |Microsoft Azure"
   description="Visual Studio を使用して、Azure Service Fabric クラスターでサポートされているセキュリティで保護された接続を構成する方法について説明します。"
   services="service-fabric"
   documentationCenter="na"
   authors="cawaMS"
   manager="paulyuk"
   editor="tglee" />

<tags
   ms.service="multiple"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="multiple"
   ms.date="10/08/2015"
   ms.author="cawaMS" />

# Visual Studio から Service Fabric クラスターにセキュリティ保護された接続を構成する

アクセス制御ポリシーが構成されている Azure Service Fabric クラスターに Visual Studio を使用して安全にアクセスする方法について説明します。

## クラスターの接続の種類

Azure Service Fabric クラスターでサポートされている接続には、**セキュリティ保護されていない**接続と **x509 証明書に基づいて**セキュリティ保護された接続の 2 つの種類があります (Service Fabric クラスターがオンプレミスでホストされている場合は、**Windows** 認証と **dSTS** 認証もサポートされます)。 クラスターの作成中に、クラスターの接続の種類を構成する必要があります。作成した接続の種類を後で変更することはできません。

Visual Studio Service Fabric Tools は、クラスターに接続して発行するために必要なすべての種類の認証をサポートしています。セキュリティ保護された Service Fabric クラスターをセットアップする方法については、「[Azure ポータルからの Service Fabric クラスターのセットアップ](service-fabric-cluster-creation-via-portal.md)」を参照してください。

## 発行プロファイルにクラスター接続を構成する

Visual Studio から Service Fabric プロジェクトを発行するには、**[Service Fabric アプリケーションの発行]** ダイアログ ボックスで、**[接続エンドポイント]** セクションの **[選択]** をクリックして Azure Service Fabric クラスターを選択します。Azure アカウントにサインインし、サブスクリプションで利用可能な既存のクラスターを選択できます。

![**[Service Fabric アプリケーションの発行]** ダイアログ ボックスを使用して Service Fabric 接続を構成します。][publishdialog]

**[Service Fabric クラスターの選択]** ダイアログ ボックスによって自動的にクラスターの接続が検証されます。検証に成功した場合、システムに正しい証明書がインストールされていてクラスターに安全に接続できるか、クラスターがセキュリティ保護されていないことを意味しています。検証に失敗した場合は、ネットワークに問題が発生しているか、システムが正しく構成されていないためにセキュリティ保護されたクラスターに接続できない可能性があります。

![**[Service Fabric クラスターの選択]** ダイアログ ボックスで、既存の Service Fabric クラスター接続を構成するか、新しいクラスター接続を作成して構成することができます。][selectsfcluster]

### セキュリティ保護されたクラスターに接続するには

1.	接続先のクラスターによって信頼されているクライアント証明書のいずれかにアクセスできることを確認します。証明書は通常、Personal Information Exchange (.pfx) ファイルとして共有されています。クライアントへのアクセスを許可するようにサーバーを構成する方法については、「[Azure ポータルからの Service Fabric クラスターのセットアップ](service-fabric-cluster-creation-via-portal.md)」を参照してください。

2.	信頼された証明書をインストールします。そのためには、.pfx ファイルをダブルクリックするか、PowerShell スクリプト Import-PfxCertificate を使用して証明書をインポートします。証明書を **Cert:\\LocalMachine\\My** にインストールします。証明書のインポート時、すべての既定設定をそのまま使用して問題ありません。

3.	プロジェクトのショートカット メニューで **[発行]** を選択し、**[Azure アプリケーションの公開]** ダイアログ ボックスを開いて、対象のクラスターを選択します。自動的に接続が解決され、発行プロファイルにセキュリティ保護された接続パラメーターが保存されます。

4.	[省略可能]: 発行プロファイルを編集して、セキュリティ保護されたクラスターの接続を指定できます。

    発行プロファイルの XML ファイルを手動で編集して証明書の情報を指定するため、証明書ストアの名前、ストアの場所、および証明書の拇印をメモしておいてください。後で、証明書ストアの名前とストアの場所に対して値の指定が必要になります。詳細については、[方法: 証明書のサムプリントを取得する](https://msdn.microsoft.com/library/ms734695(v=vs.110).aspx) を参照してください。

    *ClusterConnectionParameters* パラメーターを使用して、Service Fabric クラスターに接続するときに使用する PowerShell のパラメーターを指定できます。Connect-ServiceFabricCluster コマンドレットが受け取るパラメーターをすべて使用できます。使用可能なパラメーターの一覧については、「[Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx)」を参照してください。

    リモート クラスターに発行する場合は、そのクラスターに適したパラメーターを指定する必要があります。次は、セキュリティ保護されていないクラスターに接続する場合の例です。

    `<ClusterConnectionParameters ConnectionEndpoint="mycluster.westus.cloudapp.azure.com:19000" />`

    次は、x509 証明書ベースのセキュリティ保護されたクラスターに接続する場合の例です。

    ```
    <ClusterConnectionParameters
    ConnectionEndpoint="mycluster.westus.cloudapp.azure.com:19000"
    X509Credential="true"
    ServerCertThumbprint="0123456789012345678901234567890123456789"
    FindType="FindByThumbprint"
    FindValue="9876543210987654321098765432109876543210"
    StoreLocation="CurrentUser"
    StoreName="My" />
    ```

5.	アップグレード パラメーターやアプリケーション パラメーター ファイルの場所など、他の必要な設定を編集してから、Visual Studio の **[Service Fabric アプリケーションの発行]** ダイアログ ボックスからアプリケーションを発行します。

## 次のステップ
Service Fabric クラスターにアクセスする方法の詳細については、「[Service Fabric Explorer を使用したクラスターの視覚化](service-fabric-visualizing-your-cluster.md)」を参照してください。

<!--Image references-->
[publishdialog]: ./media/service-fabric-visualstudio-configure-secure-connections/publishdialog.png
[selectsfcluster]: ./media/service-fabric-visualstudio-configure-secure-connections/selectsfcluster.png

<!---HONumber=AcomDC_0114_2016-->