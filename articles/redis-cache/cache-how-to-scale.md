<properties 
	pageTitle="Azure Redis Cache のスケーリング方法" 
	description="Azure Redis Cache インスタンスをスケーリングする方法を学習する" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/30/2015" 
	ms.author="sdanie"/>

# Azure Redis Cache のスケーリング方法

>[AZURE.NOTE]Azure Redis Cache のスケーリング機能は現在プレビュー中です。プレビュー期間中は、Premium レベル キャッシュへのスケーリング、キャッシュからのスケーリング、またはキャッシュ内でのスケーリングは行えません。

Azure Redis Cache は、キャッシュ サイズや機能の選択に柔軟性を持たせるために、さまざまなキャッシュ オファリングを用意しています。キャッシュを作成した後にご利用のアプリケーションの要件が変わった場合は、[Azure プレビュー ポータル](https://portal.azure.com)の **[価格レベルの変更]** ブレードを使用して、キャッシュのサイズを変更することができます。

## スケーリングするタイミング

Azure Redis Cache の[監視](cache-how-to-monitor.md)機能を使用して、キャッシュ アプリケーションの正常性とパフォーマンスを監視し、キャッシュのスケーリングが必要であるかどうかを判断できます。

次のメトリックを監視して、スケーリングの必要性の判断に役立てることができます。

-	Redis サーバーの負荷
-	メモリ使用量
-	ネットワーク帯域幅
-	CPU 使用率

現在のキャッシュがアプリケーションの要件を満たさないと判断した場合は、ご利用のアプリケーションにとって適切な価格レベルのキャッシュに変更できます。使用するキャッシュの価格レベルを決定する方法の詳細については、「[Redis Cache のサービス内容と適切なサイズの選択](cache-faq.md#what-redis-cache-offering-and-size-should-i-use)」を参照してください。

## キャッシュのスケーリング
キャッシュをスケーリングするには、[プレビュー ポータル](https://portal.azure.com)で[キャッシュを参照](cache-configure.md#configure-redis-cache-settings)し、**[設定]**、**[価格レベル]** をクリックします。

あるいは、**[Redis Cache]** ブレードの **[Standard レベル]** または **[Basic レベル]** 部分をクリックできます。

![[価格レベル]][redis-cache-pricing-tier-part]

**[価格レベル]** ブレードから希望の価格レベルを選択し、**[選択]** をクリックします。

![[価格レベル]][redis-cache-pricing-tier-blade]

>[AZURE.NOTE]別の価格レベルにスケーリングできますが、次のような制約があります。
>
>-	**Premium** キャッシュへのスケーリング、またはキャッシュからのスケーリングは行えません。
>-	**Standard** キャッシュから **Basic** キャッシュにスケーリングすることはできません。
>-	**Basic** キャッシュから **Standard** キャッシュにスケーリングすることはできますが、同時にサイズを変更することはできません。サイズを変更する必要がある場合、後続のスケーリング操作でサイズを変更できます。
>-	**C0 (250 MB)** サイズに、それよりも大きいサイズからスケールダウンすることはできません。

キャッシュを新しい価格レベルにスケーリングする際に、**[Redis Cache]** ブレードで状態が **[拡大中]** と表示されます。

![スケーリング][redis-cache-scaling]

スケーリングが完了すると、状態が **[拡大中]** から **[実行中]** に変わります。

## スケーリング処理を自動化する方法

Azure Redis Cache インスタンスは、プレビュー ポータルのほか、[Microsoft Azure 管理ライブラリ (MAML)](http://azure.microsoft.com/updates/management-libraries-for-net-release-announcement/) を使用してスケーリングすることができます。キャッシュをスケーリングするには、`IRedisOperations.CreateOrUpdate` メソッドを呼び出して `RedisProperties.SKU.Capacity` に新しいサイズを指定します。

    static void Main(string[] args)
    {
        // For instructions on getting the access token, see
        // https://azure.microsoft.com/documentation/articles/cache-configure/#access-keys
        string token = GetAuthorizationHeader();

        TokenCloudCredentials creds = new TokenCloudCredentials(subscriptionId,token);

        RedisManagementClient client = new RedisManagementClient(creds);
        var redisProperties = new RedisProperties();

        // To scale, set a new size for the redisSKUCapacity parameter.
        redisProperties.Sku = new Sku(redisSKUName,redisSKUFamily,redisSKUCapacity);
        redisProperties.RedisVersion = redisVersion;
        var redisParams = new RedisCreateOrUpdateParameters(redisProperties, redisCacheRegion);
        client.Redis.CreateOrUpdate(resourceGroupName,cacheName, redisParams);
    }

詳細については、[MAML を使用した Redis Cache の管理に関するページ](https://github.com/rustd/RedisSamples/tree/master/ManageCacheUsingMAML)のサンプルを参照してください。

## スケーリングに関する FAQ

次の一覧は、Azure Redis Cache のスケーリングに関するよく寄せられる質問への回答です。

## Premium キャッシュへのスケーリング、キャッシュからのスケーリング、またはキャッシュ内でのスケーリングを行えますか

プレビュー期間中は、**Premium** キャッシュに対するスケーリングは行えません。

## スケーリング後にキャッシュ名やアクセス キーを変更する必要はありますか

いいえ、スケーリング処理中にキャッシュ名やキーが変更されることはありません。

## スケーリングはどのように処理されますか

**Basic** キャッシュを別のサイズにスケーリングすると、キャッシュはシャット ダウンされて新しいキャッシュが新しいサイズでプロビジョニングされます。この間キャッシュは使用できず、キャッシュ内のすべてのデータは失われます。

**Basic** キャッシュを **Standard** キャッシュにスケーリングすると、レプリカ キャッシュがプロビジョニングされ、データがプライマリ キャッシュからレプリカ キャッシュにコピーされます。スケーリングの処理中、キャッシュは引き続き使用できます。

**Standard** キャッシュを別のサイズにスケーリングすると、一方のレプリカはシャット ダウンし、新しいサイズに再プロビジョニングされてデータが転送されます。もう一方のレプリカは、キャッシュ ノードの 1 つでエラーが発生した際に実行される処理と同様に、フェールオーバーを実行してから、再プロビジョニングされます。

## スケーリング中にキャッシュからデータは失われますか

新しいサイズに **Basic** キャッシュをスケーリングすると、すべてのデータが失われ、スケーリング処理中にキャッシュは使用できなくなります。

**Basic** キャッシュを **Standard** キャッシュにスケーリングする場合、通常、キャッシュのデータは保存されます。

**Standard** キャッシュをより大きいサイズにスケールアップする場合、通常、すべてのデータが保持されます。**Standard** キャッシュをより小さいサイズにスケールダウンする場合、キャッシュ内のデータ量によってはスケーリング後にデータが失われる可能性があります。スケールダウンの際にデータが失われた場合、[allkeys-lru](http://redis.io/topics/lru-cache) 削除ポリシーを使用してキーが削除されます。

Standard キャッシュには可用性について 99.9% の SLA がある一方で、データの喪失については SLA がありませんのでご注意ください。

## スケーリング中にキャッシュを使用できますか

**Standard** キャッシュはスケーリング処理中でも使用できます。

別のサイズにスケーリングする場合、**Basic** キャッシュはオフラインになりますが、**Basic** から **Standard** にスケーリングするときは引き続き利用できます。

## サポートされていない処理

**Premium** キャッシュへのスケーリング、キャッシュからのスケーリング、あるいはキャッシュ内でのスケーリングを行うことはできません。

**Standard** キャッシュから **Basic** キャッシュに変更することはできません。

**Basic** キャッシュから **Standard** キャッシュにスケーリングすることはできますが、同時にサイズを変更することはできません。サイズを変更する必要がある場合、後続のスケーリング操作でサイズを変更できます。

**C0** (250 MB) キャッシュからより大きなサイズにスケールアップすることはできますが、**C0** キャッシュをより小さいサイズにスケールダウンすることはできません。

スケーリング処理でエラーが発生すると、サービスは処理を取り消してキャッシュを元のサイズに戻すように試行します。

## スケーリングにはどのくらいの時間がかかりますか

スケーリングには約 20 分かかりますが、キャッシュ内のデータ量によって変わります。

## スケーリングが完了したことをどのようにして確認できますか

スケーリング処理の進捗はプレビュー ポータルで確認できます。スケーリングが完了すると、キャッシュの状態が **[実行中]** に変わります。

## この機能をプレビューする理由は何ですか

私たちはフィードバックを取得する目的でこの機能をリリースしています。フィードバックに基づき、まもなくこの機能の一般提供を開始する予定です。





  
<!-- IMAGES -->
[redis-cache-pricing-tier-part]: ./media/cache-how-to-scale/redis-cache-pricing-tier-part.png

[redis-cache-pricing-tier-blade]: ./media/cache-how-to-scale/redis-cache-pricing-tier-blade.png

[redis-cache-scaling]: ./media/cache-how-to-scale/redis-cache-scaling.png

<!---HONumber=Oct15_HO3-->