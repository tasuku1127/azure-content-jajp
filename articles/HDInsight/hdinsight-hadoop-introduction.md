<properties 
	pageTitle="HDInsight での Hadoop とは: クラウドのビッグ データ分析 | Microsoft Azure" 
	description="HDInsight のクラウドでの Hadoop コンポーネントの紹介です。HDInsight が Hadoop クラスターを使用してビッグ データを管理、分析、レポート生成する方法について説明します。" 
	keywords="big data,big data analysis,hadoop,introduction to hadoop,what is hadoop"
	services="hdinsight" 
	documentationCenter="" 
	authors="cjgronlund" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="02/18/2015"
   ms.author="cgronlun"/>


# HDInsight での Hadoop 入門: クラウドでのビッグ データの分析と処理

Azure HDInsight の Hadoop、そのエコシステム、Azure HDInsight のビッグ データについて、HDInsight の Hadoop、Hadoop のコンポーネント、一般的な用語、ビッグ データ分析のためのシナリオを説明します。また、HDInsight の Hadoop を使用するためのドキュメント、チュートリアル、リソースについても説明します。

## HDInsight での Hadoop とは

Azure HDInsight は、Apache Hadoop クラスターをクラウドにデプロイしてプロビジョニングするためのサービスです。ビッグ データの管理、分析、レポート生成を支援するソフトウェア フレームワークとなります。Hadoop コアには、データ ストレージの信頼性を高める Hadoop 分散ファイル システム (HDFS) と、この分散システムに格納されているデータの処理と分析を同時に行うシンプルな MapReduce プログラミング モデルが用意されています。


## ビッグ データとは
"ビッグ データ" とは、絶えず急速なペースで増加し、さまざまな非構造化形式と意味的文脈から成る巨大なデータ群を指しています。

ビッグ データは、Twitter フィードのテキストから、工業機器のセンサー情報、オンライン カタログでの消費者の閲覧、購入行動に関する情報に至るまでの、大規模なデジタル情報を表します。ビッグ データの種類には、履歴データ (つまり、保存されたデータ) とリアルタイム データ (つまり、ソースから直接ストリーミングされるデータ) があります。

ビッグ データから実用的な情報や洞察を導くうえで必要なのは、適切な質問をして問題に関連するデータを収集することだけではありません。データは、アクセス可能であり、クリーンアップと分析を行った後実用的な方法で定義される必要があります。そこで、HDInsight の Hadoop のビッグ データ分析の出番となります。


## <a name="overview"></a>HDInsight の Hadoop エコシステムの概要

HDInsight は、急激に拡大を続けている Apache Hadoop テクノロジ スタックの Microsoft Azure 上のクラウド実装であり、ビッグ データ分析用の信頼できるソリューションです。これには、Storm、HBase、Pig、Hive、Sqoop、Oozie、Ambari などの実装が含まれます。また、HDInsight は、Excel、SQL Server Analysis Services、SQL Server Reporting Services などのビジネス インテリジェンス (BI) ツールとも統合されます。


* Azure HDInsight は、**Linux** または **Windows** を基盤 OS として使用し、Hadoop クラスターをクラウドにデプロイしてプロビジョニングします。

	* **Linux での HDInsight (プレビュー)** - Ubuntu 上の Hadoop クラスター。Linux または Unix に詳しい場合、または Linux 向けに構築された Hadoop エコシステム コンポーネントとの簡単な統合が必要な場合は、既存の Linux ベースの Hadoop ソリューションから移行することで、これを使用します。

	* **Windows での HDInsight** - Windows Server 上の Hadoop クラスター。Windows に詳しい場合、または .NET や他の Windows 機能との統合が必要な場合は、既存の Windows ベースの Hadoop ソリューションから移行することで、これを使用します。

	次の表では、両者の比較を示します。

	カテゴリ | Linux での HDInsight | Windows での HDInsight 
	---------| -------------------| --------------------
	**クラスターの OS** | Ubuntu 12.04 Long Term Support (LTS) | Windows Server 2012 R2
	**クラスターの種類** | Hadoop は、 | Hadoop、HBase、Storm
	**デプロイメント** | Azure 管理ポータル、クロスプラット フォーム コマンド ライン、Azure PowerShell | Azure 管理ポータル、クロスプラット フォーム コマンド ライン、Azure PowerShell、HDInsight .NET SDK
	**クラスター UI** | Ambari | クラスター ダッシュボード
	**リモート アクセス** | Secure Shell (SSH) | リモート デスクトップ プロトコル (RDP)
	

