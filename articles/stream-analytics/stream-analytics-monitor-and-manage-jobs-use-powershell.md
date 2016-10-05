<properties 
	pageTitle="PowerShell | Microsoft Azure を使用した Stream Analytics ジョブの監視および管理" 
	description="Azure PowerShell およびコマンドレットを使用して Stream Analytics ジョブを監視および管理する方法について説明します。" 
	keywords="azure powershell、azure powershell コマンドレット、powershell コマンド、powershell スクリプト"	
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="jhubbard" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.tgt_pltfrm="na" 
	ms.workload="data-services" 
	ms.date="07/27/2016" 
	ms.author="jeffstok"/>


# Azure PowerShell コマンドレットを使用した Stream Analytics ジョブの監視および管理

基本的な Stream Analytics タスクを実行する Azure PowerShell コマンドレットと PowerShell スクリプトを使用して、Stream Analytics リソースを監視および管理する方法について説明します。

## Stream Analytics 用に Azure PowerShell コマンドレットを実行するための前提条件

 - サブスクリプションに Azure リソース グループを作成する。次に、サンプルの Azure PowerShell スクリプトを示します。Azure PowerShell については、「[Azure PowerShell のインストールおよび構成](../powershell-install-configure.md)」を参照してください。

Azure PowerShell 0.9.8:

 		# Log in to your Azure account
		Add-AzureAccount

		# Select the Azure subscription you want to use to create the resource group if you have more than one subscription on your account.
		Select-AzureSubscription -SubscriptionName <subscription name>
 
		# If Stream Analytics has not been registered to the subscription, remove remark symbol below (#) to run the Register-AzureProvider cmdlet to register the provider namespace.
		#Register-AzureProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'

		# Create an Azure resource group
		New-AzureResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>

Azure PowerShell 1.0:

 		# Log in to your Azure account
		Login-AzureRmAccount

		# Select the Azure subscription you want to use to create the resource group.
		Get-AzureRmSubscription –SubscriptionName “your sub” | Select-AzureRmSubscription

		# If Stream Analytics has not been registered to the subscription, remove remark symbol below (#) to run the Register-AzureProvider cmdlet to register the provider namespace.
		#Register-AzureRmResourceProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'
		
		# Create an Azure resource group
		New-AzureRMResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>
		


> [AZURE.NOTE] プログラムで作成された Stream Analytics ジョブは、既定では監視は有効になっていません。Azure ポータルで、ジョブの [監視] ページに移動し、[有効にする] ボタンをクリックして、監視を手動で有効にすることができます。また、「[Stream Analytics ジョブ モニターをプログラムで作成する](stream-analytics-monitor-jobs.md)」に記載されている手順に従って、プログラムで有効にすることもできます。

## Stream Analytics 用の Azure PowerShell コマンドレット
次の Azure PowerShell コマンドレットは、Azure Stream Analytics ジョブの監視と管理に使用できます。Azure PowerShell にはさまざまなバージョンがあります。**ここに示す各例では、最初が Azure PowerShell 0.9.8 のコマンド、2 番目が Azure PowerShell 1.0 のコマンドです。** Azure PowerShell 1.0 のコマンドには、常に "AzureRM" が含まれます。

### Get-AzureStreamAnalyticsJob | Get-AzureRMStreamAnalyticsJob
Azure サブスクリプションまたは指定したリソースグループに定義されているすべての Stream Analytics ジョブを一覧表示したり、リソース グループ内の特定のジョブに関するジョブ情報を取得したりします。

**例 1**

Azure PowerShell 0.9.8:

	Get-AzureStreamAnalyticsJob

Azure PowerShell 1.0:

	Get-AzureRMStreamAnalyticsJob

この PowerShell コマンドは、Azure サブスクリプションのすべての Stream Analytics ジョブに関する情報を返します。

**例 2**

Azure PowerShell 0.9.8:

	Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US 

Azure PowerShell 1.0:

	Get-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US 

この PowerShell コマンドは、リソース グループ StreamAnalytics-Default-Central-US のすべての Stream Analytics ジョブに関する情報を返します。

**例 3**

Azure PowerShell 0.9.8:

	Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob

Azure PowerShell 1.0:

	Get-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob

この PowerShell コマンドは、リソース グループ StreamAnalytics-Default-Central-US の Stream Analytics ジョブ StreamingJob に関する情報を返します。

### Get-AzureStreamAnalyticsInput | Get-AzureRMStreamAnalyticsInput
指定した Stream Analytics ジョブに定義されたすべての入力を一覧表示したり、特定の入力に関する情報を取得したりします。

**例 1**

Azure PowerShell 0.9.8:

	Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob

Azure PowerShell 1.0:

	Get-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob

