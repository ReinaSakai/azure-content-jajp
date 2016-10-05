<properties 
	pageTitle="Azure Machine Learning 用にデータを Azure SQL Database に移動する | Azure" 
	description="SQL テーブルを作成して SQL テーブルにデータを読み込みます" 
	services="machine-learning" 
	documentationCenter="" 
	authors="bradsev"
	manager="jhubbard"
	editor="cgronlun" />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/14/2016"
	ms.author="bradsev" />

# Azure Machine Learning 用にデータを Azure SQL Database に移動する

このトピックでは、フラット ファイル (CSV 形式または TSV 形式) のデータまたはオンプレミスの SQL Server に格納されているデータを、Azure SQL Database に移動するためのオプションについて説明します。このクラウドへのデータ移動タスクは、Team Data Science Process の一部です。

Machine Learning 用にオンプレミスの SQL Server にデータを移動するためのオプションについては、「[Azure Virtual Machine 上の SQL Server にデータを移動する](machine-learning-data-science-move-sql-server-virtual-machine.md)」をご覧ください。

次の**メニュー**は、Team Data Science Process (TDSP) でデータを保存および処理できるターゲット環境にデータを取り込む方法について説明するトピックにリンクしています。

[AZURE.INCLUDE [cap-ingest-data-selector](../../includes/cap-ingest-data-selector.md)]

次の表は、Azure SQL Database にデータを移動するためのオプションをまとめたものです。

<b>ソース</b> |<b>移動先: Azure SQL Database</b> |
-------------- |--------------------------------|
<b>フラット ファイル (CSV または TSV 形式)</b> |<a href="#bulk-insert-sql-query">一括挿入 SQL クエリ |
<b>オンプレミスの SQL Server</b> | 1\.<a href="#export-flat-file">フラット ファイルへのエクスポート<br> 2.<a href="#insert-tables-bcp">SQL Database 移行ウィザード<br> 3.<a href="#db-migration">データベースのバックアップと復元<br> 4.<a href="#adf">Azure Data Factory |


## <a name="prereqs"></a>前提条件
ここに記載されている手順には次のものが必要です。

* **Azure サブスクリプション**。サブスクリプションがない場合は、[無料試用版](https://azure.microsoft.com/pricing/free-trial/)にサインアップできます。
* **Azure ストレージ アカウント**。このチュートリアルでは、データの格納に Azure ストレージ アカウントを使用します。Azure ストレージ アカウントがない場合は、「[ストレージ アカウントの作成](storage-create-storage-account.md#create-a-storage-account)」を参照してください。ストレージ アカウントを作成したら、ストレージへのアクセスに使用するアカウント キーを取得する必要があります。「[ストレージ アクセス キーの表示、コピーおよび再生成](storage-create-storage-account.md#view-copy-and-regenerate-storage-access-keys)」を参照してください。
* **Azure SQL Database** へのアクセス権。Azure SQL Database をセットアップする必要がある場合、Azure SQL Database の新しいインスタンスをプロビジョニングする方法については、[Microsoft Azure SQL Database の概要](../sql-database/sql-database-get-started.md)に関する記事をご覧ください。
* **Azure PowerShell** がローカルにインストールされ構成されていること。手順については、「[Azure PowerShell のインストールおよび構成方法](../powershell-install-configure.md)」を参照してください。

**データ**: 移行プロセスは、[NYC タクシー データセット](http://chriswhong.com/open-data/foil_nyc_taxi/)を使用して説明されています。NYC タクシー データセットには乗車データと料金についての情報が含まれています。投稿で説明されているように、NYC タクシー データセットは Azure BLOB ストレージの [NYC タクシー データ](http://www.andresmh.com/nyctaxitrips/)で入手できます。これらのファイルのサンプルと説明については、「[NYC タクシー乗車データセットの説明](machine-learning-data-science-process-sql-walkthrough.md#dataset)」をご覧ください。
 
ここで説明されている手順は、自身のデータに適用することも、NYC タクシー データセットを使用してこの手順に従って行うこともできます。NYC タクシー データセットを自身のオンプレミス SQL Server データベースにアップロードするには、「[SQL Server データベースにデータを一括インポートする](machine-learning-data-science-process-sql-walkthrough.md#dbload)」に記載されている手順に従います。これらは Azure Virtual Machine 上の SQL Server にアップロードする手順ですが、オンプレミスの SQL Server へのアップロード手順も同じです。


## <a name="file-to-azure-sql-database"></a>フラット ファイル ソースから Azure SQL Database へのデータの移動

フラット ファイル (CSV 形式または TSV 形式) のデータは、一括挿入 SQL クエリを使用して Azure SQL Database に移動できます。

### <a name="bulk-insert-sql-query"></a>一括挿入 SQL クエリ

一括挿入 SQL クエリを使用する手順は、フラット ファイル ソースから Azure VM 上の SQL Server にデータを移動する手順と似ています。詳細については、「[一括挿入 SQL クエリ](machine-learning-data-science-move-sql-server-virtual-machine.md#insert-tables-bulkquery)」をご覧ください。


##<a name="sql-on-prem-to-sazure-sql-database"></a>オンプレミスの SQL Server から Azure SQL Database へのデータの移動

ソース データがオンプレミスの SQL Server に保存されている場合は、さまざまな方法で Azure SQL Database にデータを移動できます。

1. [フラット ファイルへのエクスポート](#export-flat-file)
2. [SQL Database 移行ウィザード](#insert-tables-bcp)
3. [データベースのバックアップと復元](#db-migration)
4. [Azure Data Factory](#adf)

最初の 3 つの手順は、「[Azure Virtual Machine 上の SQL Server にデータを移動する](machine-learning-data-science-move-sql-server-virtual-machine.md)」の対応する手順とよく似ています。以降の説明には、そのトピックの該当するセクションへのリンクが含まれています。

###<a name="export-flat-file"></a>フラット ファイルへのエクスポート

フラット ファイルにエクスポートする手順は、「[フラット ファイルへのエクスポート](machine-learning-data-science-move-sql-server-virtual-machine.md#export-flat-file)」の手順と似ています。

###<a name="insert-tables-bcp"></a>SQL Database 移行ウィザード

SQL Database 移行ウィザードを使用する手順は、「[SQL Database 移行ウィザード](machine-learning-data-science-move-sql-server-virtual-machine.md#sql-migration)」の手順と似ています。

###<a name="db-migration"></a>データベースのバックアップと復元

データベースのバックアップと復元を使用する手順は、「[データベースのバックアップと復元](machine-learning-data-science-move-sql-server-virtual-machine.md#sql-backup)」の手順と似ています。

###<a name="adf"></a>Azure Data Factory

Azure Data Factory (ADF) を使用して Azure SQL Database にデータを移動する手順は、「[Azure Data Factory を使用してオンプレミスの SQL Server から SQL Azure にデータを移動する](machine-learning-data-science-move-sql-azure-adf.md)」に記載されています。このトピックでは、ADF を使用してオンプレミスの SQL Server データベースから Azure BLOB ストレージ経由で Azure SQL Database にデータを移動する方法が示されています。

オンプレミスとクラウドの両方のリソースにアクセスするハイブリッド シナリオで、データを継続的に移行する必要がある場合、移行時にデータを処理する場合、移行時にデータを変更したり、ビジネス ロジックを追加したりする必要がある場合には、ADF の使用を検討してください。ADF では、定期的にデータの移動を管理するシンプルな JSON スクリプトを使用して、ジョブのスケジュールと監視ができます。ADF には他にも、複雑な操作のサポートなどの機能があります。

<!---HONumber=AcomDC_0921_2016-->