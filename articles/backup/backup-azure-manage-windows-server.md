<properties
	pageTitle="Azure Recovery Services コンテナーとサーバーの管理 | Microsoft Azure"
	description="このチュートリアルでは、Azure Recovery Services コンテナーとサーバーを管理する方法について説明します。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor="tysonn"/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/19/2016"
	ms.author="jimpark; markgal"/>


# Windows コンピューター用 Azure Recovery Services コンテナーとサーバーの監視と管理

> [AZURE.SELECTOR]
- [リソース マネージャー](backup-azure-manage-windows-server.md)
- [クラシック](backup-azure-manage-windows-server-classic.md)

この記事では、Microsoft Azure 管理ポータルと Microsoft Azure Backup エージェントで使用できるバックアップ管理タスクの概要を説明します。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)] クラシック デプロイメント モデル。

## 管理ポータルのタスク

### Recovery Services コンテナーへのアクセス

1. Azure サブスクリプションを使用して、[Azure ポータル](https://portal.azure.com/)にサインインします。

2. ハブ メニューで **[参照]** をクリックし、リソースの一覧で「**Recovery Services**」と入力します。入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。**[Recovery Services コンテナー]** をクリックします。

    ![Create Recovery Services Vault step 1](./media/backup-azure-manage-windows-server/browse-to-rs-vaults.png) <br/>

2. 表示するコンテナーの名前を一覧から選択して、Recovery Services コンテナーのダッシュボード ブレードを開きます。

    ![Recovery Services コンテナーのダッシュボード](./media/backup-azure-manage-windows-server/rs-vault-dashboard.png) <br/>

## ジョブとアラートの監視
ジョブとアラートは、Recovery Services コンテナーのダッシュボードから監視できます。このダッシュボードには、次の情報が表示されます。

- バックアップ アラートの詳細
- ファイルとフォルダー、およびクラウドで保護されている Azure 仮想マシン
- Azure で使用されているストレージの合計
- バックアップ ジョブの状態

![バックアップ ダッシュボードのタスク](./media/backup-azure-manage-windows-server/dashboard-tiles.png)

それぞれのタイルで情報をクリックすると関連するブレードが開き、ここで関連タスクを管理できます。

ダッシュボードの上部では次の操作を行うことができます。

- 設定 - 使用可能なバックアップ タスクにアクセスできます。
- バックアップ - 新しいファイルおよびフォルダー (または Azure VM) を Recovery Services コンテナーにバックアップできます。
- 削除 - 不要になった Recovery Services コンテナーを削除して、ストレージ領域を解放できます。[削除] は、保護されているサーバーすべてがコンテナーから削除された後にのみ有効になります。

![バックアップ ダッシュボードのタスク](./media/backup-azure-manage-windows-server/dashboard-tasks.png)
## Azure Backup エージェントを使用したバックアップに関するアラート:
| アラート レベル | 送信されるアラート |
| ------------- | ------------- |
| 重大 | バックアップの失敗、回復エラー |
| 警告 | 警告を伴ってバックアップが完了した (破損が原因で 100 個未満のファイルがバックアップされず、膨大な数のファイルが正常にバックアップされたとき) |
| 情報 | なし |
## バックアップ アラートの管理
**[バックアップ アラート]** タイルをクリックして **[バックアップ アラート]** ブレードを開き、アラートを管理します。

![バックアップ アラート](./media/backup-azure-manage-windows-server/manage-backup-alerts.png)

[バックアップ アラート] タイルには、次のアラートの数が表示されます。

- 過去 24 時間の解決されていない重大なアラート
- 過去 24 時間の解決されていない警告アラート

各リンクをクリックすると、**[バックアップ アラート]** ブレードが開き、このアラート (重大または警告) でフィルター処理されたビューが表示されます。

[バックアップ アラート] ブレードでは、次の操作を行います。

- アラートに含める情報を選択します。

    ![列の選択](./media/backup-azure-manage-windows-server/choose-alerts-colunms.png)

- 重大度、状態、開始/終了時間のアラートをフィルター処理します。

    ![アラートのフィルター処理](./media/backup-azure-manage-windows-server/filter-alerts.png)

- 重大度、頻度、および受信者の通知を構成し、アラートをオンまたはオフにします。

    ![アラートのフィルター処理](./media/backup-azure-manage-windows-server/configure-notifications.png)

**[通知]** 頻度に **[アラートごと]** が選択されている場合、グループ化や電子メールの削減は行われません。アラートごとに通知されます。これは既定の設定で、解決策が示された電子メールが直ちに送信されます。

**[通知]** 頻度に **[1 時間ごとのダイジェスト]** が選択されている場合、過去 1 時間に新しいアラートが生成され、未解決であることが、電子メールでユーザーに通知されます。解決策が示された電子メールは、その時間の終了時に送信されます。

次の重大度レベルに対してアラートを送信できます。

- 重大
- 警告
- 情報

アラートを無効にするには、[ジョブの詳細] ブレードの **[非アクティブ化]** ボタンを使用します。[非アクティブ化] をクリックした場合は、解決策に関するメモを提供できます。

アラートの一部として表示する列を選択するには、**[列の選択]** ボタンを使用します。

>[AZURE.NOTE] バックアップ アラートを管理するには、**[設定]** ブレードで、**[Monitoring and Reports (監視およびレポート)]、[アラートとイベント]、[バックアップ アラート]** の順に選択し、**[フィルター]** または **[通知の構成]** をクリックします。

## バックアップ項目の管理
オンプレミスのバックアップを管理ポータルで管理できるようになりました。ダッシュボードの [バックアップ] セクションの **[バックアップ項目]** タイルには、コンテナーに対して保護されているバックアップ項目の数が表示されます。

[バックアップ項目] タイルで **[ファイル フォルダー]** をクリックします。

![Backup items tile](./media/backup-azure-manage-windows-server/backup-items-tile.png)

[ファイル フォルダー] でフィルター処理された [バックアップ項目] ブレードが開き、特定のバックアップ項目が表示されます。

![バックアップ項目](./media/backup-azure-manage-windows-server/backup-item-list.png)

一覧で特定のバックアップ項目をクリックすると、その項目の重要な詳細情報が表示されます。

>[AZURE.NOTE] ファイルとフォルダーを管理するには、**[設定]** ブレードで、**[保護された広告]、[バックアップ項目]** の順に選択し、ドロップダウン メニューから **[ファイル フォルダー]** を選択します。

![設定のバックアップ項目](./media/backup-azure-manage-windows-server/backup-files-and-folders.png)

## バックアップ ジョブの管理
ダッシュボードには、オンプレミス (オンプレミス サーバーが Azure にバックアップされている場合) と Azure バックアップの両方が表示されます。

ダッシュボードの [バックアップ] セクションの [バックアップ ジョブ] タイルには、次のジョブの数が示されます。

- 進行中
- 過去 24 時間に失敗。

バックアップ ジョブを管理するには、**[バックアップ ジョブ]** タイルをクリックして、[バックアップ ジョブ] ブレードを開きます。

![設定のバックアップ項目](./media/backup-azure-manage-windows-server/backup-jobs.png)

[バックアップ ジョブ] ブレードで使用できる情報を変更するには、ページ上部にある **[列の選択]** ボタンを使用します。

**[フィルター]** ボタンを使用して、ファイルとフォルダーまたは Azure 仮想マシンのバックアップを選択します。

バックアップされたファイルとフォルダーが表示されない場合は、ページ上部にある **[フィルター]** ボタンをクリックし、[アイテムの種類] メニューから **[ファイルとフォルダー]** を選択します。

>[AZURE.NOTE] バックアップ ジョブを管理するには、**[設定]** ブレードで、**[Monitoring and Reports (監視およびレポート)]、[ジョブ]、[バックアップ ジョブ]** の順に選択し、ドロップダウン メニューから **[ファイル フォルダー]** を選択します。

## バックアップの利用状況の監視
ダッシュボードの [バックアップ] セクションでは、[バックアップの利用状況] タイルに Azure で使用されているストレージが表示されます。次のストレージの利用状況を確認できます。
- コンテナーに関連付けられているクラウド LRS ストレージの利用状況
- コンテナーに関連付けられているクラウド GRS ストレージの利用状況

## 実稼働サーバー
実稼働サーバーを管理するには、**[設定]** をクリックします。[管理] で、**[バックアップ インフラストラクチャ]、[実稼働サーバー]** の順にクリックします。

[実稼働サーバー] ブレードには、利用可能な実稼働サーバーの一覧が表示されます。サーバーの詳細を表示するには、一覧で目的のサーバーをクリックします。

![保護された項目](./media/backup-azure-manage-windows-server/production-server-list.png)

## Microsoft Azure Backup エージェントのタスク

## Backup エージェントを開く

**Microsoft Azure Backup エージェント**を開きます (コンピューターで "*Microsoft Azure Backup*" を検索すると見つかります)。

![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/snap-in-search.png)

Backup エージェント コンソールの右側にある **[アクション]** から、次の管理タスクを実行します。

- サーバーの登録
- Schedule Backup
- 今すぐバックアップ
- プロパティの変更

![Microsoft Azure Backup エージェント コンソールのアクション](./media/backup-azure-manage-windows-server/console-actions.png)

>[AZURE.NOTE] **データを回復する**方法については、[Windows サーバーまたは Windows クライアント コンピューターへのファイルの復元](backup-azure-restore-windows-server.md)に関するページをご覧ください。

## 既存のバックアップの変更

1. Microsoft Azure Backup エージェントで、**[バックアップのスケジュール]** をクリックします。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/schedule-backup.png)