* HDInsight は、**Hortonworks Data Platform (HDP)** Hadoop ディストリビューションを使用します。

* Apache Hadoop は、ビッグ データの管理と分析に役立つソフトウェア フレームワークです。HDInsight は特定のワークロードにいくつかの構成を提供しますが、ユーザーは <a href="http://azure.microsoft.com/documentation/articles/hdinsight-hadoop-customize-cluster/" target="_blank">Script Action を使ってクラスターをカスタマイズ</a>できます。

	* **Hadoop**: データ ストレージの信頼性を高める [HDFS](#HDFS) と、処理と分析を同時に行うシンプルな [MapReduce](#mapreduce) プログラミング モデルが用意されています。
	
	* **<a target="_blank" href="http://hbase.apache.org/">HBase</a>**: Hadoop 上に構築された NoSQL データベースです。数十億行 x 数百万列という規模の巨大な非構造化データと半構造化データに対するランダム アクセスと強力な一貫性を実現します。「[HDInsight HBase の概要](hdinsight-hbase-overview.md)」を参照してください。
	
	* **<a  target="_blank" href="https://storm.incubator.apache.org/">Apache Storm</a>** は、大量のデータ ストリームを高速に処理するための分散型リアルタイム計算システムです。Storm は、HDInsight で管理されるクラスターとして提供されます。「[HDInsight (Hadoop) での Storm と HBase を使ったセンサー データの分析](hdinsight-storm-sensor-data-analysis.md)」を参照してください。

## Hadoop コンポーネントとは

以前の構成全体に加え、HDInsight クラスターには次の各コンポーネントも含まれています。

* **[Ambari](#ambari)** - クラスターのプロビジョニング、管理、監視。 

* **[Avro](#avro)** (Microsoft .NET Library for Avro) - Microsoft.NET 環境向けのデータ シリアル化。

* **[Hive](#hive)** - クエリに似た構造化照会言語 (SQL)。

* **[Mahout](#mahout)** - 機械学習。

* **[MapReduce and YARN](#mapreduce)** - 分散処理とリソース管理。

* **[Oozie](#oozie)** - ワークフロー管理。

* **[Pig](#pig)** - MapReduce 変換のためのシンプルなスクリプト作成。

* **[Sqoop](#sqoop)** - データのインポートとエクスポート。

* **[ZooKeeper](#zookeeper)** - 分散システムにおけるプロセスの調整。

> [AZURE.NOTE]特定のコンポーネントとバージョン情報については、「[HDInsight で提供される Hadoop クラスター バージョンの新機能][component-versioning]」をご覧ください。

### <a name="ambari"></a>Ambari

Apache Ambari は、Apache Hadoop クラスターのプロビジョニング、管理、および監視を目的としています。演算子ツールの直観的なコレクションおよび Hadoop の複雑さが見えないようにする信頼性の高い、一連の API が含まれており、クラスターの操作を単純化しています。「[Ambari を使用した HDInsight クラスターの管理](hdinsight-hadoop-manage-ambari.md)」、「[Ambari API を使用した HDInsight の Hadoop クラスターの監視](hdinsight-monitor-use-ambari-api.md)」、「<a target="_blank" href="https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md">Apache Ambari API リファレンス</a>」をご覧ください。

### <a name="avro"></a>Avro (Microsoft .NET Library for Avro)

Microsoft .NET Library for Avro は、Microsoft.NET 環境向けのシリアル化のために、Apache Avro のコンパクトなバイナリ データ交換形式を実装します。Avro は、<a target="_blank" href="http://www.json.org/">JavaScript Object Notation (JSON)</a> を使用して、言語の相互運用性を保証する、言語に依存しないスキーマを定義します。これは、ある言語でシリアル化されたデータを別の言語で読むことができることを意味します。この形式の詳細については、「<a target=_"blank" href="http://avro.apache.org/docs/current/spec.html">Apache Avro の仕様</a>」をご覧ください。Avro ファイルの形式は、分散 MapReduce プログラミング モデルをサポートしています。ファイル内の任意のポイントを検索し、特定のブロックから読み取りを開始できるという意味で、ファイルは "分割可能" です。詳細については、「[Microsoft .NET Library for Avro を使用したデータのシリアル化](hdinsight-dotnet-avro-serialization.md)」を参照してください。


### <a name="hdfs"></a>HDFS

Hadoop 分散ファイル システム (HDFS) は、MapReduce と YARN と共に Hadoop エコシステムの中核を構成する分散ファイル システムです。HDFS は、HDInsight の Hadoop クラスターの標準ファイル システムです。

### <a name="hive"></a>Hive

<a target="_blank" href="http://hive.apache.org/">Apache Hive</a> は、Hadoop に組み込まれているデータ ウェアハウス ソフトウェアで、HiveQL と呼ばれる SQL に似た言語を使って分散ストレージ内の大量のデータセットに対するクエリと管理を可能にします。Pig と同様、Hive は MapReduce 上の抽象コンポーネントであり、実行時にクエリを一連の MapReduce ジョブに変換します。Hive は、概念的には Pig よりもリレーショナル データベース管理システムに近いため、より構造化されたデータでの使用に適しています。非構造化データを操作する場合は Pig が適しています。「[HDInsight での Hive と Hadoop の使用](hdinsight-use-hive.md)」をご覧ください。

### <a name="mahout"></a>Mahout

<a target="_blank" href="https://mahout.apache.org/">Apache Mahout</a> は、Hadoop 上で実行される機械学習アルゴリズムのスケーラブルなライブラリです。統計の原則を使用して、機械学習アプリケーションは、データから学習すること、および過去の結果から将来の動作を決めることをシステムに教えます。「[HDInsight (Hadoop) で Apache Mahout を使用して映画のリコメンデーションを生成する](hdinsight-mahout.md)」を参照してください。

### <a name="mapreduce"></a>MapReduce と YARN
Hadoop MapReduce は、大規模なデータセットを並列処理するアプリケーションを記述するためのソフトウェア フレームワークです。MapReduce ジョブは、大規模なデータセットを分割し、データを処理のためにキーと値のペアに整理します。

Apache YARN は、次世代の MapReduce (MapReduce 2.0、または MRv2) で、リソース管理とジョブ スケジューリング/監視という JobTracker の 2 つの主要なタスクを別個のエンティティに分けたものです。

MapReduce の詳細については、Hadoop Wiki で「<a target="_blank" href="http://wiki.apache.org/hadoop/MapReduce">MapReduce</a>」をご覧ください。YARN の詳細については、「<a target="_blank" href="http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html">Apache Hadoop NextGen MapReduce (YARN) (Apache Hadoop の次世代の MapReduce (YARN))</a>」をご覧ください。

### <a name="oozie"></a>Oozie
<a target="_blank" href="http://oozie.apache.org/">Apache Oozie</a> は Hadoop ジョブを管理するワークフローと調整システムです。Hadoop スタックと統合されていて、MapReduce、Pig、Hive、Sqoop の Hadoop ジョブをサポートしています。Java プログラムやシェル スクリプトのような、システム特有のジョブのスケジュールを設定するのに使用することもできます。「[HDInsight の Hadoop での時間ベースの Oozie コーディネーターの使用](hdinsight-use-oozie-coordinator-time.md)」を参照してください。

### <a name="pig"></a>Pig

<a  target="_blank" href="http://pig.apache.org/">Apache Pig</a> は、Pig Latin と呼ばれるシンプルなスクリプト言語を使って大規模なデータセットに対して複雑な MapReduce 変換を実行することを可能にする高レベル プラットフォームです。Pig は、Hadoop 内で実行できるように Pig Latin スクリプトを変換します。ユーザー定義関数 (UDF) を作成して Pig Latin を拡張できます。「[HDInsight の Hadoop での Pig の使用](hdinsight-use-pig.md)」を参照してください。

### <a name="sqoop"></a>Sqoop
<a  target="_blank" href="http://sqoop.apache.org/">Apache Sqoop</a> は、リレーショナル データベース (SQL など) やその他の構造化データ ストアと Hadoop との間でデータを効率的に一括転送するためのツールです。「[HDInsight の Hadoop での Sqoop の使用](hdinsight-use-sqoop.md)」を参照してください。


### <a name="zookeeper"></a>ZooKeeper
<a  target="_blank" href="http://zookeeper.apache.org/">Apache ZooKeeper</a> は、データ レジスタ (znode) の共有階層名前空間を使用して、大規模な分散システムのプロセスを調整します。znode には、状態、場所、構成など、プロセスを調整するのに必要な少量のメタ情報が含まれます。

## <a name="advantage"></a>クラウドで Hadoop を使用するメリット

HDInsight の Hadoop は、Azure クラウド エコシステムの一部としていくつかのメリットを提供します。その一部を次に示します。

* Hadoop クラスターの自動プロビジョニング。HDInsight クラスターは、手動で Hadoop クラスターを構成するよりも作成がはるかに簡単です。詳細については、「[HDInsight での Hadoop クラスターのプロビジョニング](hdinsight-provision-clusters.md)」に関するページをご覧ください。

* 最新の Hadoop コンポーネント。詳細については、
* 「[HDInsight で提供される Hadoop クラスター バージョンの新機能]」をご覧ください。

* クラスターの高い可用性と信頼性。「[HDInsight における Hadoop クラスターの可用性と信頼性](hdinsight-high-availability.md)」を参照してください。

* Hadoop 互換オプションの Azure BLOB ストレージによって実現される効率的で経済的なデータ ストレージ。詳細については、「[HDInsight の Hadoop での Azure BLOB ストレージの使用](../hdinsight-use-blob-storage.md)」を参照してください。

* [Websites](../documentation/services/websites/)、[SQL Database](../documentation/services/sql-database/) などの他の Azure サービスとの統合。

* 低い導入コスト。[無料評価版](/pricing/free-trial/)から始めることができます。「[HDInsight の料金詳細](/pricing/details/hdinsight/)」も参照してください。


HDInsight の Hadoop を使うその他のメリットについては、[HDInsight 向けの Azure の機能のページ][marketing-page]を参照してください。



## <a id="resources"></a>ビッグ データ分析、Hadoop、HDInsight に関する情報の収集に役立つリソース

この HDInsight の Hadoop とビッグ データ分析への入門は、次のリソースを使用して作成されました。


### Linux での HDInsight (プレビュー)

* [Linux での HDInsight の概要](../hdinsight-hadoop-linux-get-started.md) - Linux と実行中のサンプルの Hive クエリで HDInsight Hadoop クラスターをプロビジョニングするためのクイック スタート チュートリアル。

* [カスタム オプションを使用した Linux での HDInsight のプロビジョニング](hdinsight-hadoop-provision-linux-clusters.md) - Azure 管理ポータル、Azure クロス プラットフォーム コマンド ライン、Azure PowerShell のカスタム オプションを使用して、Linux での HDInsight Hadoop クラスターのプロビジョニング方法について説明します。

* [Linux での HDInsight の使用](hdinsight-hadoop-linux-information.md) - Azure でプロビジョニングされた Hadoop Linux クラスターの使用に関するクイック ヒントを示します。

* [Ambari を使用した HDInsight クラスターの管理](hdinsight-hadoop-manage-ambari.md) - Ambari Web または Ambari REST API を使用して、HDInsight クラスターで Linux ベースの Hadoop を監視し、管理する方法を説明します。


### Windows での HDInsight

* [HDInsight ドキュメント](http://azure.microsoft.com/documentation/services/hdinsight/) - Azure HDInsight のドキュメント ページ。記事やビデオなどのリソースに対するリンクが記載されています。

* [HDInsight の学習マップ](hdinsight-learn-map.md) - HDInsight の Hadoop ドキュメントのガイド付きツアー。

* [Azure HDInsight の概要](../hdinsight-get-started.md) - HDInsight の Hadoop を使用するためのクイック スタート チュートリアル。

* [HDInsight のサンプルの実行](hdinsight-run-samples.md) - HDInsight に付属するサンプルの実行方法に関するチュートリアル。
	
* [Azure HDInsight SDK](http://msdnstage.redmond.corp.microsoft.com/library/dn479185.aspx) - HDinsight SDK のリファレンス ドキュメント。


### Apache Hadoop			

* <a target="_blank" href="http://hadoop.apache.org/">Apache Hadoop</a> - 大規模なデータ セットを複数のコンピューターで分散処理するためのフレームワークとなる Apache Hadoop ソフトウェア ライブラリについて説明します。	

* <a target="_blank" href="http://hadoop.apache.org/docs/r0.18.1/hdfs_design.html">HDFS</a> - Hadoop 分散ファイル システムは、Hadoop アプリケーションで使用する主要なストレージ システムで、そのアーキテクチャと設計について説明します。

* <a target="_blank" href="http://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html">MapReduce チュートリアル</a> - 複数の計算ノードから成る大規模なクラスターを利用して、大量のデータを迅速に並列処理する Hadoop アプリケーションを記述するためのプログラミング フレームワークについて説明します。

### Azure 上の SQL Database:	
		
* [Azure SQL データベース](http://msdn.microsoft.com/library/windowsazure/ee336279.aspx) - SQL データベースに関する MSDN ドキュメント。
	
* [SQL データベースの管理ポータル](https://msdn.microsoft.com/library/azure/dn771027.aspx) - 軽量で使いやすいデータベース管理ツール。クラウド環境で SQL データベースを管理する際に役立ちます。

* [Adventure Works for SQL Database](http://msftdbprodsamples.codeplex.com/releases/view/37304) - SQL データベースのサンプル データベースのダウンロード ページです。

### Microsoft ビジネス インテリジェンス (Windows 上の HDInsight 用)

Power Query アドインまたは Microsoft Hive ODBC ドライバーを使用すれば、Excel、PowerPivot、SQL Server Analysis Services、SQL Server Reporting Services などの使い慣れたビジネス インテリジェンス (BI) ツールを HDInsight と連携して、データの取得、分析、レポート生成を行うことができます。

これらの BI ツールは、ビッグ データ分析に有用です。
 
* [Power Query を使用した Excel から Hadoop への接続](hdinsight-connect-excel-power-query.md) - Microsoft Power Query for Excel を使用して、HDInsight クラスターに関連付けられたデータを格納する Azure ストレージ アカウントに Excel を接続する方法を説明します。 

* [Hive ODBC ドライバーを使用した Excel から Hadoop への接続](hdinsight-connect-excel-hive-ODBC-driver.md) - Microsoft Hive ODBC ドライバーを使用して、HDInsight からデータをインポートする方法を説明します。

* [Microsoft クラウド プラットフォーム](http://www.microsoft.com/server-cloud/solutions/business-intelligence/default.aspx) - Power BI for Office 365 の詳細、SQL Server 評価版のダウンロード、SharePoint Server 2013 と SQL Server BI のセットアップについて説明します。

* <a target="_blank" https://msdn.microsoft.com/library/hh231701.aspx">SQL Server Analysis Services の詳細</a>。

* <a target="_blank" href="http://msdn.microsoft.com/library/ms159106.aspx">SQL Server Reporting Services の詳細</a>。


### Hadoop のビッグ データ分析のソリューションを試してみる (Windows 上の HDInsight 用)

組織のビッグ データ分析を使用して、ビジネスの洞察を得ましょう。次に例をいくつか示します。

* [Analyze HVAC センサー データ](hdinsight-hive-analyze-sensor-data.md) - Hive を HDInsight (Hadoop) と共に使用してセンサー データを分析し、Microsoft Excel を使ってデータを視覚化する方法について説明します。このサンプルでは、Hive を使用して、HVAC システムによって生成された履歴データを処理し、設定した温度を正確に維持できないシステムを識別する方法について説明します。

* [Hive を HDInsight と共に使用して Web サイトのログを分析する](hdinsight-hive-analyze-website-log.md) - HDInsight で HiveQL を使用して、Web サイトのログを分析する方法について説明します。このサンプルでは、1 日の間に発生した外部 Web サイトからのアクセス数を調べ、ユーザーが経験した Web サイト エラーの概要を取得します。

* [Storm と HBase を HDInsight (Hadoop) で使用してリアルタイムでセンサー データを分析する](hdinsight-storm-sensor-data-analysis.md) - HDInsight で Storm クラスターを使用して、Azure Event Hubs からのセンサー データを処理し、処理したセンサー データをほぼリアルタイムの情報として Web ベースのダッシュボードに表示するソリューションを作成する方法について説明します。


[marketing-page]: ../services/hdinsight/
[component-versioning]: hdinsight-component-versioning.md
[HDInsight で提供される Hadoop クラスター バージョンの新機能]: hdinsight-component-versioning.md
[zookeeper]: http://zookeeper.apache.org/

<!--HONumber=54--> 