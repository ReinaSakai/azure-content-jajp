<properties
	pageTitle="Mobile Apps 用 Node.js バックエンド サーバー SDK を操作する方法 | Azure App Service"
	description="Azure App Service Mobile Apps 用の Node.js バックエンド サーバー SDK を操作する方法について説明します。"
	services="app-service\mobile"
	documentationCenter=""
	authors="adrianhall"
	manager="erikre"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-multiple"
	ms.devlang="node"
	ms.topic="article"
	ms.date="03/07/2016"
	ms.author="adrianhall"/>

# Azure Mobile Apps Node.js SDK の使用方法

[AZURE.INCLUDE [app-service-mobile-selector-server-sdk](../../includes/app-service-mobile-selector-server-sdk.md)]

この記事では、Azure App Service Mobile Apps で Node.js バックエンドを使用する方法についての詳細な情報と例を提供します。

## <a name="Introduction"></a>はじめに

Azure App Service Mobile Apps は、Web アプリケーションにモバイルに最適化されたデータ アクセス Web API を追加する機能を提供します。Azure App Service Mobile Apps SDK は、ASP.NET と Node.js の Web アプリケーション向けに用意されています。この SDK を使用すると、次の処理を実行できます。

- データ アクセスのためのテーブル操作 (読み取り、挿入、更新、削除)
- カスタム API 操作

両方の操作では、エンタープライズ ID の Azure Active Directory だけでなく、Facebook、Twitter、Google および Microsoft などのソーシャル ID プロバイダーを含む、Azure App Service で許可されているすべての ID プロバイダーでの認証が提供されます。

各ユース ケースのサンプルは、[GitHub の samples ディレクトリ]にあります。

## <a name="howto-cmdline-basicapp"></a>方法: コマンド ラインを使用して基本的な Node.js バックエンドを作成する

Azure App Service Mobile Apps の Node.js バックエンドはすべて ExpressJS アプリケーションとして開始されます。ExpressJS は、Node.js で使用可能な最も人気のある Web サービス フレームワークです。次のように、基本的な [Express] アプリケーションを作成できます。

1. コマンドまたは PowerShell ウィンドウで、プロジェクト用の新しいディレクトリを作成します。

        mkdir basicapp

2. npm init を実行して、パッケージの構造を初期化します。

        cd basicapp
        npm init

    npm init コマンドでは、プロジェクトを初期化するための一連の質問が示されます。以下の出力例を参照してください。

    ![npm init の出力][0]

3. npm リポジトリから express および azure-mobile-apps ライブラリをインストールします。

        npm install --save express azure-mobile-apps

4. app.js ファイルを作成して、基本的なモバイル サーバーを実装します。

		var express = require('express'),
			azureMobileApps = require('azure-mobile-apps');

		var app = express(),
			mobile = azureMobileApps();

		// Define a TodoItem table
		mobile.tables.add('TodoItem');

		// Add the mobile API so it is accessible as a Web API
		app.use(mobile);

		// Start listening on HTTP
		app.listen(process.env.PORT || 3000);

このアプリケーションでは、単一のエンドポイント (`/tables/TodoItem`) でモバイルに最適化された WebAPI が作成され、動的スキーマを使用する基になる SQL データ ストアへの非認証アクセスが可能になります。次のクライアント ライブラリのクイック スタートに従う場合に適しています。

- [Android クライアントのクイック スタート]
- [Apache Cordova クライアントのクイック スタート]
- [iOS クライアントのクイック スタート]
- [Windows ストア クライアントのクイック スタート]
- [Xamarin.iOS クライアントのクイック スタート]
- [Xamarin.Android クライアントのクイック スタート]
- [Xamarin.Forms クライアントのクイック スタート]


この基本的なアプリケーションのコードは、[GitHub の basicapp サンプル]にあります。

## <a name="howto-vs2015-basicapp"></a>方法: Visual Studio 2015 を使用して Node バックエンドを作成する

Visual Studio 2015 には、IDE 内で Node.js アプリケーションを開発するための拡張機能が必要です。最初に、[Node.js Tools 1.1 for Visual Studio] をダウンロードしてインストールします。Node.js Tools for Visual Studio をインストールしたら、Express 4.x アプリケーションを作成します。

1. **[新しいプロジェクト]** ダイアログを開きます (**[ファイル]**、**[新規作成]**、**[プロジェクト...]** の順にクリック)。

2. **[テンプレート]**、**[JavaScript]**、**[Node.js]** の順に展開します。

3. **[Basic Azure Node.js Express 4 Application]** を選択します。

4. プロジェクト名を入力します。*[OK]* をクリックします。

	![Visual Studio 2015 の新しいプロジェクト][1]

5. **npm** ノードを右クリックし、**[新しい npm パッケージのインストール]** を選択します。

6. 最初の Node.js アプリケーションの作成時に、npm カタログを更新することが必要な場合があります。更新が必要な場合は、更新するよう求められるので、**[更新]** をクリックします。

7. 検索ボックスに「_azure-mobile-apps_」と入力します。**azure-mobile-apps 2.0.0** パッケージをクリックしてから、**[パッケージのインストール]** をクリックします。

	![新しい npm パッケージのインストール][2]

8. **[閉じる]** をクリックします。

9. _app.js_ ファイルを開き、Azure Mobile Apps SDK のサポートを追加します。ライブラリの require ステートメントの下の 6 行目に、次のコードを追加します。

        var bodyParser = require('body-parser');
        var azureMobileApps = require('azure-mobile-apps');

    他の app.use ステートメントの後の約 27 行目に、次のコードを追加します。

        app.use('/users', users);

        // Azure Mobile Apps Initialization
        var mobile = azureMobileApps();
        mobile.tables.add('TodoItem');
        app.use(mobile);

    ファイルを保存します。

