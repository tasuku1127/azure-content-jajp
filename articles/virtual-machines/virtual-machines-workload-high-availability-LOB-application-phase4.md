<properties 
	pageTitle="基幹業務アプリケーションのワークロード フェーズ 4: Web サーバーを構成する" 
	description="高可用な基幹業務アプリケーションを Azure インフラストラクチャ サービスにデプロイする作業のこの第 4 フェーズでは、Web サーバーを作成して基幹業務アプリケーションをロードします。" 
	documentationCenter=""
	services="virtual-machines" 
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""
	tags="azure-resource-manager"/>

<tags 
	ms.service="virtual-machines" 
	ms.workload="infrastructure-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="08/11/2015" 
	ms.author="josephd"/>

# 基幹業務アプリケーションのワークロード フェーズ 4: Web サーバーを構成する

高可用な基幹業務アプリケーションを Azure インフラストラクチャ サービスにデプロイする作業のこのフェーズでは、Web サーバーを構築して基幹業務アプリケーションをロードします。

[フェーズ 5](virtual-machines-workload-high-availability-LOB-application-phase5.md) に進むには、このフェーズを完了する必要があります。全フェーズについては、「[Azure での高可用な基幹業務アプリケーションのデプロイ](virtual-machines-workload-high-availability-LOB-application-overview.md)」をご覧ください。

## Azure での Web サーバー仮想マシンの作成

ASP.NET アプリケーション、または Windows Server 2012 R2 のインターネット インフォメーション サービス (IIS) 8 でホストできる古いアプリケーションをデプロイできる、2 つの Web サーバー仮想マシンがあります。

最初に、基幹業務アプリケーションへのクライアント トラフィックが、Azure によって 2 つの Web サーバーに均等に配分されるように、内部ロード バランサーを構成します。この場合、名前とサブネットのアドレス空間 (Azure 仮想ネットワークに割り当てたアドレス空間) から割り当てられた独自の IP アドレスで構成される内部負荷分散インスタンスを指定する必要があります。内部ロード バランサー用に選択した IP アドレスが使用できるかどうかを確認するには、Azure PowerShell プロンプトで次のコマンドを使用します。< and > の文字を削除して、各変数の値を指定します。

	Switch-AzureMode AzureServiceManagement
	$vnet="<Table V – Item 1 – Value column>"
	$testIP="<a chosen IP address from the subnet address space, Table S - Item 1 – Subnet address space column>"
	Test-AzureStaticVNetIP –VNetName $vnet –IPAddress $testIP

Test-AzureStaticVNetIP コマンドで表示される **IsAvailable** フィールドが **True** の場合は、その IP アドレスを使用できます。

次のコマンドを実行して、リソース マネージャー モードの PowerShell に戻します。

	Switch-AzureMode AzureResourceManager

次に、変数を入力し、以下の一連のコマンドを実行します。

	$rgName="<resource group name>"
	$locName="<Azure location of your resource group>"
	$vnetName="<Table V – Item 1 – Value column>"
	$privIP="<available IP address on the subnet>"
	$vnet=Get-AzureVirtualNetwork -Name $vnetName -ResourceGroupName $rgName

	$frontendIP=New-AzureLoadBalancerFrontendIpConfig -Name WebServers-LBFE -PrivateIPAddress $privIP -SubnetId $vnet.Subnets[1].Id
	$beAddressPool=New-AzureLoadBalancerBackendAddressPoolConfig -Name WebServers-LBBE

	# This example assumes unsecured (HTTP-based) web traffic to the web servers.
	$healthProbe=New-AzureLoadBalancerProbeConfig -Name WebServersProbe -Protocol "TCP" -Port 80 -IntervalInSeconds 15 -ProbeCount 2
	$lbrule=New-AzureLoadBalancerRuleConfig -Name "WebTraffic" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol "TCP" -FrontendPort 80 -BackendPort 80
	New-AzureLoadBalancer -ResourceGroupName $rgName -Name "WebServersInAzure" -Location $locName -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe -FrontendIpConfiguration $frontendIP

次に、基幹業務アプリケーションの完全修飾ドメイン名 (lobapp.corp.contoso.com など) を、内部ロード バランサーに割り当てられている IP アドレス (前の Azure PowerShell コマンド ブロックの $privIP の値) に解決する DNS アドレス レコードを、組織の内部 DNS インフラストラクチャに追加します。

次に、PowerShell コマンドの次のブロックを使用して、2 つの Web サーバーの仮想マシンを作成します。この PowerShell コマンド セットでは、次の表の値を使用します。

- 表 M (仮想マシン)
- 表 V (仮想ネットワーク設定)
- 表 S (サブネット)
- 表 ST (ストレージ アカウント)
- 表 A (可用性セット)

表 M は[フェーズ 2](virtual-machines-workload-high-availability-LOB-application-phase2.md) で、V、S、ST、Aの各表は[フェーズ 1](virtual-machines-workload-high-availability-LOB-application-phase1.md) で定義したものです。

