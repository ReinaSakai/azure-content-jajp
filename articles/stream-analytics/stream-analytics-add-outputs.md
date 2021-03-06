<properties 
	pageTitle="Stream Analytics ジョブのデータ出力を構成する方法 | Microsoft Azure" 
	description="Stream Analytics ジョブの出力の構成 | ラーニング パス セグメント。"
	keywords="データ出力、データ移動"
	documentationCenter=""
	services="stream-analytics"
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.tgt_pltfrm="na" 
	ms.workload="data-services" 
	ms.date="03/18/2016" 
	ms.author="jeffstok"/>

# Stream Analytics ジョブのデータ出力を構成する方法

Azure Stream Analytics ジョブは 1 つ以上のデータ出力に接続できます。これにより、既存のデータ シンクへの接続が定義されます。Stream Analytics ジョブが受信データを処理して変換するとき、データ出力イベントのストリームがジョブの出力に書き込まれます。

Stream Analytics のデータ出力は、ダッシュボードまたはアラートのリアルタイム ソースとして使用したり、データ移動ワークフローをトリガーしたり、単純に後でバッチ処理するためのデータをアーカイブしたりするために使用できます。Stream Analytics は、さまざまな Azure サービスの最上位の統合です。詳細はここに説明があります。

Stream Analytics ジョブに出力を追加するには:

1. Azure クラシック ポータルの Stream Analytics ジョブで、**[出力]** をクリックし、**[出力の追加]** をクリックします。

    ![出力の追加](./media/stream-analytics-add-outputs/1-stream-analytics-add-outputs.png)

    Azure ポータルの Stream Analytics ジョブで、**[出力]** タイルをクリックします。

    ![Azure ポータルでの出力の追加](./media/stream-analytics-add-outputs/5-stream-analytics-add-outputs.png)

2. 出力の種類を指定します。

    ![データ移動の種類の選択](./media/stream-analytics-add-outputs/2-stream-analytics-add-outputs.png)

    ![Azure ポータルでのデータ移動の種類の選択](./media/stream-analytics-add-outputs/6-stream-analytics-add-outputs.png)

3. **[出力のエイリアス]** ボックスに、この出力のわかりやすい名前を入力します。この名前は、後で出力を参照するために、ジョブのクエリの中で使用できます。
    
    出力に接続するために必要な他の接続プロパティを入力します。これらのフィールドは、出力の種類によって異なります。詳細はここに定義されています。

    ![データ出力のプロパティの追加](./media/stream-analytics-add-outputs/3-stream-analytics-add-outputs.png)

4. 出力の種類によっては、データをシリアル化または書式設定する方法を指定する必要があります。それぞれの出力の種類に固有のシリアル化の設定は、ここに説明があります。

    データ ソースに接続するために必要な他の接続プロパティを入力します。これらのフィールドは入力の種類とソースの種類によって異なります。詳細は[ここ](stream-analytics-create-a-job.md)に定義されています。

    ![イベント ハブへのデータ出力の追加](./media/stream-analytics-add-outputs/4-stream-analytics-add-outputs.png)

    ![イベント ハブへの Azure ポータルのデータ出力](./media/stream-analytics-add-outputs/7-stream-analytics-add-outputs.png)

> [Azure.Note] ジョブに追加される出力要素は、ジョブが開始されてイベントがフローを開始する前に存在している必要があります。たとえば、出力として BLOB Storage を使用する場合、ジョブはストレージ アカウントを自動的に作成しません。ASA ジョブを開始する前に、ユーザーが作成する必要があります。

## 問い合わせ
さらにサポートが必要な場合は、[Azure Stream Analytics フォーラム](https://social.msdn.microsoft.com/Forums/ja-JP/home?forum=AzureStreamAnalytics)を参照してください。

## 次のステップ

- [Azure Stream Analytics の概要](stream-analytics-introduction.md)
- [Azure Stream Analytics の使用](stream-analytics-get-started.md)
- [Azure Stream Analytics ジョブのスケーリング](stream-analytics-scale-jobs.md)
- [Stream Analytics Query Language Reference (Stream Analytics クエリ言語リファレンス)](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure Stream Analytics management REST API reference (Azure ストリーム分析の管理 REST API リファレンス)](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!---HONumber=AcomDC_0323_2016-->