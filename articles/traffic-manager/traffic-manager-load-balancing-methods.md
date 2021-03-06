<properties 
   pageTitle="Traffic Manager - トラフィック ルーティング方法 | Microsoft Azure"
   description="この記事では、Traffic Manager で使用されるさまざまなトラフィック ルーティング方法について説明します。"
   services="traffic-manager"
   documentationCenter=""
   authors="joaoma"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="traffic-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="08/19/2015"
   ms.author="joaoma" />

# Traffic Manager のルーティング方法

Traffic Manager では、3 つのトラフィック ルーティング方法を使用できます。各 Traffic Manager プロファイルで一度に使用できるトラフィック ルーティング方法は 1 つだけですが、プロファイルに対していつでも別のトラフィック ルーティング方法を選択できます。

重要なのは、すべてのトラフィック ルーティング方法にエンドポイント監視が含まれている点です。Traffic Manager プロファイルを構成して、最も要件に合うトラフィック ルーティング方法を指定したら、監視の設定を構成します。監視が適切に構成されていると、Traffic Manager は、クラウド サービスや Web サイトで構成されているエンドポイントの状態を監視し、利用できないエンドポイントに対してはトラフィックを送信しません。Traffic Manager の監視の詳細については、「[Traffic Manager の監視について](traffic-manager-monitoring.md)」を参照してください。

Traffic Manager での 3 つのトラフィック ルーティング方法を次に示します。