この PowerShell コマンドは、ジョブ StreamingJob に定義されたすべての入力に関する情報を返します。

**例 2**

Azure PowerShell 0.9.8:

	Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name EntryStream

Azure PowerShell 1.0:

	Get-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name EntryStream

この PowerShell コマンドは、ジョブ StreamingJob に定義された EntryStream という名前の入力に関する情報を返します。

### Get-AzureStreamAnalyticsOutput | Get-AzureRMStreamAnalyticsOutput
指定した Stream Analytics ジョブに定義されたすべての出力を一覧表示したり、特定の出力に関する情報を取得したりします。

**例 1**

Azure PowerShell 0.9.8:

	Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob

Azure PowerShell 1.0:

	Get-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob

この PowerShell コマンドは、ジョブ StreamingJob に定義された出力に関する情報を返します。

**例 2**

Azure PowerShell 0.9.8:

	Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name Output

Azure PowerShell 1.0:

	Get-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name Output

この PowerShell コマンドは、ジョブ StreamingJob に定義された Output という名前の出力に関する情報を返します。

### Get-AzureStreamAnalyticsQuota | Get-AzureRMStreamAnalyticsQuota
指定したリージョンのストリーミング ユニットのクォータに関する情報を取得します。

**例 1**

Azure PowerShell 0.9.8:

	Get-AzureStreamAnalyticsQuota –Location "Central US" 

Azure PowerShell 1.0:

	Get-AzureRMStreamAnalyticsQuota –Location "Central US" 

この PowerShell コマンドは、米国中部リージョンのストリーミング ユニットのクォータと使用状況に関する情報を返します。

### Get-AzureStreamAnalyticsTransformation | GetAzureRMStreamAnalyticsTransformation
Stream Analytics ジョブに定義されている特定の変換に関する情報を取得します。

**例 1**

Azure PowerShell 0.9.8:

	Get-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name StreamingJob

Azure PowerShell 1.0:

	Get-AzureRMStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –Name StreamingJob

この PowerShell コマンドは、StreamingJob ジョブ上の StreamingJob という名前の変換に関する情報を返します。

### New-AzureStreamAnalyticsInput | New-AzureRMStreamAnalyticsInput
Stream Analytics ジョブ内に新しい入力を作成したり、指定した既存の入力を更新したりします。
  
入力の名前は .json ファイルまたはコマンド ラインで指定できます。その両方で名前を指定する場合は、コマンド ラインで指定する名前をファイル内の名前と同じものにする必要があります。

既に存在する入力を指定し、–Force パラメーターを指定しない場合は、既存の入力を置き換えるかどうかを尋ねられます。

–Force パラメーターを指定し、既存の入力名を指定すると、確認を求められることなく、入力が置き換えられます。

JSON ファイルの構造と内容に関する詳細については、「[Stream Analytics 管理 REST API 参照][stream.analytics.rest.api.reference]」ライブラリの『[入力の作成 (Azure Stream Analytics)][msdn-rest-api-create-stream-analytics-input]』のセクションを参照してください。

**例 1**

Azure PowerShell 0.9.8:

	New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" 

Azure PowerShell 1.0:

	New-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" 

この PowerShell コマンドは、ファイル Input.json から新しい入力を作成します。入力定義ファイル内で指定した名前と同じ名前を持つ入力が既に存在する場合は、入力を置き換えるかどうかを尋ねられます。

**例 2**

Azure PowerShell 0.9.8:

	New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" –Name EntryStream

Azure PowerShell 1.0:

	New-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" –Name EntryStream

この PowerShell コマンドは EntryStream という名前のジョブに新しい入力を作成します。この名前の入力が既に定義されている場合は、入力を置き換えるかどうかを尋ねられます。

**例 3**

Azure PowerShell 0.9.8:

	New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" –Name EntryStream -Force

Azure PowerShell 1.0:

	New-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob –File "C:\Input.json" –Name EntryStream -Force

この PowerShell コマンドは EntryStream という名前の既存の入力ソースの定義をファイルの定義に置き換えます。

### New-AzureStreamAnalyticsJob | New-AzureRMStreamAnalyticsJob
Microsoft Azure に新しい Stream Analytics ジョブを作成したり、指定した既存のジョブの定義を更新したりします。

ジョブの名前は .json ファイルまたはコマンド ラインで指定できます。その両方で名前を指定する場合は、コマンド ラインで指定する名前をファイル内の名前と同じものにする必要があります。

既に存在するジョブ名を指定し、–Force パラメーターを指定しない場合は、既存のジョブを置き換えるかどうかを尋ねられます。

–Force パラメーターを指定し、既存のジョブ名を指定すると、確認を求められることなく、ジョブ定義が置き換えられます。

