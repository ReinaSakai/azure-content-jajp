<properties
	pageTitle="Spark を使用したデータ探索とモデリング | Microsoft Azure"
	description="Spark MLlib ツールキットのデータ探索およびモデリング機能を紹介します。"
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
	ms.date="06/14/2016"
	ms.author="deguhath;bradsev;gokuma" />

# Spark を使用したデータ探索とモデリング

[AZURE.INCLUDE [machine-learning-spark-modeling](../../includes/machine-learning-spark-modeling.md)]

このチュートリアルでは、HDInsight Spark を使用して、2013 年 NYC タクシー乗車および料金データセットのサンプルでデータ探索を実行し、二項分類および回帰モデリング タスクを実行します。チュートリアルでは、エンド ツー エンドの[データ サイエンス プロセス](http://aka.ms/datascienceprocess)の手順について説明します。処理には HDInsight Spark クラスターを使用し、Azure BLOB にデータとモデルを保存します。プロセスでは、Azure Storage BLOB のデータを探索し、視覚化した後、予測モデルを構築するためのデータを準備します。これらのモデルは、二項分類および回帰モデリング タスクを実行する Spark MLlib キットを使用して構築されます。

- **二項分類**タスクでは、乗車でチップが支払われるかどうかを予測します。
- **回帰**タスクでは、チップの他の特徴に基づいてチップの金額を予測します。

使用するモデルは、ロジスティック回帰と線形回帰、ランダム フォレスト、勾配ブースティング ツリーです。

- [SGD を使用した線形回帰](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LinearRegressionWithSGD)は、最適化に確率的勾配降下 (SGD) 法を使用し、特徴のスケーリングを使用して支払われるチップの金額を予測する線形回帰モデルです。
- [LBFGS を使用したロジスティック回帰](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.classification.LogisticRegressionWithLBFGS) ("ロジット" 回帰) は、データ分類を実行するために従属変数がカテゴリ型である場合に使用できる回帰モデルです。LBFGS は、限られた量のコンピューター メモリを使用する Broyden–Fletcher–Goldfarb–Shanno (BFGS) アルゴリズムに近い準ニュートン最適化アルゴリズムであり、機械学習で広く使用されています。
- [ランダム フォレスト](http://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests)は、複数のデシジョン ツリーをまとめたものです。オーバーフィットのリスクを軽減するために、多くのデシジョン ツリーが結合されています。ランダム フォレストは回帰と分類に使用されます。カテゴリの特徴を処理し、多クラス分類設定に拡張できますが、特徴のスケーリングは不要であり、非線形性や特徴の相互作用をキャプチャできます。ランダム フォレストは、分類と回帰に使用される最も成功している機械学習モデルの 1 つです。
- [勾配ブースティング ツリー](http://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) は、複数のデシジョン ツリーをまとめたものです。GBT は、デシジョン ツリーを繰り返しトレーニングすることで損失関数を最小限に抑えます。GBT は回帰と分類に使用されます。カテゴリの特徴を処理できますが、特徴のスケーリングは不要であり、非線形性や特徴の相互作用をキャプチャできます。また、多クラス分類設定でも使用できます。

また、モデリング手順には、各種モデルをトレーニング、評価し、保存する方法を示すコードも含まれています。ソリューションのコーディングと関連するプロットの表示には、Python が使用されています。


>[AZURE.NOTE] Spark MLlib ツールキットは、大規模なデータセットを操作することを目的としていますが、そのモデリング機能を紹介するために、便宜上、比較的小規模なサンプル (最大30 MB、170,000 行使用、元の NYC データセットの約 0.1%) を使用しています。ここでの演習は、2 つのワーカー ノードを含む HDInsight クラスターで効率的に実行されます (約 10 分)。少し変更を加えれば、同じコードを使用して大規模なデータセットを処理できます。メモリへのデータのキャッシュやクラスター サイズの変更など、適切な変更を加えてください。

## 前提条件

Azure アカウントと HDInsight Spark が必要となります。このチュートリアルを実行するには HDInsight 3.4 Spark 1.6 クラスターが必要です。このチュートリアルの要件、チュートリアルで使用される 2013 年 NYC タクシー データの説明、Spark クラスターで Jupyter Notebook のコードを実行する方法については、「[Overview of Data Science using Spark on Azure HDInsight (Azure HDInsight での Spark を使用したデータ サイエンスの概要)](machine-learning-data-science-spark-overview.md)」をご覧ください。このトピックのコード サンプルが含まれた **machine-learning-data-science-spark-data-exploration-modeling.ipynb** ノートブックは、[GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/pySpark) で入手できます。


[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]


## セットアップ: ストレージの場所、ライブラリ、プリセットの Spark コンテキスト

Spark は、Azure Storage BLOB (WASB とも呼ばれます) に対する読み取りと書き込みを実行できます。そのため、WASB に保存されている既存のデータは Spark を使用して処理することができ、結果も WASB に保存できます。

WASB にモデルやファイルを保存するには、パスを正しく指定する必要があります。Spark クラスターに接続されている既定のコンテナーは、"wasb///" で始まるパスを使用して参照できます。他の場所は "wasb://" で参照します。


### WASB 内のストレージの場所のディレクトリ パスを設定する

次のコード サンプルでは、読み取るデータの場所と、モデルの出力の保存先となるモデル ストレージ ディレクトリのパスを指定しています。


	# SET PATHS TO FILE LOCATIONS: DATA AND MODEL STORAGE

	# LOCATION OF TRAINING DATA
	taxi_train_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Train.tsv";

	# SET THE MODEL STORAGE DIRECTORY PATH 
	# NOTE THAT THE FINAL BACKSLASH IN THE PATH IS NEEDED.
	modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 


### ライブラリのインポート

セットアップでは、必要なライブラリのインポートも行う必要があります。次のコードを使用して、Spark コンテキストを設定し、必要なライブラリをインポートします。


	# IMPORT LIBRARIES
	import pyspark
	from pyspark import SparkConf
	from pyspark import SparkContext
	from pyspark.sql import SQLContext
	import matplotlib
	import matplotlib.pyplot as plt
	from pyspark.sql import Row
	from pyspark.sql.functions import UserDefinedFunction
	from pyspark.sql.types import *
	import atexit
	from numpy import array
	import numpy as np
	import datetime


### プリセットの Spark コンテキストと PySpark マジック

Jupyter Notebook で提供される PySpark カーネルは、コンテキストがあらかじめ設定されており、開発しているアプリケーションの操作を開始する前に、Spark または Hive コンテキストを明示的に設定する必要がありません。これらのカーネルは、既定で利用できます。各コンテキストは次のとおりです。

- sc: Spark 用
- sqlContext: Hive 用

PySpark カーネルには、"マジック"、つまり、%% で呼び出すことができる特別なコマンドがいくつか事前定義されています。そのようなコマンドが、以降のコード サンプルでは 2 つ使用されています。

- **%%local** このコマンドを指定した場合、後続行のすべてのコードがローカルで実行されます。コードは有効な Python コードにする必要があります。
- **%%sql -o <変数名>** sqlContext に対して Hive クエリを実行します。-o パラメーターが渡される場合、クエリの結果は、Pandas データフレームとして %%local Python コンテキストで永続化されます。
 

Jupyter Notebook のカーネルと、%% で呼び出される定義済みの "マジック" (例: %%local) の詳細については、[HDInsight の HDInsight Spark Linux クラスターと Jupyter Notebook で使用可能なカーネル](../hdinsight/hdinsight-apache-spark-jupyter-notebook-kernels.md)に関する記事をご覧ください。
 

## パブリック BLOB からのデータの取り込み

このセクションでは、モデル化するデータ サンプルの取り込みに必要な一連のタスクを実行するコードを紹介します。タクシーの乗車データおよび料金ファイル (.tsv ファイルとして保存) の結合された 0.1% サンプルを読み取り、データのフォーマットとクリーニングを行います。次に、データ フレームを作成してメモリにキャッシュし、SQL コンテキストで一時テーブルとして登録します。

データ サイエンス プロセスでは、まず、分析するデータを外部のソースまたはシステムからデータ探索およびモデリング環境に取り込みます。このチュートリアルでは、この環境が Spark です。ここでは、次の一連のタスクを実行するコードを示します。

- モデル化するデータ サンプルの取り込み
- 入力データセット (.tsv ファイルとして保存) の読み取り
- データのフォーマットとクリーニング
- オブジェクト (RDD またはデータ フレーム) の作成とメモリへのキャッシュ
- SQL コンテキストでオブジェクトを一時テーブルとして登録

データ取り込みのコードを次に示します。

	# INGEST DATA

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# IMPORT FILE FROM PUBLIC BLOB
	taxi_train_file = sc.textFile(taxi_train_file_loc)
	
	# GET SCHEMA OF THE FILE FROM HEADER
	schema_string = taxi_train_file.first()
	fields = [StructField(field_name, StringType(), True) for field_name in schema_string.split('\t')]
	fields[7].dataType = IntegerType() #Pickup hour
	fields[8].dataType = IntegerType() # Pickup week
	fields[9].dataType = IntegerType() # Weekday
	fields[10].dataType = IntegerType() # Passenger count
	fields[11].dataType = FloatType() # Trip time in secs
	fields[12].dataType = FloatType() # Trip distance
	fields[19].dataType = FloatType() # Fare amount
	fields[20].dataType = FloatType() # Surcharge
	fields[21].dataType = FloatType() # Mta_tax
	fields[22].dataType = FloatType() # Tip amount
	fields[23].dataType = FloatType() # Tolls amount
	fields[24].dataType = FloatType() # Total amount
	fields[25].dataType = IntegerType() # Tipped or not
	fields[26].dataType = IntegerType() # Tip class
	taxi_schema = StructType(fields)
	
	# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
	taxi_header = taxi_train_file.filter(lambda l: "medallion" in l)
	taxi_temp = taxi_train_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
	        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
	                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
	                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))
	
	
	# CREATE DATA FRAME
	taxi_train_df = sqlContext.createDataFrame(taxi_temp, taxi_schema)
	
	# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
	taxi_df_train_cleaned = taxi_train_df.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
	    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
	    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
	    .drop('direct_distance').drop('surcharge')\
	    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )

	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	taxi_df_train_cleaned.cache()
	taxi_df_train_cleaned.count()
	
	# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
	taxi_df_train_cleaned.registerTempTable("taxi_train")
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**出力:**

