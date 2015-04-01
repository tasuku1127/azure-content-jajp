﻿<properties      
    pageTitle="DocumentDB でのデータのパーティション分割 | Azure"      
    description="DocumentDB でのデータのパーティション分割の方法や、ハッシュ、範囲および検索パーティション分割をどのようなときに使用するかについて説明します。"          services="documentdb"      
    authors="arramac"      
    manager="johnmac"      
    editor="monicar"      
    documentationCenter=""/> 
<tags      
    ms.service="documentdb"      
    ms.workload="data-services"      
    ms.tgt_pltfrm="na"      
    ms.devlang="na"      
    ms.topic="article"      
    ms.date="02/20/2015"      
    ms.author="arramac"/> 

# DocumentDB でのデータのパーティション分割

[Microsoft Azure DocumentDB](../../services/documentdb/) は、高速で予測可能なパフォーマンスの実現を支援し、アプリケーションの規模の拡大に合わせてシームレスに *スケールアウト*するよう設計されています。DocumentDB は、Web およびモバイル アプリの MSN スイートを強化する User Data Store のように、Microsoft の大規模プロダクション サービスを強化するために利用されてきました。 

一般には**シャーディング**と呼ばれる概念であるデータの水平パーティション分割によって、DocumentDB アプリケーションのストレージとスループットに関しては、無限に近い拡張が実現できます。DocumentDB アカウントは、スタック可能な単位、別名**コレクション**による、コストに比例した拡張ができます。コレクション全体にわたるデータのパーティション分割をいかに適切に行うことができるかは、データの形式とアクセス パターンによって決まります。 

この記事を最後まで読むと、次に挙げる疑問への解答が得られます。   

 - ハッシュ、範囲および検索パーティション分割とは何か
 - 各パーティション分割手法はどのようなときに使用するのか、またその理由
 - パーティション分割済みのアプリケーションを Azure DocumentDB で構築するには、どうすればいいのか

## コレクション = パーティション

データのパーティション分割手法について詳しく説明する前に、コレクションとは何か、また何がコレクションではないのかを理解する必要があります。既にご存じのように、コレクションとは JSON ドキュメントのコンテナーです。DocumentDB におけるコレクションは、 *論理*コンテナーであると同時に、 *物理*コンテナーでもあります。これらは、ストアド プロシージャやトリガーにとってのトランザクション境界であり、クエリや CRUD 操作へのエントリ ポイントです。各コレクションは、同じアカウント内の他のコレクションとは共有されない、予約済みの量のスループットが割り当てられます。そのため、コレクションをさらに追加し、それにドキュメントを分配することで、ストレージとスループットの両方に関してアプリケーションをスケール アウトすることができます。

コレクションは、リレーショナル データベースにおけるテーブルとは異なります。コレクションは、スキーマを適用しません。そのため、さまざまなスキーマを持つ、異なる種類のドキュメントを同じコレクション内に格納できます。ただし、テーブルを扱う場合と同様、1 種類のオブジェクトの格納にコレクションを使用するという選択肢もあります。何が最善のモデルであるかは、クエリやトランザクションにおいて、データがどのような組み合わせで現れるかによって決まります。

## DocumentDB を使用したパーティション分割

Azure DocumentDB でのデータのパーティション分割に使用される最も一般的な手法は、 *範囲パーティション分割*、 *検索パーティション分割*、および *ハッシュ パーティション分割*です。通常、"timestamp" または "userID" のように、ドキュメント内の単一の JSON プロパティ名を、パーティション キーとして指定します。場合によっては、このキーが代わりに内部 JSON プロパティや、各種ドキュメントの異なるプロパティ名になります。

それでは、これらの手法について、もう少し詳しく見ていきましょう。

## 範囲パーティション分割

