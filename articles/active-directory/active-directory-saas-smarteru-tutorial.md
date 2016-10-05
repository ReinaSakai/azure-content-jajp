<properties 
    pageTitle="チュートリアル: Azure Active Directory と SmarterU の統合 | Microsoft Azure" 
    description="Azure Active Directory で SmarterU を使用して、シングル サインオンや自動プロビジョニングなどを有効にする方法について説明します。" 
    services="active-directory" 
    authors="jeevansd"  
    documentationCenter="na" 
    manager="femila"/>
<tags 
    ms.service="active-directory" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.tgt_pltfrm="na" 
    ms.workload="identity" 
    ms.date="09/19/2016" 
    ms.author="jeedes" />

#チュートリアル: Azure Active Directory と SmarterU の統合
  
このチュートリアルでは、Azure と SmarterU の統合について説明します。このチュートリアルで説明するシナリオでは、次の項目があることを前提としています。

-   有効な Azure サブスクリプション
-   SmarterU テナント
  
このチュートリアルを完了すると、SmarterU に割り当てた Azure AD ユーザーは、SmarterU 企業サイト (サービス プロバイダーが開始したサインオン) で、または「[アクセス パネルの概要](active-directory-saas-access-panel-introduction.md)」の説明に従って、アプリケーションにシングル サインオンできるようになります。
  
このチュートリアルで説明するシナリオは、次の要素で構成されています。

1.  SmarterU のアプリケーション統合の有効化
2.  シングル サインオンの構成
3.  ユーザー プロビジョニングの構成
4.  ユーザーの割り当て

![シナリオ](./media/active-directory-saas-smarteru-tutorial/IC777320.png "シナリオ")

##SmarterU のアプリケーション統合の有効化
  
このセクションでは、SmarterU のアプリケーション統合を有効にする方法について説明します。

###SmarterU のアプリケーション統合を有効にするには、次の手順に従います。

1.  Azure クラシック ポータルの左側のナビゲーション ウィンドウで、**[Active Directory]** をクリックします。

    ![Active Directory](./media/active-directory-saas-smarteru-tutorial/IC700993.png "Active Directory")

2.  **[ディレクトリ]** の一覧から、ディレクトリ統合を有効にするディレクトリを選択します。

3.  アプリケーション ビューを開くには、ディレクトリ ビューでトップ メニューの **[アプリケーション]** をクリックします。

    ![アプリケーション](./media/active-directory-saas-smarteru-tutorial/IC700994.png "アプリケーション")

4.  ページの下部にある **[追加]** をクリックします。

    ![アプリケーションの追加](./media/active-directory-saas-smarteru-tutorial/IC749321.png "アプリケーションの追加")

5.  **[実行する内容]** ダイアログで、**[ギャラリーからアプリケーションを追加します]** をクリックします。

    ![ギャラリーからのアプリケーションの追加](./media/active-directory-saas-smarteru-tutorial/IC749322.png "ギャラリーからのアプリケーションの追加")

6.  **検索ボックス**に、「**SmarterU**」と入力します。

    ![アプリケーション ギャラリー](./media/active-directory-saas-smarteru-tutorial/IC777321.png "アプリケーション ギャラリー")

7.  結果ウィンドウで **[SmarterU]** を選択し、**[完了]** をクリックしてアプリケーションを追加します。

    ![SmarterU](./media/active-directory-saas-smarteru-tutorial/IC777322.png "SmarterU")

##シングル サインオンの構成
  
このセクションでは、SAML プロトコルに基づくフェデレーションを使用して、SmarterU で Azure AD のユーザー アカウントを使用してユーザーを認証できるようにする方法を説明します。

###シングル サインオンを構成するには、次の手順に従います。

1.  Azure クラシック ポータルの **[SmarterU]** アプリケーション統合ページで **[シングル サインオンの構成]** をクリックし、**[シングル サインオンの構成]** ダイアログを開きます。

    ![Configure Single Sign-On](./media/active-directory-saas-smarteru-tutorial/IC777323.png "Configure Single Sign-On")

