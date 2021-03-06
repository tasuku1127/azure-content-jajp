<properties
	pageTitle="VM でのアプリケーション アクセスのトラブルシューティング |Microsoft Azure"
	description="Azure 仮想マシンで実行されているアプリケーションにアクセスできない場合は、次の手順を使用して問題の原因を特定してください。"
	services="virtual-machines"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor=""
	tags="top-support-issue,azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/05/2015"
	ms.author="dkshir"/>

# Azure 仮想マシンで実行されているアプリケーションへのアクセスに関するトラブルシューティング

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]


この記事では、Azure 仮想マシンで実行されているアプリケーションにアクセスできない場合にその原因を特定し、問題を解決するための系統的アプローチについて説明します。

> [AZURE.NOTE]Azure 仮想マシンへの接続に関する詳細は、「[Windows ベースの Azure 仮想マシンへのリモート デスクトップ接続に関するトラブルシューティング](virtual-machines-troubleshoot-remote-desktop-connections.md)」または「[Linux ベースの Azure 仮想マシンに対する Secure Shell (SSH) 接続のトラブルシューティング](virtual-machines-troubleshoot-ssh-connections.md)」参照してください。

Azure 仮想マシンで実行されているアプリケーションへのアクセスに問題がある場合は、次の 4 つの主要領域からトラブルシューティングを行います。

![](./media/virtual-machines-troubleshoot-access-application/tshoot_app_access1.png)

1.	Azure 仮想マシンで実行されているアプリケーション。
2.	Azure 仮想マシン。
3.	仮想マシンが属しているクラウド サービスの Azure エンドポイント (サービス管理 API を使用して作成された仮想マシンの場合)、受信 NAT 規則 (リソース マネージャーで作成された仮想マシンの場合)、およびネットワーク セキュリティ グループ。
4.	インターネット エッジ デバイス。

サイト間 VPN または ExpressRoute 接続を介してアプリケーションにアクセスするクライアント コンピューターの場合、問題が起こる主な領域はアプリケーションと Azure 仮想マシンです。原因を特定して問題を修正するには、次の手順に従ってください。

## 手順 1: ターゲットの仮想マシンからアプリケーションにアクセスできますか。

アプリケーションが実行されている仮想マシンから、適切なクライアント プログラムを使用してアプリケーションへのアクセスを試行してください。ローカル ホスト名、ローカル IP アドレス、またはループバック アドレス (127.0.0.1) を使用します。

![](./media/virtual-machines-troubleshoot-access-application/tshoot_app_access2.png)

たとえば、アプリケーションが Web サーバーである場合は、仮想マシンでブラウザーを実行し、仮想マシンでホストされている Web ページへのアクセスを試行します。

