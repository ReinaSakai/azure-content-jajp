<properties
	pageTitle="Azure App Service での Node.js Web アプリの使用"
	description="Node.js アプリケーションを Azure App Service の Web アプリにデプロイする方法を説明します。"
	services="app-service\web"
	documentationCenter="nodejs"
	authors="cephalin"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="nodejs"
	ms.topic="get-started-article"
	ms.date="07/01/2016"
	ms.author="cephalin"/>

# Azure App Service での Node.js Web アプリの使用

[AZURE.INCLUDE [タブ](../../includes/app-service-web-get-started-nav-tabs.md)]

このチュートリアルでは、簡単な [Node.js] アプリケーションを作成し、cmd.exe や bash などのコマンド ライン環境を使用して [Azure App Service] にデプロイする方法について説明します。このチュートリアルの手順は、Node.js を実行できる任意のオペレーティング システムで使用できます。

<a name="prereq"></a>
## 前提条件

- [Node.JS]
- [Bower]
- [Yeoman]
- [Git]
- [Azure CLI]
- Microsoft Azure アカウント。アカウントを持っていない場合は、[無料試用版にサインアップする]か、または [Visual Studio サブスクライバー特典を有効にする]ことができます。

## 単純な Node.js Web アプリの作成とデプロイ

1. 任意のコマンド ライン ターミナルを開いて、[Express generator for Yeoman] をインストールします。

        npm install -g generator-express

2. `CD` で作業ディレクトリに移動し、次の構文を使用して、express アプリを生成します。

        yo express
        
    プロンプトで次のオプションを選択します。

    `? Would you like to create a new directory for your project?` **Yes** `? Enter directory name` **{appname}** `? Select a version to install:` **MVC** `? Select a view engine to use:` **Jade** `? Select a css preprocessor to use (Sass Requires Ruby):` **None** `? Select a database to use:` **None** `? Select a build tool to use:` **Grunt**

3. `CD` で新しいアプリのルート ディレクトリに移動し、アプリを起動して開発環境で動作することを確認します。

        npm start

    ブラウザーで <http://localhost:3000> に移動し、Express のホーム ページが表示されることを確認します。アプリが適切に実行されることを確認したら、`Ctrl-C` を使用してアプリを停止します。
    
