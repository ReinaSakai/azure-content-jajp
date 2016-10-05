<properties
	pageTitle="Machine Learning Web サービスをデプロイする | Microsoft Azure"
	description="トレーニング実験を予測実験に変換する方法、デプロイメントの準備をする方法、Azure Machine Learning Web サービスとしてデプロイする方法について説明します。"
	services="machine-learning"
	documentationCenter=""
	authors="garyericson"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/06/2016"
	ms.author="garye"/>

# Azure Machine Learning Web サービスをデプロイする

Azure Machine Learning では、予測分析ソリューションをビルド、テスト、およびデプロイできます。

大まかに次の 3 つの手順で行われます。

- **[トレーニング実験を作成する]** - Azure Machine Learning Studio は、供給するトレーニング データを活用した予測分析モデルの学習とテストに使用できる、コラボレーションと視覚化に対応した開発環境です。
- **[予測実験に変換する]** - 既存のデータでモデルが学習され、それを使用して新しいデータをスコア付けする準備ができると、予測用に実験を用意し、合理化します。
- **Web サービスとしてデプロイする** - 予測実験を[新規]または[従来]の Azure Web サービスとしてデプロイできます。ユーザーはご利用のモデルにデータを送信し、モデルの予測を受信できます。

[AZURE.INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]

## トレーニング実験を作成する

予測分析モデルを学習するには、Azure Machine Learning Studio を使用してトレーニング実験を作成します。ここではトレーニング データを読み込み、必要に応じてデータを用意し、機械学習アルゴリズムに適用し、結果を評価するさまざまなモジュールを含めることができます。実験を繰り返し、別の機械学習アルゴリズムを試して結果を比較したり評価したりできます。

トレーニング実験を作成したり管理したりするプロセスはあらゆる場所でより包括的に使用できます。詳細と例については、次の記事をご覧ください。

- [Azure Machine Learning Studio での簡単な実験の作成](machine-learning-create-experiment.md)
- [Azure Machine Learning を使用した予測ソリューションの開発](machine-learning-walkthrough-develop-predictive-solution.md)
- [Azure Machine Learning Studio への学習データのインポート](machine-learning-data-science-import-data.md)
- [Azure Machine Learning Studio での繰り返し学習の管理](machine-learning-manage-experiment-iterations.md)

## トレーニング実験を予測実験に変換する

モデルをトレーニングすると、新しいデータをスコア付けで使用できるようになります。そのためには、トレーニング実験を予測実験に変換します。

予測実験に変換することで、トレーニングされたモデルをスコア付け Web サービスとしてデプロイできるようになります。Web サービスのユーザーがモデルに入力データを送信し、モデルは予測結果を返します。したがって、予測実験を変換した場合は、モデルが他のユーザーに使用されることを想定しておいてください。

トレーニング実験を予測実験に変換するには、実験キャンバスの下部にある **[実行]** をクリックし、**[Web サービスの設定]** をクリックして、**[予測 Web サービス]** を選択します。

![スコア付け実験に変換する](./media/machine-learning-publish-a-machine-learning-web-service/figure-1.png)

この変換を実行する方法の詳細については、「[Machine Learning トレーニング実験を予測実験に変換する](machine-learning-convert-training-experiment-to-scoring-experiment.md)」をご覧ください。

次の手順では、新規として予測実験をデプロイする方法を示します。

次の手順では、[新しい] Web サービスとして予測実験をデプロイする方法を説明します。実験は、[従来の] Web サービスとしてデプロイすることもできます。

## 新しい Web サービスとして予測実験をデプロイする

予測実験の準備を十分に実行したので、Azure Web サービスとしてデプロイできます。Web サービスを使用してユーザーはデータをモデルに送信でき、モデルは予測を返します。

予測実験をデプロイするには、実験キャンバスの下部にある **[実行]** をクリックします。実験の実行が終了したら **[Web サービスのデプロイ]** をクリックし、**[Deploy Web Service [New](Web サービスのデプロイ [[新規]])]** を選択します。Machine Learning Web サービス ポータルの [デプロイ] ページが開きます。

### Machine Learning Web サービス ポータルの [Deploy Experiment (実験のデプロイ)] ページ
[Deploy Experiment (実験のデプロイ]) ページで、Web サービスの名前を入力します。料金プランを選択します。既存の料金プランがある場合はそのプランを選択できます。ない場合は、サービス用に新しい料金プランを作成する必要があります。

1.	**[Price Plan (料金プラン)]** ドロップダウンで、既存のプランを選択するか、**[Select new plan (新しいプランを選択する)]** オプションを選択します。
2.	**[プラン名]** に、請求書でプランを識別する名前を入力します。
3.	**[Monthly Plan Tiers (月額プラン レベル)]** のいずれか 1 つを選択します。プラン レベルは既定では既定のリージョンのプランになり、Web サービスはそのリージョンにデプロイされることにご注意ください。

