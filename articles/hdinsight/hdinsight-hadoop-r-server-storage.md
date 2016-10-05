
<properties
   pageTitle="HDInsight の R Server (プレビュー) の Azure Storage オプション | Microsoft Azure"
   description="HDInsight の R Server (プレビュー) でユーザーが利用できるさまざまなストレージ オプションについて説明します。"
   services="HDInsight"
   documentationCenter=""
   authors="jeffstokes72"
   manager="jhubbard"
   editor="cgronlun"
/>

<tags
   ms.service="HDInsight"
   ms.devlang="R"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="data-services"
   ms.date="09/01/2016"
   ms.author="jeffstok"
/>

# HDInsight の R Server (プレビュー) の Azure Storage オプション

HDInsight の Microsoft R Server (プレビュー) は、データ、コード、分析の結果オブジェクトなどを保持するための手段として、Azure BLOB と [Azure Data Lake ストレージ](https://azure.microsoft.com/services/data-lake-store/)の両方にアクセスできます。

HDInsight で Hadoop クラスターを作成するときに、Azure ストレージ アカウントを指定します。そのアカウントの特定の Blob Storage コンテナーが、作成したクラスターのファイル システム (Hadoop 分散ファイル システムなど) を保持します。パフォーマンス上の理由から、HDInsight クラスターは、指定したプライマリ ストレージ アカウントと同じデータ センターに作成されます。詳細については、「[HDInsight での Azure BLOB ストレージの使用](hdinsight-hadoop-use-blob-storage.md "HDInsight での Azure BLOB ストレージの使用")」をご覧ください。


## 複数の Azure BLOB ストレージ アカウントの使用

必要に応じて、HDI クラスターを持つ複数の Azure ストレージ アカウントまたはコンテナーにアクセスすることができます。そのためには、クラスターの作成時に UI で追加のストレージ アカウントを指定し、次の手順に従ってそのストレージ アカウントを R で使用する必要があります。

1.	**storage1** というストレージ アカウント名で、**container1** という名前の既定のコンテナーを持つ HDInsight クラスターを作成します。
2. **storage2** という名前の追加のストレージ アカウントを指定します。
3. mycsv.csv ファイルを /share ディレクトリにコピーし、このファイルに対して分析を実行します。

    ````
    hadoop fs –mkdir /share
    hadoop fs –copyFromLocal myscsv.scv /share  
    ````

3.	R コードで、名前ノードを **default** に設定し、処理対象のディレクトリとファイルを設定します。

    ````
    myNameNode <- "default"
    myPort <- 0
    ````

  データの場所:

    bigDataDirRoot <- "/share"  

  Spark コンピューティング コンテキストの定義:

    mySparkCluster <- RxSpark(consoleOutput=TRUE)

  コンピューティング コンテキストの設定:

    rxSetComputeContext(mySparkCluster)

  Hadoop 分散ファイル システム (HDFS) ファイル システムの定義:

    hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

  分析する HDFS 内の入力ファイルの指定:

    inputFile <-file.path(bigDataDirRoot,"mycsv.csv")

ディレクトリとファイルの参照はすべて、ストレージ アカウント wasbs://container1@storage1.blob.core.windows.net を指しています。これは、HDInsight クラスターに関連付けられる**既定のストレージ アカウント**です。

ここで、**storage2** の **container2** の /private ディレクトリにある mySpecial.csv というファイルを処理するとします。

R コードで、名前ノード参照が **storage2** ストレージ アカウントを指すようにします。

    myNameNode <- "wasbs://container2@storage2.blob.core.windows.net"
    myPort <- 0

  データの場所:

    bigDataDirRoot <- "/private"

  Spark コンピューティング コンテキストの定義:

    mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)

  コンピューティング コンテキストの設定:

    rxSetComputeContext(mySparkCluster)

  HDFS ファイル システムの定義:

    hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

  分析する HDFS 内の入力ファイルの指定:

    inputFile <-file.path(bigDataDirRoot,"mySpecial.csv")

ここでは、ディレクトリとファイルの参照はすべて、ストレージ アカウント wasbs://container2@storage2.blob.core.windows.net を指しています。これは、先ほど指定した**名前ノード**です。

次のように、**storage2** で /user/RevoShare/<SSH ユーザー名> ディレクトリを構成する必要があることに注意してください。

    hadoop fs -mkdir wasbs://container2@storage2.blob.core.windows.net/user
    hadoop fs -mkdir wasbs://container2@storage2.blob.core.windows.net/user/RevoShare
    hadoop fs -mkdir wasbs://container2@storage2.blob.core.windows.net/user/RevoShare/<RDP username>

## Azure Data Lake Store の使用

HDInsight アカウントで Data Lake Store を使用するには、使用する各 Azure Data Lake Store にクラスター アクセスを付与する必要があります。(前の手順で説明した) セカンダリ ストレージ アカウントと同様に、ストアを R スクリプトで使用します。

