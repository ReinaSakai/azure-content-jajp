<properties
    pageTitle="Azure Automation での初めての PowerShell Runbook | Microsoft Azure"
    description="簡単な PowerShell Runbook を作成、テスト、および発行する手順を説明するチュートリアルです。"
    services="automation"
    documentationCenter=""
    authors="mgoedtel"
    manager="jwhit"
    editor="tysonn"/>

<tags
    ms.service="automation"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="04/21/2016"
    ms.author="magoedte;sngun"/>

# 初めての PowerShell Runbook

> [AZURE.SELECTOR] - [Graphical](automation-first-runbook-graphical.md) - [PowerShell Workflow](automation-first-runbook-textual.md) - [PowerShell](automation-first-runbook-textual-PowerShell.md)

このチュートリアルでは、Azure Automation で [PowerShell Runbook](automation-runbook-types.md#powershell-runbooks) を作成する手順について説明します。簡単な Runbook を作成、テスト、発行しながら、Runbook ジョブの状態を追跡する方法を説明します。その後、実際に Azure リソースを管理するように Runbook を変更し、ここでは Azure 仮想マシンを開始します。そして、Runbook のパラメーターを追加することによって、Runbook をいっそう堅牢にします。

## 前提条件

このチュートリアルを最後まで行うには、以下のものが必要です。

-	として機能します。まだお持ちでない場合は、[MSDN サブスクライバーの特典を有効にする](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)か、<a href="/pricing/free-account/" target="_blank">[無料アカウントにサインアップ](https://azure.microsoft.com/free/)してください。
-	[Automation アカウント](automation-security-overview.md)。Runbook の保存と Azure リソースの認証に使用します。このアカウントには、仮想マシンを開始および停止するアクセス許可が必要です。
-	Azure 仮想マシン。マシンを停止して起動するので、運用環境のものは使用しないでください。

## 手順 1 - 新しい Runbook を作成する

最初に、*Hello World* というテキストを出力する簡単な Runbook を作成します。

1.	Azure ポータルで、Automation アカウントを開きます。  
	Automation アカウント ページでは、そのアカウントのリソースを簡単に確認できます。既に資産がいくつかあります。これらのほとんどは、新しい Automation アカウントに自動的に含まれるモジュールです。[前提条件](#prerequisites)で説明されている資格情報資産も必要です。
2.	**[Runbook]** タイルをクリックして、Runbook の一覧を開きます。  
	![RunbooksControl](media/automation-first-runbook-textual-powershell/automation-runbooks-control.png)  
3.	**[Runbook の追加]** ボタンをクリックし、次に **[新しい Runbook の作成]** をクリックして、新しい Runbook を作成します
4.	Runbook に *MyFirstRunbook-PowerShell* という名前を付けます。
5.	ここでは、[PowerShell Runbook](automation-runbook-types.md#powershell-runbooks) を作成するため、**[Runbook の種類]** として **[PowerShell]** を選択します。  
	![Runbook Type](media/automation-first-runbook-textual-powershell/automation-runbook-type.png)  
6.	**[作成]** をクリックして Runbook を作成し、テキスト エディターを開きます。

## 手順 2 - コードを Runbook に追加する

Runbook に直接コードを入力するか、ライブラリ コントロールからコマンドレット、Runbook、資産を選択し、関連するパラメーターを使って Runbook に追加できます。このチュートリアルでは、Runbook に直接コードを入力します。

1.	作成した Runbook は現在空であるため、「 *Write-Output "Hello World."* 」と入力します。  
	![Hello World](media/automation-first-runbook-textual-powershell/automation-helloworld.png)  
2.	**[保存]** をクリックして Runbook を保存します。
	![Save Button](media/automation-first-runbook-textual-powershell/automation-save-button.png)  

## 手順 3 - Runbook をテストする

Runbook を発行して運用環境で使用できるようにする前に、Runbook をテストして正常に動作することを確認します。Runbook をテストするときは、**ドラフト** バージョンを実行し、その出力を対話形式で表示します。

1.	**[テスト ウィンドウ]** をクリックして、テスト ウィンドウを開きます。  
	![Test Pane](media/automation-first-runbook-textual-powershell/automation-testpane.png)  
2.	**[開始]** をクリックしてテストを開始します。有効なオプションはこれだけです。
3.	[Runbook ジョブ](automation-runbook-execution.md)が作成され、その状態が表示されます。
	最初のジョブの状態は*キューに設定*であり、クラウドの Runbook ワーカーが使用できるようになるのを待っていることを示します。その後、ワーカーがジョブを要求すると*開始中*になり、Runbook が実際に実行を開始すると*実行中*になります。  
4.	Runbook ジョブが完了すると、その出力が表示されます。この例では、"*Hello World*" と表示されます。  
	![Test Pane Output](media/automation-first-runbook-textual-powershell/automation-testpane-output.png)  
5.	テスト ウィンドウを閉じてキャンバスに戻ります。

## 手順 4 - Runbook を発行して開始する

前の手順で作成した Runbook はまだドラフト モードです。運用環境で実行できるようにするには、発行する必要があります。Runbook を発行するときは、既存の発行済みバージョンをドラフト バージョンで上書きします。この例では、Runbook を作成したばかりなので、発行済みバージョンはまだありません。

1.	**[発行]** をクリックして Runbook を発行し、確認を要求されたら **[はい]** をクリックします。  
	![[発行] ボタン](media/automation-first-runbook-textual-powershell/automation-publish-button.png)  
2.	**[Runbook]** ウィンドウで左にスクロールして Runbook を表示すると、**[作成状態]** は **[発行済]** になっています。
3.	右にスクロールして戻り、**MyFirstRunbook-PowerShell** のウィンドウを表示します。  
	上部のオプションを使用すると、Runbook の開始、Runbook の表示、将来の開始スケジュールの設定を行ったり、HTTP 呼び出しで開始できるように [Webhook](automation-webhooks.md) を作成したりすることができます。
4.	ここでは Runbook を開始するだけなので、**[開始]** をクリックし、[Runbook の開始] ブレードが開いたら **[OK]** をクリックします。  
	![Start button](media/automation-first-runbook-textual-powershell/automation-start-button.png)  
5.	作成した Runbook ジョブのジョブ ウィンドウが開かれます。このウィンドウは閉じてもかまいませんが、ここではジョブの進行状況を確認できるように開いたままにします。
6.	ジョブの状態が **[ジョブの概要]** に表示され、Runbook をテストしたときに確認した状態と一致しています。  
	![ジョブの概要](media/automation-first-runbook-textual-powershell/automation-job-summary.png)  
7.	Runbook の状態が *[完了]* になったら、**[出力]** をクリックします。[出力] ウィンドウが開き、"*Hello World*" と表示されます。  
	![ジョブの出力](media/automation-first-runbook-textual-powershell/automation-job-output.png)
8.	[出力] ウィンドウを閉じます。
9.	**[すべてのログ]** をクリックして、Runbook ジョブのストリーム ウィンドウを開きます。出力ストリームでは "*Hello World*" だけが表示されますが、Runbook が Verbose や Error などの Runbook ジョブに書き込んでいる場合は、そのような Runbook ジョブのストリームも表示されます。
	![All Logs](media/automation-first-runbook-textual-powershell/automation-alllogs.png)  
10.	ストリーム ウィンドウとジョブ ウィンドウを閉じると、MyFirstRunbook-PowerShell ウィンドウに戻ります。
11.	**[ジョブ]** をクリックして、この Runbook のジョブ ウィンドウを開きます。この Runbook によって作成されたジョブの一覧が表示されます。ジョブを 1 回実行しただけなので、一覧に表示されるジョブは 1 つだけです。  
	![Job List](media/automation-first-runbook-textual-powershell/automation-job-list.png)  
12.	このジョブをクリックすると、Runbook を開始したときに表示されたものと同じジョブ ウィンドウが開きます。これにより前に戻って、特定の Runbook に対して作成されたジョブの詳細を見ることができます。

## 手順 5 - Azure リソースを管理するための認証を追加する

Runbook をテストして発行しましたが、これまでのところ役に立つことは何もしていません。ここでは、Runbook で Azure リソースを管理します。ただし、[前提条件](#prerequisites)で示されている資格情報を使用して認証を行わないと、これを実現することはできません。**Add-AzureAccount** コマンドレットでこれを行います。

1.	MyFirstRunbook-PowerShell ウィンドウで **[編集]** をクリックして、テキスト エディターを開きます。  
	![Edit Runbook](media/automation-first-runbook-textual-powershell/automation-edit-runbook.png)  
2.	**Write-Output** の行は不要になったので削除します。
3.	ライブラリ コントロールで、**[資産]**、**[資格情報]** の順に展開します。
4.	資格情報を右クリックし、**[キャンバスに追加]** をクリックします。これにより、資格情報に対する **Get-AutomationPSCredential** アクティビティが追加されます。
5.	**Get-AutomationPSCredential** の前に「*$Credential =*」と入力して、資格情報を変数に割り当てます。
6.	次の行に、「*Add-AzureAccount -Credential $Credential*」と入力します。  
	![資格情報](media/automation-first-runbook-textual-powershell/automation-get-credential.png)
7.	**テスト ウィンドウ**をクリックして、Runbook をテストできるようにします。
8.	**[開始]** をクリックしてテストを開始します。テストが完了すると、アカウントのサブスクリプション ID、種類、テナントを含む出力が返されます。これは、資格情報が有効であることの確認になります。

## 手順 6 - 仮想マシンを開始するコードを追加する

これで、Runbook で Azure サブスクリプションに対する認証が行われ、リソースを管理できるようになります。仮想マシンを起動するコマンドを追加します。Azure サブスクリプション内の任意の仮想マシンを選択し、ここではその名前をコマンドレットにハードコーディングします。

1.	*Add-AzureAccount* の後ろに、「 *Start-AzureVM -Name 'VMName' -ServiceName 'VMServiceName'* 」と入力して、起動する仮想マシンの名前とサービス名を指定します。  
	![StartVM](media/automation-first-runbook-textual-powershell/automation-startvm.png)  
2.	Runbook を保存し、**テスト ウィンドウ**をクリックしてテストできるようにします。
3.	**[開始]** をクリックしてテストを開始します。完了したら、仮想マシンが開始されたことを確認します。

## 手順 7 - Runbook に入力パラメーターを追加する

この Runbook では、Runbook にハードコードした仮想マシンを開始しますが、Runbook を開始するときに仮想マシンを指定できるようにすればもっと便利です。そこで次に、入力パラメーターを Runbook に追加して、その機能を提供します。

1.	*VMName* と *VMServiceName* のパラメーターを Runbook に追加して、次の図に示すように、**Start-AzureVM** コマンドレットでこれらの変数を使用します。  
	![Add Parameter](media/automation-first-runbook-textual-powershell/automation-add-parameter.png)  
2.	Runbook を保存してテスト ウィンドウを開きます。テストで使用される 2 つの入力変数の値を指定できることに注意してください。
3.	テスト ウィンドウを閉じます。
4.	**[発行]** をクリックして、新しいバージョンの Runbook を発行します。
5.	前の手順で開始した仮想マシンを停止します。
6.	**[開始]** をクリックして Runbook を開始します。開始する仮想マシンの **VMName** と **VMServiceName** を入力します。  
	![Pass Parameter](media/automation-first-runbook-textual-powershell/automation-pass-parameter.png)  
7.	Runbook が完了したら、仮想マシンが開始されたことを確認します。

## PowerShell Workflow との違い

PowerShell Runbook のライフサイクル、機能、および管理は、PowerShell Workflow Runbook と同じですが、相違点や制限事項がいくつかあります。

1.	PowerShell Runbook は、コンパイル手順がないため、PowerShell Workflow Runbook と比較した場合、動作が速くなります。
2.	PowerShell Workflow Runbook では、チェックポイントがサポートされています。チェックポイントの使用により、PowerShell Workflow Runbook は、Runbook の任意のポイントから再開できます。それに対して、PowerShell Runbook は最初から再開することしかできません。
3.	PowerShell Workflow Runbook では、並列実行と順次実行がサポートされていますが、PowerShell Runbook では、コマンドを順次実行することしかできません。
4.	PowerShell Workflow Runbook では、アクティビティ、コマンド、またはスクリプト ブロックは独自の実行空間を持つことができますが、PowerShell Runbook では、スクリプトのすべての内容が 1 つの実行空間で実行されます。さらに、ネイティブの PowerShell Runbook と PowerShell Workflow Runbook には、[構文の違い](https://technet.microsoft.com/magazine/dn151046.aspx)もあります。

## 次のステップ

-	グラフィカルな Runbook の使用を開始するには、「[初めてのグラフィカルな Runbook](automation-first-runbook-graphical.md)」をご覧ください。
-	PowerShell Workflow Runbook の使用を開始するには、「[最初の PowerShell Workflow Runbook](automation-first-runbook-textual.md)」をご覧ください。
-	Runbook の種類とそれらの利点や制限事項の詳細については、「[Azure Automation の Runbook の種類](automation-runbook-types.md)」をご覧ください。
-	PowerShell スクリプトのサポート機能の詳細については、「[Native PowerShell Script Support in Azure Automation (Azure Automation でのネイティブ PowerShell スクリプトのサポート)](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/)」をご覧ください。

<!---HONumber=AcomDC_0427_2016-->