上記のセルの実行に要した時間: 51.72 秒


## データの探索と視覚化

データが Spark に取り込まれたら、次に、探索と視覚化によってデータの理解を深めます。このセクションでは、SQL クエリを使用してタクシー データを調べ、視覚化するためにターゲット変数と予想される特徴をプロットします。具体的には、タクシー乗車における乗客数の頻度、チップの金額の頻度、支払金額と支払の種類によるチップの変化をプロットします。

### タクシー乗車データのサンプルで乗客数の頻度のヒストグラムをプロットする

このコードおよび後続のスニペットでは、SQL マジックを使用してサンプルを照会し、ローカル マジックを使用してデータをプロットしています。

- **SQL マジック (`%%sql`)** HDInsight PySpark カーネルは、sqlContext に対する簡単なインライン HiveQL クエリをサポートしています。引数 (-o VARIABLE\_NAME) を指定すると、SQL クエリの出力結果が Pandas データフレームとして Jupyter サーバー上に永続化されます。つまり、出力結果をローカルから使用できるようになります。
- **`%%local` マジック**は、Jupyter サーバー (HDInsight クラスターのヘッド ノード) でコードをローカルで実行するときに使用します。通常、`%%local` マジックは、-o パラメーターを指定した `%%sql` マジックと組み合わせて使用します。SQL クエリの出力結果を -o パラメーターでローカルに永続化したうえで、%%local マジックを使用すると、それに続く一連のコード スニペットが、ローカルに永続化されている SQL クエリの出力結果に対してローカルに実行されます。

その出力結果は、コードの実行後、自動的に視覚化されます。

