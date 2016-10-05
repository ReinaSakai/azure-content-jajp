<properties 
    pageTitle="Split-Merge セキュリティの構成 | Microsoft Azure" 
    description="暗号化のための x409 証明書の設定" 
    metaKeywords="Elastic Database certificates security" 
    services="sql-database" 
    documentationCenter="" 
    manager="jhubbard" 
    authors="torsteng"/>

<tags 
    ms.service="sql-database" 
    ms.workload="sql-database" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="05/27/2016" 
    ms.author="torsteng" />


# Split-Merge セキュリティの構成  

Split/Merge サービスを使用するには、セキュリティが正しく構成されていなければなりません。このサービスは、Microsoft Azure SQL Database の Elastic Scale 機能の一部です。詳しくは、「[Elastic Scale の分割とマージ サービス チュートリアル](sql-database-elastic-scale-configure-deploy-split-and-merge.md)」をご覧ください。

## 証明書の構成

証明書は次の 2 つの方法で構成されます。

1. [SSL 証明書を構成するには](#To-Configure-the-SSL#Certificate)
2. [クライアント証明書を構成するには](#To-Configure-Client-Certificates) 

## 証明書を取得するには

証明書はパブリック証明機関 (CA) または [Windows Certificate Service](http://msdn.microsoft.com/library/windows/desktop/aa376539.aspx) から取得できます。これは証明書を取得するための推奨方法です。

これらの方法が利用可能でない場合は、**自己署名証明書**を生成できます。
 
## 証明書を生成するツール

* [makecert.exe](http://msdn.microsoft.com/library/bfsktky3.aspx)
* [pvk2pfx.exe](http://msdn.microsoft.com/library/windows/hardware/ff550672.aspx)

### ツールを実行するには

* Visual Studio の開発者コマンド プロンプトで、「[Visual Studio コマンド プロンプト](http://msdn.microsoft.com/library/ms229859.aspx)」を参照します。 

    インストールされている場合は、次のように参照します。

        %ProgramFiles(x86)%\Windows Kits\x.y\bin\x86 

* 「[Windows 8.1: キットとツールのダウンロード](http://msdn.microsoft.com/windows/hardware/gg454513#drivers)」から WDK を入手します。

## SSL 証明書を構成するには
通信の暗号化やサーバーの認証には SSL 証明書が必要です。以下の 3 つのシナリオから最適なものを選択し、すべての手順を実行します。

### 新しい自己署名証明書を作成する

1.    [自己署名証明書を作成する](#Create-a-Self-Signed-Certificate)
2.    [自己署名 SSL 証明書用の PFX ファイルを作成する](#Create-PFX-file-for-Self-Signed-SSL-Certificate)
3.    [クラウド サービスに SSL 証明書をアップロードする](#Upload-SSL-Certificate-to-Cloud-Service)
4.    [サービス構成ファイルの SSL 証明書を更新する](#Update-SSL-Certificate-in-Service-Configuration-File)
5.    [SSL 証明機関をインポートする](#Import-SSL-Certification-Authority)

### 証明書ストアから既存の証明書を使用するには
1. [証明書ストアから SSL 証明書をエクスポートする](#Export-SSL-Certificate-From-Certificate-Store)
2. [クラウド サービスに SSL 証明書をアップロードする](#Upload-SSL-Certificate-to-Cloud-Service)
3. [サービス構成ファイルの SSL 証明書を更新する](#Update-SSL-Certificate-in-Service-Configuration-File)

### PFX ファイルの既存の証明書を使用するには

1. [クラウド サービスに SSL 証明書をアップロードする](#Upload-SSL-Certificate-to-Cloud-Service)
2. [サービス構成ファイルの SSL 証明書を更新する](#Update-SSL-Certificate-in-Service-Configuration-File)

## クライアント証明書を構成するには
サービスへの要求を認証するには、クライアント証明書が必要です。以下の 3 つのシナリオから最適なものを選択し、すべての手順を実行します。

### クライアント証明書をオフにする
1.    [クライアント証明書ベースの認証をオフにする](#Turn-Off-Client-Certificate-Based-Authentication)

### 新しい自己署名証明書を発行する
1.    [自己署名証明機関を作成する](#Create-a-Self-Signed-Certification-Authority)
2.    [CA 証明書をクラウド サービスにアップロードする](#Upload-CA-Certificate-to-Cloud-Service)
3.    [サービス構成ファイルの CA 証明書を更新する](#Update-CA-Certificate-in-Service-Configuration-File)
4.    [クライアント証明書を発行する](#Issue-Client-Certificates)
5.    [クライアント証明書の PFX ファイルを作成する](#Create-PFX-files-for-Client-Certificates)
6.    [クライアント証明書をインポートする](#Import-Client-Certificate)
7.    [クライアント証明書のサムプリントをコピーする](#Copy-Client-Certificate-Thumbprints)
8.    [許可されているクライアントをサービス構成ファイルに構成する](#Configure-Allowed-Clients-in-the-Service-Configuration-File)

### 既存のクライアント証明書を使用する
1.    [CA の公開キーを検索する](#Find-CA-Public Key)
2.    [CA 証明書をクラウド サービスにアップロードする](#Upload-CA-certificate-to-cloud-service)
3.    [サービス構成ファイルの CA 証明書を更新する](#Update-CA-Certificate-in-Service-Configuration-File)
4.    [クライアント証明書のサムプリントをコピーする](#Copy-Client-Certificate-Thumbprints)
5.    [許可されているクライアントをサービス構成ファイルに構成する](#Configure-Allowed-Clients-in-the-Service-Configuration File)
6.    [クライアント証明書の失効確認を構成する](#Configure-Client-Certificate-Revocation-Check)

## 許可された IP アドレス

サービス エンドポイントへのアクセスは特定範囲の IP アドレスに制限できます。

## ストアの暗号化を構成するには

メタデータ ストアに格納されている資格情報を暗号化するには、証明書が必要です。以下の 3 つのシナリオから最適なものを選択し、すべての手順を実行します。

### 新しい自己署名証明書を使用する

1.     [自己署名証明書を作成する](#Create-a-Self-Signed-Certificate)
2.     [自己署名の暗号化証明書の PFX ファイルを作成する](#Create-PFX-file-for-Self-Signed-Encryption-Certificate)
3.     [クラウド サービスに暗号化証明書をアップロードする](#Upload-Encryption-Certificate-to-Cloud-Service)
4.     [サービス構成ファイルの暗号化証明書を更新する](#Update-Encryption-Certificate-in-Service-Configuration-File)

### 証明書ストアにある既存の証明書を使用する

1.     [証明書ストアから暗号化証明書をエクスポートする](#Export-Encryption-Certificate-From-Certificate-Store)
2.     [クラウド サービスに暗号化証明書をアップロードする](#Upload-Encryption-Certificate-to-Cloud-Service)
3.     [サービス構成ファイルの暗号化証明書を更新する](#Update-Encryption-Certificate-in-Service-Configuration-File)

### PFX ファイル内に既存の証明書を使用する

1.     [クラウド サービスに暗号化証明書をアップロードする](#Upload-Encryption-Certificate-to-Cloud-Service)
2.     [サービス構成ファイルの暗号化証明書を更新する](#Update-Encryption-Certificate-in-Service-Configuration-File)

## 既定の構成

既定の構成では、HTTP エンドポイントへのすべてのアクセスを拒否します。これらのエンドポイントへの要求にはデータベースの資格情報などの機密情報が含まれていることがあるため、これが推奨される設定です。既定の構成では、HTTPS エンドポイントへのすべてのアクセスを許可します。この設定は、さらに制限できます。

### 構成の変更

エンドポイントに適用されるアクセス制御ルールのグループは、**サービス構成ファイル**の **<EndpointAcls>** セクションに構成されます。

    <EndpointAcls>
      <EndpointAcl role="SplitMergeWeb" endPoint="HttpIn" accessControl="DenyAll" />
      <EndpointAcl role="SplitMergeWeb" endPoint="HttpsIn" accessControl="AllowAll" />
    </EndpointAcls>

アクセス制御グループ内のルールは、サービス構成ファイルの <AccessControl name=""> セクションに構成されます。

形式は、ネットワーク アクセス制御リスト ドキュメントに説明があります。たとえば、HTTPS エンドポイントへのアクセスを範囲 100.100.0.0 ～ 100.100.255.255 の IP のみ許可する場合、ルールは次のようになります。

    <AccessControl name="Retricted">
      <Rule action="permit" description="Some" order="1" remoteSubnet="100.100.0.0/16"/>
      <Rule action="deny" description="None" order="2" remoteSubnet="0.0.0.0/0" />
    </AccessControl>
    <EndpointAcls>
    <EndpointAcl role="SplitMergeWeb" endPoint="HttpsIn" accessControl="Restricted" />

## サービス拒否 (DOS) 防止

サービス拒否の攻撃を検出および防止するための支援として次の 2 種類のメカニズムがあります。

*    リモート ホストあたりの同時要求数の制限 (既定ではオフ)
*    リモート ホストあたりのアクセス レートの制限 (既定ではオン)

このような機能の基本となる機能の詳細については、IIS の Dynamic IP Security に関するページをご覧ください。この構成を変更する場合、次の要素に注意が必要です。

* リモート ホスト情報を扱うプロキシおよびネットワーク アドレス変換デバイスの動作
* Web ロールにおける任意のリソースに対する各要求が考慮されている (例: スクリプト、イメージなどの読み込み)

## 同時実行アクセス数の制御

この動作を構成するための設定は、次のとおりです。

    <Setting name="DynamicIpRestrictionDenyByConcurrentRequests" value="false" />
    <Setting name="DynamicIpRestrictionMaxConcurrentRequests" value="20" />

この保護を有効にするには、DynamicIpRestrictionDenyByConcurrentRequests を true に変更します。

## アクセス レートの制限

この動作を構成するための設定は、次のとおりです。

    <Setting name="DynamicIpRestrictionDenyByRequestRate" value="true" />
    <Setting name="DynamicIpRestrictionMaxRequests" value="100" />
    <Setting name="DynamicIpRestrictionRequestIntervalInMilliseconds" value="2000" />

## 拒否された要求に対する応答の構成

次の設定は、拒否された要求への応答を構成します。

    <Setting name="DynamicIpRestrictionDenyAction" value="AbortRequest" />
サポートされている他の値については、IIS の Dynamic IP Security に関するドキュメントを参照してください。

## サービス証明書を構成する操作
このトピックは参照専用です。次に概要を説明している手順に従って構成してください。

* SSL 証明書の構成
* クライアント証明書の構成

## 自己署名証明書の作成
次のように実行します。

    makecert ^
      -n "CN=myservice.cloudapp.net" ^
      -e MM/DD/YYYY ^
      -r -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.1" ^
      -a sha1 -len 2048 ^
      -sv MySSL.pvk MySSL.cer

カスタマイズするには、次のように実行します。

*    サービスの URL に -n を指定します。ワイルドカード ("CN=*.cloudapp.net") と代替名 ("CN=myservice1.cloudapp.net、CN=myservice2.cloudapp.net") がサポートされています。
*    証明書の有効期限の日付に -e を指定します。強力なパスワードを作成し、要求されたときにこれを指定してください。

## 自己署名 SSL 証明書のための PFX ファイルを作成する

次のように実行します。

        pvk2pfx -pvk MySSL.pvk -spc MySSL.cer

パスワードを入力し、その後、次のオプションを使用して証明書をエクスポートします。
* はい、秘密キーをエクスポートします
* すべての拡張プロパティをエクスポートする

## 証明書ストアから SSL 証明書をエクスポートする

* 証明書を検索する
* [アクション]、[すべてのタスク]、[エクスポート] の順にクリックする
* 次のオプションを使用して証明書を .PFX ファイルにエクスポートします。
    * はい、秘密キーをエクスポートします
    * 可能であれば、証明書パスのすべての証明書を含めます *すべての拡張プロパティをエクスポートします

## SSL 証明書をクラウド サービスにアップロードする

既存または生成された .PFX ファイルと SSL キーのペアを使用して、証明書を次のようにアップロードします。

* 秘密キーの情報を保護するパスワードを入力する

## サービス構成ファイルの中の SSL 証明書を更新する

サービス構成ファイルの次の設定のサムプリント値を、クラウド サービスにアップロードされた証明書のサムプリントを使用して、次のように更新します。

    <Certificate name="SSL" thumbprint="" thumbprintAlgorithm="sha1" />

## SSL 証明機関をインポートする

サービスと通信するすべてのアカウントおよびマシンで、次の手順に従います。

* Windows エクスプローラーで .CER ファイルをダブルクリックする
* [証明書] ダイアログ ボックスで [証明書のインストール] をクリックする
* 信頼されたルート証明機関のストアに証明書をインポートする

## クライアント証明書ベースの認証をオフにする

クライアント証明書ベースの認証のみがサポートされています。これを無効にすると、他のメカニズムが (Microsoft Azure Virtual Network など) が機能していない限り、サービス エンドポイントへのパブリック アクセスが可能になります。

この機能を無効にするには、次のようにして、サービス構成ファイルでこの設定を false にします。

    <Setting name="SetupWebAppForClientCertificates" value="false" />
    <Setting name="SetupWebserverForClientCertificates" value="false" />

次に、CA 証明書の設定で、SSL 証明書と同じサムプリントを次のようにコピーします。

    <Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />

## 自己署名証明機関を作成する
認証機関として機能する自己署名証明書を作成するには、次の手順を実行します。

    makecert ^
    -n "CN=MyCA" ^
    -e MM/DD/YYYY ^
     -r -cy authority -h 1 ^
     -a sha1 -len 2048 ^
      -sr localmachine -ss my ^
      MyCA.cer

これをカスタマイズするには次のようにします。

*    証明の有効期限の日付に -e を指定する


## CA の公開キーを検索する

すべてのクライアント証明書は、サービスで信頼されている証明機関によって発行される必要があります。クラウド サービスにアップロードするために、認証に使用するクライアント証明書を発行した認証機関への公開キーを検索します。

公開キーのファイルが利用可能でない場合、証明書ストアから次のようにエクスポートしてください。

* 証明書を検索する
    * 同じ証明機関によって発行されたクライアント証明書を検索します。
* 証明書をダブルクリックする
* 証明書のダイアログ ボックスで、[証明のパス] タブをクリックする
* パスの CA エントリをダブルクリックする
* 証明書のプロパティを書き留める
* **[証明書]** ダイアログ ボックスを閉じる
* 証明書を検索する
    * 前のように CA を検索します。
* [アクション]、[すべてのタスク]、[エクスポート] の順にクリックする
* 次のオプションを指定した証明書を .CER にエクスポートする
    * **いいえ、秘密キーをエクスポートしません**
    * 可能であれば、証明書パスにあるすべての証明書を含む
    * すべての拡張プロパティをエクスポートする

## CA 証明書をクラウド サービスにアップロードする

既存または生成された .CER ファイルを CA 公開キーと共にアップロードします。

## サービス構成ファイルの CA 証明書を更新する

サービス構成ファイルの次の設定のサムプリント値を、クラウド サービスにアップロードされた証明書のサムプリントを使用して、次のように更新します。

    <Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />

同じサムプリントを使用して、次の設定の値を更新します。

    <Setting name="AdditionalTrustedRootCertificationAuthorities" value="" />

## クライアント証明書を発行する

サービスへのアクセスが許可された各個人は、排他的に使用するクライアント証明書を持っている必要があります。また、秘密キーを保護するための強力なパスワードを独自に選択する必要があります。

自己署名 CA 証明書が生成および格納された同じマシンで、次の手順を実行する必要があります。

    makecert ^
      -n "CN=My ID" ^
      -e MM/DD/YYYY ^
      -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.2" ^
      -a sha1 -len 2048 ^
      -in "MyCA" -ir localmachine -is my ^
      -sv MyID.pvk MyID.cer

次のようにカスタマイズします。

* この証明書で認証されるクライアントの ID に -n を指定します。
* 証明書の有効期限の日付に -e を指定します。
* MyID.pvk および MyID.cer には、このクライアント証明書用の一意のファイル名を指定します。

このコマンドでは、パスワードの作成と 1 回の使用が求められます。強力なパスワードを使用します。

## クライアント証明書の PFX ファイルを作成する

生成された各クライアント証明書で、次のように実行します。

    pvk2pfx -pvk MyID.pvk -spc MyID.cer

次のようにカスタマイズします。

    MyID.pvk and MyID.cer with the filename for the client certificate

パスワードを入力し、その後、次のオプションを使用して証明書をエクスポートします。

* はい、秘密キーをエクスポートします
* すべての拡張プロパティをエクスポートする
* この証明書の発行先である個人は、パスワードのエクスポートを選択する必要があります。

## クライアント証明書をインポートする

クライアント証明書が発行されている各ユーザーは、キー ペアを、サービスとの通信で使用するマシンに次のようにインポートする必要があります。

* Windows エクスプローラーで .PFX ファイルをダブルクリックする
* 少なくとも次のオプションを使用して、個人用ストアに証明書をインポートします。
    * チェック済みの拡張されたすべてのプロパティを含める

## クライアント証明書のサムプリントをコピーする
証明書が発行されている各ユーザーは、サービス構成ファイルに追加される自分の証明書のサムプリントを取得するために、次の手順を実行する必要があります。
* certmgr.exe を実行する
* [個人設定] タブをクリックする
* 認証に使用するクライアント証明書をダブルクリックする
* 表示される [証明書] ダイアログ ボックスで [詳細] タブをクリックする
* すべてが表示されていることを確認する
* 一覧の Thumbprint という名前のフィールドを選択する
* サムプリントの値をコピーする 
** 最初の桁の前にある非表示の Unicode 文字を削除する
** すべてのスペースを削除する

## 許可されているクライアントをサービス構成ファイルに構成する

サービス構成ファイルの次の設定値を、サービスへのアクセスが許可されたクライアント証明書のコンマで区切られたサムプリント一覧を使用して更新します。

    <Setting name="AllowedClientCertificateThumbprints" value="" />

## クライアント証明書の失効確認を構成する

既定の設定では、証明機関によるクライアント証明書の失効状態の確認は行われません。クライアント証明書を発行した証明機関がこのような確認をサポートする場合にこの確認をオンにするには、X509RevocationMode 列挙型に定義された値の 1 つを使用して次のように設定を変更します。

    <Setting name="ClientCertificateRevocationCheck" value="NoCheck" />

## 自己署名の暗号化証明書の PFX ファイルを作成する

暗号化証明書は、次のように実行します。

    pvk2pfx -pvk MyID.pvk -spc MyID.cer

次のようにカスタマイズします。

    MyID.pvk and MyID.cer with the filename for the encryption certificate

パスワードを入力し、その後、次のオプションを使用して証明書をエクスポートします。
*    はい、秘密キーをエクスポートします
*    すべての拡張プロパティをエクスポートする
*    クラウド サービスに証明書をアップロードする際に、このパスワードが必要になります。

## 証明書ストアから暗号化証明書をエクスポートする

*    証明書を検索する
*    [アクション]、[すべてのタスク]、[エクスポート] の順にクリックする
*    次のオプションを使用して証明書を .PFX ファイルにエクスポートします。 
  *    はい、秘密キーをエクスポートします
  *    可能であれば、証明書パスにあるすべての証明書を含む 
*    すべての拡張プロパティをエクスポートする

## クラウド サービスに暗号化証明書をアップロードする

既存または生成された .PFX ファイルと 暗号化キーのペアを使用して、証明書を次のようにアップロードします。

* 秘密キーの情報を保護するパスワードを入力する

## サービス構成ファイルの暗号化証明書を更新する

サービス構成ファイルの次の設定のサムプリント値を、クラウド サービスにアップロードされた証明書のサムプリントを使用して、次のように更新します。

    <Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />

## 一般的な証明操作

* SSL 証明書の構成
* クライアント証明書の構成

## 証明書を検索する

次の手順に従います。

1. Mmc.exe を実行します。
2. [ファイル]、[スナップインの追加と削除] の順にクリックします。
3. **[証明書]** を選択します。
4. **[追加]** をクリックします。
5. 証明書ストアの場所を選択します。
6. **[完了]** をクリックします。
7. **[OK]** をクリックします。
8. **[証明書]** を展開します。
9. 証明書ストアを展開します。
10. 証明書の子ノードを展開します。
11. 一覧から 1 つの証明書を選択します。

## 証明書をエクスポートします。
**証明書のエクスポート ウィザード**で次のように実行します。

1. **[次へ]** をクリックします。
2. **[はい]**、**[秘密キーをエクスポートします]** の順に選択します。
3. **[次へ]** をクリックします。
4. 目的の出力ファイル形式を選択します。
5. 必要なオプションを確認します。
6. **[パスワード]** をオンにします。
7. 強力なパスワードを入力し、確定します。
8. **[次へ]** をクリックします。
9. 証明書が格納されているファイル名を入力するか参照します (拡張子 .PFX を使用)。
10. **[次へ]** をクリックします。
11. **[完了]** をクリックします。
12. **[OK]** をクリックします。

## 証明書のインポート

証明書のインポート ウィザードで次のように実行します。

1. ストアの場所を選択します。

    * 現在のユーザーが実行中のプロセスのみがサービスにアクセスする場合は、**[現在のユーザー]** を選択します。
    * このコンピューターの他のプロセスがサービスにアクセスする場合は、**[ローカル マシン]** を選択します。
2. **[次へ]** をクリックします。
3. ファイルからインポートしている場合は、ファイルのパスを確認します。
4. .PFX ファイルをインポートする場合は、次のようにします。
    1.     秘密キーの情報を保護するパスワードを入力する
    2.     インポート オプションを選択する
5.     次のストアに証明書の "場所" を選択します。
6.     **[参照]** をクリックします。
7.     目的のストアを選択します。
8.     **[完了]** をクリックします。
       
    * [信頼されたルート証明機関ストア] を選択した場合は、**[はい]** をクリックします。
9.     すべてのダイアログ ウィンドウで **[OK]** をクリックします。

## 証明書のアップロード

[Azure ポータル](https://portal.azure.com/)で次の操作を行います

1. **[クラウド サービス]** を選択します。
2. クラウド サービスを選択します。
3. 上部メニューで **[証明書]** をクリックします。
4. 下部のバーで **[アップロード]** をクリックします。
5. 証明書ファイルを選択します。
6. .PFX ファイルの場合は、秘密キーのパスワードを入力します。
7. 完了したら、一覧内の新しいエントリから証明書の拇印をコピーします。

## その他のセキュリティの考慮事項
 
このドキュメントで説明した SSL の設定では、HTTPS エンドポイント使用時のサービスとクライアント間の通信を暗号化します。この暗号化が重要なのは、通信には、データベース アクセスの資格証明および他の潜在的な機密情報が含まれているためです。しかし、ここで注意が必要なのは、サービスでは、資格情報を含む内部の状態が、Microsoft Azure SQL Database の内部テーブルに保持されるという点です。このテーブルは Microsoft Azure サブスクリプションのメタデータ ストレージ用にユーザーが提供したテーブルです。このデータベースは、サービス構成ファイルの設定の一部として次のように定義されたものです (.CSCFG ファイル)。

    <Setting name="ElasticScaleMetadata" value="Server=…" />

このデータベースに格納されている資格情報が暗号化されます。ただし、ベスト プラクティスとして、サービス デプロイメントにおける Web ロールとワーカー ロールの両方を最新に保ち、両者がメタデータのデータベースと保存された資格情報の暗号化と解読に使用する証明書へアクセスする際の安全性が保たれるようにします。

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!---HONumber=AcomDC_0601_2016-->