<properties
	pageTitle="VMM と Hyper-V サイト保護の監視とトラブルシューティング ガイド" 
	description="Azure Site Recovery は、オンプレミスのサーバーに配置されている仮想マシンの Azure またはセカンダリ データセンターへのレプリケーション、フェールオーバー、および復旧を調整します。この記事を利用し、VMM または Hyper-V サイト保護を監視し、トラブルシューティングします。" 
	services="site-recovery" 
	documentationCenter="" 
	authors="anbacker" 
	manager="mkjain" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery" 
	ms.date="06/16/2015" 
	ms.author="anbacker"/>
	
# VMM と Hyper-V サイト保護の監視とトラブルシューティング ガイド

この監視とトラブルシューティング ガイドでは、Azure Site Recovery のレプリケーション正常性を追跡する方法とトラブルシューティング手法について説明します。

## コンポーネントについて

### オンプレミス サイト間のレプリケーションのための VMM サイト展開

2 つのオンプレミス サイト間で DR を設定するとき、Azure Site Recovery プロバイダーをダウンロードし、VMM サーバーにインストールする必要があります。Azure ポータルからトリガーされるすべての操作が保護の有効化やフェールオーバーの一環としてのプライマリ側の仮想マシンのシャットダウンなどのオンプレミス操作に変換されるように、プロバイダーはインターネット接続を必要とします。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image1.png)

### オンプレミスと Azure の間のレプリケーションのための VMM サイト展開

オンプレミスと Azure の間で DR を設定するとき、Azure Site Recovery プロバイダーをダウンロードして VMM サーバーにインストールし、Azure Recovery Services エージェントを各 Hyper-V ホストにインストールする必要があります。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image2.png)

### オンプレミスと Azure の間のレプリケーションのための Hyper-V サイト展開

これは VMM 展開の場合と同じです。プロバイダーとエージェントが Hyper-V ホスト自体にインストールされるという点だけ異なります。

## 構成、保護、復旧操作の監視

ASR のすべての操作が「ジョブ」タブの下で監査され、追跡されます。構成、保護、復旧にエラーが発生した場合、[ジョブ] タブに移動し、障害がないかどうかを確認します。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image3.png)

[ジョブ] ビューの下で障害が見つかったら、そのジョブを選択し、そのジョブの [エラー詳細] をクリックします。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image4.png)

エラー詳細は問題の考えられる原因と推奨措置を特定するために役立ちます。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image5.png)

上の例では、別の操作が進行中になっているようです。その保護構成にエラーがあるためです。推奨措置に従って問題を解決します。その後、[再起動] をクリックし、操作を再開します。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image6.png)

一部の操作では再起動オプションを利用できません。その場合、オブジェクトに戻り、操作をやり直してください。進行中のジョブは [キャンセル] ボタンでいつでも取り消すことができます。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image7.png)

## 仮想マシンのレプリケーション正常性の監視

ASR は Azure ポータルを利用し、それぞれの保護対象を集中/リモート管理します。保護対象アイテムに移動し、[VMM クラウド] または [保護グループ] を選択します。[VMM クラウド] タブは VMM 基準の展開専用です。その他すべてのシナリオでは、[保護グループ] タブの下に保護対象エンティティがあります。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image8.png)

次に、それぞれのクラウドまたは保護グループの下で保護対象エンティティを選択します。保護対象エンティティを選択すると、下のウィンドウに許可される操作が表示されます。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image9.png)

上の例では、仮想マシンの正常性が「重大」です。下の「エラー詳細」ボタンをクリックすると、エラーを確認できます。「考えられる原因」と「推奨」に基づいて問題を解決します。この例では、仮想マシンを再同期する必要があります。その作業はポータルで実行できます。[再同期] ボタンをクリックしてください。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image10.png)

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image11.png)

注: アクティブな操作が進行中であるか、失敗した場合、先に述べたように [ジョブ] ビューに移動し、そのジョブ固有のエラーを確認します。

## オンプレミス問題の解決

オンプレミス Hyper-V マネージャー コンソールに接続し、仮想マシンを選択し、レプリケーション正常性を確認します。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image12.png)

この場合、*レプリケーション正常性*が「重大」になっています。*レプリケーション正常性*で詳細を確認してください。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image13.png)

