<properties 
	pageTitle="DocumentDB のリソース モデルと概念 | Azure" 
	description="Microsoft Azure DocumentDB は完全に管理された NoSQL ドキュメント データベースであり、データベース アカウント、データベース、コレクション、ストアド プロシージャ、トリガー、UDF、ドキュメント、添付ファイル、メディア、ユーザー、リソースの管理権限の階層モデルを使用します。"  
	services="documentdb" 
	documentationCenter="" 
	authors="mimig1" 
	manager="jhubbard" 
	editor="monicar"/>

<tags 
	ms.service="documentdb" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/03/2015" 
	ms.author="mimig"/>

#DocumentDB のリソース モデルと概念

DocumentDB が管理するデータベースのエンティティを**リソース**と呼びます。各リソースは、論理 URI によって一意に識別されます。リソースを対話的に扱う際は、標準の HTTP 動詞、要求/応答ヘッダー、ステータス コードを使用します。 

この記事を読むと、次の質問の答えを見つけることができます。

- DocumentDB リソースとは。
- DocumentDB リソースの階層とは。
- システム定義リソースとユーザー定義リソースの違いは。
- リソースを扱うにはどのようにするか。
- コレクションを使用するにはどのようにするか。
- ストアド プロシージャ、トリガー、およびユーザー定義関数を操作するにはどのようにするか。

##階層型リソース モデル
次の図に示したように、DocumentDB の**リソース モデル**は、データベース アカウントに従属する一連のリソースから成り、個々のリソースは、不変の論理 URI アドレスで参照することができます。このドキュメントでは、そうした一連のリソースを**フィード**と呼ぶことにします。 

>[AZURE.NOTE] RESTful なプログラミング モデルに加え、DocumentDB にはきわめて効率的な TCP プロトコルが用意されています。やはり RESTful な通信モデルを特徴としており、[.NET クライアント SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx) を通じて利用することができます。

![][1]  
**データベース アカウント下の階層型リソース モデル**   

リソースの操作を開始するには、Azure サブスクリプションを使用して [DocumentDB データベース アカウントを作成する](documentdb-create-account.md)必要があります。データベース アカウントは、一連の**データベース**から構成できます。それぞれのデータベースには、複数の**コレクション**が含まれており、さらに、それぞれのコレクションに、**ストアド プロシージャ、トリガー、UDF、ドキュメント**および関連する**添付ファイル**が含まれています。また、データベースには**ユーザー**が関連付けられ、それぞれのユーザーには、他のさまざまなコレクション、ストアド プロシージャ、トリガー、UDF、ドキュメント、添付ファイルにアクセスするための一連の**アクセス許可**が関連付けられます。データベース、ユーザー、アクセス許可、コレクションが、既知のスキーマを持ったシステム定義のリソースであるのに対し、ドキュメントと添付ファイルは、ユーザーが自由に定義できる JSON コンテンツを格納します。  

|リソース 	|説明
|-----------|-----------
|データベース アカウント	|データベース アカウントには、容量単位が少なくとも 1 つ関連付けられます。プロビジョニングされているドキュメント ストレージとスループット (一連のデータベースと BLOB ストレージ) は、容量単位によって表されます。データベース アカウントは、Azure サブスクリプションを使用して作成できます。
|データベース	|データベースは、コレクションに分割されたドキュメント ストレージの論理上のコンテナーです。ユーザーのコンテナーとしても使用されます。
|ユーザー	|アクセス許可の範囲や区分を決める論理上の名前空間です。 
|アクセス許可	|特定のリソースへのアクセスを許可するためにユーザーに関連付けられる承認トークンです。
|コレクション	|コレクションには、JSON ドキュメントのほか、関連する JavaScript アプリケーション ロジックが格納されます。
|ストアド プロシージャ	|JavaScript で記述されたアプリケーション ロジックです。コレクションに登録され、データベース エンジン内でひとまとまりの処理として実行されます。
|トリガー	|挿入、置換、削除の各操作に伴って副次的に生じる作用をモデル化し、JavaScript で記述したアプリケーション ロジックです。
|UDF	|副次的な作用を持たないアプリケーション ロジックです。JavaScript で記述されます。UDF を使用してクエリ演算子をカスタマイズすることにより、DocumentDB のコア クエリ言語を拡張することができます。
|ドキュメント	|ユーザー定義 (ユーザーが自由に定義できる) JSON コンテンツです。既定では、スキーマを定義する必要や、コレクションに追加されたすべてのドキュメントに対してセカンダリ インデックスを用意する必要はありません。
|添付ファイル	|添付ファイルは、外部の BLOB またはメディアに対する参照や関連するメタデータを含んだ特殊なドキュメントです。開発者は、BLOB を DocumentDB の管理下におくか、外部の BLOB サービス プロバイダー (OneDrive、Dropbox など) を使って保存するかを選択できます。 


##システム定義リソースとユーザー定義リソース
データベース アカウント、データベース、コレクション、ユーザー、アクセス許可、ストアド プロシージャ、トリガー、UDF などのリソースには、いずれも一定のスキーマが存在し、それらを "システム リソース" といいます。これに対し、ドキュメントや添付ファイルなどのリソースには、スキーマに関する制約がありません。このようなリソースを "ユーザー定義リソース" といいます。DocumentDB では、システム定義リソースとユーザー定義リソースがどちらも、標準準拠の JSON として表現されて管理されます。システム定義であるかユーザー定義であるかに関係なく、すべてのリソースには共通するプロパティがあります。それらのプロパティを以下に示します。

