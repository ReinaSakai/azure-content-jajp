<properties 
	pageTitle=".NET SDK を使用して資産配信ポリシーを構成する | Microsoft Azure" 
	description="このトピックでは、Azure Media Services .NET SDK を利用し、さまざまな資産配信ポリシーを構成する方法を示します。" 
	services="media-services" 
	documentationCenter="" 
	authors="Mingfeiy" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="dotnet" 
	ms.topic="article" 
	ms.date="09/19/2016"
	ms.author="juliako;mingfeiy"/>

#.NET SDK を使用して資産配信ポリシーを構成する
[AZURE.INCLUDE [media-services-selector-asset-delivery-policy](../../includes/media-services-selector-asset-delivery-policy.md)]

##Overview

Media Services で暗号化した資産を配信する場合、コンテンツ配信ワークフローの手順の 1 つとして、資産の配信ポリシーを構成します。資産の配信ポリシーは、資産を配信する方法、つまりどのストリーミング プロトコルで資産を動的パッケージングするか (例 : MPEG DASH、HLS、スムーズ ストリーミング、またはすべて)、資産を動的に暗号化するかどうか、どの暗号化方法を使用するか (エンベロープ暗号化または共通暗号化) を Media Services に示します。

このトピックでは、アセットの配信ポリシーを作成して構成する理由と方法をご説明します。

>[AZURE.NOTE]動的パッケージ化と動的暗号化を使用するには、少なくとも 1 つのスケール ユニット (ストリーミング ユニットとも呼ばれる) が存在している必要があります。詳細については、「[Media Services の規模の設定方法](media-services-portal-manage-streaming-endpoints.md)」をご覧ください。
>
>また、資産には、一連のアダプティブ ビットレート MP4、またはアダプティブ ビットレート スムーズ ストリーミング ファイルが含まれている必要があります。

1 つの資産にはさまざまなポリシーを適用できます。たとえば、スムーズ ストリーミングに PlayReady 暗号化を適用し、MPEG DASH と HLS に AES エンベロープ暗号化を適用できます。配信ポリシーで定義されていないプロトコル (たとえば、プロトコルとして HLS のみを指定する 1 つのポリシーを追加した場合) は、ストリーミングからブロックされます。ただし、資産配信ポリシーをまったく定義していない場合は例外となります。この場合、すべてのプロトコルが平文で許可されます。

ストレージ暗号化資産を配信する場合は、資産の配信ポリシーを構成する必要があります。資産をストリームするには、ストリーミング サーバーでストレージ暗号化を解除し、指定された配信ポリシーを使用してコンテンツをストリームする必要があります。たとえば、Advanced Encryption Standard (AES) エンベロープ暗号化キーを使用して暗号化された資産を配信するには、ポリシーの種類を **DynamicEnvelopeEncryption** に設定します。ストレージ暗号化を解除して資産を平文でストリームするには、ポリシーの種類を **NoDynamicEncryption** に設定します。これらのポリシーの種類を構成する例を次に示します。

資産の配信ポリシーの構成方法に応じて、動的パッケージングと動的暗号化を実行し、次のストリーミング プロトコル (スムーズ ストリーミング、HLS、MPEG DASH、HDS ストリーム) を使用してストリームを行うことができます。

次の一覧に、スムース、HLS、DASH、HDS でストリームを行うために使用するフォーマットを示します。

スムーズ ストリーミング:

{ストリーミング エンドポイント名-Media Services アカウント名}.streaming.mediaservices.windows.net/{ロケーター ID}/{ファイル名}.ism/Manifest

HLS:

{ストリーミング エンドポイント名-Media Services アカウント名}.streaming.mediaservices.windows.net/{ロケーター ID}/{ファイル名}.ism/Manifest(format=m3u8-aapl)

MPEG DASH

{ストリーミング エンドポイント名-Media Services アカウント名}.streaming.mediaservices.windows.net/{ロケーター ID}/{ファイル名}.ism/Manifest(format=mpd-time-csf)

HDS

{ストリーミング エンドポイント名-Media Services アカウント名}.streaming.mediaservices.windows.net/{ロケーター ID}/{ファイル名}.ism/Manifest(format=f4m-f4f)

