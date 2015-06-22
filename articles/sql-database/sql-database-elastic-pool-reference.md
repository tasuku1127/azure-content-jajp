<properties 
	pageTitle="Azure SQL 弾力性データベース プールのリファレンス" 
	description="このリファレンスでは、弾力性データベース プールの記事やプログラミング情報のリンクと詳細が提供されています。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database"
	ms.devlang="NA"
	ms.date="04/29/2015" 
	ms.author="sstein" 
	ms.workload="data-management" 
	ms.topic="article" 
	ms.tgt_pltfrm="NA"/>


# SQL Database 弾力性データベース プールのリファレンス (プレビュー)

数十台、数百台、何千ものデータベースを持つ SaaS 開発者にとっては、弾力性データベース プールを使用することで、プロセスの作成、保守、データベースのグループ全体にわたるパフォーマンスとコストの両方の管理が簡素化されます。

このリファレンスでは、弾力性プールの記事やプログラミング情報のリンクと詳細が提供されています。

## 概要

弾力性プールは、データベース スループット ユニット (DTU) と、複数のデータベースで共有される記憶域 (GB) のコレクションです。弾力性データベースは、いつでも、プールに追加したり、プールから削除したりできます。プール内の弾力性データベースでは、使用可能なリソースを解放するプールから必要なリソースのみが、それを必要とするアクティブなデータベース用にのみ使用されます。



## 弾力性プールを作成、管理するための前提条件


- 弾力性プールは、Azure SQL Database の V12 サーバーでのみ使用できます。   
- PowerShell と弾力性プール用の REST API は、Azure リソース マネージャー (ARM) でのみサポートされています。サービス管理コマンド (RDFE) はサポートされていません。 
- 弾力性プールの作成と管理は、[Microsoft Azure ポータル](https:portal.azure.com) でのみサポートされています。 


## 現在のプレビューの制限

- 現在のプレビューの弾力性プールに対する価格レベルは Standard です。  
- 弾力性プールにデータベースを直接インポートすることはサポートされていません。スタンドアロン データベースにインポートして、そのデータベースをプールに移動できます。プール内からのデータベースのエクスポートはサポートされています。


## 記事の一覧

次の記事は、弾力性データベースと弾力性ジョブを使い始める時に役立ちます。

| 記事 | 説明 |
| :-- | :-- |
| [SQL Database の弾力性プール](sql-database-elastic-pool.md) | 弾力性プールの概要 |
| [Azure ポータルを使った SQL Database の弾力性プールの作成と管理](sql-database-elastic-pool-portal.md) | Azure ポータルを使った弾力性プールの作成と管理方法 |
| [PowerShell を使った SQL Database の弾力性プールの作成と管理](sql-database-elastic-pool-powershell.md) | PowerShell コマンドレットを使った弾力性プールの作成と管理方法 |
| [弾力性データベース ジョブの概要](sql-database-elastic-jobs-overview.md) | プール内のすべての弾力性データベース間で T-SQL スクリプトを実行できるようにする、弾力性ジョブ サービスの概要 |
| [弾力性データベース ジョブ コンポーネントのインストール](sql-database-elastic-jobs-service-installation.md) | 弾力性データベース ジョブ サービスのインストール方法 |
| [弾力性ジョブ サービスに必要なユーザーの作成](sql-database-elastic-jobs-add-logins-to-dbs.md) | 弾力性データベース ジョブのスクリプトを実行するには、プール内のすべてのデータベースに適切なアクセス許可を持つユーザーを追加する必要があります。 |
| [弾力性データベース ジョブ コンポーネントのアンインストール方法](sql-database-elastic-jobs-uninstall.md) | 弾力性データベース ジョブ サービスをインストールしようとするときに発生するエラーからの回復 |



## 名前空間とエンドポイントの詳細
弾力性プールは、Microsoft Azure SQL Database の “ElasticPool” タイプの ARM リソースです。