>[AZURE.NOTE] リソースに対してシステムが生成するすべてのプロパティは、そのリソースの JSON 表現の中では、先頭にアンダースコア (_) が付きます。  


<table width="500"> 
<tbody>
<tr>
<td valign="top" ><p><b>プロパティ</b></p></td>
<td valign="top" ><p><b>ユーザー設定可能/システム生成</b></p></td>
<td valign="top" ><p><b>目的</b></p></td>
</tr>

<tr>
<td valign="top" ><p>_rid</p></td>
<td valign="top" ><p>システム生成</p></td>
<td valign="top" ><p>リソースに対してシステムが生成する一意かつ階層型の識別子</p></td>
</tr>

<tr>
<td valign="top" ><p>_etag</p></td>
<td valign="top" ><p>システム生成</p></td>
<td valign="top" ><p>オプティミスティック同時実行制御に必要なリソースの ETag</p></td>
</tr>

<tr>
<td valign="top" ><p>_ts</p></td>
<td valign="top" ><p>システム生成</p></td>
<td valign="top" ><p>リソースの最終更新時のタイムスタンプ</p></td>
</tr>

<tr>
<td valign="top" ><p>_self</p></td>
<td valign="top" ><p>システム生成</p></td>
<td valign="top" ><p>リソースに対するアドレス指定可能な一意の URI</p></td>
</tr>

<tr>
<td valign="top" ><p>id</p></td>
<td valign="top" ><p>ユーザー設定可能</p></td>
<td valign="top" ><p>リソースに対するユーザー定義の一意の名前</p></td>
</tr>

</tbody>
</table>  

###リソースの電子的表現
DocumentDB は、JSON 標準に対する独自の拡張や特殊なエンコードを必要としません。標準準拠の JSON ドキュメントで動作します。  
 
###リソースのアドレス指定
すべてのリソースは URI でアドレス指定されます。リソースの相対 URI は、その **_self** プロパティの値で表されます。URI の形式は、/<feed>/{_rid} パス セグメントから成ります。  

|_self の値	|説明
|-------------------|-----------
|/dbs	|データベース アカウント下のデータベースのフィード
|/dbs/{_rid-db}	|{_rid-db} の値を持つ一意の id プロパティを持つデータベース
|/dbs/{_rid-db}/colls/	|データベース下のコレクションのフィード
|/dbs/{_rid-db}/colls/{_rid-coll}	|{_rid-coll} の値を持つ一意の id プロパティを持つコレクション
|/dbs/{_rid-db}/users/	|データベース下のユーザーのフィード 
|/dbs/{_rid-db}/users/{_rid-user}	|{_rid-user} の値を持つ一意の id プロパティを持つユーザー
|/dbs/{_rid-db}/users/{_rid-user}/permissions	|データベース下のアクセス許可のフィード 
|/dbs/{_rid-db}/users/{_rid-user}/permissions/{_rid-permission}	|{_rid-permission} の値を持つ一意の id プロパティを持つアクセス許可  
  
リソースには、ユーザーによって定義される一意の名前も存在します。その名前は、リソースの id プロパティによって公開されます。ID は、特定の親リソースのコンテキストで一意となる最大 256 文字のユーザーが定義する文字列です。たとえば、特定のコレクションに含まれるいずれのドキュメントもその id プロパティの値が重複することはありませんが、その一意性が複数のコレクションにわたって保証されるわけではありません。同様に、アクセス許可の id プロパティの値が 1 人のユーザーの範囲内で重複することはありませんが、複数のユーザー間で重複する可能性はあります。_rid プロパティは、アドレス指定可能なリソースの _self link を構築する目的で使用されます。   

すべてのリソースには、システムによって生成される階層型のリソース識別子 (RID) も存在し、_rid プロパティでそれらにアクセスすることができます。RID は、特定のリソースの階層全体をコード化したものであり、分散型の手法で参照整合性を適用するための内部的な表現としてきわめて高い利便性を発揮します。RID は、データベース アカウント内で一意となっています。DocumentDB は、内部的に RID を使用することで、パーティションの境界を越えてルックアップする必要のない、効率的なルーティングを可能にしています。 

リソースの _self と _rid の値は、いずれも代替表現を持った正規のプロパティとして存在しています。  

##データベース アカウント
管理者は Azure サブスクリプションを使用して 1 つまたは複数の DocumentDB データベース アカウントをプロビジョニングできます。次の図のように、データベース アカウントには、SSD ベースのドキュメント ストレージとスループットから構成されるスタック可能な単位を容量単位 (CU) として購入できます。    

![][2] 

CU は、アプリケーションの規模やパフォーマンスのニーズに応じて段階的に増減できます。各 CU には、一連の柔軟なコレクション郡と、SSD ベースのプロビジョニング済みドキュメント ストレージおよびプロビジョニング済みスループットが用意されています。CU に関連付けられたプロビジョニング済みストレージおよびスループット容量は、データベース アカウントで各種のデータベースに作成した DocumentDB コレクション間で配分されます。データベース アカウントにプロビジョニングされた容量は、既に存在するかまたはそのアカウントに作成されるすべてのデータベースとコレクションで使用できます。データベース アカウントには、実際的な拡張の上限はありません。提供上の制約に従う限り、容量単位は経時的に任意の数を追加できます。CU 内で管理されるリソースは、可用性を高めるためにパーティション分割でスケールアウトされ、レプリケートされます。 