適切な値をすべて指定したら、Azure PowerShell プロンプトでそのブロックを実行します。

	# Set up subscription and key variables
	$subscr="<name of the Azure subscription>"
	Set-AzureSubscription -SubscriptionName $subscr
	Switch-AzureMode AzureResourceManager
	$rgName="<resource group name>"
	$locName="<Azure location of your resource group>"
	$webLB=Get-AzureLoadBalancer -ResourceGroupName $rgName -Name "WebServersInAzure"	
	
	# Use the standard storage account
	$saName="<Table ST – Item 2 – Storage account name column>"

	$vnetName="<Table V – Item 1 – Value column>"
	$beSubnetName="<Table S - Item 2 - Name column>"
	$avName="<Table A – Item 3 – Availability set name column>"
	$vnet=Get-AzurevirtualNetwork -Name $vnetName -ResourceGroupName $rgName
	$backendSubnet=Get-AzureVirtualNetworkSubnetConfig -Name $beSubnetName -VirtualNetwork $vnet
	
	# Create the first web server virtual machine
	$vmName="<Table M – Item 6 - Virtual machine name column>"
	$vmSize="<Table M – Item 6 - Minimum size column>"
	$nic=New-AzureNetworkInterface -Name ($vmName + "-NIC") -ResourceGroupName $rgName -Location $locName -Subnet $backendSubnet -LoadBalancerBackendAddressPool $webLB.BackendAddressPools[0]
	$avSet=Get-AzureAvailabilitySet -Name $avName –ResourceGroupName $rgName 
	$vm=New-AzureVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the first web server." 
	$vm=Set-AzureVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureVM -ResourceGroupName $rgName -Location $locName -VM $vm
	
	# Create the second web server virtual machine
	$vmName="<Table M – Item 7 - Virtual machine name column>"
	$vmSize="<Table M – Item 7 - Minimum size column>"
	$nic=New-AzureNetworkInterface -Name ($vmName + "-NIC") -ResourceGroupName $rgName -Location $locName -Subnet $backendSubnet -LoadBalancerBackendAddressPool $webLB.BackendAddressPools[0]
	$vm=New-AzureVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $avset.Id
	$cred=Get-Credential -Message "Type the name and password of the local administrator account for the second second SQL Server computer." 
	$vm=Set-AzureVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
	$vm=Set-AzureVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
	$vm=Add-AzureVMNetworkInterface -VM $vm -Id $nic.Id
	$storageAcc=Get-AzureStorageAccount -ResourceGroupName $rgName -Name $saName
	$osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/" + $vmName + "-OSDisk.vhd"
	$vm=Set-AzureVMOSDisk -VM $vm -Name "OSDisk" -VhdUri $osDiskUri -CreateOption fromImage
	New-AzureVM -ResourceGroupName $rgName -Location $locName -VM $vm

好みのリモート デスクトップ クライアントを使用し、Web サーバー仮想マシンのそれぞれへのリモート デスクトップ接続を作成します。仮想マシンのイントラネット DNS 名またはコンピューター名と、ローカル管理者アカウントの資格情報を使用します。

次に、Windows PowerShell プロンプトで次のコマンドを実行し、各 Web サーバー仮想マシンを適切な Active Directory ドメインに参加させます。

	$domName="<Active Directory domain name to join, such as corp.contoso.com>"
	Add-Computer -DomainName $domName
	Restart-Computer

**Add-Computer** コマンドを入力した後、ドメイン アカウントの資格情報を指定する必要があることに注意してください。

仮想マシンの再起動後、ローカル管理者特権を持つアカウントを使用して再接続します。

次に、各 Web サーバーに IIS をインストールして構成します。

1. サーバー マネージャーを実行し、**[役割と機能の追加]** をクリックします。
2. [開始する前に] ページで **[次へ]** をクリックします。
3. [インストールの種類の選択] ページで **[次へ]** をクリックします。
4. [対象サーバーの選択] ページで **[次へ]** をクリックします。
5. [サーバーの役割] ページの **[役割]** の一覧で、**[Web サーバー (IIS)]** をクリックします。
6. メッセージが表示されたら、**[機能の追加]** をクリックし、**[次へ]** をクリックします。
7. [機能の選択] ページで **[次へ]** をクリックします。
8. [Web サーバー (IIS)] ページで **[次へ]** をクリックします。
9. [役割サービスの選択] ページで、LOB アプリケーションでの必要性に応じて、サービスのチェック ボックスをオンまたはオフにし、**[次へ]** をクリックします。10. [インストール オプションの確認] ページで、**[インストール]** をクリックします。

## Web サーバー仮想マシンへの基幹業務アプリケーションのデプロイ

オンプレミス ネットワーク上のコンピューターで、次を実行します。

1.	基幹業務アプリケーション用のファイルを 2 つの Web サーバーに追加します。
2.	SQL Server クラスターに基幹業務アプリケーション用のデータベースを作成します。
3.	基幹業務アプリケーションへのアクセスとその機能をテストします。

次の図は、このフェーズが問題なく完了したときの構成を示しています。

![](./media/virtual-machines-workload-high-availability-LOB-application-phase4/workload-lobapp-phase4.png)

## 次の手順

このワークロードを引き続き構成するには、「[フェーズ 5: 可用性グループを作成してアプリケーション データベースを追加する](virtual-machines-workload-high-availability-LOB-application-phase5.md)」に進んでください。

## その他のリソース

[Azure での高可用な基幹業務アプリケーションのデプロイ](virtual-machines-workload-high-availability-LOB-application-overview.md)

[基幹業務アプリケーションのアーキテクチャ ブループリント](http://msdn.microsoft.com/dn630664)

[テスト用のハイブリッド クラウドでの Web ベース LOB アプリケーションの設定](../virtual-network/virtual-networks-setup-lobapp-hybrid-cloud-testing.md)

[Azure インフラストラクチャ サービス実装ガイドライン](virtual-machines-infrastructure-services-implementation-guidelines.md)

<!---HONumber=August15_HO7-->