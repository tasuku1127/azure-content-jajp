<properties
	pageTitle="PowerShell を使用した HDInsight での Hadoop クラスターの管理 | Microsoft Azure"
	description="HDInsight の Hadoop クラスターを Azure PowerShell を使用して管理する方法について説明します。"
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	tags="azure-portal"
	authors="mumian"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="11/04/2015"
	ms.author="jgao"/>

# Azure PowerShell を使用した HDInsight での Hadoop クラスターの管理

[AZURE.INCLUDE [セレクター](../../includes/hdinsight-portal-management-selector.md)]

Azure PowerShell は、Azure のワークロードのデプロイと管理を制御し自動化するために使用できる強力なスクリプティング環境です。この記事では、ローカルの Azure PowerShell コンソールを使って Azure HDInsight で Hadoop クラスターを管理する方法を紹介します。例として、Windows PowerShell を使用します。HDInsight PowerShell コマンドレットの一覧については、「[HDInsight コマンドレット リファレンス][hdinsight-powershell-reference]」を参照してください。



**前提条件**

この記事を読み始める前に、次の項目を用意する必要があります。

- **Azure サブスクリプション**。[Azure 無料試用版の取得](http://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)に関するページを参照してください。
- **Azure PowerShell を実行できるワークステーション**。[Azure PowerShell のインストールおよび使用](http://azure.microsoft.com/documentation/videos/install-and-use-azure-powershell/)に関するページを参照してください。

	> [AZURE.NOTE]この記事で説明する PowerShell スクリプトは、Azure リソース マネージャー モードを使用します。サンプルを確実に動作させるには、Microsoft Web Platform Installer を使用して最新の Azure PowerShell をダウンロードしてください。

##クラスターの作成

HDInsight クラスターを使用するには、Azure ストレージ アカウントに Azure リソース グループと BLOB コンテナーが必要です。

- Azure リソース グループは、Azure リソース用の論理的なコンテナーです。Azure リソース グループと HDInsight クラスターを同じ場所にする必要はありません。詳細については、[リソース マネージャーでの Windows PowerShell の使用](powershell-azure-resource-manager.md)をご覧ください。
- HDInsight は、Azure ストレージ アカウントの BLOB コンテナーを既定のファイル システムとして使用します。HDInsight クラスターを作成するには Azure ストレージ アカウントとストレージ コンテナーが必要です。既定のストレージ アカウントと HDInsight クラスターは、同じ場所に存在する必要があります。

[AZURE.INCLUDE [provisioningnote](../../includes/hdinsight-provisioning.md)]

**Azure に接続するには**

	Login-AzureRmAccount
	Get-AzureRmSubscription  # list your subscriptions and get your subscription ID
	Select-AzureRmSubscription -SubscriptionId "<Your Azure Subscription ID>"

複数の Azure サブスクリプションがある場合は、**Select-AzureRMSubscription** が呼び出されます。
	
**新しいリソース グループを作成するには**

	New-AzureRmResourceGroup -name <New Azure Resource Group Name> -Location "<Azure Location>"  # For example, "EAST US 2"

**Azure ストレージ アカウントを作成するには**

	New-AzureRmStorageAccount -ResourceGroupName <Azure Resource Group Name> -Name <Azure Storage Account Name> -Location "<Azure Location>" -Type <AccountType> # account type example: Standard_LRS for zero redundancy storage
	
	Don't use **Standard_ZRS** because it deson't support Azure Table.  HDInsight uses Azure Table to logging. For a full list of the storage account types, see [https://msdn.microsoft.com/library/azure/hh264518.aspx](https://msdn.microsoft.com/library/azure/hh264518.aspx).

[AZURE.INCLUDE [データ センターの一覧](../../includes/hdinsight-pricing-data-centers-clusters.md)]


Azure プレビュー ポータルを使用した Azure ストレージ アカウントの作成については、「[Azure ストレージ アカウントについて](storage-create-storage-account.md)」を参照してください。

既にストレージ アカウントを持っていて、アカウント名とアカウント キーがわからない場合は、次のコマンドを使ってその情報を取得できます。

	# List Storage accounts for the current subscription
	Get-AzureRmStorageAccount
	# List the keys for a Storage account
	Get-AzureRmStorageAccountKey -ResourceGroupName <Azure Resource Group Name> -name $storageAccountName <Azure Storage Account Name>

プレビュー ポータルを使用して情報を取得する方法の詳細については、「[Azure ストレージ アカウントについて](storage-create-storage-account.md)」の「ストレージ アクセス キーの表示、コピーおよび再生成」セクションを参照してください。

**Azure Storage コンテナーを作成するには**

Azure PowerShell では、HDInsight の作成プロセス中に BLOB コンテナーを作成することはできません。コンテナーは次のスクリプトを使って作成します。

	$resourceGroupName = "<AzureResoureGroupName>"
	$storageAccountName = "<Azure Storage Account Name>"
	$storageAccountKey = Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccount |  %{ $_.Key1 }
	$containerName="<AzureBlobContainerName>"

	# Create a storage context object
	$destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

	# Create a Blob storage container
	New-AzureStorageContainer -Name $containerName -Context $destContext

**クラスターを作成するには**

ストレージ アカウントを用意して、BLOB コンテナーを準備したら、クラスターを作成する準備は整いました。

	$resourceGroupName = "<AzureResoureGroupName>"

	$storageAccountName = "<Azure Storage Account Name>"
	$containerName = "<AzureBlobContainerName>"

	$clusterName = "<HDInsightClusterName>"
	$location = "<AzureDataCenter>"
	$clusterNodes = <ClusterSizeInNodes>

	# Get the Storage account key
	$storageAccountKey = Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName | %{ $_.Key1 }

	# Create a new HDInsight cluster
	New-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName `
		-ClusterName $clusterName `
		-Location $location `
		-DefaultStorageAccountName "$storageAccountName.blob.core.windows.net" `
		-DefaultStorageAccountKey $storageAccountKey `
		-DefaultStorageContainer $containerName  `
		-ClusterSizeInNodes $clusterNodes

##クラスターの一覧表示
現在のサブスクリプションにあるクラスターすべてを一覧表示するには次のコマンドを使用します。

	Get-AzureRmHDInsightCluster

##クラスターの表示

現在のサブスクリプションにある特定のクラスターの詳細を表示するには次のコマンドを使用します。

	Get-AzureRmHDInsightCluster -ClusterName <Cluster Name>

##クラスターの削除
クラスターを削除するには、次のコマンドを使用します。

	Remove-AzureRmHDInsightCluster -ClusterName <Cluster Name>

##クラスターのスケール
クラスターのスケール設定機能を使用すると、Azure HDInsight で実行しているクラスターによって使用される worker ノードの数を、クラスターを再作成することなく、変更できます。

>[AZURE.NOTE]HDInsight バージョン 3.1.3 以降を使用しているクラスターのみがサポートされます。クラスターのバージョンがわからない場合、[プロパティ] ページを確認できます。「[クラスター ポータル インターフェイスについての理解する](hdinsight-adminster-use-management-portal/#Get-familiar-with-the-cluster-portal-interface)」を参照してください。

HDInsight でサポートされているクラスターの種類ごとに、データ ノード数を変更した場合の影響:

- Hadoop は、

	保留中または実行中のジョブに影響を与えることなく、実行中の Hadoop クラスター内の worker ノードの数をシームレスに増加できます。処理の進行中に新しいジョブを送信することもできます。スケール設定処理の失敗は正常に処理され、クラスターは常に機能状態になります。

	データ ノードの数を減らして Hadoop クラスターのスケールを小さくした場合、クラスター内の一部のサービスが再起動されます。これにより、スケール設定処理の完了時に、実行中および保留中のすべてのジョブが失敗します。ただし、処理が完了した後にジョブを再送信できます。

- HBase

	実行中の HBase クラスターに対して、ノードの追加または削除をシームレスに実行できます。地域サーバーは、スケール設定処理の完了の数分以内に自動的に分散されます。ただし、クラスターのヘッドノードにログインし、コマンド プロンプト ウィンドウから次のコマンドを実行して、地域サーバーを手動で分散することもできます。

		>pushd %HBASE_HOME%\bin
		>hbase shell
		>balancer

	HBase シェルの使用方法の詳細については、以下を参照してください。
- Storm

	実行中の Storm クラスターに対して、データ ノードの追加または削除をシームレスに実行できます。ただし、スケール設定処理が正常に完了した後、トポロジのバランス再調整が必要になります。

	バランス再調整は、次の 2 つの方法で実行できます。

	* Storm Web UI
	* コマンド ライン インターフェイス (CLI) ツール

	詳細については、[Apache Storm に関するドキュメント](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html)をご覧ください。

	Storm Web UI は、HDInsight クラスターで使用できます。

	![HDInsight Storm の規模のバランス調整](./media/hdinsight-administer-use-management-portal/hdinsight.portal.scale.cluster.storm.rebalance.png)

	CLI コマンドを使用して Storm トポロジのバランスを再調整する方法を次の例で示します。

		## Reconfigure the topology "mytopology" to use 5 worker processes,
		## the spout "blue-spout" to use 3 executors, and
		## the bolt "yellow-bolt" to use 10 executors

		$ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10

Azure PowerShell を使用して Hadoop クラスターのサイズを変更するには、クライアント コンピューターから次のコマンドを実行します。

	Set-AzureRmHDInsightClusterSize -ClusterName <Cluster Name> -TargetInstanceCount <NewSize>
	

##アクセス権の付与/取り消し

HDInsight クラスターには、以下の HTTP Web サービスがあります (これらすべてのサービスには、REST ベースのエンドポイントがあります)。

- ODBC
- JDBC
- Ambari
- Oozie
- Templeton


既定では、これらのサービスへのアクセス許可が付与されます。アクセス許可を取り消す/付与することができます。取り消するには、次を実行します。

	Revoke-AzureRmHDInsightHttpServicesAccess -ClusterName <Cluster Name>

許可するには、次を実行します。

	$clusterName = "<HDInsight Cluster Name>"

	# Credential option 1
	$hadoopUserName = "admin"
	$hadoopUserPassword = "Pass@word123"
	$hadoopUserPW = ConvertTo-SecureString -String $hadoopUserPassword -AsPlainText -Force
	$credential = New-Object System.Management.Automation.PSCredential($hadoopUserName,$hadoopUserPW)

	# Credential option 2
	#$credential = Get-Credential -Message "Enter the HTTP username and password:" -UserName "admin"
	
	Grant-AzureRmHDInsightHttpServicesAccess -ClusterName $clusterName -HttpCredential $credential

>[AZURE.NOTE]アクセス許可を付与するか、取り消すことで、クラスターのユーザー名とパスワードがリセットされます。

これは、プレビュー ポータルを使用して行うこともできます。[Azure プレビュー ポータルを使用した HDInsight の管理][hdinsight-admin-portal]に関するページを参照してください。

##HTTP ユーザーの資格情報の更新

[HTTP アクセス許可の付与/取り消し](#grant/revoke-access)と同じ手順です。HTTP アクセスがクラスターで許可されている場合、最初に取り消す必要があります。次に、新しい HTTP ユーザーの資格情報を使用してアクセス許可を付与してください。


##既定のストレージ アカウントの検索

次の Powershell スクリプトでは、クラスターの既定のストレージ アカウント名と既定のストレージ アカウント キーを取得する方法を示します。

	$clusterName = "<HDInsight Cluster Name>"
	
	$cluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
	$resourceGroupName = $cluster.ResourceGroup
	$defaultStorageAccountName = ($cluster.DefaultStorageAccount).Replace(".blob.core.windows.net", "")
	$defaultBlobContainerName = $cluster.DefaultStorageContainer
	$defaultStorageAccountKey = Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName |  %{ $_.Key1 }
	$defaultStorageAccountContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey 

##リソース グループの検索

ARM モードでは、各 HDInsight クラスターは Azure リソース グループに属しています。リソース グループを検索するには

	$clusterName = "<HDInsight Cluster Name>"
	
	$cluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
	$resourceGroupName = $cluster.ResourceGroup


##ジョブの送信

**MapReduce ジョブを送信するには**

「[Windows ベースの HDInsight での Hadoop MapReduce サンプルの実行](hdinsight-run-samples.md)」を参照してください。

**Hive ジョブを送信するには**

「[PowerShell を使用して Hive クエリを実行](hdinsight-hadoop-use-hive-powershell.md)」を参照してください。

**Pig ジョブを送信するには**

「[PowerShell を使用した Pig ジョブの実行](hdinsight-hadoop-use-pig-powershell.md)」を参照してください。

**Sqoop ジョブを送信するには**

「[Hadoop .NET SDK と HDInsight の使用](hdinsight-use-sqoop.md)」を参照してください。

**Oozie ジョブを送信するには**

「[HDInsight での Oozie と Hadoop を使用したワークフローの定義と実行](hdinsight-use-oozie.md)」を参照してください。

##Azure BLOB ストレージにデータをアップロードする
「[データを HDInsight にアップロードする方法][hdinsight-upload-data]」を参照してください。


## 関連項目
* [HDInsight コマンドレット リファレンス ドキュメント][hdinsight-powershell-reference]
* [Azure プレビュー ポータルを使用した HDInsight の管理][hdinsight-admin-portal]
* [コマンド ライン インターフェイスを使用した HDInsight の管理][hdinsight-admin-cli]
* [HDInsight クラスターの作成][hdinsight-provision]
* [HDInsight へのデータのアップロード][hdinsight-upload-data]
* [プログラムによる Hadoop ジョブの送信][hdinsight-submit-jobs]
* [Azure HDInsight の概要][hdinsight-get-started]


[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[hdinsight-get-started]: ../hdinsight-get-started.md
[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-provision-custom-options]: hdinsight-provision-clusters.md#configuration
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md

[hdinsight-admin-cli]: hdinsight-administer-use-command-line.md
[hdinsight-admin-portal]: hdinsight-administer-use-management-portal.md
[hdinsight-storage]: ../hdinsight-use-blob-storage.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-mapreduce]: hdinsight-use-mapreduce.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-flight]: hdinsight-analyze-flight-delay-data.md

[hdinsight-powershell-reference]: https://msdn.microsoft.com/library/dn858087.aspx

[powershell-install-configure]: ../install-configure-powershell.md

[image-hdi-ps-provision]: ./media/hdinsight-administer-use-powershell/HDI.PS.Provision.png

<!---HONumber=Nov15_HO3-->