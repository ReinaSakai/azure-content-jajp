<properties 
	pageTitle="Media Services REST API を使用したコンテンツ キー承認ポリシーの構成 | Microsoft Azure" 
	description="Media Services REST API を利用し、コンテンツ キー承認ポリシーを構成する方法について説明します。" 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
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


#動的暗号化: コンテンツ キー承認ポリシーを構成する
[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../../includes/media-services-selector-content-key-auth-policy.md)]


##Overview

Microsoft Azure Media Services では、Advanced Encryption Standard (AES) (128 ビット暗号化キーを使用) と PlayReady または Widevine DRM を使用して (動的に) 暗号化されたコンテンツを配信できます。Media Services では、承認されたクライアントにキーと PlayReady/Widevine ライセンスを配信するためのサービスも提供しています。

Media Services で資産を暗号化する場合は、[こちら](media-services-rest-create-contentkey.md)の説明に従って暗号化キー (**CommonEncryption** か **EnvelopeEncryption**) を資産に関連付ける必要があります。また、このトピックでの説明に従って、キーの承認ポリシーを構成する必要があります。


プレーヤーがストリームを要求すると、Media Services は指定されたキーを使用して、AES か PlayReady でコンテンツを動的に暗号化します。ストリームの暗号化を解除するには、プレーヤーはキー配信サービスからキーを要求します。ユーザーのキーの取得が承認されているかどうかを判断するために、サービスはキーに指定した承認ポリシーを評価します。

Media Services では、キーを要求するユーザーを承認する複数の方法がサポートされています。コンテンツ キー承認ポリシーには、1 つまたは複数の承認制限 (**オープン**または**トークン**制限) を指定できます。トークン制限ポリシーには、STS (セキュリティ トークン サービス) によって発行されたトークンを含める必要があります。Media Services では、**Simple Web Tokens** ([SWT](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_2)) 形式と **JSON Web Token** (JWT) 形式のトークンがサポートされます。

Media Services では、Secure Token Services は提供されません。トークンを発行するには、カスタム STS を作成するか、Microsoft Azure ACS を活用できます。STS は、指定されたキーで署名されたトークンを作成し、トークン制限構成で指定した要求を発行するよう構成する必要があります (この記事の説明を参照)。Media Services のキー配信サービスは、トークンが有効で、トークン内の要求がコンテンツ キー向けに構成された要求と一致する場合、暗号化キーをクライアントに返します。

詳細については、次をご覧ください。

[JWT トークンの承認](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)

[Azure Media Services OWIN MVC ベースのアプリを Azure Active Directory と統合し、JWT 要求に基づいてコンテンツ キーの配信を制限する](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/)

