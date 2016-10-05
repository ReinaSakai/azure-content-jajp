<properties
	pageTitle="Spark で構築した機械学習モデルのスコア付け | Microsoft Azure"
	description="Azure Blob Storage (WASB) に保存されている学習モデルをスコア付けする方法です。"
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

# Spark で構築した機械学習モデルのスコア付け 

[AZURE.INCLUDE [machine-learning-spark-modeling](../../includes/machine-learning-spark-modeling.md)]

このトピックでは、Spark MLlib を使用して構築され、Azure Blob ストレージ (WASB) に保存された機械学習 (ML) モデルを読み込む方法と、やはり WASB に保存されているデータセットを使用してその機械学習モデルをスコア付けする方法を説明します。入力データを前処理し、MLlib ツールキットのインデックス機能とエンコード機能を使用して特徴を変換する方法と、ML モデルをスコア付けする際に入力として使用できるラベル付けされたポイント データ オブジェクトの作成方法を示します。スコア付けに使用するモデルには、線形回帰、ロジスティック回帰、ランダム フォレスト モデル、勾配ブースティング ツリー モデルなどがあります。


## 前提条件

1. Azure アカウントと HDInsight Spark が必要となります。このチュートリアルを実行するには HDInsight 3.4 Spark 1.6 クラスターが必要です。このチュートリアルの要件、チュートリアルで使用される 2013 年 NYC タクシー データの説明、Spark クラスターで Jupyter Notebook のコードを実行する方法については、「[Overview of Data Science using Spark on Azure HDInsight (Azure HDInsight での Spark を使用したデータ サイエンスの概要)](machine-learning-data-science-spark-overview.md)」をご覧ください。このトピックのコード サンプルが含まれた **machine-learning-data-science-spark-data-exploration-modeling.ipynb** ノートブックは、[GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/pySpark) で入手できます。

2. ここでスコア付けする機械学習モデルは、「[Spark を使用したデータ探索とモデリング](machine-learning-data-science-spark-data-exploration-modeling.md)」のチュートリアルでも作成できます。


[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]
 

## セットアップ: ストレージの場所、ライブラリ、プリセットの Spark コンテキスト

Spark は Azure Storage Blob (WASB) の読み取りと書き込みを実行できます。そのため、そこに保存されている既存データはすべて Spark を使って処理でき、結果も再び WASB に保存できます。

WASB にモデルやファイルを保存するには、パスを正しく指定する必要があります。Spark クラスターに接続されている既定のコンテナーは、*"wasb///"* で始まるパスを使用して参照できます。次のコード サンプルでは、読み取るデータの場所と、モデルの出力の保存先となるモデル ストレージ ディレクトリのパスを指定しています。


### WASB のストレージの場所となるディレクトリ パスの設定

モデルが保存されるのは、"wasb:///user/remoteuser/NYCTaxi/Models" です。このパスが正しく設定されていない場合、スコア付けするモデルが読み込まれません。

スコア付けの結果が保存されるのは、"wasb:///user/remoteuser/NYCTaxi/ScoredResults" です。フォルダーへのパスを正しく指定しなければ、このフォルダーには結果が保存されません。


>[AZURE.NOTE] **machine-learning-data-science-spark-data-exploration-modeling.ipynb** ノートブックの最後のセルの出力からファイル パスの場所をコピーし、このコード内のプレースホルダーに貼り付けることができます。


ディレクトリ パスを設定するコードを次に示します。

	# LOCATION OF DATA TO BE SCORED (TEST DATA)
	taxi_test_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Test.tsv";
	
	# SET THE MODEL STORAGE DIRECTORY PATH 
	# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
	modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 
	
	# SET SCORDED RESULT DIRECTORY PATH
	# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
	scoredResultDir = "wasb:///user/remoteuser/NYCTaxi/ScoredResults/"; 
	
	# FILE LOCATIONS FOR THE MODELS TO BE SCORED
	logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-04-1817_40_35.796789"
	linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-04-1817_44_00.993832"
	randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-04-1817_42_58.899412"
	randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-04-1817_44_27.204734"
	BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-04-1817_43_16.354770"
	BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-04-1817_44_46.206262"

	# RECORD START TIME
	import datetime
	datetime.datetime.now()

