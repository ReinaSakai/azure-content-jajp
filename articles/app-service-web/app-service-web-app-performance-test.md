<properties
   pageTitle="Azure Web アプリのパフォーマンスをテストする | Microsoft Azure"
   description="Azure Web アプリのパフォーマンスを実行して、アプリでユーザーの負荷がどのように処理されるかを確認します。応答時間を測定し、問題を示す可能性があるエラーを見つけます。"
   services="app-service\web"
   documentationCenter=""
   authors="ecfan"
   manager="douge"
   editor="jimbe"/>

<tags
   ms.service="app-service-web"
   ms.workload="web"
   ms.tgt_pltfrm="na"
   ms.devlang="na"
   ms.topic="article"
   ms.date="01/13/2016"
   ms.author="estfan; manasma"/>

# 負荷をかけた状態での Azure Web アプリのパフォーマンス テスト

Web アプリを起動する前または更新プログラムを運用環境にデプロイする前に、パフォーマンスを確認しましょう。これにより、アプリをリリースする準備が整っているかどうかをより適切に評価できます。アプリの使用がピーク状態にあるときや次のマーケティング プッシュ時にアプリがトラフィックを処理できることを確認しましょう。

パブリック プレビューの段階では、Azure ポータルでアプリのパフォーマンスを無料でテストできます。これらのテストでは、一定期間にわたってアプリのユーザー負荷をシミュレートし、アプリの応答を測定します。たとえば、一定の数のユーザーに対してアプリがどれだけすばやく応答したかがテスト結果に示されます。また、アプリに存在する問題を示す可能性がある失敗した要求の数も示されます。

![Web アプリのパフォーマンスの問題を見つける][TestOverview]

## 開始する前に

*	[Azure サブスクリプション][AzureSubscription]が必要です (まだ取得していない場合)。[無料で Azure アカウントを開く方法については、このページを参照してください][AzureFreeTrial]。

