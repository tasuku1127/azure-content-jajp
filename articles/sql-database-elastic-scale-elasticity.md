﻿<properties title="Data Dependent Routing" pageTitle="シャードの弾力性" description="Azure SQL データベースを簡単にスケールアウトする機能であるシャードの弾力性の背後にある概念を説明すると共に、その例を提供します。" metaKeywords="sharding scaling, Azure SQL DB sharding, elastic scale, elasticity" services="sql-database" documentationCenter=""  manager="jhubbard" authors="sidneyh@microsoft.com"/>

<tags ms.service="sql-database" ms.workload="sql-database" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/02/2014" ms.author="sidneyh" />

# シャードの弾力性 

**シャードの弾力性**により、アプリケーション開発者は必要に応じて、データベース リソースを動的に拡大および縮小することができ、アプリケーションのパフォーマンスを最大化してコストを最小限に抑えることが可能になります。 Azure SQL Database の Elastic Scale と [Basic、Standard、および Premium の各サービス層](http://msdn.microsoft.com/ja-jp/library/azure/dn741340.aspx)を組み合わせることにより、非常に説得力のある弾力性シナリオが実現します。  Elastic Scale では**水平スケーリング**ができます。これは、データベース ([Elastic Scale 用語 ](sql-database-elastic-scale-glossary.md) ではこれを "シャード" と呼びます) の容量を拡大または縮小するために、これを**シャード セット**に追加したり、ここから削除する設計パターンの一種です。 同様に、SQL Database のサービス層は、**垂直スケーリング**機能も提供します。これは、需要に応じて 1 つのデータベースのリソースをスケールアップまたはスケールダウンすることができる機能です。  1 つのシャードの垂直スケーリングと多数のシャードの水平スケーリングを同時に実施することで、アプリケーション開発者にとって、パフォーマンス、容量、コスト最適化ニーズを満たす、非常に柔軟性の高い環境が実現します。

### 水平スケーリングの例: 人気急上昇のコンサート

水平スケーリングの標準的なシナリオとして、コンサート チケットのトランザクションを処理するアプリケーションがあります。 通常の顧客数の場合、アプリケーションは最小限のデータベース リソースを使用してチケット購入のトランザクションを処理します。  しかし、人気の高いコンサート チケットが発売されると、1 つのデータベースでは急増した顧客の需要を処理できなくなります。 

急増したトランザクションを処理するために、アプリケーションを水平にスケールアップします。 これで、アプリケーションはトランザクションの負荷を複数のシャードに分散できるようになります。 追加のリソースが不要になると、データベース層は縮小して通常の使用状況に戻ります。 この水平スケーリングにより、アプリケーションは顧客の需要に合わせてスケールアウトできます。また、リソースが不要になるとスケールインできます。   

### 垂直スケーリングの例:  テレメトリ

垂直スケーリングの標準的なシナリオとして、**シャード セット**を使用して操作テレメトリを格納するアプリケーションがあります。 このシナリオでは、1 日のテレメトリ データはすべて 1 つのシャードに配置することをお勧めします。 このアプリケーションでは、ある 1 つのシャードに今日のデータをインジェストし、明日以降のために新しいシャードを準備しておきます。 これで、オペレーショナル データの経過時間を進めたり、クエリを実行したりできます。 

テレメトリ データを高負荷でインジェストするには、より上位のサービス層を使用することをお勧めします。 つまり、Premium データベースの方が Basic データベースよりも望ましいと言えます。 データベースの容量に達した場合は、インジェストから分析およびレポートに切り替わります。 このため、Standard 層のパフォーマンスはタスクと同等です。 したがって、以前の (最近作成されたものではない) シャードのデータの低いパフォーマンス要件に適合するように、シャードのサービス層を垂直にスケールダウンすることができます。 

また、より高いパフォーマンスを達成するために、単一のデータベースのパフォーマンスを垂直スケーリングによって向上させることもできます。 1 つの例として納税申告を処理するアプリケーションがあります。 申告処理では、1 人の顧客のデータはすべて同じデータベースに保管し、そのシャードのパフォーマンスを上げる方が良いとされています。 アプリケーションによっては、リソースの垂直スケールアップまたはスケールダウンが、コストとパフォーマンスの両要件を最適化するために有効な場合もあります。 

データベース層の水平スケーリングと垂直スケーリングのどちらも、Elastic Scale アプリケーションおよびシャードの弾力性にとっての中核的な機能です。 

このドキュメントは、シャードの弾力性シナリオのほかにも、垂直スケーリングと水平スケーリングの両方の実装例を参照するコンテキストを提供します。 

## シャードの弾力性のメカニズム 

垂直スケーリングおよび水平スケーリングは、次の 3 つの基本的なコンポーネントで機能します。 

1. **テレメトリ**
2. **ルール**
3. **アクション**   

## <a name="telemetry"> </a>テレメトリ

**データ主導型の弾力性**は Elastic Scale アプリケーションの核となる機能です。 パフォーマンス要件に応じて、垂直または水平のどちらにスケールするか、テレメトリを用いたデータ主導で決定します。  

#### テレメトリ データ ソース
Azure SQL DB には、シャード弾力性のデータ ソースとして利用できる主要なソースがいくつかあります。 

1. **sys.resource_stats** ビューには、5 分間隔で**パフォーマンス テレメトリ**が公開されています。
2. **sys.resource_usage** ビューには、1 時間ごとの**データベース容量テレメトリ**が公開されています。  

マスター DB に次のクエリを実行することでリソースの使用状況のパフォーマンスを分析できます。このクエリでは、"Shard_20140623" がターゲット データベースの名前です。 

    SELECT TOP 10 *  
    FROM sys.resource_stats  
    WHERE database_name = 'Shard_20140623'  
    ORDER BY start_time DESC 

**パフォーマンス テレメトリ**は、一時的な動作を排除する目的で、一定の期間にわたって集計できます (下記のクエリでは 7 日間)。 

    SELECT  
    avg(avg_cpu_percent) AS 'Average CPU Utilization In Percent', 
        max(avg_cpu_percent) AS 'Maximum CPU Utilization In Percent', 
        avg(avg_physical_data_read_percent) AS 'Average Physical Data Read Utilization In Percent', 
        max(avg_physical_data_read_percent) AS 'Maximum Physical Data Read Utilization In Percent', 
        avg(avg_log_write_percent) AS 'Average Log Write Utilization In Percent', 
        max(avg_log_write_percent) AS 'Maximum Log Write Utilization In Percent', 
        avg(active_session_count) AS 'Average # of Sessions', 
        max(active_session_count) AS 'Maximum # of Sessions', 
        avg(active_worker_count) AS 'Average # of Workers', 
        max(active_worker_count) AS 'Maximum # of Workers' 
    FROM sys.resource_stats  
    WHERE database_name = ' Shard_20140623' AND start_time > DATEADD(day, -7, GETDATE()); 

**データベース容量**は、**sys.resource_usage** ビューに対する同様のクエリを通じて計測できます。 **storage_in_megabytes** 列の最大値はデータベースの現在のサイズになります。 このようなテレメトリは、特定のシャードの容量到達時にアプリケーションを水平スケーリングする場合に有効です。 

    SELECT TOP 10 * 
    FROM [sys].[resource_usage] 
    WHERE database_name = 'Shard_20140623'  
    ORDER BY time DESC 

データが特定のシャードにインジェストされたら、プロジェクトを 1 日先に進めて、渡されたワークロードを処理できるだけの十分な容量がシャードにあるかどうかを判断する方法が有効です。 線形回帰の実装が true でない期間に次のクエリを実行すると、連続する 2 日間のデータベース容量の差分の最大値が返されます。  このようなテレメトリは、その後、ルールに適用され、結果的に、アクション (または非アクション) が実施されます。 

    WITH MaxDatabaseDailySize AS( 
        SELECT 
            ROW_NUMBER() OVER (ORDER BY convert(date, [time]) DESC) as [Order], 
            CONVERT(date,[time]) as [date],  
            MAX(storage_in_megabytes) as [MaxSizeDay] 
        FROM [sys].[resource_usage] 
        WHERE  
            database_name = 'Shard_20140623' 
        GROUP BY CONVERT(date,[time]) 
        ) 
    
    SELECT 
        MAX(ISNULL(Size.[MaxSizeDay] - PreviousDaySize.[MaxSizeDay], 0)) 
    FROM  
        MaxDatabaseDailySize Size INNER JOIN 
        MaxDatabaseDailySize PreviousDaySize ON Size.[order]+1 = PreviousDaySize.[order] 
    WHERE 
        Size.[order] < 8 

## <a name="rule"></a>ルール  

このルールは、アクションを起こすかどうかの意思決定エンジンです。 ルールには、ごく単純なものもあれば、かなり複雑なものもあります。 次のコード スニペットで示すように、容量に重点を置いたルールを構成することができます。このルールでは、シャードがその最大容量の $SafetyMargin、つまり 80% に到達すると、新しいシャードがプロビジョニングされます。

    # 現在の DB サイズと日次の最大差分サイズを加算した数値がしきい値を超過しているかを判別します。 
    if( ($CurrentDbSizeMb + $MaxDbDeltaMb) -gt ($MaxDbSizeMb * $SafetyMargin))  
    {#provision new shard} 

上記のデータ ソースを前提とすると、数々のルールを作成して多様なシャード弾力性のシナリオを実現することができます。 

## <a name="action"></a>アクション     

ルールの結果に従うと、アクション (または非アクション) が結果になります。 最も一般的な 2 つのアクションには、次のものがあります。

* シャードのサービス層またはパフォーマンス レベルの増減 
* シャード セットからのシャードの削除または追加

水平および垂直スケーリングソリューションでは、いずれの場合も、アクションの結果が即時に現れるわけではないことに注意してください。 たとえば、垂直スケーリングで、サービス層を Basic データベースから Premium データベースに増やすために ALTER DATABASE コマンドを発行する場合、ある程度の時間がかかります。 この時間は、データベースのサイズによって大きく異なります (詳細については、「[データベースのサービス階層とパフォーマンス レベルの変更](http://msdn.microsoft.com/library/azure/dn369872.aspx)」を参照してください)。 同様に、新しいシャードの割り当てまたはプロビジョニングはいずれも即時には実施されません。 したがって、垂直および水平の両方のスケーリングでは、アプリケーションは、新しいデータベースを変更またはプロビジョニングする際、一定の時間を考慮する必要があります。  

## シャード弾力性のシナリオ例 

下図の例は、2 つの Elastic Scale のシナリオを表しています。 
1. シャード マップの水平スケーリング 
2. 個々のシャードの垂直スケーリング  

![Operationl Data Ingestion][1]

水平スケーリングを行うために、ルール (日付またはデータベースのサイズに基づく) を使用して、新しいシャードをプロビジョニングし、これをシャード マップに登録します。これで、データベース層が水平方向に拡大します。 次に、垂直スケーリングを実施するための第 2 のルールが実装されます。このルールでは、1 日以上前のすべてのシャードは、Premium エディションから Standard または Basic のエディションにダウングレードされます。 

テレメトリのシナリオをもう一度検討してみましょう。 アプリケーションのシャードは日付で指定されます。 テレメトリ データは継続的に収集されます。読み込み時には高パフォーマンスのエディションが必要とされますが、データの日数が経過するにつれ必要なパフォーマンスは低下します。 現在の日付のデータである [Tnow] は、高パフォーマンスのデータベース (Premium) に書き込まれます。 午前 0 時を過ぎると、前日のシャード (この時点で [T-1]) は、インジェストに使用されなくなります。 現在のデータは、現在の [Tnow] によってインジェストされます。 翌日になる前に、新しいシャードをプロビジョニングしておき、シャード マップ ([T+1]) に登録しておく必要があります。  

これは、前日に新しいシャードを毎日プロビジョニングするか、現在のシャードである ([Tnow]) が容量の上限に近づいた時点で新しいシャードをプロビジョニングことで実行可能です。 このメソッドのいずれかを使用すると、特定の 1 日に書き込まれた全テレメトリのデータ ローカリティが保持されます。 粒度をより細密にするために、時間ごとのシャーディングを適用することもできます。 新しいシャードのプロビジョニングが終了したら、[T-1] はクエリ実行とレポートのために使用されるため、データベースのパフォーマンス レベルを落としてコストを削減することをお勧めします。 データベース内のコンテンツが古くなったら、パフォーマンス レベルをさらに落とすことや、アプリケーションに応じてデータベースのコンテンツを Azure ストレージにアーカイブしたり、削除したりできます。 

## シャードの弾力性のシナリオの実行  

水平および垂直の両方のスケーリング シナリオの実装を実際に促進するために、スクリプト センターでは、数々の[シャード弾力性サンプル スクリプト](http://go.microsoft.com/?linkid=9862617)を作成および通知してきました。 Azure Automation サービスで実行するように記述されたこれらの PowerShell Runbook は、Elastic Scale クライアント ライブラリと Azure SQL Database とで通信するための数々のメソッドを提供しています。  これらのコード サンプルでビルドまたはコード サンプルから抽出する方法により、使用するアプリケーションで水平、垂直、またはその両方のスケーリング シナリオを実現するために必要なスクリプトを作成することができます。 


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-elasticity/data-ingestion.png

<!--anchors-->
[Telemetry]:#telemetry
[Rule]:#rule
[Action]:#action