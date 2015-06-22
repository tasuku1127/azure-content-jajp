<properties 
	pageTitle="Azure インフラストラクチャ サービス実装ガイドライン" 
	description="Azure インフラストラクチャ サービスでの IT ワークロードのデプロイに関する主要な設計と実装のガイドラインについて説明します。" 
	documentationCenter=""
	services="virtual-machines" 
	authors="JoeDavies-MSFT" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="virtual-machines" 
	ms.workload="infrastructure-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="05/05/2015" 
	ms.author="josephd"/>

# Azure インフラストラクチャ サービス実装ガイドライン
 
次のガイドラインは、Azure インフラストラクチャ サービスのほとんどの実装に関係のあるさまざまなリソースを決定するための、設計に関する重要な決定事項とタスクに関するものです。

Azure は、わずかな投資でソリューションの実装に対する具体的なアプローチをテストできる、概念実証の構成を実装するのに最適なプラットフォームです。ただし、概念実証の簡単な作業と、それよりはるかに困難で詳細な、IT ワークロードの全機能を備えた実稼働可能な実装の作業を、区別する必要があります。

このガイダンスでは、Azure で IT インフラストラクチャまたはワークロードを成功させるために計画が鍵となる多くの領域を明らかにします。また、Azure プラットフォームでのソリューションの実装に役立つ、必要なリソースの作成順序も示します。ある程度の柔軟性はありますが、この順序に従って計画と意思決定を行うことをお勧めします。

1.	名前付け規則
2.	サブスクリプションとアカウント
3.	ストレージ
4.	仮想ネットワーク
5.	サブスクリプションあたりの
6.	可用性セット
7.	仮想マシン

適切な名前付け規則を確立し、Azure での具体的かつ体系的なリソース作成順序に従うことにより、管理面での負担が大きく軽減され、実装プロジェクト成功の可能性が高くなります。