範囲パーティション分割では、パーティション キーが特定の範囲内にあるかどうかに基づいてパーティションが割り当てられます。この手法は、 *タイム スタンプ* プロパティ (たとえば、2015 年 2 月 1 日から 2015 年 2 月 2 日までの eventTime) を用いたパーティション分割に使用されるのが一般的です。 

> [AZURE.TIP] パーティション キーに対してクエリが特定の範囲の値に制限されている場合は、範囲パーティション分割の使用をお勧めします。

## 検索パーティション分割

検索パーティション分割では、個別のパーティション値を特定のパーティションに割り当てる検索マップ、別名パーティション マップまたはシャード マップに基づいてパーティションが割り当てられます。この手法は、地域ごと (たとえば、ノルウェー、デンマーク、スウェーデンを含むスカンジナビア地域についてのパーティション) のパーティション分割に使用されるのが一般的です。

> [AZURE.TIP] 検索パーティション分割では、マルチテナント アプリケーションの管理において最高水準の制御ができます。複数のテナントを 1 つのコレクションに割り当てることや、1 つのテナントを 1 つのコレクションに、または 1 つのテナントを複数のコレクションにわたって割り当てることもできます。 

## ハッシュ パーティション分割

ハッシュ パーティション分割では、ハッシュ関数の値に基づいてパーティションが割り当てられ、複数のパーティションにわたって要求やデータを均等に分配できます。この手法は、数多くの異なるクライアントによって生成または消費されるデータのパーティション分割に使用されるのが一般的で、ユーザー プロファイル、カタログ項目、IoT (Internet of Things: モノのインターネット) テレメトリ データなどの格納に便利です。 

> [AZURE.TIP] 検索パーティション分割によって列挙するにはエンティティの数が多すぎる場合や (ユーザーやデバイスなど)、要求レートがエンティティ間で比較的均一な場合は、常にハッシュ パーティション分割を使用することをお勧めします。

## 適切なパーティション分割手法を選択する

では、どのパーティション分割手法が自分にとって適切なのでしょうか。それは、データの種類と普段のアクセス パターンによって決まります。設計時に適切なパーティション分割手法を選ぶことによって、技術的負債を回避し、データ サイズや要求ボリュームの増大に対処することができます。

- **範囲パーティション分割**は、タイムスタンプによってパーティションをエージ アウトするという簡単かつ自然なメカニズムが利用できる点から、日付に関連して使用されるのが一般的です。また、クエリが通常はある時間範囲に制限されている場合、それがパーティション分割の境界と一致することになるので便利です。 
- **検索パーティション分割**では、順序付けも関連付けもされていないデータ セットについて、自然な形でグループ分けや整理ができます。たとえば、テナントを組織ごとにグループ分けする、州を地域ごとにグループ分けするなどです。また、検索パーティション分割は、コレクション間でのデータ移行について、きめ細かく制御することも可能です。 
- **ハッシュ パーティション分割**は、プロビジョニング済みのストレージおよびスループットを効率的に使用するため、要求への負荷分散を均等にするのに役立ちます。 *コンシステント ハッシュ* アルゴリズムによって、パーティションの追加または削除時に移動する必要があるデータの量を最小化できます。

選択するパーティション分割手法を 1 つだけに絞る必要はありません。状況によっては、これらを *複合*した手法が役に立つ場合もあります。たとえば、車両テレメトリ データを格納している場合、適切なアプローチとしては、まず管理しやすいデバイス テレメトリ データをタイムスタンプの範囲ごとにパーティション分割し、その後、スループットをスケールアウトするために VIN (車両識別番号) に対してサブパーティション分割を行います (範囲ハッシュ複合パーティション分割)。

## パーティション分割済みアプリケーションの開発
DocumentDB でパーティション分割済みアプリケーションを開発する場合、着目すべき 3 つの重要な設計領域があります。

- どのようにして作成および読み取り (クエリを含む) を適切なコレクションにルーティングするか
- パーティション解決構成、別名パーティション マップをどのようにして保存し、取得するか
- データや要求のボリュームが増大した場合、どのようにしてパーティションを追加または削除するか

