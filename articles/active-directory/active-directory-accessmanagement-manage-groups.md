<properties

	pageTitle="Managing groups in Azure Active Directory | Microsoft Azure"
	description="How to create and manage groups to manage Azure users using Azure Active Directory."
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="09/22/2016"
	ms.author="curtand"/>


# Azure Active Directory におけるグループの管理

Azure Active Directory (Azure AD) ユーザー管理の機能の 1 つに、ユーザーのグループを作成する機能があります。グループは、複数のユーザーにライセンスまたはアクセス許可を一度に割り当てるような管理タスクを実行するために使用します。また、グループは次のリソースに対するアクセス許可の割り当てにも使用できます。

- ディレクトリ内のオブジェクトなどのリソース
- SaaS アプリケーション、Azure サービス、SharePoint サイト、オンプレミスのリソースなど、ディレクトリの外部に存在するリソース

加えてリソース所有者は、リソースへのアクセス権を他のユーザーが所有する Azure AD グループに対して割り当てることもできます。これにより、割り当て先のグループのメンバーにリソースへのアクセス権が付与されます。グループのメンバーシップは、グループの所有者が管理します。実質的にはユーザーをリソースに割り当てる権限が、リソースの所有者からグループの所有者に委任されます。

## どのようにしてグループを作成しますか?

組織がサブスクライブしているサービスに応じて、次のいずれかを使用してグループを作成できます。
- Azure クラシック ポータル
- Office 365 アカウント ポータル
- Windows Intune アカウント ポータル

ここでは、Azure クラシック ポータルで実行するタスクについて説明します。Azure 以外のポータルを使用した Azure AD ディレクトリの管理の詳細については、[Azure AD ディレクトリの管理](active-directory-administer.md)に関するページを参照してください。

1. [Azure クラシック ポータル](https://manage.windowsazure.com)で **[Active Directory]** を選択し、該当する組織のディレクトリ名を選択します。

2. **[グループ]** タブを選択します。

3. **[グループの追加]** を選択します。

4. **[グループの追加]** ウィンドウで、グループの名前と説明を指定します。


## セキュリティ グループのユーザーを個別に追加したり削除したりする方法

**個々のユーザーをグループに追加するには**

1. [Azure クラシック ポータル](https://manage.windowsazure.com)で **[Active Directory]** を選択し、該当する組織のディレクトリ名を選択します。

2. **[グループ]** タブを選択します。

3. メンバーの追加先となるグループを開きます。選択したグループの **[メンバー]** タブを開きます (表示されていない場合)。

4. **[メンバーの追加]** を選択します。

5. **[メンバーの追加]** ページで、このグループのメンバーとして追加するユーザーまたはグループの名前を選択します。この名前が **[選択済み]** ウィンドウに追加されていることを確認してください。


**個々のユーザーをグループから削除するには**

1. [Azure クラシック ポータル](https://manage.windowsazure.com)で **[Active Directory]** を選択し、該当する組織のディレクトリ名を選択します。

2. **[グループ]** タブを選択します。

3. 削除するメンバーのグループを開きます。

4. **[メンバー]** タブを選択し、そのグループから削除するメンバーの名前を選択して **[削除]** をクリックします。

6. このメンバーをグループから削除してよいか確認するメッセージが表示されるので、削除してよい場合は操作を確定します。


## グループのメンバーシップを動的に管理する方法

Azure AD では、グループのメンバーとして追加するユーザーを特定のルールに基づいて指定できます。単純なルールであればその設定はごく簡単です。単純なルールとは、比較を 1 回だけ行うルールです。たとえば、グループが SaaS アプリケーションに割り当てられている場合、"営業担当者" の役職を持つユーザーを追加するルールを設定できます。 設定したこのルールによって、ディレクトリ内でその役職を持つすべてのユーザーに、この SaaS アプリケーションへのアクセスが許可されます。

ユーザーのいずれかの属性が変更されると、システムはディレクトリ内のすべての動的なグループ ルールを評価して、このユーザーの属性の変更によってグループの追加または削除がトリガーされるかどうかを確認します。ユーザーがグループのルールを満たしている場合は、そのグループにメンバーとして追加されます。メンバーになっているグループのルールを満たさなくなった場合は、そのグループのメンバーから削除されます。

> [AZURE.NOTE] セキュリティ グループまたは Office 365 グループには、動的メンバーシップのルールを設定できます。現在、アプリケーションに対するグループ ベースの割り当てでは入れ子になったグループ メンバーシップはサポートされていません。
>
> グループの動的メンバーシップを実行するには、次のユーザーに Azure AD Premium ライセンスが割り当てられている必要があります。
>
> - グループに対するルールを管理する管理者
> - グループのすべてのメンバー

**グループに対する動的メンバーシップを有効にするには**

1. [Azure クラシック ポータル](https://manage.windowsazure.com)で **[Active Directory]** を選択し、該当する組織のディレクトリ名を選択します。

2. **[グループ]** タブを選択し、編集するグループを開きます。

3. **[構成]** タブを選択し、**[動的メンバーシップを有効にする]** を **[はい]** に設定します。

4. このグループに対する動的メンバーシップの働きを制御する単純なルールを 1 つグループに設定します。**[追加するユーザーの所属]** オプションがオンになっていることを確認し、ユーザーのプロパティ (department、jobTitle など) をリストから選択します。

5. 次に、条件 (等しくない、等しい、指定値で始まらない、指定値で始まる、指定値を含まない、指定値を含む、一致しない、一致する) を選択します。

6. 選択したユーザー プロパティの比較値を指定します。

動的グループのメンバーシップ管理を目的とした*高度*なルール (複数の比較を伴うルール) を作成する方法については、「[属性を使用した高度なルールの作成](active-directory-accessmanagement-groups-with-advanced-rules.md)」を参照してください。

## 追加情報

次の記事は、Azure Active Directory に関する追加情報を示します。

* [Azure Active Directory グループによるリソースのアクセス管理](active-directory-manage-groups.md)

* [グループの設定を構成するための Azure Active Directory コマンドレット](active-directory-accessmanagement-groups-settings-cmdlets.md)

* [Article Index for Application Management in Azure Active Directory](active-directory-apps-index.md)

* [Azure Active Directory とは](active-directory-whatis.md)

* [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0928_2016-->