次のクエリは、乗客数ごとの乗車回数を取得しています。

	# PLOT FREQUENCY OF PASSENGER COUNTS IN TAXI TRIPS

	# HIVEQL QUERY AGAINST THE sqlContext
	%%sql -q -o sqlResults
	SELECT passenger_count, COUNT(*) as trip_counts 
	FROM taxi_train 
	WHERE passenger_count > 0 and passenger_count < 7 
	GROUP BY passenger_count 

以下のコードは、クエリの出力結果からローカル データフレームを作成し、データをプロットします。`%%local` マジックでローカル データフレーム (`sqlResults`) を作成し、そのデータを matplotlib でプロットします。

>[AZURE.NOTE] この PySpark マジックは、このチュートリアルの中でたびたび使用しています。データの量が大きい場合はサンプリングして、ローカル メモリに収まるようにデータフレームを作成する必要があります。

	#CREATE LOCAL DATA-FRAME AND USE FOR MATPLOTLIB PLOTTING

	# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
	%%local
	
	# USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES. 
	# CLICK ON THE TYPE OF PLOT TO BE GENERATED (E.G. LINE, AREA, BAR ETC.)
	sqlResults

乗客数ごとの乗車回数をプロットするコードを次に示します。

	# PLOT PASSENGER NUMBER VS. TRIP COUNTS
	%%local
	import matplotlib.pyplot as plt
	%matplotlib inline
	
	x_labels = sqlResults['passenger_count'].values
	fig = sqlResults[['trip_counts']].plot(kind='bar', facecolor='lightblue')
	fig.set_xticklabels(x_labels)
	fig.set_title('Counts of trips by passenger count')
	fig.set_xlabel('Passenger count in trips')
	fig.set_ylabel('Trip counts')
	plt.show()

**出力:**

![乗客数別乗車頻度](./media/machine-learning-data-science-spark-data-exploration-modeling/trip-freqency-by-passenger-count.png)

視覚化にはいくつかの種類 (表、円グラフ、折れ線グラフ、面グラフ、棒グラフ) があり、Notebook の **[Type (タイプ)]** メニュー ボタンで選択できます。ここに示したのは棒グラフによるプロットです。
	
### チップの金額と、乗客数別および料金別のチップ金額の変化のヒストグラムをプロットする

SQL クエリを使用してデータをサンプリングします。

	#PLOT HISTOGRAM OF TIP AMOUNTS AND VARIATION BY PASSENGER COUNT AND PAYMENT TYPE
	
	# HIVEQL QUERY AGAINST THE sqlContext
	%%sql -q -o sqlResults
	SELECT fare_amount, passenger_count, tip_amount, tipped 
	FROM taxi_train 
	WHERE passenger_count > 0 
	AND passenger_count < 7 
	AND fare_amount > 0 
	AND fare_amount < 200 
	AND payment_type in ('CSH', 'CRD') 
	AND tip_amount > 0 
	AND tip_amount < 25


次のコード セルでは、サンプリングされたデータに対し、SQL クエリを使用して 3 つのプロットを作成しています。

	# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
	%%local
	
	# HISTOGRAM OF TIP AMOUNTS AND PASSENGER COUNT
	ax1 = sqlResults[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
	ax1.set_title('Tip amount distribution')
	ax1.set_xlabel('Tip Amount ($)')
	ax1.set_ylabel('Counts')
	plt.suptitle('')
	plt.show()
	
	# TIP BY PASSENGER COUNT
	ax2 = sqlResults.boxplot(column=['tip_amount'], by=['passenger_count'])
	ax2.set_title('Tip amount by Passenger count')
	ax2.set_xlabel('Passenger count')
	ax2.set_ylabel('Tip Amount ($)')
	plt.suptitle('')
	plt.show()
	
	# TIP AMOUNT BY FARE AMOUNT, POINTS ARE SCALED BY PASSENGER COUNT
	ax = sqlResults.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(sqlResults.passenger_count))
	ax.set_title('Tip amount by Fare amount')
	ax.set_xlabel('Fare Amount ($)')
	ax.set_ylabel('Tip Amount ($)')
	plt.axis([-2, 100, -2, 20])
	plt.show()


**出力:**

![チップの金額の分布](./media/machine-learning-data-science-spark-data-exploration-modeling/tip-amount-distribution.png)

![乗客数別チップの金額](./media/machine-learning-data-science-spark-data-exploration-modeling/tip-amount-by-passenger-count.png)

![料金別チップの金額](./media/machine-learning-data-science-spark-data-exploration-modeling/tip-amount-by-fare-amount.png)


## モデリングのための特徴エンジニアリング、変換、およびデータの準備
このセクションでは、ML モデリングで使用するデータを準備する手順について説明し、手順を実行するコードを示します。次のタスクを実行する方法を示します。

- 時間を乗車時間のバケットにビン分割して新しい特徴を作成する
- カテゴリの特徴のインデックス作成とエンコードを実行する
- ML 関数への入力用にラベル付きポイント オブジェクトを作成する
- データのランダム サブサンプリングを作成し、トレーニング セットとテスト セットに分ける
- 特徴のスケーリング
- オブジェクトをメモリにキャッシュする


### 時間を乗車時間のバケットにビン分割して新しい特徴を作成する

次のコードでは、時間を乗車時間のバケットにビン分割して新しい特徴を作成する方法と、結果として生成されたデータ フレームをメモリにキャッシュする方法を示します。Resilient Distributed Dataset (RDD) とデータ フレームを繰り返し使用する場合、キャッシュによって実行時間が短縮されます。そのため、このチュートリアルでは複数の段階で RDD とデータ フレームをキャッシュしています。

	# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
	sqlStatement = """
	    SELECT *,
	    CASE
	     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
	     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
	     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
	     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
	    END as TrafficTimeBins
	    FROM taxi_train 
	"""
	taxi_df_train_with_newFeatures = sqlContext.sql(sqlStatement)
	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	# THE .COUNT() GOES THROUGH THE ENTIRE DATA-FRAME,
	# MATERIALIZES IT IN MEMORY, AND GIVES THE COUNT OF ROWS.
	taxi_df_train_with_newFeatures.cache()
	taxi_df_train_with_newFeatures.count()

