
ディスクの詳細については、[About Disks and VHDs for Virtual Machines (Virtual Machines 用のディスクと VHD について)](../articles/virtual-machines/virtual-machines-linux-about-disks-vhds.md) を参照してください。

##<a id="attachempty"></a>方法: 空のディスクの接続

空のディスクを接続すると、Azure が .vhd ファイルを作成してそれをストレージ アカウントに保存するため、データ ディスクを簡単に追加できます。

1. **[Virtual Machines]** をクリックし、適切な VM を選択します。

2. コマンド バーで、**[ディスクの接続]**、**[空のディスクの接続]** の順にクリックします。


	![Attach an empty disk](./media/howto-attach-disk-windows-linux/AttachEmptyDisk.png)

3.	**[空のディスクの接続]** ダイアログ ボックスが表示されます。


	![Attach a new empty disk](./media/howto-attach-disk-windows-linux/AttachEmptyDetail.png)


	以下の手順を実行します。

	- **[ファイル名]** で、.vhd ファイルのファイル名として自動的に生成される名前を受け入れるか、別の名前を入力します。.vhd ファイルの別の名前を入力した場合でも、データ ディスクには自動的に生成された名前が使用されます。

	- データ ディスクの **[サイズ (GB)]** を入力します。

	- チェック マークをクリックして、完了します。

4.	データ ディスクが作成されて接続されると、VM のダッシュボードに表示されます。

	![Empty data disk successfully attached](./media/howto-attach-disk-windows-linux/AttachEmptySuccess.png)

> [AZURE.NOTE] 新しいデータ ディスクを追加した後、VM にログオンし、ディスクを初期化して使用できるようにする必要があります。


##<a id="attachexisting"></a>方法: 既存のディスクの接続

既存のディスクを接続する場合は、ストレージ アカウントで利用できる .vhd を持っている必要があります。.vhd ファイルをストレージ アカウントにアップロードするには、[Add-AzureVhd](https://msdn.microsoft.com/library/azure/dn495173.aspx) コマンドレットを使用します。.vhd ファイルを作成してアップロードした後は、それを VM に接続できます。

1. **[Virtual Machines]** をクリックし、適切な仮想マシンを選択します。

2. コマンド バーで **[接続]** をクリックし、**[ディスクの接続]** を選択します。


	![Attach data disk](./media/howto-attach-disk-windows-linux/AttachExistingDisk.png)


3. データ ディスクを選択し、チェック マークをクリックしてデータ ディスクを接続します。

	![Enter data disk details](./media/howto-attach-disk-windows-linux/AttachExistingDetail.png)

4.	データ ディスクが接続されると、VM のダッシュボードに表示されます。


	![Data disk successfully attached](./media/howto-attach-disk-windows-linux/AttachExistingSuccess.png)

<!---HONumber=AcomDC_0330_2016------>