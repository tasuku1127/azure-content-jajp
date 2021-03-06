<properties 
	pageTitle="Azure DocumentDB のデータのモデル化 | Microsoft Azure" 
	description="Azure DocumentDB のような NoSQL ドキュメント データベースのデータをモデル化する方法について説明します。" 
	services="documentdb" 
	authors="ryancrawcour" 
	manager="jhubbard" 
	editor="mimig1" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="08/24/2015" 
	ms.author="ryancraw"/>

#DocumentDB のデータのモデル化#
DocumentDB のようなスキーマのないデータベースでは、データ モデルの変更はとても簡単ですが、データについて十分検討する必要があります。

データをどのように格納するか。 アプリケーションがどのようにデータを取得してクエリを実行するか。 アプリケーションの負荷は読み取りと書き込みのどちらが高いか。

この記事を読むと、次の質問に回答できるようになります。

- ドキュメント データベース内のドキュメントについてどのように考えるか。
- データのモデル化とは何か、なぜ考慮する必要があるか。 
- ドキュメント データベース内のデータのモデル化とリレーショナル データベースはどう違うか。
- 非リレーショナル データベース内のデータのリレーションシップをどのように表現するか。
- データの埋め込みを使用する場合とデータのリンクを使用する場合。

##データの埋め込み##
DocumentDB などのドキュメント ストア内のデータのモデル化を開始する際、JSON で表現された **内蔵ドキュメント**としてエンティティを扱ってみてください。

先を急ぐ前に、一歩引いてリレーショナル データベース内のデータをどのようにモデル化するかを考えてみましょう。これは、多くのユーザーが慣れ親しんでいるテーマです。次の例は、ある個人がどのようにリレーショナル データベースに格納されるかを示しています。

![リレーショナル データベース モデル](./media/documentdb-modeling-data/relational-data-model.png)

リレーショナル データベースを操作する際に、長年教え込まれてきたことは "正規化" です。

データの正規化には通常、個人などのエンティティを取得し、細かいデータ要素に分解する操作が含まれます。上の例では、ある個人は複数の連絡先詳細レコードや複数の住所レコードを持っています。連絡先詳細をさらに分解して、型などの共通フィールドを抽出します。住所も同様で、各レコードには*自宅*や*会社*などの型があります。

データを正規化する際に指標となる前提は、各レコードについて**冗長なデータを格納するのではなく**、データを参照することです。この例では、連絡先詳細と住所のすべてを含めて個人のデータを読み取るには、JOIN を使用して実行時にデータを効率的に統合します。

	SELECT p.FirstName, p.LastName, a.City, cd.Detail
	FROM Person p
    JOIN ContactDetail cd ON cd.PersonId = p.Id
    JOIN ContactDetailType on cdt ON cdt.Id = cd.TypeId
    JOIN Address a ON a.PersonId = p.Id

1 人の個人の連絡先詳細や住所を更新するには、多数の個別のテーブルへの書き込み操作が必要です。

