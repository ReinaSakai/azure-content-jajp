<properties
	pageTitle="Team Data Science Process の特徴選択 | Microsoft Azure" 
	description="特徴選択の目的について説明し、機械学習のデータ強化プロセスにおけるその役割の例を示します。"
	services="machine-learning"
	documentationCenter=""
	authors="bradsev"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/19/2016"
	ms.author="zhangya;bradsev" />


# Team Data Science Process (TDSP) の特徴選択

この記事では、特徴選択の目的について説明し、機械学習のデータ強化プロセスにおける特徴選択の役割の例を示します。例は、Azure Machine Learning Studio で描画しています。

[AZURE.INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]


このトピックでは、特徴選択の目的について説明し、機械学習のデータ強化プロセスにおけるその役割の例を示します。例は、Azure Machine Learning Studio で描画しています。

特徴のエンジニアリングと選択は TDSP の一部です。TDSP の概要については、[Team Data Science Process の概要](data-science-process-overview.md)に関する記事をご覧ください。特徴エンジニアリングと特徴選択は、TDSP の**特徴の開発**ステップの一部です。

* **特徴エンジニアリング**: このプロセスは、データ内の既存の生の特徴から、関連する特徴を作成し、学習アルゴリズムの予測力を高めようとします。

* **特徴選択**: このプロセスは、トレーニング問題の次元を削減するために、元のデータの特徴のキーのサブセットを選択します。

通常、**特徴エンジニアリング**は追加の特徴を生成するために最初に適用され、その後、無関係な特徴、重複した特徴、関連性の高い特徴を排除するために**特徴選択**の手順が実行されます。


## データから特徴をフィルタリングする - 特徴選択 

特徴選択は、分類や回帰タスクなどの予測モデリング タスク用のトレーニング データセットの構築に一般的に適用されるプロセスです。目標は、最小限の特徴セットを使用してデータ内の最大量の分散を表すことで、元のデータセットからその次元を削減する特徴のサブセットを選択することです。次に、この特徴のサブセットは、モデルのトレーニングに含まれる唯一の特徴になります。特徴選択は、次の 2 つの主な目的を果たします。

* まず、特徴選択は多くの場合、無関係な特徴、重複した特徴、関連性の高い特徴を排除することで、分類の精度を向上させます。
* 次に、特徴の数を減らすことで、モデルのトレーニング プロセスの効率を高めます。これは、サポート ベクター マシンなどのトレーニングに費用がかかる学習者にとって特に重要です。

特徴選択は、モデルのトレーニングに使用されるデータセット内の特徴の数を減らそうと努めていますが、通常「次元削減」とは呼ばれません。特徴選択メソッドは、データ内の元の特徴のサブセットを変更せずに抽出します。次元削減メソッドは、元の機能を変換し、さらに変更できるようにエンジニアリングされた特徴を使用します。次元削減メソッドの例には、主成分分析、正準相関分析、特異値分解が含まれます。

なかでも、教師ありコンテキストの特徴選択メソッドの 1 つとして広く適用されているカテゴリは、”フィルターに基づく特徴選択” と呼ばれます。各特徴と対象の属性間の相関関係を評価することで、これらのメソッドは、統計的尺度を適用して各特徴にスコアを割り当てます。特徴は、スコアによって順位付けされます。これは、しきい値を設定して特定の特徴を保持や削除する場合に役立ちます。これらのメソッドで使用される統計的尺度の例には、ユーザーの相関関係、相互情報、カイ二乗検定が含まれます。

Azure Machine Learning Studio には、特徴選択に提供されるモジュールがあります。次の図に示すように、これらのモジュールには、[フィルターに基づく特徴選択][filter-based-feature-selection]と [Fisher 線形判別分析][fisher-linear-discriminant-analysis]が含まれます。

![特徴選択の例](./media/machine-learning-data-science-select-features/feature-Selection.png)


たとえば、[フィルターに基づく特徴選択][filter-based-feature-selection]モジュールを考えてみます。便宜上、上記で説明したテキスト マイニングの例を引き続き使用します。[特徴ハッシュ][feature-hashing] モジュールから 256 の特徴セットが作成された後、回帰モデルを作成する場合を想定します。応答変数は "Col1" で、書籍レビューの評価は 1～ 5 で表すこととします。[特徴スコア付けの方法] を [Pearson Correlation]、[フィルターに基づく特徴選択] を [Col1]、[目的の特徴の数] を [50] に設定すると、[フィルターに基づく特徴の選択][filter-based-feature-selection] モジュールに、"Col1" という対象の属性を持つ 50 の特徴を含むデータセットが生成されます。次の図に、この実験のフローと説明した入力パラメーターを示します。

![特徴選択の例](./media/machine-learning-data-science-select-features/feature-Selection1.png)

次の図は、結果として得られるデータセットを示します。それぞれの特徴は、それ自体と対象の属性 "Col1" 間のピアソンの相関関係に基づいて評価されます。上位のスコアを持つ特徴は保持されます。

![特徴選択の例](./media/machine-learning-data-science-select-features/feature-Selection2.png)

次の図は、選択した特徴に対応するスコアを示します。

![特徴選択の例](./media/machine-learning-data-science-select-features/feature-Selection3.png)

この[フィルターに基づく特徴選択][filter-based-feature-selection]を適用すると、256 のうち 50 の特徴が選択されます。これらの特性は、”ピアソンの相関関係” スコアリング メソッドに基づいて、対象の属性 "Col1" を持つ最も関連性が高い特徴であるためです。

## まとめ
特徴エンジニアリングと特徴選択は、2 つの一般的にエンジニアリングされ、選択された特徴です。データに含まれるキー情報の抽出を試みるトレーニング プロセスの効率を高めます。また、入力データを正確に分類して、関心のある結果をより確実に予測するために、これらのモデルのパワーを向上させます。特徴エンジニアリングと特徴選択は、学習を計算的により扱いやすくするために組み合わせることもできます。これは、強化した後、モデルの調整やトレーニングに必要な特徴の数を減らすことによって行われます。数学的に言うと、モデルのトレーニングに選択される特徴は、データのパターンを説明し、正常に結果を予測する独立変数の最小セットです。

必ずしも特徴エンジニアリングや特徴選択を実行する必要はありません。必要があるかどうかは、持っているデータや収集するデータ、選択するアルゴリズム、実験の目的によって異なります。

<!-- Module References -->
[feature-hashing]: https://msdn.microsoft.com/library/azure/c9a82660-2d9c-411d-8122-4d9e0b3ce92a/
[filter-based-feature-selection]: https://msdn.microsoft.com/library/azure/918b356b-045c-412b-aa12-94a1d2dad90f/
[fisher-linear-discriminant-analysis]: https://msdn.microsoft.com/library/azure/dcaab0b2-59ca-4bec-bb66-79fd23540080/
 

<!---HONumber=AcomDC_0921_2016-->