2. **バックアップのスケジュール ウィザード**で、**[バックアップ項目または時刻を変更する]** を選択した状態のまま、**[次へ]** をクリックします。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/modify-or-stop-a-scheduled-backup.png)

3. 項目を追加または変更する場合は、**[バックアップする項目の選択]** 画面で **[項目の追加]** をクリックします。

    ウィザードのこのページで、**[除外の設定]** を設定することもできます。ファイルまたはファイルの種類を除外する場合は、[除外の設定](#exclusion-settings)を追加する手順をご覧ください。

4. バックアップするファイルとフォルダーを選択し、**[OK]** をクリックします。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/add-items-modify.png)

5. **バックアップのスケジュール**を指定し、**[次へ]** をクリックします。

    毎日 (1 日に最大 3 回) または毎週のバックアップをスケジュールすることができます。

    ![Windows Server のバックアップ項目](./media/backup-azure-manage-windows-server/specify-backup-schedule-modify-close.png)

    >[AZURE.NOTE] バックアップ スケジュールの指定の詳細については、こちらの[記事](backup-azure-backup-cloud-as-tape.md)をご覧ください。

6. バックアップ コピーの **[アイテム保持ポリシー]** を選択し、**[次へ]** をクリックします。

    ![Windows Server のバックアップ項目](./media/backup-azure-manage-windows-server/select-retention-policy-modify.png)