- **名前空間**: Microsoft.Sql/ElasticPool
- REST API 呼び出しの**セカンダリ エンドポイント** (Azure リソース マネージャー): https://management.azure.com 



## 弾力性データベース プールのプロパティ

| プロパティ | 説明 |
| :-- | :-- |
| creationDate | プールが作成された日付。 |
| databaseDtuMax | プール内の 1 つのデータベースによって使用可能な DTU の最大数。データベースの DTU の最大値はリソース保証ではありません。DTU の最大値は、プール内のすべてのデータベースに適用されます。 |
| databaseDtuMin | プール内の 1 つのデータベースで保証される DTU の最小数。データベースの DTU の最小値 は 0 に設定されることがあります。DTU の最小値 は、プール内のすべてのデータベースに適用されます。プール内のデータベース数の積とデータベースの DTU の最小値はプール自体の DTU を超えることはできません。 |
| DTU | プール内のすべてのデータベースで共有されている DTU 数。 |
| edition | プールのサービス階層。プール内のすべてのデータベースが、このエディションです。 |
| elasticPoolId | プールのインスタンスの GUID。 |
| elasticPoolName | プールの名前。名前は、その親サーバーを基準にして一意です。 |
| location | プールが作成されたデータ センターの場所。 |
| state | サブスクリプションの請求書の支払いが延滞すると、状態は「無効」になり、それ以外の場合は「準備完了」になります。 |
| storageMB | プールの記憶域上限 (MB) です。プール内の 1 つのデータベースは、Standard Edition 記憶域の上限 (250 MB) まで使用できますが、プール内のすべてのデータベースによって使用されるストレージの合計がこのプールの制限を超えることはできません。 |


## 弾力性プールと弾力性データベースの DTU と記憶域の上限

プールの記憶域の上限は、プールの DTU の量によって決まります。各 DTU = 1 GB の記憶域です。たとえば、200 DTU プールの記憶域の上限は 200 GB になります。

| プロパティ | 既定値 | 有効な値 |
| :-- | :-- | :-- |
| DTU | 200 | 200、400、800、1200 |
| databaseDtuMax | 100 | 10、20、50、100 |
| databaseDtuMin | 0 | 0、10、20、50 |
| storageMB | 200 GB* | 200 GB、400 GB、800 GB、1200 GB |

*API の単位は GB ではなく、MB

## ワーカーとセッションの制限

同時実行ワーカーと弾力性プール内のすべてのデータベースに対してサポートされている同時実行セッションの最大数は、プールの DTU 設定に依存します。

| DTU | 最大同時実行ワーカー数 | 最大同時セッション数 |
| :-- | :-- | :-- |
| 200 | 400 | 4,800 |
| 400 | 800 | 9,600 |
| 800 | 1,600 | 19,200 |
| 1,200 | 2,400 | 28,800 |


## Azure リソース マネージャー の制限事項

弾力性プールには、Azure SQL Database V12 サーバーが必要です。サーバーは、リソース グループ内にあります。

- 各リソース グループには、最大で 800 台のサーバーを含めることができます。
- 各サーバーには、最大で 800 個の弾力性プールを含めることができます。
- 各弾力性プールには、最大で 100 個のデータベースを含めることができます。


## 弾力性プール操作の待機時間

- 通常、データベースあたりに保証される DTU (databaseDtuMin) またはデータベースあたりの最大 DTU (databaseDtuMax) の変更は、5 分以内で完了します。 
- プールの DTU や記憶域の上限 (storageMB) の変更は、プール内のすべてのデータベースで使用される領域の合計に依存します。変更の平均時間は、100 GB あたり 90 分以下です。たとえば、プール内のすべてのデータベースで使用される領域の合計が 200 GB の場合、プールの DTU や記憶域上限の変更にかかる想定待機時間は、3 時間以下になります。 



## 弾力性データベース プールの PowerShell コマンドレットと REST API コマンド (Azure リソース マネージャーのみ)

