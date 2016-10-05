<properties 
	pageTitle="SQL Azure からデータを処理する | Microsoft Azure" 
	description="SQL Azure からのデータを処理する" 
	services="machine-learning" 
	documentationCenter="" 
	authors="garyericson" 
	manager="jhubbard" 
	editor="" />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/16/2016" 
	ms.author="fashah;garye;bradsev" />

#<a name="heading">Azure の SQL Server Virtual Machine でデータを処理する</a>

このドキュメントでは、Azure の SQL Server VM に保存されたデータを探索し、データの特徴を生成する方法について説明します。これは、SQL を使用してデータを処理するか、Python などのプログラミング言語を使用して実行できます。


> [AZURE.NOTE] このドキュメントのサンプルの SQL ステートメントは、データが SQL Server に存在することを前提としています。存在しない場合は、クラウド データ サイエンス プロセス マップを参照して、SQL Server へデータを移動する方法を確認してください。

##<a name="SQL"></a>SQL の使用

このセクションでは、SQL を使用した次のデータ処理タスクについて説明します。

1. [データの探索](#sql-dataexploration)
2. [特徴の生成](#sql-featuregen)

###<a name="sql-dataexploration"></a>データの探索
SQL Server のデータ ストアの探索に使用できるいくつかのサンプル SQL スクリプトを次に示します。


> [AZURE.NOTE] 実用的な例として、[NYC タクシー データセット](http://www.andresmh.com/nyctaxitrips/)を使用し、エンドツーエンドのチュートリアルの「[IPython Notebook と SQL Server を使用した NYC データの処理](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-sql-walkthrough.ipynb)」というタイトルの IPNB を参照することができます。

1. 1 日ごとの所見の数を取得する

	`SELECT CONVERT(date, <date_columnname>) as date, count(*) as c from <tablename> group by CONVERT(date, <date_columnname>)`

2. カテゴリ列内のレベルを取得する

	`select  distinct <column_name> from <databasename>`

3. 2 つのカテゴリ列の組み合わせ内のレベルの数を取得する

	`select <column_a>, <column_b>,count(*) from <tablename> group by <column_a>, <column_b>`

4. 数値型列の分布を取得する

	`select <column_name>, count(*) from <tablename> group by <column_name>`


###<a name="sql-featuregen"></a>特徴の生成

このセクションでは SQL を使用して特徴を生成する方法について説明します。

1. [カウント ベースの特徴の生成](#sql-countfeature)
2. [ビン分割特徴の生成](#sql-binningfeature)
3. [1 つの列からの特徴の展開](#sql-featurerollout)


> [AZURE.NOTE] 追加の特徴を生成すると、既存のテーブルに列として追加するか、追加の特徴と主キーを持つ新しいテーブルを作成して元のテーブルと結合することができます。

###<a name="sql-countfeature"></a>カウント ベースの特徴の生成

このドキュメントでは、カウント特徴を生成する 2 つの方法を示します。最初の方法は、条件付きの合計を使用します。2 番目の方法は、Where 句を使用します。これらを (主キーの列を使用することで) 元のテーブルと結合して、カウント特徴と元のデータを一緒にすることができます。

	select <column_name1>,<column_name2>,<column_name3>, COUNT(*) as Count_Features from <tablename> group by <column_name1>,<column_name2>,<column_name3> 

	select <column_name1>,<column_name2> , sum(1) as Count_Features from <tablename> 
	where <column_name3> = '<some_value>' group by <column_name1>,<column_name2> 

###<a name="sql-binningfeature"></a>ビン分割特徴の生成

次の例は、数値型の列をビン分割 (5 つの箱を使用) して、特徴として代わりに使用できる、ビン分割特徴を生成する方法を示しています。

	`SELECT <column_name>, NTILE(5) OVER (ORDER BY <column_name>) AS BinNumber from <tablename>`


###<a name="sql-featurerollout"></a>1 つの列からの特徴の展開

このセクションでは、テーブル内の 1 つの列を展開して追加の特徴を生成する方法を示します。この例は、特徴を生成しようとするテーブルに、緯度や経度の列があることを前提としています。

緯度と経度の位置データ (リソースは stackoverflow の「[How to measure the accuracy of latitude and longitude? (緯度と経度の精度を測定する方法)](http://gis.stackexchange.com/questions/8650/how-to-measure-the-accuracy-of-latitude-and-longitude)」) の簡単な概要を次に示します。これは、Featurize する前に [場所] フィールドを理解するうえで役立ちます。

- 記号は、私たちが地球の北半球か南半球、東半球か西半球、それぞれどちらにいるかを示します。
- 100 の位が 0 でなければ、それは経度を使用していることを示します。緯度ではありません。
- 10 の位は、最大で約 1,000 キロメートル単位で位置を示します。これは、私たちがどの大陸または海洋にいるかに関する役立つ情報になります。
- 1 の位 (1 つの 10 進数の角度) は、最大 111 キロメートル (60 海里、約 69 マイル) 単位で位置を示します。これは、私たちが大まかにどの大きい州または国にいるかを示します。
- 小数第 1 位は最大 11.1 km: になります。この位で、大都市と隣接する大都市の位置を識別できます。
- 小数第 2 位は、最大 1.1 km に値します。ある村と隣接する村を識別できます。
- 小数第 3 位は、最大 110 m に値します。大規模な農地または施設の構内を識別できます。
- 小数第 4 位は、最大 11 m に値します。土地の一区画を識別できます。これは、未修正の GPS ユニットの一般的な支障のない精度と比較できます。
- 小数第 5 位は、1.1 m に値します。木々を互いに識別することができます。商用の GPS ユニットにおいて、このレベルの精度は微分補正によってのみ実現できます。
- 小数第 6 位は、最大 0.11 m に値します。これは、造園設計、道路建設において構造を詳細に配置するために使用できます。氷河と川の動きを追跡するには十分すぎるはずです。これは、GPS の微分補正など、GPS を使用した精密な測定で実現できます。

位置情報の Featurize は、地域、位置、および都市の情報に分けて、次のように実行します。地域または地区の情報は、「[Find a Location by Point (ポイントで位置情報を特定する)](https://msdn.microsoft.com/library/ff701710.aspx)」で紹介されている Bing Maps API などの REST エンド ポイントを呼び出すことで取得することもできます。

	select 
		<location_columnname>
		,round(<location_columnname>,0) as l1		
		,l2=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 1 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),1,1) else '0' end 	
		,l3=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 2 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),2,1) else '0' end 	
		,l4=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 3 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),3,1) else '0' end 	
		,l5=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 4 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),4,1) else '0' end 	
		,l6=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 5 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),5,1) else '0' end 	
		,l7=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 6 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),6,1) else '0' end 	
	from <tablename>