アプリケーションにアクセスできる場合は、 [手順 2](#step2) に進んでください。

アプリケーションにアクセスできない場合は、次の点を確認してください。

- アプリケーションがターゲットの仮想マシンで実行されている。
- アプリケーションが正しい TCP ポートと UDP ポートをリッスンしている。

Windows ベースと Linux ベースの両方の仮想マシンで、 **netstat-a** コマンドを使用してアクティブなリスニング ポートを表示します。アプリケーションがリッスンしているポートの出力を確認します。アプリケーションを再起動するか、必要であれば、正しいポートを使用するように構成を変更します。

## <a id="step2"></a>手順 2: 同じ仮想ネットワーク内の別の仮想マシンからアプリケーションにアクセスできますか。

仮想マシンのホスト名か、Azure によって割り当てられたパブリック、プライベート、またはプロバイダー IP アドレスを使用して、アプリケーションが実行されている仮想マシンと同じ仮想ネットワーク内の別の仮想マシンから、アプリケーションへのアクセスを試行してください。サービス管理 API を使用して作成された仮想マシンの場合、クラウド サービスのパブリック IP アドレスは使用しないでください。

![](./media/virtual-machines-troubleshoot-access-application/tshoot_app_access3.png)

たとえば、アプリケーションが Web サーバーである場合は、同じ仮想ネットワーク内の別の仮想マシン上のブラウザーから、Web ページへのアクセスを試行します。

アプリケーションにアクセスできる場合は、 [手順 3](#step3) に進んでください。

アプリケーションにアクセスできない場合は、次の点を確認してください。

- ターゲットの仮想マシン上のホスト ファイアウォールで、リクエスト受信と応答送信のトラフィックが許可されている。
- ターゲットの仮想マシンで実行されている侵入検出ソフトウェアやネットワーク監視ソフトウェアで、トラフィックが許可されている。
- ネットワーク セキュリティ グループでトラフィックが許可されている。
- テスト仮想マシンと仮想マシン間のパスにある仮想ネットワーク内で実行されている別のコンポーネント (ロード バランサーやファイアウォールなど) で、トラフィックが許可されている。

Windows ベースの仮想マシンについては、セキュリティ強化機能搭載 Windows ファイアウォールを使用して、ファイアウォール規則がアプリケーションの受信トラフィックと送信トラフィックを除外していないかどうかを確認してください。

## <a id="step3"></a>手順 3: 仮想ネットワークの外部にあるが、使用しているコンピューターと同じネットワークに接続していないコンピューターからアプリケーションにアクセスできますか。

アプリケーションが実行されている仮想マシンの仮想ネットワーク外にあるが、元のクライアント コンピューターと同じネットワーク上にないコンピューターから、アプリケーションへのアクセスを試行してください。

![](./media/virtual-machines-troubleshoot-access-application/tshoot_app_access4.png)

たとえば、アプリケーションが Web サーバーである場合は、仮想ネットワーク内にないコンピューターで実行されているブラウザーから、Web ページへのアクセスを試行します。

アプリケーションにアクセスできない場合は、次の点を確認してください。

- サービス管理 API を使用して作成された仮想マシンの場合、仮想マシンのエンドポイント構成で、受信トラフィック (特にプロトコル (TCP または UDP) とパブリックおよびプライベート ポート番号) が許可されているかどうか。詳細については、「[仮想マシンに対してエンドポイントを設定する方法](virtual-machines-set-up-endpoints.md)」を参照してください。
- サービス管理 API を使用して作成された仮想マシンの場合、エンドポイント上のアクセス制御リスト (ACL) によって、インターネットからの着信トラフィックが遮断されていないかどうか。詳細については、「[仮想マシンに対してエンドポイントを設定する方法](virtual-machines-set-up-endpoints.md)」を参照してください。
- リソース マネージャーで作成された仮想マシンの場合、受信 NAT 規則構成で、受信トラフィック (特にプロトコル (TCP または UDP) とパブリックおよびプライベート ポート番号) が許可されているかどうか。
- ネットワーク セキュリティ グループで、リクエスト受信と応答送信のトラフィックが許可されているかどうか。詳細については、「[ネットワーク セキュリティ グループ (NSG) について](virtual-networks-nsg.md)」を参照してください。

仮想マシンまたはエンドポイントが負荷分散セットのメンバーである場合:

- プローブ プロトコル (TCP または UDP) とポート番号が正しいことを確認してください。
- プローブ プロトコルとポートが、負荷分散セットのプロトコルおよびポートと異なる場合は、次の点を確認してください。
	- アプリケーションがプローブ プロトコル (TCP または UDP) とポート番号をリッスンしている (ターゲットの仮想マシンで **netstat – a** を使用します)。
	- ターゲットの仮想マシン上のホスト ファイアウォールで、プローブ リクエスト受信とプローブ応答送信のトラフィックが許可されている。

アプリケーションにアクセスできる場合は、インターネット エッジ デバイスで次のトラフィックが許可されていることを確認してください。

- クライアント コンピューターから Azure 仮想マシンへのアプリケーション リクエスト送信トラフィック。
- Azure 仮想マシンからのアプリケーション応答受信トラフィック。

## 次のステップ

この記事の手順 1 ～ 3 を実行したうえで、問題解決のためにさらなる支援が必要な場合は、次のことが可能です。

- 全世界の Azure エキスパートに支援を要請する。MSDN の Azure フォーラムまたはスタック オーバーフロー フォーラムに問題を投稿してください。詳細については、「[Microsoft Azure フォーラム](http://azure.microsoft.com/support/forums/)」参照してください。
- Azure サポート インシデントを送信する。[Azure サポート サイト](http://azure.microsoft.com/support/options/) にアクセスし、**[テクニカル/課金サポート]** の **[サポートの要求]** をクリックしてください。

## その他のリソース

[Windows ベースの Azure 仮想マシンへのリモート デスクトップ接続に関するトラブルシューティング](virtual-machines-troubleshoot-remote-desktop-connections.md)

[Linux ベースの Azure 仮想マシンに対する Secure Shell (SSH) 接続のトラブルシューティング](virtual-machines-troubleshoot-ssh-connections.md)

<!---HONumber=Oct15_HO3-->