弾力性プールの作成と管理には、次の PowerShell コマンドレットと REST API コマンドを使用できます。

| [PowerShell コマンドレット](https://msdn.microsoft.com/library/mt125356.aspx) | [REST API のコマンド](https://msdn.microsoft.com/library/azure/mt163571.aspx) |
| :-- | :-- |
| Get-AzureSqlDatabase | Get Azure SQL database |
| Get-AzureSqLElasticPool | Get Azure SQL Database elastic database pool |
| Get-AzureSqlElasticPoolActivity | Get Azure SQL Database elastic database pool operations |
| Get-AzureSqlElasticPoolDatabase | Get Azure SQL Database elastic database |
| Get-AzureSqlElasticPoolDatabaseActivity | Get Azure SQL Database elastic database operations |
| Get-AzureSqlServer | Get Azure SQL Database server |
| Get-AzureSqlServerFirewallRule | Get Azure SQL Database server firewall rule |
| Get-AzureSqlServerServiceObjective | Get Azure SQL Database server service objective |
| New-AzureSqlDatabase | Create Azure SQL database |
| New-AzureSqlElasticPool | Create Azure SQL Database elastic database pool |
| New-AzureSqlServer | Create Azure SQL Database server |
| New-AzureSqlServerFirewallRule | Create Azure SQL Database server firewall rule) |
| Remove-AzureSqlDatabase | Remove Azure SQL database |
| Remove-AzureSqlElasticPool | Remove Azure SQL Database elastic database pool |
| Remove-AzureSqlServer | Remove Azure SQL Database server |
| Set-AzureSqlDatabase | Set Azure SQL database |
| Set-AzureSqlElasticPool | Set Azure SQL Database elastic database pool |
| Set-AzureSqlServer | Set Azure SQL Database server |
| Set-AzureSqlServerFirewallRule | Set Azure SQL Database server firewall rule |
| Get-Metrics | Get Metrics |


## 課金と料金情報

弾力性データベース プールは、次の特性ごとに課金されます。

- 弾力性プールは、プールにデータベースがない場合でも、その作成時に課金されます。 
- 弾力性プールは 1 時間ごとに課金されます。これは、スタンドアロン データベースのパフォーマンス レベルと同じ使用状況測定の頻度です。 
- 弾力性プールが新しい DTU 量にサイズ変更されると、サイズ変更操作が完了するまでは新しい DTUS 量に応じた課金はされません。これは、スタンドアロン データベースのパフォーマンス レベルを変更する場合と同様のパターンに従っています。 


- 弾力性プールの価格は、プールの DTU 数と、プール内のデータベース数に基づきます。
- 価格は、(プール DTU 数) x (DTU あたりの単価) + (データベース数) x (データベースあたりの単価) で計算されます。