7. **[確認]** 画面で情報を確認し、**[完了]** をクリックします。

8. ウィザードで**バックアップ スケジュール**の作成が完了したら、**[閉じる]** をクリックします。

    保護の変更後、**[ジョブ]** タブに移動し、バックアップ ジョブに変更が反映されていることを確認することで、バックアップが正しくトリガーされていることを確認できます。

## ネットワーク調整の有効化  
Azure Backup エージェントには、データ転送時のネットワーク帯域幅の使用方法を制御できる [調整] タブがあります。データのバックアップを業務時間中に行う必要があり、バックアップ処理が他のインターネット トラフィックに影響を与えないようにする場合などに、この制御が便利です。データ転送のスロットルはバックアップと復元のアクティビティに適用されます。

調整を有効にするには、次の手順を実行します。

1. **Backup エージェント**で、**[プロパティの変更]** をクリックします。

2. **[バックアップ操作用のインターネット使用帯域幅の調整を有効にする]** チェックタイルをオンにします。

    ![Network throttling](./media/backup-azure-manage-windows-server/throttling-dialog.png)

3. スロットルを有効にしたら、**[作業時間]** と **[作業時間外]** で、バックアップ データの転送時に使用できる帯域幅を指定します。

    帯域幅の値は、512 キロバイト/秒 (Kbps) から始まり、最大で 1023 メガバイト/秒 (Mbps) まで指定できます。また、**[作業時間]** の開始および終了時刻や、作業日と見なされる曜日も指定できます。指定した作業時間以外は、作業時間外と見なされます。

