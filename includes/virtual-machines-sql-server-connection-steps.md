### データベース エンジンの既定のインスタンス用に Windows ファイアウォールで TCP ポートを開く

1. Windows リモート デスクトップを介して仮想マシンに接続します。ログインしたら、スタート画面で「**WF.msc**」と入力し、Enter キーを押します。 

	![ファイアウォール プログラムを開始する](./media/virtual-machines-sql-server-connection-steps/12Open-WF.png)
2. **[セキュリティが強化された Windows ファイアウォール]** の左ペインで、**[受信の規則]** を右クリックし、[操作] ペインの **[新しい規則]** をクリックします。

	![新しい規則](./media/virtual-machines-sql-server-connection-steps/13New-FW-Rule.png)

3. **[新規の受信の規則ウィザード]** ダイアログ ボックスの **[規則の種類]** で、**[ポート]** を選択し、**[次へ]** をクリックします。

4. **[プロトコルおよびポート]** ダイアログ ボックスで、既定の **[TCP]** を使用します。**[特定のローカル ポート]** ボックスで、データベース エンジン インスタンスのポート番号を入力します (既定のインスタンスの場合は「**1433**」を指定し、エンドポイントの手順でプライベート ポートに別のポート番号を指定した場合はその番号を指定します)。

	![TCP ポート 1433](./media/virtual-machines-sql-server-connection-steps/14Port-1433.png)

5. **[次へ]** をクリックします。

6. **[操作]** ダイアログ ボックスで、**[接続を許可する]** を選択し、**[次へ]** をクリックします。

	**セキュリティ上の注意:** **[セキュリティで保護されている場合、接続を許可する]** を選択すると、セキュリティが追加されます。お使いの環境で追加のセキュリティ オプションを構成する場合はこのオプションを選択してください。

	![接続を許可する](./media/virtual-machines-sql-server-connection-steps/15Allow-Connection.png)

7. **[プロファイル]** ダイアログ ボックスで、**[パブリック]** 、**[プライベート]** および**[ドメイン]** を選択します。その後、**[次へ]** をクリックします。

    **セキュリティ上の注意:** **[パブリック]** をオンにすると、インターネット経由のアクセスが許可されます。可能であれば、できるだけ制限の厳しいプロファイルを選択してください。

	![パブリック プロファイル](./media/virtual-machines-sql-server-connection-steps/16Public-Private-Domain-Profile.png)

8. **[名前]** ダイアログ ボックスで、この規則の名前と説明を入力し、**[完了]** をクリックします。

	![規則の名前](./media/virtual-machines-sql-server-connection-steps/17Rule-Name.png)