**出力:**

126050

### モデリング関数への入力用にカテゴリの特徴のインデックス作成とエンコードを実行する

ここでは、モデリング関数への入力用に、カテゴリの特徴のインデックスを作成し、特徴をエンコードする方法について説明します。MLlib のモデリング関数と予測関数では、特徴のカテゴリ入力データを使用する前に、データのインデックスを作成するか、データをエンコードする必要があります。モデルに応じて、さまざまな方法でカテゴリ入力データのインデックス作成またはエンコードを実行する必要があります。

- **ツリー ベースのモデリング**では、カテゴリを数値としてエンコードする必要があります (たとえば、3 つのカテゴリを持つ特徴は、0、1、2 でエンコードできます)。これは、MLlib の [StringIndexer](http://spark.apache.org/docs/latest/ml-features.html#stringindexer) 関数によって提供されます。この関数は、ラベルの文字列型の列を、ラベルの頻度で順序付けられたラベル インデックスの列にエンコードします。入力とデータ処理のために数値でインデックスを作成しますが、ツリー ベースのアルゴリズムでは、これらの数値をカテゴリとして適切に処理するように指定できます。

- **ロジスティック回帰モデルと線形回帰モデル**では、ワンホット エンコードが必要です。たとえば、3 つのカテゴリを持つ特徴は、観察のカテゴリに応じてそれぞれ 0 または 1 が含まれる 3 つの特徴列に展開できます。MLlib には、ワンホット エンコードを実行する [OneHotEncoder](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) 関数が用意されています。このエンコーダーは、ラベル インデックスの列をバイナリ ベクトルの列にマップします。この列に含まれる値は最大でも 1 つだけです。このエンコードにより、数値を持つ特徴を必要とするアルゴリズム (ロジスティック回帰など) をカテゴリの特徴に適用できます。

カテゴリの特徴のインデックス作成とエンコードを実行するコードを次に示します。


	# INDEX AND ENCODE CATEGORICAL FEATURES

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# LOAD PYSPARK LIBRARIES	
	from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, VectorIndexer
	
	# INDEX AND ENCODE VENDOR_ID
	stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
	model = stringIndexer.fit(taxi_df_train_with_newFeatures) # Input data-frame is the cleaned one from above
	indexed = model.transform(taxi_df_train_with_newFeatures)
	encoder = OneHotEncoder(dropLast=False, inputCol="vendorIndex", outputCol="vendorVec")
	encoded1 = encoder.transform(indexed)
	
	# INDEX AND ENCODE RATE_CODE
	stringIndexer = StringIndexer(inputCol="rate_code", outputCol="rateIndex")
	model = stringIndexer.fit(encoded1)
	indexed = model.transform(encoded1)
	encoder = OneHotEncoder(dropLast=False, inputCol="rateIndex", outputCol="rateVec")
	encoded2 = encoder.transform(indexed)
	
	# INDEX AND ENCODE PAYMENT_TYPE
	stringIndexer = StringIndexer(inputCol="payment_type", outputCol="paymentIndex")
	model = stringIndexer.fit(encoded2)
	indexed = model.transform(encoded2)
	encoder = OneHotEncoder(dropLast=False, inputCol="paymentIndex", outputCol="paymentVec")
	encoded3 = encoder.transform(indexed)
	
	# INDEX AND TRAFFIC TIME BINS
	stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
	model = stringIndexer.fit(encoded3)
	indexed = model.transform(encoded3)
	encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
	encodedFinal = encoder.transform(indexed)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

上記のセルの実行に要した時間: 1.28 秒

### ML 関数への入力用にラベル付きポイント オブジェクトを作成する

このセクションのコードでは、ラベル付きポイント データ型としてカテゴリ テキスト データのインデックスを作成し、MLlib ロジスティック回帰モデルや他の分類モデルのトレーニングとテストに使用できるように、カテゴリ テキスト データをエンコードする方法を示します。ラベル付きポイント オブジェクトは、MLlib のほとんどの ML アルゴリズムで入力データとして必要とされる方法でフォーマットされた Resilient Distributed Dataset (RDD) です。[ラベル付きポイント](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point)は、ラベル/応答に関連付けられたローカル ベクトル (密または疎) です。

このセクションのコードでは、[ラベル付きポイント](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) データ型としてカテゴリ テキスト データのインデックスを作成し、MLlib ロジスティック回帰モデルや他の分類モデルのトレーニングとテストに使用できるように、カテゴリ テキスト データをエンコードする方法を示します。ラベル付きポイント オブジェクトは、ラベル (ターゲット変数/応答変数) と特徴ベクトルで構成された Resilient Distributed Dataset (RDD) です。この形式は、MLlib の多くの ML アルゴリズムで入力として必要とされます。

以下に示したのは、二項分類に使用するテキストの特徴のインデックスを作成してエンコードするコードです。

	# FUNCTIONS FOR BINARY CLASSIFICATION

	# LOAD LIBRARIES
	from pyspark.mllib.regression import LabeledPoint
	from numpy import array

	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingBinary(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
	                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
	                         line.trip_distance, line.fare_amount])
	    labPt = LabeledPoint(line.tipped, features)
	    return  labPt
	
	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC RERESSION MODELS
	def parseRowOneHotBinary(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
	                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
	    labPt = LabeledPoint(line.tipped, features)
	    return  labPt


以下に示したのは、線形回帰分析に使用するカテゴリ テキストの特徴をエンコードしてインデックスを作成するコードです。

	# FUNCTIONS FOR REGRESSION WITH TIP AMOUNT AS TARGET VARIABLE

	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingRegression(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
	                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
	                         line.trip_distance, line.fare_amount])

	    labPt = LabeledPoint(line.tip_amount, features)
	    return  labPt
	
	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
	def parseRowOneHotRegression(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
	                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
	    labPt = LabeledPoint(line.tip_amount, features)
	    return  labPt


### データのランダム サブサンプリングを作成し、トレーニング セットとテスト セットに分ける

次のコードでは、データのランダム サンプリングを作成します (ここでは 25% を使用)。この例のデータセットのサイズでは、ランダム サンプリングを作成する必要はありませんが、必要に応じて独自の問題にランダム サンプリングを使用する方法がわかるように、ここではサンプリングの方法を示します。サンプル数が多い場合、ランダム サンプリングを作成することで、モデルのトレーニング時に時間を大幅に節約できます。次に、分類および回帰モデリングで使用するために、サンプルをトレーニング用 (ここでは 75%) とテスト用 (ここでは 25%) に分けます。


	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.sql.functions import rand

	# SPECIFY SAMPLING AND SPLITTING FRACTIONS
	samplingFraction = 0.25;
	trainingFraction = 0.75; testingFraction = (1-trainingFraction);
	seed = 1234;
	encodedFinalSampled = encodedFinal.sample(False, samplingFraction, seed=seed)
	
	# SPLIT SAMPLED DATA-FRAME INTO TRAIN/TEST
	# INCLUDE RAND COLUMN FOR CREATING CROSS-VALIDATION FOLDS (FOR USE LATER IN AN ADVANCED TOPIC)
	dfTmpRand = encodedFinalSampled.select("*", rand(0).alias("rand"));
	trainData, testData = dfTmpRand.randomSplit([trainingFraction, testingFraction], seed=seed);
	
	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTRAINbinary = trainData.map(parseRowIndexingBinary)
	indexedTESTbinary = testData.map(parseRowIndexingBinary)
	oneHotTRAINbinary = trainData.map(parseRowOneHotBinary)
	oneHotTESTbinary = testData.map(parseRowOneHotBinary)
	
	# FOR REGRESSION TRAINING AND TESTING
	indexedTRAINreg = trainData.map(parseRowIndexingRegression)
	indexedTESTreg = testData.map(parseRowIndexingRegression)
	oneHotTRAINreg = trainData.map(parseRowOneHotRegression)
	oneHotTESTreg = testData.map(parseRowOneHotRegression)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

上記のセルの実行に要した時間: 0.24 秒


### 特徴のスケーリング

特徴のスケーリングはデータ正規化とも呼ばれ、目的関数で広く分散した値を持つ特徴に過大な重みが与えられないようにします。特徴のスケーリングのコードでは、[StandardScaler](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.feature.StandardScaler) を使用して特徴を単価差異にスケーリングします。これは、確率的勾配降下 (SGD) による線形回帰に使用するために、MLlib で提供されています。SGD は、正規化回帰やサポート ベクター マシン (SVM) などの他のさまざまな機械学習モデルのトレーニングに広く使用されているアルゴリズムです。

>[AZURE.NOTE] LinearRegressionWithSGD アルゴリズムは特徴のスケーリングの影響を受けやすいことがわかりました。

正規化線形 SGD アルゴリズムで使用するために変数をスケーリングするコードを次に示します。

	# FEATURE SCALING

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.regression import LabeledPoint
	from pyspark.mllib.linalg import Vectors
	from pyspark.mllib.feature import StandardScaler, StandardScalerModel
	from pyspark.mllib.util import MLUtils
	
	# SCALE VARIABLES FOR REGULARIZED LINEAR SGD ALGORITHM
	label = oneHotTRAINreg.map(lambda x: x.label)
	features = oneHotTRAINreg.map(lambda x: x.features)
	scaler = StandardScaler(withMean=False, withStd=True).fit(features)
	dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
	oneHotTRAINregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))
	
	label = oneHotTESTreg.map(lambda x: x.label)
	features = oneHotTESTreg.map(lambda x: x.features)
	scaler = StandardScaler(withMean=False, withStd=True).fit(features)
	dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
	oneHotTESTregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

