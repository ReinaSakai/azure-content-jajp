<properties 
	pageTitle="Stream Analytics イベント処理によるリアルタイム イベント処理 | Microsoft Azure" 
	description="リアルタイム イベント処理と分析を可能にする Azure サービスの相互運用性について説明します。" 
    keywords="リアルタイム処理、イベント処理、参照アーキテクチャ"
	services="stream-analytics,event-hubs,storage,sql-database" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="jhubbard" 
	editor=""/>

<tags 
	ms.service="stream-analytics" 
	ms.workload="big-data" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="07/27/2016" 
	ms.author="jeffstok"/>

# 参照アーキテクチャ: Microsoft Azure Stream Analytics によるリアルタイム イベント処理

Azure Stream Analytics を使用したリアルタイム イベント処理のための参照アーキテクチャの目的は、リアルタイムのサービスとしてのプラットフォーム (PaaS) のストリーム処理ソリューションを、 Microsoft Azure でデプロイするための一般的な青写真を提供することです。

## 概要

これまで分析ソリューションは、ETL (抽出、変換、読み込み) やデータ ウェアハウジングなどの機能を基盤としており、分析前のデータが格納されていました。データ受信速度の向上を含む要件の変化によって、この既存モデルが限界に達しました。データを、格納される前のストリームで流れている間に分析する機能は 1 つのソリューションとなります。これは新しい機能でないですが、このようなアプローチはすべての業種で広く採用されることはありませんでした。

Microsoft Azure は、さまざまなソリューションのシナリオと要件をサポートできる分析テクノロジの幅広い選択肢を提供します。エンド ツー エンド ソリューション導入のためにデプロイする Azure サービスを選択するのは、サービスの幅広さを考えると困難な作業です。このドキュメントの目的は、イベント ストリーミング ソリューションをサポートするさまざまな Azure サービスの機能および相互運用性について説明することです。また、このようなアプローチによってユーザーがメリットを得られるシナリオもいくつか説明します。

## 目次

- 概要
- リアルタイム分析の概要
- Azure でのリアルタイム データの価値提案
- リアルタイム分析の一般的なシナリオ
- アーキテクチャとコンポーネント
	- データ ソース
	- データ統合層
	- リアルタイム分析層
	- データ ストレージ 層
	- プレゼンテーション層 / 消費層
- まとめ

**執筆者:** Charles Feddersen、Microsoft Corporation のData Insights Center of Excellence チームに所属するソリューション設計者

**公開:** 2015 年 1 月

**リビジョン:** 1.0

**ダウンロード:** [Microsoft Azure Stream Analytics によるリアルタイム イベント処理](http://download.microsoft.com/download/6/2/3/623924DE-B083-4561-9624-C1AB62B5F82B/real-time-event-processing-with-microsoft-azure-stream-analytics.pdf)


## 問い合わせ
さらにサポートが必要な場合は、[Azure Stream Analytics フォーラム](https://social.msdn.microsoft.com/Forums/ja-JP/home?forum=AzureStreamAnalytics)を参照してください。

## 次のステップ

- [Azure Stream Analytics の概要](stream-analytics-introduction.md)
- [Azure Stream Analytics の使用](stream-analytics-get-started.md)
- [Azure Stream Analytics ジョブのスケーリング](stream-analytics-scale-jobs.md)
- [Stream Analytics Query Language Reference (Stream Analytics クエリ言語リファレンス)](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure Stream Analytics management REST API reference (Azure ストリーム分析の管理 REST API リファレンス)](https://msdn.microsoft.com/library/azure/dn835031.aspx)

 

<!---HONumber=AcomDC_0921_2016-->