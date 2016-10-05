<properties
	pageTitle="Azure Media Hyperlapse を使用する Hyperlapse メディア ファイル | Microsoft Azure"
	description="Azure Media Hyperlapse は、最初のユーザーまたはアクション カメラのコンテンツから滑らかな低速度撮影ビデオを作成します。このトピックでは、Media Indexer の使用方法について説明します。"
	services="media-services"
	documentationCenter=""
	authors="asolanki"
	manager="johndeu"
	editor=""/>

<tags
	ms.service="media-services"
	ms.workload="media"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="09/19/2016"  
	ms.author="adsolank"/>


# Azure Media Hyperlapse を使用する Hyperlapse メディア ファイル

Azure Media Hyperlapse は、最初のユーザーまたはアクション カメラのコンテンツから滑らかな低速度撮影ビデオを作成するメディア プロセッサ (MP) です。[Microsoft Research のデスクトップ版 Hyperlapse Pro および電話ベースの Hyperlapse Mobile](http://aka.ms/hyperlapse) とはクラウド ベースの兄弟である、Azure Media Services 向けの Microsoft Hyperlapse では大規模な Azure Media Services メディア処理プラットフォームを利用して、Hyperlapse の一括処理を水平方向にスケーリングし、並列化します。

>[AZURE.IMPORTANT]Microsoft Hyperlapse は、移動カメラを使用する最初のユーザー コンテンツで最適に機能するように設計されています。静止カメラの映像は引き続き動作しますが、Azure Media Hyperlapse メディア プロセッサのパフォーマンスと品質は他の種類のコンテンツに対して保証できません。Azure Media Services 向けの Microsoft Hyperlapse に関する詳細といくつかのビデオ例については、パブリック プレビューの[概要のブログ投稿](http://aka.ms/azurehyperlapseblog)を確認してください。

Azure Media Hyperlapse ジョブでは、低速度撮影のビデオ フレームとその速度 (たとえば、最初の 10,000 フレームは 2 倍速) を指定する構成ファイルと共に、MP4、MOV、または WMV ファイルを入力として使用します。出力は、入力ビデオの安定した低速度撮影画像となります。

Azure Media Hyperlapse の最新の更新プログラムについては、[Media Services のブログ](https://azure.microsoft.com/blog/topics/media-services/)をご覧ください。

## Hyperlapse における資産

最初に、必要な入力ファイルを Azure Media Services にアップロードする必要があります。コンテンツのアップロードと管理に関する概念の詳細については、[コンテンツ管理の記事](media-services-portal-vod-get-started.md)をお読みください。

###  <a id="configuration"></a>Hyperlapse の構成プリセット

コンテンツを Media Services アカウントにアップロードしたら、構成プリセットを作成する必要があります。次の表では、ユーザー指定のフィールドについて説明します。

 フィールド | Description
-------|-------------
StartFrame|Microsoft Hyperlapse 処理を開始する必要があるフレーム。
NumFrames|処理するフレームの数。
速度|入力ビデオの高速化に使用する係数。

XML と JSON で準拠する構成ファイルの例を以下に示します。

**XML プリセット:**

	<?xml version="1.0" encoding="utf-16"?>
	<Preset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" Version="1.0" xmlns="http://www.windowsazure.com/media/encoding/Preset/2014/03">
		<Sources>
			<Source StartFrame="0" NumFrames="10000" />
		</Sources>
		<Options>
			<Speed>12</Speed>
		</Options>
	</Preset>

**JSON プリセット:**

	{
		"Version":1.0,
		"Sources": [
			{
				"StartFrame":0,
				"NumFrames":2147483647
			}
		],
		"Options": {
			"Speed":1,
			"Stabilize":false
		}
	}

###  <a id="sample_code"></a> AMS .NET SDK を使用する Microsoft Hyperlapse

次のメソッドは、資産としてメディア ファイルをアップロードし、Azure Media Hyperlapse メディア プロセッサでジョブを作成します。

> [AZURE.NOTE] このコードを動作させるには、スコープ内に "context" という名前の CloudMediaContext が既に存在している必要があります。この詳細については、[コンテンツ管理の記事](media-services-dotnet-get-started.md)をお読みください。

> [AZURE.NOTE] 上記の JSON または XML の準拠する構成プリセットには、"hyperConfig" という文字列引数が必要です。

static bool RunHyperlapseJob(string input, string output, string hyperConfig) { // create asset with input file IAsset asset = context .Assets .CreateAssetAndUploadSingleFile(input, "My Hyperlapse Input", AssetCreationOptions.None);

// grab instances of Azure Media Hyperlapse MP IMediaProcessor mp = context .MediaProcessors .GetLatestMediaProcessorByName("Azure Media Hyperlapse");

// create Job with Hyperlapse task IJob job = context .Jobs .Create(String.Format("Hyperlapse {0}", input));

if (String.IsNullOrEmpty(hyperConfig)) { // config cannot be empty return false; }

hyperConfig = File.ReadAllText(hyperConfig);

ITask hyperlapseTask = job.Tasks.AddNew("Hyperlapse task", mp, hyperConfig, TaskOptions.None); hyperlapseTask.InputAssets.Add(asset); hyperlapseTask.OutputAssets.AddNew("Hyperlapse output", AssetCreationOptions.None);


job.Submit();

// Create progress printing and querying tasks Task progressPrintTask = new Task(() => {

IJob jobQuery = null; do { var progressContext = context; jobQuery = progressContext.Jobs .Where(j => j.Id == job.Id) .First(); Console.WriteLine(string.Format("{0}\\t{1}\\t{2}", DateTime.Now, jobQuery.State, jobQuery.Tasks[0].Progress)); Thread.Sleep(10000); } while (jobQuery.State != JobState.Finished && jobQuery.State != JobState.Error && jobQuery.State != JobState.Canceled); }); progressPrintTask.Start();

			Task progressJobTask = job.GetExecutionProgressTask(
												 CancellationToken.None);
			progressJobTask.Wait();

			// If job state is Error, the event handling
			// method for job progress should log errors.  Here we check
			// for error state and exit if needed.
			if (job.State == JobState.Error)
			{
				ErrorDetail error = job.Tasks.First().ErrorDetails.First();
				Console.WriteLine(string.Format("Error: {0}. {1}",
												error.Code,
												error.Message));  
				return false;                  
			}

		DownloadAsset(job.OutputMediaAssets.First(), output);
		return true;
	}

	static void DownloadAsset(IAsset asset, string outputDirectory)
	{
		foreach (IAssetFile file in asset.AssetFiles)
		{
			file.Download(Path.Combine(outputDirectory, file.Name));
		}
	}


	static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
	{
	    IAsset asset = context.Assets.Create(assetName, options);

	    var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
	    assetFile.Upload(filePath);

	    return asset;
	}

	static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
	{
	    var processor = context.MediaProcessors
	    .Where(p => p.Name == mediaProcessorName)
	    .ToList()
	    .OrderBy(p => new Version(p.Version))
	    .LastOrDefault();

	    if (processor == null)
	        throw new ArgumentException(string.Format("Unknown media processor",
	                                                   mediaProcessorName));

	    return processor;
	}

### <a id="file_types"></a>サポートされるファイルの種類

- MP4
- MOV
- WMV



##Media Services のラーニング パス

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]


##関連リンク

[Azure Media Services Analytics の概要](media-services-analytics-overview.md)

[Azure Media Analytics デモ](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)

<!---HONumber=AcomDC_0921_2016-->