**[デプロイ]** をクリックすると、Web サービスの **[クイック スタート]** ページが開きます。

Web サービスの [クイック スタート] ページでは、新しい Web サービスの作成後に実行するほとんどの一般的なタスクにアクセスし、そのガイダンスを得ることができます。ここから、[テスト] ページと [Consume (使用)] ページの両方に簡単にアクセスできます。

<!-- ![Deploy the web service](./media/machine-learning-publish-a-machine-learning-web-service/figure-2.png)-->

### Web サービスのテスト

新しい Web サービスをテストするには、一般的なタスクの下の **[Test web service (Web サービスのテスト)]** をクリックします。[テスト] ページでは、Web サービスを要求応答サービス (RRS) またはバッチ実行サービス (BES) としてテストできます。

RRS テストのページには、入力、出力、および実験用に定義したすべてのグローバル パラメーターが表示されます。Web サービスをテストするには、手動で適切な入力値を入力するか、テスト値を含むコンマ区切り値 (CSV) の書式設定されたファイルを指定できます。

RRS を使用してテストするには、リスト ビュー モードから適切な入力値を入力し、**[Test Request-Response (要求応答のテスト)]** をクリックします。予測結果は、左側の出力列に表示されます。

![Web サービスをデプロイする](./media/machine-learning-publish-a-machine-learning-web-service/figure-5-test-request-response.png)

BES をテストするには、**[バッチ]** をクリックします。バッチ テストのページで、入力の下の [参照] をクリックし、適切なサンプル値を含む CSV ファイルを選択します。CSV ファイルがなく、Machine Learning Studio を使用して予測実験を作成した場合は、予測実験用のデータ セットをダウンロードし、それを使用できます。

データ セットをダウンロードするには、Machine Learning Studio を開きます。予測実験を開き、実験の入力を右クリックします。コンテキスト メニューの **[データセット]** を選択し、**[ダウンロード]** を選択します。

![Web サービスをデプロイする](./media/machine-learning-publish-a-machine-learning-web-service/figure-7-mls-download.png)

**[Test]** をクリックします。バッチ実行ジョブの進行状況は、**[Test Batch Jobs (バッチ ジョブのテスト)]** の右下に表示されます。

![Web サービスをデプロイする](./media/machine-learning-publish-a-machine-learning-web-service/figure-6-test-batch-execution.png)

<!--![Test the web service](./media/machine-learning-publish-a-machine-learning-web-service/figure-3.png)-->

**[構成]** ページでは、Web サービスの説明やタイトルの変更、ストレージ アカウント キーの更新、およびサンプル データの有効化を行うことができます。

![Web サービスを構成する](./media/machine-learning-publish-a-machine-learning-web-service/figure-8-arm-configure.png)

Web サービスをデプロイすると、次のことが可能になります。

- Web サービス API からの**アクセス**
- Azure Machine Learning Web サービス ポータルまたは Azure クラシック ポータルからの**管理**
- モデルが変更された際の**更新**

### Web サービスにアクセスする

Machine Learning Studio から Web サービスをデプロイすると、サービスにデータを送信し、プログラムからの応答が得られます。

**[Consume (使用)]** ページでは、Web サービスへのアクセスに必要なすべての情報が提供されます。たとえば、API キーは、サービスへの承認済みアクセスを許可するために提供されます。

Machine Learning Web サービスへのアクセスの詳細については、[デプロイされた Azure Machine Learning Web サービスの使用方法](machine-learning-consume-web-services.md)に関するページをご覧ください。

### 新しい Web サービスを管理する

