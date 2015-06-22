<properties 
	pageTitle="オンプレミスの VMM サイト間の保護の設定" 
	description="Azure Site Recovery は、オンプレミスの VMM サイト間で、Hyper-V 仮想マシンのレプリケーション、フェールオーバー、回復を調整します。." 
	services="site-recovery" 
	documentationCenter="" 
	authors="raynew" 
	manager="jwhit" 
	editor="tysonn"/>

<tags 
	ms.service="site-recovery" 
	ms.workload="backup-recovery" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/19/2015" 
	ms.author="raynew"/>

# オンプレミスの VMM サイト間の保護の設定


## 概要


Azure Site Recovery は、さまざまなデプロイ シナリオでの仮想マシンのレプリケーション、フェールオーバー、復旧を調整してビジネス継続性と障害復旧 (BCDR) 戦略に貢献します。デプロイ シナリオのすべての一覧については、[Azure Site Recovery の概要](../hyper-v-recovery-manager-overview.md)を参照してください。

このシナリオのガイドでは、Site Recovery をデプロイして、VMM プライベート クラウドに配置されている Hyper-V ホスト サーバー上の仮想マシンで実行されているワークロードの保護を調整および自動化する方法について説明します。このシナリオでは、仮想マシンはプライマリ VMM サイトからセカンダリ VMM サイトに Hyper-V Replica を使用してレプリケートされます。

ガイドには、シナリオの前提条件が含まれています。また、Site Recovery コンテナーを設定する方法、ソースとターゲットの VMM サーバーに Azure Site Recovery プロバイダーをインストールする方法、このコンテナーにサーバーを登録する方法、保護されるすべての仮想マシンに適用される VMM クラウドの保護設定を構成する方法、およびこれらの仮想マシンの保護を有効にする方法についても説明しています。すべてが正しく動作していることを確認するために、最後にフェールオーバーをテストします。