資産を発行し、ストリーミング URL を構築する手順については、「[Build a streaming URL (ストリーミング URL の構築)](media-services-deliver-streaming-content.md)」をご覧ください。

##考慮事項

- 資産に対して OnDemand (ストリーミング) ロケーターが存在するときは、資産に関連付けられている AssetDeliveryPolicy を削除することはできません。ポリシーを削除する前に、資産からポリシーを削除することをお勧めします。
- 資産配信ポリシーが設定されていない場合、ストレージ暗号化資産のストリーミング ロケーターは作成できません。資産がストレージ暗号化資産でない場合はロケーターを作成でき、資産配信ポリシーのない暗号化されていない資産がストリーミングされます。
- 1 つの資産に複数の資産配信ポリシーを関連付けることができますが、特定の AssetDeliveryProtocol を処理する方法は 1 つだけ指定できます。つまり、AssetDeliveryProtocol.SmoothStreaming プロトコルを指定する 2 つの配信ポリシーをリンクしようとすると、エラーが発生します。これは、クライアントが Smooth Streaming 要求を行ったときにどのポリシーを適用するか、システムがわからないためです。
- 既存のストリーミング ロケーターを持つ資産が存在する場合、その資産に新しいポリシーをリンクすることはできません (資産から既存のポリシーのリンクを解除するか、資産に関連付けられている配信ポリシーを更新できます)。先にストリーミング ロケーターを削除し、ポリシーを調整した後、ストリーミング ロケーターを再作成する必要があります。ストリーミング ロケーターを再作成するときに同じ locatorId を使用できますが、コンテンツが最初の CDN またはダウンストリーム CDN によってキャッシュされる可能性があるため、クライアントで問題が発生しないことを確認する必要があります。


##資産の配信ポリシーを解除する

次の **ConfigureClearAssetDeliveryPolicy** メソッドでは、動的暗号化を適用せず、MPEG DASH、HLS、スムーズ ストリーミングのいずれかのプロトコルでストリームを配信することを指定します。ストレージで暗号化された資産にこのポリシーを適用する場合があります。

AssetDeliveryPolicy を作成する際に指定できる値については、[AssetDeliveryPolicy を定義するときに使用される種類](#types)セクションをご覧ください。

static public void ConfigureClearAssetDeliveryPolicy(IAsset asset) { IAssetDeliveryPolicy policy = \_context.AssetDeliveryPolicies.Create("Clear Policy", AssetDeliveryPolicyType.NoDynamicEncryption, AssetDeliveryProtocol.HLS | AssetDeliveryProtocol.SmoothStreaming | AssetDeliveryProtocol.Dash, null);

asset.DeliveryPolicies.Add(policy); }

##DynamicCommonEncryption 資産の配信ポリシー


次の **CreateAssetDeliveryPolicy** メソッドでは、動的で一般的な暗号化 (**DynamicCommonEncryption**) をスムーズ ストリーミング プロトコルに適用するよう構成された **AssetDeliveryPolicy** (他のプロトコルはストリーミングからブロック) を作成します。メソッドには、**Asset** (配信ポリシーを適用する資産) と **IContentKey** (**CommonEncryption** タイプのコンテンツ キーの 2 つのパラメーターがあります。詳細については、「[Creating a content key (コンテンツ キーの作成)](media-services-dotnet-create-contentkey.md#common_contentkey)」をご覧ください)。

AssetDeliveryPolicy を作成する際に指定できる値については、[AssetDeliveryPolicy を定義するときに使用される種類](#types)セクションをご覧ください。


static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key) { Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);

Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration = new Dictionary<AssetDeliveryPolicyConfigurationKey, string> { {AssetDeliveryPolicyConfigurationKey.PlayReadyLicenseAcquisitionUrl, acquisitionUrl.ToString()}, };

        var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryption,
            AssetDeliveryProtocol.SmoothStreaming,
            assetDeliveryPolicyConfiguration);

        // Add AssetDelivery Policy to the asset
        asset.DeliveryPolicies.Add(assetDeliveryPolicy);

        Console.WriteLine();
        Console.WriteLine("Adding Asset Delivery Policy: " +
            assetDeliveryPolicy.AssetDeliveryPolicyType);
    }

Azure Media Services では、Widevine による暗号化を追加することもできます。次の例は、PlayReady と Widevine の両方を資産の配信ポリシーに追加する方法を示しています。


    static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
    {
    	// Get the PlayReady license service URL.
        Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);
        

        // GetKeyDeliveryUrl for Widevine attaches the KID to the URL.
        // For example: https://amsaccount1.keydelivery.mediaservices.windows.net/Widevine/?KID=268a6dcb-18c8-4648-8c95-f46429e4927c.  
        // The WidevineBaseLicenseAcquisitionUrl (used below) also tells Dynamaic Encryption 
        // to append /? KID =< keyId > to the end of the url when creating the manifest.
        // As a result Widevine license acquisition URL will have KID appended twice, 
        // so we need to remove the KID that in the URL when we call GetKeyDeliveryUrl.

        Uri widevineUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.Widevine);
        UriBuilder uriBuilder = new UriBuilder(widevineUrl);
        uriBuilder.Query = String.Empty;
        widevineUrl = uriBuilder.Uri;

        Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
        {
            {AssetDeliveryPolicyConfigurationKey.PlayReadyLicenseAcquisitionUrl, acquisitionUrl.ToString()},
            {AssetDeliveryPolicyConfigurationKey.WidevineLicenseAcquisitionUrl, widevineUrl.ToString()}

        };

        var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryption,
            AssetDeliveryProtocol.Dash,
            assetDeliveryPolicyConfiguration);


        // Add AssetDelivery Policy to the asset
        asset.DeliveryPolicies.Add(assetDeliveryPolicy);

    }