上記のセルの実行に要した時間: 13.17 秒


### オブジェクトをメモリにキャッシュする

ML アルゴリズムのトレーニングとテストの所要時間は、分類、回帰、およびスケーリングされた特徴に使用される入力データ フレーム オブジェクトをキャッシュすることで短縮できます。

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTRAINbinary.cache()
	indexedTESTbinary.cache()
	oneHotTRAINbinary.cache()
	oneHotTESTbinary.cache()
	
	# FOR REGRESSION TRAINING AND TESTING
	indexedTRAINreg.cache()
	indexedTESTreg.cache()
	oneHotTRAINreg.cache()
	oneHotTESTreg.cache()
	
	# SCALED FEATURES
	oneHotTRAINregScaled.cache()
	oneHotTESTregScaled.cache()
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

上記のセルの実行に要した時間: 0.15 秒


## 二項分類モデルを使用してチップが支払われるかどうかを予測する

このセクションでは、タクシーの乗車でチップが支払われるかどうかを予測する二項分類タスクで 3 つのモデルを使用する方法を示します。使用するモデルは次のとおりです。

- 正規化ロジスティック回帰
- ランダム フォレスト モデル
- 勾配ブースティング ツリー

各モデル構築コードのセクションは、次のステップに分けることができます。

1. 1 つのパラメーター セットを使用する**モデルのトレーニング** データ
2. メトリックを含むテスト データ セットでの**モデルの評価**
3. 今後使用できるようにするための BLOB への**モデルの保存**

### ロジスティック回帰を使用した分類

このセクションのコードでは、NYC タクシーの乗車と料金のデータセットで、乗車でチップが支払われるかどうかを予測するロジスティック回帰モデル ([LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm) を使用) をトレーニング、評価し、保存する方法を示します。

