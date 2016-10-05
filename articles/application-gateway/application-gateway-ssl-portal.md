<properties
   pageTitle="ポータルを使用した SSL オフロード用のアプリケーション ゲートウェイの構成 | Microsoft Azure"
   description="このページでは、ポータルを使用して、SSL オフロード用のアプリケーション ゲートウェイを作成する方法について説明します。"
   documentationCenter="na"
   services="application-gateway"
   authors="georgewallace"
   manager="carmonm"
   editor="tysonn"/>
<tags
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/09/2016"
   ms.author="gwallace"/>

# ポータルを使用した SSL オフロード用のアプリケーション ゲートウェイの構成

> [AZURE.SELECTOR]
-[Azure portal](application-gateway-ssl-portal.md)
-[Azure Resource Manager PowerShell](application-gateway-ssl-arm.md)
-[Azure Classic PowerShell](application-gateway-ssl.md)

Azure Application Gateway をゲートウェイでの Secure Sockets Layer (SSL) セッションを停止するように構成し、Web ファーム上で発生するコストのかかる SSL 暗号化解除タスクを回避することができます。また、SSL オフロードはフロントエンド サーバーのセットアップと Web アプリケーションの管理も簡素化します。

## シナリオ

次のシナリオでは、既存のアプリケーション ゲートウェイに、SSL オフロードを構成します。このシナリオでは、[アプリケーション ゲートウェイの作成](application-gateway-create-gateway-portal.md)に関する手順を既に実行したことを前提としています。

## 開始する前に

アプリケーション ゲートウェイで SSL オフロードを構成するには、証明書が必要です。この証明書は、アプリケーション ゲートウェイに読み込まれ、SSL 経由で送信されるトラフィックの暗号化と暗号化解除に使用されます。証明書は、Personal Information Exchange (pfx) 形式である必要があります。このファイル形式により、秘密キーをエクスポートすることができます。このキーは、アプリケーション ゲートウェイがトラフィックの暗号化および暗号化解除を実行する際に必要です。

## HTTPS リスナーの追加

HTTPS リスナーは、構成に基づいてトラフィックを検出し、バックエンド プールへのトラフィックのルーティングをサポートします。

### 手順 1

Azure Portal に移動し、既存のアプリケーション ゲートウェイを選択します。

![app gateway overview blade][1]

### 手順 2.

[リスナー] をクリックし、[追加] ボタンをクリックしてリスナーを追加します。

### 手順 3.

リスナーに必要な情報を入力し、.pfx 証明書をアップロードします。完了したら、[OK] をクリックします。

**[名前]** - この値は、リスナーのフレンドリ名です。

**[フロントエンド IP 構成]** - この値は、リスナー用に使用されるフロントエンド IP 構成です。

**[フロントエンド ポート] \([名前]/[ポート])** - アプリケーション ゲートウェイのフロントエンドで使用されるポートのフレンドリ名と、実際に使用されるポートです。

**[プロトコル]** - フロントエンドに https と http のどちらを使用するかを決定するスイッチです。

**[証明書] \([名前]/[パスワード])** - SSL オフロードを使用する場合、この設定に .pfx 証明書が必要で、フレンドリ名とパスワードも必要になります。

![add listener blade][2]

## ルールの作成とリスナーへの関連付け

ここまでで、リスナーが作成されました。今度は、リスナーからのトラフィックを処理するルールを作成します。

### 手順 1

アプリケーション ゲートウェイの **[ルール]** をクリックし、[追加] をクリックします。

![app gateway rules blade][3]

### 手順 2.

**[基本規則の追加]** ブレードで、ルールのフレンドリ名を入力し、前の手順で作成したリスナーを選択します。適切なバックエンド プールと http 設定を選択し、**[OK]** をクリックします。

![https settings window][4]

これで、設定がアプリケーション ゲートウェイに保存されます。これらの設定の保存処理には時間がかかる場合があります。この処理が完了すると、これらの設定はポータルまたは PowerShell で表示できるようになます。保存後は、アプリケーション ゲートウェイがトラフィックの暗号化と暗号化解除を処理します。アプリケーション ゲートウェイとバックエンド Web サーバーの間のすべてのトラフィックは HTTP 経由で処理されます。クライアントに対する通信は、HTTPS 経由で開始された場合、暗号化されてクライアントに返されます。

## 次のステップ

Azure Application Gateway でカスタムの正常性プローブを構成する方法について学習するには、[カスタムの正常性プローブの作成](application-gateway-create-gateway-portal.md)に関するページを参照してください。

[1]: ./media/application-gateway-ssl-portal/figure1.png
[2]: ./media/application-gateway-ssl-portal/figure2.png
[3]: ./media/application-gateway-ssl-portal/figure3.png
[4]: ./media/application-gateway-ssl-portal/figure4.png

<!---HONumber=AcomDC_0921_2016-->