それぞれの領域について詳しく見てみましょう。

## 作成およびクエリのルーティング

ドキュメント作成要求のルーティングは単純で、これは今までに説明した 3 つの手法すべてに共通します。ドキュメントは、パーティション キーに一致するハッシュ値、検索値、または範囲値のパーティション上に作成されます。

クエリや読み取りは、通常、パーティション キーが 1 つに制限されているため、クエリは一致するパーティションのみにファン アウトできます。ただし、すべてのデータに対するクエリの場合は、複数のパーティションにわたって要求を *ファンアウト*し、その後、結果をマージする必要があります。上位 N 個の結果をフェッチする際など、場合によってはカスタム ロジックを実行して結果をマージすることが必要なクエリもある点に注意してください。

## パーティション マップの管理

パーティション マップを格納する方法、クライアントがそれを読み込み、変更があった場合には更新情報を受け取る方法、さらには複数のクライアントの間でそれを共有する方法について決めておくことも必要です。パーティション マップの更新が頻繁ではない場合は、単にそれをアプリケーションの構成ファイルに保存することができます。 

それ以外の場合は、任意の永続ストアに保存することができます。実稼働環境でよく目にする設計パターンは、パーティション マップを JSON としてシリアル化し、それを DocumentDB のコレクション内にも格納するというものです。クライアントは、再度取得する手間を省くためにマップをキャッシュに格納し、その後は変更がないか定期的にポーリングします。クライアントがシャード マップを変更する可能性がある場合、そのクライアントが一貫性のある名前付けスキーマを使用しており、また、パーティション マップに対して一貫性のある更新ができるオプティミスティック同時実行制御 (ETag) を使用していることを確認してください。

## パーティションの追加と削除

DocumentDB を使用することで、いつでも作成したコレクションの追加や削除ができるほか、それを新しい受信データの格納や、既存のコレクション上で利用できるデータの再調整に使用できます。コレクションの数については、[制限][documentdb-limits]に関するページを確認してください。このページにある制限を引き上げる場合は、いつでも弊社にお問い合わせください。

検索および範囲パーティション分割での新しいパーティションの追加や削除は単純です。たとえば、最新データの新しい地域や新しい時間範囲を追加するために必要な操作は、パーティション マップに新しいパーティションを追加することだけです。既存のパーティションを複数のパーティションに分割する、または 2 つのパーティションをマージするには、さらに少しの操作が必要になります。必要な操作は、次のいずれかです。 

- 読み取りのためにシャードをオフラインにする
- 移行時に、古いパーティション分割構成と新しいパーティション分割構成の両方に読み取りをルーティングするトランザクションおよび整合性レベルの保証は、移行が完了するまで利用できませんので、注意してください。

ハッシュ パーティション分割の場合、パーティションの追加と削除は比較的複雑です。単純なハッシュ手法はシャッフリングを引き起こし、データの大部分を移動する必要が生じます。**コンシステント ハッシュ**を使用することで、ごく一部のデータを移動するだけで済むようになります。

データの移動を必要としない、比較的簡単な新しいパーティションの追加方法としては、データを新しいコレクションへと "溢れさせ" て、その後、古いコレクションと新しいコレクションの両方に要求をファンアウトするというものがあります。ただし、この方法はめったにない状況 (ピーク時のワークロードを溢れさせて、移動できるようになるまでデータを一時的に保持するなど) のみに、その使用を制限する必要があります。

### 次のステップ
この記事では、DocumentDB を使用したデータのパーティション分割の方法に関するいくつかの一般的な手法と、その手法または手法の組み合わせをどのようなときに使用するかについて紹介しました。[サポートされている SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx) のいずれかを使用し、疑問がある場合は [MSDN のサポート フォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=AzureDocumentDB)からお問い合わせください。


<!--HONumber=47-->