**CV とハイパーパラメーター スイープを使用してロジスティック回帰モデルをトレーニングする**

	# LOGISTIC REGRESSION CLASSIFICATION WITH CV AND HYPERPARAMETER SWEEPING

	# GET ACCURACY FOR HYPERPARAMETERS BASED ON CROSS-VALIDATION IN TRAINING DATA-SET

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD LIBRARIES
	from pyspark.mllib.classification import LogisticRegressionWithLBFGS 
	from sklearn.metrics import roc_curve,auc
	from pyspark.mllib.evaluation import BinaryClassificationMetrics
	from pyspark.mllib.evaluation import MulticlassMetrics
	
	
	# CREATE MODEL WITH ONE SET OF PARAMETERS
	logitModel = LogisticRegressionWithLBFGS.train(oneHotTRAINbinary, iterations=20, initialWeights=None, 
	                                               regParam=0.01, regType='l2', intercept=True, corrections=10, 
	                                               tolerance=0.0001, validateData=True, numClasses=2)
	
	# PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
	# NOTE: There are 20 coefficient terms for the 10 features, 
	#       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
	print("Coefficients: " + str(logitModel.weights))
	print("Intercept: " + str(logitModel.intercept))
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**出力:**

係数: [0.0082065285375, -0.0223675576104, -0.0183812028036, -3.48124578069e-05, -0.00247646947233, -0.00165897881503, 0.0675394837328, -0.111823113101, -0.324609912762, -0.204549780032, -1.36499216354, 0.591088507921, -0.664263411392, -1.00439726852, 3.46567827545, -3.51025855172, -0.0471341112232, -0.043521833294, 0.000243375810385, 0.054518719222]

切片: -0.0111216486893

上記のセルの実行に要した時間: 14.43 秒

**標準メトリックで二項分類モデルを評価する**

	#EVALUATE LOGISTIC REGRESSION MODEL WITH LBFGS

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# PREDICT ON TEST DATA WITH MODEL
	predictionAndLabels = oneHotTESTbinary.map(lambda lp: (float(logitModel.predict(lp.features)), lp.label))
	
	# INSTANTIATE METRICS OBJECT
	metrics = BinaryClassificationMetrics(predictionAndLabels)

	# AREA UNDER PRECISION-RECALL CURVE
	print("Area under PR = %s" % metrics.areaUnderPR)

	# AREA UNDER ROC CURVE
	print("Area under ROC = %s" % metrics.areaUnderROC)
	metrics = MulticlassMetrics(predictionAndLabels)

	# OVERALL STATISTICS
	precision = metrics.precision()
	recall = metrics.recall()
	f1Score = metrics.fMeasure()
	print("Summary Stats")
	print("Precision = %s" % precision)
	print("Recall = %s" % recall)
	print("F1 Score = %s" % f1Score)


	## SAVE MODEL WITH DATE-STAMP
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp;
	dirfilename = modelDir + logisticregressionfilename;
	logitModel.save(sc, dirfilename);
	
	# OUTPUT PROBABILITIES AND REGISTER TEMP TABLE
	logitModel.clearThreshold(); # This clears threshold for classification (0.5) and outputs probabilities
	predictionAndLabelsDF = predictionAndLabels.toDF()
	predictionAndLabelsDF.registerTempTable("tmp_results");
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**出力:**

PR 下面積 = 0.985297691373

ROC 下面積 = 0.983714670256

要約統計量

精度 = 0.984304060189

再現率 = 0.984304060189

F1 スコア = 0.984304060189

上記のセルの実行に要した時間: 57.61 秒

**ROC 曲線をプロットする**

*predictionAndLabelsDF* は、既出のセルでテーブル (*tmp\_results*) として登録されています。*tmp\_results* を使用してクエリを実行したり、sqlResults データフレームに結果を出力してプロットしたりすることができます。次にコードを示します。


	# QUERY RESULTS                              
	%%sql -q -o sqlResults
	SELECT * from tmp_results


予測を行って ROC 曲線をプロットするコードを次に示します。

	# MAKE PREDICTIONS AND PLOT ROC-CURVE

	# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
	%%local
	%matplotlib inline
	from sklearn.metrics import roc_curve,auc

	# MAKE PREDICTIONS
	predictions_pddf = test_predictions.rename(columns={'_1': 'probability', '_2': 'label'})
	prob = predictions_pddf["probability"] 
	fpr, tpr, thresholds = roc_curve(predictions_pddf['label'], prob, pos_label=1);
	roc_auc = auc(fpr, tpr)

	# PLOT ROC CURVE
	plt.figure(figsize=(5,5))
	plt.plot(fpr, tpr, label='ROC curve (area = %0.2f)' % roc_auc)
	plt.plot([0, 1], [0, 1], 'k--')
	plt.xlim([0.0, 1.0])
	plt.ylim([0.0, 1.05])
	plt.xlabel('False Positive Rate')
	plt.ylabel('True Positive Rate')
	plt.title('ROC Curve')
	plt.legend(loc="lower right")
	plt.show()
	

**出力:**

![Logistic regression ROC curve.png](./media/machine-learning-data-science-spark-data-exploration-modeling/logistic-regression-roc-curve.png)


### ランダム フォレスト分類

このセクションのコードでは、NYC タクシー乗車および料金データセットで、乗車でチップが支払われるかどうかを予測するランダム フォレスト モデルをトレーニング、評価し、保存する方法を示します。
	
	#PREDICT WHETHER A TIP IS PAID OR NOT USING RANDOM FOREST

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import RandomForest, RandomForestModel
	from pyspark.mllib.util import MLUtils
	from pyspark.mllib.evaluation import BinaryClassificationMetrics
	from pyspark.mllib.evaluation import MulticlassMetrics
	
	# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	
	# TRAIN RANDOMFOREST MODEL
	rfModel = RandomForest.trainClassifier(indexedTRAINbinary, numClasses=2, 
	                                       categoricalFeaturesInfo=categoricalFeaturesInfo,
	                                       numTrees=25, featureSubsetStrategy="auto",
	                                       impurity='gini', maxDepth=5, maxBins=32)
	## UN-COMMENT IF YOU WANT TO PRINT TREES
	#print('Learned classification forest model:')
	#print(rfModel.toDebugString())
	
	# PREDICT ON TEST DATA AND EVALUATE
	predictions = rfModel.predict(indexedTESTbinary.map(lambda x: x.features))
	predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)
	
	# AREA UNDER ROC CURVE
	metrics = BinaryClassificationMetrics(predictionAndLabels)
	print("Area under ROC = %s" % metrics.areaUnderROC)
	
	# PERSIST MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfclassificationfilename = "RandomForestClassification_" + datestamp;
	dirfilename = modelDir + rfclassificationfilename;
	
	rfModel.save(sc, dirfilename);
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

