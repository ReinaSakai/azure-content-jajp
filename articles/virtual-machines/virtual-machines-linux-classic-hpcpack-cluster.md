<properties
 pageTitle="HPC Pack クラスター内の Linux コンピューティング VM | Microsoft Azure"
 description="Linux ハイ パフォーマンス コンピューティング (HPC) ワークロード用の HPC Pack クラスターを Azure で作成して管理する方法を説明します。"
 services="virtual-machines-linux"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management,azure-resource-manager,hpc-pack"/>
<tags
 ms.service="virtual-machines-linux"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="vm-linux"
 ms.workload="big-compute"
 ms.date="03/21/2016"
 ms.author="danlep"/>

# Azure の HPC Pack クラスターで Linux コンピューティング ノードの使用を開始する

Windows Server を実行するヘッド ノードとサポートされる Linux ディストリビューションを実行するコンピューティング ノードを含む Microsoft HPC Pack クラスターを Azure に設定します。クラスターの Linux ノードと Windows ヘッド ノードの間でデータを移動するオプションを調べます。クラスターに Linux HPC ジョブを送信する方法について説明します。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)].


これから作成して使用する HPC Pack クラスターの概要は次の図のようになります。

![HPC Pack クラスターと Linux ノード][scenario]

>[AZURE.TIP]オンプレミスの HPC Pack クラスターでの Linux ノードの使用については、[TechNet のガイダンス](https://technet.microsoft.com/library/mt595803.aspx)を参照してください。

## Linux コンピューティング ノードと共に HPC Pack クラスターをデプロイします。

Linux コンピューティング ノードで Azure に HPC Pack クラスターを作成する 2 つの推奨される方法を次に示します。

* **Azure リソース マネージャー テンプレート** - Azure Marketplace のテンプレートまたはコミュニティ ギャラリーのテンプレートを使用して、リソース マネージャー デプロイ モデルでのクラスターの作成を自動化します。たとえば、Azure Marketplace の [HPC Pack cluster for Linux workloads](https://azure.microsoft.com/marketplace/partners/microsofthpc/newclusterlinuxcn/) テンプレートは、Azure Virtual Network、ヘッド ノードとローカル SQL データベース、Active Directory ドメイン フォレスト (およびドメイン コントローラーとして設定されたヘッド ノード)、およびサポートされる Linux ディストリビューションを実行する複数のコンピューティング ノードを含む、完全な HPC Pack クラスターを作成します。

* **PowerShell スクリプト** - [Microsoft HPC Pack IaaS デプロイ スクリプト](virtual-machines-windows-classic-hpcpack-cluster-powershell-script.md) (**New-HpcIaaSCluster.ps1**) を使用して、クラシック デプロイ モデルのクラスター デプロイを自動化します。この Azure PowerShell スクリプトは迅速なプロイメントのために Azure Marketplace の HPC Pack VM イメージを使用します。包括的な構成パラメーターを備え、簡単かつ柔軟なデプロイを可能にします。このスクリプトにより、Azure Virtual Network 、ストレージ アカウント、Cloud Services、ドメイン コントローラー、任意の個別の SQL Server Database サーバー、クラスター ヘッド ノード、コンピューティング ノード、ブローカー ノード、Azure PaaS ("バースト") ノード、Linux コンピューティング ノードがデプロイされます。

HPC Pack クラスター デプロイ オプションの概要については、「[HPC Pack 2012 R2 と HPC Pack 2012 の入門ガイド](https://technet.microsoft.com/library/jj884144.aspx)」および「[Microsoft HPC Pack を使用して Azure でハイパフォーマンス コンピューティング (HPC) クラスターを作成および管理するためのオプション](virtual-machines-linux-hpcpack-cluster-options.md)」を参照してください。

### 前提条件

* **Azure サブスクリプション** - Azure Global または Azure China サービスのサブスクリプションを利用できます。アカウントがない場合は、[無料アカウント](https://azure.microsoft.com/pricing/free-trial/)を数分で作成できます。

* **コア クォータ** - 場合によっては、コアのクォータを増やす必要があります。特に、マルチコア VM サイズのクラスター ノードをいくつかデプロイする場合に必要となる可能性があります。クォータを増やすには、[オンライン カスタマー サポートに申請](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/) (無料) してください。

* **Linux ディストリビューション** - 現在、HPC Pack がコンピューティング ノード用にサポートしている Linux ディストリビューションは、Ubuntu Server 14.04、CentOS 6.6 または 7.0、Red Hat Enterprise Linux 6.7 または 7.2、SUSE Linux Enterprise Server 12 です。使用可能な場合はこれらのディストリビューションの Marketplace バージョンを使用できます。または、ユーザーが自分で用意することもできます。

    >[AZURE.TIP]サイズ A8 および A9 のコンピューティング ノード VM で Azure RDMA ネットワークを使用するには、Marketplace で SUSE Linux Enterprise Server 12 – Optimized for High Performance Compute イメージを指定します。アプリケーションのニーズに基づいてクラスターをデプロイした後、サポートされている MPI ライブラリをノードにインストールして構成する必要があります。たとえば、「[Azure の Linux RDMA クラスター上で Microsoft HPC Pack を使用して OpenFOAM を実行する](virtual-machines-linux-classic-hpcpack-cluster-openfoam.md)」を参照してください。

HPC Pack IaaS デプロイ スクリプトを使用してクラスターをデプロイする場合のその他の前提を条件:

* **クライアント コンピューター** - クラスター デプロイ スクリプトを実行するには、Windows ベースのクライアント コンピューターが必要です。

* **Azure PowerShell** - クライアント コンピューターに [Azure PowerShell をインストールして構成します](../powershell-install-configure.md) (バージョン 0.8.10 以降)。

* **HPC Pack IaaS デプロイ スクリプト** - [Microsoft Download Center](https://www.microsoft.com/download/details.aspx?id=44949) から最新版のスクリプトをダウンロードし、解凍します。`New-HPCIaaSCluster.ps1 –Version` を実行すると、スクリプトのバージョンを確認できます。この記事はバージョン 4.4.0 以降のスクリプトに基づきます。

### デプロイ シナリオ 1:Marketplace テンプレートを使用する

1. Azure Marketplace で [HPC Pack cluster for Linux workloads](https://azure.microsoft.com/marketplace/partners/microsofthpc/newclusterlinuxcn/) テンプレートに移動し、**[デプロイ]** をクリックします。

2. Azure ポータルで情報を確認し、**[作成]** をクリックします。

    ![ポータルの作成][portal]

3. **[基本]** ブレードで、クラスターの名前を入力します。これは、ヘッド ノード VM の名前にもなります。デプロイ用には、既存のリソース グループを選択することも、新しいリソース グループを作成することもできます。

4. 初めてのデプロイの場合、**[ヘッド ノードの設定]** ブレードでは、通常、既定の設定をそのまま使用できます。

    >[AZURE.NOTE]**[構成後スクリプト URL]** は、稼働後のヘッド ノード VM で実行するパブリックに利用可能な Windows PowerShell スクリプトを指定するオプションの設定です。
    
5. **[コンピューティング ノードの設定]** ブレードで、ノードの名前付けパターン、ノードの数とサイズ、デプロイする Linux ディストリビューションのイメージを選択します。

6. **[インフラストラクチャの設定]** ブレードで、クラスターの仮想ネットワークと Active Directory ドメインの名前、ドメインおよび VM の管理者の資格情報、クラスターに必要なストレージ アカウントの名前付けパターンを入力します。

    >[AZURE.NOTE]HPC Pack は、Active Directory ドメインを使用してクラスターのユーザーを認証します。

7. 検証テストを実行し、デプロイの準備ができたら、**[作成]** をクリックします。


### デプロイ シナリオ 2:IaaS デプロイ スクリプトを使用する

HPC Pack IaaS デプロイ スクリプトは HPC クラスターのインフラストラクチャについて記載された XML 構成ファイルを入力として使用します。次のサンプル構成ファイルは、1 つのヘッド ノードとサイズ A7 の 2 つの CentOS 7 Linux コンピューティング ノードで構成される小さいクラスターをデプロイします。必要であれば、環境および目的のクラスター構成に応じてファイルを変更します。構成ファイルの要素の詳細については、スクリプト フォルダーにある Manual.rtf ファイルおよび「[Create a high performance computing (HPC) cluster with Windows VMs with the HPC Pack IaaS deployment script (HPC Pack IaaS デプロイ スクリプトを使用した Windows VM でのハイ パフォーマンス コンピューティング (HPC) クラスターの作成)](virtual-machines-windows-classic-hpcpack-cluster-powershell-script.md)」参照してください。

```
<?xml version="1.0" encoding="utf-8" ?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>Subscription-1</SubscriptionName>
    <StorageAccount>allvhdsje</StorageAccount>
  </Subscription>
  <Location>Japan East</Location>  
  <VNet>
    <VNetName>centos7rdmavnetje</VNetName>
    <SubnetName>CentOS7RDMACluster</SubnetName>
  </VNet>
  <Domain>
    <DCOption>HeadNodeAsDC</DCOption>
    <DomainFQDN>hpc.local</DomainFQDN>
  </Domain>
  <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>CentOS7RDMA-HN</VMName>
    <ServiceName>centos7rdma-je</ServiceName>
  <VMSize>A4</VMSize>
  <EnableRESTAPI />
  <EnableWebPortal />
  </HeadNode>
  <LinuxComputeNodes>
    <VMNamePattern>CentOS7RDMA-LN%1%</VMNamePattern>
    <ServiceName>centos7rdma-je</ServiceName>
    <VMSize>A7</VMSize>
    <NodeCount>2</NodeCount>
    <ImageName>5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-70-20150325</ImageName>
  </LinuxComputeNodes>
</IaaSClusterConfig>
```

**HPC Pack IaaS デプロイ スクリプトを実行するには**

1. 管理者としてクライアント コンピューターで PowerShell を開きます。

2. ディレクトリをスクリプト フォルダー (この例では、E:\\IaaSClusterScript) に変更します。

    ```
    cd E:\IaaSClusterScript
    ```

3. 次のコマンドを実行し、HPC Pack クラスターをデプロイします。この例では、構成ファイルは E:\\HPCDemoConfig.xml にあるものと想定しています。

    ```
    .\New-HpcIaaSCluster.ps1 –ConfigFile E:\HPCDemoConfig.xml –AdminUserName MyAdminName
    ```

    このスクリプトでは、**-LogFile** パラメーターが指定されていないため、ログ ファイルが自動的に生成されます。ログはリアルタイムでは書き込まれず、検証とデプロイの最後に収集されます。スクリプトの実行中に PowerShell プロセスが停止した場合、一部のログが失われます。

    a.上記のコマンドには **AdminPassword** が指定されていないため、ユーザー *MyAdminName* のパスワードを入力するように促されます。

    b.スクリプトにより構成ファイルの検証が開始されます。ネットワーク接続の状態にもよりますが、数秒から数分かかります。

    ![検証][validate]

    c.検証に合格すると、HPC クラスターに作成されるリソースが一覧表示されます。「*Y*」と入力して続行します。

    ![リソース][resources]

    d.スクリプトにより HPC Pack クラスターのデプロイが開始され、ユーザーの介入なしで構成が完了します。これには数分かかる場合があります。

    ![デプロイ][deploy]

## ヘッド ノードに接続する

デプロイが正常に完了した後、クラスターをデプロイしたときに指定したドメイン資格情報 (たとえば、*hpc\\clusteradmin*) を使用して、[リモート デスクトップでヘッド ノードに接続](virtual-machines-windows-connect-logon.md)します。

ヘッド ノードで、HPC クラスター マネージャーを起動して、HPC Pack クラスターの状態を確認します。Windows コンピューティング ノードの場合と同じように Linux コンピューティング ノードを管理し、監視できます。たとえば、Linux ノードは **[リソース管理]** に表示されます (これらのノードは **LinuxNode** テンプレートでデプロイされます)。

![ノード管理][management]

**[ヒート マップ]** ビューにも Linux ノードが表示されます。

![ヒート マップ][heatmap]

## Linux ノードのあるクラスターにデータを移動する方法

クラスターの Linux ノードと Windows ヘッド ノードの間でデータを移動するにはいくつかの方法があります。次は 3 つの一般的な方法です。

* **Azure ファイル** - Azure Storage にデータ ファイルを格納する管理された SMB ファイル共有を公開します。デプロイされている仮想ネットワークが異なる場合でも、Windows ノードと Linux ノードの両方でドライブまたはフォルダーとして同時に Azure File 共有をマウントできます。

* **ヘッド ノード SMB 共有** - Linux ノードにヘッド ノードの標準 Windows 共有フォルダーをマウントします。

* **ヘッド ノード NFS サーバー** - Windows と Linux の混在環境にファイル共有ソリューションを提供します。

### Azure File ストレージ

[Azure File](https://azure.microsoft.com/services/storage/files/) サービスは、標準の SMB 2.1 プロトコルを使用してファイル共有を公開します。Azure の VM とクラウド サービスでは、マウントされている共有を介して、アプリケーション コンポーネント間でファイル データを共有できます。また、オンプレミスのアプリケーションでは、File ストレージ API を介して、共有内のファイル データにアクセスできます。

Azure File 共有を作成してヘッド ノードにマウントする詳細な手順については、「[Windows で Azure File Storage を使用する](../storage/storage-dotnet-how-to-use-files.md)」を参照してください。Linux ノードで Azure File 共有をマウントするには、「[Linux で Azure File Storage を使用する方法](../storage/storage-how-to-use-files-linux.md)」を参照してください。固定接続を設定するには、「[Microsoft Azure Files への接続の保持](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)」を参照してください。

この例では、ストレージ アカウント allvhdsje に「rdma」という名前の Azure File 共有を作成します。ヘッド ノードに共有をマウントするには、コマンド プロンプトを開き、次のコマンドを入力します。

```
> cmdkey /add:allvhdsje.file.core.windows.net /user:allvhdsje /pass:<storageaccountkey>
> net use Z: \\allvhdje.file.core.windows.net\rdma /persistent:yes
```

この例では、allvhdsje はストレージ アカウントの名前、storageaccountkey はストレージ アカウントのキー、rdma は Azure File 共有名になります。Azure File 共有はヘッド ノードの Z: にマウントされます。

Linux ノードに Azure File 共有をマウントするには、ヘッド ノードで **clusrun** コマンドを実行します。**[Clusrun](https://technet.microsoft.com/library/cc947685.aspx)** は、複数のノードで管理タスクを実行するための便利な HPC Pack ツールです (この記事の「[Linux ノードの CLusrun](#CLusrun-for-Linux-nodes)」も参照してください)。

Windows PowerShell ウィンドウを開き、次のコマンドを入力します。

```
PS > clusrun /nodegroup:LinuxNodes mkdir -p /rdma

PS > clusrun /nodegroup:LinuxNodes mount -t cifs //allvhdsje.file.core.windows.net/rdma /rdma -o vers=2.1`,username=allvhdsje`,password=<storageaccountkey>'`,dir_mode=0777`,file_mode=0777
```

最初のコマンドで「/rdma」という名前のフォルダーが LinuxNodes グループのすべてのノードで作成されます。2 つ目のコマンドにより、ディレクトリとファイル モードのビットが「777」に設定された「/rdma」フォルダーに Azure File 共有 allvhdsjw.file.core.windows.net/rdma がマウントされます。2 つ目のコマンドで、allvhdsje はストレージ アカウント名で、storageaccountkey はストレージ アカウント キーです。

>[AZURE.NOTE]2 番目のコマンドの "`" 記号は、PowerShell のエスケープ記号です。"`," は "," (コンマ) がコマンドの一部であることを意味します。

### ヘッド ノード共有

または、Linux ノードにヘッド ノードの共有フォルダーをマウントできます。これはファイルを共有する最も簡単な方法です。ただし、ヘッド ノードとすべての Linux ノードを同じ仮想ネットワークにデプロイする必要があります。手順は次のようになります。

1. ヘッド ノードでフォルダーを作成し、読み書き権限を持つ「全員」と共有します。たとえば、ヘッド ノードの D:\\OpenFOAM を \\CentOS7RDMA-HN\\OpenFOAM として共有します。ここでは CentOS7RDMA-HN がヘッド ノードのホスト名です。

    ![ファイル共有のアクセス許可][fileshareperms]

    ![ファイル共有][filesharing]

2. Windows PowerShell ウィンドウを開き、次のコマンドを実行し、共有フォルダーをマウントします。

```
PS > clusrun /nodegroup:LinuxNodes mkdir -p /openfoam

PS > clusrun /nodegroup:LinuxNodes mount -t cifs //CentOS7RDMA-HN/OpenFOAM /openfoam -o vers=2.1`,username=<username>`,password='<password>'`,dir_mode=0777`,file_mode=0777
```

最初のコマンドで「/openfoam」という名前のフォルダーが LinuxNodes グループのすべてのノードで作成されます。2 つ目のコマンドにより、ディレクトリとファイル モードのビットが「777」に設定されたフォルダーに共有フォルダー //CentOS7RDMA-HN/OpenFOAM がマウントされます。コマンド内のユーザー名とパスワードは、ヘッド ノードのクラスター ユーザーのユーザー名とパスワードにする必要があります。(「[ユーザーと管理者の追加または削除](https://technet.microsoft.com/library/ff919330.aspx)」を参照してください。)

>[AZURE.NOTE]2 番目のコマンドの "`" 記号は、PowerShell のエスケープ記号です。"`," は "," (コンマ) がコマンドの一部であることを意味します。


### NFS サーバー

NFS サービスでは、SMB プロトコルを利用して Windows Server 2012 オペレーティング システムを実行するコンピューターと NFS プロトコルを利用する Linux ベースのコンピューターの間でファイルを共有し、移行できます。NFS サーバーとその他のすべてのノードを同じ仮想ネットワークにデプロイする必要があります。SMB 共有と比較すると、Linux との互換性がよくなります。たとえば、ファイル リンクがサポートされます。

1. NFS サーバーをインストールして設定するには、「[Server for Network File System First Share End-to-End (ネットワーク ファイル システムのエンド ツー エンドの最初の共有のためのサーバー)](http://blogs.technet.com/b/filecab/archive/2012/10/08/server-for-network-file-system-first-share-end-to-end.aspx)」の手順に従います。

    たとえば、次のプロパティを持つ「nfs」という名前の NFS 共有を作成します。

    ![NFS 認証][nfsauth]

    ![NFS 共有のアクセス許可][nfsshare]

    ![NFS NTFS アクセス許可][nfsperm]

    ![NFS 管理のプロパティ][nfsmanage]

2. Windows PowerShell ウィンドウを開き、次のコマンドを実行して NFS 共有をマウントします。

  ```
  PS > clusrun /nodegroup:LinuxNodes mkdir -p /nfsshare
  PS > clusrun /nodegroup:LinuxNodes mount CentOS7RDMA-HN:/nfs /nfsshared
  ```

  最初のコマンドで「/nfsshared」という名前のフォルダーが LinuxNodes グループのすべてのノードで作成されます。2 つ目のコマンドにより、NFS 共有 CentOS7RDMA-HN:/nfs がフォルダーにマウントされます。ここで、CentOS7RDMA-HN:/nfs は NFS 共有のリモート パスです。

## ジョブの送信方法
いくつかの方法で HPC Pack クラスターにジョブを送信できます。

* HPC Cluster Manager または HPC Job Manager GUI

* HPC Web ポータル

* REST API

HPC Pack GUI ツールと HPC Web ポータル経由で Azure のクラスターにジョブを送信する方法は Windows コンピューティング ノードの場合と同じです。「[HPC ジョブ マネージャー](https://technet.microsoft.com/library/ff919691.aspx)」および「[Submit HPC jobs from an on-premises computer to an HPC Pack cluster deployed in Azure (オンプレミスのコンピューターから Azure にデプロイされた HPC Pack クラスターに HPC ジョブを送信する)](virtual-machines-windows-hpcpack-cluster-submit-jobs.md)」を参照してください。

REST API でジョブを送信する方法については、「[Creating and Submitting Jobs by Using the REST API in Microsoft HPC Pack](http://social.technet.microsoft.com/wiki/contents/articles/7737.creating-and-submitting-jobs-by-using-the-rest-api-in-microsoft-hpc-pack-windows-hpc-server.aspx)」を参照してください。Linux クライアントからジョブを送信する場合は、[HPC Pack SDK](https://www.microsoft.com/download/details.aspx?id=47756) の Python サンプルを参照してください。

## Linux ノードの clusrun

HPC Pack **clusrun** ツールを使用して、コマンド プロンプトまたは HPC クラスター マネージャーを介して Linux ノードでコマンドを実行できます。基本的な例の一部を次に示します。

* クラスター内のすべてのノードの現在のユーザー名を表示します。

    ```
    > clusrun whoami
    ```

* linuxnodes グループ内のすべてのノードに **gdb** デバッガー ツールを **yum** でインストールし、10 分後にノードを再起動します。

    ```
    > clusrun /nodegroup:linuxnodes yum install gdb –y; shutdown –r 10
    ```

* クラスター内の各 Linux ノードに 1 ～ 10 の番号を 1 秒間表示するシェル スクリプトを作成して実行すると、ノードからの出力がすぐに表示されます。

    ```
    > clusrun /interleaved /nodegroup:linuxnodes echo "for i in {1..10}; do echo \\"\$i\\"; sleep 1; done" ^> script.sh; chmod +x script.sh; ./script.sh
    ```

>[AZURE.NOTE] 場合によっては、**clusrun** コマンドで特定のエスケープ文字を使用する必要があります。この例に示すように、コマンド プロンプトで ^ を使用すると、">" 記号がエスケープされます。

## 次のステップ

* ノード数を増やしてクラスターをスケール アップするか、クラスター上で Linux のワークロードの実行を試行します。例については、[Azure の Linux コンピューティング ノード上で Microsoft HPC Pack を使用して NAMD を実行する](virtual-machines-linux-classic-hpcpack-cluster-namd.md)方法に関するページを参照してください。

* MPI のワークロードの実行に、[A8 または A9](virtual-machines-windows-a8-a9-a10-a11-specs.md) のサイズのコンピューティング ノードを試行します。たとえば、「[Azure の Linux RDMA クラスター上で Microsoft HPC Pack を使用して OpenFOAM を実行する](virtual-machines-linux-classic-hpcpack-cluster-openfoam.md)」を参照してください。

<!--Image references-->
[scenario]: ./media/virtual-machines-linux-classic-hpcpack-cluster/scenario.png
[portal]: ./media/virtual-machines-linux-classic-hpcpack-cluster/portal.png
[validate]: ./media/virtual-machines-linux-classic-hpcpack-cluster/validate.png
[resources]: ./media/virtual-machines-linux-classic-hpcpack-cluster/resources.png
[deploy]: ./media/virtual-machines-linux-classic-hpcpack-cluster/deploy.png
[management]: ./media/virtual-machines-linux-classic-hpcpack-cluster/management.png
[heatmap]: ./media/virtual-machines-linux-classic-hpcpack-cluster/heatmap.png
[fileshareperms]: ./media/virtual-machines-linux-classic-hpcpack-cluster/fileshare1.png
[filesharing]: ./media/virtual-machines-linux-classic-hpcpack-cluster/fileshare2.png
[nfsauth]: ./media/virtual-machines-linux-classic-hpcpack-cluster/nfsauth.png
[nfsshare]: ./media/virtual-machines-linux-classic-hpcpack-cluster/nfsshare.png
[nfsperm]: ./media/virtual-machines-linux-classic-hpcpack-cluster/nfsperm.png
[nfsmanage]: ./media/virtual-machines-linux-classic-hpcpack-cluster/nfsmanage.png

<!---HONumber=AcomDC_0330_2016------>