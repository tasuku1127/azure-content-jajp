<properties
   pageTitle="リソース マネージャーでサポートされるサービスとリージョン |Microsoft Azure"
   description="リソース マネージャーをサポートするリソース プロバイダーおよびリソースをホストできるリージョンについて説明します。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="10/19/2015"
   ms.author="tomfitz"/>

# リソース マネージャーによるサービス、リージョン、API バージョンのサポート

Azure リソース マネージャーは、アプリケーションを構成するサービスをデプロイして管理するための新しい方法を提供します。すべてではありませんがほとんどのサービスはリソース マネージャーをサポートし、一部のサービスはリソース マネージャーを部分的にのみサポートします。Microsoft は将来のソリューションに重要なすべてのサービスに対してリソース マネージャーを有効にする予定ですが、サポートが一貫したものになるまでは、各サービスの現在の状態を把握する必要があります。このトピックでは、Azure リソース マネージャーに対してサポートされるリソース プロバイダーの一覧を示します。

リソースをデプロイするときは、リソースをサポートしているリージョンとリソースで利用できる API バージョンも知っておく必要があります。「[サポートされているリージョン](#supported-regions)」セクションでは、サブスクリプションおよびリソースで有効なリージョンを確認する方法が示されています。「[サポートされる API バージョン](#supported-api-versions)」セクションでは、利用できる API バージョンを決定する方法について紹介します。

次の表では、各サービスがリソース マネージャーによるデプロイメントと管理をサポートするかどうかを示します。「**リソースの移動**」列は、この種類のリソースを新しいリソース グループおよび新しいサブスクリプションの両方に移動できるかどうかを示します。「**プレビュー ポータル**」列は、プレビュー ポータルを使用してサービスを作成できるかどうかを示します。


## Compute

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------------------------ | -------------- | -------------- |-------- | ------ |
| Virtual Machines | はい | はい | いいえ | [VM の作成](https://msdn.microsoft.com/library/azure/mt163591.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Compute.json) |
| Batch  
 | はい | いいえ | | [Batch REST](https://msdn.microsoft.com/library/azure/dn820158.aspx) | |
| Dynamics Lifecycle Services | はい | いいえ | | | |
| Virtual Machines (クラシック) | 限定的 | はい | 部分的 (後述参照) | - | - | | RemoteApp | いいえ | - | - | - | - | | Service Fabric | いいえ | - | - | - | - |

「Virtual Machines (クラシック)」とは、リソース マネージャー デプロイメント モデルではなくクラシック デプロイメント モデルを使用してデプロイされたリソースのことです。一般に、このようなリソースはリソース マネージャーの操作をサポートしていませんが、有効になっている操作がいくつかあります。これらのデプロイメント モデルの詳細については、「[リソース マネージャー デプロイと従来のデプロイを理解する](resource-manager-deployment-model.md)」を参照してください。

Virtual Machines (クラシック) リソースは、新しいリソース グループには移動できますが、新しいサブスクリプションには移動できません。

## Web とモバイル

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------- | -------- | -------------- | -------- | ------ |
| API Management| はい | いいえ | はい | [Create API](https://msdn.microsoft.com/library/azure/dn781423.aspx#CreateAPI) | |
| API Apps | はい | はい | | | [2015-03-01-preview](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-03-01-preview/Microsoft.AppService.json) |
| Web Apps | はい | はい | はい、制限あり (後述参照) | | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Web.json) |
| Notification Hubs | はい | はい | | [Notification Hub の作成](https://msdn.microsoft.com/library/azure/dn223269.aspx) | [2015-04-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-04-01/Microsoft.NotificationHubs.json) |
| Logic Apps | はい | はい | | | |
| Mobile Engagement | はい | いいえ | はい | | |

Web アプリを使用している場合、App Service プランのみを移動することはできません。Web アプリを移動するには、次のオプションがあります。

- 移動先のリソース グループにまだ Microsoft.Web リソースがない場合は、すべてのリソースをあるリソース グループから別のリソース グループに移動します。
- Web アプリは別のリソース グループに移動しますが、App Service プランは元のリソース グループで保持します。


## データ + ストレージ

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------- | ------- | -------------- | -------- | ------ |
| DocumentDB | はい | あり | はい | [DocumentDB REST](https://msdn.microsoft.com/library/azure/dn781481.aspx) | |
| Storage | はい | はい | | [Storage の作成](https://msdn.microsoft.com/library/azure/mt163564.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Storage.json) |
| Redis Cache | はい | あり | はい | | [2014-04-01-preview](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01-preview/Microsoft.Cache.json) |
| SQL Database | はい | あり | はい | [データベースの作成](https://msdn.microsoft.com/library/azure/mt163685.aspx) | [2014-04-01-preview](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01-preview/Microsoft.Sql.json) |
| 検索 | はい | あり | はい | [Search REST](https://msdn.microsoft.com/library/azure/dn798935.aspx) | |
| SQL Data Warehouse | はい | はい | | | |
| StorSimple | いいえ | いいえ | - | - | - | | Backup |いいえo | いいえ | - | - | - | | Site Recovery | いいえ | いいえ | - | - | - | | Managed cache | いいえ | いいえ | - | - | - | | Data Catalog | いいえ | いいえ | - | - | - |

## 分析

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------- | --------- | -------------- | -------- | ------ |
| イベント ハブ | はい | いいえ | | [Event Hub の作成](https://msdn.microsoft.com/library/azure/dn790676.aspx) | |
| Stream Analytics | はい | はい | | | |
| HDInsights | はい | はい | | | |
| Data Factory | はい | あり | はい | [Data Factory の作成](https://msdn.microsoft.com/library/azure/dn906717.aspx) | |
| Machine Learning | いいえ | いいえ | - | - | - |

## ネットワーク

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------- | -------- | -------------- | -------- | ------ |
| Application Gateway | はい | | | | |
| DNS | はい | | | [DNS ゾーンの作成](https://msdn.microsoft.com/library/azure/mt130622.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Network.json) |
| Load Balancer | はい | | | [Load Balancer の作成](https://msdn.microsoft.com/library/azure/mt163574.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Network.json) |
| Virtual Networks | はい | はい | いいえ | [Create Virtual Network](https://msdn.microsoft.com/library/azure/mt163661.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Network.json) |
| Traffic Manager | はい | いいえ | | [Traffic Manager プロファイルの作成](https://msdn.microsoft.com/library/azure/mt163581.aspx) | |
| ExpressRoute | あり | なし | いいえ | [ExpressRoute REST](https://msdn.microsoft.com/library/azure/mt586720.aspx) | |

## メディアと CDN

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------- | -------- | -------------- | -------- | ------ |
| メディア サービス | いいえ | いいえ | | | |
| CDN | いいえ | いいえ | | | |

## ハイブリッド統合

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------- | -------------- | -------------- | -------- | ------ |
| BizTalk Services | はい | いいえ | | | [2014-04-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01/Microsoft.BizTalkServices.json) |
| Service Bus | あり | いいえ | | [Service Bus REST](https://msdn.microsoft.com/library/azure/hh780717.aspx) | |

## ID 管理とアクセス管理 

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------- | -------------- | -------------- | -------- | ------ |
| Azure Active Directory | いいえ | いいえ | - | - | - | | Azure Actice Directory B2C | いいえ | いいえ | - | - | - | | Multi-Factor Authentication | いいえ | いいえ | - | - | - |

## 開発者サービス 

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------- | ---------- | -------------- | -------- | ------ |
| アプリケーション インサイト | はい | はい | | | [2014-04-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01/Microsoft.Insights.json) |
| Bing Maps | はい | はい | | | |
| Visual Studio アカウント | はい | | | | [2014-02-26](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-02-26/microsoft.visualstudio.json) |

## 管理 

| サービス | リソース マネージャーが有効 | プレビュー ポータル | リソースの移動 | REST API | スキーマ |
| ------- | ------- | --------- | -------------- | -------- | ------ |
| Automation | はい | はい | | | |
| Key Vault | はい | いいえ | はい | [Key Vault](https://msdn.microsoft.com/library/azure/dn903609.aspx) | |
| Scheduler | はい | いいえ | | | [2014-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-08-01/Microsoft.Scheduler.json) |
| Operational Insights | はい | いいえ | はい | | |
| IoTHubs | あり | あり | | | |


## サポートされているリージョン

通常、リソースをデプロイするときはリソースのリージョンを指定する必要があります。リソース マネージャーはすべてのリージョンでサポートされていますが、デプロイするリソースはすべてのリージョンではサポートされていない場合があります。さらに、サブスクリプションでの制限により、リソースをサポートする一部のリージョンを使用できない場合があります。これらの制限事項は、本国での税金に関する問題のため、またはサブスクリプション管理者によって設けられた、使用を特定のリージョンに制限するポリシーの結果である場合があります。

リソースをデプロイする前に、次のコマンドのいずれかを実行して、リソースの種類のサポート対象リージョンを確認してください。

### REST API

ご利用のサブスクリプションの特定のリソースの種類で使用可能なリージョンを確認するには、[すべてのリソース プロバイダーの一覧表示](https://msdn.microsoft.com/library/azure/dn790524.aspx)操作を利用します。

### PowerShell

次の例では、Azure PowerShell 1.0 Preview を使用して Web サイトのサポートされるリージョンを取得する方法を示します。1.0 Preview リリースの詳細については、「[Azure PowerShell 1.0 Preview](https://azure.microsoft.com/blog/azps-1-0-pre/)」を参照してください。

    PS C:\> ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations
    
次のように出力されます。

    Brazil South
    East Asia
    East US
    Japan East
    Japan West
    North Central US
    North Europe
    South Central US
    West Europe
    West US
    Southeast Asia
    Central US
    East US 2

### Azure CLI

次の例では、リソースの種類ごとにサポートされるすべての場所が返されます。

    azure location list

また、**jq** などのツールを使用して場所の結果をフィルターすることもできます。jq などのツールについて学習するには、「[Azure とやり取りする便利なツール](/virtual-machines/resource-group-deploy-debug/#useful-tools-to-interact-with-azure)」を参照してください。

    azure location list --json | jq '.[] | select(.name == "Microsoft.Web/sites")'

次のような結果が返されます。

    {
      "name": "Microsoft.Web/sites",
      "location": "Brazil South,East Asia,East US,Japan East,Japan West,North Central US,
            North Europe,South Central US,West Europe,West US,Southeast Asia,Central US,East US 2"
    }

## サポートされている API バージョン

テンプレートをデプロイするとき、各リソースの作成に使用する API バージョンを指定する必要があります。API バージョンは、リリース プロバイダーがリリースする REST API のバージョンに一致します。リソース プロバイダーは新しい機能を有効にするとき、REST API の新しいバージョンをリリースします。そのため、テンプレートで指定した API のバージョンにより、テンプレートで指定できるプロパティが変わります。一般的に、新しいテンプレートを作成するとき、最新の API バージョンを選択します。既存のテンプレートの場合、以前の API バージョンの使用を続けるか、テンプレートを最新版に更新して新しい機能を最大限に活用するか決定できます。

### REST API

リソースの種類で使用可能な API バージョンを確認するには、[すべてのリソース プロバイダーの一覧表示](https://msdn.microsoft.com/library/azure/dn790524.aspx)操作を利用します。

### PowerShell

次の例では、Azure PowerShell 1.0 プレビューを利用し、特定のリソースの種類で利用できる API バージョンを取得する方法を示します。

    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).ApiVersions
    
次のように出力されます。
    
    2015-08-01
    2015-07-01
    2015-06-01
    2015-05-01
    2015-04-01
    2015-02-01
    2014-11-01
    2014-06-01
    2014-04-01-preview
    2014-04-01

### Azure CLI

次のコマンドで、リソース プロバイダーの情報 (利用可能な API バージョンを含む) をファイルに保存できます。

    azure provider show Microsoft.Web -vv --json > c:\temp.json

ファイルを開き、**apiVersions** 要素を検索できます。

## 次のステップ

- リソース マネージャーのテンプレートの作成の詳細については、[Azure リソース マネージャーのテンプレートの作成](resource-group-authoring-templates.md)に関するページを参照してください。
- リソースをデプロイする方法を確認するには、「[Azure リソース マネージャーのテンプレートを使用したアプリケーションのデプロイ](./azure-portal/resource-group-template-deploy.md)」を参照してください。

<!---HONumber=Oct15_HO4-->