ROC 下面積 = 0.985297691373

上記のセルの実行に要した時間: 31.09 秒


### 勾配ブースティング ツリー分類

このセクションのコードでは、NYC タクシー乗車および料金データセットで、乗車でチップが支払われるかどうかを予測する勾配ブースティング ツリー モデルをトレーニング、評価し、保存する方法を示します。

	#PREDICT WHETHER A TIP IS PAID OR NOT USING GRADIENT BOOSTING TREES

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
	
	# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	
	gbtModel = GradientBoostedTrees.trainClassifier(indexedTRAINbinary, categoricalFeaturesInfo=categoricalFeaturesInfo, numIterations=5)
	## UNCOMMENT IF YOU WANT TO PRINT TREE DETAILS
	#print('Learned classification GBT model:')
	#print(bgtModel.toDebugString())
	
	# PREDICT ON TEST DATA AND EVALUATE
	predictions = gbtModel.predict(indexedTESTbinary.map(lambda x: x.features))
	predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)
	
	# AREA UNDER ROC CURVE
	metrics = BinaryClassificationMetrics(predictionAndLabels)
	print("Area under ROC = %s" % metrics.areaUnderROC)
	
	# PERSIST MODEL IN A BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp;
	dirfilename = modelDir + btclassificationfilename;
	
	gbtModel.save(sc, dirfilename)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**出力:**

ROC 下面積 = 0.985297691373

上記のセルの実行に要した時間: 19.76 秒


## 回帰モデルを使用してタクシー乗車でのチップの金額を予測する

このセクションでは、チップの他の特徴に基づいて、タクシーの乗車で支払われるチップの金額を予測する回帰タスクで 3 つのモデルを使用する方法を示します。使用するモデルは次のとおりです。

- 正規化線形回帰
- ランダム フォレスト
- 勾配ブースティング ツリー

これらのモデルについては、「はじめに」で説明しました。各モデル構築コードのセクションは、次のステップに分けることができます。

1. 1 つのパラメーター セットを使用する**モデルのトレーニング** データ
2. メトリックを含むテスト データ セットでの**モデルの評価**
3. 今後使用できるようにするための BLOB への**モデルの保存**

### SGD を使用した線形回帰 

このセクションのコードでは、スケーリングされた特徴を使用して、最適化に確率的勾配降下 (SGD) を使用する線形回帰をトレーニングする方法と、このモデルにスコアを付け、評価し、Azure BLOB Storage (WASB) に保存する方法を示します。

>[AZURE.TIP] 経験上、LinearRegressionWithSGD モデルの収束で問題が発生する可能性があるため、有効なモデルを入手するために、パラメーターを慎重に変更/最適化する必要があります。収束には変数のスケーリングがきわめて有効です。


	#PREDICT TIP AMOUNTS USING LINEAR REGRESSION WITH SGD

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD LIBRARIES
	from pyspark.mllib.regression import LabeledPoint, LinearRegressionWithSGD, LinearRegressionModel
	from pyspark.mllib.evaluation import RegressionMetrics
	from scipy import stats
	
	# USE SCALED FEATURES TO TRAIN MODEL
	linearModel = LinearRegressionWithSGD.train(oneHotTRAINregScaled, iterations=100, step = 0.1, regType='l2', regParam=0.1, intercept = True)

	# PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
	# NOTE: There are 20 coefficient terms for the 10 features, 
	#       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
	print("Coefficients: " + str(linearModel.weights))
	print("Intercept: " + str(linearModel.intercept))
	
	# SCORE ON SCALED TEST DATA-SET & EVALUATE
	predictionAndLabels = oneHotTESTregScaled.map(lambda lp: (float(linearModel.predict(lp.features)), lp.label))
	testMetrics = RegressionMetrics(predictionAndLabels)
	
	# PRINT TEST METRICS
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)
	
	# SAVE MODEL WITH DATE-STAMP IN THE DEFAULT BLOB FOR THE CLUSTER
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
	dirfilename = modelDir + linearregressionfilename;
	
	linearModel.save(sc, dirfilename)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

係数: [0.00457675809917, -0.0226314167349, -0.0191910355236, 0.246793409578, 0.312047890459, 0.359634405999, 0.00928692253981, -0.000987181489428, -0.0888306617845, 0.0569376211553, 0.115519551711, 0.149250164995, -0.00990211159703, -0.00637410344522, 0.545083566179, -0.536756072402, 0.0105762393099, -0.0130117577055, 0.0129304737772, -0.00171065945959]

切片: 0.853872718283

RMSE = 1.24190115863

R-sqr = 0.608017146081

上記のセルの実行に要した時間: 58.42 秒


### ランダム フォレスト回帰

このセクションのコードでは、NYC タクシー乗車データでチップの金額を予測するランダム フォレスト回帰をトレーニング、評価し、保存する方法を示します。


	#PREDICT TIP AMOUNTS USING RANDOM FOREST

	# RECORD START TIME
	timestart= datetime.datetime.now()

	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import RandomForest, RandomForestModel
	from pyspark.mllib.util import MLUtils
	from pyspark.mllib.evaluation import RegressionMetrics
	
	
	## TRAIN MODEL
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
	                                    numTrees=25, featureSubsetStrategy="auto",
	                                    impurity='variance', maxDepth=10, maxBins=32)
	## UN-COMMENT IF YOU WANT TO PRING TREES
	#print('Learned classification forest model:')
	#print(rfModel.toDebugString())
	
	## PREDICT AND EVALUATE ON TEST DATA-SET
	predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
	predictionAndLabels = oneHotTESTreg.map(lambda lp: lp.label).zip(predictions)

	# TEST METRICS
	testMetrics = RegressionMetrics(predictionAndLabels)
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)
	
	# SAVE MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfregressionfilename = "RandomForestRegression_" + datestamp;
	dirfilename = modelDir + rfregressionfilename;
	
	rfModel.save(sc, dirfilename);
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