従来の Web サービスの Machine Learning Web サービス ポータルを管理することができます。[メイン ポータル ページ](https://services.azureml-test.net/)で **[Web サービス]** をクリックします。Web サービスのページでは、サービスを削除したり、コピーしたりすることができます。特定のサービスを監視するには、サービスをクリックし、**[ダッシュボード]** をクリックします。Web サービスに関連付けられたバッチ ジョブを監視するには、**[Batch Request Log (バッチ要求ログ)]** をクリックします。

## 従来の Web サービスとして予測実験をデプロイする

予測実験の準備を十分に実行したので、Azure Web サービスとしてデプロイできます。Web サービスを使用してユーザーはデータをモデルに送信でき、モデルは予測を返します。

予測実験をデプロイするには、実験キャンバスの下部にある **[実行]** をクリックし、**[Web サービスのデプロイ]** をクリックします。Web サービスがセットアップされ、Web サービスのダッシュ ボードに配置されます。

![Web サービスをデプロイする](./media/machine-learning-publish-a-machine-learning-web-service/figure-2.png)

Web サービスをテストするには、Web サービス ダッシュボードで **[テスト]** リンクをクリックします。ダイアログ ボックスが表示され、サービスへのデータの入力が促されます。これらはスコア付け実験で想定される列になります。データのセットを入力し、**[OK]** をクリックします。Web サービスによって生成された結果がダッシュボードの下部に表示されます。

![Web サービスをテストする](./media/machine-learning-publish-a-machine-learning-web-service/figure-3.png)

**[構成]** タブで、サービスの表示名を変更したり、説明を付けたりできます。この名前と説明は、Web サービスを管理する [Azure クラシック ポータル](http://manage.windowsazure.com/)に表示されます。

**INPUT SCHEMA**、**OUTPUT SCHEMA**、**WEB SERVICE PARAMETER** の各列に文字列を入力し、入力データ、出力データ、Web サービス パラメーターの説明を追加できます。これらの説明は、Web サービスのサンプル コードのドキュメントで使用されます。また、ログ記録を有効にすれば、Web サービスのアクセスで発生するすべてのエラーを診断できます。

詳細については、「[Enable logging for Machine Learning web services (Machine Learning Web サービスのログ記録を有効にする)](machine-learning-web-services-logging.md)」をご覧ください。

![Web サービスを構成する](./media/machine-learning-publish-a-machine-learning-web-service/figure-4.png)

### Web サービスにアクセスする

Machine Learning Studio から Web サービスをデプロイすると、サービスにデータを送信し、プログラムからの応答が得られます。

ダッシュ ボードでは、Web サービスへのアクセスが必要なすべての情報が提供されます。たとえば、サービスへの承認済みアクセスを許可する API キーや、コードの記述に役立つ API のヘルプ ページが提供されます。

Machine Learning Web サービスへのアクセスの詳細については、[デプロイされた Azure Machine Learning Web サービスの使用方法](machine-learning-consume-web-services.md)に関するページをご覧ください。

### Azure クラシック ポータルで Web サービスを管理する

[Azure クラシック ポータル](http://manage.windowsazure.com/)では、**[Machine Learning]** サービスをクリックして Machine Learning ワーク スペースを開き、**[Web サービス]** タブから Web サービスを開くことで Web サービスを管理できます。このページから Web サービスの監視、更新、削除ができます。Web サービスのデプロイ時に作成される既定のエンドポイントに加え、Web サービスの 2 つ目のエンドポイントを追加することもできます。

詳細については、「[Azure Machine Learning ワークスペースの管理](machine-learning-manage-workspace.md)」をご覧ください。
<!-- When this article gets published, fix the link and uncomment
For more information on how to manage Azure Machine Learning web service endpoints using the REST API, see **Azure machine learning web service endpoints**.
-->

## Web サービスを更新する

別のトレーニング データでのモデルの更新など、Web サービスに変更を加えたり、元の Web サービスを上書きしてもう一度デプロイしたりできます。

Web サービスを更新するには、Web サービスのデプロイに使用した元の予測実験を開き、**[名前を付けて保存]** をクリックして編集可能なコピーを作成します。必要な変更を加えて **[WEB サービスのデプロイ]** をクリックします。

この実験は前にデプロイ済みであるため、既存のサービスを上書きするか (従来の Web サービス)、更新するか (新しい Web サービス) 確認されます。**[はい]** または **[更新]** をクリックすると、既存の Web サービスが停止され、新しい予測実験が代わりにデプロイされます。

> [AZURE.NOTE] たとえば元の Web サービスの構成に変更を加えた場合や新しい表示名または説明を入力した場合、それらの値をもう一度入力する必要が生じます。

Web サービスを更新するオプションの 1 つに、モデルをプログラムによって再トレーニングする方法があります。詳細については、「[プログラムによる Machine Learning のモデルの再トレーニング](machine-learning-retrain-models-programmatically.md)」をご覧ください。


<!-- internal links -->
[トレーニング実験を作成する]: #create-a-training-experiment
[予測実験に変換する]: #convert-the-training-experiment-to-a-predictive-experiment
[新規]: #deploy-the-predictive-experiment-as-a-new-web-service
[新しい]: #deploy-the-predictive-experiment-as-a-new-web-service
[新規]: #deploy-the-predictive-experiment-as-a-new-web-service
[従来]: #deploy-the-predictive-experiment-as-a-new-web-service
[従来の]: #deploy-the-predictive-experiment-as-a-new-web-service
[Access]: #access-the-web-service
[Manage]: #manage-the-web-service-in-the-azure-management-portal
[Update]: #update-the-web-service

<!---HONumber=AcomDC_0914_2016-->