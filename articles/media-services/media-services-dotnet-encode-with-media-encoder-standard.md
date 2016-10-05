<properties 
	pageTitle=".NET を使用した Media Encoder Standard での資産のエンコード | Microsoft Azure" 
	description="このトピックでは、.NET を使用して、Media Encoder Standard で資産をエンコードする方法について説明します。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
 	ms.date="09/19/2016"
	ms.author="juliako;anilmur"/>


# .NET を使用した Media Encoder Standard での資産のエンコード

エンコード ジョブは Media Services で最も一般的な処理の 1 つです。エンコード ジョブを作成することで、メディア ファイルをあるエンコードから別のエンコードに変換できるようになります。エンコードには、Media Services の組み込み Media Encoder を使用できます。Media Services のパートナーから提供されているエンコーダーを使うこともできます。サードパーティのエンコーダーは Azure Marketplace から入手できます。

このトピックでは、.NET を使用して、Media Encoder Standard (MES) で資産をエンコードする方法について説明します。Media Encoder Standard は、[ここ](http://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409)で説明されているエンコーダーのプリセット文字列のいずれかを使用して構成されます。

中間ファイルは常にアダプティブ ビットレート MP4 セットにエンコードして、その後[動的パッケージ](media-services-dynamic-packaging-overview.md)を使用して目的の形式に変換することをお勧めします。動的パッケージを活用するには、コンテンツの配信元となるストリーミング エンドポイントのオンデマンド ストリーミング ユニットを 1 つ以上取得する必要があります。詳細については、「[Media Services の規模の設定方法](media-services-portal-manage-streaming-endpoints.md)」を参照してください。

出力資産がストレージで暗号化されている場合は、資産配信のポリシーを構成する必要があります。詳細については、「[資産配信ポリシーの構成](media-services-dotnet-configure-asset-delivery-policy.md)」をご覧ください。

>[AZURE.NOTE]MES は、入力ファイル名の最初の 32 文字を含む名前で出力ファイルを生成します。この名前は、プリセット ファイルに指定されているものに基づきます。たとえば、"FileName": "{Basename}\_{Index}{Extension}" の場合、{Basename} が入力ファイル名の最初の 32 文字で置換されます。

###MES の形式

[形式とコーデック](media-services-media-encoder-standard-formats.md)

###MES のプリセット

Media Encoder Standard は、[ここ](http://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409)で説明されているエンコーダーのプリセット文字列のいずれかを使用して構成されます。

###入力メタデータと出力メタデータ

MES を使用して 1 つ (または複数) の入力資産をエンコードし、そのエンコード タスクが正常に終了すると、出力資産が得られます。出力資産には、使用するエンコード プリセットに基づいて、ビデオ、オーディオ、サムネイル、マニフェストなどが含まれます。

この出力資産には、入力資産に関するメタデータが格納されたファイルも含まれます。メタデータ XML ファイルの名前は、<asset\_id>\_metadata.xml という形式になっています (たとえば、41114ad3-eb5e-4c57-8d92-5354e2b7d4a4\_metadata.xml)。ここで、<asset\_id> は入力資産の AssetId の値です。この入力メタデータ XML のスキーマについては、[こちら](http://msdn.microsoft.com/library/azure/dn783120.aspx)で説明されています。

出力資産には、出力資産に関するメタデータが格納されたファイルも含まれます。メタデータ XML ファイルの名前は、<source_file_name>\_manifest.xml という形式になっています (たとえば、BigBuckBunny\_manifest.xml)。この出力メタデータ XML のスキーマについては、[こちら](http://msdn.microsoft.com/library/azure/dn783217.aspx)で説明されています。

2 つのメタデータ ファイルのいずれかを確認したい場合は、SAS ロケータ―を作成してファイルをローカル コンピューターにダウンロードできます。Media Services .NET SDK Extensions を使って SAS ロケータ―を作成し、ファイルをダウンロードする方法の例をご覧ください。

##サンプルのダウンロード

[ここ](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/)からサンプルを取得し、実行します。

##例

次のコード サンプルでは、Media Services SDK を使用して次のタスクを実行します。

- エンコード ジョブを作成します。
- Media Encoder Standard エンコーダーの参照を取得します。
- "H264 Multiple Bitrate 720p" プリセットを使用するように指定します。すべてのプリセットは、[こちら](http://go.microsoft.com/fwlink/?linkid=618336&clcid=0x409)で確認できます。また、これらのプリセットが従う必要のあるスキーマについては、[こちら](https://msdn.microsoft.com/library/mt269962.aspx)のトピックで確認できます。
- 1 つのエンコード タスクをジョブに追加します。
- エンコードする入力資産を指定します。
- エンコードされた資産が含まれる出力資産を作成します。
- ジョブの進行状況を確認するイベント ハンドラーを追加します。
- ジョブを送信します。
		
		static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
		{
		    // Declare a new job.
		    IJob job = _context.Jobs.Create("Media Encoder Standard Job");
		    // Get a media processor reference, and pass to it the name of the 
		    // processor to use for the specific task.
		    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");
		

		    // Create a task with the encoding details, using a string preset.
		    // In this case "H264 Multiple Bitrate 720p" preset is used.
		    ITask task = job.Tasks.AddNew("My encoding task",
		        processor,
		        "H264 Multiple Bitrate 720p",
		        TaskOptions.None);
		
		    // Specify the input asset to be encoded.
		    task.InputAssets.Add(asset);
		    // Add an output asset to contain the results of the job. 
		    // This output is specified as AssetCreationOptions.None, which 
		    // means the output asset is not encrypted. 
		    task.OutputAssets.AddNew("Output asset",
		        AssetCreationOptions.None);
		
		    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
		    job.Submit();
		    job.GetExecutionProgressTask(CancellationToken.None).Wait();
		
		    return job.OutputMediaAssets[0];
		}
		
		private static void JobStateChanged(object sender, JobStateChangedEventArgs e)
		{
		    Console.WriteLine("Job state changed event:");
		    Console.WriteLine("  Previous state: " + e.PreviousState);
		    Console.WriteLine("  Current state: " + e.CurrentState);
		    switch (e.CurrentState)
		    {
		        case JobState.Finished:
		            Console.WriteLine();
		            Console.WriteLine("Job is finished. Please wait while local tasks or downloads complete...");
		            break;
		        case JobState.Canceling:
		        case JobState.Queued:
		        case JobState.Scheduled:
		        case JobState.Processing:
		            Console.WriteLine("Please wait...\n");
		            break;
		        case JobState.Canceled:
		        case JobState.Error:
		
		            // Cast sender as a job.
		            IJob job = (IJob)sender;
		
		            // Display or log error details as needed.
		            break;
		        default:
		            break;
		    }
		}
		
		
		private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
		{
		    var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
		    ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();
		
		    if (processor == null)
		        throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));
		
		    return processor;
		}


##Media Services のラーニング パス

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

##関連項目 

[Media Encoder Standard と .NET を使用してサムネイルを生成する方法](media-services-dotnet-generate-thumbnail-with-mes.md) [Media Services でのエンコーディングの概要](media-services-encode-asset.md)

<!---HONumber=AcomDC_0921_2016-->