*	パフォーマンス テストの履歴を保存するには、[Visual Studio Team Services (Team Services)][WhatIsTeamServices] アカウントが必要です。パフォーマンス テストを設定するときに新しいアカウントを作成するか、アカウントの所有者である場合は既存のアカウントを使用します。[Visual Studio Team Services アカウントでほかにできること](#TeamServicesAccount)

*	非運用環境にテストするアプリをデプロイします。運用環境で使用するプラン以外の App Service プランをアプリで使用するように設定します。これで、既存の顧客に影響が及んだり、運用環境のアプリの速度が低下したりすることがなくなります。

## パフォーマンス テストの設定と実行

0.	[Azure ポータル][AzurePortal]にサインインします。所有している Visual Studio Team Services アカウントを使用するには、アカウント所有者としてサインインします。

0.	Web アプリに移動します。

	![[すべてを参照] > [Web アプリ] > アプリを選択][WebApp]

0.	**[パフォーマンス テスト]** に移動します。

	![[ツール] > [パフォーマンス テスト] を選択][ExpandedTools]
 
0.	ここで、パフォーマンス テストの履歴を保存するための、[Visual Studio Team Services (Team Services)][WhatIsTeamServices] アカウントにリンクします。

	使用できる Team Services アカウントがある場合は、そのアカウントを選択します。VSO アカウントを持っていない場合は、新しいアカウントを作成します。

	![既存の Team Services アカウントを選択するか、新しいアカウントを作成します][ExistingNewTeamServicesAccount]

0.	パフォーマンス テストを作成します。詳細を設定し、テストを実行します。テストの実行中、リアルタイムで結果を確認できます。

	たとえば、昨年の休日セールでクーポンを配布したアプリがあるとします。このイベントは、100 人の同時接続顧客数をピーク負荷として 15 分間にわたって続きました。今年は、顧客の数を 2 倍にしたいと考えています。さらに、ページの読み込み時間を 5 秒から 2 秒に短縮することにより、顧客満足度を高めたいと考えています。そこで、更新したアプリのパフォーマンスを 250 人のユーザーを想定して 15 分間にわたってテストします。

	Web サイトを同時に訪れる仮想ユーザー (顧客) を生成して、アプリに対する負荷をシミュレートします。これで、失敗した要求や応答が遅い要求の数がわかります。

	![パフォーマンス テストの作成、設定、実行][NewTest]

	 *	Web アプリの既定の URL は自動的に追加されます。URL を変更して他のページ (HTTP GET 要求のみ) をテストすることができます。

	 *	ローカル状態をシミュレートし、待機時間を短縮するには、最も近い場所のユーザーを選択して負荷を生成します。

	テストが実行されているときのようすを次に示します。最初の 1 分間は、ページの読み込みに予定よりも時間がかかっています。

	![実行中のパフォーマンス テストとリアルタイムのデータ][TestRunning]

	テストが終了すると、最初の 1 分が経過した後はページの読み込みが高速化されていることがわかります。これは、問題のトラブルシューティングを始める場所を特定するのに役立ちます。

	![パフォーマンス テスト完了、失敗したリクエストなどの結果がわかる][TestDone]
	
ご意見をお待ちしております。質問やその他の問題については、<vsoloadtest@microsoft.com> までお問い合わせください。

##	Q & A

#### Q: テストの実行時間に関して制限はありますか? 

A: はい。Azure ポータルでは、最長 1 時間までテストを実行できます。

#### Q: パフォーマンス テストの実行に使用できる時間はどれだけユーザーに与えられますか。 

A: パブリック プレビュー以降、Visual Studio Team Services アカウントに対して毎月 20,000 仮想ユーザー時間 (VUM) が無料で提供されます。VUM は、仮想ユーザー数にテスト時間 (分) を掛けた値です。この無料分を超える時間を使用する必要がある場合は、時間を購入し、使用した時間分のみ支払うことができます。

#### Q: これまで使用した VUM はどこで確認できますか。

A: Azure ポータルでこの時間を確認できます。

![Team Services アカウントに移動します。][TeamServicesAccount]

![使用した VUM の確認][CheckTestTime]

<a name="VSOAccount"></a>
#### Q: Visual Studio Team Services アカウントでほかにできることは何ですか。

A: 新しいアカウントを見つけるには、```https://{accountname}.visualstudio.com``` にアクセスしてください。任意のツールまたは言語を使用して、コードの共有、ビルド、テスト、作業の追跡、ソフトウェアの出荷などの操作をすべてクラウドで実行できます。[Visual Studio Team Services][WhatIsTeamServices] の機能とサービスがどのようにチームの共同作業と継続的なデプロイを支援するかについて学習してください。

<!--Image references-->
[WebApp]: ./media/app-service-web-app-performance-test/azure-np-web-apps.png
[TestOverview]: ./media/app-service-web-app-performance-test/azure-np-perf-test-overview.png
[ExpandedTools]: ./media/app-service-web-app-performance-test/azure-np-web-app-details-tools-expanded.png
[ExistingNewTeamServicesAccount]: ./media/app-service-web-app-performance-test/azure-np-no-vso-account.png
[NewTest]: ./media/app-service-web-app-performance-test/azure-np-new-performance-test.png
[TestRunning]: ./media/app-service-web-app-performance-test/azure-np-running-perf-test.png
[TestDone]: ./media/app-service-web-app-performance-test/azure-np-perf-test-done.png
[TeamServicesAccount]: ./media/app-service-web-app-performance-test/azure-np-vso-accounts.png
[CheckTestTime]: ./media/app-service-web-app-performance-test/azure-np-vso-accounts-vum-summary.png

<!--Reference links -->
[AzurePortal]: https://portal.azure.com
[AzureSubscription]: https://account.windowsazure.com/subscriptions
[AzureFreeTrial]: https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F
[WhatIsTeamServices]: https://www.visualstudio.com/products/what-is-visual-studio-online-vs

<!---HONumber=AcomDC_0128_2016-->