[Azure ACS を使用してトークンを発行する](http://mingfeiy.com/acs-with-key-services)。

###いくつかの考慮事項が適用されます。

- 動的パッケージングや動的暗号化を使用するには、少なくとも 1 つのストリーミング予約ユニットが必要です。詳細については、「[Media Services の規模の設定方法](media-services-portal-manage-streaming-endpoints.md)」をご覧ください。
- 資産には、一連のアダプティブ ビットレート MP4 または アダプティブ ビットレート スムーズ ストリーミング ファイルが含まれている必要があります。詳細については、「[資産をエンコードする](media-services-encode-asset.md)」をご覧ください。
- **AssetCreationOptions.StorageEncrypted** オプションを使用して、資産をアップロードしてエンコードします。
- 複数のコンテンツ キーで同じポリシー構成を必要とする場合は、1 つの承認ポリシーを作成して、複数のコンテンツ キーに利用することを強くお勧めします。
- キー配信サービスでは、ContentKeyAuthorizationPolicy とそれに関連するオブジェクト (ポリシーのオプションと制限) を 15 分間キャッシュします。ContentKeyAuthorizationPolicy を作成して、"Token" 制限を使用するように指定した場合に、"Token" 制限をテストしてから、ポリシーを "Open" 制限に更新すると、ポリシーが "Open" バージョンのポリシーに切り替わるまで、約 15 分かかります。
- 資産の配信ポリシーを追加または更新する場合は、既存のロケーターを削除し (存在する場合)、新しいロケーターを作成する必要があります。
- 現在、HDS ストリーミング形式およびプログレッシブ ダウンロードは暗号化できません。


##AES-128 動的暗号化

>[AZURE.NOTE] Media Services REST API を使用する場合は、次のことに考慮します。
>
>Media Services でエンティティにアクセスするときは、HTTP 要求で特定のヘッダー フィールドと値を設定する必要があります。詳細については、「[Media Services REST API の概要](media-services-rest-how-to-use.md)」をご覧ください。

>https://media.windows.net に正常に接続すると、別の Media Services URI が指定された 301 リダイレクトが表示されます。「[Media Services REST API を使用して Media Services アカウントに接続する](media-services-rest-connect-programmatically.md)」で説明するとおり、続けて新しい URI を呼び出す必要があります。


###オープン制限

オープン制限とは、キーを要求するすべてのユーザーに、システムがキーを提供することを意味します。この制限は、テストに便利です。

次の例では、オープン承認ポリシーを作成し、それをコンテンツ キーに追加します。

####<a id="ContentKeyAuthorizationPolicies"></a>ContentKeyAuthorizationPolicies を作成する

要求:
		
	POST https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=bbbef702-e769-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423578086&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=lZlyQ2%2bvH73qtJsb42%2fH3xF7r7EvQFR3UXyezuDENFU%3d
	x-ms-version: 2.11
	x-ms-client-request-id: d732dbfa-54fc-474c-99d6-9b46a006f389
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 36
	
	{"Name":"Open Authorization Policy"}
	
応答:
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 211
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicies('nb%3Ackpid%3AUUID%3Adb4593da-f4d1-4cc5-a92a-d20eacbabee4')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: d732dbfa-54fc-474c-99d6-9b46a006f389
	request-id: aabfa731-e884-4bf3-8314-492b04747ac4
	x-ms-request-id: aabfa731-e884-4bf3-8314-492b04747ac4
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 08:25:56 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeyAuthorizationPolicies/@Element","Id":"nb:ckpid:UUID:db4593da-f4d1-4cc5-a92a-d20eacbabee4","Name":"Open Authorization Policy"}

####<a id="ContentKeyAuthorizationPolicyOptions"></a>ContentKeyAuthorizationPolicyOptions を作成する
	
要求:

	POST https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicyOptions HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 3.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=bbbef702-e769-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423580006&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=Ref3EsonGF7fUKCwGwGgiMnZitzIzsDOvvMTeVrVVPg%3d
	x-ms-version: 2.11
	x-ms-client-request-id: d225e357-e60e-4f42-add8-9d93aba1409a
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 168
	
	{"Name":"policy","KeyDeliveryType":2,"KeyDeliveryConfiguration":"","Restrictions":[{"Name":"HLS Open Authorization Policy","KeyRestrictionType":0,"Requirements":null}]}

応答:
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 349
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3A57829b17-1101-4797-919b-f816f4a007b7')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: d225e357-e60e-4f42-add8-9d93aba1409a
	request-id: 81bcad37-295b-431f-972f-b23f2e4172c9
	x-ms-request-id: 81bcad37-295b-431f-972f-b23f2e4172c9
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 08:56:40 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeyAuthorizationPolicyOptions/@Element","Id":"nb:ckpoid:UUID:57829b17-1101-4797-919b-f816f4a007b7","Name":"policy","KeyDeliveryType":2,"KeyDeliveryConfiguration":"","Restrictions":[{"Name":"HLS Open Authorization Policy","KeyRestrictionType":0,"Requirements":null}]}

####<a id="LinkContentKeyAuthorizationPoliciesWithOptions"></a>ContentKeyAuthorizationPolicies を Options にリンクする

要求:
	
	POST https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicies('nb%3Ackpid%3AUUID%3A0baa438b-8ac2-4c40-a53c-4d4722b78715')/$links/Options HTTP/1.1
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Content-Type: application/json
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423580006&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=Ref3EsonGF7fUKCwGwGgiMnZitzIzsDOvvMTeVrVVPg%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 9847f705-f2ca-4e95-a478-8f823dbbaa29
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 154
	
	{"uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3A57829b17-1101-4797-919b-f816f4a007b7')"}

応答:

	HTTP/1.1 204 No Content

####<a id="AddAuthorizationPolicyToKey"></a>承認ポリシーをコンテンツ キーに追加する

要求:

	PUT https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeys('nb%3Akid%3AUUID%3A2e6d36a7-a17c-4e9a-830d-eca23ad1a6f9') HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423581565&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=JiNSG3w6r2C0nIyfKvTZj1uPJGjuitD%2b0sbfZ%2b2JDZI%3d
	x-ms-version: 2.11
	x-ms-client-request-id: e613efff-cb6a-41b4-984a-f4f8fb6e76a4
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 78
	
	{"AuthorizationPolicyId":"nb:ckpid:UUID:c06cebb8-c4f0-4d1a-ba00-3273fb2bc3ad"}

応答:

	HTTP/1.1 204 No Content

###トークン制限

このセクションでは、コンテンツ キー承認ポリシーを作成し、それをコンテンツ キーに関連付ける方法について説明します。承認ポリシーには、ユーザーがキーを受け取ることを承認されているかどうかを判断するために、承認要求が満たす必要がある内容について記載されています (トークンの署名に使用されたキーが「検証キー」リストに含まれているなど)。

トークン制限オプションを構成するには、XML を使用してトークンの承認要件を記述する必要があります。トークン制限の構成 XML は、次の XML スキーマに準拠する必要があります。

####<a id="schema"></a>トークン制限スキーマ
	
	<?xml version="1.0" encoding="utf-8"?>
	<xs:schema xmlns:tns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" elementFormDefault="qualified" targetNamespace="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" xmlns:xs="http://www.w3.org/2001/XMLSchema">
	  <xs:complexType name="TokenClaim">
	    <xs:sequence>
	      <xs:element name="ClaimType" nillable="true" type="xs:string" />
	      <xs:element minOccurs="0" name="ClaimValue" nillable="true" type="xs:string" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="TokenClaim" nillable="true" type="tns:TokenClaim" />
	  <xs:complexType name="TokenRestrictionTemplate">
	    <xs:sequence>
	      <xs:element minOccurs="0" name="AlternateVerificationKeys" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
	      <xs:element name="Audience" nillable="true" type="xs:anyURI" />
	      <xs:element name="Issuer" nillable="true" type="xs:anyURI" />
	      <xs:element name="PrimaryVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
	      <xs:element minOccurs="0" name="RequiredClaims" nillable="true" type="tns:ArrayOfTokenClaim" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="TokenRestrictionTemplate" nillable="true" type="tns:TokenRestrictionTemplate" />
	  <xs:complexType name="ArrayOfTokenVerificationKey">
	    <xs:sequence>
	      <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="ArrayOfTokenVerificationKey" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
	  <xs:complexType name="TokenVerificationKey">
	    <xs:sequence />
	  </xs:complexType>
	  <xs:element name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
	  <xs:complexType name="ArrayOfTokenClaim">
	    <xs:sequence>
	      <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenClaim" nillable="true" type="tns:TokenClaim" />
	    </xs:sequence>
	  </xs:complexType>
	  <xs:element name="ArrayOfTokenClaim" nillable="true" type="tns:ArrayOfTokenClaim" />
	  <xs:complexType name="SymmetricVerificationKey">
	    <xs:complexContent mixed="false">
	      <xs:extension base="tns:TokenVerificationKey">
	        <xs:sequence>
	          <xs:element name="KeyValue" nillable="true" type="xs:base64Binary" />
	        </xs:sequence>
	      </xs:extension>
	    </xs:complexContent>
	  </xs:complexType>
	  <xs:element name="SymmetricVerificationKey" nillable="true" type="tns:SymmetricVerificationKey" />
	</xs:schema>

**トークン**制限ポリシーを構成する際は、プライマリ**検証キー**、**発行者**、**対象ユーザー**の各パラメーターを指定する必要があります。**プライマリ検証キー**には、トークンの署名に使用されたキーが含まれ、**発行者**は、トークンを発行するセキュリティ トークン サービスです。**対象ユーザー** (**スコープ**とも呼ばれる) には、トークンの目的か、トークンがアクセスを承認するリソースが記述されます。Media Services キー配信サービスでは、トークン内のこれらの値がテンプレート内の値と一致することが検証されます。

次の例では、トークン制限を含む承認ポリシーを作成します。この例では、クライアントが署名キー (VerificationKey)、トークン発行者、必要な要求を含むトークンを提示する必要があります。
	
###ContentKeyAuthorizationPolicies を作成する

[ここ](#ContentKeyAuthorizationPolicies)を参照して "トークン制限ポリシー" を作成します。


###ContentKeyAuthorizationPolicyOptions を作成する

要求:
	
	POST https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicyOptions HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 3.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=bbbef702-e769-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423580720&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=5LsNu%2b0D4eD3UOP3BviTLDkUjaErdUx0ekJ8402xidQ%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 2643d836-bfe7-438e-9ba2-bc6ff28e4a53
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 1079
	
	{"Name":"Token option for HLS","KeyDeliveryType":2,"KeyDeliveryConfiguration":null,"Restrictions":[{"Name":"Token Authorization Policy","KeyRestrictionType":1,"Requirements":"<TokenRestrictionTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1"><AlternateVerificationKeys><TokenVerificationKey i:type="SymmetricVerificationKey"><KeyValue>BklyAFiPTQsuJNKriQJBZHYaKM2CkCTDQX2bw9sMYuvEC9sjW0W7GUIBygQL/+POEeUqCYPnmEU2g0o1GW2Oqg==</KeyValue></TokenVerificationKey></AlternateVerificationKeys><Audience>urn:test</Audience><Issuer>http://testacs.com/</Issuer><PrimaryVerificationKey i:type="SymmetricVerificationKey"><KeyValue>E5BUHiN4vBdzUzdP0IWaHFMMU3D1uRZgF16TOhSfwwHGSw+Kbf0XqsHzEIYk11M372viB9vbiacsdcQksA0ftw==</KeyValue></PrimaryVerificationKey><RequiredClaims><TokenClaim><ClaimType>urn:microsoft:azure:mediaservices:contentkeyidentifier</ClaimType><ClaimValue i:nil="true" /></TokenClaim></RequiredClaims><TokenType>SWT</TokenType></TokenRestrictionTemplate>"}]}

応答:
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 1260
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3Ae1ef6145-46e8-4ee6-9756-b1cf96328c23')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 2643d836-bfe7-438e-9ba2-bc6ff28e4a53
	request-id: 2310b716-aeaa-421e-913e-3ce2f6f685ca
	x-ms-request-id: 2310b716-aeaa-421e-913e-3ce2f6f685ca
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 09:10:37 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeyAuthorizationPolicyOptions/@Element","Id":"nb:ckpoid:UUID:e1ef6145-46e8-4ee6-9756-b1cf96328c23","Name":"Token option for HLS","KeyDeliveryType":2,"KeyDeliveryConfiguration":null,"Restrictions":[{"Name":"Token Authorization Policy","KeyRestrictionType":1,"Requirements":"<TokenRestrictionTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1"><AlternateVerificationKeys><TokenVerificationKey i:type="SymmetricVerificationKey"><KeyValue>BklyAFiPTQsuJNKriQJBZHYaKM2CkCTDQX2bw9sMYuvEC9sjW0W7GUIBygQL/+POEeUqCYPnmEU2g0o1GW2Oqg==</KeyValue></TokenVerificationKey></AlternateVerificationKeys><Audience>urn:test</Audience><Issuer>http://testacs.com/</Issuer><PrimaryVerificationKey i:type="SymmetricVerificationKey"><KeyValue>E5BUHiN4vBdzUzdP0IWaHFMMU3D1uRZgF16TOhSfwwHGSw+Kbf0XqsHzEIYk11M372viB9vbiacsdcQksA0ftw==</KeyValue></PrimaryVerificationKey><RequiredClaims><TokenClaim><ClaimType>urn:microsoft:azure:mediaservices:contentkeyidentifier</ClaimType><ClaimValue i:nil="true" /></TokenClaim></RequiredClaims><TokenType>SWT</TokenType></TokenRestrictionTemplate>"}]}
	
####ContentKeyAuthorizationPolicies を Options にリンクする

[ここ](#ContentKeyAuthorizationPolicies)を参照して ContentKeyAuthorizationPolicies を Options にリンクします。

####承認ポリシーをコンテンツ キーに追加する

[ここ](#AddAuthorizationPolicyToKey)を参照して AuthorizationPolicy を ContentKey に追加します。


##PlayReady 動的暗号化 

Media Services により、ユーザーが保護されたコンテンツを再生する際に、PlayReady DRM ランタイムに適用される権限と制限を構成できます。

PlayReady を使用してコンテンツを保護する場合、承認ポリシーの指定の 1 つとして、[PlayReady ライセンス テンプレート](https://msdn.microsoft.com/library/azure/dn783459.aspx)を定義する XML 文字列を指定する必要があります。

###オープン制限
	
オープン制限とは、キーを要求するすべてのユーザーに、システムがキーを提供することを意味します。この制限は、テストに便利です。

次の例では、オープン承認ポリシーを作成し、それをコンテンツ キーに追加します。
	
####<a id="ContentKeyAuthorizationPolicies2"></a>ContentKeyAuthorizationPolicies を作成する

要求:

	POST https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicies HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 1.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=bbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423581565&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=JiNSG3w6r2C0nIyfKvTZj1uPJGjuitD%2b0sbfZ%2b2JDZI%3d
	x-ms-version: 2.11
	x-ms-client-request-id: 9e7fa407-f84e-43aa-8f05-9790b46e279b
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 58
	
	{"Name":"Deliver Common Content Key"}
	
応答:
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 233
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicies('nb%3Ackpid%3AUUID%3Acc3c64a8-e2fc-4e09-bf60-ac954251a387')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: 9e7fa407-f84e-43aa-8f05-9790b46e279b
	request-id: b3d33c1b-a9cb-4120-ac0c-18f64846c147
	x-ms-request-id: b3d33c1b-a9cb-4120-ac0c-18f64846c147
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 09:26:00 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeyAuthorizationPolicies/@Element","Id":"nb:ckpid:UUID:cc3c64a8-e2fc-4e09-bf60-ac954251a387","Name":"Deliver Common Content Key"}
	

#### ContentKeyAuthorizationPolicyOptions を作成する

要求:
	
	POST https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicyOptions HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 3.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423581565&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=JiNSG3w6r2C0nIyfKvTZj1uPJGjuitD%2b0sbfZ%2b2JDZI%3d
	x-ms-version: 2.11
	x-ms-client-request-id: f160ad25-b457-4bc6-8197-315604c5e585
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 593
	
	{"Name":"","KeyDeliveryType":1,"KeyDeliveryConfiguration":"<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1"><LicenseTemplates><PlayReadyLicenseTemplate><AllowTestDevices>false</AllowTestDevices><ContentKey i:type="ContentEncryptionKeyFromHeader" /><LicenseType>Nonpersistent</LicenseType><PlayRight /></PlayReadyLicenseTemplate></LicenseTemplates></PlayReadyLicenseResponseTemplate>","Restrictions":[{"Name":"Open","KeyRestrictionType":0,"Requirements":null}]}
	
応答:
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 774
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3A1052308c-4df7-4fdb-8d21-4d2141fc2be0')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: f160ad25-b457-4bc6-8197-315604c5e585
	request-id: 563f5a42-50a4-4c4a-add8-a833f8364231
	x-ms-request-id: 563f5a42-50a4-4c4a-add8-a833f8364231
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 09:23:24 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeyAuthorizationPolicyOptions/@Element","Id":"nb:ckpoid:UUID:1052308c-4df7-4fdb-8d21-4d2141fc2be0","Name":"","KeyDeliveryType":1,"KeyDeliveryConfiguration":"<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1"><LicenseTemplates><PlayReadyLicenseTemplate><AllowTestDevices>false</AllowTestDevices><ContentKey i:type="ContentEncryptionKeyFromHeader" /><LicenseType>Nonpersistent</LicenseType><PlayRight /></PlayReadyLicenseTemplate></LicenseTemplates></PlayReadyLicenseResponseTemplate>","Restrictions":[{"Name":"Open","KeyRestrictionType":0,"Requirements":null}]}

####ContentKeyAuthorizationPolicies を Options にリンクする

[ここ](#ContentKeyAuthorizationPolicies)を参照して ContentKeyAuthorizationPolicies を Options にリンクします。

####承認ポリシーをコンテンツ キーに追加する

[ここ](#AddAuthorizationPolicyToKey)を参照して AuthorizationPolicy を ContentKey に追加します。


###トークン制限

トークン制限オプションを構成するには、XML を使用してトークンの承認要件を記述する必要があります。トークン制限の構成 XML は、[この](#schema)セクションで表示される XML スキーマに一致する必要があります。
	
####ContentKeyAuthorizationPolicies を作成する
	
[ここ](#ContentKeyAuthorizationPolicies2)を参照して ContentKeyAuthorizationPolicies を作成します。

####ContentKeyAuthorizationPolicyOptions を作成する
	
要求:

	POST https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicyOptions HTTP/1.1
	Content-Type: application/json
	DataServiceVersion: 3.0;NetFx
	MaxDataServiceVersion: 3.0;NetFx
	Accept: application/json
	Accept-Charset: UTF-8
	Authorization: Bearer http%3a%2f%2fschemas.xmlsoap.org%2fws%2f2005%2f05%2fidentity%2fclaims%2fnameidentifier=juliakoams1&urn%3aSubscriptionId=zbbef702-2233-477b-9f16-bc4d3aa97387&http%3a%2f%2fschemas.microsoft.com%2faccesscontrolservice%2f2010%2f07%2fclaims%2fidentityprovider=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&Audience=urn%3aWindowsAzureMediaServices&ExpiresOn=1423583561&Issuer=https%3a%2f%2fwamsprodglobal001acs.accesscontrol.windows.net%2f&HMACSHA256=5eZnkOsSv%2fLLEKmS%2bWObBlsNYyee8BQlp%2bUYbjugcJg%3d
	x-ms-version: 2.11
	x-ms-client-request-id: ab079b0e-2ba9-4cf1-b549-a97bfa6cd2d3
	Host: wamsbayclus001rest-hs.cloudapp.net
	Content-Length: 1525
	
	{"Name":"Token option","KeyDeliveryType":1,"KeyDeliveryConfiguration":"<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1"><LicenseTemplates><PlayReadyLicenseTemplate><AllowTestDevices>false</AllowTestDevices><ContentKey i:type="ContentEncryptionKeyFromHeader" /><LicenseType>Nonpersistent</LicenseType><PlayRight /></PlayReadyLicenseTemplate></LicenseTemplates></PlayReadyLicenseResponseTemplate>","Restrictions":[{"Name":"Token Authorization Policy","KeyRestrictionType":1,"Requirements":"<TokenRestrictionTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1"><AlternateVerificationKeys><TokenVerificationKey i:type="SymmetricVerificationKey"><KeyValue>w52OyHVqXT8aaupGxuJ3NGt8M6opHDOtx132p4r6q4hLI6ffnLusgEGie1kedUewVoIe1tqDkVE6xsIV7O91KA==</KeyValue></TokenVerificationKey></AlternateVerificationKeys><Audience>urn:test</Audience><Issuer>http://testacs.com/</Issuer><PrimaryVerificationKey i:type="SymmetricVerificationKey"><KeyValue>dYwLKIEMBljLeY9VM7vWdlhps31Fbt0XXhqP5VyjQa33bJXleBtkzQ6dF5AtwI9gDcdM2dV2TvYNhCilBKjMCg==</KeyValue></PrimaryVerificationKey><RequiredClaims><TokenClaim><ClaimType>urn:microsoft:azure:mediaservices:contentkeyidentifier</ClaimType><ClaimValue i:nil="true" /></TokenClaim></RequiredClaims><TokenType>SWT</TokenType></TokenRestrictionTemplate>"}]}

応答:
	
	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Content-Length: 1706
	Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
	Location: https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeyAuthorizationPolicyOptions('nb%3Ackpoid%3AUUID%3Ae42bbeae-de42-4077-90e9-a844f297ef70')
	Server: Microsoft-IIS/8.5
	x-ms-client-request-id: ab079b0e-2ba9-4cf1-b549-a97bfa6cd2d3
	request-id: ccf8a4ba-731e-4124-8192-079592c251cc
	x-ms-request-id: ccf8a4ba-731e-4124-8192-079592c251cc
	X-Content-Type-Options: nosniff
	DataServiceVersion: 3.0;
	X-Powered-By: ASP.NET
	Strict-Transport-Security: max-age=31536000; includeSubDomains
	Date: Tue, 10 Feb 2015 09:58:47 GMT
	
	{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeyAuthorizationPolicyOptions/@Element","Id":"nb:ckpoid:UUID:e42bbeae-de42-4077-90e9-a844f297ef70","Name":"Token option","KeyDeliveryType":1,"KeyDeliveryConfiguration":"<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1"><LicenseTemplates><PlayReadyLicenseTemplate><AllowTestDevices>false</AllowTestDevices><ContentKey i:type="ContentEncryptionKeyFromHeader" /><LicenseType>Nonpersistent</LicenseType><PlayRight /></PlayReadyLicenseTemplate></LicenseTemplates></PlayReadyLicenseResponseTemplate>","Restrictions":[{"Name":"Token Authorization Policy","KeyRestrictionType":1,"Requirements":"<TokenRestrictionTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1"><AlternateVerificationKeys><TokenVerificationKey i:type="SymmetricVerificationKey"><KeyValue>w52OyHVqXT8aaupGxuJ3NGt8M6opHDOtx132p4r6q4hLI6ffnLusgEGie1kedUewVoIe1tqDkVE6xsIV7O91KA==</KeyValue></TokenVerificationKey></AlternateVerificationKeys><Audience>urn:test</Audience><Issuer>http://testacs.com/</Issuer><PrimaryVerificationKey i:type="SymmetricVerificationKey"><KeyValue>dYwLKIEMBljLeY9VM7vWdlhps31Fbt0XXhqP5VyjQa33bJXleBtkzQ6dF5AtwI9gDcdM2dV2TvYNhCilBKjMCg==</KeyValue></PrimaryVerificationKey><RequiredClaims><TokenClaim><ClaimType>urn:microsoft:azure:mediaservices:contentkeyidentifier</ClaimType><ClaimValue i:nil="true" /></TokenClaim></RequiredClaims><TokenType>SWT</TokenType></TokenRestrictionTemplate>"}]}

####ContentKeyAuthorizationPolicies を Options にリンクする

[ここ](#ContentKeyAuthorizationPolicies)を参照して ContentKeyAuthorizationPolicies を Options にリンクします。

####承認ポリシーをコンテンツ キーに追加する

[ここ](#AddAuthorizationPolicyToKey)を参照して AuthorizationPolicy を ContentKey に追加します。


##<a id="types"></a>ContentKeyAuthorizationPolicy を定義するときに使用される種類

###<a id="ContentKeyRestrictionType"></a>ContentKeyRestrictionType

    public enum ContentKeyRestrictionType
    {
        Open = 0,
        TokenRestricted = 1,
        IPRestricted = 2,
    }

###<a id="ContentKeyDeliveryType"></a>ContentKeyDeliveryType

    public enum ContentKeyDeliveryType
    {
        None = 0,
        PlayReadyLicense = 1,
        BaselineHttp = 2,
        Widevine = 3
    }


##Media Services のラーニング パス

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]



##次のステップ
これで、コンテンツ キーの承認ポリシーの構成が完了しました。次は、「[アセットの配信ポリシーの構成方法](media-services-rest-configure-asset-delivery-policy.md)」トピックにお進みください。

 

<!---HONumber=AcomDC_0921_2016-->