4. **[OK]** をクリックします。

## 除外の設定の管理

1. **Microsoft Azure Backup エージェント**を開きます (コンピューターで "*Microsoft Azure Backup*" を検索すると見つかります)。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/snap-in-search.png)

2. Microsoft Azure Backup エージェントで、**[バックアップのスケジュール]** をクリックします。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/schedule-backup.png)

3. バックアップのスケジュール ウィザードで、**[バックアップ項目または時刻を変更する]** を選択した状態のまま、**[次へ]** をクリックします。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/modify-or-stop-a-scheduled-backup.png)

4. **[除外の設定]** をクリックします。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/exclusion-settings.png)

5. **[除外の追加]** をクリックします。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/add-exclusion.png)

6. 場所を選択し、**[OK]** をクリックします。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/exclusion-location.png)

7. **[ファイルの種類]** フィールドにファイル拡張子を追加します。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/exclude-file-type.png)

    .mp3 拡張子の追加

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/exclude-mp3.png)

    別の拡張子を追加するには、**[除外の追加]** をクリックし、別の種類のファイル拡張子を入力します (.jpeg 拡張子を追加)。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/exclude-jpg.png)

8. すべての拡張子を追加したら、**[OK]** をクリックします。

9. **[確認] ページ**が表示されるまで **[次へ]** をクリックしてバックアップのスケジュール ウィザードを続行し、**[完了]** をクリックします。

    ![Windows Server のバックアップ スケジュールを設定します](./media/backup-azure-manage-windows-server/finish-exclusions.png)

## よく寄せられる質問
**Q1.Azure Backup エージェントではバックアップ ジョブの状態が完了済みになっているのに、その状態がポータルにすぐに反映されないのはなぜですか。**

A1.Azure Backup エージェントのバックアップ ジョブの状態が Azure ポータルに反映されるには、最大で 15 分かかります。

**Q2. バックアップ ジョブが失敗した場合、アラートが発生するまでどのくらい時間がかかりますか。**

A2. アラートは、Azure のバックアップが失敗してから 5 分以内に発生します。

**Q3.通知が設定されているのに、電子メールが送信されないことはありますか。**

A3.アラートのノイズを軽減する目的で、次の場合は通知が送信されません。

   - 通知頻度が 1 時間ごとに設定されており、アラートが発生してから 1 時間以内に解決した。
   - ジョブが取り消された。
   - 元のバックアップ ジョブが進行中のために 2 番目のバックアップ ジョブが失敗した。

## 監視に関する問題のトラブルシューティング<br>
#### 問題: Azure Backup エージェントからのジョブとアラートがポータルに表示されない。
##### トラブルシューティングの手順:
ジョブやアラートのデータを Azure Backup サービスに送信するには、"OBRecoveryServicesManagementAgent" を使用します。タスク マネージャーを開き、"OBRecoveryServicesManagementAgent" プロセスが実行されているかどうかを確認します。場合によっては、このプロセスが停止またはシャットダウンしていることがあります。このプロセスが実行されていない場合は、コントロール パネルからサービスの一覧を表示し、"Microsoft Azure Recovery Services 管理エージェント" を開始または再起動してください。詳細については、"Azure Backup エージェントのインストール フォルダー"\\Microsoft Azure Recovery Services Agent\\Temp\\GatewayProvider* にあるログを参照してください。<b>例:</b> C:\\Program Files\\Microsoft Azure Recovery Services Agent\\Temp\\GatewayProvider0.errlog

## 次のステップ
- [Azure からの Windows Server または Windows クライアントの復元](backup-azure-restore-windows-server.md)
- Azure Backup の詳細については、「[Azure Backup の概要](backup-introduction-to-azure-backup.md)」を参照してください。
- [Azure Backup フォーラム](http://go.microsoft.com/fwlink/p/?LinkId=290933)にアクセスします。

<!---HONumber=AcomDC_0921_2016-->