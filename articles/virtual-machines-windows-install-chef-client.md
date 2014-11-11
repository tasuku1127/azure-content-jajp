<properties title="About Chef and Azure Virtual Machines" pageTitle="About Chef and Azure Virtual Machines" description="Describes installing and configuring Chef on a VM in Azure" metaKeywords="" services="virtual machines" solutions="" documentationCenter="" authors="kathydav" videoId="" scriptId="" />

<tags ms.service="virtual-machines" ms.workload="infrastructure-services" ms.tgt_pltfrm="vm-windows" ms.devlang="na" ms.topic="article" ms.date="01/01/1900" ms.author="kathydav" />

# Chef と Azure Virtual Machines について

Chef は、インフラストラクチャの構築、デプロイ、管理を行う自動化システムです。リソースの管理は、レシピという再利用可能な定義を使用して、Web サーバー構成などのタスクに関する指示を出すことで実行します。

Chef はクライアント サーバー システムです。Chef サーバーを使用するためのオプションについては、「[Choose your installation (インストールの種類を選択してください)][Choose your installation (インストールの種類を選択してください)]」を参照してください。クライアントを設定するには、Chef サーバーについての情報が必要です。Azure 仮想マシンに Chef クライアントをインストールするには、以下の方法を利用します。

-   管理ポータルを使用して、Windows Server 2012 または Windows Server 2012 R2 を実行する仮想マシンを作成する際に Chef クライアントをインストールする。手順については、「[Microsoft Azure Portal (Microsoft Azure ポータル)][Microsoft Azure Portal (Microsoft Azure ポータル)]」を参照してください。
-   Windows PowerShell を使用して、既存の仮想マシンに Chef クライアントをインストールする。Github でサンプル [スクリプト][スクリプト]が公開されています。
-   [knife-azure][knife-azure] という Chef プラグインを使用して、仮想マシンのインスタンスを作成し、Chef クライアントをインストールする。

## その他のリソース

[Chef and Microsoft Azure (Chef と Microsoft Azure)][Chef and Microsoft Azure (Chef と Microsoft Azure)]

[Windows Server が実行されている仮想マシンにログオンする方法][Windows Server が実行されている仮想マシンにログオンする方法]

[Linux を実行する仮想マシンにログオンする方法][Linux を実行する仮想マシンにログオンする方法]

[拡張機能の管理][拡張機能の管理]

<!--Link references-->

  [スクリプト]: https://gist.github.com/kaustubh-d/cea1aa75baebd3615609
  [knife-azure]: http://docs.getchef.com/plugin_knife_azure.html
  [Windows Server が実行されている仮想マシンにログオンする方法]: ../virtual-machines-log-on-windows-server/
  [Linux を実行する仮想マシンにログオンする方法]: ../virtual-machines-linux-how-to-log-on
  [拡張機能の管理]: http://go.microsoft.com/fwlink/p/?linkid=390493&clcid=0x409