1. ASM モードに切り替え、Azure にログインします ([Azure CLI](#prereq) が必要です)。

        azure config mode asm
        azure login

    画面の指示に従い、Azure サブスクリプションのある Microsoft アカウントを使用してブラウザーでログイン操作を進めます。

2. まだアプリのルート ディレクトリにいることを確認し、次のコマンドを使用して、一意のアプリ名で Azure に App Service アプリ リソースを作成します。次に例を示します。http://{appname}.azurewebsites.net

        azure site create --git {appname}

    プロンプトに従って、デプロイ先の Azure リージョンを選択します。まだ Azure サブスクリプションに Git/FTP デプロイの資格情報を設定していない場合は、資格情報を作成するように求められます。

3. アプリケーションのルートから ./config/config.js ファイルを開き、運用環境のポートを `process.env.port` に変更します。`config` オブジェクトの `production` プロパティは次の例のようになります。

        production: {
            root: rootPath,
            app: {
                name: 'express1'
            },
            port: process.env.port,
        }

    これにより Node.js アプリは、iisnode がリッスンする既定のポートで Web 要求に応答できるようになります。
    
4. ./package.json を開き、[必要な Node.js バージョンを指定する](#version)ために `engines` プロパティを追加します。

        "engines": {
            "node": "6.6.0"
        }, 

4. 変更内容を保存し、git を使用してアプリを Azure にデプロイします。

        git add .
        git commit -m "{your commit message}"
        git push azure master

    Express ジェネレーターによって .gitignore ファイルがあらかじめ提供されているため、`git push` が node\_modules/ ディレクトリのアップロードを試みることによる帯域幅の消費は発生しません。

5. 最後に、ブラウザーでライブ Azure アプリを起動します。

        azure site browse

    これで対象の Node.js Web アプリが Azure App Service でライブ実行されていることを確認できます。
    
    ![Example of browsing to the deployed application.][deployed-express-app]

## Node.js Web アプリの更新

App Service で実行されている Node.js Web アプリを更新するには、最初にデプロイしたときと同様に、`git add`、`git commit`、`git push` を実行します。
     
## App Service における Node.js アプリのデプロイ

Azure App Service では、Node.js アプリを実行するために [iisnode] が使用されます。Node.js アプリの開発とデプロイは、Azure CLI と Kudu エンジン (Git デプロイ) を組み合わせることでコマンド ラインから効率的に行うことができます。

- `azure site create --git` は、server.js または app.js の一般的な Node.js パターンを認識し、iisnode.yml をルート ディレクトリに作成します。このファイルを使用して iisnode をカスタマイズできます。
- `git push azure master` では、次のデプロイ タスクが Kudu によって自動化されます。

    - package.json がリポジトリのルートに存在する場合、`npm install --production` を実行します。
    - package.json 内の起動スクリプト (server.js、app.js など) を参照する iisnode 用の Web.config を生成します。
    - Node-Inspector でアプリをデバッグできるよう Web.config をカスタマイズします。
    
## Node.js フレームワークの使用

[Sails.js][SAILSJS] や [MEAN.js][MEANJS] などの広く使われている Node.js フレームワークを使用して開発したアプリは、App Service にデプロイすることができます。広く使われている Node.js フレームワークには、それぞれ独自のくせがあり、パッケージの依存関係も絶えず更新されます。しかし App Service では、stdout と stderr のログが利用できるようになっているため、アプリで起こっていることを正確に把握し、それに応じた変更を加えることができます。詳細については、「[iisnode から stdout ログと stderr ログを取得する](#iisnodelog)」を参照してください。

以下のチュートリアルには、App Service における特定のフレームワークの使用方法が解説されています。

- [Deploy a Sails.js web app to Azure App Service (Sails.js Web アプリを Azure App Service にデプロイする)]
- [Azure App Service での Socket.IO を使用する Node.js チャット アプリケーションの構築]
- [Azure App Service Web Apps で io.js を使用する方法]

<a name="version"></a>
## 特定の Node.js エンジンの使用

一般的なワークフローでは、通常 package.json で行うように、特定の Node.js エンジンの使用を App Service に指示します。次に例を示します。

    "engines": {
        "node": "6.6.0"
    }, 

Kudu デプロイ エンジンは、使用する Node.js エンジンを次の順序で決定します。

- まず iisnode.yml を参照し、`nodeProcessCommandLine` が指定されているかどうかを確認します。指定されている場合はそれを使用します。
- 次に package.json を参照し、`engines` オブジェクトに `"node": "..."` が指定されているかどうかを確認します。指定されている場合はそれを使用します。
- 特に指定されていない場合、既定の Node.js バージョンを選択します。

>[AZURE.NOTE] 必要な Node.js エンジンを明示的に定義することをお勧めします。既定の Node.js バージョンは変更できます。既定の Node.js バージョンがアプリに適していないことが理由で Azure Web アプリでエラーが発生する場合があります。

<a name="iisnodelog"></a>
## iisnode から stdout ログと stderr ログを取得する

iisnode ログを読み取るには、次の手順を実行します。

> [AZURE.NOTE] 次の手順を完了した後も、エラーが発生するまでログ ファイルが生成されない場合があります。

1. Azure CLI に用意されている iisnode.yml ファイルを開きます。

2. 次の 2 つのパラメーターを設定します。

        loggingEnabled: true
        logDirectory: iisnode
    
    この 2 つのパラメーターを組み合わせることで、stdout と stderror の出力先が D:\\home\\site\\wwwroot**iisnode** ディレクトリとなるよう、App Service 内の iisnode に伝えられます。

3. 次の Git コマンドで変更内容を保存し、Azure にプッシュします。

        git add .
        git commit -m "{your commit message}"
        git push azure master
   
    これで iisnode の構成は完了です。以下、これらのログにアクセスする方法を説明します。
     
4. ブラウザーでアプリの Kudu デバッグ コンソールにアクセスします。URL は次のとおりです。

        https://{appname}.scm.azurewebsites.net/DebugConsole 

    この URL は、Web アプリの URL とは異なるもので、DNS 名に "*.scm.*" が追加されています。この追加部分を URL から省くと、404 エラーが表示されます。

5. D:\\home\\site\\wwwroot\\iisnode に移動します。

    ![Navigating to the location of the iisnode log files.][iislog-kudu-console-find]

6. 確認するログの **[編集]** アイコンをクリックします。必要に応じて **[ダウンロード]** または **[削除]** をクリックすることもできます。

    ![Opening an iisnode log file.][iislog-kudu-console-open]

    これでログを参照できるようになりました。ログを基に App Service のデプロイをデバッグすることができます。
    
    ![Examining an iisnode log file.][iislog-kudu-console-read]

## Node-Inspector によるアプリのデバッグ

Node.js アプリのデバッグに Node-Inspector を使用している場合、実稼働の App Service アプリに Node-Inspector を使用できます。Node-Inspector は、App Service の iisnode のインストールでプレインストールされます。また、デプロイに Git を使用している場合、Kudu によって自動生成された Web.config には既に、Node-Inspector を有効にするうえで必要な構成がすべて含まれています。

Node-Inspector を有効にするには、次の手順を実行します。

1. リポジトリのルートにある iisnode.yml を開いて、次のパラメーターを指定します。

        debuggingEnabled: true
        debuggerExtensionDll: iisnode-inspector.dll

3. 次の Git コマンドで変更内容を保存し、Azure にプッシュします。

        git add .
        git commit -m "{your commit message}"
        git push azure master
   
4. 後は、package.json 内の起動スクリプトで指定されたアプリの起動ファイルに移動するだけです。URL の最後に /debug を付けてください。たとえば、次のように入力します。

        http://{appname}.azurewebsites.net/server.js/debug
    
    または、
    
        http://{appname}.azurewebsites.net/app.js/debug

## その他のリソース

- [Azure アプリケーションでの Node.js のバージョンの指定](../nodejs-specify-node-version-azure-apps.md)
- [Azure での Node.js アプリケーションのベスト プラクティスとトラブルシューティング ガイド](app-service-web-nodejs-best-practices-and-troubleshoot-guide.md)
- [Azure App Service で Node.js Web アプリをデバッグする方法](web-sites-nodejs-debug.md)
- [Azure アプリケーションでの Node.js モジュールの使用](../nodejs-use-node-modules-azure-apps.md)
- [Azure App Service Web Apps: Node.js](http://blogs.msdn.com/b/silverlining/archive/2012/06/14/windows-azure-websites-node-js.aspx)
- [Node.js デベロッパー センター](/develop/nodejs/)
- [Azure App Service での Web アプリの使用](app-service-web-get-started.md)
- [優れた Kudu デバッグ コンソールの詳細]

<!-- URL List -->

[Azure CLI]: ../xplat-cli-install.md
[Azure App Service]: ../app-service/app-service-value-prop-what-is.md
[Visual Studio サブスクライバー特典を有効にする]: http://go.microsoft.com/fwlink/?LinkId=623901
[Bower]: http://bower.io/
[Azure App Service での Socket.IO を使用する Node.js チャット アプリケーションの構築]: ./web-sites-nodejs-chat-app-socketio.md
[Deploy a Sails.js web app to Azure App Service (Sails.js Web アプリを Azure App Service にデプロイする)]: ./app-service-web-nodejs-sails.md
[優れた Kudu デバッグ コンソールの詳細]: /documentation/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[Express generator for Yeoman]: https://github.com/petecoop/generator-express
[Git]: http://www.git-scm.com/downloads
[Azure App Service Web Apps で io.js を使用する方法]: ./web-sites-nodejs-iojs.md
[iisnode]: https://github.com/tjanczuk/iisnode/wiki
[MEANJS]: http://meanjs.org/
[Node.js]: http://nodejs.org
[SAILSJS]: http://sailsjs.org/
[無料試用版にサインアップする]: http://go.microsoft.com/fwlink/?LinkId=623901
[web app]: ./app-service-web-overview.md
[Yeoman]: http://yeoman.io/

<!-- IMG List -->

[deployed-express-app]: ./media/app-service-web-nodejs-get-started/deployed-express-app.png
[iislog-kudu-console-find]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-navigate.png
[iislog-kudu-console-open]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-open.png
[iislog-kudu-console-read]: ./media/app-service-web-nodejs-get-started/iislog-kudu-console-read.png

<!---HONumber=AcomDC_0928_2016-->