>[AZURE.NOTE]Widevine を使用して暗号化する場合、配信は DASH でのみ実行できます。資産配信プロトコルには必ず DASH を指定してください。


##DynamicEnvelopeEncryption 資産の配信ポリシー 

次の **CreateAssetDeliveryPolicy** メソッドでは、動的エンベロープ暗号化 (**DynamicEnvelopeEncryption**) を Smooth Streaming、HLS、DASH の各プロトコルに適用するように構成された **AssetDeliveryPolicy** を作成します (指定していないプロトコルはストリーミングからブロックされます)。メソッドには、**Asset** (配信ポリシーを適用する資産) と **IContentKey** (**EnvelopeEncryption** タイプのコンテンツ キーの 2 つのパラメーターがあります。詳細については、[コンテンツ キーの作成](media-services-dotnet-create-contentkey.md#envelope_contentkey)に関するページを参照してください)。


AssetDeliveryPolicy を作成する際に指定できる値については、[AssetDeliveryPolicy を定義するときに使用される種類](#types)セクションをご覧ください。

    private static void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
    {
        
        //  Get the Key Delivery Base Url by removing the Query parameter.  The Dynamic Encryption service will
        //  automatically add the correct key identifier to the url when it generates the Envelope encrypted content
        //  manifest.  Omitting the IV will also cause the Dynamice Encryption service to generate a deterministic
        //  IV for the content automatically.  By using the EnvelopeBaseKeyAcquisitionUrl and omitting the IV, this
        //  allows the AssetDelivery policy to be reused by more than one asset.
        //
        Uri keyAcquisitionUri = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.BaselineHttp);
        UriBuilder uriBuilder = new UriBuilder(keyAcquisitionUri);
        uriBuilder.Query = String.Empty;
        keyAcquisitionUri = uriBuilder.Uri;

        // The following policy configuration specifies: 
        //   key url that will have KID=<Guid> appended to the envelope and
        //   the Initialization Vector (IV) to use for the envelope encryption.
        Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string> 
        {
            {AssetDeliveryPolicyConfigurationKey.EnvelopeBaseKeyAcquisitionUrl, keyAcquisitionUri.ToString()},
        };

        IAssetDeliveryPolicy assetDeliveryPolicy =
            _context.AssetDeliveryPolicies.Create(
                        "AssetDeliveryPolicy",
                        AssetDeliveryPolicyType.DynamicEnvelopeEncryption,
                        AssetDeliveryProtocol.SmoothStreaming | AssetDeliveryProtocol.HLS | AssetDeliveryProtocol.Dash,
                        assetDeliveryPolicyConfiguration);

        // Add AssetDelivery Policy to the asset
        asset.DeliveryPolicies.Add(assetDeliveryPolicy);

        Console.WriteLine();
        Console.WriteLine("Adding Asset Delivery Policy: " + assetDeliveryPolicy.AssetDeliveryPolicyType);
    }


##<a id="types"></a>AssetDeliveryPolicy の定義に使用する種類

###<a id="AssetDeliveryProtocol"></a>AssetDeliveryProtocol 

    /// <summary>
    /// Delivery protocol for an asset delivery policy.
    /// </summary>
    [Flags]
    public enum AssetDeliveryProtocol
    {
        /// <summary>
        /// No protocols.
        /// </summary>
        None = 0x0,

        /// <summary>
        /// Smooth streaming protocol.
        /// </summary>
        SmoothStreaming = 0x1,

        /// <summary>
        /// MPEG Dynamic Adaptive Streaming over HTTP (DASH)
        /// </summary>
        Dash = 0x2,

        /// <summary>
        /// Apple HTTP Live Streaming protocol.
        /// </summary>
        HLS = 0x4,

        /// <summary>
        /// Adobe HTTP Dynamic Streaming (HDS)
        /// </summary>
        Hds = 0x8,

        /// <summary>
        /// Include all protocols.
        /// </summary>
        All = 0xFFFF
    }

###<a id="AssetDeliveryPolicyType"></a>AssetDeliveryPolicyType

    /// <summary>
    /// Policy type for dynamic encryption of assets.
    /// </summary>
    public enum AssetDeliveryPolicyType
    {
        /// <summary>
        /// Delivery Policy Type not set.  An invalid value.
        /// </summary>
        None,

        /// <summary>
        /// The Asset should not be delivered via this AssetDeliveryProtocol. 
        /// </summary>
        Blocked, 

        /// <summary>
        /// Do not apply dynamic encryption to the asset.
        /// </summary>
        /// 
        NoDynamicEncryption,  

        /// <summary>
        /// Apply Dynamic Envelope encryption.
        /// </summary>
        DynamicEnvelopeEncryption,

        /// <summary>
        /// Apply Dynamic Common encryption.
        /// </summary>
        DynamicCommonEncryption
    }

###<a id="ContentKeyDeliveryType"></a>ContentKeyDeliveryType

    /// <summary>
    /// Delivery method of the content key to the client.
    /// </summary>
    public enum ContentKeyDeliveryType
    {
        /// <summary>
        /// None.
        /// </summary>
        None = 0,

        /// <summary>
        /// Use PlayReady License acquistion protocol
        /// </summary>
        PlayReadyLicense = 1,

        /// <summary>
        /// Use MPEG Baseline HTTP key protocol.
        /// </summary>
        BaselineHttp = 2,

        /// <summary>
        /// Use Widevine License acquistion protocol
        ///
        </summary>
        Widevine = 3

    }

###<a id="AssetDeliveryPolicyConfigurationKey"></a>AssetDeliveryPolicyConfigurationKey

    /// <summary>
    /// Keys used to get specific configuration for an asset delivery policy.
    /// </summary>
    public enum AssetDeliveryPolicyConfigurationKey
    {
        /// <summary>
        /// No policies.
        /// </summary>
        None,

        /// <summary>
        /// Exact Envelope key URL.
        /// </summary>
        EnvelopeKeyAcquisitionUrl,

        /// <summary>
        /// Base key url that will have KID=<Guid> appended for Envelope.
        /// </summary>
        EnvelopeBaseKeyAcquisitionUrl,
        
        /// <summary>
        /// The initialization vector to use for envelope encryption in Base64 format.
        /// </summary>
        EnvelopeEncryptionIVAsBase64,

        /// <summary>
        /// The PlayReady License Acquisition Url to use for common encryption.
        /// </summary>
        PlayReadyLicenseAcquisitionUrl,

        /// <summary>
        /// The PlayReady Custom Attributes to add to the PlayReady Content Header
        /// </summary>
        PlayReadyCustomAttributes,

        /// <summary>
        /// The initialization vector to use for envelope encryption.
        /// </summary>
        EnvelopeEncryptionIV,

        /// <summary>
        /// Widevine DRM acquisition url
        /// </summary>
        WidevineLicenseAcquisitionUrl
    }

##Media Services のラーニング パス

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

<!---HONumber=AcomDC_0921_2016-->