**出力:**

datetime.datetime(2016, 4, 25, 23, 56, 19, 229403)


### ライブラリのインポート

次のコードを使用して、Spark コンテキストを設定し、必要なライブラリをインポートします。

	#IMPORT LIBRARIES
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


## データの取り込みとクリーンなデータ フレームの作成

このセクションでは、スコア付けするデータの取り込みに必要な一連のタスクを実行するコードを紹介します。タクシーの乗車データと料金ファイル (.tsv ファイルとして保存されています) を結合した 0.1% サンプルを読み取り、データの形式を設定して、クリーンなデータ フレームを作成します。

タクシーの乗車データと料金ファイルは、「[Team Data Science Process の活用: HDInsight Hadoop クラスターの使用](machine-learning-data-science-process-hive-walkthrough.md)」で説明されている手順に基づいて結合されています。

	# INGEST DATA AND CREATE A CLEANED DATA FRAME

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# IMPORT FILE FROM PUBLIC BLOB
	taxi_test_file = sc.textFile(taxi_test_file_loc)
	
	# GET SCHEMA OF THE FILE FROM HEADER
	taxi_header = taxi_test_file.filter(lambda l: "medallion" in l)
	
	# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
	taxi_temp = taxi_test_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
	        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
	                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
	                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))
	    
	# GET SCHEMA OF THE FILE FROM HEADER
	schema_string = taxi_test_file.first()
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
	
	# CREATE DATA FRAME
	taxi_df_test = sqlContext.createDataFrame(taxi_temp, taxi_schema)
	
	# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
	taxi_df_test_cleaned = taxi_df_test.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
	    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
	    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
	    .drop('direct_distance').drop('surcharge')\
	    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )
	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	taxi_df_test_cleaned.cache()
	taxi_df_test_cleaned.count()
	
	# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
	taxi_df_test_cleaned.registerTempTable("taxi_test")
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

上記のセルの実行に要した時間: 46.37 秒


## Spark でスコア付けを行うためのデータの準備 

このセクションでは、カテゴリの特徴のインデックスを作成し、エンコードし、スケーリングして、MLlib 監視学習アルゴリズムで分類と回帰分析に使用できるように準備する方法を説明します。

### 特徴の変換: カテゴリの特徴のインデックスを作成してエンコードし、モデルに入力してスコア付けできるようにする 

このセクションでは、`StringIndexer` を使用してカテゴリ データのインデックスを作成し、`OneHotEncoder` 入力を使って特徴をモデルにエンコードする方法について説明します。