このシナリオの設定時に問題が発生した場合は、[Azure Recovery Services フォーラム](http://go.microsoft.com/fwlink/?LinkId=313628)に質問を投稿してください。


## 開始する前に
次の前提条件を満たしていることを確認してください。
### Azure の前提条件

- [Microsoft Azure](http://azure.microsoft.com/) のアカウントが必要です。お持ちでない場合は、[無料評価版](http://aka.ms/try-azure)をお試しください。また、「[Azure Site Recovery Manager の料金](http://go.microsoft.com/fwlink/?LinkId=378268)」も参照してください。
- 情報およびデータの利用方法を理解するために、[Microsoft Azure のプライバシーに関する声明](http://go.microsoft.com/fwlink/?LinkId=324899)を参照してください。さらに、このトピックの最後にある、「<a href="#privacy">Site Recovery のプライバシー情報</a>」も併せてご覧ください。

### VMM の前提条件
- VMM サーバーが少なくとも 1 つ必要です。
- VMM サーバーは、少なくとも累積的な更新プログラムがインストールされている System Center 2012 SP1 を実行している必要があります。 
- 保護する仮想マシンを含むすべての VMM サーバーが Azure Site Recovery プロバイダーを実行している必要があります。このプロバイダーは Azure Site Recovery のデプロイ時にインストールされます。
- 1 つの VMM サーバーで保護を設定する場合、そのサーバーには少なくとも 2 つのクラウドを構成する必要があります。
- 2 つの VMM サーバーで保護をデプロイする場合、保護するプライマリ VMM サーバーに少なくとも 1 つのクラウドを構成し、保護と復旧に使用するセカンダリ VMM サーバーに 1 つのクラウドを構成する必要があります。
- すべての VMM クラウドには Hyper-V キャパシティ プロファイル セットが必要です。
- 保護するソース クラウドには、以下のものが含まれている必要があります。
	- 1 つ以上の VMM ホスト グループ。
	- 各ホスト グループに 1 つ以上の Hyper-V ホスト サーバー。
	- ホスト サーバー上に配置された 1 つ以上の仮想マシン。 
- VMM クラウドの設定について理解を深めます。
	- プライベート VMM クラウドの詳細については、[System Center 2012 R2 VMM のプライベート クラウドの新機能](http://go.microsoft.com/fwlink/?LinkId=324952)に関するページおよび [VMM 2012 とクラウド](http://go.microsoft.com/fwlink/?LinkId=324956)に関するページを参照してください。 
	- [VMM クラウド ファブリックの構成](https://msdn.microsoft.com/library/azure/dn469075.aspx#BKMK_Fabric)に関するページを参照してください。
	- クラウド ファブリック要素の配置後、プライベート クラウドの作成については、「[VMM でのプライベート クラウドの作成](http://go.microsoft.com/fwlink/?LinkId=324953)」および[チュートリアル: System Center 2012 SP1 VMM でのプライベート クラウドの作成](http://go.microsoft.com/fwlink/?LinkId=324954)に関するページを参照してください。

### Hyper-V の前提条件

- ホストおよびターゲットの Hyper-V サーバーは、少なくとも Hyper-V ロールを持つ Windows Server 2012 を実行しており、最新の更新プログラムがインストールされている必要があります。
- クラスターで Hyper-V を実行している場合に、静的 IP アドレス ベースのクラスターが存在すると、クラスター ブローカーが自動的に作成されません。クラスター ブローカーを手動で構成する必要があります。手順については、「[Hyper-V レプリカ ブローカーを構成する](http://go.microsoft.com/fwlink/?LinkId=403937).」を参照してください。
- 保護を管理するすべての Hyper-V ホスト サーバーまたはクラスターが VMM クラウドに属している必要があります。
 
### ネットワーク マッピングの前提条件
ネットワーク マッピングにより、フェールオーバー後にレプリカの仮想マシンが Hyper-V ホスト サーバーへ最適に配置され、適切な VM ネットワークに接続できるようになります。ネットワーク マッピングを構成しないと、レプリカの仮想マシンはフェールオーバー後に VM ネットワークに接続されません。ネットワーク マッピングをデプロイする場合は、以下のことが必要になります。

- ソース VMM サーバー上の保護する仮想マシンが VM ネットワークに接続している。そのネットワークは、クラウドに関連付けられた論理ネットワークにリンクされている必要があります。
- 復旧に使用するセカンダリ VMM サーバーのターゲット クラウドには対応する VM ネットワークが構成されており、そのネットワークが、ターゲット クラウドに関連付けられている、対応する論理ネットワークにリンクされている必要があります。

ネットワーク マッピングについての理解を深める。

- [VMM での論理ネットワークの構成](http://go.microsoft.com/fwlink/?LinkId=386307)
- [VMM での VM ネットワークとゲートウェイの構成](http://go.microsoft.com/fwlink/?LinkId=386308)

### ストレージ マッピングの前提条件
ソースの Hyper-V ホスト サーバーにある仮想マシンをターゲットの Hyper-V ホスト サーバーにレプリケートする場合、既定では、レプリケートされたデータは既定の場所に格納されます。その場所は、Hyper-V マネージャーでターゲットの Hyper-V ホストに対して指定されています。レプリケートされたデータの格納場所を細かく制御する場合は、ストレージ マッピングを構成します。そのためには、デプロイの開始前に、ソースとターゲットの VMM サーバーでストレージ分類を設定する必要があります。
手順については、[VMM のストレージ分類の作成方法](http://go.microsoft.com/fwlink/?LinkId=400937)に関するページを参照してください。


## 手順 1:Site Recovery コンテナーの作成

1. 登録する VMM サーバーから[管理ポータル](https://portal.azure.com)にサインインします。

2. **[データ サービス]**、**[復旧サービス]** の順に展開し、**[Site Recovery コンテナー]** をクリックします。


3. **[新規作成]**、**[簡易作成]** の順にクリックします。
	
4. **[名前]** ボックスに、コンテナーを識別する表示名を入力します。

5. **[リージョン]** ボックスで、コンテナーのリージョンを選択します。サポートされているリージョンを確認するには、[Azure Site Recovery の料金の詳細](http://go.microsoft.com/fwlink/?LinkId=389880)に関するページで利用可能地域を参照してください。</a>

6. **[コンテナーの作成]** をクリックします。

	![Create vault](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_CreateVault.png)

ステータス バーでコンテナーが作成されたことを確認します。メインの [復旧サービス] ページで、コンテナーは **[アクティブ]** と表示されています。

## 手順 2:コンテナー登録キーの生成

コンテナーの登録キーを生成します。Azure Site Recovery プロバイダーをダウンロードして VMM サーバーにインストールした後で、このキーを使用して、VMM サーバーをコンテナーに登録します。

1. <b>[復旧サービス]</b> ページで、コンテナーをクリックして [クイック スタート] ページを開きます。[クイック スタート] は、アイコンを使っていつでも開くことができます。

	![Quick Start Icon](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_QuickStartIcon.png)

2. ドロップダウン リストで、**[2 つの内部設置型 Hyper-V サイト間]** を選択します。
3. **[VMM サーバーの準備]** で、**[登録キーの生成]** ファイルをクリックします。キー ファイルは自動的に生成され、生成後 5 日間有効です。VMM サーバーから Azure ポータルにアクセスしていない場合は、このファイルをサーバーにコピーする必要があります。 

	![Registration key](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_E2ERegisterKey.png)
	
## 手順 3:Azure Site Recovery プロバイダーのインストール	

1. <b>[クイック スタート]</b> ページの **[VMM サーバーの準備]** で、<b>[VMM サーバーへのインストール用の Microsoft Azure Site Recovery プロバイダーのダウンロード]</b> をクリックして、最新バージョンのプロバイダー インストール ファイルを取得します。

2. ソース VMM サーバーとターゲット VMM サーバーでこのファイルを実行します。プロバイダーを初めてインストールするときに、VMM がクラスターにデプロイされている場合は、プロバイダーをアクティブなノードにインストールします。インストールが終了したら、VMM サーバーをコンテナーに登録します。次に、プロバイダーを他のノードにインストールします。プロバイダーをアップグレードする場合は、すべてのノードでアップグレードする必要があります。これは、すべてのノードで同じバージョンのプロバイダーを実行する必要があるためです。

3. **[前提条件の確認]** で、VMM サービスを停止して、プロバイダーのセットアップを開始します。VMM サービスは停止し、セットアップの終了時に自動的に再起動します。VMM クラスターにインストールしている場合は、クラスター ロールを停止するよう求められます。

	![Prerequisites](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderPrereq.png)

4. **[Microsoft Update]** で、更新プログラムの内容を選択できます。この設定を行うことで、設定した Microsoft Update のポリシーに従って、プロバイダーの有効な更新がインストールされます。

	![Microsoft Updates](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderUpdate.png)

プロバイダーのインストール後は、セットアップを続行し、サーバーをコンテナーに登録します。

5. [インターネット接続] ページで、VMM サーバーで実行中のプロバイダーがインターネットに接続する方法を指定します。<b>[既定のシステム プロキシ設定を使用]</b> を選択して、サーバー上に構成されている既定のインターネット接続設定を使用します。

	![Internet Settings](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderProxy.png)

	- カスタム プロキシを使用する場合は、プロバイダーをインストールする前に設定する必要があります。カスタム プロキシ設定を構成すると、プロキシの接続を確認するためのテストが実施されます。
	- カスタム プロキシを使用する場合、または既定のプロキシで認証が必要な場合、プロキシのアドレスやポートなどの詳細を入力する必要があります。
	- プロキシ経由のルーティングからは以下のアドレスを除外する必要があります。
		- Azure Site Recovery に接続するための URL:*.hypervrecoverymanager.windowsazure.com
		- *.accesscontrol.windows.net
		- *.backup.windowsazure.com
		- *.blob.core.windows.net 
		- *.store.core.windows.net 
	- Azure ドメイン コントローラーへの送信接続を許可する必要がある場合、「[Azure Datacenter の IP 範囲](http://go.microsoft.com/fwlink/?LinkId=511094)」に記載されている IP アドレスを許可し、HTTP (80) および HTTPS (443) プロトコルを許可します。 
	- カスタム プロキシを使用する場合、指定されたプロキシの資格情報を使用して VMM RunAs アカウント (DRAProxyAccount) が自動的に作成されます。このアカウントが正しく認証されるようにプロキシ サーバーを構成します。VMM RunAs アカウントの設定は VMM コンソールで変更できます。変更するには、[設定] ワークスペースを開いて [セキュリティ] を展開し、[実行アカウント] をクリックします。その後、DRAProxyAccount のパスワードを変更します。新しい設定を有効にするには、VMM サービスを再起動する必要があります。
6. **[登録キー]** で、Azure Site Recovery からダウンロードして VMM サーバーにコピーした登録キーを選択します。
7. **[コンテナー名]** で、サーバーが登録されるコンテナーの名前を確認します。
8. **[サーバー名]** に、コンテナーで VMM サーバーを識別する表示名を入力します。クラスター構成で、VMM クラスターのロール名を指定します。 

	![Server registration](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderRegKeyServerName.png)


9. **[初期クラウド メタデータ]** 同期で、VMM サーバー上のすべてのクラウドのメタデータをコンテナーと同期するかどうか選択します。この操作は、各サーバーで 1 回のみ実行する必要があります。すべてのクラウドを同期したくない場合は、この設定をオフのままにして、VMM コンソールのクラウドのプロパティで各クラウドを個別に同期できます。


7. **[データの暗号化]** オプションは、オンプレミス間の保護には該当しません。

	![Server registration](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_ProviderSyncEncrypt.png)

8. <b>[登録]</b> をクリックしてプロセスを完了します。VMM サーバーからのメタデータが Azure Site Recovery によって取得されます。サーバーは、コンテナーの **[サーバー]** ページの **[リソース]** タブに表示されます。

登録後、VMM コンソールまたはコマンド ラインを使用してプロバイダーの設定を変更することができます。

## 手順 4:クラウドの保護設定の構成

VMM サーバーを登録した後、クラウドの保護設定を構成することができます。プロバイダーのインストール時にオプション **[コンテナーとクラウド データの同期]** を有効にしたので、VMM サーバー上のすべてのクラウドが、コンテナーの <b>[保護された項目]</b> タブに表示されます。有効にしていない場合でも、VMM コンソールでクラウドのプロパティのページにある **[全般]** タブを使用すると、特定のクラウドを Azure Site Recovery と同期することができます。

![Published Cloud](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_CloudsList.png)

1. [クイック スタート] ページで、**[VMM クラウドの保護の設定]** をクリックします。
2. **[VMM クラウド]** タブで、構成するクラウドを選択し、**[構成]** タブに移動します。 
3. <b>[ターゲット]</b> で、<b>[VMM]</b> を選択します。
4. <b>[ターゲットの場所]</b> で、復旧時に使用するクラウドを管理するオンサイト VMM サーバーを選択します。
4. <b>[ターゲット クラウド]</b> で、ソース クラウド内の仮想マシンのフェールオーバー時に使用するターゲット クラウドを選択します。以下の点に注意してください。
	- 保護する仮想マシンの復旧要件を満たしたターゲット クラウドを選択することをお勧めします。
	- クラウドは、プライマリ クラウドまたはターゲット クラウドとして、1 つのクラウド ペアに属することしかできません。
6. <b>[コピーの頻度]</b> で、ソースとターゲットの場所の間でデータが同期される頻度を指定します。この設定は、Hyper-V ホストで Windows Server 2012 R2 を実行している場合にのみ該当します。他のサーバーでは、5 分間という既定の設定が使用されます。
7. <b>[追加の復旧ポイント]</b> では、追加の復旧ポイントを作成するかどうかを指定します。既定値の 0 は、プライマリ仮想マシンの最新の復旧ポイントのみがレプリカのホスト サーバーに格納されることを示します。複数の復旧ポイントを有効にすると、各復旧ポイントで格納されるスナップショット用に追加のストレージが必要になります。既定では、1 時間ごとに復旧ポイントが作成されるため、各復旧ポイントで 1 時間分のデータが格納されます。VMM コンソールで仮想マシンに割り当てる復旧ポイントの値は、Azure Site Recovery コンソールで割り当てる値より小さくしないでください。
8. <b>[アプリケーション整合性スナップショットの頻度]</b> に、アプリケーション整合性スナップショットを作成する頻度を指定します。Hyper-V では 2 種類のバックアップを使用します。1 つは標準バックアップで、仮想マシン全体の増分バックアップを実行します。もう 1 つは、アプリケーション整合性スナップショットで、仮想マシン内部のアプリケーション データの特定の時点のスナップショットを作成します。アプリケーション整合性スナップショットでは、ボリューム シャドウ コピー サービス (VSS) を使用して、スナップショットを作成するときにアプリケーションを一貫性のある状態に保ちます。アプリケーション整合性スナップショットを有効にすると、ソースの仮想マシンで実行するアプリケーションのパフォーマンスに影響があります。設定する値は、追加で構成する復旧ポイントの数より少ない数にしてください。

	![Configure protection settings](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_CloudSettings.png)

9. <b>[データ転送の圧縮]</b> で、転送されるレプリケート済みデータを圧縮するかどうかを指定します。 
10. <b>[認証]</b> で、プライマリおよび復旧用の Hyper-V ホスト サーバー間でトラフィックを認証する方法を指定します。実際に利用できる Kerberos 環境を構成している場合を除き、HTTPS を選択してください。Azure Site Recovery は、HTTPS 認証に使用する証明書を自動的に構成します手動で構成する必要はありません。Kerberos を選択した場合は、ホスト サーバーの相互認証に Kerberos チケットが使用されます。既定では、Hyper-V ホスト サーバーの Windows ファイアウォールで、ポート 8083 と 8084 (認証用) が開きます。この設定は、Windows Server 2012 R2 で実行されている Hyper-V ホスト サーバーだけに関連することに注意してください。
11. <b>[ポート]</b> で、ソースとターゲットのホスト コンピューターがレプリケーション トラフィックをリッスンするポートの番号を変更します。たとえば、レプリケーション トラフィックにサービス品質 (QoS) ネットワーク帯域幅調整を適用する場合などに、設定を変更します。ポートが他のアプリケーションで使用されていないことと、ファイアウォール設定で開いていることを確認します。
12. <b>[レプリケーション方法]</b> で、通常のレプリケーションを開始する前の、ソースからターゲットへのデータの初期レプリケーションの処理方法を指定します。 
	- <b>[ネットワーク経由]</b> - ネットワーク経由でデータをコピーすると、時間がかかり大量のリソースを消費する場合があります。クラウドの仮想マシンで使用する仮想ハード ディスクの容量が比較的小さい場合、そして、プライマリ VMM サーバーがセカンダリ VMM サーバーに高速回線で接続されている場合に、このオプションを使用することをお勧めします。コピーを直ちに開始することも、実行時刻を選択することもできます。ネットワーク レプリケーションを使用する場合は、ピーク時間以外に実行することをお勧めします。
	- <b>[オフライン]</b> - この方法では、外部メディアを使用して初期レプリケーションを実行することを指定します。これは、ネットワーク性能の低下を避ける場合、または、場所が地理的に離れている場合に便利です。この方法を使用するには、ソース クラウド上のエクスポート場所と、ターゲット クラウド上のインポート場所を指定します。仮想マシンの保護を有効にすると、指定されたエクスポート場所に仮想ハード ディスクがコピーされます。それをターゲット サイトに送信して、インポート場所にコピーします。インポートされた情報はレプリカ仮想マシンに自動的にコピーされます。オフライン レプリケーションの前提条件の詳細については、デプロイ ガイド<a href="http://go.microsoft.com/fwlink/?LinkId=323469">の「手順 3: VMM クラウドの保護設定の構成</a>」を参照してください。
13. クラウドのプロパティにある [仮想マシン] タブの **[仮想マシンの保護を削除する]** オプションを選択して、仮想マシンの保護を停止する場合に削除するレプリカ仮想マシンを、**[レプリカ仮想マシンの削除]** を選択して指定します。この設定を有効にすると、保護を無効にしたときに、仮想マシンの Azure Site Recovery からの削除、VMM コンソールでの仮想マシンの Site Recovery 設定の削除、レプリカの削除が行われます。
	![Configure protection settings](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_CloudSettingsRep.png)

<p>この設定を保存するとジョブが作成され、これを <b>[ジョブ]</b> タブで監視できます。VMM ソース クラウド内のすべての Hyper-V ホスト サーバーは、レプリケーション用に構成されます。クラウド設定は <b>[構成]</b> タブで変更できます。ターゲットの場所やターゲット クラウドを変更する場合は、クラウド構成を削除して、クラウドを再構成する必要があります。</p>

### オフラインでの初期レプリケーションの準備

オフラインで初期レプリケーションを行うための準備として、いくつかの操作を実行する必要があります。

- ソース サーバーで、データのエクスポートを実施するパスの場所を指定します。そのエクスポート パスに対する NTFS のフル コントロール アクセス許可と共有アクセス許可を VMM サービスに付与します。ターゲット サーバーで、データのインポートを実施するパスの場所を指定します。このインポート パスに対して同じアクセス許可を割り当てます。 
- インポート パスまたはエクスポート パスが共有されている場合は、その共有パスの配置先であるリモート コンピューター上の VMM サービス アカウントに対して、Administrator、Power User、Print Operator、または Server Operator グループのメンバーシップを割り当てます。
- いずれかの実行アカウントを使用してホストを追加している場合は、インポート パスおよびエクスポート パスに対する読み取りと書き込みのアクセス許可を VMM の実行アカウントに割り当てます。
- Hyper-V ではループバックの構成がサポートされていないため、インポート用の共有とエクスポート用の共有は、Hyper-V ホスト サーバーとして使用するどのコンピューターにも配置しないでください。
- Active Directory で、保護対象の仮想マシンを含む各 Hyper-V ホスト サーバーに対して、次のように制約付き委任を有効にして構成し、インポート パスとエクスポート パスの配置先であるリモート コンピューターを信頼します。
	1. ドメイン コントローラーで、**[Active Directory ユーザーとコンピューター]** を開きます。
	2. コンソール ツリーで **[ドメイン名]**、**[コンピューター]** の順にクリックします。
	3. Hyper-V ホスト サーバー名を右クリックして、**[プロパティ]** をクリックします。
	4. **[委任]** タブで、**[指定されたサービスへの委任でのみこのコンピューターを信頼する]** をクリックします。
	5. **[任意の認証プロトコルを使う]** をクリックします。
	6. **[追加]**、**[ユーザーとコンピューター]** の順にクリックします。
	7. エクスポート パスをホストするコンピューターの名前を入力し、**[OK]** をクリックします。利用可能なサービスの一覧で、Ctrl キーを押しながら **[cifs]** をクリックし、**[OK]** をクリックします。インポート パスをホストするコンピューターの名前に対して、同じ手順を繰り返します。必要に応じて、追加の Hyper-V ホスト サーバーに対して同じ手順を繰り返します。
	
## 手順 5:ネットワーク マッピングの構成
1. [クイック スタート] ページで、**[ネットワークのマップ]** をクリックします。
2. ネットワークをマップするソース VMM サーバーを選択し、次に、ネットワークがマップされるターゲット VMM サーバーを選択します。ソース ネットワークとそれに関連付けられたターゲット ネットワークの一覧が表示されます。現在マッピングされていないネットワークには空白の値が表示されます。ソース ネットワーク名とターゲット ネットワーク名の横にある情報アイコンをクリックして各ネットワークのサブネットを表示します。
3. **[ソース側のネットワーク]** でネットワークを選択し、**[マップ]** をクリックします。サービスによってターゲット サーバー上の VM ネットワークが検出され、表示されます。 

	![Configure network mapping](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_NetworkMapping1.png)

4. ダイアログ ボックスで、ターゲット VMM サーバーのいずれかの VM ネットワークを選択します。

	![Select a target network](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_NetworkMapping2.png)

5. ターゲット ネットワークを選択すると、ソース ネットワークを使用する保護されたクラウドが表示されます。保護に使用するクラウドに関連付けられている使用可能なターゲット ネットワークも表示されます。保護に使用しているすべてのクラウドで使用可能なターゲット ネットワークを選択することをお勧めします。
6. チェック マークをクリックして、マッピング処理を完了します。ジョブが開始され、マッピングの進行状況を追跡します。進行状況は **[ジョブ]** タブで確認できます。


## 手順 6:ストレージ マッピングの構成
ソースの Hyper-V ホスト サーバーにある仮想マシンをターゲットの Hyper-V ホスト サーバーにレプリケートする場合、既定では、レプリケートされたデータは既定の場所に格納されます。その場所は、Hyper-V マネージャーでターゲットの Hyper-V ホストに対して指定されています。レプリケートされたデータの格納場所を細かく制御する場合は、次のようにストレージ マッピングを構成します。

- ソースとターゲットの両方の VMM サーバーでストレージ分類を定義します。手順については、[VMM のストレージ分類の作成方法](http://go.microsoft.com/fwlink/?LinkId=400937)に関するページを参照してください。分類は、ソースおよびターゲットのクラウド内の Hyper-V ホスト サーバーで使用できる必要があります。分類のストレージの種類は同じでなくてもかまいません。たとえば、SMB 共有が含まれているソース分類を CSV が含まれているターゲット分類に割り当てることができます。
- 分類を定義したら、マッピングを作成できます。
1. **[クイック スタート]** ページで、**[ストレージのマップ]** をクリックします。
1. **[ストレージ]** タブで、**[ストレージ分類のマップ]** をクリックします。
1. **[ストレージ分類のマップ]** タブで、ソースとターゲットの VMM サーバーの分類を選択します。設定を保存します。

	![Select a target network](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_StorageMapping1.png)


## 手順 7:仮想マシンの保護の有効化
サーバー、クラウド、およびネットワークを正しく構成した後で、クラウド内の仮想マシンの保護を有効にすることができます。

1. 仮想マシンが配置されているクラウドの **[仮想マシン]** タブで、**[保護を有効にする]**、**[仮想マシンの追加]** の順にクリックします。 
2. クラウド内の仮想マシンのリストから、保護する仮想マシンを選択します。


![Enable virtual machine protection](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_EnableProtectionVM.png)

**[ジョブ]** タブで、初期レプリケーションを含む [保護の有効にする] アクションの進捗状況を確認します。保護の最終処理のジョブが実行されると、仮想マシンは、フェールオーバーを実行できる状態になります。保護が有効され仮想マシンが複製されると、Azure でそれらの状態を表示できます。

VMM コンソールでも仮想マシンの保護を有効にできます。仮想マシン プロパティの [Azure Site Recovery] タブで、ツール バーにある [保護を有効にする] をクリックします。


![Virtual machine protection job](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_VMJobs.png)

### 既存の仮想マシンの追加
Hyper-V レプリカを使用してレプリケートされる VMM 内に既存の仮想マシンがある場合、次の手順で Azure Site Recovery の保護にそれらの仮想マシンを追加する必要があります。
1. プライマリ クラウドとセカンダリ クラウドがあることを確認します。既存の仮想マシンをホストしている Hyper-V サーバーがプライマリ クラウドに存在し、レプリカ仮想マシンをホストしている Hyper-V サーバーがセカンダリ クラウドに存在することを確認します。クラウドの保護設定が構成されていることを確認します。この設定が、Hyper-V レプリカに現在構成されている設定と一致する必要があります。一致しない場合、仮想マシンのレプリケーションが正しく動作しない可能性があります。
2. 次に、プライマリ仮想マシンの保護を有効にします。Azure Site Recovery と VMM によって同じレプリカ ホストと仮想マシンが検出され、Azure Site Recovery によってクラウドを構成する際に構成された設定を使用して、レプリケーションの再使用と再確立が行われます。


## デプロイのテスト

展開をテストするために、1 台の仮想マシンに対するテスト フェールオーバーを実行することや、複数の仮想マシンで構成される復旧計画を作成して、その計画のテスト フェールオーバーを実行することができます。テスト フェールオーバーは、孤立したネットワークでフェールオーバーと復旧のシミュレーションを実行します。

### 復旧計画の作成

1. **[復旧計画]** タブで、**[復旧計画の作成]** をクリックします。
2. 復旧計画の名前、ソースおよびターゲット VMM サーバーを指定します。ソース サーバーには、フェールオーバーと復旧が有効になった仮想マシンが必要になります。**[Hyper-V]** を選択し、Hyper-V レプリケーションが構成されたクラウドのみを表示します。

	![Create recovery plan](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_RP1.png)

3. **[仮想マシンの選択]** で、レプリケーション グループを選択します。レプリケーション グループに関連付けられているすべての仮想マシンを選択し、復旧計画に追加します。これらの仮想マシンは、復旧計画の既定のグループ「グループ 1」に追加されます。必要に応じて、他のグループを追加することもできます。レプリケーションの後、仮想マシンは復旧計画のグループの順序に従って起動することに注意してください。

	![Add virtual machines](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_RP2.png)	

復旧計画が作成されると、**[復旧計画]** タブに一覧が表示されます。 

###テスト フェールオーバーの実行

1. **[復旧計画]**] タブで、計画を選択し、**[テスト フェールオーバー]** をクリックします。
2. **[テスト フェールオーバーの確認]** ページで、**[なし]** を選択します。このオプションが有効な場合、フェールオーバーしたレプリカ仮想マシンはネットワークに接続されないことに注意してください。このテストでは、仮想マシンが想定どおりにフェールオーバーすることをテストしますが、レプリケーションのネットワーク環境はテストしません。より包括的なテスト フェールオーバーを実行する場合は、MSDN の<a href="http://go.microsoft.com/fwlink/?LinkId=522291">オンプレミス間のデプロイのテスト</a>に関するページを参照してください。

	![Select test network](./media/site-recovery-vmm-to-vmm/ASRE2EHVR_TestFailover1.png)


7. レプリカ仮想マシンが存在するホストと同じホスト上に、テスト仮想マシンが作成されます。これは、レプリカ仮想マシンが配置されているクラウドには追加されません。

### 復旧計画の実行
レプリケーション後に、レプリカ仮想マシンは、プライマリ仮想マシンの IP アドレスとは異なる IP アドレスを保有します。DHCP からアドレスを発行している場合、DNS は自動的に更新されます。DHCP を使用していない場合に、アドレスが同じかどうかを確認するには、複数のスクリプトを実行する必要があります。

#### IP アドレスを取得するスクリプト
次のサンプル スクリプトを実行して、IP アドレスを取得します。
    **$vm = Get-SCVirtualMachine -Name <VM_NAME>
	$na = $vm[0].VirtualNetworkAdapters>
	$ip = Get-SCIPAddress -GrantToObjectID $na[0].id
	$ip.address**  

#### DNS を更新するスクリプト
前のサンプル スクリプトを使用して取得した IP アドレスを指定して、次のサンプル スクリプトを実行し、DNS を更新します。

	**[string]$Zone,
	[string]$name,
	[string]$IP
	)
	$Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
	$newrecord = $record.clone()
	$newrecord.RecordData[0].IPv4Address  =  $IP
	Set-DnsServerResourceRecord -zonename com -OldInputObject $record -NewInputObject $Newrecord**



<a name="privacy"></a><h2>Site Recovery のプライバシー情報</h2>

次のセクションでは、Microsoft Azure Site Recovery サービス (以降、"本サービス") の追加のプライバシー情報を示します。Microsoft Azure サービスのプライバシーに関する声明については、
[Microsoft Azure のプライバシーに関する声明](http://go.microsoft.com/fwlink/?LinkId=324899)のページを参照してください。

**特徴: 登録**

- **実行内容**: 仮想マシンを保護できるようにサービスにサーバーを登録します。
- **収集される情報**: 登録後、本サービスは、VMM サーバーからの管理証明書の情報を収集、処理、送信します。このサーバーは、VMM サーバーの本サービス名および VMM サーバー上のクラウドの仮想マシン クラウドの名前を使用して障害復旧を提供するように指定されているサーバーです。
- **情報の用途**: 
	- 管理証明書: 登録済みの VMM サーバーを識別および認証して本サービスにアクセスできるようにするために使用します。本サービスは、証明書の公開キー部分を使用して、登録済みの VMM サーバーのみがアクセスできるトークンをセキュリティで保護します。サーバーは、このトークンを使用して本サービスの機能にアクセスする必要があります。
	- VMM サーバーの名前: VMM サーバー名は、クラウドが配置されている適切な VMM サーバーを識別し、この VMM サーバーと通信するために必要です。
	- VMM サーバーからのクラウド名: クラウド名は、以下に説明する本サービスのクラウドのペアリング/ペアリング解除機能を使用するときに必要になります。プライマリ データ センターのクラウドを復旧データ センター内のクラウドとペアリングすることを決定した場合、復旧データ センターのすべてのクラウドの名前が表示されます。

- **選択**: この情報は、お客様および本サービスが Azure Site Recovery 保護を提供する対象の VMM サーバーを識別すると共に、正しい登録済み VMM サーバーを識別できるようにするために使用されるため、本サービスの登録プロセスに必要不可欠の情報です。この情報を本サービスに送信することを希望しない場合は、本サービスは使用しないでください。サーバーを登録した後で登録を解除することを希望する場合は、サービス ポータル (Azure ポータル) から VMM サーバー情報を削除することができます。

**特徴: Azure Site Recovery 保護を有効にする**

- **実行内容**: VMM サーバーにインストールされる Azure Site Recovery プロバイダーは、本サービスと通信するためのパイプの役割を果たします。プロバイダーは、VMM プロセスでホストされているダイナミック リンク ライブラリ (DLL) です。プロバイダーをインストールすると、VMM 管理者コンソールで Datacenter Recovery 機能が有効になります。クラウドのすべての新規または既存の仮想マシンでは、"Datacenter Recovery" という名前のプロパティを有効にすることで仮想マシンを保護できます。このプロパティを設定すると、プロバイダーは、仮想マシンの名前と ID を本サービスに送信します。仮想保護は、Windows Server 2012 または Windows Server 2012 R2 Hyper-V レプリケーション テクノロジによって実現されます。仮想マシンのデータは、1 つの Hyper-V ホストから (通常は別の "復旧" データ センターにある) 別の Hyper-V ホストにレプリケートされます。

- **収集される情報**: 本サービスは、仮想マシンのメタデータ (名前、ID、仮想ネットワーク、所属しているクラウドの名前など) を収集、処理、および 
送信します。

- **情報の用途**:本サービスは、上記の情報を使用して、サービス ポータルの仮想マシン情報を設定します。

- **選択**: これは、本サービスに必要不可欠であり、無効にすることはできません。この情報を本サービスに送信することを希望しない場合は、すべての仮想マシンに対して Azure Site Recovery 保護を有効にしないでください。プロバイダーによって本サービスに送信されるすべてのデータは、HTTPS を介して送信されます。

**特徴: 復旧計画**

- **実行内容**: この機能を使用すると、"復旧" データ センターのオーケストレーション計画を作成できます。仮想マシンまたは仮想マシンのグループを復旧サイトで起動する順序を定義できます。さらに、各仮想マシンの復旧時に、実行する任意の自動化スクリプトまたは任意の手動による操作を指定することができます。フェールオーバー (次のセクションで説明します) は、通常、調整された復旧を行う復旧計画レベルでトリガーされます。

- **収集される情報**: サービスは、仮想マシンのメタデータ、すべての自動スクリプトおよび手動アクションのメモのメタデータを含む、復旧計画のメタデータを収集、処理、および送信します。

- **情報の用途**:上記で説明したメタデータを使用して、サービス ポータルで復旧計画を作成します。

- **選択**: これは、本サービスに必要不可欠であり、無効にすることはできません。この情報を本サービスに送信することを希望しない場合は、本サービスで復旧計画を作成しないください。

**特徴: ネットワーク マッピング**

- **実行内容**: この機能を使用すると、プライマリ データ センターから復旧データ センターにネットワークの情報をマップすることができます。このマッピングは、仮想マシンを復旧サイトで復旧するときにネットワーク接続を確立するために使用できます。

- **収集される情報**: 本サービスは、ネットワーク マッピング機能の一部として、各サイト (プライマリ サイトとデータ センター) の論理ネットワークのメタデータを収集、処理、および送信します。

- **情報の用途**: 本サービスは、メタデータを使用して、サービス ポータルの情報を設定します。サービス ポータルでは、ネットワーク情報をマップすることができます。

- **選択**: これは、本サービスに必要不可欠であり、無効にすることはできません。この情報を本サービスに送信することを希望しない場合は、ネットワーク マッピング機能を使用しないでください。

**特徴: フェールオーバー - 計画済み、計画外、テスト**

- **実行内容**: この機能は、VMM 管理対象データ センター間で仮想マシンのフェールオーバーを行う場合に役立ちます。フェールオーバー アクションは、サービス ポータル上のユーザーによってトリガーされます。フェールオーバーは、計画外のイベント (たとえば、自然災害)、計画されたイベント (たとえば、データ センターの負荷分散)、テスト フェールオーバー (たとえば、復旧計画のリハーサル) などの理由で発生します。

VMM サーバー上のプロバイダーは、本サービスからイベントの通知を受け取ると、VMM インターフェイスを介して Hyper-V ホスト上でフェールオーバー アクションを実行します。1 つの Hyper-V ホストから (通常は別の "復旧" データ センターで実行されている) 別の Hyper-V ホストへの実際のフェールオーバーは、Windows Server 2012 または Windows Server 2012 R2 Hyper-V レプリケーション テクノロジによって処理されます。フェールオーバーが完了した後、"復旧" データ センターの VMM サーバーにインストールされているプロバイダーから処理の成功を示す情報が本サービスに送信されます。

- **収集される情報**: 本サービスは、上記の情報を使用して、サービス ポータルのフェールオーバー アクション情報の状態を設定します。

- **情報の用途**:本サービスは、上記の情報を以下の目的で使用します。

	- 管理証明書: 登録済みの VMM サーバーを識別および認証して本サービスにアクセスできるようにするために使用します。本サービスは、証明書の公開キー部分を使用して、登録済みの VMM サーバーのみがアクセスできるトークンをセキュリティで保護します。サーバーは、このトークンを使用して本サービスの機能にアクセスする必要があります。
	- VMM サーバーの名前: VMM サーバー名は、クラウドが配置されている適切な VMM サーバーを識別し、この VMM サーバーと通信するために必要です。
	- VMM サーバーからのクラウド名: クラウド名は、以下に説明する本サービスのクラウドのペアリング/ペアリング解除機能を使用するときに必要になります。プライマリ データ センターのクラウドを復旧データ センター内のクラウドとペアリングすることを決定した場合、復旧データ センターのすべてのクラウドの名前が表示されます。

- **選択**: これは、本サービスに必要不可欠であり、無効にすることはできません。この情報を本サービスに送信することを希望しない場合は、本サービスを使用しないでください。


<!--HONumber=49--> 