- **フェールオーバー**: 同一または異なる Azure データセンター (管理ポータルでは「リージョン」と呼ばれます) にエンドポイントがあり、すべてのトラフィックにプライマリ エンドポイントを使用するが、プライマリ エンドポイントやバックアップ エンドポイントが利用不可になったときに備えてバックアップを用意する場合は、フェールオーバーを選択します。詳細については、[「フェールオーバーによるトラフィック ルーティング方法」](#failover-traffic-routing-method)を参照してください。

- **ラウンド ロビン**: 同一データセンター内または異なるデータセンターにある一連のエンドポイントに負荷を分散する場合は、ラウンド ロビンを選択します。詳細については、[「ラウンド ロビンによるトラフィック ルーティング方法」](#round-robin-traffic-routing-method)を参照してください。

- **パフォーマンス**: 地理的に異なる場所にエンドポイントがあり、待機時間が最短という意味で "最も近い" エンドポイントを使用するようにクライアントに要求する場合は、パフォーマンスを選択します。詳細については、[「パフォーマンスによるトラフィック ルーティング方法」](#performance-traffic-routing-method)を参照してください。

Azure Websites では、Web サイトのモードに関係なく、データセンター内の Web サイト向けにフェールオーバーおよびラウンド ロビンによるトラフィック ルーティング方法が既に用意されています。Traffic Manager を使用すると、異なるデータセンター内の Web サイトに対して、フェールオーバーおよびラウンド ロビンによるトラフィック ルーティング方法を指定できます。

>[AZURE.NOTE]DNS の有効期限 (TTL) により、DNS クライアントおよびリゾルバーは、解決された名前が DNS サーバー上でキャッシュされる期間を認識します。クライアントは、名前のローカル DNS キャッシュ エントリの期限が切れるまで、指定されたエンドポイントを使用してドメイン名を解決します。

## フェールオーバーによるトラフィック ルーティング方法

多くの場合、組織ではサービスの信頼性を維持する必要があります。そのために、会社のプライマリ サービスがダウンした場合に備えてバックアップ サービスを提供します。サービスのフェールオーバーの一般的なパターンは、同一のエンドポイントのセットを提供し、1 つ以上のバックアップのリストと共に、トラフィックをプライマリ サービスに送信することです。プライマリ サービスが利用不可の場合、要求しているクライアントは次の順位のサービスに送られます。リストにある 1 番目と 2 番目のサービスがどちらも利用不可の場合は、トラフィックは 3 番目のサービスに送信され、以降同様に処理されます。

フェールオーバーによるトラフィック ルーティング方法を構成する場合、選択したエンドポイントの順序が重要です。管理ポータルを使用して、プロファイルの [構成] ページでフェールオーバーの順序を構成できます。

図 1 に、一連のエンドポイントに対するフェールオーバーによるトラフィック ルーティング方法の例を示します。

![Traffic Manager でのフェールオーバー負荷分散](./media/traffic-manager-load-balancing-methods/IC750592.jpg)

**図 1**

次の手順の番号は、図 1 内の番号に対応しています。

1. Traffic Manager は DNS を介してクライアントから受信要求を受け取り、プロファイルを見つけます。
2. プロファイルには、順序が指定されたエンドポイントのリストが含まれています。Traffic Manager は、リストの最初にあるエンドポイントを調べます。そのエンドポイントがオンラインである場合 (実行中のエンドポイント監視に基づいて判別します)、そのエンドポイントの DNS 名をクライアントに対する DNS 応答で指定します。エンドポイントがオフラインの場合、Traffic Manager はリスト内の次のオンライン エンドポイントを判別します。この例では、CS-A はオフライン (利用不可) ですが、CS-B はオンライン (利用可能) です。
3. Traffic Manager は CS-B のドメイン名をクライアントの DNS サーバーに返し、DNS サーバーがそのドメイン名を IP アドレスに解決してクライアントに送信します。
4. クライアントは CS-B へのトラフィックを開始します。

## ラウンド ロビンによるトラフィック ルーティング方法

トラフィック ルーティングの一般的なパターンは、同一のエンドポイントのセットを提供し、ラウンド ロビン方式で各エンドポイントにトラフィックを送信することです。ラウンド ロビン方式では、トラフィックがさまざまなエンドポイントに分割されます。正常なエンドポイントがランダムに選択され、サービスが停止していることが検出されるとトラフィックは送信されません。詳細については、「[Traffic Manager の監視](../traffic-manager-onitoring.md)」を参照してください。

図 2 に、一連のエンドポイントに対するラウンド ロビンによるトラフィック ルーティング方法の例を示します。

![Traffic Manager でのラウンド ロビン負荷分散](./media/traffic-manager-load-balancing-methods/IC750593.jpg)

**図 2**

次の手順の番号は、図 2 内の番号に対応しています。

1. Traffic Manager はクライアントから受信要求を受け取り、プロファイルを見つけます。
2. プロファイルにはエンドポイントのリストが含まれています。Traffic Manager はこのリストからランダムにエンドポイントを選択します。その際、Traffic Manager のエンドポイント監視でオフライン (利用不可) と判別されたエンドポイントはすべて除外されます。この例では、エンドポイント CS-B が選択されます。
3. Traffic Manager は、CS-B のドメイン名をクライアントの DNS サーバーに返します。クライアントの DNS サーバーは、このドメイン名を IP アドレスに解決して、クライアントに送信します。
4. クライアントは CS-B へのトラフィックを開始します。

ラウンド ロビンによるトラフィック ルーティングでは、ネットワーク トラフィックの重み付け分布もサポートされます。図 3 に、一連のエンドポイントに対する重み付けラウンド ロビン トラフィック ルーティング方法の例を示します。

![重み付けラウンド ロビン負荷分散](./media/traffic-manager-load-balancing-methods/IC750594.png)

**図 3**

重み付けラウンド ロビン トラフィック ルーティングを使用すると、各エンドポイントに割り当てた重み値に基づいて、さまざまなエンドポイントに負荷を分散できます。重みが大きければ大きいほど、頻繁にエンドポイントが返されます。この方法が役立つシナリオとして、次のようなものがあります。

- アプリケーションの段階的アップグレード: 新しいエンドポイントにルーティングするトラフィックのパーセンテージを割り当て、時間と共に 100% までトラフィックを徐々に増やします。
- Azure へのアプリケーション移行: Azure と外部エンドポイントの両方でプロファイルを作成し、各エンドポイントにルーティングするトラフィックの重みを指定します。
- 追加容量のクラウド バースト: Traffic Manager プロファイルに対応させることで、オンプレミスのデプロイメントをクラウドにすばやく拡張します。クラウドに追加容量が必要なとき、エンドポイントを追加または有効にして、各エンドポイントに送るトラフィックの割り当てを指定します。

現時点では、管理ポータルを使用して重み付けトラフィック ルーティングを構成することはできません。Azure では、関連するサービス管理 REST API や Azure PowerShell コマンドレットを使用して、プログラムでこの方法を構成することができます。

REST API の使用方法の詳細については、「[Traffic Manager の操作 (REST API リファレンス)](http://go.microsoft.com/fwlink/p/?LinkId=313584)」を参照してください。

Azure PowerShell コマンドレットの使用方法の詳細については、「[Azure Traffic Manager コマンドレット](http://go.microsoft.com/fwlink/p/?LinkId=400769)」を参照してください。構成の例については、Azure ブログの「[Azure Traffic Manager External Endpoints and Weighted Round Robin via PowerShell](http://azure.microsoft.com/blog/2014/06/26/azure-traffic-manager-external-endpoints-and-weighted-round-robin-via-powershell/)」を参照してください。

単一のクライアントからのプロファイルをテストして、均等ラウンド ロビンまたは重み付けラウンド ロビンの動作を確認するには、プロファイルの均等値または重み値に従って、DNS 名が異なるエンドポイントの IP アドレスに解決されることを確認します。テストの際には、クライアント側の DNS キャッシュを無効にするか、毎回 DNS キャッシュをクリアして、新しい DNS 名のクエリが送信されるようにする必要があります。

## パフォーマンスによるトラフィック ルーティング方法

世界各地のさまざまなデータセンターにあるエンドポイントに対するトラフィック ルーティングを行うため、要求元クライアントとエンドポイント間の最短待機時間という意味で最も近いエンドポイントに、受信トラフィックを送ることができます。通常、"最も近い" エンドポイントは、地理的な距離が最短のエンドポイントに対応します。パフォーマンスによるトラフィック ルーティング方法を選択すると、トラフィックを距離や待機時間に基づいて振り分けることはできますが、振り分け時にネットワークの構成や負荷のリアルタイムの変化を考慮することはできません。

パフォーマンスによるトラフィック ルーティング方法では、要求元のクライアントを検出し、それを最も近いエンドポイントにルーティングします。"近さの程度" は、さまざまな IP アドレスと各 Azure データセンター間のラウンドトリップ時間を示すインターネット待機時間テーブルによって判別されます。このテーブルは定期的に更新されますが、インターネット経由のパフォーマンスをリアルタイムで反映したものではありません。特定のサービスの負荷は考慮されていません。ただし、Traffic Manager は選択した方法に基づいてエンドポイントを監視し、エンドポイントが使用できない場合は DNS クエリの応答に含めません。つまり、パフォーマンスによるトラフィック ルーティングには、フェールオーバーによるトラフィック ルーティング方法も組み込まれています。

図 4 に、一連のエンドポイントに対するパフォーマンスによるトラフィック ルーティング方法の例を示します。

![Traffic Manager でのパフォーマンス負荷分散](./media/traffic-manager-load-balancing-methods/IC753237.jpg)

**図 4**

次の手順の番号は、図 4 内の番号に対応しています。

1. Traffic Manager は、定期的にインターネット待機時間テーブルを作成します。Traffic Manager のインフラストラクチャでは、世界のさまざまな地点と、エンドポイントをホストしている Azure データセンター間のラウンドトリップ時間を判別するためのテストが実行されます。
2. Traffic Manager は、ローカル DNS サーバーを介してクライアントから受信要求を受け取り、プロファイルを見つけます。
3. Traffic Manager は、受信 DNS 要求の IP アドレスに対応する行をインターネット待機時間テーブルで見つけます。ユーザーのローカル DNS サーバーが Traffic Manager プロファイル名に対して権限のある DNS サーバーを見つけるための反復的な DNS クエリを実行するため、DNS クエリはクライアントのローカル DNS サーバーの IP アドレスから送信されます。
4. Traffic Manager は、プロファイルで定義されているエンドポイントをホストするデータセンターから、待機時間が最小のものを特定します。この例では、CS-B になります。
5. Traffic Manager は CS-B のドメイン名をクライアントのローカル DNS サーバーに返し、DNS サーバーがそのドメイン名を IP アドレスに解決してクライアントに送信します。
6. クライアントは CS-B へのトラフィックを開始します。

**注意する点:**

- プロファイルに同一データセンター内の複数のエンドポイントが含まれる場合、そのデータセンターに送られるトラフィックは、エンドポイントの監視に応じて、正常で利用可能なエンドポイント間で均等に分散されます。
- (エンドポイントの監視に応じて) 特定のデータセンター内のすべてのエンドポイントが利用できない場合、これらのエンドポイントに対するトラフィックは、次に近いエンドポイントではなく、プロファイルで指定されている他のすべての利用可能なエンドポイント間で分散されます。これは、次に近いエンドポイントが過負荷になった場合に発生する可能性のある連鎖エラーを回避するのに役立ちます。
- インターネット待機時間テーブルが更新されると、トラフィック パターンとエンドポイントの負荷が変わる場合があります。これらの変更は、最小限にする必要があります。
- 外部エンドポイントでパフォーマンスによるトラフィック ルーティング方法を使用する場合、それらのエンドポイントの場所を指定する必要があります。デプロイメントに最も近い Azure リージョンを選択してください。詳細については、「[Traffic Manager でのエンドポイントの管理](traffic-manager-endpoints.md)」を参照してください。

## Traffic Manager に関する図

Traffic Manager に関する独自のプレゼンテーション用の PowerPoint スライドとして、または独自の目的に合わせて変更するために、このトピックで扱った図が必要な場合は、MSDN ドキュメント内の「[Traffic Manager に関する図](http://gallery.technet.microsoft.com/Traffic-Manager-figures-in-887e7c99)」を参照してください。

## 次のステップ

[Traffic Manager について](traffic-manager-overview.md)

[Traffic Manager の監視について](traffic-manager-monitoring.md)

[Traffic Manager の操作 (REST API リファレンス)](http://go.microsoft.com/fwlink/p/?LinkID=313584)

[Cloud Services](http://go.microsoft.com/fwlink/p/?LinkId=314074)

[Websites](http://go.microsoft.com/fwlink/p/?LinkId=393327)

[Azure Traffic Manager コマンドレット](http://go.microsoft.com/fwlink/p/?LinkId=400769)

 

<!---HONumber=Oct15_HO3-->