では、同じデータをドキュメント データベース内の内蔵エンティティとしてモデル化する方法を見ていきましょう。
		
	{
	    "id": "1",
	    "firstName": "Thomas",
	    "lastName": "Andersen",
	    "addresses": [
	        {            
	            "line1": "100 Some Street",
	            "line2": "Unit 1",
	            "city": "Seattle",
	            "state": "WA",
	            "zip": 98012
	        }
	    ],
	    "contactDetails": [
	        {"email: "thomas@andersen.com"},
	        {"phone": "+1 555 555-5555", "extension": 5555}
	    ] 
	}

前に示した方法を使用して、この個人に関するすべての情報 (連絡先詳細や住所など) を**埋め込んだ**個人レコードを**非正規化して**、1 つの JSON ドキュメントにしました。また、固定スキーマに限定されているわけではないので、まったく別の形で連絡先詳細を含めるような柔軟性もあります。

データベースから完全な個人レコード全体を取得するのは、1 つのコレクションと 1 つのドキュメントに対する 1 つの読み取り操作です。連絡先詳細や住所などの個人レコードの更新も、1 つのドキュメントに対する 1 つの書き込み操作です。

データを非正規化することにより、アプリケーションが一般的な操作を完了するために発行する必要があるクエリや更新の数は少なくなります。

###埋め込みをする場合

一般に、埋め込みデータ モデルを使用するのは次のような場合です。

- エンティティ間に **contains** リレーションシップがある場合。
- エンティティ間に **one-to-few** リレーションシップがある場合。
- **頻繁には変更されない**埋め込みデータがある場合。
- **際限なく**増大はしない埋め込みデータがある場合。
- ドキュメント内のデータに**不可欠な**埋め込みデータがある場合。

> [AZURE.NOTE]通常、非正規化データ モデルでは**読み取り**パフォーマンスが向上します。

###埋め込みをしない場合

ドキュメント データベースでよく使われる方法は、すべてを非正規化してすべてのデータを 1 つのドキュメントに埋め込むことですが、この方法でうまくいかない場合もあります。

この JSON スニペットを見てください。

	{
		"id": "1",
		"name": "What's new in the coolest Cloud",
		"summary": "A blog post by someone real famous",
		"comments": [
			{"id": 1, "author": "anon", "comment": "something useful, I'm sure"},
			{"id": 2, "author": "bob", "comment": "wisdom from the interwebs"},
			…
			{"id": 100001, "author": "jane", "comment": "and on we go ..."},
			…
			{"id": 1000000001, "author": "angry", "comment": "blah angry blah angry"},
			…
			{"id": ∞ + 1, "author": "bored", "comment": "oh man, will this ever end?"},
		]
	}

これは、典型的なブログや CMS システムをモデル化しようとしたときに、コメントが埋め込まれた post エンティティがどのようになるかを示したものです。この例の問題点は、コメントの配列が**無制限である**、つまり、1 つの投稿に対するコメントの数に実質的な制限がないということです。これにより、ドキュメントのサイズが著しく大きくなるにつれて、問題が発生します。

> [AZURE.TIP]DocumentDB 内のドキュメントには最大サイズがあります。詳細については、「[DocumentDB の制限](documentdb-limits.md)」を参照してください。

ドキュメントのサイズが増大すると、ネットワーク経由でデータを送信する機能は、ドキュメントの読み取りや更新と同様に、大きな影響を受けます。

このような場合は、次のモデルを検討することをお勧めします。
		
	Post document:
	{
		"id": 1,
		"name": "What's new in the coolest Cloud",
		"summary": "A blog post by someone real famous",
		"recentComments": [
			{"id": 1, "author": "anon", "comment": "something useful, I'm sure"},
			{"id": 2, "author": "bob", "comment": "wisdom from the interwebs"},
			{"id": 3, "author": "jane", "comment": "....."}
		]
	}

	Comment documents:
	{
		"postId": 1
		"comments": [
			{"id": 4, "author": "anon", "comment": "more goodness"},
			{"id": 5, "author": "bob", "comment": "tails from the field"},
			...
			{"id": 99, "author": "angry", "comment": "blah angry blah angry"}
		]
	},
	{
		"postId": 1
		"comments": [
			{"id": 100, "author": "anon", "comment": "yet more"},
			...
			{"id": 199, "author": "bored", "comment": "will this ever end?"}
		]
	}

このモデルは、最新の 3 つのコメントを投稿自体に埋め込むもので、この場合、配列の境界は固定です。その他のコメントは 100 個ずつグループ化され、個別のドキュメントに格納されます。グループ化するコメントのひとまとまりを 100 に指定したのは、この架空のアプリケーションが一度に読み込むことができるコメントの数が 100 個になっているためです。

データの埋め込みがうまく機能しない別の事例として、埋め込みデータがドキュメント間で頻繁に使用され、変更される場合があります。

この JSON スニペットを見てください。

	{
	    "id": "1",
	    "firstName": "Thomas",
	    "lastName": "Andersen",
	    "holdings": [
	        {
	            "numberHeld": 100,
	            "stock": { "symbol": "zaza", "open": 1, "high": 2, "low": 0.5 }
	        },
	        {
	            "numberHeld": 50,
	            "stock": { "symbol": "xcxc", "open": 89, "high": 93.24, "low": 88.87 }
	        }
	    ]
	}

これは、ある個人の株式ポートフォリオを表したものです。各ポートフォリオ ドキュメントに株式情報が埋め込まれています。株式取引アプリケーションのように関連データが頻繁に変更される環境では、埋め込みデータが頻繁に変更されるので、株が取引されるたびに各ポートフォリオ ドキュメントを絶えず更新しなければならないことになります。

株式 *zaza* は 1 日に数百回取引され、数千人ものユーザーのポートフォリオに *zaza* が含まれている可能性があります。上のようなデータ モデルでは、毎日、数千ものポートフォリオ ドキュメントを何回も更新する必要があり、システムの拡張が不十分になる可能性もあります。

##<a id="Refer"></a>データの参照##

このように、埋め込みデータは多くの場合うまく機能しますが、データの非正規化による効果よりも、引き起こされる問題の方が大きくなるシナリオがあることもまた事実です。その場合は、どうすればよいでしょうか。

エンティティ間のリレーションシップを作成できる場所は、リレーショナル データベースだけではありません。ドキュメント データベース内で、あるドキュメント内の情報と、他のドキュメント内のデータを実際に関連付けることができます。ここで、DocumentDB や他のドキュメント データベースにリレーショナル データベースに適したシステムを構築しようというわけではなく、適切で役立つのはシンプルなリレーションシップです。

以下の JSON では、前の株式ポートフォリオの例を使用していますが、ここでは株式に関する項目を埋め込むのではなく、ポートフォリオ上で参照しています。これにより、株式の項目が終日頻繁に変更された場合でも、更新する必要があるのは 1 つの株式ドキュメントのみです。

    Person document:
    {
        "id": "1",
        "firstName": "Thomas",
        "lastName": "Andersen",
        "holdings": [
            { "numberHeld":  100, "stockId": 1},
            { "numberHeld":  50, "stockId": 2}
        ]
    }
	
    Stock documents:
    {
        "id": 1,
        "symbol": "zaza",
        "open": 1,
        "high": 2,
        "low": 0.5,
        "vol": 11970000,
        "mkt-cap": 42000000,
        "pe": 5.89
    },
    {
        "id": 2,
        "symbol": "xcxc",
        "open": 89,
        "high": 93.24,
        "low": 88.87,
        "vol": 2970200,
        "mkt-cap": 1005000,
        "pe": 75.82
    }
    

ただし、この方法にも欠点があり、アプリケーションで個人のポートフォリオを表示する際、保有する各株式の情報も表示する必要がある場合には、何度もデータベースにアクセスして各株式ドキュメントの情報を読み取ってくることが必要になります。ここで、終日頻繁に発生する書き込み操作の効率を向上させ、この特定のシステムにはあまり影響がないと考えられる読み取り操作の効率を下げるという意思決定を行いました。

> [AZURE.NOTE]正規化されたデータ モデルは、サーバーに**より多くのラウンド トリップを要求する可能性があります**。

### 外部キー
現在は制約の概念がないため、ドキュメント内の外部キー、または何らかのドキュメント間リレーションシップは実際は "弱いリンク" であり、データベース自体によって検証されません。ドキュメントが参照しているデータが実際に存在していることを確認したい場合は、アプリケーション内で確認するか、または DocumentDB 上でサーバー側トリガーかストアド プロシージャを使用して確認する必要があります。

###参照を使用する場合
一般に、次のような場合に正規化されたデータ モデルを使用します。

- **一対多**リレーションシップを表している。
- **多対多**リレーションシップを表している。
- 関連するデータが**頻繁に変更される**。
- 参照されるデータに**制限がない**可能性がある。

> [AZURE.NOTE]通常は、正規化により**書き込み**パフォーマンスが向上します。

###リレーションシップの場所
リレーションシップの拡大により、どのドキュメントに参照を格納するかを決定しやすくなります。

発行元と書籍をモデル化する、以下の JSON の場合を見てみましょう。

	Publisher document:
	{
	    "id": "mspress",
	    "name": "Microsoft Press",
	    "books": [ 1, 2, 3, ..., 100, ..., 1000]
	}

	Book documents:
	{"id": 1, "name": "DocumentDB 101" }
	{"id": 2, "name": "DocumentDB for RDBMS Users" }
	{"id": 3, "name": "Taking over the world one JSON doc at a time" }
	...
	{"id": 100, "name": "Learn about Azure DocumentDB" }
	...
	{"id": 1000, "name": "Deep Dive in to DocumentDB" }

発行元あたりの書籍数が少なく、増加率も小さい場合は、書籍の参照を発行元ドキュメント内に格納することが有効な場合もあります。ただし、発行元あたりの書籍数に制限がない場合は、このデータ モデルは上の発行元ドキュメントに示すような、変更可能で拡大し続ける配列になります。

配置を少し入れ替えることにより、同じデータを表しながらも、このように大きく変更可能なコレクションにならないデータ モデルにすることもできます。

	Publisher document: 
	{
	    "id": "mspress",
	    "name": "Microsoft Press"
	}
	
	Book documents: 
	{"id": 1,"name": "DocumentDB 101", "pub-id": "mspress"}
	{"id": 2,"name": "DocumentDB for RDBMS Users", "pub-id": "mspress"}
	{"id": 3,"name": "Taking over the world one JSON doc at a time"}
	...
	{"id": 100,"name": "Learn about Azure DocumentDB", "pub-id": "mspress"}
	...
	{"id": 1000,"name": "Deep Dive in to DocumentDB", "pub-id": "mspress"}

上の例では、発行元ドキュメントで無制限のコレクションを使用していません。代わりに、各書籍ドキュメントに発行元への参照が含まれているだけです。

###多対多のリレーションシップのモデル化
リレーショナル データベースでは、*多対多*のリレーションシップは多くの場合、単に他のテーブルからのレコードを結合する結合テーブルを使用してモデル化されます。

![結合テーブル](./media/documentdb-modeling-data/join-table.png)

ドキュメントを使用して同じものを再現し、以下に示す例と同じようなデータ モデルを作成したいという気持ちになるかもしれません。

	Author documents: 
	{"id": 1, "name": "Thomas Andersen" }
	{"id": 2, "name": "William Wakefield" }
	
	Book documents:
	{"id": 1, "name": "DocumentDB 101" }
	{"id": 2, "name": "DocumentDB for RDBMS Users" }
	{"id": 3, "name": "Taking over the world one JSON doc at a time" }
	{"id": 4, "name": "Learn about Azure DocumentDB" }
	{"id": 5, "name": "Deep Dive in to DocumentDB" }
	
	Joining documents: 
	{"authorId": 1, "bookId": 1 }
	{"authorId": 2, "bookId": 1 }
	{"authorId": 1, "bookId": 2 }
	{"authorId": 1, "bookId": 3 }

これは、機能はします。ただし、作者とその著作、または書籍とその作者の読み込みは常に、少なくとも 2 つの余分なクエリをデータベースに対して実行することになります。1 つは結合するドキュメントに対するクエリ、もう 1 つは結合される実際のドキュメントを取得するクエリです。

この結合テーブルが実行しているのがデータの 2 つの要素をつなげることだけなら、これを削除してはどうでしょうか。 次の例を検討してみてください。

	Author documents:
	{"id": 1, "name": "Thomas Andersen", "books": [1, 2, 3]}
	{"id": 2, "name": "William Wakefield", "books": [1, 4]}
	
	Book documents: 
	{"id": 1, "name": "DocumentDB 101", "authors": [1, 2]}
	{"id": 2, "name": "DocumentDB for RDBMS Users", "authors": [1]}
	{"id": 3, "name": "Learn about Azure DocumentDB", "authors": [1]}
	{"id": 4, "name": "Deep Dive in to DocumentDB", "authors": [2]}

ここで、作者がわかっていれば、その著作はすぐにわかり、逆に書籍ドキュメントを読み込んでいれば、作者の ID がわかります。これにより、結合テーブルに対する中間クエリを省略し、アプリケーションが作成するサーバー ラウンド トリップの数を削減することができます。

##<a id="WrapUp"></a>ハイブリッド データ モデル##
ここまで、埋め込み (非正規化) データと参照 (正規化) データを見てきましたが、それぞれに長所と短所がありました。

常にどちらかである必要はなく、それぞれを少し併用してもかまいません。

アプリケーション固有の使用パターンと負荷に基づいて埋め込みデータと参照データを併用することが理にかなっていて、適切なパフォーマンス レベルを維持しながらアプリケーション ロジックをよりシンプルにし、サーバー ラウンド トリップを少なくする可能性があります。

以下の JSON を検討してみてください。

	Author documents: 
	{
	    "id": 1,
	    "firstName": "Thomas",
	    "lastName": "Andersen",		
	    "countOfBooks": 3,
	 	"books": [1, 2, 3],
		"images": [
			{"thumbnail": "http://....png"}
			{"profile": "http://....png"}
			{"large": "http://....png"}
		]
	},
	{
	    "id": 2,
	    "firstName": "William",
	    "lastName": "Wakefield",
	    "countOfBooks": 1,
		"books": [1, 4, 5],
		"images": [
			{"thumbnail": "http://....png"}
		]
	}
	
	Book documents:
	{
		"id": 1,
		"name": "DocumentDB 101",
		"authors": [
			{"id": 1, "name": "Thomas Andersen", "thumbnailUrl": "http://....png"},
			{"id": 2, "name": "William Wakefield", "thumbnailUrl": "http://....png"}
		]
	},
	{
		"id": 2,
		"name": "DocumentDB for RDBMS Users",
		"authors": [
			{"id": 1, "name": "Thomas Andersen", "thumbnailUrl": "http://....png"},
		]
	}

ここでは、(主に) 埋め込みモデルに従って、他のエンティティのデータを最上位のドキュメントに埋め込んでいますが、他のデータは参照しています。

書籍ドキュメントを見ると、作者の配列にいくつか興味深いフィールドがあります。作者ドキュメントに戻って参照するために使用する *id* フィールドがあり、これは正規化モデルの標準的な技法ですが、*name* フィールドと *thumbnailUrl* フィールドもあります。変わらずに *id* を保持し、アプリケーションが必要とする追加情報があればそれぞれの作者ドキュメントから "リンク" を使用して取得していますが、このアプリケーションでは各書籍に作者名とサムネイル写真を表示しているため、作者の**一部の**データを非正規化することにより、リスト内の書籍ごとのサーバー ラウンド トリップを減らすことができます。

確かに、作者名が変更されたり、写真が更新されたりした場合には、これまでに発行されたすべての書籍に対して更新を行う必要がありますが、このアプリケーションでは作者名はそう頻繁には変更されないという前提に基づいて、この設計が仕様を満たしていると判断しています。

この例には**事前に計算された集計**の値があり、読み取り操作の処理時間を削減しています。この例の作者ドキュメントに埋め込まれたデータの一部は、実行時に計算されるデータです。新しい書籍が発行されるたびに、書籍ドキュメントが作成され、**さらに**その作者に対応する書籍ドキュメントの数に基づいて計算された値が countOfBooks フィールドに設定されます。この最適化は、読み取りの負荷が高く、読み取りを最適化するために書き込み時に計算を実行する余裕のあるシステムに適しています。

モデルに事前計算フィールドを含める機能は、DocumentDB が**マルチドキュメント トランザクション**をサポートしていることにより実現されています。NoSQL ストアの多くはドキュメント間のトランザクションを実行できず、この制限によって "常にすべてを埋め込む" などの設計を取り入れています。DocumentDB では、サーバー側トリガーやストアド プロシージャを使用して、ACID トランザクション内で書籍の挿入や作者の更新をすべて実行できます。これで、データの整合性を維持するためだけに、1 つのドキュメントにすべてを埋め込む必要は**なくなります**。

##<a name="NextSteps"></a>次のステップ

この記事からの最大の収穫は、スキーマのない世界でのデータのモデル化は、依然として重要であると理解することです。

データの要素を画面上に表現する単一の方法がないのと同様に、データをモデル化する単一の方法もありません。使用するアプリケーションを理解し、アプリケーションがどのようにデータを作成、使用、処理するかを理解することが必要です。次に、ここで示したガイドラインのいくつかを適用することにより、アプリケーションの当面のニーズに対応するモデルの作成を開始することができます。アプリケーションの変更が必要な場合にも、スキーマのないデータベースの柔軟性を活用して、その変更を受け入れ、データ モデルを容易に進化させることができます。

Azure DocumentDB の詳細については、サービスの[ドキュメント](../../services/documentdb/)のページを参照してください。

Azure DocumentDB でのインデックスのチューニングの詳細については、[インデックス作成ポリシー](documentdb-indexing-policies.md)に関する記事を参照してください。

複数のパーティション間でデータをシャーディングする方法については、「[DocumentDB でのデータのパーティション分割](documentdb-partition-data.md)」を参照してください。

最後に、マルチテナント アプリケーションのデータのモデル化とシャーディングのガイダンスについては、「[Scaling a Multi-Tenant Application with Azure DocumentDB](http://blogs.msdn.com/b/documentdb/archive/2014/12/03/scaling-a-multi-tenant-application-with-azure-documentdb.aspx)」(Azure DocumentDB でのマルチテナント アプリケーションの拡張) を参照してください。
 

<!---HONumber=Oct15_HO3-->