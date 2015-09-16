<properties
   pageTitle="Azure リソース マネージャーおよび PowerShell を使用した VNet 間の接続の構成 | Microsoft Azure"
	description="この記事では、Azure リソース マネージャーおよび PowerShell を使用して仮想ネットワークどうしを接続する方法を説明します。"
	services="vpn-gateway"
	documentationCenter="na"
	authors="cherylmc"
	manager="carolz"
	editor=""/>

<tags
   ms.service="vpn-gateway"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="infrastructure-services"
	ms.date="08/20/2015"
	ms.author="cherylmc"/>

# Azure リソース マネージャーおよび PowerShell を使用した VNet 間の接続の構成

> [AZURE.SELECTOR]
- [Azure Portal](virtual-networks-configure-vnet-to-vnet-connection.md)
- [PowerShell - Azure Resource Manager](vpn-gateway-vnet-vnet-rm-ps.md)


仮想ネットワークどうし (VNet 間) の接続は、仮想ネットワークをオンプレミス サイトの場所に接続することとよく似ています。どちらの接続タイプでも、VPN ゲートウェイを使用して、IPsec/IKE を使った安全なトンネルが確保されます。接続する VNet は、リージョンが異なっていてもかまいません。マルチサイト構成と VNet 間通信を組み合わせることもできます。そのため、クロスプレミス接続と仮想ネットワーク間接続とを組み合わせたネットワーク トポロジを確立することができます (下図参照)。


![VNet 間接続の図](./media/virtual-networks-configure-vnet-to-vnet-connection/IC727360.png)

 

>[AZURE.NOTE]現在、Azure には、従来のデプロイ モードと Azure リソース マネージャーのデプロイ モードの 2 つのデプロイ モードがあります。構成コマンドレットと手順は、デプロイ モードによって異なります。このトピックでは、Azure リソース マネージャー モードで作成された仮想ネットワークを接続する方法について説明します。従来のデプロイ モードを使用して VNet 間接続を作成する場合は、[Azure ポータルを使用した VNet 間の接続の構成](virtual-networks-configure-vnet-to-vnet-connection.md)を参照してください。従来のモードで作成された仮想ネットワークを、Azure リソース マネージャーで作成された仮想ネットワークに接続する場合は、[従来の VNet を新しい VNet に接続する](../virtual-network/virtual-networks-arm-asm-s2s.md)を参照してください。

## 仮想ネットワークを接続する理由

仮想ネットワークを接続するのは次のような場合です。

- **リージョン間の geo 冗長性および geo プレゼンス**
	- インターネット接続エンドポイントを介さず、安全な接続を使って独自の geo レプリケーションや geo 同期をセットアップすることができます。
	- Azure Load Balancer と Microsoft (またはサード パーティ) のクラスタリング テクノロジを使用し、複数の Azure リージョンをまたぐ geo 冗長性を備えた、可用性に優れたワークロードをセットアップすることができます。たとえば、複数の Azure リージョンにまたがる SQL AlwaysOn 可用性グループをセットアップすることができます。

- **特定の地域内で強固な分離境界を備えた多層アプリケーション**
	- 同じ地域内の相互に接続された複数の仮想ネットワークを使って多層アプリケーションをセットアップすることができます。それぞれの層のアプリケーションが強固に分離され、安全な層間通信を実現することができます。



### 注意する点

この記事では、VNet1 と VNet2 の 2 つの仮想ネットワークを接続する手順を見ていきます。実際のネットワーク設計の要件に対応した IP アドレス範囲に置き換えるためには、ネットワークに関する知識が必要です。


![VNet 間の接続](./media/virtual-networks-configure-vnet-to-vnet-connection/IC727361.png)



- 仮想ネットワークが属している Azure リージョン (場所) は異なっていてもかまいません。

- クラウド サービスや負荷分散エンドポイントは、仮にそれらが相互に接続されていたとしても、仮想ネットワークの境界を越えることはできません。

- 複数の Azure Virtual Network を接続するときに、オンプレミスの VPN ゲートウェイは必要ありません (クロスプレミス接続が必要な場合を除く)。

- VNet 間接続によって仮想ネットワークを接続できます。仮想ネットワーク内に存在しない仮想マシンやクラウド サービスを接続することはできません。

- VNet 間接続には、RouteBased VPN タイプの Azure VPN ゲートウェイが必要です (以前は「動的ゲートウェイ」と呼ばれていました) 。

- 仮想ネットワーク接続は、マルチサイト VPN と同時に使用することができます。1 つの仮想ネットワーク VPN ゲートウェイに最大 10 本の VPN トンネルを確立し、他の仮想ネットワークまたはオンプレミス サイトに接続することが可能です。

