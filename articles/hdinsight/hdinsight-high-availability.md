<properties 
	pageTitle="HDInsight での Hadoop クラスターの可用性 | Microsoft Azure" 
	description="HDInsight は追加のヘッド ノードを使用して可用性と信頼性の高いクラスターをデプロイします。" 
	services="hdinsight" 
	editor="cgronlun" 
	manager="paulettm" 
	authors="bradsev" 
	documentationCenter=""/>

<tags 
	ms.service="hdinsight" 
	ms.workload="big-data" 
	ms.tgt_pltfrm="na" 
	ms.devlang="multiple" 
	ms.topic="article" 
	ms.date="05/19/2014" 
	ms.author="bradsev"/>


#HDInsight における Hadoop クラスターの可用性と信頼性


ワークロードの管理に必要なサービスの可用性と信頼性を高めるために、Azure HDInsight によってデプロイされた Hadoop クラスターに 2 つ目のヘッド ノードが追加されました。通常 Hadoop クラスターの標準的な実装には、ヘッド ノードは 1 つしかありません。これらのクラスターは、ワーカー ノードの障害を円滑に管理するよう設計されていますが、ヘッド ノードで実行中のマスター サービスが停止すると、クラスターが動作を停止する原因になることがあります。

![HDInsight の Hadoop 実装における信頼性の高いヘッド ノード図。](http://i.imgur.com/jrUmrH4.png)

HDInsight はセカンダリ ヘッド ノード (ヘッド ノード 1) を追加することで、この単一障害点をなくします。[ZooKeeper](http://zookeeper.apache.org/) ノード (ZK) を追加して、ヘッド ノードのリーダー選定に使用すると、ワーカー ノードとゲートウェイ (GW) は、アクティブ ヘッド ノード (ヘッド ノード 0) が非アクティブになったときに 2 つ目のヘッド ノード (ヘッド ノード 1) にフェールオーバーするタイミングを認識できます。


## アクティブ ヘッド ノードのサービスの状態を確認する方法 ##
アクティブ ヘッド ノードを特定し、そのヘッド ノードで実行中のサービスの状態をチェックするには、リモート デスクトップ プロトコル (RDP) を使用して Hadoop クラスターに接続する必要があります。クラスターにリモート接続する機能は、Azure では既定で無効になっているため、最初に有効にする必要があります。ポータルでのこの操作の方法に関する詳細については、「[RDP を使用した HDInsight クラスターへの接続](hdinsight-administer-use-management-portal.md#rdp)」をご覧ください。クラスターにリモート接続したら、デスクトップにある **[Hadoop サービスの可用性の状態]** アイコンをダブルクリックして、Namenode、Jobtracker、Templeton、Oozieservice、Metastore、Hiveserver2 の各サービスが実行されているヘッド ノードについて、状態を取得します。HDI 3.0 の場合、対象のサービスは、Namenode、Resource Manager、History Server、Templeton、Oozieservice、Metastore、Hiveserver2 になります。

![](http://i.imgur.com/MYTkCHW.png)


## セカンダリ ヘッド ノードのログ ファイルへのアクセス方法 \

セカンダリ ヘッド ノードがアクティブ ヘッド ノードになった場合、セカンダリ ヘッド ノードのログ ファイルにアクセスできるよう、引き続きプライマリ アクティブ ノードと同様に JobTracker UI を参照できます。Job Tracker にアクセスするには、前のセクションで説明したように RDP を使用して Hadoop クラスターに接続する必要があります。クラスターにリモート接続したら、デスクトップにある **[Hadoop 名前ノード]** アイコンをダブルクリックし、**[NameNode ログ]** をクリックしてセカンダリ ヘッド ノード上のログのディレクトリを参照します。

![](http://i.imgur.com/eL6jzgB.png)


## ヘッド ノードのサイズの構成方法 ##
既定では、ヘッド ノードは L サイズの仮想マシン (VM) として割り当てられます。このサイズは、クラスター上で実行されるほとんどの Hadoop ジョブを管理するには十分です。ただし、XL サイズの VM がヘッド ノードに必要となるシナリオもあります。その例の 1 つに、クラスターで小さな Oozie ジョブを多数管理しなければならない場合があります。

XL サイズの VM は、Azure PowerShell コマンドレットまたは HDInsight SDK を使用して構成できます。

Azure PowerShell を使用したクラスターの作成とプロビジョニングについては、「[PowerShell を使用した HDInsight の管理](hdinsight-administer-use-powershell.md)」で説明しています。XL サイズのヘッド ノードの構成には、このコードで使用される `New-AzureHDInsightcluster` コマンドレットに `-HeadNodeVMSize ExtraLarge` パラメーターを追加する必要があります。

    # Create a new HDInsight cluster in Azure PowerShell
	# Configured with an ExtraLarge head-node VM
    New-AzureHDInsightCluster -Name $clusterName -Location $location -HeadNodeVMSize ExtraLarge -DefaultStorageAccountName "$storageAccountName.blob.core.windows.net" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainerName $containerName  -ClusterSizeInNodes $clusterNodes

SDK でも、同じような方法を使用します。SDK を使用したクラスターの作成とプロビジョニングについては、「[HDInsight .NET SDK の使用](hdinsight-provision-clusters.md#sdk)」で説明しています。XL サイズのヘッド ノードの構成には、このコードで使用される `ClusterCreateParameters()` メソッドに `HeadNodeSize = NodeVMSize.ExtraLarge` パラメーターを追加する必要があります。

    # Create a new HDInsight cluster with the HDInsight SDK
	# Configured with an ExtraLarge head-node VM
    ClusterCreateParameters clusterInfo = new ClusterCreateParameters()
    {
    Name = clustername,
    Location = location,
    HeadNodeSize = NodeVMSize.ExtraLarge,
    DefaultStorageAccountName = storageaccountname,
    DefaultStorageAccountKey = storageaccountkey,
    DefaultStorageContainer = containername,
    UserName = username,
    Password = password,
    ClusterSizeInNodes = clustersize
    };


**参照**

- [ZooKeeper](http://zookeeper.apache.org/)
- [RDP を使用した HDInsight クラスターへの接続](hdinsight-administer-use-management-portal.md#rdp)
- [HDInsight .NET SDK の使用](hdinsight-provision-clusters.md#sdk) 







 

<!---HONumber=62-->