<properties 
	pageTitle="SQL パーティション テーブルを使用した並列の一括データ インポート | Microsoft Azure" 
	description="SQL パーティション テーブルを使用した並列の一括データ インポート" 
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
	ms.date="09/19/2016" 
	ms.author="bradsev" />

# SQL パーティション テーブルを使用した並列の一括データ インポート

このドキュメントでは、データを SQL Server データベースに高速に並列一括インポートするためのパーティション分割されたテーブルを作成する方法について説明します。SQL Database へのビッグ データの読み込み/転送では、"_パーティション テーブルとビュー_" を使用することによって、SQL DB へのデータのインポートと以降のクエリを向上させることができます。


## 新しいデータベースとファイル グループのセットの作成

- [新しいデータベースを作成する](https://technet.microsoft.com/library/ms176061.aspx) (データベースが存在しない場合)
- パーティション分割された物理ファイルを格納するデータベースにデータベース ファイル グループを追加する

  注: 新規の場合は [CREATE DATABASE](https://technet.microsoft.com/library/ms176061.aspx)、データベースが既に存在する場合は [ALTER DATABASE](https://msdn.microsoft.com/library/bb522682.aspx) を使用してこれを行うことができます。

- 1 つまたは複数のファイル (必要に応じて) を各データベース ファイル グループに追加する

 > [AZURE.NOTE] このパーティションのデータを保持するターゲット ファイル グループと、ファイル グループのデータが格納される物理データベース ファイル名を指定します。
 
次の例では、それぞれに 1 つの物理ファイルが含まれている、プライマリとログ グループ以外に 3 つのファイル グループを持つ新しいデータベースを作成します。データベース ファイルは、SQL Server インスタンスで構成されているとおりに、既定の SQL Server データ フォルダーに作成されます。既定のファイルの場所の詳細については、「[SQL Server の既定のインスタンスおよび名前付きインスタンスのファイルの場所](https://msdn.microsoft.com/library/ms143547.aspx)」を参照してください。

    DECLARE @data_path nvarchar(256);
    SET @data_path = (SELECT SUBSTRING(physical_name, 1, CHARINDEX(N'master.mdf', LOWER(physical_name)) - 1)
      FROM master.sys.master_files
      WHERE database_id = 1 AND file_id = 1);
    
    EXECUTE ('
    	CREATE DATABASE <database_name>
     	ON  PRIMARY 
    	( NAME = ''Primary'', FILENAME = ''' + @data_path + '<primary_file_name>.mdf'', SIZE = 4096KB , FILEGROWTH = 1024KB ), 
     	FILEGROUP [filegroup_1] 
    	( NAME = ''FileGroup1'', FILENAME = ''' + @data_path + '<file_name_1>.ndf'' , SIZE = 4096KB , FILEGROWTH = 1024KB ), 
     	FILEGROUP [filegroup_2] 
    	( NAME = ''FileGroup1'', FILENAME = ''' + @data_path + '<file_name_2>.ndf'' , SIZE = 4096KB , FILEGROWTH = 1024KB ), 
     	FILEGROUP [filegroup_3] 
    	( NAME = ''FileGroup1'', FILENAME = ''' + @data_path + '<file_name>.ndf'' , SIZE = 102400KB , FILEGROWTH = 10240KB ), 
     	LOG ON 
    	( NAME = ''LogFileGroup'', FILENAME = ''' + @data_path + '<log_file_name>.ldf'' , SIZE = 1024KB , FILEGROWTH = 10%)
    ')
    
## パーティション テーブルを作成する

データ スキーマに従って、前の手順で作成されたデータベースのファイル グループにマップされる、パーティション分割されたテーブルを作成します。パーティション分割されたテーブルにデータが一括インポートされると、以下に説明されているように、パーティション構成に従ってレコードがファイル グループ間で配布されます。

**パーティション テーブルを作成するには、以下の操作を行う必要があります。**

- 個別のパーティション テーブルに含める値/境界の範囲を定義する[パーティション関数を作成します](https://msdn.microsoft.com/library/ms187802.aspx)。たとえば、2013 年の月別 (some\_datetime\_field) にパーティションを制限するには以下のようにします。

	    CREATE PARTITION FUNCTION <DatetimeFieldPFN>(<datetime_field>)  
	    AS RANGE RIGHT FOR VALUES (
	    	'20130201', '20130301', '20130401',
	    	'20130501', '20130601', '20130701', '20130801',
	    	'20130901', '20131001', '20131101', '20131201' )

- パーティション関数の各パーティションの範囲を物理ファイル グループにマップする[パーティション スキームを作成します](https://msdn.microsoft.com/library/ms179854.aspx)。たとえば、以下のようにします。

	    CREATE PARTITION SCHEME <DatetimeFieldPScheme> AS  
	    PARTITION <DatetimeFieldPFN> TO (
	    <filegroup_1>, <filegroup_2>, <filegroup_3>, <filegroup_4>,
	    <filegroup_5>, <filegroup_6>, <filegroup_7>, <filegroup_8>,
	    <filegroup_9>, <filegroup_10>, <filegroup_11>, <filegroup_12> )

  ヒント: 関数/構成に従って各パーティションで有効な範囲を確認するには、次のクエリを実行します。

	    SELECT psch.name as PartitionScheme,
	    	prng.value AS ParitionValue,
	    	prng.boundary_id AS BoundaryID
	    FROM sys.partition_functions AS pfun
	    INNER JOIN sys.partition_schemes psch ON pfun.function_id = psch.function_id
	    INNER JOIN sys.partition_range_values prng ON prng.function_id=pfun.function_id
	    WHERE pfun.name = <DatetimeFieldPFN>

- データ スキーマに従って[パーティション テーブルを作成](https://msdn.microsoft.com/library/ms174979.aspx)し、テーブルのパーティション分割に使用されるパーティション スキーマと制約フィールドを指定します。たとえば、以下のようにします。

	    CREATE TABLE <table_name> ( [include schema definition here] )
	    ON <TablePScheme>(<partition_field>)

詳細については、「[パーティション テーブルとパーティション インデックスの作成](https://msdn.microsoft.com/library/ms188730.aspx)」を参照してください。


## 個別のパーティション テーブルごとにデータを一括インポートする

- BCP、BULK INSERT、[SQL Server 移行ウィザード](http://sqlazuremw.codeplex.com/)などの他の方法を使用することができます。ここで示されている例では BCP による方法を使用します。

- [データベースを変更](https://msdn.microsoft.com/library/bb522682.aspx)して、トランザクション ログの設定を BULK\_LOGGED に変更し、ログのオーバーヘッドを最小限に抑えます。たとえば、以下のようにします。

	    ALTER DATABASE <database_name> SET RECOVERY BULK_LOGGED

- データの読み込み時間を短縮するには、一括インポート操作を並列に起動します。SQL Server データベースへのビッグ データの一括インポートを高速化するヒントについては、「[1 時間未満で 1 TB を読み込む](http://blogs.msdn.com/b/sqlcat/archive/2006/05/19/602142.aspx)」を参照してください。

次の PowerShell スクリプトは、BCP を使用した並行データ読み込みの例を示しています。

    # Set database name, input data directory, and output log directory
	# This example loads comma-separated input data files
	# The example assumes the partitioned data files are named as <base_file_name>_<partition_number>.csv
	# Assumes the input data files include a header line. Loading starts at line number 2.

	$dbname = "<database_name>"
	$indir  = "<path_to_data_files>"
	$logdir = "<path_to_log_directory>"

	# Select authentication mode
    $sqlauth = 0
    
    # For SQL authentication, set the server and user credentials
    $sqlusr = "<user@server>"
    $server = "<tcp:serverdns>"
    $pass   = "<password>"

    # Set number of partitions per table - Should match the number of input data files per table
    $numofparts = <number_of_partitions>
       
	# Set table name to be loaded, basename of input data files, input format file, and number of partitions
	$tbname = "<table_name>"
	$basename = "<base_input_data_filename_no_extension>"
	$fmtfile = "<full_path_to_format_file>"
   
    # Create log directory if it does not exist
    New-Item -ErrorAction Ignore -ItemType directory -Path $logdir
      
    # BCP example using Windows authentication
    $ScriptBlock1 = {
       param($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $num)
       bcp ($dbname + ".." + $tbname) in ($indir + "" + $basename + "_" + $num + ".csv") -o ($logdir + "" + $tbname + "_" + $num + ".txt") -h "TABLOCK" -F 2 -C "RAW" -f ($fmtfile) -T -b 2500 -t "," -r \n
    }
    
    # BCP example using SQL authentication
    $ScriptBlock2 = {
       param($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $num, $sqlusr, $server, $pass)
       bcp ($dbname + ".." + $tbname) in ($indir + "" + $basename + "_" + $num + ".csv") -o ($logdir + "" + $tbname + "_" + $num + ".txt") -h "TABLOCK" -F 2 -C "RAW" -f ($fmtfile) -U $sqlusr -S $server -P $pass -b 2500 -t "," -r \n
    }
    
    # Background processing of all partitions
    for ($i=1; $i -le $numofparts; $i++)
    {
       Write-Output "Submit loading trip and fare partitions # $i"
       if ($sqlauth -eq 0) {
          # Use Windows authentication
          Start-Job -ScriptBlock $ScriptBlock1 -Arg ($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $i)
       } 
       else {
          # Use SQL authentication
          Start-Job -ScriptBlock $ScriptBlock2 -Arg ($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $i, $sqlusr, $server, $pass)
       }
    }
    
    Get-Job
    
    # Optional - Wait till all jobs complete and report date and time
    date
    While (Get-Job -State "Running") { Start-Sleep 10 }
    date


## インデックスを作成して結合およびクエリのパフォーマンスを最適化する

- 複数のテーブルからモデリング用のデータを抽出する場合、結合のパフォーマンスを向上させるには、結合キーのインデックスを作成します。

- パーティションごとに同じファイル グループをターゲットとする (クラスター化または非クラスター化された) [インデックスを作成](https://technet.microsoft.com/library/ms188783.aspx)します。たとえば、以下のようにします。

	    CREATE CLUSTERED INDEX <table_idx> ON <table_name>( [include index columns here] )
	    ON <TablePScheme>(<partition)field>)
または、

	    CREATE INDEX <table_idx> ON <table_name>( [include index columns here] )
	    ON <TablePScheme>(<partition)field>)

 > [AZURE.NOTE] データを一括インポートする前に、インデックスを作成することもできます。一括インポートする前にインデックスを作成すると、データの読み込みが低下します。


## 実行中の Advanced Analytics Process and Technology の例

パブリック データセットを使用した Cortana Analytics Process のエンドツーエンドのチュートリアル例については、「[Cortana Analytics Process の活用: SQL Server を使用する](machine-learning-data-science-process-sql-walkthrough.md)」を参照してください。
 

<!---HONumber=AcomDC_0921_2016-->