JSON ファイルの構造と内容に関する詳細については、「[Stream Analytics 管理 REST API 参照][stream.analytics.rest.api.reference]」ライブラリの『[Stream Analytics ジョブの作成][msdn-rest-api-create-stream-analytics-job]』のセクションを参照してください。

**例 1**

Azure PowerShell 0.9.8:

	New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\JobDefinition.json" 

Azure PowerShell 1.0:

	New-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\JobDefinition.json" 

この PowerShell コマンドは JobDefinition.json の定義から新しいジョブを作成します。ジョブ定義ファイル内で指定した名前と同じ名前を持つジョブが既に存在する場合は、ジョブを置き換えるかどうかを尋ねられます。

**例 2**

Azure PowerShell 0.9.8:

	New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\JobDefinition.json" –Name StreamingJob -Force

Azure PowerShell 1.0:

	New-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\JobDefinition.json" –Name StreamingJob -Force

この PowerShell コマンドは StreamingJob のジョブ定義を置き換えます。

### New-AzureStreamAnalyticsOutput | New-AzureRMStreamAnalyticsOutput
Stream Analytics ジョブ内に新しい出力を作成したり、既存の出力を更新したりします。

出力の名前は .json ファイルまたはコマンド ラインで指定できます。その両方で名前を指定する場合は、コマンド ラインで指定する名前をファイル内の名前と同じものにする必要があります。

既に存在する出力を指定し、–Force パラメーターを指定しない場合は、既存の出力を置き換えるかどうかを尋ねられます。

–Force パラメーターを指定し、既存の出力名を指定すると、確認を求められることなく、出力が置き換えられます。

JSON ファイルの構造と内容に関する詳細については、「[Stream Analytics 管理 REST API 参照][stream.analytics.rest.api.reference]」ライブラリの『[出力の作成 (Azure Stream Analytics)][msdn-rest-api-create-stream-analytics-output]』のセクションを参照してください。

**例 1**

Azure PowerShell 0.9.8:

	New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Output.json" –JobName StreamingJob –Name output

Azure PowerShell 1.0:

	New-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Output.json" –JobName StreamingJob –Name output

この PowerShell コマンドは、ジョブ StreamingJob に "output" という名前の新しい出力を作成します。この名前の出力が既に定義されている場合は、出力を置き換えるかどうかを尋ねられます。

**例 2**

Azure PowerShell 0.9.8:

	New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Output.json" –JobName StreamingJob –Name output -Force

Azure PowerShell 1.0:

	New-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Output.json" –JobName StreamingJob –Name output -Force

この PowerShell コマンドは、ジョブ StreamingJob の "output" の定義を置き換えます。

### New-AzureStreamAnalyticsTransformation | New-AzureRMStreamAnalyticsTransformation
Stream Analytics ジョブ内に新しい変換を作成したり、既存の変換を更新したります。
  
変換の名前は .json ファイルまたはコマンド ラインで指定できます。その両方で名前を指定する場合は、コマンド ラインで指定する名前をファイル内の名前と同じものにする必要があります。

既に存在する変換を指定し、–Force パラメーターを指定しない場合は、既存の変換を置き換えるかどうかを尋ねられます。

–Force パラメーターを指定し、既存の変換名を指定すると、確認を求められることなく、変換が置き換えられます

JSON ファイルの構造と内容に関する詳細については、「[Stream Analytics 管理 REST API 参照][stream.analytics.rest.api.reference]」ライブラリの『[変換の作成 (Azure Stream Analytics)][msdn-rest-api-create-stream-analytics-transformation]』のセクションを参照してください。

**例 1**

Azure PowerShell 0.9.8:

	New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform

Azure PowerShell 1.0:

	New-AzureRMStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform

この PowerShell コマンドは、ジョブ StreamingJob に StreamingJobTransform という名前の新しい変換を作成します。この名前と同じ名前を持つ変換が既に存在する場合は、置き換えるかどうかを尋ねられます。

**例 2**

Azure PowerShell 0.9.8:

	New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform -Force

Azure PowerShell 1.0:

	New-AzureRMStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US –File "C:\Transformation.json" –JobName StreamingJob –Name StreamingJobTransform -Force

 この PowerShell コマンドは、ジョブ StreamingJob の StreamingJobTransform の定義を置き換えます。

### Remove-AzureStreamAnalyticsInput | Remove-AzureRMStreamAnalyticsInput
Microsoft Azure 内の Stream Analytics ジョブの特定の入力を非同期的に削除します。–Force パラメーターを指定すると、確認を求められることなく、入力が削除されます。

**例 1**

Azure PowerShell 0.9.8:

	Remove-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name EventStream

Azure PowerShell 1.0:

	Remove-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name EventStream

