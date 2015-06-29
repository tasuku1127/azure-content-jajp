<properties
	pageTitle="登録と保護の管理" 
	description="Azure Site Recovery は、オンプレミスのサーバーに配置されている仮想マシンの Azure またはセカンダリ データセンターへのレプリケーション、フェールオーバー、および復旧を調整します。この記事を使用して、Site Recovery コンテナーからサーバーの登録を解除して、仮想マシンと物理サーバーの保護を無効にします。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="rayne-wiselman" 
	manager="jwhit" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery" 
	ms.date="05/29/2015" 
	ms.author="raynew"/>

# 登録と保護の管理

この記事では、Site Recovery コンテナーからサーバーの登録を解除する方法と Site Recovery によって保護される仮想マシンの保護を無効にする方法について説明します。この記事の内容について質問がある場合は、[Azure Recovery Services フォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr)に投稿してください。

## VMM サーバーの登録解除

コンテナーから VMM サーバーの登録を解除するには、Azure Site Recovery ポータルの **[サーバー]** タブでサーバーを削除します。以下の点に注意してください。

-  Azure に接続されているときに、VMM サーバーの登録を解除することをお勧めします。これにより、オンプレミスの VMM サーバーの設定、およびオンプレミスの VMM サーバーと関連付けられている VMM サーバー (削除するサーバー上のクラウドにマップされているクラウドを含む VMM サーバー) の設定が正しくクリーンアップされます。接続に永続的な問題がある場合は、接続されていないサーバーのみを削除することをお勧めします。
- VMM サーバーを削除するときに、VMM サーバーが接続されていない場合は、手動でスクリプトを実行して、クリーンアップを実行する必要があります。スクリプトは、[Microsoft ギャラリー](https://gallery.technet.microsoft.com/scriptcenter/Cleanup-Script-for-Windows-95101439)で使用可能です。手動のクリーンアップ プロセスを完了するために、サーバーの VMM ID を書き留めてください。
- クラスターにデプロイされている VMM サーバーの登録を解除する必要がある場合は、次の操作を行います。

	- サーバーが接続されている場合は、**[サーバー]** タブで、接続されている VMM サーバーを削除します。サーバー上のプロバイダーをアンインストールするには、各クラスター ノードにログインし、コントロール パネルからアンインストールします。登録エントリを削除するには、クラスター内のすべてのパッシブ ノードで、前のセクションに記載されたクリーンアップ スクリプトを実行します。
	- サーバーが接続されていない場合は、すべてのクラスター ノードでクリーンアップ スクリプトを実行する必要があります。

### 接続されていない VMM サーバーの登録解除

削除する VMM サーバーで、次の手順を実行します。

1. Azure ポータルから VMM サーバーの登録を解除します。
2. VMM サーバーで、クリーンアップ スクリプトをダウンロードします。
3. [管理者として実行] オプションを使用して PowerShell を開き、既定 (LocalMachine) のスコープの実行ポリシーを変更します。
4. スクリプトの手順に従います。 

削除するサーバー上のクラウドとペアになっているクラウドが存在する VMM サーバーで、次の手順を実行します。

1. クリーンアップ スクリプトを実行し、ステップ 2 ～ 4 に従います。
2. 登録が解除されている VMM サーバーの VMM ID を指定します。 
3. このスクリプトは、VMM サーバーの登録情報とクラウドのペアリング情報を削除します。


## Hyper-V サイトでの Hyper-V サーバーの登録解除

(VMM サーバーがない) Hyper-V サイトの Hyper-V サーバー上にある仮想マシンを保護するために Azure Site Recovery がデプロイされている場合、次のように、コンテナーから Hyper-V サーバーの登録を解除できます。

1. Hyper-V サーバーに配置された仮想マシンの保護を無効にします。
2. Azure Site Recovery ポータルの **[サーバー]** タブで、サーバーを選択してから、[削除] を選択します。この操作を実行するときに、このサーバーが Azure に接続されている必要はありません。
3. 次のスクリプトを実行して、サーバー上の設定をクリーンアップし、コンテナーからサーバーの登録を解除します。 

	    pushd .
	    try
	    {
		     $windowsIdentity=[System.Security.Principal.WindowsIdentity]::GetCurrent()
		     $principal=new-object System.Security.Principal.WindowsPrincipal($windowsIdentity)
		     $administrators=[System.Security.Principal.WindowsBuiltInRole]::Administrator
		     $isAdmin=$principal.IsInRole($administrators)
		     if (!$isAdmin)
		     {
		        "Please run the script as an administrator in elevated mode."
		        $choice = Read-Host
		        return;       
		     }
	
		    $error.Clear()    
			"This script will remove the old Azure Site Recovery Provider related properties. Do you want to continue (Y/N) ?"
			$choice =  Read-Host
		
			if (!($choice -eq 'Y' -or $choice -eq 'y'))
			{
			"Stopping cleanup."
			return;
			}
		
			$serviceName = "dra"
			$service = Get-Service -Name $serviceName
			if ($service.Status -eq "Running")
		    {
				"Stopping the Azure Site Recovery service..."
				net stop $serviceName
			}
		
		    $asrHivePath = "HKLM:\SOFTWARE\Microsoft\Azure Site Recovery"
			$registrationPath = $asrHivePath + '\Registration'
			$proxySettingsPath = $asrHivePath + '\ProxySettings'
		    $draIdvalue = 'DraID'
	    
		    if (Test-Path $asrHivePath)
		    {
		        if (Test-Path $registrationPath)
		        {
		            "Removing registration related registry keys."	
			        Remove-Item -Recurse -Path $registrationPath
		        }
	
		        if (Test-Path $proxySettingsPath)
	        {
			        "Removing proxy settings"
			        Remove-Item -Recurse -Path $proxySettingsPath
		        }
	
		        $regNode = Get-ItemProperty -Path $asrHivePath
		        if($regNode.DraID -ne $null)
		        {            
		            "Removing DraId"
		            Remove-ItemProperty -Path $asrHivePath -Name $draIdValue
		        }
			    "Registry keys removed."
		    }
	
		    # First retrive all the certificates to be deleted
			$ASRcerts = Get-ChildItem -Path cert:\localmachine\my | where-object {$_.friendlyname.startswith('ASR_SRSAUTH_CERT_KEY_CONTAINER') -or $_.friendlyname.startswith('ASR_HYPER_V_HOST_CERT_KEY_CONTAINER')}
			# Open a cert store object
			$store = New-Object System.Security.Cryptography.X509Certificates.X509Store("My","LocalMachine")
			$store.Open('ReadWrite')
			# Delete the certs
			"Removing all related certificates"
		    foreach ($cert in $ASRcerts)
		    {
			    $store.Remove($cert)
		    }
	    }catch
	    {	
		    [system.exception]
		    Write-Host "Error occured" -ForegroundColor "Red"
		    $error[0] 
		    Write-Host "FAILED" -ForegroundColor "Red"
	    }
	    popd


## 仮想マシンの保護を停止する

仮想マシンの保護を停止するには、次の操作を行います。

1. クラウドのプロパティの **[仮想マシン]** タブで、仮想マシンを選択してから、**[削除]** を選択します。
2. **[仮想マシンの削除の確認]** ページには、2 つのオプションがあります。

	- **仮想マシンの保護の無効化**—このオプションを有効にして保存すると、仮想マシンは Azure Site Recovery によって保護されなくなります。仮想マシンの保護設定は、自動的にクリーンアップされます。
	- **[仮想マシンの管理を停止します]**—このオプションを選択すると、Azure Site Recovery サービスの仮想マシンのみが削除されます。仮想マシンに対するオンプレミスの保護設定には影響しないため、次の手順を使用して手動でクリーンアップする必要があります。

仮想マシンとそのハード ディスクを削除することを選択した場合は、それらはターゲットの場所から削除されます。

### 保護設定を手動でクリーンアップする (VMM サイト間)

**[仮想マシンの管理の停止]** を選択した場合は、次のように手動で設定をクリーンアップします。

1. プライマリ サーバーで VMM コンソールからこのスクリプトを実行し、プライマリ仮想マシンの設定をクリーンアップします。VMM コンソールで、[PowerShell] ボタンをクリックし、VMM PowerShell コンソールを開きます。SQLVM1 を仮想マシンの名前に置き換えます。

	     $vm = get-scvirtualmachine -Name "SQLVM1"
	     Set-SCVirtualMachine -VM $vm -ClearDRProtection

2. 次のように、セカンダリ VMM サーバーでこのスクリプトを実行して、セカンダリ仮想マシンの設定をクリーンアップします。

	    $vm = get-scvirtualmachine -Name "SQLVM1"
	    Remove-SCVirtualMachine -VM $vm -Force

3. セカンダリ VMM サーバーで、スクリプトを実行した後に Hyper-V ホスト サーバー上の仮想マシンを更新し、セカンダリ仮想マシンが VMM コンソールで再検出されるようにします。
4. 上記の手順によって、VMM サーバーのみのレプリケーション設定がクリアされます。仮想マシンの仮想マシン レプリケーションを削除する場合、プライマリおよびセカンダリ仮想マシンの両方に対して以下の手順を実行する必要があります。以下のスクリプトを実行してレプリケーションを削除し、SQLVM1 を仮想マシンの名前に置き換えます。

	    Remove-VMReplication –VMName “SQLVM1”


### 保護設定を手動でクリーンアップする (オンプレミスの VMM サイトと Azure の間)

1. 次のように、ソース VMM サーバーでこのスクリプトを実行して、プライマリ仮想マシンの設定をクリーンアップします。

	    $vm = get-scvirtualmachine -Name "SQLVM1"
	    Set-SCVirtualMachine -VM $vm -ClearDRProtection

2. 上記の手順によって、VMM サーバーのみのレプリケーション設定がクリアされます。VMM サーバーからレプリケーションを削除したら、このスクリプトを使用して Hyper-V ホスト サーバー上で実行されている仮想マシンのレプリケーションを削除してください。SQLVM1 を仮想マシンの名前で、host01.contoso.com を Hyper-V ホスト サーバーの名前に置き換えます。

	    $vmName = "SQLVM1"
	    $hostName  = "host01.contoso.com"
	    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'" -computername $hostName
	    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"  -computername $hostName
	    $replicationService.RemoveReplicationRelationship($vm.__PATH)


### 保護設定を手動でクリーンアップする (Hyper-V サイトと Azure の間)

1. ソース Hyper-V ホスト サーバーで仮想マシンのレプリケーションを削除するには、このスクリプトを使用します。SQLVM1 を仮想マシンの名前に置き換えます。

	    $vmName = "SQLVM1"
	    $vm = Get-WmiObject -Namespace "root\virtualization\v2" -Query "Select * From Msvm_ComputerSystem Where ElementName = '$vmName'"
	    $replicationService = Get-WmiObject -Namespace "root\virtualization\v2"  -Query "Select * From Msvm_ReplicationService"
	    $replicationService.RemoveReplicationRelationship($vm.__PATH)

### 保護設定を手動でクリーンアップする (VMware サイトと Azure の間)

ソース サーバーで、モビリティ サービスをアンインストールします。














 

<!---HONumber=58_postMigration-->