- 仮想ネットワークのアドレス空間とオンプレミスのローカル ネットワーク サイトのアドレス空間とが重複しないようにする必要があります。アドレス空間が重複していると、仮想ネットワークの作成は失敗します。

- 一対の仮想ネットワーク間に冗長トンネルを確立することはできません。

- 仮想ネットワークのすべての VPN トンネルは、Azure VPN ゲートウェイ上の使用可能な帯域幅を共有し、Azure 内の同じ VPN ゲートウェイ アップタイム SLA を共有します。

- VNet 間のトラフィックは、Azure バックボーン経由で送信できます。


## 作業を開始する前に


開始する前に、以下がそろっていることを確認します。


- 最新バージョンの Azure PowerShell コマンドレット。最新バージョンは、[ダウンロード ページ](http://azure.microsoft.com/downloads/)の Windows PowerShell セクションからダウンロードしてインストールできます。 
- Azure サブスクリプション。Azure サブスクリプションを持っていない場合は、[MSDN サブスクライバーの特典](http://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)を有効にするか、[無料試用版](http://azure.microsoft.com/pricing/free-trial/)にサインアップしてください。
- 既に仮想ネットワークを作成済みの場合は、[既存 VNet の接続](#connecting-existing-vnets)を参照してください。
	


VNet 間接続を作成および構成する手順は複数あります。以下の順にそれぞれのセクションを構成してください。


1. [IP アドレス範囲を決める](#plan-your-ip-address-ranges)
2. [サブスクリプションへの接続](#connect-to-your-subscription)
3. [仮想ネットワークの作成](#create-a-virtual-network)
4. [ゲートウェイのパブリック IP アドレスの要求](#request-a-public-ip-address)
5. [ゲートウェイ構成の作成](#create-the-gateway-configuration)
6. [ゲートウェイの作成](#create-the-gateway)
7. [手順を繰り返して VNet2 を構成](#create-vnet2)
8. [VPN ゲートウェイの接続](#connect-the-gateways)


## IP アドレス範囲を決める

### 手順 1.

ネットワーク構成に使用する範囲を決めることは重要です。VNet1 から見ると、VNet2 は、Azure プラットフォームに定義されている別の VPN 接続にすぎません。また、VNet2 から見れば、VNet1 も別の VPN 接続です。VNet の範囲やローカル ネットワークの範囲が重複することは、どのような形であれ許容されないので注意してください。


以下の手順に従って、2 つの仮想ネットワークを、それぞれのゲートウェイ サブネットおよび構成とともに作成します。その後に、これら 2 つの VNet 間の VPN ゲートウェイ接続を作成します。

この演習では、VNet に次の値を使用します。

VNet1 の値:

- 仮想ネットワーク名 = VNet1
- リソース グループ = testrg1
- アドレス空間: 10.1.0.0/16 
- リージョン = 米国西部
- GatewaySubnet = 10.1.0.0/28
- Subnet1 = 10.1.1.0/28

VNet2 の値:

- 仮想ネットワーク名 = VNet2
- リソース グループ = testrg2
- アドレス空間: 10.2.0.0/16
- リージョン = 東日本
- GatewaySubnet = 10.2.0.0/28
- Subnet1 = 10.2.1.0/28

## サブスクリプションへの接続 

### 手順 2.


PowerShell コンソールを開き、Azure リソース マネージャー モードに切り替えます。接続するには、次のサンプルを参照してください。

		Add-AzureAccount

Select-AzureSubscription を実行して、使用するサブスクリプションに接続します。

		Select-AzureSubscription "yoursubscription"

次に、ARM モードに切り替えます。ARM モードを使用できるモードに切り替わります。

		Switch-AzureMode -Name AzureResourceManager

## 仮想ネットワークの作成

### 手順 3.


仮想ネットワークとゲートウェイ サブネットを作成するには、次のサンプルを使用してください。サンプル値は実際の値に変更してください。この例では、VNet1 を作成します。後で同じ手順を繰り返して VNet2 を作成します。

まず、リソース グループを作成します。

			New-AzureResourceGroup -Name testrg1 -Location 'West US'

次に、仮想ネットワークを作成します。次のサンプルでは、*VNet1* という仮想ネットワークと、*GatewaySubnet* と *Subnet1* という 2 つのサブネットを作成します。正確に *GatewaySubnet* というサブネットを作成する必要があります。別の名前にすると、接続の構成は失敗します。次の例では、ゲートウェイ サブネットで /28 を使用しています。ゲートウェイ サブネットには最小値 /29 まで使用できます。機能によっては (ExpressRoute とサイト間 VPN が共存する接続など)、/27 より大きなゲートウェイ サブネットが必要となる場合があるため、将来追加する可能性がある機能に対応できるようにゲートウェイ サブネットを作成してください。

 		$subnet = New-AzureVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.0.0/28
		$subnet1 = New-AzureVirtualNetworkSubnetConfig -Name 'Subnet1' -AddressPrefix '10.1.1.0/28'
		New-AzurevirtualNetwork -Name VNet1 -ResourceGroupName testrg1 -Location 'West US' -AddressPrefix 10.1.0.0/16 -Subnet $subnet, $subnet1

## パブリック IP アドレスの要求

### 手順 4.

次に、VNet で作成するゲートウェイに割り当てるパブリック IP アドレスを要求します。使用する IP アドレスは指定できません。IP アドレスはゲートウェイに動的に割り当てられます。この IP アドレスは、次の構成セクションで使用します。

以下のサンプルを使用します。このアドレスの割り当て方法は動的にする必要があります。


		$gwpip= New-AzurePublicIpAddress -Name gwpip1 -ResourceGroupName testrg1 -Location 'West US' -AllocationMethod Dynamic


## ゲートウェイ構成の作成

### 手順 5

ゲートウェイの構成で、使用するサブネットとパブリック IP アドレスを定義します。次のサンプルを使用して、ゲートウェイの構成を作成します。


		$vnet = Get-AzureVirtualNetwork -Name VNet1 -ResourceGroupName testrg1
		$subnet = Get-AzureVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
		$gwipconfig = New-AzureVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id 


## ゲートウェイの作成

### 手順 6

この手順では、VNet の仮想ネットワーク ゲートウェイを作成します。VNet 間構成では、RouteBased VpnType が必要です。ゲートウェイの作成には時間がかかるため、気長に取り組んでください。

		New-AzureVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg1 -Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn -VpnType RouteBased

## VNet2 の作成

### 手順 7.

VNet1 の構成後、前述の手順を繰り返して、VNet2 とそのゲートウェイ構成を構成します。これら 2 つの VNet とそれぞれのゲートウェイの構成が完了したら、 [ゲートウェイの接続](#connect-the-gateways)に進みます。


## ゲートウェイの接続

### 手順 8.

このステップでは、2 つの仮想ネットワーク ゲートウェイ間の VPN ゲートウェイ接続を作成します。この例では、参照される共有キーが表示されます。共有キーには独自の値を使用することができます。両方の構成の共有キーが一致することが重要です。

接続の作成には、多少時間がかかります。

**VNet1 から VNet2 の場合**
    
    $vnetgw1 = Get-AzureVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg1
    $vnetgw2 = Get-AzureVirtualNetworkGateway -Name vnetgw2 -ResourceGroupName testrg2
    
    New-AzureVirtualNetworkGatewayConnection -Name conn1 -ResourceGroupName testrg1 -VirtualNetworkGateway1 $vnetgw1 -VirtualNetworkGateway2 $vnetgw2 -Location 'West US' -ConnectionType Vnet2Vnet -SharedKey 'abc123'


**VNet2 から VNet1 の場合**
    
    $vnetgw1 = Get-AzureVirtualNetworkGateway -Name vnetgw2 -ResourceGroupName testrg2
    $vnetgw2 = Get-AzureVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg1
    
    New-AzureVirtualNetworkGatewayConnection -Name conn2 -ResourceGroupName testrg2 -VirtualNetworkGateway1 $vnetgw1 -VirtualNetworkGateway2 $vnetgw2 -Location 'Japan East' -ConnectionType Vnet2Vnet -SharedKey 'abc123'
    


数分後に接続が確立されます。この時点では、Azure リソース マネージャーで作成されたゲートウェイおよび接続は、プレビュー ポータルでは表示されません。

## 既存 VNet の接続

Azure リソース マネージャー モードで既に作成した複数の仮想ネットワークを接続するには、次を確認します。

- 各 VNet には最低 /29 またはそれ以上の規模のゲートウェイ サブネットがある。
- 仮想ネットワークのアドレス範囲が重複していない。

VNet にゲートウェイ サブネットを追加する必要がある場合は、次のサンプルを使用して、値を置き換えます。ゲートウェイ サブネットには、必ず 'GatewaySubnet' という名前を付けてください。別の名前にすると、VPN の構成は期待どおりに機能しません。

	
		$vnet = Get-AzureVirtualNetwork -ResourceGroupName testrg -Name testvnet
		Add-AzureVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/28 -VirtualNetwork $vnet
		Set-AzureVirtualNetwork -VirtualNetwork $vnet

ゲートウェイ サブネットが正しく構成されていることを確認したら、引き続き[パブリック IP アドレスを要求](#request-a-public-ip-address)の作業を続け、その後の手順に従います。

## 次のステップ

仮想ネットワークに仮想マシンを追加できます。[仮想マシンの作成](../virtual-machines/virtual-machines-windows-tutorial.md)。

<!---HONumber=August15_HO9-->