## Azure Data Lake Store へのクラスター アクセスの追加

HDInsight クラスターに関連付けられている Azure Active Directory (Azure AD) サービス プリンシパルを使用して、Data Lake Store にアクセスします。

### サービス プリンシパルを追加するには
1. HDInsight クラスターを作成するときに、**[データ ソース]** タブの **[クラスター AAD ID]** を選択します。
2. **[クラスター AAD ID]** ダイアログ ボックスの **[AD サービス プリンシパルの選択]** で、**[新規作成]** を選択します。

サービス プリンシパルに名前を付けてパスワードを設定すると、新しいタブが開きます。このタブで、サービス プリンシパルと Data Lake Store を関連付けることができます。

Data Lake Store へのアクセスを後から追加するには、Azure ポータルで Data Lake Store を開いて、**[データ エクスプローラー]**、**[アクセス]** の順に移動します。サービス プリンシパルを作成して "rkadl11" Data Lake Store に関連付ける方法を示すダイアログ ボックスの例を次に示します。

![Create Data Lake store Service Principle 1](./media/hdinsight-hadoop-r-server-storage/hdinsight-hadoop-r-server-storage-adls-sp1.png)


![Create Data Lake store Service Principle 2](./media/hdinsight-hadoop-r-server-storage/hdinsight-hadoop-r-server-storage-adls-sp2.png)

## Data Lake Store と R Server の使用
Data Lake Store へのアクセスを付与したら、Azure セカンダリ ストレージ アカウントと同様に、HDInsight の R Server で Store を使用できます。唯一の違いは、プレフィックスが次のように **wasb://** から **adl://** に変わることです。

````
# Point to the ADL store (e.g. ADLtest)
myNameNode <- "adl://rkadl1.azuredatalakestore.net"
myPort <- 0

# Location of the data (assumes a /share directory on the ADL account)
bigDataDirRoot <- "/share"  

# Define Spark compute context
mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)

# Set compute context
rxSetComputeContext(mySparkCluster)

# Define HDFS file system
hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

# Specify the input file in HDFS to analyze
inputFile <-file.path(bigDataDirRoot,"AirlineDemoSmall.csv")

# Create factors for days of the week
colInfo <- list(DayOfWeek = list(type = "factor",
               levels = c("Monday", "Tuesday", "Wednesday", "Thursday",
                          "Friday", "Saturday", "Sunday")))

# Define the data source
airDS <- RxTextData(file = inputFile, missingValueString = "M",
                    colInfo  = colInfo, fileSystem = hdfsFS)

# Run a linear regression
model <- rxLinMod(ArrDelay~CRSDepTime+DayOfWeek, data = airDS)
````

次に示すコマンドは、RevoShare ディレクトリを含む Data Lake ストレージ アカウントを構成し、前の例のサンプル .csv ファイルを追加します。

````
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare/<user>

hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/share

hadoop fs -copyFromLocal /usr/lib64/R Server-7.4.1/library/RevoScaleR/SampleData/AirlineDemoSmall.csv adl://rkadl1.azuredatalakestore.net/share

hadoop fs –ls adl://rkadl1.azuredatalakestore.net/share
````

## エッジ ノードでの Azure Files の使用

エッジ ノードでの使用に適した [Azure Files](../storage/storage-how-to-use-files-linux.md "Azure Files") というデータ ストレージ オプションもあります。Azure Files を使用すると、Azure Storage ファイル共有を Linux ファイル システムにマウントできます。これは、HDFS ではなくエッジ ノード上でネイティブ ファイル システムを使用する方がよい場合に、後で必要となる可能性があるデータ ファイル、R スクリプト、結果オブジェクトを格納するのに便利です。

Azure Files の大きな利点は、サポートされている OS (Windows や Linux など) を使用している任意のシステムで、ファイル共有をマウントして使用できることです。たとえば、チーム内のメンバーが所有する他の HDInsight クラスターや、Azure VM、オンプレミスのシステムでも使用できます。


## 次のステップ

ここでは、SSH セッションから R コンソールを使用する方法と、R Server を含む新しい HDInsight クラスターを作成する方法の基本を説明しました。HDInsight で R Server を使用する他の方法については、次のリンクを参照してください。

- [概要: HDInsight の R Server](hdinsight-hadoop-r-server-overview.md)
- [Hadoop での R Server の使用開始](hdinsight-hadoop-r-server-get-started.md)
- [HDInsight Premium への RStudio Server の追加](hdinsight-hadoop-r-server-install-r-studio.md)
- [HDInsight の R Server (プレビュー) の計算コンテキストのオプション](hdinsight-hadoop-r-server-compute-contexts.md)

<!---HONumber=AcomDC_0921_2016-->