[StringIndexer](http://spark.apache.org/docs/latest/ml-features.html#stringindexer) によって、ラベルの文字列型の列をラベル インデックスの列にエンコードします。インデックスの順序は、ラベルの頻度順になります。

[OneHotEncoder](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) によって、ラベル インデックスの列をバイナリのベクトルの列にマッピングします。値は最大でも 1 つのみです。このエンコードによって、ロジスティック回帰などの連続する値を持つ特徴を受け取るアルゴリズムをカテゴリの特徴に適用できます。
	
	#INDEX AND ONE-HOT ENCODE CATEGORICAL FEATURES

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, VectorIndexer
	
	# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
	sqlStatement = """
	    SELECT *,
	    CASE
	     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
	     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
	     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
	     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
	    END as TrafficTimeBins
	    FROM taxi_test 
	"""
	taxi_df_test_with_newFeatures = sqlContext.sql(sqlStatement)
	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	taxi_df_test_with_newFeatures.cache()
	taxi_df_test_with_newFeatures.count()
	
	# INDEX AND ONE-HOT ENCODING
	stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
	model = stringIndexer.fit(taxi_df_test_with_newFeatures) # Input data-frame is the cleaned one from above
	indexed = model.transform(taxi_df_test_with_newFeatures)
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
	
	# INDEX AND ENCODE TRAFFIC TIME BINS
	stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
	model = stringIndexer.fit(encoded3)
	indexed = model.transform(encoded3)
	encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
	encodedFinal = encoder.transform(indexed)
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

上記のセルの実行に要した時間: 5.37 秒


### モデルに入力する特徴配列を含む RDD オブジェクトの作成

このセクションのコードでは、RDD オブジェクトとしてカテゴリ テキスト データのインデックスを作成し、それをワン ホット エンコーディングすることで、MLlib ロジスティック回帰モデルやツリーベース モデルのトレーニングおよびテストに使用できるようにする方法を示します。インデックス付きデータは、[Resilient Distributed Dataset (RDD)](http://spark.apache.org/docs/latest/api/java/org/apache/spark/rdd/RDD.html) オブジェクトに格納されます。これは、Spark における基本的な抽象化です。RDD オブジェクトは、パーティション分割された変更できない要素のコレクションであり、Spark と平行して処理することができます。

さらにここでは、確率的勾配降下 (SGD) による線形回帰に使用できるように、MLlib の `StandardScalar` を使ってデータをスケーリングする方法を示すコードも示しています。SGD はさまざまな機械学習モデルのトレーニングに広く使用されているアルゴリズムです。[StandardScaler](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.feature.StandardScaler) を使用して、特徴を単価差異にスケーリングします。この特徴のスケーリングはデータの正規化とも呼ばれ、目標関数において幅広く分散した値を持つ特徴に過大な重みが与えられないようにします。


	# CREATE RDD OBJECTS WITH FEATURE ARRAYS FOR INPUT INTO MODELS

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# IMPORT LIBRARIES
	from pyspark.mllib.linalg import Vectors
	from pyspark.mllib.feature import StandardScaler, StandardScalerModel
	from pyspark.mllib.util import MLUtils
	from numpy import array
	
	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingBinary(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
	                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
	                         line.trip_distance, line.fare_amount])
	    return  features
	
	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC RERESSION MODELS
	def parseRowOneHotBinary(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
	                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
	    return  features
	
	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingRegression(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
	                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
	                         line.trip_distance, line.fare_amount])
	    return  features
	
	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
	def parseRowOneHotRegression(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
	                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
	    return  features

	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTESTbinary = encodedFinal.map(parseRowIndexingBinary)
	oneHotTESTbinary = encodedFinal.map(parseRowOneHotBinary)
	
	# FOR REGRESSION CLASSIFICATION TRAINING AND TESTING
	indexedTESTreg = encodedFinal.map(parseRowIndexingRegression)
	oneHotTESTreg = encodedFinal.map(parseRowOneHotRegression)
	
	# SCALING FEATURES FOR LINEARREGRESSIONWITHSGD MODEL
	scaler = StandardScaler(withMean=False, withStd=True).fit(oneHotTESTreg)
	oneHotTESTregScaled = scaler.transform(oneHotTESTreg)
	
	# CACHE RDDS IN MEMORY
	indexedTESTbinary.cache();
	oneHotTESTbinary.cache();
	indexedTESTreg.cache();
	oneHotTESTreg.cache();
	oneHotTESTregScaled.cache();
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**出力:**

上記のセルの実行に要した時間: 11.72 秒


## ロジスティック回帰モデルを利用したスコア付けと BLOB への出力の保存

このセクションのコードでは、Azure Blob Storage に保存されているロジスティック回帰モデルを読み込み、それを使ってタクシーの乗車時にチップが支払われるかどうかを予測し、標準の分類メトリックを使ってスコア付けした後、結果を Blob Storage に保存してプロットする方法を示します。スコア付けの結果は RDD オブジェクトに保存されます。


	# SCORE AND EVALUATE LOGISTIC REGRESSION MODEL

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# IMPORT LIBRARIES
	from pyspark.mllib.classification import LogisticRegressionModel
	
	## LOAD SAVED MODEL
	savedModel = LogisticRegressionModel.load(sc, logisticRegFileLoc)
	predictions = oneHotTESTbinary.map(lambda features: (float(savedModel.predict(features))))
	
	## SAVE SCORED RESULTS (RDD) TO BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp + ".txt";
	dirfilename = scoredResultDir + logisticregressionfilename;
	predictions.saveAsTextFile(dirfilename)
	
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**出力:**

上記のセルの実行に要した時間: 19.22 秒


## 線形回帰モデルのスコア付け

[LinearRegressionWithSGD](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LinearRegressionWithSGD) により、確率的勾配降下 (SGD) を使用した線形回帰モデルをトレーニングすることで、チップの金額を予測するための最適化を行いました。

このセクションのコードでは、Azure Blob Storage ストレージから線形回帰モデルを読み込み、スケーリングされた変数を使用してスコア付けを行った後、BLOB に結果を保存する方法を示します。

	#SCORE LINEAR REGRESSION MODEL

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	#LOAD LIBRARIES​
	from pyspark.mllib.regression import LinearRegressionWithSGD, LinearRegressionModel
	
	# LOAD MODEL AND SCORE USING **SCALED VARIABLES**
	savedModel = LinearRegressionModel.load(sc, linearRegFileLoc)
	predictions = oneHotTESTregScaled.map(lambda features: (float(savedModel.predict(features))))
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
	dirfilename = scoredResultDir + linearregressionfilename;
	predictions.saveAsTextFile(dirfilename)
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL​
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**出力:**

上記のセルの実行に要した時間: 16.63 秒


## 分類と回帰のランダム フォレスト モデルのスコア付け

このセクションのコードでは、Azure Blob Storage に保存されている分類と回帰のランダム フォレスト モデルを読み込み、標準の分類方法と回帰方法でそのパフォーマンスをスコア付けした後、結果を Blob Storage に保存する方法を示します。

[ランダム フォレスト](http://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests)は、複数のデシジョン ツリーをまとめたものです。オーバーフィットのリスクを軽減するために、多くのデシジョン ツリーが結合されています。ランダム フォレストによって、カテゴリの特徴を処理し、多クラス分類の設定に拡張できますが、特徴のスケーリングは不要であり、非線形性や特徴の相互作用をキャプチャすることができます。ランダム フォレストは、分類と回帰に使用される最も一般的な機械学習モデルの 1 つです。

[spark.mllib](http://spark.apache.org/mllib/) は、連続的な特徴とカテゴリの特徴の両方を使った、二項分類および多クラス分類と回帰のためのランダム フォレストをサポートします。

	# SCORE RANDOM FOREST MODELS FOR CLASSIFICATION AND REGRESSION

	# RECORD START TIME
	timestart = datetime.datetime.now()

	#IMPORT MLLIB LIBRARIES	
	from pyspark.mllib.tree import RandomForest, RandomForestModel
	
	
	# CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
	savedModel = RandomForestModel.load(sc, randomForestClassificationFileLoc)
	predictions = savedModel.predict(indexedTESTbinary)
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfclassificationfilename = "RandomForestClassification_" + datestamp + ".txt";
	dirfilename = scoredResultDir + rfclassificationfilename;
	predictions.saveAsTextFile(dirfilename)
	

	# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
	savedModel = RandomForestModel.load(sc, randomForestRegFileLoc)
	predictions = savedModel.predict(indexedTESTreg)
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfregressionfilename = "RandomForestRegression_" + datestamp + ".txt";
	dirfilename = scoredResultDir + rfregressionfilename;
	predictions.saveAsTextFile(dirfilename)

	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**出力:**

上記のセルの実行に要した時間: 31.07 秒


## 分類と回帰の勾配ブースティング ツリー モデルのスコア付け

このセクションのコードでは、Azure Blob Storage から分類と回帰の勾配ブースティング ツリー モデルを読み込み、標準の分類方法と回帰方法でそのパフォーマンスをスコア付けした後、結果を Blob Storage に保存する方法を示します。

**spark.mllib** は、連続的な特徴とカテゴリの特徴の両方を使った、二項分類と回帰のための GBT をサポートします。

[勾配ブースティング ツリー](http://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) は、複数のデシジョン ツリーをまとめたものです。GBT ではデシジョン ツリーを繰り返しトレーニングすることで損失関数を最小限に抑えます。GBT によって、カテゴリの特徴を処理できますが、特徴のスケーリングは不要であり、非線形性や特徴の相互作用をキャプチャすることができます。また、多クラス分類の設定にも使用できます。


	# SCORE GRADIENT BOOSTING TREE MODELS FOR CLASSIFICATION AND REGRESSION

	# RECORD START TIME
	timestart = datetime.datetime.now()

	#IMPORT MLLIB LIBRARIES
	from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
	
	# CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

	#LOAD AND SCORE THE MODEL
	savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeClassificationFileLoc)
	predictions = savedModel.predict(indexedTESTbinary)
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp + ".txt";
	dirfilename = scoredResultDir + btclassificationfilename;
	predictions.saveAsTextFile(dirfilename)
	

	# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

	# LOAD AND SCORE MODEL 
	savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeRegressionFileLoc)
	predictions = savedModel.predict(indexedTESTreg)
	
	# SAVE RESULTS
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btregressionfilename = "GradientBoostingTreeRegression_" + datestamp + ".txt";
	dirfilename = scoredResultDir + btregressionfilename;
	predictions.saveAsTextFile(dirfilename)


	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
	
**出力:**

上記のセルの実行に要した時間: 14.6 秒


## メモリ内のオブジェクトのクリーンアップとスコア付けファイルの場所の出力

	# UNPERSIST OBJECTS CACHED IN MEMORY
	taxi_df_test_cleaned.unpersist()
	indexedTESTbinary.unpersist();
	oneHotTESTbinary.unpersist();
	indexedTESTreg.unpersist();
	oneHotTESTreg.unpersist();
	oneHotTESTregScaled.unpersist();


	# PRINT OUT PATH TO SCORED OUTPUT FILES
	print "logisticRegFileLoc: " + logisticregressionfilename;
	print "linearRegFileLoc: " + linearregressionfilename;
	print "randomForestClassificationFileLoc: " + rfclassificationfilename;
	print "randomForestRegFileLoc: " + rfregressionfilename;
	print "BoostedTreeClassificationFileLoc: " + btclassificationfilename;
	print "BoostedTreeRegressionFileLoc: " + btregressionfilename;


**出力:**

logisticRegFileLoc: LogisticRegressionWithLBFGS_2016-05-0317_22\_38.953814.txt

linearRegFileLoc: LinearRegressionWithSGD_2016-05-0317_22\_58.878949

randomForestClassificationFileLoc: RandomForestClassification_2016-05-0317_23\_15.939247.txt

randomForestRegFileLoc: RandomForestRegression_2016-05-0317_23\_31.459140.txt

BoostedTreeClassificationFileLoc: GradientBoostingTreeClassification_2016-05-0317_23\_49.648334.txt

BoostedTreeRegressionFileLoc: GradientBoostingTreeRegression_2016-05-0317_23\_56.860740.txt



## Web インターフェイス経由での Spark モデルの利用

Spark には、Livy と呼ばれるコンポーネントを使用して、REST インターフェイス経由でバッチ ジョブや対話型クエリをリモートから送信できるメカニズムが備えられています。HDInsight Spark クラスターでは Livy が既定で有効になっています。Livy の詳細については、[Livy を使用した Spark ジョブのリモート送信](../hdinsight/hdinsight-apache-spark-livy-rest-interface.md)に関するページを参照してください。

Livy を使用して、Azure BLOB に格納されているファイルをバッチ処理でスコア付けし、結果を別の BLOB に書き込むジョブをリモートから送信できます。それには、[GitHub](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/Spark/Python/ConsumeGBNYCReg.py) の Python スクリプトを Spark クラスターの BLOB にアップロードします。**Microsoft Azure Storage Explorer** や **AzCopy** などのツールを使って、スクリプトをクラスターの BLOB にコピーできます。この例では、スクリプトを ***wasb:///example/python/ConsumeGBNYCReg.py*** にアップロードしています。


>[AZURE.NOTE] 必要なアクセス キーは、Spark クラスターに関連付けられているストレージ アカウント用のポータルで見つけることができます。


このスクリプトをこの場所にアップロードすると、 Spark クラスター内の分散コンテキストでスクリプトが実行されます。スクリプトによってモデルが読み込まれ、モデルに基づいて入力ファイルに対して予測が実行されます。

このスクリプトは、Livy で単純な HTTPS/REST 要求を実行することで、リモートで呼び出すことができます。Python スクリプトをリモートから呼び出す HTTP 要求を作成する curl コマンドを次に示します。CLUSTERLOGIN、CLUSTERPASSWORD、CLUSTERNAME は、使用する Spark クラスターに応じて適切な値に置き換えてください。


	# CURL COMMAND TO INVOKE PYTHON SCRIPT WITH HTTP REQUEST

    curl -k --user "CLUSTERLOGIN:CLUSTERPASSWORD" -X POST --data "{"file": "wasb:///example/python/ConsumeGBNYCReg.py"}" -H "Content-Type: application/json" https://CLUSTERNAME.azurehdinsight.net/livy/batches

Livy を利用し、基本認証を使った簡単な HTTPS 呼び出しを実行することで、リモート システムから任意の言語を使って Spark ジョブを呼び出すことができます。


>[AZURE.NOTE] この HTTP 呼び出しを実行するときには Python 要求ライブラリを使うと便利ですが、現在このライブラリは Azure Functions に既定でインストールされていません。そのため、従来の HTTP ライブラリが使用されます。


この HTTP 呼び出しの Python コードを次に示します。

	#MAKE AN HTTPS CALL ON LIVY. 

	import os

	# OLDER HTTP LIBRARIES USED HERE INSTEAD OF THE REQUEST LIBRARY AS THEY ARE AVAILBLE BY DEFAULT
	import httplib, urllib, base64
	
	# REPLACE VALUE WITH ONES FOR YOUR SPARK CLUSTER
	host = '<spark cluster name>.azurehdinsight.net:443'
	username='<username>'
	password='<password>'
	
	#AUTHORIZATION
	conn = httplib.HTTPSConnection(host)
	auth = base64.encodestring('%s:%s' % (username, password)).replace('\n', '')
	headers = {'Content-Type': 'application/json', 'Authorization': 'Basic %s' % auth}
	
	# SPECIFY THE PYTHON SCRIPT TO RUN ON THE SPARK CLUSTER
	# IN THE FILE PARAMETER OF THE JSON POST REQUEST BODY
	r=conn.request("POST", '/livy/batches', '{"file": "wasb:///example/python/ConsumeGBNYCReg.py"}', headers )
	response = conn.getresponse().read()
	print(response)
	conn.close()


この Python コードを [Azure Functions](https://azure.microsoft.com/documentation/services/functions/) に追加することで、タイマー、BLOB の作成、BLOB の更新などのさまざまなイベントに基づいて BLOB をスコア付けする Spark ジョブの送信をトリガーすることもできます。

コードを使用しないクライアント エクスペリエンスを実現するには、[Azure Logic Apps](https://azure.microsoft.com/documentation/services/app-service/logic/) を使用し、**Logic Apps デザイナー**で HTTP 操作を定義してそのパラメーターを設定することで、Spark バッチ スコアリングを呼び出します。

- Azure Portal で、**[+ 新規]**、**[Web + モバイル]**、**[Logic App]** の順に選択して、新しいロジック アプリを作成します。
- ロジック アプリと App Service プランの名前を入力して、**Logic Apps デザイナー**を起動します。
- HTTP 操作を選択して、次の図に示すようにパラメーターを入力します。

![](./media/machine-learning-data-science-spark-model-consumption/spark-logica-app-client.png)


## 次の手順 

**クロス検証とハイパーパラメーター スイープ**: クロス検証とハイパーパラメーター スイープを使用したモデルのトレーニング方法については、「[Spark を使用した高度なデータ探索とモデリング](machine-learning-data-science-spark-advanced-data-exploration-modeling.md)」をご覧ください。

<!---HONumber=AcomDC_0921_2016-->