この PowerShell コマンドは、ジョブ StreamingJob 内の入力 EventStream を削除します。

### Remove-AzureStreamAnalyticsJob | Remove-AzureRMStreamAnalyticsJob
Microsoft Azure 内の特定の Stream Analytics ジョブを非同期的に削除します。–Force パラメーターを指定すると、確認を求められることなく、ジョブが削除されます。

**例 1**

Azure PowerShell 0.9.8:

	Remove-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –Name StreamingJob 

Azure PowerShell 1.0:

	Remove-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –Name StreamingJob 

この PowerShell コマンドはジョブ StreamingJob を削除します。

### Remove-AzureStreamAnalyticsOutput | Remove-AzureRMStreamAnalyticsOutput
Microsoft Azure 内の Stream Analytics ジョブの特定の出力を非同期的に削除します。–Force パラメーターを指定すると、確認を求められることなく、出力が削除されます。

**例 1**

Azure PowerShell 0.9.8:

	Remove-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name Output

Azure PowerShell 1.0:

	Remove-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name Output

この PowerShell コマンドは、ジョブ StreamingJob 内の出力 Output を削除します。

### Start-AzureStreamAnalyticsJob | Start-AzureRMStreamAnalyticsJob
Microsoft Azure に Stream Analytics ジョブを非同期的に展開し開始します。

**例 1**

Azure PowerShell 0.9.8:

	Start-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z

Azure PowerShell 1.0:

	Start-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z

この PowerShell コマンドは、カスタム出力の開始時刻が UTC の 2012 年 12 月 12 日 12:12:12 に設定された StreamingJob ジョブを開始します。


### Stop-AzureStreamAnalyticsJob | Stop-AzureRMStreamAnalyticsJob
Microsoft Azure で実行中の Stream Analytics ジョブを非同期的に停止し、使用していたリソースの割り当てを解除します。ジョブを編集したり再開したりできるように、Azure Portal と管理 API を使えば、ジョブ定義とメタデータをサブスクリプション内で引き続き利用できます。停止状態のジョブに対しては課金されません。

**例 1**

Azure PowerShell 0.9.8:

	Stop-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –Name StreamingJob 

Azure PowerShell 1.0:

	Stop-AzureRMStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US –Name StreamingJob 

この PowerShell コマンドはジョブ StreamingJob を停止します。

### Test-AzureStreamAnalyticsInput | Test-AzureRMStreamAnalyticsInput
指定した入力に接続するために、Stream Analytics の機能をテストします。

**例 1**

Azure PowerShell 0.9.8:

	Test-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name EntryStream

Azure PowerShell 1.0:

	Test-AzureRMStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name EntryStream

この PowerShell コマンドは、StreamingJob 内の入力 EntryStream の接続状態をテストします。

###Test-AzureStreamAnalyticsOutput | Test-AzureRMStreamAnalyticsOutput
指定した出力に接続するために、Stream Analytics の機能をテストします。

**例 1**

Azure PowerShell 0.9.8:

	Test-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name Output

Azure PowerShell 1.0:

	Test-AzureRMStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US –JobName StreamingJob –Name Output

この PowerShell コマンドは、StreamingJob 内の出力 Output の接続状態をテストします。

## サポートを受ける
さらにサポートが必要な場合は、[Azure Stream Analytics フォーラム](https://social.msdn.microsoft.com/Forums/ja-JP/home?forum=AzureStreamAnalytics)を参照してください。


## 次のステップ

- [Azure Stream Analytics の概要](stream-analytics-introduction.md)
- [Azure Stream Analytics の使用](stream-analytics-get-started.md)
- [Azure Stream Analytics ジョブのスケーリング](stream-analytics-scale-jobs.md)
- [Stream Analytics Query Language Reference (Stream Analytics クエリ言語リファレンス)](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure Stream Analytics management REST API reference (Azure ストリーム分析の管理 REST API リファレンス)](https://msdn.microsoft.com/library/azure/dn835031.aspx)
 



[msdn-switch-azuremode]: http://msdn.microsoft.com/library/dn722470.aspx
[powershell-install]: http://azure.microsoft.com/documentation/articles/powershell-install-configure/
[msdn-rest-api-create-stream-analytics-job]: https://msdn.microsoft.com/library/dn834994.aspx
[msdn-rest-api-create-stream-analytics-input]: https://msdn.microsoft.com/library/dn835010.aspx
[msdn-rest-api-create-stream-analytics-output]: https://msdn.microsoft.com/library/dn835015.aspx
[msdn-rest-api-create-stream-analytics-transformation]: https://msdn.microsoft.com/library/dn835007.aspx

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-get-started.md
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301
 

<!---HONumber=AcomDC_0921_2016-->