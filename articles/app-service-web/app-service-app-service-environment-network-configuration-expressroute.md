<properties 
	pageTitle="ExpressRoute を操作するためのネットワーク構成の詳細" 
	description="App Service Environment を ExpressRoute 回線に接続された Virtual Networks 内で実行するためのネットワーク構成の詳細です。" 
	services="app-service" 
	documentationCenter="" 
	authors="stefsch" 
	manager="nirma" 
	editor=""/>

<tags 
	ms.service="app-service" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="10/23/2015" 
	ms.author="stefsch"/>

# ExpressRoute を使用した App Service Environment のネットワーク構成の詳細 

## 概要 ##
顧客は、[Azure ExpressRoute][ExpressRoute] 回線を自分の仮想ネットワーク インフラストラクチャに接続することで、オンプレミスのネットワークを Azure に拡張できます。この[仮想ネットワーク][virtualnetwork] インフラストラクチャのサブネットの中に App Service Environment を作成できます。App Service Environment で実行されるアプリは、ExpressRoute 接続でのみアクセスできる、バックエンド リソースに対する安全な接続を確立できます。

**注:** "v2"仮想ネットワーク内で App Service Environment を作成することはできません。App Service Environment は現在、クラシック "v1" 仮想ネットワークでしかサポートされていません。

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## 必要なネットワーク接続 ##
ExpressRoute に接続された仮想ネットワークでは最初は満たされていない場合がある App Service Environment のネットワーク接続要件があります。App Service Environment が正常に機能するには、次のすべてを満たす必要があります。


-  世界各国の Azure Storage エンドポイントに対する発信ネットワーク接続これには、App Service Environment と同じリージョンにあるエンドポイントと、**他の** Azure リージョンにあるストレージ エンドポイントが含まれます。Azure Storage エンドポイントは、次の DNS ドメインで解決されます: *table.core.windows.net*、*blob.core.windows.net*、*queue.core.windows.net*、*file.core.windows.net*。  
-  App Service Environment と同じリージョンにある Sql DB エンドポイントに対する発信ネットワーク接続。SQL DB エンドポイントは、次のドメインで解決されます: *database.windows.net*。
-  Azure 管理プレーン エンドポイント (ASM エンドポイントと ARM エンドポイントの両方) に対する発信ネットワーク接続これには、*management.core.windows.net* と *management.azure.com* の両方に対する発信接続が含まれます。 
-  *ocsp.msocsp.com* への送信ネットワーク接続これは、SSL 機能をサポートするために必要です。
-  仮想ネットワークの DNS 構成は、前述したすべてのエンドポイントとドメインを解決できるようにする必要があります。これらのエンドポイントを解決できない場合、App Service Environment の作成処理に失敗し、既存の App Service Environment は異常とマークされます。
-  カスタム DNS サーバーが VPN ゲートウェイの相手側にある場合、DNS サーバーは App Service Environment を含むサブネットから到達できる必要があります。 
-  発信ネットワーク パスは、社内プロキシを経由したり、オンプレミスに強制的にトンネリングしたりすることができません。実行した場合、App Service Environment からの発信ネットワーク トラフィックの実質的な NAT アドレスが変わります。App Service Environment の発信ネットワーク トラフィックの NAT アドレスを変更すると、上記の多数のエンドポイントに対して接続エラーが発生します。その結果、App Service Environment の作成処理は失敗し、以前は正常動作していた App Service Environment も異常とマークされます。  
-  この[記事][requiredports]の説明に従って、App Service 環境の必要なポートへの着信ネットワーク アクセスを許可する必要があります。

DNS 要件を満たすには、仮想ネットワークの有効な DNS インフラストラクチャを構成し、保守します。何らかの理由で、App Service Environment の作成後に DNS 構成が変わった場合、開発者は強制的に App Service Environment から新しい DNS 構成を選択することができます。[新しい管理ポータル][NewPortal]の App Service Environment 管理ブレードの上部にある [再起動] アイコンを使用して、ローリングする環境の再起動をトリガーすると、新しい DNS 構成が自動的に選択されます。

