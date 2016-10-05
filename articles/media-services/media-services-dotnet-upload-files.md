<properties 
	pageTitle=".NET を使用した Media Services アカウントへのファイルのアップロード | Microsoft Azure" 
	description="資産を作成し、アップロードすることによって、Media Services にメディア コンテンツを取得する方法について説明します。" 
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
	ms.author="juliako"/>



# .NET を使用した Media Services アカウントへのファイルのアップロード

 > [AZURE.SELECTOR]
 - [.NET](media-services-dotnet-upload-files.md)
 - [REST ()](media-services-rest-upload-files.md)
 - [ポータル](media-services-portal-upload-files.md)

Media Services で、デジタル ファイルを資産にアップロードし (取り込み) ます。**Asset** エンティティには、ビデオ、オーディオ、画像、縮小表示のコレクション、テキスト トラック、クローズド キャプション ファイル (各ファイルのメタデータを含む) を追加できます。 ファイルをアップロードすると、クラウドにコンテンツが安全に保存され、処理したりストリーミングしたりできるようになります。

資産内のこれらのファイルを**資産ファイル**といいます。**AssetFile** インスタンスと実際のメディア ファイルは、別々の 2 つのオブジェクトです。AssetFile インスタンスには、メディア ファイルに関するメタデータが含まれており、メディア ファイルには実際のメディア コンテンツが含まれています。

>[AZURE.NOTE]資産ファイル名を選択するときは、次の考慮事項が適用されます。
>
>- Media Services は、ストリーミング コンテンツ (例: http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters.) の URL を構築する際に、IAssetFile.Name プロパティの値を使用します。このため、パーセントエンコーディングは利用できません。**Name** プロパティの値には、[パーセント エンコーディング予約文字](http://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters) !*'();:@&=+$,/?%#" は使用できません。また、ファイル名拡張子で使用できる "." は 1 つのみです。
>
>- 名前は 260 文字以内である必要があります。

資産を作成する際には、次の暗号化オプションを指定できます。

- **None**: 暗号化は使用されません。これが既定値です。このオプションを使用した場合、送信経路上とストレージ内のいずれにおいてもコンテンツが保護されないので注意してください。プログレッシブ ダウンロードを使用して MP4 を配信する場合はこのオプションを使用します。
- **CommonEncryption**: 既に Common Encryption や PlayReady DRM で暗号化されて保護されているコンテンツ (PlayReady DRM で保護されたスムーズ ストリーミングなど) をアップロードする場合は、このオプションを使用します。
- **EnvelopeEncrypted**: AES で暗号化された HLS をアップロードする場合はこのオプションを使用します。この場合ファイルは、Transform Manager によってあらかじめエンコードされて暗号化されている必要があります。
- **StorageEncrypted**: ローカルで AES-256 ビット暗号化を使用し、平文のコンテンツを暗号化したうえで、それを Azure Storage にアップロードします。アップロードされたデータは、暗号化された状態で保存されます。StorageEncrypted で保護された資産は、エンコーディングの前に自動的に暗号化が解除され、暗号化されたファイル システムに配置されます。その後、必要に応じて再度暗号化を適用して、新しい出力資産として再びアップロードできます。StorageEncrypted の主な目的は、高品質の入力メディア ファイルを強力な暗号化によって保護したうえでディスクに保存するというニーズに応えることです。

	Media Services では、Digital Rights Manager (DRM) のようにネットワーク経由ではなく、資産のオンディスクでのストレージ暗号化を提供します。

	資産がストレージで暗号化されている場合は、資産配信ポリシーを構成する必要があります。詳細については、「[方法: アセットの配信ポリシーを構成する](media-services-dotnet-configure-asset-delivery-policy.md)」を参照してください。

資産を **CommonEncrypted** オプションか **EnvelopeEncypted** オプションで暗号化することを指定した場合、資産を **ContentKey** に関連付ける必要があります。詳細については、「[How to create a ContentKey (ContentKey の作成方法)](media-services-dotnet-create-contentkey.md)」をご覧ください

資産を **StorageEncrypted** オプションで暗号化することを指定した場合、Media Services SDK for .NET によって、資産の **StorateEncrypted** の **ContentKey** が作成されます。


このトピックでは、Media Services .NET SDK と Media Services .NET SDK Extensions を使用してファイルを Media Services 資産にアップロードする方法を説明します。

 
## Media Services .NET SDK を使用して 1 つのファイルをアップロードする 

以下のサンプル コードでは、.NET SDK を使用して、次のタスクを実行します。

- 空の資産を作成します。
- 資産に関連付ける AssetFile インスタンスを作成します。
- 資産へのアクセス許可とアクセス期間を定義する AccessPolicy インスタンスを作成します。
- 資産へのアクセスを提供する Locator インスタンスを作成します。
- 単一のメディア ファイルを Media Services にアップロードします。

		
		static public IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string singleFilePath)
		{
            if (!File.Exists(singleFilePath))
            {
                Console.WriteLine("File does not exist.");
                return null;
            }

            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
            IAsset inputAsset = _context.Assets.Create(assetName, assetCreationOptions); 

            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            var policy = _context.AccessPolicies.Create(
                                    assetName,
                                    TimeSpan.FromDays(30),
                                    AccessPermissions.Write | AccessPermissions.List);

            var locator = _context.Locators.CreateLocator(LocatorType.Sas, inputAsset, policy);

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            locator.Delete();
            policy.Delete();

            return inputAsset;
		}

