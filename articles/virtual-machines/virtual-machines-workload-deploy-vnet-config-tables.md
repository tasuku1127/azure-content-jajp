<properties
	pageTitle="構成テーブルを使用した仮想ネットワーク | Microsoft Azure"
	description="事前に設定済みの構成テーブル設定を使用した、クロスプレミス Azure 仮想ネットワークの構成方法について説明します。"
	documentationCenter=""
	services="virtual-machines"
	authors="JoeDavies-MSFT"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="Windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/20/2015"
	ms.author="josephd"/>

# 構成テーブルを使用してクロスプレミス仮想ネットワークを作成する

[AZURE.INCLUDE [learn-about-deployment-models-classic-include](../../includes/learn-about-deployment-models-classic-include.md)]リソース マネージャーのデプロイ モデル。

このトピックでは、以下の構成テーブルのセットで先に指定した設定を使用してクロスプレミス仮想ネットワークを作成する手順について説明します。

- テーブル V: クロスプレミス仮想ネットワークの構成
- テーブル S: 仮想ネットワークのサブネット
- テーブル D: オンプレミス DNS サーバー
- テーブル L: ローカル ネットワークのアドレス プレフィックス

これらのテーブルは通常、Azure 内の IT ワークロードの構成を説明するトピックで設定され、クロスプレミス仮想ネットワークを含みます。例については、「[フェーズ 1: Azure を構成する](virtual-machines-workload-intranet-sharepoint-phase1.md)」を参照してください。

次の手順では、これらのテーブルの情報を参考にして、仮想ネットワークの構成プロセスの手順を説明します。別のトピックでこれらのテーブルの設定をまだ指定していない状態で、クロスプレミス仮想ネットワークを構成する場合は、「[Azure 仮想ネットワークへのクロスプレミス サイト間接続の構成](../vpn-gateway/vpn-gateway-site-to-site-create.md)」を参照してください。

> [AZURE.NOTE]この手順では、サイト間 VPN 接続を使用する仮想ネットワークを作成します。サイト間接続に Azure ExpressRoute を使用する方法については、「[ExpressRoute の技術概要](../expressroute/expressroute-introduction.md)」を参照してください。

## 構成テーブルの設定を使用して新しいクロスプレミス Azure Virtual Network を作成する

1. [Azure ポータル](https://manage.windowsazure.com/)にサインインします。
2. タスク バーから、**[新規]、[Network Services]、[Virtual Network ]、[カスタム作成]** の順にクリックします。
3. **[Virtual Network の詳細]** ページが表示されます。
	- **[名前]** に、テーブル V の項目 1 の名前を入力します。
	- **[場所]** で、テーブル V の項目 2 のリージョンを選択します。
4. 矢印をクリックして次へ進みます。
5. **[DNS サーバーと VPN 接続]** ページが表示されます。
	- **[DNS サーバー]** で、テーブル D の各エントリについて、オンプレミス DNS サーバーのフレンドリ名と IP アドレスを構成します。
	- **[サイト間接続]** で、**[サイト間 VPN の構成]** を選択します。
	- ローカル ネットワークを既に構成してあり、それを使用する場合は、**[ローカル ネットワーク]** でその名前を選択します。新しいローカル ネットワークを作成する場合は、**[ローカル ネットワーク]** で **[新しいローカル ネットワークを指定する]** を選択します。
	- サブスクリプション用のローカル ネットワークをまだ構成していない場合、**[ローカル ネットワーク]** フィールドは表示されません。
6. 矢印をクリックして次へ進みます。
7. **[サイト間接続]** ページが表示されます (手順 5. で **[新しいローカル ネットワークを指定する]** を選択した場合)。
	- **[名前]** に、テーブル V の項目 3 の名前を入力します (ローカル ネットワーク名)。
	- **[VPN デバイスの IP アドレス]** に、テーブル V の項目 4 のアドレスを入力します。
	- **[アドレス空間]** では、テーブル L の各エントリについて、プレフィックスに関する組織ネットワークの IP アドレス空間 (**[開始 IP]**) およびプレフィックスの長さ (**[CIDR (アドレス数)]**) を入力します。
8. 矢印をクリックして次へ進みます。
9. **[Virtual Network アドレス空間]** ページで、次の内容を構成します。
	- **[アドレス空間]** で、テーブル V の項目 5 から得られる仮想ネットワークのプライベート IP アドレス空間の、プレフィックス (**[開始 IP]**) とプレフィックスの長さ (**[CIDR (アドレス数)]**) を入力します。
	- **[サブネット]** では、テーブル S の各エントリについて:
		- サブネットの名前を **[サブネット]** 列に入力します。必要な場合は、既定の名前を上書きします。
		- サブネットのプライベート IP アドレス空間の、プレフィックス (**[開始 IP]**) とプレフィックスの長さ (**[CIDR (アドレス数)]**) を入力します。
	- **[ゲートウェイ サブネットの追加]** をクリックします。
10. チェック マークをクリックして、構成を完了します。

## その他のリソース

[Virtual Network の概要](../virtual-network/virtual-networks-overview.md)

[仮想ネットワークの構成タスク](../documentation/services/virtual-machines/)

[Azure Virtual Network へのクロスプレミスのサイト間接続の構成](../vpn-gateway/vpn-gateway-site-to-site-create.md)

<!---HONumber=Oct15_HO4-->