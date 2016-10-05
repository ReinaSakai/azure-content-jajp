<properties 
	pageTitle="Azure Multi-Factor Authentication レポート"
	description="ここでは、Azure Multi-Factor Authentication 機能のレポートを使用する方法について説明します。"
	services="multi-factor-authentication"
	documentationCenter=""
	authors="kgremban"
	manager="femila"
	editor="curtand"/>

<tags
	ms.service="multi-factor-authentication"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/04/2016"
	ms.author="kgremban"/>

# Azure Multi-Factor Authentication のレポート

Azure Multi-Factor Authentication は、個人や組織が使用できるいくつかのレポートを提供します。これらのレポートには、Multi-Factor Authentication 管理ポータルからアクセスできます。管理ポータルを利用するには、Azure MFA プロバイダー、あるいは Azure MFA、Azure AD Premium、または Enterprise Mobility Suite のライセンスが必要です。次に、利用可能なレポートの一覧を示します。

レポートは、Azure 管理ポータルを介してアクセスできます。

Name| Description
:------------- | :------------- |
使用法 | 使用法レポートは、全体的な使用状況、ユーザーの概要およびユーザーの詳細に関する情報を示します。
サーバーの状態|このレポートは、アカウントに関連付けられている Multi-Factor Authentication Server の状態を示します。
ユーザーのブロックの履歴|これらのレポートは、ユーザーのブロックまたはブロック解除の要求の履歴を示します。
ユーザーの認証バイパスの履歴|ユーザーの電話番号について、Multi-Factor Authentication をバイパスする要求の履歴を示します。
不正アクセスのアラート|指定した日付範囲で送信された不正アクセスのアラートの履歴を示します。
キューに登録済み|処理するためにキューに追加されたリストとその状態を一覧表示します。レポートが完成すると、そのレポートのダウンロードまたは表示を行うためのリンクが提供されます。

## レポートを表示するには

1.	http://azure.microsoft.com にログオンします。
2.	左側で、[Active Directory] を選択します。
3.	次のいずれかのオプションを選択します。
	- **オプション 1**: [多要素認証プロバイダー] タブをクリックします。MFA プロバイダーを選択し、下部にある [管理] ボタンをクリックします。
	- **オプション 2**: ディレクトリを選択し、[構成] タブをクリックします。[多要素認証] セクションで、[サービス設定の管理] を選択します。[MFA サービス設定] ページの下部にある [ポータルに移動する] リンクをクリックします。
4.	Azure Multi-Factor Authentication 管理ポータルの左側のナビゲーションに、[レポートの表示] セクションが表示されます。ここから、上記で説明したレポートを選択できます。

<center>![Cloud](./media/multi-factor-authentication-manage-reports/report.png)</center>


**その他のリソース**

* [ユーザー向け](multi-factor-authentication-end-user.md)
* [Azure Multi-Factor Authentication (MSDN)](https://msdn.microsoft.com/library/azure/dn249471.aspx)

<!---HONumber=AcomDC_0921_2016-->