10. アプリケーションをローカルで実行するか (API は http://localhost:3000 で動作します)、Azure に発行します。

## <a name="download-quickstart"></a>方法: Git を使用して Node.js バックエンド クイック スタート コード プロジェクトをダウンロードする

ポータルの **[クイック スタート]** ブレードを使用して新しい Node.js モバイル アプリ バックエンドを作成すると、新しい Node.js プロジェクトが作成され、サイトにデプロイされます。ポータルでは、テーブルと API を追加したり、Node.js バックエンドのコード ファイルを編集したりできます。また、さまざまなデプロイ ツールの 1 つを使用してバックエンド プロジェクトをダウンロードすると、テーブルと API を追加または変更した後でプロジェクトを再発行できます。詳細については、[Azure App Service のデプロイ ガイド]を参照してください。次の手順では、Git リポジトリを使用して、クイック スタート プロジェクトのコードをダウンロードします。

1. Git をまだインストールしていない場合はインストールします。Git をインストールするために必要な手順は、オペレーティング システムによって異なります。オペレーティング システム固有の配布とインストールのガイダンスについては、「[Installing Git (Git のインストール)](http://git-scm.com/book/en/Getting-Started-Installing-Git)」を参照してください。

2. 「[Web アプリのリポジトリの有効化](../app-service-web/web-sites-publish-source-control.md#Step4)」の手順に従って、バックエンド サイトの Git リポジトリを有効にします。このとき、デプロイメント用のユーザー名とパスワードをメモしておきます。

3. モバイル アプリ バックエンドのブレードで、**[Git クローン URL]** の設定をメモしておきます。

4.  次の例のように、Git 対応のコマンド ライン ツールで Git クローン URL を使用して `git clone` コマンドを実行します。このとき、必要な場合はパスワードを入力します。

		$ git clone https://username@todolist.scm.azurewebsites.net:443/todolist.git

5. ローカル ディレクトリ (上の例では /todolist) を参照し、プロジェクト ファイルがダウンロード済みであることを確認します。/tables サブフォルダーには、todoitem.json ファイル (テーブルに対するアクセス許可を定義します) と、todoitem.js ファイル (テーブルの CRUD 操作スクリプトを定義します) があります。

6. プロジェクト ファイルに変更を加えた後に、次のコマンドを実行して、変更をサイトに追加、コミット、アップロードします。

		$ git commit -m "updated the table script"
		$ git push origin master

	新しいファイルをプロジェクトに追加する場合は、最初に `git add .` コマンドを実行する必要があります。

一連の新しいコミットがサイトにプッシュされるたびに、そのサイトは再発行されます。

## <a name="howto-publish-to-azure"></a>方法: Azure に Node.js バックエンドを発行する

Microsoft Azure では、Azure サービスに Azure App Service Mobile Apps Node.js バックエンドを発行するための多数のメカニズムが提供されます。Visual Studio に統合されたデプロイ ツール、コマンドライン ツールおよびソース管理に基づく継続的なデプロイ オプションも利用します。このトピックの詳細については、[Azure App Service のデプロイ ガイド]を参照してください。

Azure App Service には、デプロイ前に確認する必要がある Node.js アプリケーションに関する以下の特定のアドバイスがあります。

- [Node のバージョンを指定する]方法
- [Node モジュールを使用する]方法

## <a name="howto-enable-homepage"></a>方法: アプリケーションのホーム ページを有効にする

多くのアプリケーションは Web とモバイル アプリを組み合わせており、ExpressJS フレームワークを使用すると、この 2 つを結合することができます。しかし、場合によっては、モバイル インターフェイスだけを実装することができます。アプリ サービスを確実に稼働させるために、ランディング ページを提供すると便利です。ホーム ページを指定するか、一時的なホーム ページを有効にします。一時的なホーム ページを有効にするには、Mobile App コンストラクターを次のように調整します。

    var mobile = azureMobileApps({ homePage: true });

ローカルで開発するときのみ、このオプションを利用する場合は、この設定を `azureMobile.js` ファイルに追加することができます。

## <a name="TableOperations"></a>概要: テーブル操作

azure-mobile-apps Node.js Server SDK では、WebAPI として Azure SQL Database に格納されたデータ テーブルを公開するためのメカニズムが提供されます。以下の 5 つの操作が提供されます。

| 操作 | 説明 |
| --------- | ----------- |
| GET /tables/\_tablename\_ | テーブルのすべてのレコードを取得します。 |
| GET /tables/\_tablename\_/:id | テーブルの特定のレコードを取得します。 |
| POST /tables/\_tablename\_ | テーブルに新しいレコードを作成します。 |
| PATCH /tables/\_tablename\_/:id | テーブルの既存のレコードを更新します。 |
| DELETE /tables/\_tablename\_/:id | テーブルのレコードを削除します。 |

この WebAPI は [OData] をサポートし、テーブル スキーマを拡張して[オフライン データ同期]をサポートします。

## <a name="howto-dynamicschema"></a>方法: 動的スキーマを使用してテーブルを定義する

テーブルを使用する前に定義する必要があります。テーブルは、静的スキーマで (開発者がスキーマ内の列を定義する場合)、または動的に (SDK で受信要求に基づいてスキーマを制御する場合) 定義できます。さらに、開発者は、定義に Javascript コードを追加することで、WebAPI の特定の側面を制御できます。

ベスト プラクティスとして、テーブル ディレクトリ内の Javascript ファイルに各テーブルを定義し、tables.import() メソッドを使用してテーブルをインポートする必要があります。basic-app を拡張すると、app.js ファイルが次のように調整されます。

    var express = require('express'),
        azureMobileApps = require('azure-mobile-apps');

    var app = express(),
	    mobile = azureMobileApps();

    // Define the database schema that is exposed
    mobile.tables.import('./tables');

    // Provide initialization of any tables that are statically defined
    mobile.tables.initialize().then(function () {
        // Add the mobile API so it is accessible as a Web API
        app.use(mobile);

        // Start listening on HTTP
        app.listen(process.env.PORT || 3000);
    });

次のように ./tables/TodoItem.js にテーブルを定義します。

    var azureMobileApps = require('azure-mobile-apps');

    var table = azureMobileApps.table();

    // Additional configuration for the table goes here

    module.exports = table;

テーブルでは、既定で動的スキーマが使用されます。動的スキーマをグローバルに無効にするには、Azure ポータル内でアプリ設定の **MS\_DynamicSchema** を false に設定します。

完全な例は [GitHub の todo サンプル]にあります。

## <a name="howto-staticschema"></a>方法: 静的スキーマを使用してテーブルを定義する

WebAPI を使用して公開する列を明示的に定義することができます。azure-mobile-apps Node.js SDK では、指定した一覧にオフライン データ同期に必要な他の列が自動的に追加されます。たとえば、クイック スタート クライアント アプリケーションには、text (文字列) と complete (ブール値) という 2 つの列を持つテーブルが必要になります。これは、テーブル定義 JavaScript ファイル (テーブル ディレクトリにある) に次のように定義できます。

    var azureMobileApps = require('azure-mobile-apps');

    var table = azureMobileApps.table();

    // Define the columns within the table
    table.columns = {
        "text": "string",
        "complete": "boolean"
    };

    // Turn off dynamic schema
    table.dynamicSchema = false;

    module.exports = table;

静的にテーブルを定義する場合は、tables.initialize() メソッドを呼び出して起動時にデータベース スキーマを作成する必要もあります。tables.initialize() メソッドは [Promise] を返します。これは、Web サービスがデータベースの初期化前に要求を処理しないようにするために使用されます。

## <a name="howto-sqlexpress-setup"></a>方法: ローカル コンピューター上で開発データ ストアとして SQL Express を使用する

Azure Mobile Apps Node SDK には、データをすぐに使用できる 3 つのオプションが用意されています。

- **メモリ** ドライバーを使用して、非永続ストアの例を提供します。
- **mssql** ドライバーを使用して、開発用の SQL Express データ ストアを提供します。
- **mssql** ドライバーを使用して、実稼働用の Azure SQL Database データ ストアを提供します。

Azure Mobile Apps Node.js SDK では [mssql Node.js パッケージ]を使用して、SQL Express と SQL Database 両方への接続を確立および使用します。このパッケージでは、SQL Express インスタンスで TCP 接続を有効にする必要があります。

> [AZURE.TIP] メモリ ドライバーでは、テスト用の完全な機能セットは提供されません。ローカルでバックエンドをテストする場合は、SQL Express データ ストアと mssql ドライバーを使用することをお勧めします。

1. [Microsoft SQL Server 2014 Express] をダウンロードしてインストールします。必ず、SQL Server 2014 Express with Tools エディションをインストールしてください。明示的に 64 ビットのサポートが要求された場合を除き、32 ビット バージョンを使用することで実行時のメモリ使用量が少なくなります。

2. SQL Server 2014 構成マネージャーを実行します。

  1. 左側のツリー メニューにある **[SQL Server ネットワークの構成]** ノードを展開します。
  2. **[SQLEXPRESS のプロトコル]** をクリックします。
  3. **[TCP/IP]** を右クリックし、**[有効化]** を選択します。ポップアップ ダイアログで **[OK]** をクリックします。
  4. **[TCP/IP]** を右クリックし、**[プロパティ]** を選択します。
  5. **[IP アドレス]** タブをクリックします。
  6. **[IPAll]** ノードを見つけます。**[TCP ポート]** フィールドに、「**1433**」と入力します。

	 	 ![Configure SQL Express for TCP/IP][3]
  7. **[OK]** をクリックします。ポップアップ ダイアログで **[OK]** をクリックします。
  8. 左側のツリー メニューにある **[SQL Server のサービス]** をクリックします。
  9. **[SQL Server (SQLEXPRESS)]** を右クリックし、**[再起動]** を選択します。
  10. SQL Server 2014 構成マネージャーを閉じます。

3. SQL Server 2014 Management Studio を実行して、ローカルの SQL Express インスタンスに接続します。

  1. オブジェクト エクスプローラーでインスタンスを右クリックし、**[プロパティ]** を選択します。
  2. **[セキュリティ]** ページを選択します。
  3. **[SQL Server 認証モードと Windows 認証モード]** が選択されていることを確認します。
  4. **[OK]** をクリックします。

  		![Configure SQL Express Authentication][4]

  5. オブジェクト エクスプローラーで、**[セキュリティ]**、**[ログイン]** の順に展開します。
  6. **[ログイン]** を右クリックし、**[新しいログイン]** を選択します。
  7. ログイン名を入力します。**[SQL Server 認証]** を選択します。パスワードを入力し、**[パスワードの確認入力]** に同じパスワードを入力します。パスワードは、Windows の複雑さの要件を満たしている必要があります。
  8. **[OK]** をクリックします。

  		![Add a new user to SQL Express][5]

  9. 新しいログインを右クリックし、**[プロパティ]** を選択します。
  10. **[サーバー ロール]** ページを選択します。
  11. **dbcreator** サーバー ロールの横にあるチェック ボックスをオンにします。
  12. **[OK]** をクリックします。
  13. SQL Server 2015 Management Studio を閉じます。

選択したユーザー名とパスワードは必ず記録してください。特定のデータベース要件に応じて、他のサーバーの役割またはアクセス許可の割り当てが必要になる場合があります。

Node.js アプリケーションは、**SQLCONNSTR\_MS\_TableConnectionString** 環境変数を読み取って、このデータベースの接続文字列を確認します。ご使用の環境内にこれを設定することができます。たとえば、PowerShell を使用して、以下のようにこの環境変数を設定することができます。

    $env:SQLCONNSTR_MS_TableConnectionString = "Server=127.0.0.1; Database=mytestdatabase; User Id=azuremobile; Password=T3stPa55word;"

TCP/IP 接続を介して、データベースにアクセスし、接続用のユーザー名とパスワードを入力する必要があることに注意してください。

## <a name="howto-config-localdev"></a>方法: ローカル開発用のプロジェクトを構成する

Azure Mobile Apps は、ローカル ファイルシステムから _azureMobile.js_ という JavaScript ファイルを読み取ります。運用環境で Azure Mobile Apps SDK を構成する場合は、このファイルを使用しないでください。代わりに、[Azure ポータル]内の [アプリ設定] を使用します。_azureMobile.js_ ファイルでは構成オブジェクトをエクスポートする必要があります。最も一般的な設定は次のとおりです。

- データベース設定
- 診断ログ設定
- 代替 CORS 設定

上記のデータベース設定を実装する _azureMobile.js_ ファイルの例を次に示します。

    module.exports = {
        cors: {
            origins: [ 'localhost' ]
        },
        data: {
            provider: 'mssql',
            server: '127.0.0.1',
            database: 'mytestdatabase',
            user: 'azuremobile',
            password: 'T3stPa55word'
        },
        logging: {
            level: 'verbose'
        }
    };

パスワードがクラウドに保存されないように、_azureMobile.js_ を _.gitignore_ ファイル (または他のソース コード管理の無視ファイル) に追加することをお勧めします。必ず、[Azure ポータル]内の [アプリ設定] で運用設定を構成してください。

## <a name="howto-appsettings"></a>方法: モバイル アプリのアプリ設定を構成する

_azureMobile.js_ ファイル内のほとんどの設定には、[Azure ポータル] 内に対応するアプリ設定があります。[アプリ設定] でアプリを構成するには、次の一覧を使用します。

| アプリ設定 | _azureMobile.js_ 設定 | 説明 | 有効な値 |
| :-------------------------- | :------------------------ | :---------------------------------------- | :------------------------------------------ |
| **MS\_MobileAppName** | name | アプリの名前 | string |
| **MS\_MobileLoggingLevel** | logging.level | ログ記録するメッセージの最小ログ レベル | error、warning、info、verbose、debug、silly |
| **MS\_DebugMode** | debug | デバッグ モードの有効化または無効化 | true、false |
| **MS\_TableSchema** | data.schema | SQL テーブルの既定のスキーマ名 | string (既定: dbo) |
| **MS\_DynamicSchema** | data.dynamicSchema | デバッグ モードの有効化または無効化 | true、false |
| **MS\_DisableVersionHeader** | version (undefined に設定)| X-ZUMO-Server-Version ヘッダーの無効化 | true、false |
| **MS\_SkipVersionCheck** | skipversioncheck | クライアント API バージョン チェックの無効化 | true、false |

アプリ設定を設定するには:

1. [Azure ポータル]にログインします。
2. **[すべてのリソース]** または **[App Services]** を選択し、モバイル アプリの名前をクリックします。
3. 既定で [設定] ブレードが開きます。開かない場合は、**[設定]** をクリックします。
4. [全般] メニューで、**[アプリケーション設定]** をクリックします。
5. [アプリ設定] セクションまでスクロールします。
6. アプリ設定が既に存在する場合は、アプリ設定の値をクリックして値を編集します。
7. アプリ設定が存在しない場合は、[キー] ボックスにアプリ設定を入力し、[値] ボックスに値を入力します。
8. 完了したら、**[保存]** をクリックします。

ほとんどのアプリ設定を変更した場合、サービスの再起動が必要になります。

## <a name="howto-use-sqlazure"></a>方法: SQL Database を運用データ ストアとして使用する

<!--- ALTERNATE INCLUDE - we can't use ../includes/app-service-mobile-dotnet-backend-create-new-service.md - slightly different semantics -->

Azure SQL Database をデータ ストアとして使用する方法は、Azure App Service アプリケーションのすべての種類で同じです。モバイル アプリ バックエンドをまだ作成していない場合は、次の手順に従って新しいものを作成します。

1. [Azure ポータル]にログインします。

2. ウィンドウの左上で、**[+ 新規]**、**[Web + モバイル]**、**[モバイル アプリ]** の順にクリックし、モバイル アプリ バックエンドの名前を指定します。

3. **[リソース グループ]** ボックスで、アプリと同じ名前を入力します。

4. 既定の App Service プランが選択されます。App Service プランを変更するには、[App Service プラン]、**[+ 新規作成]** の順にクリックします。新しい App Service プランの名前を指定し、適切な場所を選択します。[価格レベル] をクリックし、サービスに適切な価格レベルを選択します。**[すべて表示]** を選択して、**Free** や **Shared** などの価格オプションをさらに表示します。価格レベルを選択したら、**[選択]** をクリックします。**[App Service プラン]** ブレードに戻り、**[OK]** をクリックします。

5. **[作成]** をクリックします。これにより、モバイル アプリ バックエンドが作成され、後でサーバー プロジェクトをデプロイすることができます。モバイル アプリ バックエンドのプロビジョニングには数分かかる場合があります。モバイル アプリ バックエンドのプロビジョニングが完了すると、ポータルでモバイル アプリ バックエンドの **[設定]** ブレードが開きます。

モバイル アプリ バックエンドが作成されたら、既存の SQL データベースをモバイル アプリ バックエンドに接続するか、新しい SQL データベースを作成するかを選択できます。このセクションでは、新しい SQL データベースを作成します。

> [AZURE.NOTE] 新しいモバイル アプリ バックエンドと同じ場所に、既にデータベースがある場合は、**[既存のデータベースを使用する]** を選ぶと、そのデータベースを選択できます。別の場所にあるデータベースを使用することは、帯域幅コストと待機時間が増加するため、お勧めしません。

6. 新しいモバイル アプリ バックエンドで、**[設定]**、**[モバイル アプリ]**、**[データ]**、**[+ 追加]** の順にクリックします。

7. **[データ接続の追加]** ブレードで、**[SQL Database - 必要な設定の構成]**、**[新しいデータベースの作成]** の順にクリックします。**[名前]** フィールドに新しいデータベースの名前を入力します。

8. **[サーバー]** をクリックします。**[新しいサーバー]** ブレードで、**[サーバー名]** フィールドに一意のサーバー名を入力し、**[サーバー管理ログイン]** と **[パスワード]** に適切な内容を指定します。**[Azure サービスにサーバーへのアクセスを許可する]** チェック ボックスがオンになっていることを確認します。**[OK]** をクリックします。

	![Azure SQL Database の作成][6]

9. **[新しいデータベース]** ブレードで、**[OK]** をクリックします。

10. **[データ接続の追加]** ブレードに戻り、**[接続文字列]** を選択して、データベースの作成時に指定したログイン名とパスワードを入力します。既存のデータベースを使用する場合は、そのデータベースのログイン資格情報を入力します。入力したら、**[OK]** をクリックします。

11. もう一度 **[データ接続の追加]** ブレードに戻り、**[OK]** をクリックしてデータベースを作成します。

<!--- END OF ALTERNATE INCLUDE -->

データベースの作成には数分かかる場合があります。**[通知]** 領域を使用して、デプロイの進行状況を監視します。データベースのデプロイが正常に完了するまでは、先に進まないでください。正常にデプロイされると、モバイル バックエンドの [アプリ設定] で、SQL Database インスタンスの接続文字列が作成されます。このアプリ設定を確認するには、**[設定]**、**[アプリケーション設定]**、**[接続文字列]** を順に選択します。

## <a name="howto-tables-auth"></a>方法: テーブルへのアクセスに認証を要求する

テーブル エンドポイントで App Service 認証を使用する場合は、まず、[Azure ポータル]で App Service 認証を構成する必要があります。Azure App Service での認証構成の詳細については、以下の使用する ID プロバイダーの構成ガイドを参照してください。

- [Azure Active Directory 認証の構成方法]
- [Facebook 認証の構成方法]
- [Google 認証の構成方法]
- [Microsoft 認証の構成方法]
- [Twitter 認証の構成方法]

各テーブルには、テーブルへのアクセスを制御するために使用できる access プロパティがあります。次のサンプルでは、認証を必要とする静的に定義されたテーブルを示します。

    var azureMobileApps = require('azure-mobile-apps');

    var table = azureMobileApps.table();

    // Define the columns within the table
    table.columns = {
        "text": "string",
        "complete": "boolean"
    };

    // Turn off dynamic schema
    table.dynamicSchema = false;

    // Require authentication to access the table
    table.access = 'authenticated';

    module.exports = table;

access プロパティには、次の 3 つの値を使用できます。

  - *anonymous* は、認証なしでデータを読み取る許可をクライアント アプリケーションに与えることを示します。
  - *authenticated* は、クライアント アプリケーションが要求で有効な認証トークンを送信する必要があることを示します。
  - *disabled* は、このテーブルが現在無効になっていることを示します。

access プロパティが定義されていない場合は、非認証アクセスが許可されます。

## <a name="howto-tables-getidentity"></a>方法: テーブルで認証要求を使用する

認証が設定されている場合に、要求されている要求の数を設定することができます。これらの要求は `context.user` オブジェクトでは通常使用できません。ただし、`context.user.getIdentity()` メソッドで取得できます。`getIdentity()` メソッドは、オブジェクトに解決される Promise を返します。オブジェクトは、認証方法 (facebook、google、twitter、microsoftaccount または aad) によってキー付けされます。

たとえば、Microsoft アカウント認証を設定し、電子メール アドレス要求を要求する場合は、次のレコードに電子メール アドレスを追加できます。

    var azureMobileApps = require('azure-mobile-apps');

    // Create a new table definition
    var table = azureMobileApps.table();

    table.columns = {
        "emailAddress": "string",
        "text": "string",
        "complete": "boolean"
    };
    table.dynamicSchema = false;
    table.access = 'authenticated';

    /**
    * Limit the context query to those records with the authenticated user email address
    * @param {Context} context the operation context
    * @returns {Promise} context execution Promise
    */
    function queryContextForEmail(context) {
        return context.user.getIdentity().then((data) => {
            context.query.where({ emailAddress: data.microsoftaccount.claims.emailaddress });
            return context.execute();
        });
    }

    /**
    * Adds the email address from the claims to the context item - used for
    * insert operations
    * @param {Context} context the operation context
    * @returns {Promise} context execution Promise
    */
    function addEmailToContext(context) {
        return context.user.getIdentity().then((data) => {
            context.item.emailAddress = data.microsoftaccount.claims.emailaddress;
            return context.execute();
        });
    }

    // Configure specific code when the client does a request
    // READ - only return records belonging to the authenticated user
    table.read(queryContextForEmail);

    // CREATE - add or overwrite the userId based on the authenticated user
    table.insert(addEmailToContext);

    // UPDATE - only allow updating of record belong to the authenticated user
    table.update(queryContextForEmail);

    // DELETE - only allow deletion of records belong to the authenticated uer
    table.delete(queryContextForEmail);

    module.exports = table;

どのような要求が使用できるかを確認するには、Web ブラウザーを使用し、サイトの `/.auth/me` エンドポイントを表示します 。

## <a name="howto-tables-disabled"></a>方法: 特定のテーブル操作へのアクセスを無効にする

テーブルでの表示だけでなく、個々の操作を制御するために access プロパティを使用できます。操作には以下の 4 つがあります。

  - *read* は、テーブルに対する RESTful GET 操作です。
  - *insert* は、テーブルに対する RESTful POST 操作です。
  - *update* は、テーブルに対する RESTful PATCH 操作です。
  - *delete* は、テーブルに対する RESTful DELETE 操作です。

たとえば、読み取り専用の非認証テーブルを指定するとします。これは、次のようなテーブル定義で指定することができます。

    var azureMobileApps = require('azure-mobile-apps');

    var table = azureMobileApps.table();

    // Read-Only table - only allow READ operations
    table.read.access = 'anonymous';
    table.insert.access = 'disabled';
    table.update.access = 'disabled';
    table.delete.access = 'disabled';

    module.exports = table;

## <a name="howto-tables-query"></a>方法: テーブル操作で使用されるクエリを調整する

テーブル操作の一般的な要件は、データの制限付きビューを提供することです。たとえば、ユーザーが独自のレコードの読み取りまたは更新のみができるように、認証済みユーザー ID がタグ付けされているテーブルを提供できます。次のようなテーブル定義では、この機能が提供されます。

    var azureMobileApps = require('azure-mobile-apps');

    var table = azureMobileApps.table();

    // Define a static schema for the table
    table.columns = {
        "userId": "string",
        "text": "string",
        "complete": "boolean"
    };
    table.dynamicSchema = false;

    // Require authentication for this table
    table.access = 'authenticated';

    // Ensure that only records for the authenticated user are retrieved
    table.read(function (context) {
		context.query.where({ userId: context.user.id });
		return context.execute();
	});

    // When adding records, add or overwrite the userId with the authenticated user
    table.insert(function (context) {
	    context.item.userId = context.user.id;
	    return context.execute();
    });

    module.exports = table;

通常はクエリを実行する操作には、where 句で調整できる query プロパティがあります。query プロパティは [QueryJS] オブジェクトであり、これを使用して、データ バックエンドで処理できるものに OData クエリを変換します。(上記のような) 単純な等式の場合、マップを使用できます。特定の SQL 句の追加も比較的容易です。

    context.query.where('myfield eq ?', 'value');

## <a name="howto-tables-softdelete"></a>方法: テーブルの論理削除を構成する

論理削除では実際にレコードは削除されません。代わりに、削除列を true に設定して、データベース内のレコードを削除済みとしてマークします。Mobile Client SDK で IncludeDeleted() が使用されない限り、Azure Mobile Apps SDK によって結果から論理削除レコードが自動的に削除されます。論理削除のテーブルを構成するには、テーブル定義ファイルで softDelete プロパティを設定します。例を以下に示します。

    var azureMobileApps = require('azure-mobile-apps');

    var table = azureMobileApps.table();

    // Define the columns within the table
    table.columns = {
        "text": "string",
		"complete": "boolean"
	};

	// Turn off dynamic schema
	table.dynamicSchema = false;

	// Turn on Soft Delete
	table.softDelete = true;

	// Require authentication to access the table
	table.access = 'authenticated';

	module.exports = table;

クライアント アプリケーションから、WebJob またはカスタム メカニズムを使用して、レコードを消去するためのメカニズムを確立する必要があります。

## <a name="howto-tables-seeding"></a>方法: データベースに対するデータのシード処理を実行する

新しいアプリケーションを作成する場合、データでのテーブルのシード処理が必要になることがあります。これは、以下のようにテーブル定義 JavaScript ファイル内で行うことができます。

	var azureMobileApps = require('azure-mobile-apps');

	var table = azureMobileApps.table();

	// Define the columns within the table
	table.columns = {
		"text": "string",
		"complete": "boolean"
	};
	table.seed = [
		{ text: 'Example 1', complete: false },
		{ text: 'Example 2', complete: true }
	];

	// Turn off dynamic schema
	table.dynamicSchema = false;

	// Require authentication to access the table
	table.access = 'authenticated';

	module.exports = table;

データのシード処理は、テーブルが Azure Mobile Apps SDK で作成されている場合にのみ行われることに注意してください。データベース内にテーブルが既に存在する場合、テーブルにデータは挿入されません。動的スキーマが有効な場合、スキーマはシード処理されたデータから推論されます。

initialize() メソッドを明示的に呼び出して、サービスの実行開始時にテーブルを作成することをお勧めします。

## <a name="Swagger"></a>方法: Swagger のサポートを有効にする

Azure App Service Mobile Apps には、組み込みの [Swagger] のサポートが付属しています。Swagger のサポートを有効にするには、最初に依存関係として swagger ui をインストールします。

    npm install --save swagger-ui

インストールすると、Azure Mobile Apps コンストラクターで Swagger のサポートを有効にできます。

    var mobile = azureMobileApps({ swagger: true });

開発エディションでのみ Swagger サポートを有効にするには、`NODE_ENV` アプリの設定を利用します。

    var mobile = azureMobileApps({ swagger: process.env.NODE_ENV !== 'production' });

Swagger エンドポイントは http://_yoursite_.azurewebsites.net/swagger にあります。Swagger UI には `/swagger/ui` エンドポイントからアクセスできます。アプリケーション全体で認証を必要とするように選択している場合、Swagger は / エンドポイントに対してエラーを生成することに注意してください。最良の結果を得るには、Azure App Service の [認証/承認] 設定で認証されていない要求を許可し、`table.access` プロパティを使用して認証を制御します。

また、ローカルで開発する場合にのみ、Swagger のサポートが必要な場合は、`azureMobile.js` ファイルに Swagger オプションを追加できます。

## <a name="push"></a>概要: プッシュ通知
Mobile Apps と Azure Notification Hubs を統合することで、あらゆる主要なプラットフォーム間で、数百万台のデバイスに対して、対象設定済みのプッシュ通知を送信できます。Notification Hubs を使用して、iOS デバイス、Android デバイス、および Windows デバイスにプッシュ通知を送信できます。Notification Hubs で実行可能なすべての操作については、「[Azure 通知ハブ](../notification-hubs/notification-hubs-overview.md)」をご覧ください。

## <a name="send-push"></a>方法: プッシュ通知を送信する

次のコードに、プッシュ オブジェクトを使用して、登録済みの iOS デバイスにブロードキャスト プッシュ通知を送信する方法を示します。

	// Create an APNS payload.
    var payload = '{"aps": {"alert": "This is an APNS payload."}}';

    // Only do the push if configured
    if (context.push) {
	    // Send a push notification using APNS.
        context.push.apns.send(null, payload, function (error) {
            if (error) {
                // Do something or log the error.
	        }
        });
    }

クライアントからテンプレート プッシュ登録を作成することで、代わりに、サポートされるすべてのプラットフォーム上のデバイスにテンプレート プッシュ メッセージを送信できます。次のコードに、テンプレート通知を送信する方法を示します。

	// Define the template payload.
	var payload = '{"messageParam": "This is a template payload."}';

    // Only do the push if configured
    if (context.push) {
		// Send a template notification.
        context.push.send(null, payload, function (error) {
            if (error) {
                // Do something or log the error.
            }
        });
    }


##<a name="push-user"></a>方法: タグを使用して認証済みのユーザーにプッシュ通知を送信する
認証済みのユーザーがプッシュ通知に登録すると、ユーザー ID タグが登録に自動的に追加されます。このタグを使用すると、特定のユーザーが登録したすべてのデバイスにプッシュ通知を送信できます。次のコードでは、要求を行ったユーザーの SID を取得し、そのユーザーのすべてのデバイス登録にテンプレート プッシュ通知を送信します。

    // Only do the push if configured
    if (context.push) {
		// Send a notification to the current user.
        context.push.send(context.user.id, payload, function (error) {
            if (error) {
                // Do something or log the error.
            }
        });
    }

認証済みのクライアントからプッシュ通知に登録する場合は、登録を試みる前に、認証が完了していることを確認してください。

## <a name="CustomAPI"></a>概要: カスタム API

/tables エンドポイント経由のデータ アクセス API に加え、Azure Mobile Apps ではカスタム API も提供できます。カスタム API はテーブル定義と同じような方法で定義され、認証を含む、すべての同じ機能にアクセスできます。

カスタム API で App Service 認証を使用する場合は、まず、[Azure ポータル]で App Service 認証を構成する必要があります。Azure App Service での認証構成の詳細については、以下の使用する ID プロバイダーの構成ガイドを参照してください。

- [Azure Active Directory 認証の構成方法]
- [Facebook 認証の構成方法]
- [Google 認証の構成方法]
- [Microsoft 認証の構成方法]
- [Twitter 認証の構成方法]

## <a name="howto-customapi-basic"></a>方法: 単純なカスタム API を定義する

カスタム API は、テーブル API とほぼ同じ方法で定義されます。

1. **api** ディレクトリを作成します。
2. **api** ディレクトリに API 定義 JavaScript ファイルを作成します。
3. import メソッドを使用して、**api** ディレクトリをインポートします。

前に使用した basic-app サンプルに基づくプロトタイプの api 定義を以下に示します。

	var express = require('express'),
		azureMobileApps = require('azure-mobile-apps');

	var app = express(),
		mobile = azureMobileApps();

	// Import the Custom API
	mobile.api.import('./api');

	// Add the mobile API so it is accessible as a Web API
	app.use(mobile);

	// Start listening on HTTP
	app.listen(process.env.PORT || 3000);

ここでは、_Date.now()_ メソッドを使用してサーバーの日付を返す単純な API を使用します。api/date.js ファイルを以下に示します。

	var api = {
		get: function (req, res, next) {
			var date = { currentTime: Date.now() };
			res.status(200).type('application/json').send(date);
		});
	};

	module.exports = api;

各パラメーターは、標準的な RESTful 動詞 (GET、POST、PATCH または DELETE) のいずれかです。メソッドは、必要な出力を送信する標準的な [ExpressJS ミドルウェア]関数です。

## <a name="howto-customapi-auth"></a>方法: カスタム API へのアクセスに認証を要求する

Azure Mobile Apps SDK では、テーブル エンドポイントとカスタム API の両方に対して同じ方法で認証を実装します。前のセクションで開発した API に認証を追加するには、**access** プロパティを追加します。

	var api = {
		get: function (req, res, next) {
			var date = { currentTime: Date.now() };
			res.status(200).type('application/json').send(date);
		});
	};
	// All methods must be authenticated.
	api.access = 'authenticated';

	module.exports = api;

以下のように、特定の操作で認証を指定することもできます。

	var api = {
		get: function (req, res, next) {
			var date = { currentTime: Date.now() };
			res.status(200).type('application/json').send(date);
		}
	};
	// The GET methods must be authenticated.
	api.get.access = 'authenticated';

	module.exports = api;

認証を必要とするカスタム API には、テーブル エンドポイントで使用されるものと同じトークンを使用する必要があります。

## <a name="howto-customapi-auth"></a>方法: 大きなファイルのアップロードを処理する

Azure Mobile Apps SDK では、[body-parser ミドルウェア](https://github.com/expressjs/body-parser)を使用して、送信された本文のコンテンツを受け入れ、デコードします。大きなファイルのアップロードを受け入れるように body-parser を事前構成できます。

	var express = require('express'),
        bodyParser = require('body-parser'),
		azureMobileApps = require('azure-mobile-apps');

	var app = express(),
		mobile = azureMobileApps();

    // Set up large body content handling
    app.use(bodyParser.json({ limit: '50mb' }));
    app.use(bodyParser.urlencoded({ limit: '50mb', extended: true }));

	// Import the Custom API
	mobile.api.import('./api');

	// Add the mobile API so it is accessible as a Web API
	app.use(mobile);

	// Start listening on HTTP
	app.listen(process.env.PORT || 3000);

上記の 50 MB の制限は調整できます。このファイルは、送信前に Base-64 でエンコードされるため、実際のアップロードのサイズは増加します。

## <a name="howto-customapi-sql"></a>方法: カスタム SQL ステートメントを実行する

Azure Mobile Apps SDK では、要求オブジェクトを通じてコンテキスト全体にアクセスできるため、定義済みのデータ プロバイダーに対して、パラメーター化された SQL ステートメントを簡単に実行することができます。

    var api = {
        get: function (request, response, next) {
            // Check for parameters - if not there, pass on to a later API call
            if (typeof request.params.completed === 'undefined')
                return next();

            // Define the query - anything that can be handled by the mssql
            // driver is allowed.
            var query = {
                sql: 'UPDATE TodoItem SET complete=@completed',
                parameters: [{
                    completed: request.params.completed
                }]
            };

            // Execute the query.  The context for Azure Mobile Apps is available through
            // request.azureMobile - the data object contains the configured data provider.
            request.azureMobile.data.execute(query)
            .then(function (results) {
                response.json(results);
            });
        }
    };

    api.get.access = 'authenticated';
    module.exports = api;

## <a name="Debugging"></a><a name="howto-diagnostic-logs"></a>方法: Azure Mobile Apps をデバッグ、診断、およびトラブルシューティングする

Azure App Service では、Node.js アプリケーションに関するいくつかのデバッグとトラブルシューティングの手法が提供されます。Node.js モバイル バックエンドのトラブルシューティングを開始する場合は、次の記事を参照してください。

- [Azure App Service の監視]
- [Azure App Service での診断ログの有効化]
- [Visual Studio での Azure App Service のトラブルシューティング]

Node.js アプリケーションは、広範囲の診断ログ ツールにアクセスできます。Azure Mobile Apps Node.js SDK は、内部で診断ログに [Winston] を使用します。これを自動的に有効にするには、デバッグ モードを有効にするか、[Azure ポータル]で **MS\_DebugMode** アプリ設定を true に設定します。生成されたログは、[Azure ポータル]の [診断ログ] に表示されます。

## <a name="in-portal-editing"></a><a name="work-easy-tables"></a>方法: Azure ポータルで [テーブルの簡単操作] を使用する

ポータルの [テーブルの簡単操作] を使用すると、ポータル内でテーブルをすぐに作成して操作できます。Visual Studio Online エディターを使用してテーブルの操作を編集することもできます。

バックエンド サイトの設定で **[テーブルの簡単操作]** をクリックすると、新しいテーブルを追加したり、既存のテーブルを変更または削除したりできます。さらに、テーブル内のデータを表示することもできます。

![Work with Easy Tables](./media/app-service-mobile-node-backend-how-to-use-server-sdk/mobile-apps-easy-tables.png)

テーブル用のコマンド バーには、次のコマンドが用意されています。

+ **[アクセス許可の変更]** - テーブルに対する読み取り、挿入、更新、削除操作のアクセス許可を変更します。匿名アクセスを許可するオプション、認証を要求するオプション、操作に対するすべてのアクセスを無効にするオプションがあります。これにより、table.json プロジェクト コード ファイルは変更されます。
+ **[スクリプトの編集]** - テーブルのスクリプト ファイルが Visual Studio Team Services エディターで開かれます。
+ **[スキーマの管理]** - 列の追加または削除やテーブルのインデックスの変更を実行します。
+ **[テーブルのクリア]** - 既存のテーブルですべてのデータ行が削除されるように切り捨てます。ただし、スキーマは変更されません。
+ **[行の削除]** - 個々のデータ行を削除します。
+ **[ストリーミング ログの表示]** - サイトのストリーミング ログ サービスに接続します。

##<a name="work-easy-apis"></a>方法: Azure ポータルで [API の簡単操作] を使用する

ポータルの [API の簡単操作] を使用すると、ポータル内でカスタム API をすぐに作成して操作できます。Visual Studio Online エディターを使用して API スクリプトを編集することもできます。

バックエンド サイトの設定で **[API の簡単操作]** をクリックすると、新しいカスタム API エンドポイントを追加したり、既存の API エンドポイントを変更または削除したりできます。

![Work with Easy APIs](./media/app-service-mobile-node-backend-how-to-use-server-sdk/mobile-apps-easy-apis.png)

ポータルでは、特定の HTTP アクションのアクセス許可を変更できます。また、Visual Studio Team Services エディターで API スクリプト ファイルをしたり、ストリーミング ログを表示したりできます。

##<a name="online-editor"></a>方法: Visual Studio Team Services でコードを編集する

Azure ポータルでは、ローカル コンピューターにプロジェクトをダウンロードする必要なく、Visual Studio Team Services を使用して Node.js バックエンド スクリプト ファイルを編集できます。オンライン エディターでスクリプト ファイルを編集するには、次の手順に従います。

1. モバイル アプリ バックエンドのブレードで、**[すべての設定]**、**[テーブルの簡単操作]** または **[API の簡単操作]** の順にクリックし、テーブルまたは API をクリックしてから、**[スクリプトの編集]** をクリックします。スクリプト ファイルが Visual Studio Team Services エディターで開かれます。

	![Visual Studio Team Services コード エディター](./media/app-service-mobile-node-backend-how-to-use-server-sdk/mobile-apps-visual-studio-editor.png)

2. オンライン エディターで、コード ファイルを変更します。変更内容は、入力時に自動的に保存されています。


<!-- Images -->
[0]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/npm-init.png
[1]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/vs2015-new-project.png
[2]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/vs2015-install-npm.png
[3]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/sqlexpress-config.png
[4]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/sqlexpress-authconfig.png
[5]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/sqlexpress-newuser-1.png
[6]: ./media/app-service-mobile-node-backend-how-to-use-server-sdk/dotnet-backend-create-db.png

<!-- URLs -->
[Android クライアントのクイック スタート]: app-service-mobile-android-get-started.md
[Apache Cordova クライアントのクイック スタート]: app-service-mobile-cordova-get-started.md
[iOS クライアントのクイック スタート]: app-service-mobile-ios-get-started.md
[Xamarin.iOS クライアントのクイック スタート]: app-service-mobile-xamarin-ios-get-started.md
[Xamarin.Android クライアントのクイック スタート]: app-service-mobile-xamarin-android-get-started.md
[Xamarin.Forms クライアントのクイック スタート]: app-service-mobile-xamarin-forms-get-started.md
[Windows ストア クライアントのクイック スタート]: app-service-mobile-windows-store-dotnet-get-started.md
[HTML/Javascript Client QuickStart]: app-service-html-get-started.md
[オフライン データ同期]: app-service-mobile-offline-data-sync.md
[Azure Active Directory 認証の構成方法]: app-service-mobile-how-to-configure-active-directory-authentication.md
[Facebook 認証の構成方法]: app-service-mobile-how-to-configure-facebook-authentication.md
[Google 認証の構成方法]: app-service-mobile-how-to-configure-google-authentication.md
[Microsoft 認証の構成方法]: app-service-mobile-how-to-configure-microsoft-authentication.md
[Twitter 認証の構成方法]: app-service-mobile-how-to-configure-twitter-authentication.md
[Azure App Service のデプロイ ガイド]: ../app-service-web/web-sites-deploy.md
[Azure App Service の監視]: ../app-service-web/web-sites-monitor.md
[Azure App Service での診断ログの有効化]: ../app-service-web/web-sites-enable-diagnostic-log.md
[Visual Studio での Azure App Service のトラブルシューティング]: ../app-service-web/web-sites-dotnet-troubleshoot-visual-studio.md
[Node のバージョンを指定する]: ../nodejs-specify-node-version-azure-apps.md
[Node モジュールを使用する]: ../nodejs-use-node-modules-azure-apps.md
[Create a new Azure App Service]: ../app-service-web/
[azure-mobile-apps]: https://www.npmjs.com/package/azure-mobile-apps
[Express]: http://expressjs.com/
[Swagger]: http://swagger.io/

[Azure ポータル]: https://portal.azure.com/
[OData]: http://www.odata.org
[Promise]: https://developer.mozilla.org/ja-JP/docs/Web/JavaScript/Reference/Global_Objects/Promise
[GitHub の basicapp サンプル]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/basic-app
[GitHub の todo サンプル]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/todo
[GitHub の samples ディレクトリ]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples
[static-schema sample on GitHub]: https://github.com/azure/azure-mobile-apps-node/tree/master/samples/static-schema
[QueryJS]: https://github.com/Azure/queryjs
[Node.js Tools 1.1 for Visual Studio]: https://github.com/Microsoft/nodejstools/releases/tag/v1.1-RC.2.1
[mssql Node.js パッケージ]: https://www.npmjs.com/package/mssql
[Microsoft SQL Server 2014 Express]: http://www.microsoft.com/ja-JP/server-cloud/Products/sql-server-editions/sql-server-express.aspx
[ExpressJS ミドルウェア]: http://expressjs.com/guide/using-middleware.html
[Winston]: https://github.com/winstonjs/winston

<!---HONumber=AcomDC_0316_2016-->