上記の位置ベースの特徴をさらに使用すると、前述した追加のカウント特徴を生成できます。


> [AZURE.TIP] お好みのプログラム言語でレコードを挿入できます。書き込み効率を向上させるためにデータをチャンクで挿入する必要があります。「[A HelloWorld sample to access SQLServer with python (SQL Server に python でアクセスするための HelloWorld サンプル)](https://code.google.com/p/pypyodbc/wiki/A_HelloWorld_sample_to_access_mssql_with_python)」で pyodbc を使用した実行方法の例を確認してください。もう 1 つの選択肢は、[BCP ユーティリティ](https://msdn.microsoft.com/library/ms162802.aspx)を使用してデータベースにデータを挿入することです。

###<a name="sql-aml"></a>Azure Machine Learning への接続

新しく生成された特徴は、既存のテーブルに列として追加するか、新しいテーブルに格納して機械学習の元のテーブルと結合することができます。特徴は生成できるほか、作成済みであれば、次に示すように、Azure Machine Learning の[データのインポート][import-data] モジュールを使用してアクセスすることができます。

![Azure ML リーダー][1]

##<a name="python"></a>Python などのプログラミング言語の使用

データが SQL Server に格納されている場合に、Python を使用してデータを探索し、特徴を生成する手順は、「[Azure BLOB データを高度な分析を使用して処理する](machine-learning-data-science-process-data-blob.md)」で説明されているように、Python を使用して Azure BLOB のデータを処理する手順と似ています。データは、データベースから pandas データ フレームに読み込む必要があります。その後、さらに処理することができます。このセクションでは、データベースに接続して、データ フレームにデータを読み込むプロセスについて記載します。

次の接続文字列形式を使用して pyodbc を使用し Python から SQL Server データベースに接続することができます (サーバー名、データベース名、ユーザー名およびパスワードは使用する特定の値に置き換えてください)。

	#Set up the SQL Azure connection
	import pyodbc	
	conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')

Python の [Pandas ライブラリ](http://pandas.pydata.org/)には、Python プログラミングでデータを操作するためのデータ構造とデータ解析ツールの豊富なセットが用意されています。次のコードは、SQL Server データベースから返される結果を Pandas データ フレームに読み取ります。

	# Query database and load the returned results in pandas data frame
	data_frame = pd.read_sql('''select <columnname1>, <cloumnname2>... from <tablename>''', conn)

これで、「[データ サイエンス環境で Azure BLOB データを処理する](machine-learning-data-science-process-data-blob.md)」の記事で扱うとおりに Pandas データ フレームを操作できるようになりました。

## 実行中の Azure データ サイエンスの例

公開されているデータセットを使用した Azure のデータ サイエンス プロセスのエンドツーエンドのチュートリアルの例については、「[実行中の Azure データ サイエンス プロセス](machine-learning-data-science-process-sql-walkthrough.md)」をご覧ください。

[1]: ./media/machine-learning-data-science-process-sql-server-virtual-machine/reader_db_featurizedinput.png


<!-- Module References -->
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
 

<!---HONumber=AcomDC_0921_2016-->