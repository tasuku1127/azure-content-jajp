<properties 
   pageTitle="プレビュー ポータルを使用したインターネットに接続するクラシック デプロイメント モデルのロード バランサーの作成の開始 | Microsoft Azure"
   description="プレビュー ポータルを使用し、インターネットに接続するクラシック デプロイメント モデルのロード バランサーを作成する方法について説明します"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carolz"
   editor=""
   tags="azure-service-management"
/>
<tags  
   ms.service="load-balancer"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="11/03/2015"
   ms.author="joaoma" />

#ポータル ビューを使用したインターネットに接続するロード バランサー (クラシック) の作成の開始

[AZURE.INCLUDE [load-balancer-get-started-internet-classic-selectors-include.md](../../includes/load-balancer-get-started-internet-classic-selectors-include.md)]

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]この記事では、クラシック デプロイメント モデルについて説明します。[Azure リソース マネージャー PowerShell を使用してロード バランサーの作成を開始](load-balancer-get-started-internet-arm-ps.md)することもできます。
 
[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]



## プレビュー ポータルを使用したロード バランサーのエンドポイントの作成	

プレビュー ポータルを使用してインターネットに接続するロード バランサー (クラシック) デプロイメント モデルを作成するには、次の手順に従います。

1. ブラウザーから http://portal.azure.com に移動し、必要に応じて Azure アカウントでサインインします。

2. [仮想マシン (クラシック)] ブレードに進み、仮想マシンを選択します。

3. [仮想マシン "基本情報"] ブレードで [すべての設定] を選択します。

4. [負荷分散セット] をクリックします。

5. [負荷分散セット] ブレード上の [結合] アイコンをクリックし、新しいロード バランサーを作成します。

6. インターネットに接続するロード バランサーでは [負荷分散セット タイプ] に [パブリック] を選択します。

7. [必要な設定の構成] をクリックし、[負荷分散セットの選択] を開き、[負荷分散セットの作成] をクリックします。

8. [負荷分散セットの作成] ブレードでロード バランサー セットの名前を作成します。名前、パブリック ポート、プローブ プロトコル、プローブ ポートを入力します。

9. 必要に応じてプローブ間隔と再試行を変更します。

10. (省略可能) 必要に応じてロード バランサー セットの作成ブレードからアクセス制御規則を構成できます。

11. [OK] をクリックし、[負荷分散セットの結合] ブレードに戻ります。

12. [OK] をクリックし、[ロード バランサー セット] ブレードに新しいロード バランサー リソースが表示されるのを待ちます。
 
## 次のステップ

[内部ロード バランサーの構成の開始](load-balancer-internal-getstarted.md)

[ロード バランサー分散モードの構成](load-balancer-distribution-mode.md)

[ロード バランサーのアイドル TCP タイムアウト設定の構成](load-balancer-tcp-idle-timeout.md)

<!---HONumber=Nov15_HO3-->