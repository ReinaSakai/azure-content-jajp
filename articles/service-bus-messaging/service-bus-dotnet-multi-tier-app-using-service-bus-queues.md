<properties
	pageTitle=".NET 多層アプリケーション | Microsoft Azure"
	description="Service Bus キューを使用して層間で通信する多層アプリケーションを Azure で開発するのに役立つ .NET チュートリアルです。"
	services="service-bus-messaging"
	documentationCenter=".net"
	authors="sethmanheim"
	manager="timlt"
	editor=""/>

<tags
	ms.service="service-bus-messaging"
	ms.workload="tbd"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="get-started-article"
	ms.date="09/01/2016"
	ms.author="sethm"/>

# Azure Service Bus キューを使用する .NET 多層アプリケーション

## はじめに

Microsoft Azure 向けアプリケーションは、Visual Studio および無料の Azure SDK for .NET を使用して簡単に開発できます。このチュートリアルでは、ローカル環境で実行されている複数の Azure リソースを使用するアプリケーションを作成する手順について説明します。Azure を初めて使用するユーザーを対象としています。

学習内容は次のとおりです。

-   1 回のダウンロードとインストールで、Azure 開発用に コンピューターを準備する方法
-   Azure 開発のための Visual Studio の使用方法
-   Web ロールおよび Worker ロールを使用して Azure に 多層アプリケーションを作成する方法
-   Service Bus キューを使用して各層間で通信する方法

[AZURE.INCLUDE [アカウント作成メモ](../../includes/create-account-note.md)]