RMSE = 0.891209218139

R-sqr = 0.759661334921

上記のセルの実行に要した時間: 49.21 秒


### 勾配ブースティング ツリー回帰

このセクションのコードでは、NYC タクシー乗車データでチップの金額を予測する勾配ブースティング ツリー モデルをトレーニング、評価し、保存する方法を示します。

**トレーニングと評価**

	#PREDICT TIP AMOUNTS USING GRADIENT BOOSTING TREES

	# RECORD START TIME
	timestart= datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
	from pyspark.mllib.util import MLUtils
	
	## TRAIN MODEL
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	gbtModel = GradientBoostedTrees.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo, 
	                                                numIterations=10, maxBins=32, maxDepth = 4, learningRate=0.1)
	
	## EVALUATE A TEST DATA-SET
	predictions = gbtModel.predict(indexedTESTreg.map(lambda x: x.features))
	predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)

	# TEST METRICS
	testMetrics = RegressionMetrics(predictionAndLabels)
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)

	# SAVE MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btregressionfilename = "GradientBoostingTreeRegression_" + datestamp;
	dirfilename = modelDir + btregressionfilename;
	gbtModel.save(sc, dirfilename)
	
	# CONVER RESULTS TO DF AND REGISER TEMP TABLE
	test_predictions = sqlContext.createDataFrame(predictionAndLabels)
	test_predictions.registerTempTable("tmp_results");
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

RMSE = 0.908473148639

R-sqr = 0.753835096681

上記のセルの実行に要した時間: 34.52 秒

**プロット**

*tmp\_results* は、既出のセルで Hive テーブルとして登録されています。このテーブルからの結果を *sqlResults* データフレームに出力し、プロットに使用しています。次にコードを示します。

	# PLOT SCATTER-PLOT BETWEEN ACTUAL AND PREDICTED TIP VALUES

	# SELECT RESULTS
	%%sql -q -o sqlResults
	SELECT * from tmp_results

Jupyter サーバーを使用してデータをプロットするコードを次に示します。

	# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
	%%local
	%matplotlib inline
	import numpy as np

	# PLOT 
	ax = test_predictions_pddf.plot(kind='scatter', figsize = (6,6), x='_1', y='_2', color='blue', alpha = 0.25, label='Actual vs. predicted');
	fit = np.polyfit(test_predictions_pddf['_1'], test_predictions_pddf['_2'], deg=1)
	ax.set_title('Actual vs. Predicted Tip Amounts ($)')
	ax.set_xlabel("Actual")
	ax.set_ylabel("Predicted")
	ax.plot(test_predictions_pddf['_1'], fit[0] * test_predictions_pddf['_1'] + fit[1], color='magenta')
	plt.axis([-1, 20, -1, 20])
	plt.show(ax)
	

**出力:**

![Actual-vs-predicted-tip-amounts](./media/machine-learning-data-science-spark-data-exploration-modeling/actual-vs-predicted-tips.png)

	
## メモリからオブジェクトをクリーンアップする

メモリにキャッシュされたオブジェクトを削除するには、`unpersist()` を使用します。
		
	# REMOVE ORIGINAL DFs
	taxi_df_train_cleaned.unpersist()
	taxi_df_train_with_newFeatures.unpersist()
	
	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTRAINbinary.unpersist()
	indexedTESTbinary.unpersist()
	oneHotTRAINbinary.unpersist()
	oneHotTESTbinary.unpersist()
	
	# FOR REGRESSION TRAINING AND TESTING
	indexedTRAINreg.unpersist()
	indexedTESTreg.unpersist()
	oneHotTRAINreg.unpersist()
	oneHotTESTreg.unpersist()
	
	# SCALED FEATURES
	oneHotTRAINregScaled.unpersist()
	oneHotTESTregScaled.unpersist()


## 使用およびスコア付けのためにモデルのストレージの場所を記録する

[Spark で構築した機械学習モデルのスコア付けと評価](machine-learning-data-science-spark-model-consumption.md)に関するページで説明する独立データセットを使用し、スコアを付けるために、ここで作成した保存済みモデルを含むファイル名をコピーして、Consumption Jupyter Notebook に貼り付ける必要があります。必要なモデル ファイルのパスを出力するコードを次に示します。

	# MODEL FILE LOCATIONS FOR CONSUMPTION
	print "logisticRegFileLoc = modelDir + "" + logisticregressionfilename + """;
	print "linearRegFileLoc = modelDir + "" + linearregressionfilename + """;
	print "randomForestClassificationFileLoc = modelDir + "" + rfclassificationfilename + """;
	print "randomForestRegFileLoc = modelDir + "" + rfregressionfilename + """;
	print "BoostedTreeClassificationFileLoc = modelDir + "" + btclassificationfilename + """;
	print "BoostedTreeRegressionFileLoc = modelDir + "" + btregressionfilename + """;


**出力**

logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-05-0317_03\_23.516568"

linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-05-0317_05\_21.577773"

randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-05-0317_04\_11.950206"

randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-05-0317_06\_08.723736"

BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-05-0317_04\_36.346583"

BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-05-0317_06\_51.737282"


## 次の手順

Spark MlLib を使用して回帰モデルと分類モデルを作成しました。これで、これらのモデルにスコアを付け、評価する方法を学習する準備ができました。高度なデータの探索と Notebook のモデリングでは、クロス検証、ハイパー パラメーター スイープやモデルの評価などに深く踏み込みます。

**モデルの使用:** このトピックで作成した分類モデルと回帰モデルにスコアを付け、評価する方法については、[Spark で構築した機械学習モデルのスコア付けと評価](machine-learning-data-science-spark-model-consumption.md)に関するページをご覧ください。

**クロス検証とハイパーパラメーター スイープ**: クロス検証とハイパーパラメーター スイープを使用したモデルのトレーニング方法については、「[Spark を使用した高度なデータ探索とモデリング](machine-learning-data-science-spark-advanced-data-exploration-modeling.md)」をご覧ください。

<!---HONumber=AcomDC_0921_2016-->