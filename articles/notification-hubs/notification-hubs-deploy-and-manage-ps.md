<properties 
	pageTitle="PowerShell を使用した Notification Hubs のデプロイと管理" 
	description="PowerShell を使用して Notification Hubs の作成と管理を自動化する方法" 
	services="notification-hubs" 
	documentationCenter="" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor="" />

<tags 
	ms.service="notification-hubs" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="powershell" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/24/2015" 
	ms.author="wesmc"/>

# PowerShell を使用した Notification Hubs のデプロイと管理

##概要

この記事では PowerShell を使用して Azure Notification Hubs を作成および管理する方法を説明します。このトピックでは、次の一般的なオートメーション タスクが表示されます。

+ 通知ハブの作成
+ 資格情報の設定

通知ハブに新しい Service Bus 名前空間を作成する必要がある場合は、「[PowerShell で Service Bus を管理する](../service-bus/service-bus-powershell-how-to-provision.md)」を参照してください。

Notification Hubs は Azure PowerShell に含まれているコマンドレットを使用して直接管理することはできません。PowerShell から最良のアプローチは、Microsoft.ServiceBus.dll アセンブリを参照することです。アセンブリは [Service Bus NuGet パッケージ](http://www.nuget.org/packages/WindowsAzure.ServiceBus/)に付属しています。


## 前提条件

この記事を読み始める前に、次の項目を用意する必要があります。

- Azure サブスクリプション。Azure はサブスクリプション方式のプラットフォームです。サブスクリプションの入手方法の詳細については、[購入オプション]、[メンバー プラン]、または[無料評価版]に関するページを参照してください。

- Azure PowerShell を搭載するコンピューター手順については、[Azure PowerShell のインストールおよび構成に関するページ]を参照してください。

- PowerShell スクリプト、NuGet パッケージ、.NET Framework の一般的理解。


## Service Bus 用の .NET アセンブリへの参照を含む

Azure Notification Hubs の管理はまだ Azure PowerShell の PowerShell コマンドレットに含まれていません。既存のコマンドレットを通じて公開されていない Notification Hubs や他の Service Bus のエンティティをプロビジョニングするには、[Service Bus NuGet パッケージ](http://www.nuget.org/packages/WindowsAzure.ServiceBus/)にある Service Bus 用の .NET クライアントを使用できます。

まず、このスクリプトが Visual Studio プロジェクトの NuGet パッケージでインストールされる **Microsoft.ServiceBus.dll** アセンブリを見つけることができるかどうかを確認します。柔軟性を持たせるために、スクリプトでは次のステップを実行します。

1. 呼び出されたパスを決定します。
2. `packages` という名前のフォルダーが見つかるまでパスを走査します。このフォルダーは Visual Studio プロジェクトで NuGet パッケージをインストールする際に作成されます。
3. **Microsoft.ServiceBus.dll** という名前のアセンブリの `packages` フォルダーを反復的に検索します。
4. アセンブリを参照するので、タイプは後で利用できるようになります。

次は、こうした手順を PowerShell スクリプトで実装する方法を示しています。

``` powershell

try
{
    # WARNING: Make sure to reference the latest version of Microsoft.ServiceBus.dll
    Write-Output "Adding the [Microsoft.ServiceBus.dll] assembly to the script..."
    $scriptPath = Split-Path (Get-Variable MyInvocation -Scope 0).Value.MyCommand.Path
    $packagesFolder = (Split-Path $scriptPath -Parent) + "\packages"
    $assembly = Get-ChildItem $packagesFolder -Include "Microsoft.ServiceBus.dll" -Recurse
    Add-Type -Path $assembly.FullName

    Write-Output "The [Microsoft.ServiceBus.dll] assembly has been successfully added to the script."
}

catch [System.Exception]
{
    Write-Error("Could not add the Microsoft.ServiceBus.dll assembly to the script. Make sure you build the solution before running the provisioning script.")
}
```

## NamespaceManager クラスの作成

Notification Hubs やその他の Service Bus のエンティティをプロビジョニングするには、SDK から [NamespaceManager](http://msdn.microsoft.com/library/microsoft.servicebus.namespacemanager.aspx) クラスのインスタンスを作成します。

Azure PowerShell に付属の [Get-AzureSBAuthorizationRule] コマンドレットを使用して、接続文字列の指定に使用する承認規則を取得できます。`NamespaceManager` インスタンスへの参照は `$NamespaceManager` 変数に保存されます。`$NamespaceManager` を使用して通知ハブをプロビジョニングします。

``` powershell
$sbr = Get-AzureSBAuthorizationRule -Namespace $Namespace
# Create the NamespaceManager object to create the event hub
Write-Output "Creating a NamespaceManager object for the [$Namespace] namespace..."
$NamespaceManager=[Microsoft.ServiceBus.NamespaceManager]::CreateFromConnectionString($sbr.ConnectionString);
Write-Output "NamespaceManager object for the [$Namespace] namespace has been successfully created."
```


## 新しい通知ハブのプロビジョニング 

新しい通知ハブをプロビジョニングするには、 [Service Bus の .NET API] を使用します。この記事は、Notification Hubs に関する内容のみを取り上げます。他の Service Bus エンティティでの作業については、「[PowerShell で Service Bus を管理する](../service-bus/service-bus-powershell-how-to-provision.md)」をご覧ください。

スクリプトのこの部分では 4 つのローカル変数を設定します。

1. `$Namespace` : 通知ハブを作成する名前空間の名前に設定します。
2. `$Path` : このパスは新しい通知ハブの名前に設定します。たとえば、「Myhub」のように設定します。    
3. `$WnsPackageSid` : [Windows デベロッパー センター](http://go.microsoft.com/fwlink/p/?linkid=266582&clcid=0x409)からの Windows アプリケーションのパッケージ SID に設定します。
4. `$WnsSecretkey` : [Windows デベロッパー センター](http://go.microsoft.com/fwlink/p/?linkid=266582&clcid=0x409)からの Windows アプリケーションの秘密キーに設定します。

これらの変数は、Service Bus 名前空間に接続し、Windows アプリの WNS 資格情報を使用して Windows 通知サービス (WNS) の通知を処理するように構成された新しい通知ハブを作成するために使用します。パッケージ SID と秘密キーを取得する方法については、「[Notification Hubs の使用](notification-hubs-windows-store-dotnet-get-started.md)」を参照してください。

+ スクリプト スニペットは `NamespaceManager` オブジェクトを使用して、`$Path` によって特定された通知ハブが存在するかどうかを確認します。

+ 存在しない場合は、WNS 資格情報を使用してスクリプトによって `NotificationHubDescription` が作成され、`NamespaceManager` クラスの `CreateNotificationHub` メソッドに渡されます。

``` powershell

$Namespace = "<Enter your namespace>
$Path  = "<Enter a name for your notification hub>"
$WnsPackageSid = "<your package sid>"
$WnsSecretkey = "<enter your secret key>"

$WnsCredential = New-Object -TypeName Microsoft.ServiceBus.Notifications.WnsCredential -ArgumentList $WnsPackageSid,$WnsSecretkey

# Query the namespace
$CurrentNamespace = Get-AzureSBNamespace -Name $Namespace

# Check if the namespace already exists
if ($CurrentNamespace)
{
    Write-Output "The namespace [$Namespace] in the [$($CurrentNamespace.Region)] region was found."

    # Create the NamespaceManager object used to create a new notification hub
    $sbr = Get-AzureSBAuthorizationRule -Namespace $Namespace
    Write-Output "Creating a NamespaceManager object for the [$Namespace] namespace..."
    $NamespaceManager = [Microsoft.ServiceBus.NamespaceManager]::CreateFromConnectionString($sbr.ConnectionString);
    Write-Output "NamespaceManager object for the [$Namespace] namespace has been successfully created."

    # Check to see if the Notification Hub already exists
    if ($NamespaceManager.NotificationHubExists($Path))
    {
        Write-Output "The [$Path] notification hub already exists in the [$Namespace] namespace."  
    }
    else
    {
        Write-Output "Creating the [$Path] notification hub in the [$Namespace] namespace."
        $NHDescription = New-Object -TypeName Microsoft.ServiceBus.Notifications.NotificationHubDescription -ArgumentList $Path;
        $NHDescription.WnsCredential = $WnsCredential;
        $NamespaceManager.CreateNotificationHub($NHDescription);
        Write-Output "The [$Path] notification hub was created in the [$Namespace] namespace."
    }
}
else
{
    Write-Host "The [$Namespace] namespace does not exist."
}
```




## その他のリソース

- [PowerShell で Service Bus を管理する](../service-bus/service-bus-powershell-how-to-provision.md)
- [PowerShell スクリプトを使用してService Bus キュー、トピック、サブスクリプションを作成する方法 (ブログの投稿)](http://blogs.msdn.com/b/paolos/archive/2014/12/02/how-to-create-a-service-bus-queues-topics-and-subscriptions-using-a-powershell-script.aspx)
- [PowerShell スクリプトを使用して Service Bus の名前空間と Event Hub を作成する方法 (ブログの投稿)](http://blogs.msdn.com/b/paolos/archive/2014/12/01/how-to-create-a-service-bus-namespace-and-an-event-hub-using-a-powershell-script.aspx)

既製のスクリプトもダウンロードできます。[Service Bus PowerShell Scripts](https://code.msdn.microsoft.com/windowsazure/Service-Bus-PowerShell-a46b7059)
 

[購入オプション]: http://azure.microsoft.com/pricing/purchase-options/
[メンバー プラン]: http://azure.microsoft.com/pricing/member-offers/
[無料評価版]: http://azure.microsoft.com/pricing/free-trial/
[Azure PowerShell のインストールおよび構成に関するページ]: ../install-configure-powershell.md
[Service Bus の .NET API]: https://msdn.microsoft.com/library/microsoft.servicebus.aspx
[Get-AzureSBNamespace]: https://msdn.microsoft.com/library/azure/dn495122.aspx
[New-AzureSBNamespace]: https://msdn.microsoft.com/library/azure/dn495165.aspx
[Get-AzureSBAuthorizationRule]: https://msdn.microsoft.com/library/azure/dn495113.aspx
 

<!---HONumber=Oct15_HO3-->