着信ネットワーク アクセスの要件は、この[記事][requiredports]の説明に従って、必要なアクセスを許可する[ネットワーク セキュリティ グループ][NetworkSecurityGroups]を App Service 環境のサブネットに対して構成することによって満たすことができます。

## App Service Environment の発信ネットワーク接続を有効にする##
既定では、新しく作成された ExpressRoute 回線は、発信インターネット接続を許可する既定のルートをアドバタイズします。この構成によって、App Service Environment は、他の Azure エンドポイントに接続できます。

ただし、顧客の一般的な構成では、発信インターネット トラフィックを強制的にオンプレミスにフローさせる独自の既定のルート (0.0.0.0/0) を定義しています。このトラフィック フローでは、発信トラフィックはオンプレミスでブロックされるか、Azure エンドポイントではもはや有効ではない、認識できないアドレス セットに NAT 処理されるため、App Service Environment では接続は必ず切断されます。

解決策は、App Service Environment を含むサブネットに、1 つ (以上) のユーザー定義ルート (UDR) を定義することです。UDR は、既定のルートに優先するサブネット固有のルートを定義します。

可能であれば、次の構成を使用することをお勧めします。

- ExpressRoute 構成は 0.0.0.0/0 をアドバタイズし、既定でオンプレミスのすべての発信トラフィックを強制的にトンネリングします。
- App Service Environment を含むサブネットに適用される UDR では、次ホップの種類がインターネットの 0.0.0.0/0 を定義します (例について、この記事の後半を参照してください)。

これらの手順の複合的な結果として、サブネット レベル UDR は ExpressRoute 強制トンネリングよりも優先されるので、App Service Environment からの発信インターネット アクセスを確保できます。

**重要:** ExpressRoute 構成でアドバタイズされたルートよりも優先するには、UDR に定義されているルートを詳細にする**必要があります**。以下の例では、0.0.0.0/0 の広域なアドレス範囲を使用しているので、より詳細なアドレス範囲を使用するルート アドバタイズで誤って上書きされる可能性があります。

**非常に重要:** App Service Environment は、**パブリック ピアリング パスからプライベート ピアリング パスに誤ってルートをクロスアドバタイズした** ExpressRoute 構成ではサポートされません。パブリック ピアリングが構成された ExpressRoute 構成は、大規模な Microsoft Azure の IP アドレス範囲について Microsoft からルート アドバタイズを受信します。これらのアドレス範囲がプライベート ピアリング パスで誤ってクロスアドバタイズされている場合、App Service Environment のサブネットからのすべての発信ネットワーク パケットは、誤って顧客のオンプレミス ネットワーク インフラストラクチャに強制的にトンネリングされます。このネットワーク フローでは、App Service Environment が機能しません。この問題を解決するには、パブリック ピアリング パスからプライベート ピアリング パスへのルートのクロスアドバタイズを停止します。

ユーザー定義ルートの背景情報については、この[概要][UDROverview]を参照してください。

ユーザー定義ルートの作成と成の詳細については、この[ハウツー ガイド][UDRHowTo]を参照してください。

## App Service Environment のサンプル UDR 構成 ##

**前提条件**

1. [Azure ダウンロード ページ][AzureDownloads](2015 年 6 月以降) から最新の Azure Powershell をインストールします。[コマンド ライン ツール] の [Windows Powershell] の中に、最新の Powershell コマンドレットをインストールする [インストール] リンクがあります。

2. App Service Environment が独占的に使用する一意のサブネットを作成することをお勧めします。これにより、サブネットに適用される UDR で、App Service Environment 用の発信トラフィックのみが開くことが保証されます。
3. **重要**: App Service 環境は、以下の構成手順が**終了するまで**デプロイしないでください。これにより、App Service Environment をデプロイする前に、発信ネットワーク接続を使用できることを確認できます。

**手順 1: 名前付きのルート テーブルを作成する**

次のスニペットは、Azure の米国西部リージョンに "DirectInternetRouteTable" という名前のルート テーブルを作成します。

    New-AzureRouteTable -Name 'DirectInternetRouteTable' -Location uswest