##Media Services .NET SDK を使用して複数のファイルをアップロードする 

次のコードは、資産を作成して複数のファイルをアップロードする方法を示します。

このコードは、次の処理を実行します。
	
- 	前の手順で定義した CreateEmptyAsset メソッドを使用して空の資産を作成します。
 	
- 	資産へのアクセス許可とアクセス期間を定義する **AccessPolicy** インスタンスを作成します。
 	
- 	資産へのアクセスを提供する **Locator** インスタンスを作成します。
 	
- 	**BlobTransferClient** インスタンスを作成します。この型は、Azure BLOB で動作するクライアントを表します。この例では、クライアントを使用して、アップロードの進行状況を監視します。
 	
- 	指定したディレクトリ内のファイルの一覧を取得し、各ファイルの **AssetFile** インスタンスを作成します。
 	
- 	**UploadAsync** メソッドを使用して、Media Services にファイルをアップロードします。
 	
>[AZURE.NOTE] 非ブロッキング操作としてメソッドを呼び出し、複数のファイルを並列的にアップロードできるようにするため、UploadAsync メソッドを使用します。
 	
 	
        static public IAsset CreateAssetAndUploadMultipleFiles(AssetCreationOptions assetCreationOptions, string folderPath)
        {
            var assetName = "UploadMultipleFiles_" + DateTime.UtcNow.ToString();

            IAsset asset = _context.Assets.Create(assetName, assetCreationOptions);

            var accessPolicy = _context.AccessPolicies.Create(assetName, TimeSpan.FromDays(30),
                                                                AccessPermissions.Write | AccessPermissions.List);

            var locator = _context.Locators.CreateLocator(LocatorType.Sas, asset, accessPolicy);

            var blobTransferClient = new BlobTransferClient();
            blobTransferClient.NumberOfConcurrentTransfers = 20;
            blobTransferClient.ParallelTransferThreadCount = 20;

            blobTransferClient.TransferProgressChanged += blobTransferClient_TransferProgressChanged;

            var filePaths = Directory.EnumerateFiles(folderPath);

            Console.WriteLine("There are {0} files in {1}", filePaths.Count(), folderPath);

            if (!filePaths.Any())
            {
                throw new FileNotFoundException(String.Format("No files in directory, check folderPath: {0}", folderPath));
            }

            var uploadTasks = new List<Task>();
            foreach (var filePath in filePaths)
            {
                var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
                Console.WriteLine("Created assetFile {0}", assetFile.Name);

                // It is recommended to validate AccestFiles before upload. 
                Console.WriteLine("Start uploading of {0}", assetFile.Name);
                uploadTasks.Add(assetFile.UploadAsync(filePath, blobTransferClient, locator, CancellationToken.None));
            }

            Task.WaitAll(uploadTasks.ToArray());
            Console.WriteLine("Done uploading the files");

            blobTransferClient.TransferProgressChanged -= blobTransferClient_TransferProgressChanged;

            locator.Delete();
            accessPolicy.Delete();

            return asset;
        }
	
	static void  blobTransferClient_TransferProgressChanged(object sender, BlobTransferProgressChangedEventArgs e)
	{
	    if (e.ProgressPercentage > 4) // Avoid startup jitter, as the upload tasks are added.
	    {
	        Console.WriteLine("{0}% upload competed for {1}.", e.ProgressPercentage, e.LocalFile);
	    }
	}



