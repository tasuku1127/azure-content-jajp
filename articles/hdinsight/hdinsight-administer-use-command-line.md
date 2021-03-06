<properties
	pageTitle="Azure CLI を使用した Hadoop クラスターの管理 |Microsoft Azure"
	description="Azure CLI を使用して HDInsight で Hadoop クラスターを管理する方法"
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	authors="mumian"
	tags="azure-portal"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="11/03/2015"
	ms.author="jgao"/>

# Azure CLI を使用した HDInsight での Hadoop クラスターの管理

[AZURE.INCLUDE [セレクター](../../includes/hdinsight-portal-management-selector.md)]

[Azure コマンド ライン インターフェイス](xplat-cli-install.md)を使用して、Azure HDInsight で Hadoop クラスターを管理する方法について説明します。Azure CLI は Node.js で実装されます。Windows、Mac、Linux など、Node.js をサポートするいずれのプラットフォームでも使用できます。

Azure CLI はオープン ソースです。ソース コードは GitHub (<a href= "https://github.com/azure/azure-xplat-cli">https://github.com/azure/azure-xplat-cli</a>) で管理されています。

この記事では、HDInsight での Azure CLI の使用についてのみ説明します。Azure CLI の使用方法に関する一般的なガイドについては、「[Azure CLI のインストールと構成][azure-command-line-tools]」を参照してください。


##前提条件

この記事を読み始める前に、次の項目を用意する必要があります。

- **Azure サブスクリプション**。[Azure 無料試用版の取得](http://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)に関するページを参照してください。
- **Azure CLI** - インストールと構成に関する情報は、[Azure CLI のインストールと構成](../xplat-cli-install.md)に関するページを参照してください。
- 次のコマンドを使用して、**Azure に接続**します。

		azure login

	職場か学校のアカウントを使用した認証の詳細については、「[Azure CLI から Azure サブスクリプションへの接続する](xplat-cli-connect.md)」をご覧ください。
	
- 次のコマンドを使用して、**Azure リソース マネージャー モードに切り替えます**。

		azure config mode arm


##クラスターの作成

[AZURE.INCLUDE [provisioningnote](../../includes/hdinsight-provisioning.md)]

HDInsight クラスターを作成するには、Azure リソース マネージャー (ARM)、および Azure BLOB ストレージ アカウントが必要です。HDInsight クラスターを作成するには、以下を指定する必要があります。

- **Azure リソース グループ**: Data Lake Analytics アカウントは、Azure リソース グループ内で作成する必要があります。Azure リソース マネージャーを使用すると、アプリケーション内の複数リソースを 1 つのグループと見なして作業できます。アプリケーションのこれらすべてのリソースを、1 回の連携した操作でデプロイ、更新、または削除できます。 

	サブスクリプションのリソース グループをリストするには:
	
		azure group list 
	
	新しいリソース グループを作成するには:
	
		azure group create -n "<Resource Group Name>" -l "<Azure Location>"

- **HDInsight クラスター名**

- **場所**: HDInsight クラスターをサポートする Azure データ センターのいずれかです。サポートされている場所のリストについては、「[HDInsight の価格](https://azure.microsoft.com/pricing/details/hdinsight/)」を参照してください。

- **既定のストレージ アカウント**: HDInsight は、既定のファイル システムとして Azure BLOB ストレージ コンテナーを使用します。HDInsight クラスターを作成するには Azure ストレージ アカウントが必要です。

	新しい Azure ストレージ アカウントを作成するには:
	
		azure storage account create "<Azure Storage Account Name>" -g "<Resource Group Name>" -l "<Azure Location>" --type LRS

	> [AZURE.NOTE]ストレージ アカウントは、HDInsight と共にデータ センターに配置する必要があります。ZRS ではテーブルがサポートされないため、ストレージ アカウントの種類を ZRS にすることはできません。

	Azure プレビュー ポータルを使用した Azure ストレージ アカウントの作成については、[ストレージ アカウントの作成、管理、削除][azure-create-storageaccount]に関するページを参照してください。
	
	既にストレージ アカウントを持っていて、アカウント名とアカウント キーがわからない場合は、次のコマンドを使ってその情報を取得できます。
	
		-- Lists Storage accounts
		azure storage account list
		-- Shows a Storage account
		azure storage account show "<Storage Account Name>"
		-- Lists the keys for a Storage account
		azure storage account keys list "<Storage Account Name>" -g "<Resource Group Name>"

	Azure プレビュー ポータルを使用して情報を取得する方法の詳細については、[ストレージ アカウントの作成、管理、削除][azure-create-storageaccount]に関するページの「ストレージ アクセス キーの表示、コピーおよび再生成」セクションを参照してください。

- **(省略可能) 既定の BLOB コンテナー**: **azure hdinsight cluster create** コマンドは、コンテナーが存在しない場合、コンテナーを作成します。コンテナーを事前に作成する場合は、次のコマンドを使用できます。

	azure storage container create --account-name "<Storage Account Name>" --account-key <StorageAccountKey> [ContainerName]

ストレージ アカウントを用意して、BLOB コンテナーを準備したら、いつでもクラスターを作成できます。

	azure hdinsight cluster create --clusterName <ClusterName> --storageAccountName "<Storage Account Name>" --storageAccountKey <storageAccountKey> --storageContainer <StorageContainer> --nodes <NumberOfNodes> --location <DataCenterLocation> --username <HDInsightClusterUsername> --clusterPassword <HDInsightClusterPassword>

![HDI.CLIClusterCreation][image-cli-clustercreation]

















##構成ファイルを使用するクラスターの作成
通常は、HDInsight クラスターを作成し、そのクラスター上でジョブを実行してから、クラスターを削除してコストを削減します。コマンド ライン インターフェイスでは構成をファイルに保存することもできるため、クラスターを作成するたびにその構成を再利用できます。

	azure hdinsight cluster config create <file>

	azure hdinsight cluster config set <file> --clusterName <ClusterName> --nodes <NumberOfNodes> --location "<DataCenterLocation>" --storageAccountName ""<Storage Account Name>".blob.core.windows.net" --storageAccountKey "<StorageAccountKey>" --storageContainer "<BlobContainerName>" --username "<Username>" --clusterPassword "<UserPassword>"

	azure hdinsight cluster config storage add <file> --storageAccountName ""<Storage Account Name>".blob.core.windows.net"
	       --storageAccountKey "<StorageAccountKey>"

	azure hdinsight cluster config metastore set <file> --type "hive" --server "<SQLDatabaseName>.database.windows.net"
	       --database "<HiveDatabaseName>" --user "<Username>" --metastorePassword "<UserPassword>"

	azure hdinsight cluster config metastore set <file> --type "oozie" --server "<SQLDatabaseName>.database.windows.net"
	       --database "<OozieDatabaseName>" --user "<SQLUsername>" --metastorePassword "<SQLPassword>"

	azure hdinsight cluster create --config <file>



![HDI.CLIClusterCreationConfig][image-cli-clustercreation-config]


##クラスターの一覧と詳細の表示
クラスターの一覧と詳細を表示するには、次のコマンドを使用します。

	azure hdinsight cluster list
	azure hdinsight cluster show <ClusterName>

![HDI.CLIListCluster][image-cli-clusterlisting]


##クラスターの削除
クラスターを削除するには、次のコマンドを使用します。

	azure hdinsight cluster delete <ClusterName>

##クラスターのスケール

Azure PowerShell を使用して Hadoop クラスターのサイズを変更するには、クライアント コンピューターから次のコマンドを実行します。

	Set-AzureHDInsightClusterSize -ClusterSizeInNodes <NewSize> -name <clustername>

##次のステップ
この記事では、さまざまな HDInsight クラスター管理タスクを実行する方法について説明しました。詳細については、次の記事を参照してください。

* [Azure プレビュー ポータルを使用した HDInsight の管理][hdinsight-admin-portal]
* [Azure PowerShell を使用した HDInsight の管理][hdinsight-admin-powershell]
* [Azure HDInsight の概要][hdinsight-get-started]
* [Azure CLI の使い方][azure-command-line-tools]


[azure-command-line-tools]: ../xplat-cli.md
[azure-create-storageaccount]: ../storage-create-storage-account.md
[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/


[hdinsight-admin-portal]: hdinsight-administer-use-management-portal.md
[hdinsight-admin-powershell]: hdinsight-administer-use-powershell.md
[hdinsight-get-started]: ../hdinsight-get-started.md

[image-cli-account-download-import]: ./media/hdinsight-administer-use-command-line/HDI.CLIAccountDownloadImport.png
[image-cli-clustercreation]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreation.png
[image-cli-clustercreation-config]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreationConfig.png
[image-cli-clusterlisting]: ./media/hdinsight-administer-use-command-line/HDI.CLIListClusters.png "クラスターの一覧と表示"

<!---HONumber=Nov15_HO2-->