<properties
	pageTitle="HDInsight Linux クラスターの Hadoop で Hue を使用する | Microsoft Azure"
	description="HDInsight Linux の Hadoop クラスターに Hue をインストールし、使用する方法について説明します。"
	services="hdinsight"
	documentationCenter=""
	authors="nitinme"
	manager="jhubbard"
	editor="cgronlun"/>

<tags 
	ms.service="hdinsight" 
	ms.workload="big-data" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/13/2016" 
	ms.author="nitinme"/>

# HDInsight Hadoop クラスターに Hue をインストールして使用する

HDInsight Linux クラスターに Hue をインストールし、トンネリングを利用して Hue に要求を送信する方法について学習します。

## Hue とは

Hue は Hadoop クラスターとの情報のやりとりに使用される一連の Web アプリケーションです。Hue を利用して Hadoop クラスターに関連付けられているストレージを閲覧したり (HDInsight クラスターの場合、WASB)、Hive ジョブや Pig スクリプトを実行したりできます。HDInsight Hadoop クラスターにインストールした Hue では次のコンポーネントが利用可能です。

* Beeswax Hive エディター
* Pig
* メタストア マネージャー
* Oozie
* FileBrowser (WASB の既定のコンテナーと通信)
* ジョブ ブラウザー

> [AZURE.WARNING] HDInsight クラスターに用意されているコンポーネントは全面的にサポートされており、これらのコンポーネントに関連する問題の分離と解決については、Microsoft サポートが支援します。
>
> カスタム コンポーネントについては、問題のトラブルシューティングを進めるための支援として、商業的に妥当な範囲のサポートを受けることができます。これにより問題が解決する場合もあれば、オープン ソース テクノロジに関して、深い専門知識が入手できる場所への参加をお願いすることになる場合もあります。たとえば、[HDInsight についての MSDN フォーラム](https://social.msdn.microsoft.com/Forums/azure/ja-JP/home?forum=hdinsight)や [http://stackoverflow.com](http://stackoverflow.com) などの数多くのコミュニティ サイトを利用できます。また、Apache プロジェクトには、[http://apache.org](http://apache.org) に [Hadoop](http://hadoop.apache.org/) などのプロジェクト サイトもあります。

## スクリプト アクションを使用した Hue のインストール

次のスクリプト アクションを使用して、Hue を Linux ベースの HDInsight クラスターにインストールできます。https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh
    
このセクションでは、Azure ポータルを使用してクラスターをプロビジョニングする際にこのスクリプトを使用する方法について説明します。

> [AZURE.NOTE] スクリプト アクションは、Azure PowerShell、Azure CLI、HDInsight .NET SDK、または Azure Resource Manager のテンプレートを使用して適用することもできます。既に実行しているクラスターにスクリプト アクションを適用することもできます。詳細については、[スクリプト アクションを使用した HDInsight クラスターのカスタマイズ](hdinsight-hadoop-customize-cluster-linux.md)に関する記事を参照してください。

1. 「[Linux の HDInsight クラスターのプロビジョニング](hdinsight-hadoop-provision-linux-clusters.md#portal)」に記載されている手順を使用してクラスターのプロビジョニングを開始します。ただし、プロビジョニングを完了しないでください。

	> [AZURE.NOTE] HDInsight クラスターに Hue をインストールするには、A4 (8 コア、14 GB メモリ) 以上のヘッドノード サイズが推奨されます。

2. **[オプションの構成]** ブレードで **[スクリプト アクション]** を選択し、以下のように情報を指定します。

	![Hue のスクリプト アクション パラメーターを指定します。](./media/hdinsight-hadoop-hue-linux/hue_script_action.png "Hue のスクリプト アクション パラメーターを指定します。")

	* __[名前]__: スクリプト アクションの表示名を入力します。
	* __[スクリプト URI]__: https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh
	* __[ヘッド]__: このオプションをオンにします。
	* __[ワーカー]__: 空白のままにします。
	* __[ZOOKEEPER]__: 空白のままにします。
	* __[パラメーター]__: 空白のままにします。

3. **[スクリプト アクション]** の下部で、**[選択]** を使用して構成を保存します。最後に、**[オプションの構成]** ブレードの下部にある **[選択]** ボタンを使用し、オプションの構成情報を保存します。

4. 「[Linux の HDInsight クラスターのプロビジョニング](hdinsight-hadoop-provision-linux-clusters.md#portal)」の説明に従って、クラスターのプロビジョニングを続行します。

## HDInsight クラスターで Hue を使用する

SSH トンネリングは、実行後、クラスターの Hue にアクセスする唯一の方法です。SSH によるトンネリングでは、Hue が実行されているクラスターのヘッドノードにトラフィックが直接進むことができます。クラスターがプロビジョニングを完了したら、次の手順で HDInsight Linux クラスターで Hue を使用します。

1. 「[SSH トンネリングを使用して Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie、およびその他の Web UI にアクセスする](hdinsight-linux-ambari-ssh-tunnel.md)」の情報を使用して、クライアント システムから HDInsight クラスターへの SSH トンネルを作成し、この SSH トンネルをプロキシとして使用するように Web ブラウザーを構成します。

2. SSH トンネルを作成し、これをトラフィックのプロキシとして使用するようにブラウザーを構成したら、プライマリ ヘッド ノードのホスト名を見つける必要があります。そのためには、ポート 22 で SSH を使用してクラスターに接続します。たとえば、`ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net` を使用して、__USERNAME__ に SSH ユーザー名を、__CLUSTERNAME__ にクラスターの名前を指定します。

    SSH の使用方法の詳細については、次のドキュメントを参照してください。

    * [Linux、Unix、または Mac OS X クライアントから Linux ベースの HDInsight で SSH キーを使用する](hdinsight-hadoop-linux-use-ssh-unix.md)
    * [Windows クライアントから Linux ベースの HDInsight で SSH を使用する](hdinsight-hadoop-linux-use-ssh-windows.md)

3. 接続後に、次のコマンドを使用してプライマリ ヘッドノードの完全修飾ドメイン名を取得します。

        hostname -f

    次のような名前が返されます。

        hn0-myhdi-nfebtpfdv1nubcidphpap2eq2b.ex.internal.cloudapp.net
    
    これは、Hue の Web サイトが配置されているプライマリ ヘッドノードのホスト名です。

2. ブラウザーを使用して、Hue ポータル (http://HOSTNAME:8888) を開きます。HOSTNAME は前の手順で取得した名前で置き換えてください。

    > [AZURE.NOTE] 初めてログインするときには、Hue ポータルにログインするためのアカウントを作成するよう求められます。ここで指定した資格情報はポータルに制限され、クラスターのプロビジョニング時に指定した管理者または SSH ユーザーの資格情報には関連しません。

	![Hue ポータルにログインする](./media/hdinsight-hadoop-hue-linux/HDI.Hue.Portal.Login.png "Hue ポータルの資格情報を指定する")

### Hive クエリを実行する

1. Hue ポータルから、**[クエリ エディター]** をクリックし、**[Hive]** をクリックして Hive エディターを開きます。

	![Hive を使用する](./media/hdinsight-hadoop-hue-linux/HDI.Hue.Portal.Hive.png "Hive を使用する")

2. **[支援]** タブの **[データベース]** に **hivesampletable** が表示されるはずです。これは HDInsight のすべての Hadoop クラスターに含まれるサンプル テーブルです。右ペインでサンプル クエリを入力します。スクリーン キャプチャに示すように、下のペインの **[結果]** タブに出力が表示されます。

	![Hive クエリを実行する](./media/hdinsight-hadoop-hue-linux/HDI.Hue.Portal.Hive.Query.png "Hive クエリを実行する")

	**[グラフ]** タブを使用し、結果を視覚的に表示することもできます。

### クラスター ストレージを参照する

1. Hue ポータルから、メニュー バーの右上隅にある **[ファイル ブラウザー]** をクリックします。

2. 既定では、ファイル ブラウザーは **/user/myuser** ディレクトリで起動します。パスのユーザー ディレクトリの直前の斜線をクリックし、クラスターに関連付けられている Azure ストレージ コンテナーのルートに移動します。

	![ファイル ブラウザーを使用する](./media/hdinsight-hadoop-hue-linux/HDI.Hue.Portal.File.Browser.png "ファイル ブラウザーを使用する")

3. ファイルまたはフォルダーを右クリックし、利用可能な操作を表示します。現在のディレクトリにファイルをアップロードするには、右隅にある **[アップロード]** ボタンを使用します。新しいファイルやディレクトリを作成するには、**[新規]** ボタンを使用します。

> [AZURE.NOTE] Hue ファイル ブラウザーは HDInsight クラスターに関連付けられている既定のコンテナーのコンテンツのみを表示できます。追加のストレージ アカウント/コンテナーがクラスターに関連付けられている場合、ファイル ブラウザーではアクセスできません。ただし、Hive ジョブの場合、クラスターに関連付けられている追加のコンテナーに常にアクセスできます。たとえば、Hive エディターに `dfs -ls wasbs://newcontainer@mystore.blob.core.windows.net` コマンドを入力すると、追加コンテナーのコンテンツを表示できます。このコマンドで、**newcontainer** はクラスターに関連付けられている既定のコンテナーではありません。

## 重要な考慮事項

1. Hue のインストールに使用されるスクリプトは、クラスターのプライマリ ヘッドノードにのみ Hue をインストールします。

2. インストール中、複数の Hadoop サービス (HDFS、YARN、MR2、Oozie) が再起動し、構成を更新します。スクリプトが Hue のインストールを完了した後、他の Hadoop サービスが起動するまで少し時間がかかる場合があります。最初はこれが Hue のパフォーマンスに影響を与えることがあります。すべてのサービスが起動すると、Hue が完全に機能します。

3.	Hue は Hive の現在の既定である Tez ジョブを認識しません。Hive 実行エンジンとして MapReduce を使用する場合、スクリプトで次のコマンドを使用するようにスクリプトを更新します。

		set hive.execution.engine=mr;

4.	Linux クラスターの場合、サービスをプライマリ ヘッドノードで実行し、Resource Manager をセカンダリ ヘッドノードで実行するシナリオがありえます。そのようなシナリオの場合、Hue を利用してクラスターで「実行中」のジョブの詳細を表示するとき、エラーが発生する可能性があります (下の画像を参照)。ただし、ジョブが完了したときにジョブの詳細を表示できます。

	![Hue ポータル エラー](./media/hdinsight-hadoop-hue-linux/HDI.Hue.Portal.Error.png "Hue ポータル エラー")

	これは既知の問題によるものです。この問題を回避するには、アクティブな Resource Manager もプライマリ ヘッドノードで実行されるように Ambari を変更します。

5.	HDInsight クラスターが `wasbs://` で Azure Storage を使用するとき、Hue は WebHDFS を認識します。そのため、スクリプト アクションで使用されるカスタム スクリプトは WebWasb をインストールします。これは WASB と通信するための WebHDFS 互換サービスです。そのため、Hue ポータルに HDFS と表示されている場合でも (**ファイル ブラウザー**の上にマウスを移動したときなど)、WASB として解釈するべきです。


## 次のステップ

- [HDInsight クラスターでの Giraph のインストール](hdinsight-hadoop-giraph-install-linux.md): クラスターのカスタマイズを使用して、HDInsight Hadoop クラスターに Giraph をインストールします。Giraph は、Hadoop でグラフの処理を実行するために使用でき、Azure HDInsight で使用できます。

- [HDInsight クラスターでの Solr のインストール](hdinsight-hadoop-solr-install-linux.md):クラスターのカスタマイズを使用して、HDInsight Hadoop クラスターに Solr をインストールします。Solr は、格納されたデータに対して強力な検索操作を実行することができます。

- [HDInsight クラスターに R をインストールする](hdinsight-hadoop-r-scripts-linux.md)。クラスターのカスタマイズを使用して、HDInsight Hadoop クラスターに R をインストールします。R は、統計計算用のオープン ソースの言語および環境です。R は、数百の組み込み統計関数と、関数型プログラミングとオブジェクト指向のプログラミングの特徴を結合した独自のプログラミング言語を提供します。また、広範なグラフィカル機能も提供します。

[powershell-install-configure]: install-configure-powershell-linux.md
[hdinsight-provision]: hdinsight-provision-clusters-linux.md
[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster-linux.md

<!---HONumber=AcomDC_0921_2016-->