アップロードする資産の数が多い場合は、次の点を考慮してください。

- **CloudMediaContext** オブジェクトは、スレッドごとに作成してください。**CloudMediaContext** クラスはスレッド セーフではありません。
 
- NumberOfConcurrentTransfers を既定値の 2 から、たとえば 5 のようなより大きな値に増やしてください。このプロパティの設定は、**CloudMediaContext** のすべてのインスタンスに影響を与えます。
 
- ParallelTransferThreadCount は、既定値の 10 のままにしてください。
 
##<a id="ingest_in_bulk"></a>Media Services .NET SDK を使用したアセットの一括取り込み 

サイズの大きい資産ファイルのアップロードは、資産の作成時に、ボトルネックになることがあります。資産を一括して取り込む "一括取り込み" の場合、アップロード プロセスから資産の作成を切り離すことが必要です。一括取り込みを行うには、資産とその関連ファイルを記述するマニフェスト (IngestManifest) を作成します。その後で、お好みのアップロード方法で、マニフェストの BLOB コンテナーに、関連ファイルをアップロードします。マニフェストに関連付けられている BLOB コンテナーは、Microsoft Azure Media Services によって監視されます。ファイルが BLOB コンテナーにアップロードされると、Microsoft Azure Media Services は、マニフェスト (IngestManifestAsset) の資産の構成に基づいてア資産の作成を完了させます。


新しい IngestManifest を作成するには、CloudMediaContext の IngestManifests コレクションで公開されている Create メソッドを呼び出します。指定されたマニフェスト名で、新しい IngestManifest が作成されます。

	IIngestManifest manifest = context.IngestManifests.Create(name);

この一括 IngestManifest に関連付ける資産を作成します。一括取り込みの対象となる資産に必要な暗号化オプションを構成してください。

	// Create the assets that will be associated with this bulk ingest manifest
	IAsset destAsset1 = _context.Assets.Create(name + "_asset_1", AssetCreationOptions.None);
	IAsset destAsset2 = _context.Assets.Create(name + "_asset_2", AssetCreationOptions.None);

IngestManifestAsset は、資産を、一括取り込みのための一括 IngestManifest に関連付けます。また、各資産を構成する AssetFiles を関連付けます。IngestManifestAsset を作成するには、サーバー コンテキストの Create メソッドを使用します。

以下の例では、先に作成した 2 つの資産を一括取り込みマニフェストに関連付ける 2 つの新しい IngestManifestAssets を追加しています。また、各 IngestManifestAsset は一括取り込み中に、各資産に対してアップロードされる一連のファイルを関連付けます。

	string filename1 = _singleInputMp4Path;
	string filename2 = _primaryFilePath;
	string filename3 = _singleInputFilePath;
	
	IIngestManifestAsset bulkAsset1 =  manifest.IngestManifestAssets.Create(destAsset1, new[] { filename1 });
	IIngestManifestAsset bulkAsset2 =  manifest.IngestManifestAssets.Create(destAsset2, new[] { filename2, filename3 });
	