[DocumentDB データベース アカウントの作成および管理](documentdb-create-account.md)は、Azure のポータル ([http://portal.azure.com/](http://portal.azure.com/)) を通じて行うことができます。データベース アカウントを作成、管理するには管理アクセス権が必要で、Azure サブスクリプションの範囲内でのみ作成、管理ができます。 

###データベース アカウントのプロパティ
データベース アカウントのプロビジョニングと管理の一部として構成と読み取りが可能なプロパティは、以下のとおりです。  

<table border="1" cellspacing="0" cellpadding="0" > 
<tbody>
<tr>
<td valign="top" ><p><b>プロパティ名</b></p></td>
<td valign="top" ><p><b>説明</b></p></td>
</tr>

<tr>
<td valign="top" ><p>一貫性ポリシー</p></td>
<td valign="top" ><p>このプロパティは、データベース アカウントのすべてのコレクションに既定の一貫性レベルを構成するために設定します。[x-ms-consistency-level] 要求ヘッダーを使用すると、要求ベースごとに一貫性レベルをオーバーライドできます。今後、コレクション ベースでの一貫性レベルのオーバーライドがサポートされる可能性があります。 </p>

<p>このプロパティは、ユーザー定義リソースのみに <i>適用されます</i>。システム定義のすべてのリソースは、強力な一貫性の読み取りとクエリがサポートされるように構成されています。</p></td>
</tr>

<tr>
<td valign="top" ><p>プライマリ キーとセカンダリ キー</p></td>
<td valign="top" ><p>データベース アカウントのすべてのリソースへの管理アクセス権を提供する、プライマリとセカンダリのキー。</p></td>
</tr>

<tr>
<td valign="top" ><p>MaxMediaStorageUsageInMB (READ)</p></td>
<td valign="top" ><p>データベース アカウントで利用可能なメディア ストレージの最大量</p></td>
</tr>

<tr>
<td valign="top" ><p>CurrentMediaStorageUsageInMB (READ)</p></td>
<td valign="top" ><p>データベース アカウントのメディア ストレージの現在の使用状況</p></td>
</tr>

<tr>
<td valign="top" ><p>CapacityUnitsProvisioned</p></td>
<td valign="top" ><p>データベース アカウントでプロビジョニング済みの容量単位の合計数</p></td>
</tr>

<tr>
<td valign="top" ><p>CapacityUnitsConsumed (READ)</p></td>
<td valign="top" ><p>データベース アカウントで今までに使用された容量単位の合計数</p></td>
</tr>

<tr>
<td valign="top" ><p>ProvisionedDocumentStorageInMB (READ)</p></td>
<td valign="top" ><p>データベース アカウントのすべての容量単位でプロビジョニング済みのドキュメント ストレージの合計数 (単位: MB)</p></td>
</tr>

<tr>
<td valign="top" ><p>ReservedDocumentStorageInMB (READ)</p></td>
<td valign="top" ><p>データベース アカウントのすべての容量単位で予約されたドキュメント ストレージの合計数 (単位: MB)。ドキュメント ストレージは、コレクションにドキュメントが内包された時点で "予約された" と見なされます。 </p></td>
</tr>

<tr>
<td valign="top" ><p>ConsumedDocumentStorageInMB (READ)</p></td>
<td valign="top" ><p>データベース アカウントのすべての容量単位で実際に読み取られたドキュメント ストレージの合計数 (単位: MB)。これは、データベース アカウントのさまざまなコレクションにわたる実際のドキュメントとインデックスの合計サイズです。</p></td>
</tr>

</tbody>
</table>

Azure ポータルからデータベース アカウントをプロビジョニング、構成、管理するだけでなく、[Azure DocumentDB REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) を使用すると、[クライアント SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx) と DocumentDB データベース アカウントをプログラムで作成または管理できます。  

##データベース
DocumentDB データベースは、次の図に示すように、1 つ以上のコレクションとユーザーの論理コンテナーです。提供上の制約はありますが、DocumentDB データベース アカウントには任意の数のデータベースを作成できます。  

![][3]  
**データベースはユーザーとコレクションの論理コンテナーです**

データベースに含めることができる、コレクションごとにパーティション分割されるドキュメント ストレージ数は事実上無制限です。ここでは、内包されるドキュメントに対するトランザクション ドメインが構成されます。 

###DocumentDB データベースの柔軟な拡張
DocumentDB データベースは、既定で高い柔軟性を持ち、数 GB から数ペタバイトに上る SSD ベースのドキュメント ストレージとプロビジョニング済みスループットが用意されています。 

従来の RDBMS データベースとは異なり、DocumentDB のデータベースはその範囲が単一のコンピューターに制限されません。DocumentDB では、アプリケーションの規模を拡大することが必要になった時点で、さらに多くのコレクション、データベース、またはその両方を作成できます。実際、Microsoft 社内のさまざまなファースト パーティ アプリケーションでは、それぞれ数千個のコレクションと数テラバイトのドキュメント ストレージを含むきわめて大規模な DocumentDB データベースを作成することにより、コンシューマー規模で DocumentDB を使用しています。データベースは、アプリケーションの規模要件に合わせてコレクションを追加または削除することにより、拡大または縮小できます。 

データベース サブジェクト内では、提供状況と購入した容量単位の量に合わせて、任意の数のコレクションを作成できます。容量単位の購入を通じてプロビジョニングされた SSD ベースのストレージおよびスループットは、データベース アカウントのデータベースで複数のコレクションにわたって利用できます。   

DocumentDB データベースはユーザーのコンテナーでもあります。ユーザーは同様に、一連のアクセス許可に対する論理上の名前空間です。きめ細かな承認と、コレクション、ドキュメント、添付ファイルに対するアクセス権を提供します。  
 
DocumentDB リソース モデルの他のリソースと同様に、データベースも [Azure DocumentDB REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) や任意の[クライアント SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx) を使用すると、作成、置換、削除、読み取り、または列挙が簡単にできます。DocumentDB は、データベース リソースのメタデータの読み取りや照会で強力な一貫性を保証します。データベースを削除すると、含まれるすべてのコレクションやユーザーに自動的にアクセスできなくなります。DocumentDB では、削除されたデータベースの一部としてプロビジョニングされていたストレージとスループットをバックグラウンドで回収しますが、削除されたデータベースのプロビジョニング済みストレージとスループットは両方とも直ちに利用可能になります。   

##コレクション
DocumentDB コレクションは JSON ドキュメントのコンテナーです。また、コレクションはトランザクションとクエリのスケールの単位でもあります。コレクションを追加すると、DocumentDB データベースをスケールアウトできます。アプリケーションでスケールを拡大する必要がある場合は、SSD ベースのドキュメント ストレージ (とスループット) のプロビジョニングを増やして、データベース アカウントの 1 つ以上のコレクションに分散させることができます。
 
###弾力性のある SSD ベースのドキュメント ストレージ
コレクションは性質上弾力性があり、ドキュメントを追加または削除すると自動的に拡大または縮小します。DocumentDB を実行するファースト パーティ アプリケーションで、数ギガバイトから数テラバイト規模のデータベース内にある数千個のコレクションを使用してこの弾力性をテストしてきましたが、現在、DocumentDB の標準プレビュー機能では、指定のコレクションの弾力性を 10 GB に制限しています。 

###コレクションのインデックスの自動作成
DocumentDB は真にスキーマのないデータベース システムです。JSON ドキュメントには、前提となるスキーマはなく、どのようなスキーマも不要です。コレクションにドキュメントを追加するたびに、DocumentDB がこれらにインデックスを自動作成してクエリを利用可能にします。スキーマやセカンダリ インデックスが不要なドキュメントの自動インデックス作成は DocumentDB の中核機能であり、書き込みに対して最適化された、ロックフリーの、ログ構造化インデックス保守技法によって実現されています。DocumentDB は一貫性のあるクエリを提供する一方で、きわめて高速なボリューム書き込みもサポートします。ドキュメントとインデックスのストレージは両方とも、各コレクションに使用されるストレージの計算に使用されます。インデックス作成に関するストレージとパフォーマンス間のトレードオフは、コレクションに対するインデックス作成ポリシーを構成することで制御できます。 

###コレクションのインデックス作成ポリシーの構成
コレクションごとのインデックス作成ポリシーによって、インデックス作成に関するパフォーマンスとストレージのバランスを取ることができます。インデックス作成を構成する手順の一部として、以下のオプションがあります。  

-	コレクションの全ドキュメントにインデックスを自動作成するかどうかを選択します。既定では、すべてのドキュメントにインデックスが自動作成されます。インデックスを自動作成しない選択や、特定のドキュメントのみインデックス作成に追加する選択もできます。反対に、特定のドキュメントのみを削除する選択もできます。これは、コレクションの indexingPolicy の自動プロパティを true または false に設定し、ドキュメントの挿入、置換、または削除時に [x-ms-indexingdirective] 要求ヘッダーを使用することで実現できます。  
-	ドキュメントの特定のパスまたはパターンをインデックスから含めるか除外するかを選択します。これは、コレクションの indexingPolicy に includedPaths と excludedPaths のいずれかを設定することで実現できます。また、特定のパス パターンの範囲やハッシュ クエリに対してもストレージとパフォーマンスのバランスを構成できます。 
-	同期 (一貫した) または非同期 (遅延) のインデックス更新を選択します。既定では、コレクションに対するドキュメントの挿入、置換、削除のたびに、インデックスが同期的に更新されます。これにより、クエリには、ドキュメントの読み取りと同じ一貫性レベルが保証されます。DocumentDB は、書き込みを最適化し、同期インデックス メンテナンスによってドキュメントの持続的書き込みをサポートすると共に、クエリの一貫性を確保していますが、特定のコレクションのインデックスが遅れて更新されるように構成することもできます。インデックスの遅延作成は、書き込みのパフォーマンスを飛躍的に高める効果があり、コレクションで主に読み取りの負荷が激しい状況での一括インジェストに最適です。

DocumentDB のプレビュー リリースでは、インデックス作成ポリシーはコレクションの作成時のみ構成できます。コレクションの作成後は、ポリシーを更新できません。

###コレクションの照会
コレクション内のドキュメントは任意のスキーマを持つことができ、事前にスキーマやセカンダリ インデックスを提供しなくても、コレクション内でドキュメントを照会することができます。高度な階層および関係演算子と JavaScript ベースの UDF による拡張性を備えた [DocumentDB SQL クエリ言語](https://msdn.microsoft.com/library/azure/dn782250.aspx)を使用して、コレクションを照会できます。JSON の文法により、ツリー ノードのラベルを付けたツリーとして JSON ドキュメントをモデリングできます。これは DocumentDB の自動インデックス作成技法と DocumentDB で使用される SQL クエリ方言の両方で利用されています。DocumentDB クエリ言語の主な機能には以下の 3 つがあります。   

1.	階層型クエリやプロジェクションなどのツリー構造に自然にマップする少数のクエリ演算子のセット 
2.	コンポジション、フィルター、プロジェクション、集計、自己結合などの関係演算子のサブセット 
3.	(1) と (2) を使用する純粋な JavaScript ベースの UDF  

DocumentDB クエリ モデルは、機能、効率、簡潔さのバランスを取ることを試行します。DocumentDB のデータベース エンジンは、SQL クエリ ステートメントをネイティブでコンパイルして実行します。コレクションは [Azure DocumentDB REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) または任意の[クライアント SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx) で照会できます。.NET SDK は、LINQ プロバイダーと共に提供されます。今後のリリースでは、サーバー側のストアド プロシージャやトリガーだけでなく、クライアント側の [DocumentDB JavaScript SDK](https://github.com/Azure/azure-documentdb-js) からも使用できる、ネイティブのアンダースコア マッピングが提供される予定です。   

###マルチドキュメント トランザクション
データベース トランザクションは、データの同時変更に対応する、安全で予測可能なプログラミング モデルを提供します。RDBMS でビジネス ロジックを記述する従来の方法では、**ストアド プロシージャ**と**トリガー**のいずれかまたは両方を記述し、トランザクション実行のためにそれをデータベース サーバーに送信します。RDBMS では、アプリケーション プログラマーは、次の 2 種類の異なるプログラミング言語を扱う必要があります。 

- (非トランザクション) アプリケーション プログラミング言語 (例: JavaScript、Python、C#、Java など) 
- データベースによってネイティブで実行される、トランザクションのプログラミング言語である T-SQL

JavaScript と JSON がデータベース エンジン内で直接的に深く関与しているため、DocumentDB は、ストアド プロシージャとトリガーを使用した JavaScript ベースのアプリケーション ロジックを、コレクション上で直接実行するための直観的なプログラミング モデルを提供します。これにより、次の両方を実行できます。

- 同時実行の制御、復旧、データベース エンジンでの JSON オブジェクト グラフの直接の自動インデックス作成の効率的な実装 
- 制御フローの必然的な表現、変数のスコープ、例外処理の割り当ておよび統合、JavaScript プログラミング言語の観点からの直接のデータベース トランザクションとプリミティブの統合   

コレクション レベルで登録された JavaScript ロジックは、特定のコレクション内のドキュメントに対してデータベース操作を発行できるようになっています。DocumentDB は、周囲の ACID トランザクション内の JavaScript ベースのストアド プロシージャとトリガーを、コレクション内のドキュメントでのスナップショット分離によって暗黙的にラップします。その実行中に JavaScript で例外がスローされた場合、トランザクション全体が中止されます。結果のプログラミング モデルはきわめて簡潔でありながら強力なものになります。JavaScript 開発者は、使い慣れた言語コンストラクトとライブラリ プリミティブを引き続き利用しながら、"耐性のある" プログラミング モデルを手に入れることができます。   

バッファー プールと同じアドレス空間のデータベース エンジン内部で JavaScript を直接実行する機能により、コレクションのドキュメントに対してデータベース操作を高いパフォーマンスでトランザクション実行できます。加えて、DocumentDB データベース エンジンが JSON および JavaScript と深く関与していることから、アプリケーションとデータベースの型システムの不一致という障害が取り除かれます。   

コレクションの作成後、[Azure DocumentDB REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) または任意の[クライアント SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx) でストアド プロシージャ、トリガー、および UDF をコレクションに登録できます。登録が終了すると、これらを参照、実行できます。次に示すストアド プロシージャを検討してください。これは全体が JavaScript で記述され、2 つの引数 (書名と著者名) を持ちます。また、新しいドキュメントを作成、照会し、その後更新しますが、これらがすべて暗黙の ACID トランザクションで保護されています。実行中のいずれかの時点で JavaScript 例外がスローされた場合、トランザクション全体が中止されます。

	function businessLogic(name, author) {
	    var context = getContext();
	    var collectionManager = context.getCollection();        
	    var collectionLink = collectionManager.getSelfLink()
	        
	    // create a new document.
	    collectionManager.createDocument(collectionLink,
	        {id: name, author: author},
	        function(err, documentCreated) {
	            if(err) throw new Error(err.message);
	            
	            // filter documents by author
	            var filterQuery = "SELECT * from root r WHERE r.author = 'George R.'";
	            collectionManager.queryDocuments(collectionLink,
	                filterQuery,
	                function(err, matchingDocuments) {
	                    if(err) throw new Error(err.message);
	                    
	                    context.getResponse().setBody(matchingDocuments.length);
	                   
	                    // Replace the author name for all documents that satisfied the query.
	                    for (var i = 0; i < matchingDocuments.length; i++) {
	                        matchingDocuments[i].author = "George R. R. Martin";
	                        // we don't need to execute a callback because they are in parallel
	                        collectionManager.replaceDocument(matchingDocuments[i]._self,
	                            matchingDocuments[i]);   
	                    }
	                })
	        })
	};

クライアントは、HTTP POST 経由でトランザクションを実行するために、前の JavaScript ロジックをデータベースに送信できます。HTTP メソッドの使用に関する詳細については、「[DocumentDB リソースの RESTful な操作](documentdb-interactions-with-resources.md)」を参照してください。 

	client.createStoredProcedureAsync(collection._self, {id: "CRUDProc", body: businessLogic})
	   .then(function(createdStoredProcedure) {
	        return client.executeStoredProcedureAsync(createdStoredProcedure.resource._self,
	            "NoSQL Distilled",
	            "Martin Fowler");
	    })
	    .then(function(result) {
	        console.log(result);
	    },
	    function(error) {
	        console.log(error);
	    });


データベースはネイティブで JSON と JavaScript を認識するため、型システムの不一致は発生しません。つまり "OR マッピング" や特殊なコード生成手法は必要ありません。   

ストアド プロシージャとトリガーは、現在のコレクションのコンテキストを明示する、適切に定義されたオブジェクト モデルを通じて、コレクションやコレクション内のドキュメントと対話します。  

DocumentDB のコレクションは、[Azure DocumentDB REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) または任意の[クライアント SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx) を使用して、作成、削除、読み取り、または列挙が簡単にできます。DocumentDB は、コレクションのメタデータの読み取りや照会で強力な一貫性を常に提供します。コレクションを削除すると、含まれているドキュメント、添付ファイル、ストアド プロシージャ、トリガー、および UDF へのアクセスが自動的にできなくなります。DocumentDB では、削除されたコレクションの一部としてプロビジョニングされていたストレージとスループットをバックグラウンドで回収しますが、削除されたコレクションのプロビジョニング済みストレージとスループットは両方とも直ちに利用可能になります。   
 
##ストアド プロシージャ、トリガー、UDF
前のセクションで説明したように、実行するアプリケーション ロジックは、データベース エンジン内部のトランザクション内に直接記述できます。アプリケーション ロジック全体を JavaScript で記述し、ストアド プロシージャ、トリガーまたは UDF としてモデル化することができます。ストアド プロシージャまたはトリガー内の JavaScript コードは、コレクションの内部で、ドキュメントの挿入、置換、削除、読み取り、または照会ができます。一方、UDF 内の JavaScript は、クエリの結果セットのドキュメントの列挙による悪影響のない計算と、別の結果セットの作成のみができます。マルチテナントの DocumentDB では、厳密な予約ベースのリソース監視が施行されます。ストアド プロシージャ、トリガー、または UDF はいずれも、固定クォンタムのオペレーティング システムのリソースを入手して作業を実施します。さらに、ストアド プロシージャ、トリガー、または UDF は、外部の JavaScript ライブラリへの関連付けはできず、割り当てられたリソースの予算が超過するとブラックリストに掲載されます。ストアド プロシージャ、トリガー、または UDF は、REST API を使用してコレクションに登録と登録解除ができます。ストアド プロシージャ、トリガー、UDF を登録すると、これらはプリコンパイルされ、後で実行するためにバイト コードで格納されます。次のセクションでは、DocumentDB JavaScript SDK を使用して、ストアド プロシージャ、トリガー、UDF を登録、実行、登録解除する方法について説明します。JavaScript SDK は、[DocumentDB REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) のシンプルなラッパーです。 

###ストアド プロシージャの登録
ストアド プロシージャの登録により、新しいストアド プロシージャのリソースがコレクションに HTTP POST を介して作成されます。  

	var storedProc = {
	    id: "validateAndCreate",
	    body: function (documentToCreate) {
	        documentToCreate.id = documentToCreate.id.toUpperCase();
	        
	        var collectionManager = getContext().getCollection();
	        collectionManager.createDocument(collectionManager.getSelfLink(),
	            documentToCreate,
	            function(err, documentCreated) {
	                if(err) throw new Error('Error while creating document: ' + err.message;
	                getContext().getResponse().setBody('success - created ' + 
	                        documentCreated.name);
	            });
	    }
	};
	
	client.createStoredProcedureAsync(collection._self, storedProc)
	    .then(function (createdStoredProcedure) {
	        console.log("Successfully created stored procedure");
	    }, function(error) {
	        console.log("Error");
	    });

###ストアド プロシージャの実行
ストアド プロシージャは、既存のストアド プロシージャのリソースに対して HTTP POST を発行すると実行されます。その際、要求の本文でストアド プロシージャにパラメーターを渡します。

	var inputDocument = {id : "document1", author: "G. G. Marquez"};
	client.executeStoredProcedureAsync(createdStoredProcedure.resource._self, inputDocument)
	    .then(function(executionResult) {
	        assert.equal(executionResult, "success - created DOCUMENT1");
	    }, function(error) {
	        console.log("Error");
	    });

###ストアド プロシージャの登録解除
ストアド プロシージャは、既存のストアド プロシージャのリソースに HTTP DELETE を発行するだけで登録解除されます。   

	client.deleteStoredProcedureAsync(createdStoredProcedure.resource._self)
	    .then(function (response) {
	        return;
	    }, function(error) {
	        console.log("Error");
	    });


###プリトリガーの登録
HTTP POST を介して新しいトリガーのリソースをコレクションに作成すると、トリガーが登録されます。トリガーの種類 (プリトリガーかポストトリガーか)、および関連付け可能な操作の種類 (作成、置換、削除、すべてなど) を指定できます。   

	var preTrigger = {
	    id: "upperCaseId",
	    body: function() {
	            var item = getContext().getRequest().getBody();
	            item.id = item.id.toUpperCase();
	            getContext().getRequest().setBody(item);
	    },
	    triggerType: TriggerType.Pre,
	    triggerOperation: TriggerOperation.All
	}
	
	client.createTriggerAsync(collection._self, preTrigger)
	    .then(function (createdPreTrigger) {
	        console.log("Successfully created trigger");
	    }, function(error) {
	        console.log("Error");
	    });

###プリトリガーの実行
トリガーは、要求ヘッダーを通じてドキュメント リソースの POST/PUT/DELETE 要求を発行するときに、既存のトリガーの名前を指定することで実行されます。  
 
	client.createDocumentAsync(collection._self, { id: "doc1", key: "Love in the Time of Cholera" }, { preTriggerInclude: "upperCaseId" })
	    .then(function(createdDocument) {
	        assert.equal(createdDocument.resource.id, "DOC1");
	    }, function(error) {
	        console.log("Error");
	    });

###プリトリガーの登録解除
トリガーは、既存のトリガーのリソースに HTTP DELETE を発行するだけで登録解除されます。  

	client.deleteTriggerAsync(createdPreTrigger._self);
	    .then(function(response) {
	        return;
	    }, function(error) {
	        console.log("Error");
	    });

###UDF の登録
HTTP POST を介して新しい UDF のリソースをコレクションに作成すると、UDF が登録されます。  

	var udf = { 
	    id: "mathSqrt",
	    body: function(number) {
	            return Math.sqrt(number);
	    },
	};
	client.createUserDefinedFunctionAsync(collection._self, udf)
	    .then(function (createdUdf) {
	        console.log("Successfully created stored procedure");
	    }, function(error) {
	        console.log("Error");
	    });

###クエリの一部としての UDF の実行
UDF は SQL クエリの一部として指定でき、DocumentDB のコア [SQL クエリ言語](https://msdn.microsoft.com/library/azure/dn782250.aspx)を拡張する手段として使用されます。   

	var filterQuery = "SELECT mathSqrt(r.Age) AS sqrtAge FROM root r WHERE r.FirstName='John'";
	client.queryDocuments(collection._self, filterQuery).toArrayAsync();
	    .then(function(queryResponse) {
	        var queryResponseDocuments = queryResponse.feed;
	    }, function(error) {
	        console.log("Error");
	    });

###UDF の登録解除 
UDF は、既存の UDF のリソースに HTTP DELETE を発行するだけで登録解除されます。  

	client.deleteUserDefinedFunctionAsync(createdUdf._self)
	    .then(function(response) {
	        return;
	    }, function(error) {
	        console.log("Error");
	    });

上記では、[DocumentDB JavaScript SDK](https://github.com/Azure/azure-documentdb-js) を使用した登録 (POST)、登録解除 (PUT)、読み取りと一覧表示 (GET)、実行 (POST) のためのスニペットを示していますが、[REST API](https://msdn.microsoft.com/library/azure/dn781481.aspx) や他の[クライアントの SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx) も使用できます。 

##ドキュメント
コレクションでは、任意の JSON ドキュメントを挿入、置換、削除、読み取り、列挙、照会することができます。DocumentDB では、スキーマやセカンダリ インデックスを必要とせずに、コレクションのドキュメントへのクエリをサポートできます。   

真にオープンなデータベース サービスである DocumentDB では、JSON ドキュメントのために特殊なデータ型 (日時など) や固有のエンコードなどを開発することはしません。DocumentDB は、多様なドキュメント間の関係を記述するために特殊な JSON の規則を必要としません。DocumentDB の SQL クエリ言語は、きわめて強力な階層型のリレーショナル クエリ演算子をドキュメントの照会とプロジェクションのために提供します。ドキュメント間の関係を記述するために特殊な注釈や特別なプロパティを使用する必要はありません。  
 
他のすべてのリソースと同様に、ドキュメントも REST API または任意のクライアント SDK を使用すると、作成、置換、削除、読み取り、列挙、照会が簡単にできます。ドキュメントを削除すると、入れ子になったすべての添付ファイルに該当するクォータがすぐに解放されます。ドキュメントの読み取りの一貫性レベルは、データベース アカウントの一貫性ポリシーに従います。このポリシーは、使用するアプリケーションのデータの一貫性の要件に合わせて、要求ごとにオーバーライドできます。ドキュメント照会時の読み取りの一貫性は、コレクションに設定されたインデックス作成モードに従います。"一貫性" に関しては、アカウントの一貫性ポリシーに従います。 

##添付ファイルとメディア
バイナリの BLOB やメディアは DocumentDB に格納することも、リモート メディア ストアに格納することもできます。さらに、メディアのメタデータを、添付ファイルと呼ばれる特殊なドキュメントで表すこともできます。DocumentDB の添付ファイルは、別の場所に格納されたメディアや BLOB を参照する特殊な (JSON) ドキュメントです。添付ファイルは、リモート メディア ストレージに格納されたメディアのメタデータ (場所や著者など) をキャプチャする特殊なドキュメントにすぎません。 

DocumentDB を使用して、指定のユーザーの e ブックに関連したインク注釈、コメントなどのメタデータ、ハイライト、ブックマーク、評価、嗜好などを格納するソーシャル読み取りアプリケーションを考えてください。   

-	ブックそれ自体のコンテンツは、DocumentDB データベース アカウントまたはリモート メディア ストアの一部として利用可能なメディア ストレージに格納されます。 
-	アプリケーションは、各ユーザーのメタデータを個別のドキュメントとして格納できます。つまり、book1 についての Joe のメタデータは、/colls/joe/docs/book1 で参照されるドキュメントに格納されます。 
-	指定のブックのコンテンツのページをポイントする添付ファイルは、対応する /colls/joe/docs/book1/chapter1、/colls/joe/docs/book1/chapter2 などのドキュメントに格納されます。 

この例では、リソースの階層を伝えるのにわかりやすい ID を使用しています。リソースは一意のリソース ID を使用した REST API を通じてアクセスされます。 

DocumentDB で管理されるメディアでは、添付ファイルの _media プロパティでメディアの URI が参照されます。DocumentDB は、未処理の参照がすべてドロップされると、メディアのガベージ コレクションを実施します。新しいメディアをアップロードし、その新たに追加されたメディアをポイントする _media を設定すると、DocumentDB はその添付ファイルを自動作成します。管理するリモート BLOB ストア (OneDrive、Azure ストレージ、DropBox など) にメディアを格納する場合は、メディアを参照するために添付ファイルを引き続き使用できます。この場合は、自分自身で添付ファイルを作成し、添付ファイルの _media プロパティを設定してください。   

他のすべてのリソースと同様、添付ファイルも REST API または任意のクライアント SDK を使用すると、作成、置換、削除、読み取り、列挙が簡単にできます。ドキュメント同様、添付ファイルの読み取りの一貫性レベルは、データベース アカウントの一貫性ポリシーに従います。このポリシーは、使用するアプリケーションのデータの一貫性の要件に合わせて、要求ごとにオーバーライドできます。添付ファイルの照会時の読み取りの一貫性は、コレクションに設定されたインデックス作成モードに従います。"一貫性" に関しては、アカウントの一貫性ポリシーに従います。 
 
##ユーザー
DocumentDB ユーザーは、アクセス許可をグループ化するための論理上の名前空間を表します。DocumentDB ユーザーは、ID 管理システムのユーザーまたは事前定義されたアプリケーションのロールに対応付けることができます。DocumentDB では、ユーザーは、データベースのアクセス許可のセットをグループ化するための抽象化を表しているにすぎません。   

マルチテナントをアプリケーションに実装するために、アプリケーションの実際のユーザーまたはテナントに対応するユーザーを DocumentDB に作成することができます。その後、さまざまなコレクション、ドキュメント、添付ファイルなどに対するアクセス制御に対応したアクセス許可を指定のユーザーに作成できます。   

ユーザーの増加に合わせたアプリケーションの拡大が必要になった場合は、データを共有するさまざまな方法を適用できます。各ユーザーは、以下のようにモデル化できます。   

-	各ユーザーをデータベースにマップ
-	各ユーザーをコレクションにマップ 
-	複数のユーザーに対応するドキュメントを 1 つの専用のコレクションに 
-	複数のユーザーに対応するドキュメントをコレクションのセットに   

選択している特定のシャーディング戦略にかかわらず、実際のユーザーを DocumentDB データベースのユーザーとしてモデル化し、ユーザーごとに詳細なアクセス許可を関連付けることができます。  

![][4]  
**シャーディング戦略とユーザーのモデル化**

他のすべてのリソースと同様、DocumentDB のユーザーも REST API または任意のクライアント SDK を使用すると、作成、置換、削除、読み取り、列挙が簡単にできます。DocumentDB は、ユーザー リソースのメタデータの読み取りや照会で強力な一貫性を常に提供します。重要なこととして、ユーザーを削除すると、ユーザー内に含まれていたすべてのアクセス許可に自動的にアクセスできなくなります。DocumentDB では、削除されたユーザーの一部であったアクセス許可のクォータをバックグラウンドで回収しますが、削除されたアクセス許可は再び直ちに使用可能になります。  

##アクセス許可
アクセス制御の観点から、データベース アカウント、データベース、ユーザー、アクセス許可などのリソースは、管理者によるアクセス許可が必要なことから、 *administrative*リソースと考えられています。一方、コレクション、ドキュメント、添付ファイル、ストアド プロシージャ、トリガー、UDF などのリソースは、指定のデータベースに範囲が限定され、 *application resources*と考えられています。承認モデルでは、この 2 種類のリソースとこれにアクセスするロール (管理者とユーザー) に対応する、2 種類の  *access keys*、 *master key*と *resource key*を定義しています。マスター キーは、データベース アカウントの一部で、データベース アカウントをプロビジョニングしている開発者 (または管理者) に提供されます。このマスター キーは管理者セマンティクスを持ち、管理リソースとアプリケーション リソースの両方にアクセスを承認するために使用できます。対照的に、リソース キーは細密なアクセス キーで、 *specific*アプリケーション リソースへのアクセスを許可します。このため、リソース キーは、データベースのユーザーと、そのユーザーが特定のリソース (コレクション、ドキュメント、添付ファイル、ストアド プロシージャ、トリガー、または UDF) に対して所有するアクセス許可との間の関係を捕捉します。   

リソース キーを取得する唯一の方法は、指定のユーザーにアクセス許可リソースを作成することです。アクセス許可を作成または取得するには、承認ヘッダーにマスター キーを記述する必要があることに注意してください。アクセス許可リソースは、リソース、そのアクセス権、ユーザーに結び付いています。アクセス許可リソースの作成後は、該当するリソースへのアクセス権を取得するためにユーザーに必要とされるのは、関連付けられたリソース キーを提示することだけです。したがって、リソース キーは、アクセス許可リソースを論理的かつコンパクトに表したものと考えることができます。  

他のすべてのリソースと同様、DocumentDB のアクセス許可も REST API または任意のクライアント SDK を使用すると、作成、置換、削除、読み取り、列挙が簡単にできます。DocumentDB は、アクセス許可のメタデータの読み取りや照会で強力な一貫性を常に提供します。 

##次のステップ
HTTP コマンドを使用したリソースの操作の詳細については、「[DocumentDB リソースの RESTful な操作](documentdb-interactions-with-resources.md)」を参照してください。


[1]: ./media/documentdb-resources/resources1.png
[2]: ./media/documentdb-resources/resources2.png
[3]: ./media/documentdb-resources/resources3.png
[4]: ./media/documentdb-resources/resources4.png

<!--HONumber=49--> 