2.  **[ユーザーの SmarterU へのアクセスを設定してください]** ページで、**[Microsoft Azure AD のシングル サインオン]** を選択し、**[次へ]** をクリックします。

    ![Configure Single Sign-On](./media/active-directory-saas-smarteru-tutorial/IC777324.png "Configure Single Sign-On")

3.  **[SmarterU でのシングル サインオンの構成]** ページで、メタデータをダウンロードするために **[メタデータのダウンロード]** をクリックし、データ ファイルを **c:\\SmarterUMetaData.cer** としてローカルに保存します。

    ![Configure Single Sign-On](./media/active-directory-saas-smarteru-tutorial/IC777325.png "Configure Single Sign-On")

4.  別の Web ブラウザー ウィンドウで、SmarterU 企業サイトに管理者としてログインします。

5.  上部のメニューで **[Account Settings]** をクリックします。

    ![Account Settings](./media/active-directory-saas-smarteru-tutorial/IC777326.png "アカウント設定")

6.  アカウント構成ページで、次の手順に従います。

    ![外部認証](./media/active-directory-saas-smarteru-tutorial/IC777327.png "外部認証")

    1.  **[Enable External Authorization]** を選択します。
    2.  **[Master Login Control]** セクションで、**[SmarterU]** タブを選択します。
    3.  **[User Default Login]** セクションで、**[SmarterU]** タブを選択します。
    4.  **[Enable Okta]** を選択します。
    5.  ダウンロードしたメタデータ ファイルの内容をコピーし、**[Okta Metadata]** テキスト ボックスに貼り付けます。
    6.  [**Save**] をクリックします。

7.  Azure クラシック ポータルで、[シングル サインオンの構成の確認] を選択し、**[完了]** をクリックして **[シングル サインオンの構成]** ダイアログを閉じます。

    ![Configure Single Sign-On](./media/active-directory-saas-smarteru-tutorial/IC777328.png "Configure Single Sign-On")

##ユーザー プロビジョニングの構成
  
Azure AD ユーザーが SmarterU にログインできるようにするには、そのユーザーを SmarterU にプロビジョニングする必要があります。SmarterU の場合、プロビジョニングは手動で行います。

###ユーザー アカウントをプロビジョニングするには、次の手順に従います。

1.  **SmarterU** テナントにログインします。

2.  **[Users]** タブに移動します。

3.  ユーザー セクションで、次の手順に従います。

    ![新しいユーザー](./media/active-directory-saas-smarteru-tutorial/IC777329.png "新しいユーザー")

    1.  **[+User]** をクリックします。
    2.  次のテキスト ボックスに、Azure AD のユーザー アカウントの関連する属性の値を入力します。**[Primary Email]**、**[Employee ID]**、**[Password]**、**[Verify Password]**、**[Given Name]**、**[Surname]**。
    3.  **[Active]** をクリックします。
    4.  [**Save**] をクリックします。

>[AZURE.NOTE] SmarterU から提供されている他の SmarterU ユーザー アカウント作成ツールまたは API を使用して、AAD ユーザー アカウントをプロビジョニングできます。

##ユーザーの割り当て
  
構成をテストするには、アプリケーションの使用を許可する Azure AD ユーザーを割り当てて、そのユーザーに、アプリケーションへのアクセス権を付与する必要があります。

###ユーザーを SmarterU に割り当てるには、次の手順に従います。

1.  Azure クラシック ポータルで、テスト アカウントを作成します。

2.  **SmarterU** アプリケーション統合ページで、**[ユーザーの割り当て]** をクリックします。

    ![ユーザーの割り当て](./media/active-directory-saas-smarteru-tutorial/IC777330.png "ユーザーの割り当て")

3.  テスト ユーザーを選択し、**[割り当て]**、**[はい]** の順にクリックして、割り当てを確定します。

    ![Yes](./media/active-directory-saas-smarteru-tutorial/IC767830.png "Yes")
  
シングル サインオンの設定をテストする場合は、アクセス パネルを開きます。アクセス パネルの詳細については、「[アクセス パネルの概要](active-directory-saas-access-panel-introduction.md)」をご覧ください。

<!---HONumber=AcomDC_0921_2016-->