クライアント アプリケーションには、IngestManifest の **IIngestManifest.BlobStorageUriForUpload** プロパティによって提供された BLOB ストレージ コンテナーの URI に資産ファイルをアップロードする機能のある任意の高速クライアント アプリケーションを使用できます。有名な高速アップロードサービスには、[Aspera On Demand for Azure アプリケーション](https://datamarket.azure.com/application/2cdbc511-cb12-4715-9871-c7e7fbbb82a6)などがあります。また、次のコード例のようにコードを記述して、資産ファイルをアップロードできます。
	
	static void UploadBlobFile(string destBlobURI, string filename)
	{
	    Task copytask = new Task(() =>
	    {
	        var storageaccount = new CloudStorageAccount(new StorageCredentials(_storageAccountName, _storageAccountKey), true);
	        CloudBlobClient blobClient = storageaccount.CreateCloudBlobClient();
	        CloudBlobContainer blobContainer = blobClient.GetContainerReference(destBlobURI);
	
	        string[] splitfilename = filename.Split('\\');
	        var blob = blobContainer.GetBlockBlobReference(splitfilename[splitfilename.Length - 1]);
	
	        using (var stream = System.IO.File.OpenRead(filename))
	            blob.UploadFromStream(stream);
	
	        lock (consoleWriteLock)
	        {
	            Console.WriteLine("Upload for {0} completed.", filename);
	        }
	    });
	
	    copytask.Start();
	}

次のコード例には、このトピックで使用されているサンプルのアセット ファイルをアップロードするためのコードが表示されています。
	
	UploadBlobFile(manifest.BlobStorageUriForUpload, filename1);
	UploadBlobFile(manifest.BlobStorageUriForUpload, filename2);
	UploadBlobFile(manifest.BlobStorageUriForUpload, filename3);
	

**IngestManifest** の Statistics プロパティをポーリングすることによって、**IngestManifest** に関連付けられているすべての資産の一括インジェストの進行状況を確認できます。進行状況の情報を更新するためには、Statistics プロパティをポーリングするごとに、新しい **CloudMediaContext** を使用する必要があります。

次の例では、IngestManifest を、その **Id** でポーリングしています。
	
	static void MonitorBulkManifest(string manifestID)
	{
	   bool bContinue = true;
	   while (bContinue)
	   {
	      CloudMediaContext context = GetContext();
	      IIngestManifest manifest = context.IngestManifests.Where(m => m.Id == manifestID).FirstOrDefault();
	
	      if (manifest != null)
	      {
	         lock(consoleWriteLock)
	         {
	            Console.WriteLine("\nWaiting on all file uploads.");
	            Console.WriteLine("PendingFilesCount  : {0}", manifest.Statistics.PendingFilesCount);
	            Console.WriteLine("FinishedFilesCount : {0}", manifest.Statistics.FinishedFilesCount);
	            Console.WriteLine("{0}% complete.\n", (float)manifest.Statistics.FinishedFilesCount / (float)(manifest.Statistics.FinishedFilesCount + manifest.Statistics.PendingFilesCount) * 100);
	
	            if (manifest.Statistics.PendingFilesCount == 0)
	            {
	               Console.WriteLine("Completed\n");
	               bContinue = false;
	            }
	         }
	
	         if (manifest.Statistics.FinishedFilesCount < manifest.Statistics.PendingFilesCount)
	            Thread.Sleep(60000);
	      }
	      else // Manifest is null
	         bContinue = false;
	   }
	}
	


##.NET SDK Extensions を使用したファイルのアップロード 

次の例は、.NET SDK Extensions を使用して単一のファイルをアップロードする方法を示しています。この例では、**CreateFromFile** メソッドを使用していますが、非同期バージョン (**CreateFromFileAsync**) のメソッドも使用できます。**CreateFromFile** メソッドには、ファイル名、暗号化オプション、ファイルのアップロードの進行状況をレポートするためのコールバックを指定できます。


	static public IAsset UploadFile(string fileName, AssetCreationOptions options)
	{
	    IAsset inputAsset = _context.Assets.CreateFromFile(
	        fileName,
	        options,
	        (af, p) =>
	        {
	            Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
	        });
	
	    Console.WriteLine("Asset {0} created.", inputAsset.Id);
	
	    return inputAsset;
	}

次の例では、UploadFile 関数を呼び出し、資産作成オプションとしてストレージの暗号化を指定しています。


	var asset = UploadFile(@"C:\VideoFiles\BigBuckBunny.mp4", AssetCreationOptions.StorageEncrypted);


##Media Services のラーニング パス

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]


##次のステップ

これで、資産を Media Services にアップロードできました。次は、[メディア プロセッサを取得する方法][]に関するトピックに進んでください。

[メディア プロセッサを取得する方法]: media-services-get-media-processor.md
 

<!---HONumber=AcomDC_0921_2016-->