**手順 2: ルーティング テーブルに 1 つ以上のルートを作成する**

発信インターネット アクセスを有効にするために、ルート テーブルに 1 つ以上のルートを追加する必要があります。

インターネットへの発信アクセスを構成する場合、次のように 0.0.0.0/0 のルートを定義することをお勧めします。
  
    Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 0' -AddressPrefix 0.0.0.0/0 -NextHopType Internet

前述したように、0.0.0.0/0 は広範なアドレス範囲なので、ExpressRoute からアドバタイズされた詳細なアドレス範囲によって上書きされます。前述の推奨に従い、0.0.0.0/0 ルートの UDR を、0.0.0.0/0 のみをアドバタイズする ExressRoute 構成と組み合わせます。

または、Azure で使用されている CIDR 範囲の最新の全一覧をダウンロードする方法もあります。すべての Azure IP アドレス範囲を含む Xml ファイルは、[Microsoft ダウンロード センター][DownloadCenterAddressRanges]で入手できます。

ただし、これらの範囲は時間が経つと変わるので、定期的に UDR を手動で更新して同期状態を保つ必要があります。また、1 つの UDR には 100 ルートという上限があるので、100 ルートの上限に会わせて Azure IP アドレス範囲を "まとめる" 必要があります。このとき、UDR が定義されたルートが、ExpressRoute からアドバタイズされるルートよりも詳細になるようにします。


**手順 3: App Service Environment が含まれるサブネットにルート テーブルを関連付ける**

最後の構成手順は、App Service Environment がデプロイされるサブネットにルート テーブルを関連付けることです。次のコマンドは、"DirectInternetRouteTable" を、最終的に App Service Environment を含む "ASESubnet" に関連付けます。

    Set-AzureSubnetRouteTable -VirtualNetworkName 'YourVirtualNetworkNameHere' -SubnetName 'ASESubnet' -RouteTableName 'DirectInternetRouteTable'


**手順 4: 最後の手順**

ルート テーブルをサブネットにバインドしたら、まずテストを行って、意図した効果が出ていることを確認することを勧めします。たとえば、仮想マシンをサブネットにデプロイし、以下の点を確認します。


- この記事で前述した Azure エンドポイントと Azure 以外のエンドポイントに対する発信トラフィックは、ExpressRoute 回線を**フローしません**。サブネットからの発信トラフィックは、オンプレミスで強制トンネリングされ、App Service Environment の作成は常に失敗するので、この動作を検証することが重要です。 
- 前述のエンドポイントの DNS 参照は、すべて正しく解決されます。 

上記の手順を確認したら、仮想マシンを削除する必要があります。これは、App Service Environment の作成時にサブネットを "空" にする必要があるからです。
 
次は、App Service Environment の作成です。

## 使用の開始

App Service 環境の使用を開始するには、「[App Service 環境の概要][IntroToAppServiceEnvironment]」を参照してください。

Azure App Service プラットフォームの詳細については、[Azure App Service][AzureAppService] に関するページを参照してください。

<!-- LINKS -->
[virtualnetwork]: http://azure.microsoft.com/services/virtual-network/
[ExpressRoute]: http://azure.microsoft.com/services/expressroute/
[requiredports]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-control-inbound-traffic/
[NetworkSecurityGroups]: http://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[UDROverview]: http://azure.microsoft.com/documentation/articles/virtual-networks-udr-overview/
[UDRHowTo]: http://azure.microsoft.com/documentation/articles/virtual-networks-udr-how-to/
[HowToCreateAnAppServiceEnvironment]: http://azure.microsoft.com/documentation/articles/app-service-web-how-to-create-an-app-service-environment/
[AzureDownloads]: http://azure.microsoft.com/downloads/
[DownloadCenterAddressRanges]: http://www.microsoft.com/download/details.aspx?id=41653
[NetworkSecurityGroups]: https://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[AzureAppService]: http://azure.microsoft.com/documentation/articles/app-service-value-prop-what-is/
[IntroToAppServiceEnvironment]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-intro/
[NewPortal]: https://portal.azure.com
 

<!-- IMAGES -->

<!---HONumber=Nov15_HO2-->