必要に応じて他のコンポーネント用に追加のポートを開きます。詳細については、「[SQL Server のアクセスを許可するための Windows ファイアウォールの構成](http://msdn.microsoft.com/library/cc646023.aspx)」を参照してください。


### TCP プロトコルでリッスンするように SQL Server を構成する

1. 仮想マシンに接続している間に、[スタート] ページで「**SQL Server 構成マネージャー**」と入力し、Enter キーを押します。
	
	![SSCM を開く](./media/virtual-machines-sql-server-connection-steps/9Click-SSCM.png)

2. SQL Server 構成マネージャーのコンソール ペインで、**[SQL Server ネットワークの構成]** を展開します。

3. コンソール ペインで、**[MSSQLSERVER のプロトコル]** (既定のインスタンス名) をクリックします。 詳細ウィンドウで、[TCP] を右クリックします。ギャラリー イメージの場合、その状態は既定で [有効] です。カスタム イメージの場合、**[有効化]** をクリックします (状態が [無効] の場合)。

	![TCP を有効にする](./media/virtual-machines-sql-server-connection-steps/10Enable-TCP.png)

5. コンソール ペインで、**[SQL Server のサービス]** をクリックします詳細ペインで **[SQL Server (_インスタンス名_)]** (既定のインスタンスでは **[SQL Server (MSSQLSERVER)]**) を右クリックして、**[再起動]** をクリックします。これにより、SQL Server のインスタンスが停止し、再起動されます。

	![データベース エンジンの再起動](./media/virtual-machines-sql-server-connection-steps/11Restart.png)

7. SQL Server 構成マネージャーを閉じます。

SQL Server データベース エンジン用のプロトコルを有効にする方法の詳細については、「[サーバー ネットワーク プロトコルの有効化または無効化](http://msdn.microsoft.com/library/ms191294.aspx)」を参照してください。

### 混合モード認証用に SQL Server を構成する

ドメイン環境がない場合、SQL Server データベース エンジンで Windows 認証を使用することはできません。別のコンピューターからデータベース エンジンに接続するには、混合モード認証用に SQL Server を構成します。混合モード認証では、SQL Server 認証と Windows 認証の両方が許可されます

>[AZURE.NOTE] 構成されたドメイン環境でAzure の Virtual Network を構成した場合は、混合モード認証の構成が不要である可能性があります。

1. 仮想マシンに接続している間に、[スタート] ページで「**SQL Server 2014 Management Studio**」と入力し、選択したアイコンをクリックします。

	![SSMS を起動する](./media/virtual-machines-sql-server-connection-steps/18Start-SSMS.png)

	初めて Management Studio を開く場合は、ユーザーの Management Studio 環境の作成が必要になります。これには数分かかることがあります。

2. Management Studio では、**[サーバーへの接続]** ダイアログ ボックスが表示されます。**[サーバー名]** ボックスに、オブジェクト エクスプローラーを使用してデータベース エンジンに接続する仮想マシンの名前を入力します(**[サーバー名]** として、仮想マシン名の代わりに **[(ローカル)]** または単一のピリオドを指定することもできます)。**[Windows 認証]** を選択し、**[ユーザー名]** ボックスで _**your\_VM\_name**\\your\_local\_administrator_ をそのまま使用します。**[接続]** をクリックします。

	![サーバーへの接続](./media/virtual-machines-sql-server-connection-steps/19Connect-to-Server.png)

3. SQL Server Management Studio のオブジェクト エクスプローラーで、SQL Server のインスタンス名 (仮想マシン名) を右クリックし、**[プロパティ]** をクリックします。

	![サーバー プロパティ](./media/virtual-machines-sql-server-connection-steps/20Server-Properties.png)

4. **[セキュリティ]** ページの **[サーバー認証]** で、**[SQL Server 認証モードと Windows 認証モード]** を選択し、**[OK]** をクリックします。

	![認証モードを選択する](./media/virtual-machines-sql-server-connection-steps/21Mixed-Mode.png)

5. [SQL Server Management Studio] ダイアログ ボックスで、SQL Server の再起動が必要であるというメッセージに対して **[OK]** をクリックします。

6. オブジェクト エクスプローラーでサーバーを右クリックし、**[再起動]** をクリックします(実行中であれば、SQL Server エージェントも再起動する必要があります)。

	![再起動](./media/virtual-machines-sql-server-connection-steps/22Restart2.png)

7. [SQL Server Management Studio] ダイアログ ボックスで、SQL Server の再起動に同意を求めるメッセージに対して **[はい]** をクリックします。

### SQL Server 認証ログインを作成する

別のコンピューターからデータベース エンジンに接続するには、1 つ以上の SQL Server 認証ログインを作成する必要があります。

1. SQL Server Management Studio のオブジェクト エクスプローラーで、新しいログインを作成するサーバー インスタンスのフォルダーを展開します。

2. **[セキュリティ]** フォルダーを右クリックし、**[新規作成]** をポイントして、**[ログイン]** を選択します。

	![新しいログイン](./media/virtual-machines-sql-server-connection-steps/23New-Login.png)

3. **[ログイン - 新規作成]** ダイアログ ボックスの **[全般]** ページで、新しいユーザーの名前を **[ログイン名]** ボックスに入力します。

4. **[SQL Server 認証]** を選択します。

5. **[パスワード]** ボックスに、新しいユーザーのパスワードを入力します。**[パスワードの確認]** ボックスに、パスワードを再度入力します。

6. パスワードの強制と複雑さに関するパスワード ポリシー オプションを適用するには、**[パスワード ポリシーを適用する]** (推奨) をオンにします。[SQL Server 認証] が選択されている場合、これは既定のオプションです。

7. 失効に関するパスワード ポリシー オプションを適用するには、**[パスワードの期限を適用する]** (推奨) をオンにします。このチェック ボックスをオンにする場合は、[パスワード ポリシーを適用する] がオンになっている必要があります。[SQL Server 認証] が選択されている場合、これは既定のオプションです。

8. ログインの初回使用後に新しいパスワードの作成をユーザーに強制するには、**[ユーザーは次回ログイン時にパスワードを変更する]** をオンにします (このログインが別のユーザー用の場合は、このチェック ボックスをオンにすることをお勧めします。ログインが自分用の場合は、オンにしないでください)。 このチェック ボックスをオンにする場合は、[パスワードの期限を適用する] がオンになっている必要があります。[SQL Server 認証] が選択されている場合、これは既定のオプションです。

9. **[既定のデータベース]** ボックスの一覧から、ログインの既定のデータベースを選択します。このオプションの既定値は **master** です。ユーザー データベースをまだ作成していない場合は、**master** のままにしておきます。

10. **[既定の言語]** の値は、**[<既定>]** のままにしておきます。
    
	![ログインのプロパティ](./media/virtual-machines-sql-server-connection-steps/24Test-Login.png)

11. これが初めて作成するログインである場合は、このログインを SQL Server 管理者専用に使用することが考えられます。その場合は、**[サーバー ロール]** ページで、**[sysadmin]** をオンにします。

	**セキュリティ上の注意:** sysadmin 固定サーバー ロールのメンバーには、データベース エンジンに対する完全な制御権が与えられています。このロールのメンバーは、適切なユーザーのみに限定してください。

	![sysadmin](./media/virtual-machines-sql-server-connection-steps/25sysadmin.png)

12. [OK] をクリックします。

SQL Server のログインの詳細については、「[ログインの作成](http://msdn.microsoft.com/library/aa337562.aspx)」を参照してください。

<!---HONumber=AcomDC_0302_2016-->