このチュートリアルでは、Azure のクラウド サービスで多層アプリケーションを構築して実行します。フロントエンドは ASP.NET MVC Web ロール、バックエンドは Service Bus キューを使用する worker ロールになります。同じ多層アプリケーションを作成するときに、クラウド サービスではなく Azure Web サイトにデプロイされる Web プロジェクトとしてフロントエンドを実装することもできます。Azure Web サイトのフロントエンドの場合の手順については、「[次のステップ](#nextsteps)」を参照してください。また、[.NET オンプレミス/クラウド ハイブリッド アプリケーション](../service-bus-relay/service-bus-dotnet-hybrid-app-using-service-bus-relay.md)のチュートリアルを試すこともできます。

次のスクリーン ショットに、完成したアプリケーションを示します。

![][0]

## シナリオの概要: ロール間通信

処理を要求するため、Web ロールで実行されているフロントエンド UI コンポーネントは、worker ロールで実行されている中間層ロジックと対話する必要があります。この例では、各層間での通信に Service Bus のブローカー メッセージングが使用されています。

Web 層と中間層との間でブローカー メッセージングを使用すると、2 つのコンポーネントの結合が解除されます。直接メッセージング (TCP や HTTP) とは異なり、Web 層は中間層に直接接続しません。その代わりに、作業をメッセージとして Service Bus にプッシュします。Service Bus は、このメッセージを中間層が受け取り、処理する用意ができるまで確実に保持します。

Service Bus には、ブローカー メッセージングをサポートする、キューとトピックという 2 つのエンティティがあります。キューでは、各メッセージは単一のレシーバーが使用するキューに送信されます。トピックは発行/サブスクライブ パターンをサポートし、発行された各メッセージをトピックに登録されているサブスクリプションで使用します。各サブスクリプションは、独自のメッセージ キューを論理的に管理しています。また、サブスクリプションにフィルター ルールを構成し、サブスクリプション キューに渡すメッセージをフィルターに一致するメッセージのみに制限できます。次の例では、Service Bus キューを使用します。

![][1]

この通信メカニズムには、直接メッセージングと比較した場合に、次のような利点があります。

-   **一時的な結合の解除:** 非同期メッセージング パターンでは、プロデューサーと コンシューマーが同時にオンラインになっている必要はありません。コンシューマーが メッセージを受け取る用意ができるまで、Service Bus が確実に メッセージを保持します。このため、保守などのために、分散型アプリケーションの各コンポーネントの接続を自主的に解除できます。また、コンポーネントのクラッシュによって接続が解除されても問題なく、接続解除の際にシステム全体に影響が及ぶことがありません。さらに、コンシューマー側アプリケーションがオンラインになっている時間は、1 日のうち一定の時間だけで済みます。

-   **負荷平準化:** 多くのアプリケーションでは、システム負荷が時間の経過とともに変化しますが、各作業単位に必要な処理時間は通常一定に保たれます。メッセージ プロデューサーとメッセージ コンシューマーの間をキューで仲介することは、コンシューマー側アプリケーション (worker) はピーク時ではなく平均時の負荷に対応できるようにプロビジョニングすればいいということを意味します。キューの深さは、受信の負荷の変化に応じて増減します。このため、アプリケーション負荷への対応に必要なインフラストラクチャの観点から直接費用を節約できます。

-   **負荷分散:** 負荷の増大に合わせて、キューからの読み取りのために worker プロセスを追加できます。各メッセージは、ワーカー プロセスの中の 1 つのプロセスによって処理されます。また、このプルベースの負荷分散では、各 worker マシンがそれぞれ独自の最大レートでメッセージをプルするため、worker マシンの処理能力が異なる場合であっても最適に使用できます。このパターンは、*競合コンシューマー* パターンと 呼ばれることもあります。

    ![][2]

以降のセクションでは、このアーキテクチャを実装するコードについて説明します。

## 開発環境を設定する

Azure アプリケーションの開発を開始する前に、ツールを入手して、開発環境を設定します。

1.  「[Gツールと SDK の入手][]」にある Azure SDK for .NET をインストールします。

2. 	使用している Visual Studio のバージョンに対応する **[SDK のインストール]** をクリックします。このチュートリアルの手順では、Visual Studio 2015 を使用します。

4.  インストーラーの実行や保存を求めるメッセージが表示されたら、**[実行]** をクリックします。

5.  **Web Platform Installer** の **[インストール]** をクリックし、インストールの手順を進めます。

6.  インストールが完了すると、アプリケーションの開発に必要なツールがすべて揃います。SDK には、Visual Studio で Azure アプリケーションを簡単に開発するためのツールが用意されています。Visual Studio がインストールされていない場合、無料の Visual Studio Express もインストールされます。

## 名前空間の作成

次の手順では、サービス名前空間を作成し、Shared Access Signature (SAS) キーを取得します。名前空間は、Service Bus によって公開される各アプリケーションのアプリケーション境界を提供します。名前空間が作成された時点で、システムによって SAS キーが自動的に生成されます。名前空間と SAS キーの組み合わせが、アプリケーションへのアクセスを Service Bus が認証する資格情報になります。

[AZURE.INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## Web ロールを作成する

このセクションでは、アプリケーションのフロントエンドを作成します。最初に、アプリケーションで表示するページを作成します。その後、Service Bus キューに項目を送信し、キューに関するステータス情報を表示するコードを追加します。

### プロジェクトを作成する

1.  管理者特権を使用して、Microsoft Visual Studio を起動します。管理者特権で Visual Studio を起動するには、**Visual Studio** のプログラム アイコンを右クリックし、**[管理者として実行]** をクリックします。Azure コンピューティング エミュレーター (後ほどこの記事で解説) を使用するには、管理者特権で Visual Studio を開始する必要があります。

    Visual Studio で、**[ファイル]** メニューの **[新規作成]** をクリックした後、**[プロジェクト]** をクリックします。

2.  **[インストールされたテンプレート]** の **[Visual C#]** で **[クラウド]** をクリックし、**[Azure クラウド サービス]** をクリックします。プロジェクトの名前を「**MultiTierApp**」にします。次に、 **[OK]** をクリックします

    ![][9]

3.  **.NET Framework 4.5** ロールの **[ASP.NET Web ロール]** をダブルクリックします。

    ![][10]

4.  **[Azure のクラウド サービス ソリューション]** の **[WebRole1]** をポイントし、鉛筆のアイコンをクリックして、Web ロールの名前を「**FrontendWebRole**」に変更します。次に、 **[OK]** をクリックします(「FrontEnd」ではなく「Frontend」と入力してください。小文字の "e" です)。

    ![][11]

5.  **[新しい ASP.NET プロジェクト]** ダイアログ ボックスで、**[テンプレートの選択]** ボックスの一覧の **[MVC]** をクリックします。

    ![][12]

6. 引き続き **[新しい ASP.NET プロジェクト]** ダイアログ ボックスで、**[認証の変更]** をクリックします。**[認証の変更]** ダイアログ ボックスで、**[認証なし]** をクリックし、**[OK]** をクリックします。このチュートリアルでは、ユーザー ログインの必要がないアプリケーションをデプロイします。

	![][16]

7. **[新しい ASP.NET プロジェクト]** ダイアログ ボックスに戻り、**[OK]** をクリックして、プロジェクトを作成します。

6.  **ソリューション エクスプローラー**で **FrontendWebRole** プロジェクトの **[参照]** を右クリックし、**[NuGet パッケージの管理]** をクリックします。

7.  **[参照]** タブをクリックして、`Microsoft Azure Service Bus` を検索します。**[インストール]** をクリックし、使用条件に同意します。

    ![][13]

	これで、必要なクライアント アセンブリを参照できるようになり、新しいコード ファイルがいくつか追加されました。

9.  **ソリューション エクスプローラー**で **[Models]** を右クリックし、**[追加]**、**[クラス]** の順にクリックします。**[名前]** ボックスに「**OnlineOrder.cs**」と入力します。**[追加]** をクリックします。

### Web ロール用のコードの作成

このセクションでは、アプリケーションで表示するさまざまなページを作成します。

1.  Visual Studio で、OnlineOrder.cs ファイルの既存の名前空間定義を、次のコードに置き換えます。

	```
	namespace FrontendWebRole.Models
	{
	    public class OnlineOrder
	    {
	        public string Customer { get; set; }
	        public string Product { get; set; }
	    }
	}
	```

2.  **ソリューション エクスプローラー**で、**[Controllers\\HomeController.cs]** をダブルクリックします。次の **using** ステートメントを名前空間を含めるファイルの先頭に追加し、Service Bus と同様に先ほど作成したモデルの名前空間を追加します。

	```
	using FrontendWebRole.Models;
	using Microsoft.ServiceBus.Messaging;
	using Microsoft.ServiceBus;
	```

3.  また、Visual Studio の HomeController.cs ファイルで既存の名前空間の定義を次のコードに置き換えます。このコードには、キューへの項目の送信を処理するメソッドが含まれています。

	```
	namespace FrontendWebRole.Controllers
	{
	    public class HomeController : Controller
	    {
	        public ActionResult Index()
	        {
	            // Simply redirect to Submit, since Submit will serve as the
	            // front page of this application.
	            return RedirectToAction("Submit");
	        }
	
	        public ActionResult About()
	        {
	            return View();
	        }
	
	        // GET: /Home/Submit.
	        // Controller method for a view you will create for the submission
	        // form.
	        public ActionResult Submit()
	        {
	            // Will put code for displaying queue message count here.
	
	            return View();
	        }
	
	        // POST: /Home/Submit.
	        // Controller method for handling submissions from the submission
	        // form.
	        [HttpPost]
			// Attribute to help prevent cross-site scripting attacks and
			// cross-site request forgery.  
			[ValidateAntiForgeryToken]
	        public ActionResult Submit(OnlineOrder order)
	        {
	            if (ModelState.IsValid)
	            {
	                // Will put code for submitting to queue here.
	
	                return RedirectToAction("Submit");
	            }
	            else
	            {
	                return View(order);
	            }
	        }
	    }
	}
	```

4.  **[ビルド]** メニューの **[ソリューションのビルド]** をクリックして、ここまでの作業に問題がないことを確認します。

5.  次に、前の手順で作成した `Submit()` メソッドのビューを作成します。`Submit()` メソッド (パラメーターを受け取らない `Submit()` のオーバーロード) 内で右クリックし、**[ビューの追加]** を選択します。

    ![][14]

6.  ビューを作成するためのダイアログ ボックスが表示されます。**[テンプレート]** ボックスの一覧から **[作成]** を選択します。**[モデル クラス]** ボックスの一覧で **OnlineOrder** クラスをクリックします。

    ![][15]

7.  **[追加]** をクリックします。

8.  次に、アプリケーションの表示名を変更します。**ソリューション エクスプローラー**で、**Views\\Shared\\_Layout.cshtml** ファイルをダブルクリックして Visual Studio エディターで開きます。

9.  **My ASP.NET Application** となっている箇所をすべて **LITWARE'S Products** に置き換えます。

10. **Home**、**About**、および **Contact** の各リンクを削除します。以下の強調表示されたコードを削除してください。

	![][28]

11. 最後に、キューに関する情報を表示できるように、送信ページを変更します。**ソリューション エクスプローラー**で、**Views\\Home\\Submit.cshtml** ファイルをダブルクリックし、Visual Studio エディターで開きます。`<h2>Submit</h2>` という行の下に、次の行を追加します。この時点では、`ViewBag.MessageCount` は空の状態です。この値は後で入力します。

	```
	<p>Current number of orders in queue waiting to be processed: @ViewBag.MessageCount</p>
	```

12. これで、UI の実装が終わりました。**F5** キーを押してアプリケーションを実行し、期待どおりに表示されることを確認します。

    ![][17]

### Service Bus キューに項目を送信するためのコードの作成

次に、項目をキューに送信するためのコードを追加します。最初に、Service Bus キューの接続情報を含むクラスを作成します。次に、Global.aspx.cs から接続を初期化します。最後に、Service Bus キューに実際に項目を送信するために、HomeController.cs 内に作成してある送信用のコードを更新します。

1.  **ソリューション エクスプローラー**で、**FrontendWebRole** を右クリックします (ロールではなくプロジェクトを右クリック)。**[追加]**、**[クラス]** の順にクリックします。

2.  クラスに **QueueConnector.cs** という名前を付けます。**[追加]** をクリックしてクラスを作成します。

3.  接続情報をカプセル化して、Service Bus のキューへの接続を初期化するコードを追加します。QueueConnector.cs の内容全体を次のコードに置き換え、`your Service Bus namespace` の値 (名前空間の名前) と `yourKey` の値 (Azure ポータルから取得した**プライマリ キー**) を入力します。

	```
	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Web;
	using Microsoft.ServiceBus.Messaging;
	using Microsoft.ServiceBus;
	
	namespace FrontendWebRole
	{
	    public static class QueueConnector
	    {
	        // Thread-safe. Recommended that you cache rather than recreating it
	        // on every request.
	        public static QueueClient OrdersQueueClient;
	
	        // Obtain these values from the portal.
	        public const string Namespace = "your Service Bus namespace";
	
	        // The name of your queue.
	        public const string QueueName = "OrdersQueue";
	
	        public static NamespaceManager CreateNamespaceManager()
	        {
	            // Create the namespace manager which gives you access to
	            // management operations.
	            var uri = ServiceBusEnvironment.CreateServiceUri(
	                "sb", Namespace, String.Empty);
	            var tP = TokenProvider.CreateSharedAccessSignatureTokenProvider(
	                "RootManageSharedAccessKey", "yourKey");
	            return new NamespaceManager(uri, tP);
	        }
	
	        public static void Initialize()
	        {
	            // Using Http to be friendly with outbound firewalls.
	            ServiceBusEnvironment.SystemConnectivity.Mode =
	                ConnectivityMode.Http;
	
	            // Create the namespace manager which gives you access to
	            // management operations.
	            var namespaceManager = CreateNamespaceManager();
	
	            // Create the queue if it does not exist already.
	            if (!namespaceManager.QueueExists(QueueName))
	            {
	                namespaceManager.CreateQueue(QueueName);
	            }
	
	            // Get a client to the queue.
	            var messagingFactory = MessagingFactory.Create(
	                namespaceManager.Address,
	                namespaceManager.Settings.TokenProvider);
	            OrdersQueueClient = messagingFactory.CreateQueueClient(
	                "OrdersQueue");
	        }
	    }
	}
	```

4.  次に、**Initialize** メソッドが呼び出されるようにします。**ソリューション エクスプローラー**で、**Global.asax\\Global.asax.cs** をダブルクリックします。

5.  **Application\_Start** メソッドの最後に次のコード行を追加します。

	```
	FrontendWebRole.QueueConnector.Initialize();
	```

6.  最後に、前の手順で作成した Web コードを更新し、項目をキューに送信します。**ソリューション エクスプローラー**で、**[Controllers\\HomeController.cs]** をダブルクリックします。

7.  キューのメッセージ数を取得するために、`Submit()` メソッド (パラメーターを受け取らないオーバーロード) を次のように更新します。

	```
	public ActionResult Submit()
	{
	    // Get a NamespaceManager which allows you to perform management and
	    // diagnostic operations on your Service Bus queues.
	    var namespaceManager = QueueConnector.CreateNamespaceManager();
	
	    // Get the queue, and obtain the message count.
	    var queue = namespaceManager.GetQueue(QueueConnector.QueueName);
	    ViewBag.MessageCount = queue.MessageCount;
	
	    return View();
	}
	```

8.  キューに注文情報を送信するために、`Submit(OnlineOrder order)` メソッド (パラメーターを 1 つ受け取るオーバーロード) を次のように更新します。

	```
	public ActionResult Submit(OnlineOrder order)
	{
	    if (ModelState.IsValid)
	    {
	        // Create a message from the order.
	        var message = new BrokeredMessage(order);
	
	        // Submit the order.
	        QueueConnector.OrdersQueueClient.Send(message);
	        return RedirectToAction("Submit");
	    }
	    else
	    {
	        return View(order);
	    }
	}
	```

9.  ここで再び、アプリケーションを実行します。注文を送信するたびに、メッセージ数が増えていきます。

    ![][18]

## worker ロールを作成する

次に、送信された注文を処理する worker ロールを作成します。この例では、Visual Studio プロジェクト テンプレートの **Worker Role with Service Bus Queue** を使用します。必要な資格情報は、既にポータルから取得しています。

1. Visual Studio を Azure アカウントに接続していることを確認します。

2.  Visual Studio の**ソリューション エクスプローラー**で、**MultiTierApp** プロジェクト内の **Roles** フォルダーを右クリックします。

3.  **[追加]** をクリックし、**[新しい worker ロール プロジェクト]** をクリックします。**[新しいロール プロジェクトの追加]** ダイアログ ボックスが表示されます。

	![][26]

4.  **[新しいロール プロジェクトの追加]** ダイアログ ボックスの **[Worker Role with Service Bus Queue]** をクリックします。

	![][23]

5.  **[名前]** ボックスに、プロジェクト名として「**OrderProcessingRole**」を入力します。**[追加]** をクリックします。

6.  「Service Bus 名前空間の作成」セクションの手順 9. で取得した接続文字列をクリップボードにコピーします。

7.  **ソリューション エクスプローラー**で、手順 5. で作成した **OrderProcessingRole** を右クリックします (右クリックするのは、**[ロール]** の **OrderProcessingRole** です。クラスではありません)。**[プロパティ]** をクリックします。

8.  **[プロパティ]** ダイアログ ボックスの **[設定]** タブで、**Microsoft.ServiceBus.ConnectionString** の **[値]** ボックス内をクリックし、手順 6. でコピーしたエンドポイントの値を貼り付けます。

	![][25]

9.  キューからの注文を処理するときの注文を表す **OnlineOrder** クラスを作成します。作成済みのクラスを再利用できます。**ソリューション エクスプローラー**で、**OrderProcessingRole** クラスを右クリックします (ロールではなく、クラスを右クリック)。**[追加]**、**[既存の項目]** の順にクリックします。

10. **FrontendWebRole\\Models** のサブフォルダーに移動し、**OnlineOrder.cs** をダブルクリックしてこのプロジェクトに追加します。

11. **WorkerRole.cs** 内の **QueueName** 変数の値を `"ProcessingQueue"` から `"OrdersQueue"` に変更します (次のコードを参照)。

	```
	// The name of your queue.
	const string QueueName = "OrdersQueue";
	```

12. WorkerRole.cs ファイルの先頭に次の using ステートメントを追加します。

	```
	using FrontendWebRole.Models;
	```

13. `Run()` 関数の `OnMessage()` の呼び出し内にある `try` 句の内容を次のコードに置き換えます。

	```
	Trace.WriteLine("Processing", receivedMessage.SequenceNumber.ToString());
	// View the message as an OnlineOrder.
	OnlineOrder order = receivedMessage.GetBody<OnlineOrder>();
	Trace.WriteLine(order.Customer + ": " + order.Product, "ProcessingMessage");
	receivedMessage.Complete();
	```

14. これでアプリケーションが完成しました。ソリューション エクスプローラーで、MultiTierApp のプロジェクトを右クリックして、**[スタートアップ プロジェクトとして設定]** を選択して F5 キーを押すと、完全なアプリケーションをテストできます。メッセージ数が増えないことに注意してください。これは、worker ロールがキューの項目を処理し、完了としてマークしているためです。Azure コンピューティング エミュレーターのUI を表示すると、worker ロールのトレース出力を確認できます。これを実行するには、タスク バーの通知領域のエミュレーター アイコンを右クリックし、**[コンピューティング エミュレーター UI の表示]** をクリックします。

    ![][19]

    ![][20]

## 次のステップ  

Service Bus の詳細については、次のリソースを参照してください。

* [Azure の Service Bus][sbmsdn]
* [Service Bus サービス ページ][sbwacom]
* [Service Bus キューの使用方法][sbwacomqhowto]

多層のシナリオの詳細については、次のページを参照してください。

* [.NET Multi-Tier Application Using Storage Tables, Queues, and Blobs (ストレージ テーブル、キュー、BLOB を使用する .NET 多層アプリケーション)][mutitierstorage]

  [0]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-01.png
  [1]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-100.png
  [2]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-101.png
  [Gツールと SDK の入手]: http://go.microsoft.com/fwlink/?LinkId=271920


  [GetSetting]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudconfigurationmanager.getsetting.aspx
  [Microsoft.WindowsAzure.Configuration.CloudConfigurationManager]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.cloudconfigurationmanager.aspx
  [NamespaceMananger]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.namespacemanager.aspx

  [QueueClient]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queueclient.aspx

  [TopicClient]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.topicclient.aspx

  [EventHubClient]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.eventhubclient.aspx

  [9]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-10.png
  [10]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-11.png
  [11]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-02.png
  [12]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-12.png
  [13]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-13.png
  [14]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-33.png
  [15]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-34.png
  [16]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-14.png
  [17]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-36.png
  [18]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-37.png

  [19]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-38.png
  [20]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-39.png
  [23]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRole1.png
  [25]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRoleProperties.png
  [26]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBNewWorkerRole.png
  [28]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-40.png

  [sbmsdn]: http://msdn.microsoft.com/library/azure/ee732537.aspx
  [sbwacom]: /documentation/services/service-bus/
  [sbwacomqhowto]: service-bus-dotnet-get-started-with-queues.md
  [mutitierstorage]: https://code.msdn.microsoft.com/Windows-Azure-Multi-Tier-eadceb36
  

<!---HONumber=AcomDC_0928_2016-->