弾力性プールの DTU 単価は、同じサービス階層のスタンドアロン データベースの DTU 単価よりも高くなります。詳細については、「[SQL Database 料金](http://azure.microsoft.com/pricing/details/sql-database/)」をご覧ください。

## 弾力性データベース プールのエラー

| ErrorNumber | ErrorSeverity | ErrorFormat | ErrorInserts | ErrorCause | ErrorCorrectiveAction |
| :-- | :-- | :-- | :-- | :-- | :-- |
| 1132 | EX_RESOURCE | 弾力性プールが、その記憶域の上限に達しました。弾力性プールの記憶域の使用率が (%d) MB を超えることはできません。 | 弾力性プール領域の上限 (MB) です。 | 弾力性プールの記憶域が上限に達したときに、データベースにデータを記述しようとしています。 | 可能であれば、弾力性プールの DTU を増やして、その記憶域の上限を上げることを検討する、弾力性プール内の個々のデータベースで使用される記憶域を減らす、弾力性プールからデータベースを削除するのいずれかを行ってください。 |
| 10929 | EX_USER | %S の最低限保証は %d、最大値は %d 、データベースの現在の使用状況は %d です。ただし、サーバーは現在ビジー状態であり、このデータベースの %d を超える要求をサポートできません。詳細については、[http://go.microsoft.com/fwlink/?LinkId=267637](http://go.microsoft.com/fwlink/?LinkId=267637) をご覧ください。それ以外の場合は、後でもう一度やり直してください。 | データベースあたりの DTU の最小値、データベースあたりの DTU の最大値 | 弾力性プール内のすべてのデータベース間での同時実行ワーカー (要求) の合計数が、プールの制限を超えようとしました。 | 可能であれば、弾力性プールの DTU を増やしてワーカーの上限を上げることを検討するか、弾力性プールからデータベースを削除してください。 |
| 40844 | EX_USER | サーバー '%ls' のデータベース '%ls' は、弾力性プールの '%ls' エディションのデータベースであり、連続コピー リレーションシップを持つことはできません。 | データベース名、データベースのエディション、サーバー名 | 弾力性プール内の Premium 以外のデータベースには、StartDatabaseCopy コマンドが発行されます。 | 近日対応予定 |
| 40857 | EX_USER | サーバー '%ls ' の弾力性プールが見つかりませんでした。弾力性プール名は ' %ls ' です。 | サーバー名、弾力性プール名 | 指定された弾力性プールが、指定されたサーバー内にありません。 | 有効な弾力性プール名を指定してください。 |
| 40858 | EX_USER | 弾力性プール '%ls' は サーバー '%ls' に既に存在します。 | 弾力性プール名、サーバー名 | 指定された弾力性プールは、指定された論理サーバー内に既に存在します。 | 新しい弾力性プール名を指定してください。 |
| 40859 | EX_USER | 弾力性プールでは、サービス階層 '%ls' はサポートされていません。 | 弾力性プールのサービス階層 | 指定されたサービス階層は、弾力性プールのプロビジョニングにはサポートされていません。 | 正しいエディションを指定するか、既定のサービス階層を使用する場合はサービス階層を空のままにしてください。 |
| 40860 | EX_USER | 弾力性プール '%ls' とサービス目標 '%ls' の組み合わせが正しくありません。 | 弾力性プール名、サービス レベル目標名 | 弾力性プールとサービス目標は、サービス目標が 'ElasticPool' に指定されている場合にのみ同時に指定できます。 | 弾力性プールとサービス目標の適切な組み合わせを指定してください。 |
| 40861 | EX_USER | データベースのエディション '%.*ls' は、弾力性プールのサービス階層 ('%.*ls') と同じである必要があります。 | データベースのエディション、弾力性プールのサービス階層 | データベースのエディションの大文字と小文字が、弾力性プールのサービス階層と異なります。 | 弾力性プールのサービス階層と異なるデータベース エディションを指定しないでください。データベースのエディションを指定する必要がないことにご注意ください。 |
| 40862 | EX_USER | 弾力性プールのサービス目標が指定されている場合は、弾力性プール名を指定する必要があります。 | なし | 弾力性プールのサービス目標は、弾力性プールを一意に識別することはできません。 | 弾力性プールのサービス目標を使用する場合は、弾力性プール名を指定してください。 |
| 40864 | EX_USER | 弾力性プールの DTU は、サービス階層 '%.*ls' に対して (%d) DTU 以上である必要があります。 | 弾力性プールの DTU、弾力性プールのサービス階層 | 最小値を下回る弾力性プールの DTU を設定しようとしています。 | 弾力性プールの DTU を最小値以上に設定し直してください。 |
| 40865 | EX_USER | 弾力性プールの DTU は、サービス階層 '%.*ls' に対して (%d) DTU を超えることはできません。 | 弾力性プールの DTU、弾力性プールのサービス階層 | 弾力性プールの DTU を最大値を超える値に設定しようとしています。 | 弾力性プールの DTU を最大値を超えないように設定し直してください。 |
| 40867 | EX_USER | データベースあたりの DTU の最大値は、サービス階層 '%.*ls' に対して少なくとも (%d) である必要があります。 | データベースあたりの DTU の最大値、弾力性プールのサービス階層 | サポートされている制限を下回るデータベースあたりの DTU 最大値を設定しようとしています。 | 目的の設定をサポートする弾力性プールのサービス階層を使用することをご検討ください。 |
| 40868 | EX_USER | データベースあたりの DTU の最大値は、サービス階層 '%.*ls' に対して (%d) を超えることはできません。 | データベースあたりの DTU 最大値、弾力性プールのサービス階層 | サポートされている制限を超えるデータベースあたりの DTU 最大値を設定しようとしています。 | 目的の設定をサポートする弾力性プールのサービス階層を使用することをご検討ください。 |
| 40870 | EX_USER | データベースあたりの DTU 最小値は、サービス階層 '%.*ls' に対して (%d) を超えることはできません。 | データベースあたりの DTU 最小値、弾力性プールのサービス階層 | サポートされている制限を超えるデータベースあたりの DTU 最小値を設定しようとしています。 | 目的の設定をサポートする弾力性プールのサービス階層を使用することをご検討ください。 |
| 40873 | EX_USER | データベース数 (%d) と データベースあたりの DTU の最小値 (%d) は、弾力性プールの DTU (%d) を超えることはできません。 | 弾力性プール内のデータベース数、データベースあたりの DTU 最小値、弾力性プールの DTU | 弾力性プールの DTU を超える弾力性プール内のデータベースあたりの DTU 最小値を指定しようとしています。 | 弾力性プールの DTU を増やす、データベースあたりの DTU 最小値を減らす、弾力性プール内のデータベース数を減らすことをご検討ください。 |
| 40877 | EX_USER | データベースが含まれていない場合を除いて、弾力性プールを削除できません。 | なし | 弾力性プールに、1 つ以上のデータベースが含まれているため、削除できません。 | 弾力性プールからデータベースを削除して、弾力性プールを削除してください。 |
| 40881 | EX_USER | 弾力性プール '%.*ls' のデータベース数が上限に達しました。弾力性プールのデータベース数上限は、(%d) DTU を含む弾力性プールの場合、(%d) を超えることはできません。 | 弾力性プール名、弾力性プールのデータベース数上限、リソース プールの DTU | 弾力性プールのデータベース数が上限に達したときに、データベースを作成または弾力性プールにデータベースを追加しようとしています。 | 可能であれば、弾力性プールの DTU を増やして、データベースの上限を上げることを検討するか、弾力性プールからデータベースを削除してください。 |
| 40889 | EX_USER | 弾力性プール '%.*ls' の DTU または記憶域の上限を下げることはできません。下げると、データベースに十分な記憶域スペースを提供できなくなるためです。 | 弾力性プールの名前。 | 記憶域の使用率を下回る弾力性プールの記憶域の上限を下げようとしています。 | 弾力性プール内の個々のデータベースの記憶域使用率を減らすことを検討するか、その DTU または記憶域の上限を下げるためにプールからデータベースを削除してください。 |
| 40891 | EX_USER | データベースあたりの DTU 最小値 (%d) は、データベースあたりの DTU 最大値 (%d) を超えることはできません。 | データベースあたりの DTU 最小値、データベースあたりの DTU 最大値 | データベースあたりの DTU 最大値を超えるデータベースあたりの DTU 最小値を設定しようとしています。 | データベースあたりの DTU の最小値がデータベースあたりの DTU 最大値を超えていないことをご確認ください。 |
| TBD | EX_USER | 弾力性プール内の個々のデータベースの記憶域サイズは、サービス階層の弾力性プール '%.*ls' で許容される最大サイズを超えることはできません。 | 弾力性プールのサービス階層 | データベースの最大サイズが、弾力性プールのサービス階層によって許可されている最大サイズを超えています。 | データベースの最大サイズを、弾力性プールのサービス階層によって許可されている最大サイズの制限内に設定してください。 |

<!---HONumber=58--> 