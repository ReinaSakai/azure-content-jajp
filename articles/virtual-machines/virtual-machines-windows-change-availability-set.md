<properties
	pageTitle="VM の可用性セットの変更 | Microsoft Azure"
	description="Azure PowerShell と Resource Manager デプロイメント モデルを使用して仮想マシンの可用性セットを変更する方法について説明します。"
	keywords=""
	services="virtual-machines-windows"
	documentationCenter=""
	authors="Drewm3"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>
<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/15/2016"
	ms.author="drewm"/>



# Windows VM の可用性セットの変更

以下の手順では、Azure PowerShell を使用して VM の可用性セットを変更する方法について説明します。VM を可用性セットに追加できるのは、VM の作成時のみです。可用性セットを変更するには、仮想マシンを削除してから作成し直す必要があります。

## PowerShell を使用して可用性セットを変更する

1. 変更する VM の以下の重要な詳細を記録します。

	VM の名前
	
	```powershell
	$vm = Get-AzureRmVM -ResourceGroupName <Name-of-resource-group> -Name <name-of-VM>
	$vm.Name
	```
 
	VM サイズ
	
	```powershell
	$vm.HardwareProfile.VmSize
	```

	ネットワークのプライマリ ネットワーク インターフェイスと (VM 上に存在する場合) オプションのネットワーク インターフェイス
	
	```powershell
	$vm.NetworkProfile.NetworkInterfaces[0].Id
	```

	OS ディスク プロファイル

	```powershell
	$vm.StorageProfile.OsDisk.OsType
	$vm.StorageProfile.OsDisk.Name
	$vm.StorageProfile.OsDisk.Vhd.Uri
	```

	各データ ディスクのディスク プロファイル
	
	```powershell
	$vm.StorageProfile.DataDisks[<index>].Lun
	$vm.StorageProfile.DataDisks[<index>].Vhd.Uri
	```

	インストール済みの VM 拡張機能
	
	```powershell
	$vm.Extensions
	```

2. ディスクやネットワーク インターフェイスは削除せずに、VM を削除します。

	```powershell
	Remove-AzureRmVM -ResourceGroupName <resourceGroupName> -Name <vmName> 
	```

3. 可用性セットを作成します (存在しない場合)。

	```powershell
	New-AzureRmAvailabilitySet -ResourceGroupName <resourceGroupName> -Name <availabilitySetName> -Location "<location>" 
	```

4. 新しい可用性セットを使用して VM を再作成します。

	```powershell
	$vm2 = New-AzureRmVMConfig -VMName <VM-name> -VMSize <vm-size> -AvailabilitySetId <availability-set-id>

	Set-AzureRmVMOSDisk -CreateOption "Attach" -VM <vmConfig> -VhdUri <osDiskURI> -Name <osDiskName> [-Windows | -Linux]

	Add-AzureRmVMNetworkInterface -VM <vmConfig> -Id  <nicId> 

	New-AzureRmVM -ResourceGroupName <resourceGroupName> -Location <location> -VM <vmConfig>
	``` 

5. データ ディスクと拡張機能を追加します。詳細については、[VM へのデータ ディスクの接続](virtual-machines-windows-attach-disk-portal.md)に関するページと[拡張機能の構成例](virtual-machines-windows-extensions-configuration-samples.md)に関するページを参照してください。VM へのデータ ディスクと拡張機能の追加は、PowerShell または Azure CLI を使用して行うことができます。

## サンプル スクリプト

次のスクリプトでは、必要な情報を収集し、元の VM を削除してから新しい可用性セット内でその VM を作り直す例を示します。

```powershell
	#set variables
	$rg = "demo-resource-group"
	$vmName = "demo-vm"
	$newAvailSetName = "demo-as"
	$outFile = "C:\temp\outfile.txt"

	#Get VM Details
	$OriginalVM = get-azurermvm -ResourceGroupName $rg -Name $vmName

	#Output VM details to file
	"VM Name: " | Out-File -FilePath $outFile 
	$OriginalVM.Name | Out-File -FilePath $outFile -Append

	"Extensions: " | Out-File -FilePath $outFile -Append
	$OriginalVM.Extensions | Out-File -FilePath $outFile -Append

	"VMSize: " | Out-File -FilePath $outFile -Append
	$OriginalVM.HardwareProfile.VmSize | Out-File -FilePath $outFile -Append

	"NIC: " | Out-File -FilePath $outFile -Append
	$OriginalVM.NetworkProfile.NetworkInterfaces[0].Id | Out-File -FilePath $outFile -Append

	"OSType: " | Out-File -FilePath $outFile -Append
	$OriginalVM.StorageProfile.OsDisk.OsType | Out-File -FilePath $outFile -Append

	"OS Disk: " | Out-File -FilePath $outFile -Append
	$OriginalVM.StorageProfile.OsDisk.Vhd.Uri | Out-File -FilePath $outFile -Append

	if ($OriginalVM.StorageProfile.DataDisks) {
    "Data Disk(s): " | Out-File -FilePath $outFile -Append
    $OriginalVM.StorageProfile.DataDisks | Out-File -FilePath $outFile -Append
	}

	#Remove the original VM
	Remove-AzureRmVM -ResourceGroupName $rg -Name $vmName

	#Create new availability set if it does not exist
	$availSet = Get-AzureRmAvailabilitySet -ResourceGroupName $rg -Name $newAvailSetName -ErrorAction Ignore
	if (-Not $availSet) {
    $availset = New-AzureRmAvailabilitySet -ResourceGroupName $rg -Name $newAvailSetName -Location $OriginalVM.Location
	}

	#Create the basic configuration for the replacement VM
	$newVM = New-AzureRmVMConfig -VMName $OriginalVM.Name -VMSize $OriginalVM.HardwareProfile.VmSize -AvailabilitySetId $availSet.Id
	Set-AzureRmVMOSDisk -VM $NewVM -VhdUri $OriginalVM.StorageProfile.OsDisk.Vhd.Uri  -Name $OriginalVM.Name -CreateOption Attach -Windows

	#Add Data Disks
	foreach ($disk in $OriginalVM.StorageProfile.DataDisks ) { 
    Add-AzureRmVMDataDisk -VM $newVM -Name $disk.Name -VhdUri $disk.Vhd.Uri -Caching $disk.Caching -Lun $disk.Lun -CreateOption Attach -DiskSizeInGB $disk.DiskSizeGB
	}

	#Add NIC(s)
	foreach ($nic in $OriginalVM.NetworkInterfaceIDs) {
		Add-AzureRmVMNetworkInterface -VM $NewVM -Id $nic
	}

	#Create the VM
	New-AzureRmVM -ResourceGroupName $rg -Location $OriginalVM.Location -VM $NewVM -DisableBginfoExtension
```

## 次のステップ

[データ ディスク](virtual-machines-windows-attach-disk-portal.md)を追加して、VM に記憶域をさらに追加します。

<!---HONumber=AcomDC_0921_2016-->