#### イベント ビューアー

| シナリオ | イベント ソース |
|-------------------------	|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| VMM サイト保護 | VMM サーバー <ul><li> **Applications and Service Logs/Microsoft/VirtualMachineManager/Server/Admin** </li></ul> Hyper-V ホスト <ul><li> **Applications and Service Logs/MicrosoftAzureRecoveryServices/Replication** (Azure をターゲットとする)</li><li> **Applications and Service Logs/Microsoft/Windows/Hyper-V-VMMS/Admin** </li></ul> |
| Hyper-V サイト保護 | <ul><li> **Applications and Service Logs/MicrosoftAzureRecoveryServices/Replication** </li><li> **Applications and Service Logs/Microsoft/Azure Site Recovery/Provider/Operational** </li><li> **Applications and Service Logs/Microsoft/Windows/Hyper-V-VMMS/Admin** </li><ul>|

#### Hyper-V レプリケーションのログ記録のオプション

Hyper-V レプリカに関連するすべてのイベントは **Applications and Services Logs\Microsoft\Windows** にある Hyper-V-VMMS\Admin ログに記録されます。また、Hyper-V-VMMS の分析ログを有効にすることができます。このログを有効にするには、最初にイベント ビューアーに 分析ログとデバッグ ログを表示する必要があります。イベント ビューアーを開き、**[表示] メニュー**で、**[分析ログとデバッグ ログを表示する]** をクリックします。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image14.png)

分析ログは Hyper-V-VMMS の下に表示されます。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image15.png)

**[操作]** ウィンドウで **[ログを有効にする]** をクリックします。有効にすると、**[データ コレクター セット]** の下の **[パフォーマンス モニター]** にイベント トレース セッションとして表示されます。

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image16.png)

収集された情報を表示するには、最初にログを無効にしてトレース セッションを停止し、イベント ビューアーでログを保存し、再び開きます。あるいは、必要に応じて他のツールで変換します。


## 仮想マシンのライフサイクルについて

![](media/site-recovery-monitoring-and-troubleshooting-for-vmm-and-hyperv-site/image17.png)


## Microsoft サポートを得る

### ログ コレクション

VMM サイト保護については、「[ASR Log Collection using Support Diagnostics Platform (SDP) Tool](http://social.technet.microsoft.com/wiki/contents/articles/28198.asr-data-collection-and-analysis-using-the-vmm-support-diagnostics-platform-sdp-tool.aspx)」を参照し、必要なログを収集してください。

Hyper-V Branch Office と SMB サイト保護については、[ツール](https://dcupload.microsoft.com/tools/win7files/DIAG_ASRHyperV_global.DiagCab)をダウンロードして Hyper-V で実行し、ログを収集してください。

VMware/Physical シナリオについては、「[Azure Site Recovery Log Collection for VMware and Physical site protection](http://social.technet.microsoft.com/wiki/contents/articles/30677.azure-site-recovery-log-collection-for-vmware-and-physical-site-protection.aspx)」を参照し、必要なログを収集してください。

SDP ツールはローカルのログ ファイルを収集します。これは **%LocalAppData%\ElevatedDiagnostics** にある無作為に名前が付けられたサブフォルダーにもあります。

### サポート チケットを開く

ASR のサポート チケットを発行するには、Azure サポートにアクセスします。URL は <http://aka.ms/getazuresupport> です。

## KB 記事

-   [保護対象仮想マシンのドライブ文字を保存する方法
    > http://support.microsoft.com/kb/3031135

-   [Azure Recovery Services のトラブルシューティング方法
    > ](http://support.microsoft.com/kb/3005185)

-   [Hyper-V サイト保護で Azure Site Recovery のデバッグ ログを有効にする方法
    > ](http://support.microsoft.com/kb/3033922)

-   [ASR: 仮想マシンの保護を有効にしようとすると、「The cluster resource could not be found (クラスター リソースが見つかりませんでした)」エラーが発生する
    > ](http://support.microsoft.com/kb/3010979)
    
-   [Hyper-V レプリカを理解し、トラブルシューティングする
    > ガイド](http://www.microsoft.com/ja-jp/download/details.aspx?id=29016)

<!---HONumber=62-->