> [AZURE.NOTE]アフィニティ グループについては、その使用が廃止されたため説明しません。詳しくは、「[地域 VNet およびアフィニティ グループについて](https://msdn.microsoft.com/library/azure/jj156085.aspx)」をご覧ください。

## 1.名前付け規則

Azure で作成作業を始める前に、適切な名前付け規則を用意する必要があります。名前付け規則を設けるとすべてのリソースの名前が予測可能になり、これらのリソースの管理に関する負担が軽減されます。

組織全体に対する名前付け規則、または特定の Azure サブスクリプションまたはアカウントに対して定義されている名前付け規則のどちらに従うかを選択できます。組織内の個人が Azure リソースの作業をするときに暗黙のルールを設けるのは簡単ですが、チームで Azure のプロジェクトを作業する必要があるときに、そのようなモデルではうまく対応できません。

事前に名前付け規則をまとめておく必要があります。名前付け規則を構成する一連のルールに関していくつかの考慮事項があります。

### 接辞

Azure では、リソースを簡単に管理できるように、作成時の既定値が決められています。たとえば、新しいクラウド サービスの最初の仮想マシンを作成する場合、Azure 管理ポータルは、仮想マシンの新しいクラウド サービスの名前に、仮想マシンの名前を使おうとします。

したがって、リソースの種類を識別するために接辞が必要な種類を明らかにするのが得策です。さらに、接辞の位置を明確に指定します。

- 名前の先頭 (プレフィックス)
- 名前の末尾 (サフィックス)

たとえば、計算エンジンをホストするクラウド サービスに対して可能な 2 つの名前を次に示します。

- Svc CalculationEngine (プレフィックス)
- CalculationEngine-Svc (サフィックス)

接辞では、特定のリソースを説明するさまざまな特徴を示すことができます。一般的に使用される例を次に示します。

特徴 | 例 | メモ
--- | --- | ---
環境 | dev、stg、prod | 各環境の目的と名前によって決まります。
場所 | usw (米国西部)、use (米国東部 2) | データセンターのリージョンまたは組織のリージョンによって決まります。
Azure のコンポーネント、サービス、または製品 | Svc (クラウド サービス)、VNet (仮想ネットワーク) | リソースがサポートを提供する製品によって決まります。
役割 | sql、ora、sp、iis | 仮想マシンのロールによって決まります。
インスタンス | 01、02、03 など | 複数のインスタンスが存在する可能性のあるリソースの場合。たとえば、クラウド サービス内の負荷分散された Web サーバーなど。
		
名前付け規則を設けるときは、各リソースの種類に対してどの接辞を使用するか、およびその位置 (プレフィックスかサフィックスか) を、明確に規定する必要があります。

### 日付

多くの場合、リソースの名前から作成日がわかることが重要です。YYYYMMDD という日付形式をお勧めします。この形式を使用すると、完全な日付が記録されるだけでなく、名前の日付部分だけが異なる複数のリソースが、同時にアルファベット順と時系列順に並べ替えられます。

### リソースの名前付け
名前付け規則では、作成される各リソースに名前を割り当てる方法を定義するルールを用意するリソースの種類を、定義する必要がある。これらのルールをすべての種類のリソースに適用する必要があります。次に例を示します。

- 契約
- アカウント
- ストレージ アカウント
- 仮想ネットワーク
- サブネット
- 可用性セット
- クラウド サービス
- 仮想マシン
- エンドポイント
- ネットワーク セキュリティ グループ
- ロール

参照しているリソースを特定するのに十分な情報が提供されるよう、できるだけわかりやすい名前にする必要があります。

### コンピューター名

仮想マシンを作成するときは、仮想マシンの名前を指定する必要があります。Microsoft Azure は、仮想マシンの名前を Azure Virtual Machine のリソース名として使用します。Azure は、仮想マシンにインストールされるオペレーティング システムに対してコンピューター名と同じ名前を使用します。ただし、これらの名前は同じではない場合があります。

既にオペレーティング システムを含んでいる .VHD ファイルから仮想マシンが作成される場合、Microsoft Azure での仮想マシン名と仮想マシンの OS コンピューター名が異なる場合があります。このような状況は仮想マシンの管理が困難になるのでお勧めしません。常に、Azure Virtual Machine のリソース名が、その仮想マシンのオペレーティング システムに割り当てられたコンピューター名と同じになるようにします。

Azure Virtual Machine の名前と基になる OS のコンピューター名を同じにすることをお勧めします。このため、「[Microsoft NetBIOS コンピューターの名前付け規則](https://support.microsoft.com/kb/188997/)」で説明されている NetBIOS 名前付け規則に従います。

### ストレージ アカウント名

ストレージ アカウン名の管理には特別な規則があります。小文字と数字および割り当てられている名前をサービス (blob、table、queue) と連結したものだけをしようでき、既定のドメイン (core.windows.net) はグローバルに有効な一意の DNS 名である必要があります。たとえば、ストレージ アカウントが mystorageaccount であるとすると、以下の URL が有効な一意の DNS 名になる必要があります。

- mystorageaccount.blob.core.windows.net
- mystorageaccount.table.core.windows.net
- mystorageaccount.queue.core.windows.net

さらに、ストレージ アカウントではコンテナーを利用できます。これらは、「[コンテナー、BLOB、およびメタデータの名前付けおよび参照](https://msdn.microsoft.com/library/azure/dd135715.aspx)」で説明されている名前付け規則に従う必要があります。

### Azure の構成要素の名前

Azure の構成要素は通常は PaaS の機能を利用するアプリケーションである Azure が提供するアプリケーション レベル サービスですが、IaaS リソースは Azure SQL、Traffic Manager などを利用できます。

これらのサービスは、Azure で作成されて登録されるアーティファクトに依存します。これらについても名前付け規則で考慮する必要があります。

### 名前付け規則に関する実装ガイドラインのまとめ

決めること:

- Azure リソースの名前付け規則 

タスク:

- Azure リソースの接辞、階層、文字列値、およびその他のポリシーに関する名前付け規則を定義します。

## 2.サブスクリプションとアカウント

Azure を使用するには、1 つまたは複数の Azure サブスクリプションが必要です。クラウド サービスや仮想マシンなどのリソースは、サブスクリプションのコンテキストに存在します。

- 企業のお客様は、通常、エンタープライズ加入契約を使用します。これは、階層内の最上位のリソースであり、1 つまたは複数のアカウントに関連付けられています。 
- エンタープライズ加入契約を持たないコンシューマーおよび顧客の最上位のリソースはアカウントです。
- サブスクリプションはアカウントに関連付けられており、アカウントごとに 1 つまたは複数のサブスクリプションを使用できます。Azure はサブスクリプション レベルで課金情報を記録します。

アカウント/サブスクリプションの関係での 2 つの階層レベルの制限のため、アカウントとサブスクリプションの名前付け規則を課金のニーズに合わせることが重要です。たとえば、世界規模の企業が Azure を使用する場合、リージョンごとに 1 つのアカウントを使用し、リージョン レベルでサブスクリプションを管理できます。

![](./media/virtual-machines-infrastructure-services-implementation-guidelines/sub01.png)
  
たとえば、次のような構造を使用できます。

![](./media/virtual-machines-infrastructure-services-implementation-guidelines/sub02.png)
  
同じ例について、リージョンで特定のグループに複数のサブスクリプションを関連付ける場合は、名前付け規則に追加のアカウントまたはサブスクリプション名をエンコードする方法が組み込まれている必要があります。次の組織では、請求レポートの間にメッセージ請求データで新しいレベルの階層を生成できます。

![](./media/virtual-machines-infrastructure-services-implementation-guidelines/sub03.png)

組織は次のようになります。

![](./media/virtual-machines-infrastructure-services-implementation-guidelines/sub04.png)
 
マイクロソフトは、単一アカウントまたはエンタープライズ アグリーメントのすべてのアカウントについて、ダウンロード可能なファイルで詳細な課金を提供します。たとえば、Excel を使用してこのファイルを処理できます。このプロセスは、データを取り込み、階層の複数のレベルをエンコードするリソースを異なる列に分割し、ピボット テーブルまたは PowerPivot を使用して動的なレポート機能を提供します。

### サブスクリプションとアカウントに関する実装ガイドラインのまとめ

決めること:

- IT ワークロードやインフラストラクチャをホストするために必要なサブスクリプションとアカウントのセット

タスク:

- 名前付け規則を使用してサブスクリプションとアカウントのセットを作成します。

## 3.ストレージ

ストレージは、アプリケーション レベルのサービスを提供するだけでなく、仮想マシンをサポートするインフラストラクチャの一部でもあるため、Azure ソリューションの不可欠な部分です。
 
Azure では 2 種類のストレージを使用できます。Standard Storage では、BLOB ストレージ、テーブル ストレージ、キュー ストレージ、およびファイル ストレージにアクセスできます。Premium Storage は、AlwaysOn クラスター内の SQL Server などの高パフォーマンス アプリケーション用に設計されており、現在は Azure Virtual Machine ディスクのみをサポートします。

ストレージ アカウントはスケーラビリティ ターゲットと関連付けられています。Azure ストレージの現在の制限については、「[Microsoft Azure サブスクリプションとサービスの制限、クォータ、制約](../azure-subscription-service-limits.md#storage-limits)」をご覧ください。また、「[Azure ストレージのスケーラビリティおよびパフォーマンスのターゲット](../storage-scalability-targets.md)」もご覧ください。

Azure で作成される仮想マシンには、オペレーティング システム ディスク、一時ディスク、および 0 個以上のオプションのデータ ディスクが含まれます。オペレーティング システム ディスクとデータ ディスクは Azure BLOB ですが、一時ディスクはコンピューターが存在するノードのローカルなストレージによってサポートされます。このため、コンピューターのノードが変更されたことが認識されないとそのディスクのデータが失われる可能性があるため、一時ディスクはシステム リサイクルの間に保持する必要があるデータには適していません。一時ドライブには何も格納しないでください。

BLOB の最大サイズは 1024 GB で、それには VHD ファイルのメタデータ (フッター) が含まれるため、オペレーティング システム ディスクとデータ ディスクの最大サイズは 1023 GB です (1 GB は 10243 バイト)。Windows にディスク ストライピングを実装して、この制限を超えることができます。

### ストライピングされたディスク
1023 GB より大きいディスクを作成できるだけでなく、多くの場合、データ ディスクにストライピングを使用すると、複数の BLOB で単一ボリュームのストレージをバックアップできるので、パフォーマンスが向上します。これにより、単一ディスクのデータの読み書きに必要な I/O が並列化されます。

Azure では、仮想マシンのサイズにより、使用できるデータ ディスクの量と帯域幅が制限されます。詳細については、「[Azure の仮想マシンおよびクラウド サービスのサイズ](https://msdn.microsoft.com/library/azure/dn197896.aspx)」をご覧ください。

Azure データ ディスクにディスク ストライピングを使用する場合は、次のガイドラインを考慮してください。

- データ ディスクは常に最大サイズ (1023 GB) にする必要があります
- 仮想マシンのサイズで許可されている最大数のデータ ディスクをアタッチします
- 記憶域スペースの構成を使用します
- ストレージ ストライピングの構成を使用します
- Azure データ ディスクのキャッシュ オプションを使わないようにします (キャッシュ ポリシー = なし)

詳細については、「[Storage Spaces - Designing for Performance (記憶域スペース - パフォーマンスのための設計)](http://social.technet.microsoft.com/wiki/contents/articles/15200.storage-spaces-designing-for-performance.aspx)」を参照してください。

### 複数のストレージ アカウント

複数のストレージ アカウントを使用して多数の仮想マシンに関連付けられているディスクをバックアップすると、これらのディスクの集約された I/O は個々のストレージ アカウントに対するスケーラビリティ ターゲットより確実に小さくなります。

最初は、ストレージ アカウントごとに 1 つの仮想マシンをデプロイすることをお勧めします。

### ストレージ レイアウトの設計

これらの戦略を実装して優れたパフォーマンスの仮想マシン ディスク サブシステムを実現するため、IT ワークロードまたはインフラストラクチャは、通常、多くのストレージ アカウントを活用します。これらは多くの VHD BLOB をホストします。場合によっては、複数の BLOB が、仮想マシンの 1 つのボリュームと関連付けられます。

このような状況では管理タスクの複雑さが増します。基になるディスクや関連する VHD BLOB への適切な名前付けなど、ストレージに適切な戦略を設計することが重要です。

### ストレージに関する実装ガイドラインのまとめ

決めること:

- 500 TB を超えるディスクを作成するためにディスクのストライピングが必要か
- ワークロードに最適なパフォーマンスを実現するためにディスクのストライピングが必要か
- IT ワークロードやインフラストラクチャをホストするために必要なストレージ アカウントのセット

タスク:

- 名前付け規則を使用してストレージ アカウントのセットを作成します。Azure プレビュー ポータル、Azure 管理ポータル、または **New-AzureStorageAccount** PowerShell コマンドレットを使用できます。

## 4.クラウド サービス

クラウド サービスは、PaaS と IaaS の両方のサービスに関して、Azure の基本的な構成要素です。PaaS では、クラウド サービスはインスタンスが相互に通信できるロールの関連付けを表します。クラウド サービスは、パブリック仮想 IP (VIP) アドレスおよびロード バランサーに関連付けられており、インターネットからの着信トラフィックを受け取って、そのトラフィックを受信するように構成されているロールに負荷分散します。

IaaS の場合もクラウド サービスは同様の機能を提供しますが、ほとんどの場合、ロード バランサーの機能は、インターネットからそのクラウド サービス内の多くの仮想マシンの特定の TCP または UDP ポートにトラフィックを転送するために使われます。

Azure はディスクに対する既定の名前付け規則の一部としてクラウド サービスの名前を使用するため、IaaS ではクラウド サービス名が特に重要です。クラウド サービスの名前には、文字、数字、ハイフンのみを含めることができます。フィールドの先頭と末尾の文字は、文字または数字としてください。

クラウド サービス名はドメイン "cloudapp.net" 内の VIP に関連付けられているため、Microsoft Azure はクラウド サービスの名前を公開します。アプリケーションのユーザー エクスペリエンスをよくするには、必要に応じて完全修飾クラウド サービス名の代わりにわかりやすい名前を使用する必要があります。これは通常、リソースのパブリック DNS 名 (例: www.contoso.com) を、リソースをホストするクラウド サービスの DNS 名 (例: www.contoso.com の Web サーバーをホストするクラウド サービス) にマップする、パブリック DNS の CNAME レコードを使用して行います。

さらに、クラウド サービスの名前は、Microsoft Azure テナントに関係なく、他のすべての Microsoft Azure Cloud Services の間で一意である必要があるため、クラウド サービスに使用する名前付け規則では例外を許容することが必要になる場合があります。

Azure サブスクリプションは、最大 200 個のクラウド サービスをサポートできます。

### クラウド サービスに関する実装ガイドラインのまとめ

決めること:

- IT ワークロードまたはインフラストラクチャをホストするために必要なクラウド サービスのセット 

タスク:

- 名前付け規則を使用してクラウド サービスのセットを作成します。Azure 管理ポータルまたは **New-AzureService** PowerShell コマンドレットを使用できます。

## 5.をクリックし、

次の論理ステップは、ソリューション内の仮想マシン間の通信をサポートするために必要な仮想ネットワークを作成することです。1 つのクラウド サービスだけで IT ワークロードの複数の仮想マシンをホストできますが、仮想ネットワークを使用することをお勧めします。

仮想ネットワークは、サブネット、カスタム アドレス指定、および DNS 構成オプションも指定できる、仮想マシンのコンテナーです。同じ仮想ネットワーク内の仮想マシンとコンピューターは、メンバーになっているクラウド サービスに関係なく、相互に直接通信できます。仮想ネットワーク内では、この通信はプライベートであり、パブリック エンドポイント経由で通信する必要はありません。仮想マシンが企業ネットワークに接続されている場合、この通信は、仮想ネットワークまたはオンプレミスにインストールされている DNS サーバーを使用して、IP アドレスまたは名前により実行できます。

### サイトの接続
オンプレミスのユーザーとコンピューターが、Azure Virtual Network 内の仮想マシンへの常時接続を必要としない場合は、クラウド専用の仮想ネットワークを作成します。

![](./media/virtual-machines-infrastructure-services-implementation-guidelines/vnet01.png)
 
これは通常、インターネット ベースの Web サーバーなど、インターネットに接続されたワークロード用です。これらの仮想マシンは、リモート デスクトップ接続、リモート PowerShell セッション、Secure Shell (SSH) 接続、およびポイント対サイト VPN 接続を使用して管理できます。

オンプレミス ネットワークに接続しないので、クラウド専用仮想ネットワークではプライベート IP アドレス空間の任意の部分を使用できます。

オンプレミスのユーザーとコンピューターが、Azure Virtual Network 内の仮想マシンへの常時接続を必要とする場合は、クロスプレミスの仮想ネットワークを作成し、ExpressRoute またはサイト間 VPN 接続を使用してオンプレミス ネットワークに接続します。

![](./media/virtual-machines-infrastructure-services-implementation-guidelines/vnet02.png)
 
この構成では、Azure Virtual Network は基本的に、オンプレミス ネットワークのクラウド ベースの拡張機能です。

オンプレミス ネットワークに接続するため、クロスプレミス仮想ネットワークは、組織によって使用されるアドレス空間の一意な一部分を使用する必要があり、ルーティング インフラストラクチャは、オンプレミス VPN デバイスに転送することによってその部分へのトラフィックのルーティングをサポートする必要があります。

クロスプレミス仮想ネットワークからオンプレミス ネットワークにパケットが移動できるようにするには、仮想ネットワークのローカル ネットワークの定義の一部として、一連の関連するオンプレミス アドレスのプレフィックスを構成する必要があります。仮想ネットワークのアドレス空間および関連するオンプレミスの場所によっては、ローカル ネットワークに多数のアドレス プレフィックスがあってもかまいません。

クラウド専用仮想ネットワーク間をクロスプレミス仮想ネットワークに変換できますが、通常は、仮想ネットワーク アドレス空間、サブネット、および Azure によって割り当てられた静的 IP アドレスの番号を再設定する必要があります。これは、動的 IP (DIP) と呼ばれます。したがって、作成する前に、必要な仮想ネットワークの種類 (クラウドのみか、クロスプレミスか) を慎重に検討する必要があります。

### サブネット
サブネットを使用すると、関連するリソースを論理的に (たとえば、同じアプリケーションに関連する仮想マシン用に 1 つのサブネット)、または物理的に (たとえば、クラウド サービスごとに 1 つのサブネット) まとめたり、サブネット分離手法を使用してセキュリティを強化したりできます。

クロスプレミス仮想ネットワークでは、オンプレミス リソースに対して使用するのと同じ規則でサブネットを設計する必要があります。**Azure は常にアドレス空間の最初の 3 つの IP アドレスを各サブネットに使用する**ことに注意してください。サブネットに必要なアドレスの数を決定するには、現在必要な仮想マシンの数を数え、将来の増加を予測した後、次の表を使用してサブネットのサイズを決定します。
 
必要な仮想マシンの数 | 必要なホスト ビットの数 | サブネットのサイズ 
--- | --- | --- 
1 ～ 3 | 3 | /29
4 ～ 11 | 4 | /28
12 ～ 27 | 5 | /27
28 ～ 59 | 6 | /26
60 ～ 123 | 7 | /25

> [AZURE.NOTE]通常のオンプレミス サブネットの場合、ホスト ビット数 n のサブネットに対するホスト アドレスの最大数は 2<sup>n</sup> – 2 です。Azure サブネットの場合、ホスト ビット数 n のサブネットに対するホスト アドレスの最大数は 2<sup>n</sup> – 5 (2 に加えて、Azure が各サブネットに使用するアドレス数の 3)。

選択したサブネットのサイズが小さすぎる場合、サブネット内の仮想マシンの番号を付け直して再デプロイする必要があります。

### 仮想ネットワークに関する実装ガイドラインのまとめ

決めること:

- IT ワークロードまたはインフラストラクチャをホストするために必要な仮想ネットワークの種類は (クラウドのみまたはクロスプレミス)。
- クロスプレミス仮想ネットワークの場合、現在のサブネットと仮想マシンをホストし、将来の妥当な拡張のために必要ななアドレス空間の大きさ。

タスク:

- 仮想ネットワークのアドレス空間を定義します。
- サブネットのセットおよびそれぞれに対するアドレス空間を定義します。
- クロスプレミス仮想ネットワークの場合、仮想ネットワーク内の仮想マシンが到達する必要のあるオンプレミスの場所に対するローカル ネットワーク アドレス空間のセットを定義します。
- 名前付け規則を使用して仮想ネットワークを作成します。Azure プレビュー ポータルまたは Azure 管理ポータルを使用できます。

## 6.可用性セット

Azure PaaS の場合、クラウド サービスにはアプリケーションのコードを実行する 1 つ以上のロールが含まれます。ロールは、ファブリックが自動的にプロビジョニングする 1 つまたは複数の仮想マシン インスタンスを持つことができます。Azure はいつでもこれらのロールのインスタンスを更新できますが、同じロールの一部であるため、ロールのサービスが停止するのを防ぐため、同時にすべてを更新してはならないことを Azure は認識しています。

Azure IaaS では、各 IaaS 仮想マシンが単一インスタンスのロールを表すため、ロールの概念は重要ではありません。関連する複数のマシンを同時に停止させないように Azure に示唆するため (たとえば、マシンが存在するノードの OS を更新するため)、可用性セットの概念が導入されました。可用性セットは、サービスの停止を防ぐため、同じ可用性セット内のすべてのマシンを同時に停止しないように Azure に指示します。可用性セットの仮想マシンのメンバーには、99.95% の稼働時間サービス レベル契約があります。

可用性セットは、ソリューションの高可用性計画の一部である必要があります。可用性セットは、1 つのクラウド サービス内で同じ可用性セット名を持つ仮想マシンのセットとして定義されます。クラウド サービスを作成した後で、可用性セットを作成できます。

### 可用性セットに関する実装ガイドラインのまとめ

決めること:

- IT ワークロードまたはインフラストラクチャ内のさまざまなロールおよび階層に必要な可用性セットの数

タスク:

- 名前付け規則を使用して一連の可用性セットを定義します。仮想マシンを作成するときに、または仮想マシンを作成した後で、仮想マシンを可用性セットに関連付けることができます。

## 7.仮想マシン

Azure PaaS では、Azure が仮想マシンとそれに関連付けられているディスクを管理します。ユーザーはクラウド サービスとロールを作成して名前を付ける必要があり、Azure はそれらのロールに関連付けられるインスタンスを作成します。Azure IaaS の場合は、クラウド サービス、仮想マシン、および関連付するディスクの名前はユーザーが指定する必要があります。

管理上の負担を軽減するため、Azure 管理ポータルでは、関連するクラウド サービスの既定の名前として、コンピューター名が提案されます (ユーザーが、仮想マシン作成ウィザードの一部として、新しいクラウド サービスを作成することを選択した場合)。

さらに、Azure は、クラウド サービス名、コンピューター名、および作成日の組み合わせを使用して、ディスクおよびそれがサポートする VHD BLOB の名前を指定します。

一般に、ディスクの数は仮想マシンの数よりはるかに多くなります。ディスクの孤立化を防ぐため、仮想マシンを操作するときは注意する必要があります。また、サポートしている BLOB を削除することなくディスクを削除できます。この場合、BLOB は手動で削除するまでストレージ アカウントに残っています。

### 仮想マシンに関する実装ガイドラインのまとめ

決めること:

- IT ワークロードまたはインフラストラクチャを提供するために必要な仮想マシンの数

タスク:

- 名前付け規則を使用して各仮想マシンの名前を定義します。
- Azure プレビュー ポータル、Azure 管理ポータル、または **New-AzureVM** PowerShell コマンドレットを使用して、仮想マシンを作成します。

## IT ワークロードの例: Contoso 金融分析エンジン

Contoso Corporation は、将来の市場取引を支援するために最先端の独自アルゴリズムで次世代の金融分析エンジンを開発しました。このエンジンを Azure 内のサーバーのセットとして顧客が利用できるようにしたいと考えています。次のもので構成されます。

- Web 層でカスタム web サービスを実行する 2 つの (最終的にはそれより多くの) IIS ベースの Web サーバー
- アプリケーション層で計算を実行する 2 つの (最終的にはそれより多くの) IIS ベースのアプリケーション サーバー
- データベース層に履歴データと現在の計算データを格納する、AlwaysOn 可用性グループ (2 つの SQL Server とマジョリティ ノード監視) を含む SQL Server 2014
- SQL Server のクラスター化に必要な、認証階層内の自己完結型フォレストとドメインのための 2 つの Active Directory ドメイン コントローラー
- すべてのサーバーは、2 つのサブネット上に存在します。Web サーバー用のフロントエンド サブネットと、アプリケーション サーバー、SQL Server 2014 クラスター、ドメイン コントローラー用のバックエンド サブネットです。

![](./media/virtual-machines-infrastructure-services-implementation-guidelines/example-tiers.png)
 
インターネット上の Contoso クライアントから着信するセキュリティで保護された Web トラフィックは、Web サーバー間に負荷分散する必要があります。Web サーバーからの HTTP 要求の形式での計算要求トラフィックは、アプリケーション サーバー間に負荷分散されます。さらに、エンジンは高可用性対応に設計する必要があります。

結果として得られる設計には、次のものが組み込まれる必要があります。

- Contoso の Azure サブスクリプションとアカウント
- ストレージ アカウント
- 2 つのサブネットを含む仮想ネットワーク
- 一連のクラウド サービス
- 似たロールを持つ一連のサーバーのための可用性セット
- 仮想マシン

これらすべてが、次のような Contoso の名前付け規則に従います。

- Contoso はプレフィックスとして、[IT ワークロード]-[場所]-[Azure リソース] を使用します。この例では、"azfae" (Azure Financial Analysis Engine) が IT ワークロード名であり、Contoso の最初の顧客のほとんどは米国東海岸にいるので "use" (East US 2) が場所です。
- ストレージ アカウントは、contosoazfaeusesa[説明] を使用します。contoso は一意性のためにプレフィックスに追加されており、ストレージ アカウント名はハイフンの使用をサポートしないことに注意してください。
- クラウド サービスは、contoso-azfae-use-cs-[説明] を使用します。contoso は一意性のためにプレフィックスに追加されていることに注意してください。
- 仮想ネットワークは、AZFAE-USE-VN[番号] を使用します。
- 可用性セットは、azfae-use-as-[ロール] を使用します。
- 仮想マシン名は、azfae-use-vm-[仮想マシン名] を使用します。

### Azure サブスクリプションとアカウント

Contoso は、Contoso Enterprise Subscription という名前のエンタープライズ サブスクリプションを使用して、この IT ワークロードに対する請求を提供します。

### ストレージ アカウント

Contoso は、2 つのストレージ アカウントが必要であると判断しました。

- **contosoazfaeusesawebapp** は、Web サーバー、アプリケーション サーバー、ドメイン コントローラーとそれらの追加データ ディスクの Standard Storage 用です
- **contosoazfaeusesasqlclust** は、SQL Server クラスターおよびそれらの追加データ ディスクの Premium Storage 用です

Contoso は、次の PowerShell コマンドで 2 つのストレージ アカウントを作成しました。

	New-AzureStorageAccount -StorageAccountName "contosoazfaeusesawebapp" -Location "East US 2"
	New-AzureStorageAccount -StorageAccountName "contosoazfaeusesasqlclust" -Location "East US 2" -Type Premium_LRS

### サブネットを含む仮想ネットワーク

仮想ネットワークは Contoso のオンプレミス ネットワークに常時接続している必要はないので、Contoso はクラウド専用の仮想ネットワークに決定しました。

Contoso は、Azure プレビュー ポータルを使用して次の設定でクラウド専用仮想ネットワークを作成しました。

- 名前: AZFAE-USE-VN01
- 場所: East US 2
- 仮想ネットワークのアドレス空間: 10.0.0.0/8
- 1 番目のサブネット:
	- 名前: FrontEnd
	- アドレス空間: 10.0.1.0/24
- 2 番目のサブネット:
	- 名前: BackEnd
	- アドレス空間: 10.0.2.0/24

### サブスクリプションあたりの

Contoso は次の 2 つのクラウド サービスを作成しました。

- **contoso-azfae-use-cs-frontend**: フロントエンド Web サーバー用
- **contoso-azfae-use-cs-backend**: バックエンド アプリケーション サーバー、SQL Server クラスター サーバー、ドメイン コントローラー用

Contoso は、次の PowerShell コマンドでクラウド サービスを作成しました。

	New-AzureService -Service "contoso-azfae-use-cs-frontend" -Location "East US 2"
	New-AzureService -Service "contoso-azfae-use-cs-backend" -Location "East US 2"

### 可用性セット

金融分析エンジンの 4 つの階層すべての高可用性を維持するため、Contoso は 4 つの可用性セットを決定しました。

- **azfae-use-as-dc**: ドメイン コントローラー用
- **azfae-use-as-web**: Web サーバー用
- **azfae-use-as-app**: アプリケーション サーバー用
- **azfae-use-as-sql**: SQL Server クラスター内のサーバー用

これらの可用性セットは、仮想マシンと共に作成されます。

### 仮想マシン

Contoso は、Azure Virtual Machines に対して次の名前を決定しました。

- **azfae-use-vm-dc01**: 1 番目のドメイン コントローラー用
- **azfae-use-vm-dc02**: 2 番目のドメイン コントローラー用
- **azfae-use-vm-web01**: 1 番目の Web サーバー用
- **azfae-use-vm-web02**: 2 番目の Web サーバー用
- **azfae-use-vm-app01**: 1 番目のアプリケーション サーバー用
- **azfae-use-vm-app02**: 2 番目のアプリケーション サーバー用
- **azfae-use-vm-sql01**: SQL Server クラスター内の 1 番目の SQL Server 用
- **azfae-use-vm-sql02**: SQL Server クラスター内の 2 番目の SQL Server 用
- **azfae-use-vm-sqlmn01**: SQL Server クラスター内のマジョリティ ノード監視用

完成すると次のような構成になります。

![](./media/virtual-machines-infrastructure-services-implementation-guidelines/example-config.png)
 
この構成には次のものが含まれます。

- クラウド専用仮想ネットワークと 2 つのサブネット (FrontEnd と BackEnd)
- 2 つのクラウド サービス
- 2 つのストレージ アカウント
- 金融分析エンジンの各階層に 1 つずつ、4 つの可用性セット
- 4 つの階層用の仮想マシン
- インターネットから Web サーバーへの HTTPS ベースの Web トラフィック用の外部負荷分散セット
- Web サーバーからアプリケーション サーバーへの暗号化されていない Web トラフィック用の内部負荷分散セット

以下の Azure PowerShell コマンドにより、前に作成されたストレージ アカウント、クラウド サービス、仮想ネットワークに対するこの構成での仮想マシンが作成されます。

	#Specify the storage account for the web and application servers
	Set-AzureSubscription –SubscriptionName "Contoso Enterprise Subscription" -CurrentStorageAccountName "contosoazfaeusesawebapp"
	
	#Specify the cloud service name for the web servers
	$ServiceName="contoso-azfae-use-cs-frontend"
	
	#Get the image string for the latest version of the Windows Server 2012 R2 Datacenter image in the gallery
	$image= Get-AzureVMImage | where { $_.ImageFamily -eq "Windows Server 2012 R2 Datacenter" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	
	#Create the first web server
	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the first web server."
	$vm1=New-AzureVMConfig -Name azfae-use-vm-web01 -InstanceSize large -ImageName $image -AvailabilitySetName azfae-use-as-web
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password 
	$vm1 | Set-AzureSubnet -SubnetNames FrontEnd
	$vm1 | Add-AzureEndpoint -Name Web1 -Protocol tcp -LocalPort 443 -PublicPort 443 -LBSetName "WebSet" -DefaultProbe
	New-AzureVM –ServiceName $ServiceName -VMs $vm1 -VNetName AZFAE-USE-VN01
	
	#Create the second web server 
	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the second web server."
	$vm1=New-AzureVMConfig -Name azfae-use-vm-web02 -InstanceSize Large -ImageName $image -AvailabilitySetName azfae-use-as-web
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password 
	$vm1 | Set-AzureSubnet -SubnetNames FrontEnd
	$vm1 | Add-AzureEndpoint -Name Web2 -Protocol tcp -LocalPort 443 -PublicPort 443 -LBSetName "WebSet" -DefaultProbe
	New-AzureVM –ServiceName $ServiceName -VMs $vm1 -VNetName AZFAE-USE-VN01
	
	#Specify the cloud service name for the application, SQL server, and authentication tiers
	$ServiceName="contoso-azfae-use-cs-backend"
	
	#Create the first domain controller server
	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the first domain controller server."
	$vm1=New-AzureVMConfig -Name azfae-use-vm-dc01 -InstanceSize Small -ImageName $image -AvailabilitySetName azfae-use-as-dc
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password 
	$vm1 | Set-AzureSubnet -SubnetNames BackEnd
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 100 -DiskLabel AppFiles –LUN 0 -HostCaching None
	New-AzureVM –ServiceName $ServiceName -VMs $vm1 -VNetName AZFAE-USE-VN01
	
	#Create the second domain controller server
	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the second domain controller server."
	$vm1=New-AzureVMConfig -Name azfae-use-vm-dc02 -InstanceSize Small -ImageName $image -AvailabilitySetName azfae-use-as-dc
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password 
	$vm1 | Set-AzureSubnet -SubnetNames BackEnd
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 100 -DiskLabel AppFiles –LUN 0 -HostCaching None
	New-	AzureVM –ServiceName $ServiceName -VMs $vm1 -VNetName AZFAE-USE-VN01
	
	#Create an internal load balancer instance for the application server tier 
	Add-AzureInternalLoadBalancer -ServiceName $ServiceName -InternalLoadBalancerName "AppTierILB" –SubnetName BackEnd –StaticVNetIPAddress 10.0.2.100
	
	#Create the first application server
	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the first application server."
	$vm1=New-AzureVMConfig -Name azfae-use-vm-app01 -InstanceSize Large -ImageName $image -AvailabilitySetName azfae-use-as-app
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password 
	$vm1 | Set-AzureSubnet -SubnetNames BackEnd
	$vm1 | Add-AzureEndpoint -Name App1 -Protocol tcp -LocalPort 80 -PublicPort 80 -LBSetName "AppSet" -InternalLoadBalancerName "AppTierILB" -DefaultProbe
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 500 -DiskLabel AppFiles –LUN 0 -HostCaching None
	New-	AzureVM –ServiceName $ServiceName -VMs $vm1 -VNetName AZFAE-USE-VN01
	
	#Create the second application server 
	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the second application server."
	$vm1=New-AzureVMConfig -Name azfae-use-vm-app02 -InstanceSize Large -ImageName $image -AvailabilitySetName azfae-use-as-app
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password 
	$vm1 | Add-AzureEndpoint -Name App2 -Protocol tcp -LocalPort 80 -PublicPort 80 -LBSetName "AppSet" -InternalLoadBalancerName "AppTierILB" -DefaultProbe
	$vm1 | Set-AzureSubnet -SubnetNames BackEnd
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 500 -DiskLabel AppFiles –LUN 0 -HostCaching None
	New-AzureVM –ServiceName $ServiceName -VMs $vm1 -VNetName AZFAE-USE-VN01
	
	#Specify the premium storage account for the SQL Server cluster
	Set-AzureSubscription –SubscriptionName "Contoso Enterprise Subscription" -CurrentStorageAccountName "contosoazfaeusesasqlclust"
	
	#Create the majority node witness server for the SQL Server cluster
	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the majority node witness server."
	$vm1=New-AzureVMConfig -Name azfae-use-vm-sqlmn01 -InstanceSize Medium -ImageName $image -AvailabilitySetName azfae-use-as-sql
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password 
	$vm1 | Set-AzureSubnet -SubnetNames BackEnd
	New-AzureVM –ServiceName $ServiceName -VMs $vm1 -VNetName AZFAE-USE-VN01
	
	#Change the image string for the latest version of the SQL Server 2014 image in the gallery
	$image= Get-AzureVMImage | where { $_.ImageFamily -eq "SQL Server 2014 RTM Standard on Windows Server 2012 R2" } | sort PublishedDate -Descending | select -ExpandProperty ImageName -First 1
	
	#Create the first SQL Server
	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the first SQL Server."
	$vm1=New-AzureVMConfig -Name azfae-use-vm-sql01 -InstanceSize A5 -ImageName $image  -AvailabilitySetName azfae-use-as-sql
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password
	$vm1 | Set-AzureSubnet -SubnetNames BackEnd
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1000 -DiskLabel SQLFiles –LUN 0 -HostCaching None
	New-AzureVM –ServiceName $ServiceName -VMs $vm1 -VNetName AZFAE-USE-VN01
	
	#Create the second SQL Server
	$cred1=Get-Credential –Message "Type the name and password of the local administrator account for the second SQL Server."
	$vm1=New-AzureVMConfig -Name azfae-use-vm-sql02 -InstanceSize A5 -ImageName $image  -AvailabilitySetName azfae-use-as-sql
	$vm1 | Add-AzureProvisioningConfig -AdminUsername $cred1.GetNetworkCredential().Username -Password $cred1.GetNetworkCredential().Password
	$vm1 | Set-AzureSubnet -SubnetNames BackEnd
	$vm1 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1000 -DiskLabel SQLFiles –LUN 0 -HostCaching None
	New-AzureVM –ServiceName $ServiceName -VMs $vm1 -VNetName AZFAE-USE-VN01

## その他のリソース

[Microsoft Azure サブスクリプションとサービスの制限、クォータ、制約](../azure-subscription-service-limits.md#storage-limits)

[Azure の仮想マシンおよびクラウド サービスのサイズ](https://msdn.microsoft.com/library/azure/dn197896.aspx)

[Azure Storage のスケーラビリティおよびパフォーマンスのターゲット](../storage-scalability-targets.md)

<!--HONumber=54--> 