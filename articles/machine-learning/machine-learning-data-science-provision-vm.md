<properties 
	pageTitle="データ サイエンス仮想マシンのプロビジョニング | Microsoft Azure" 
	description="分析と機械学習を行うために、Azure でデータ サイエンス用仮想マシンを構成および作成します。" 
	services="machine-learning" 
	documentationCenter="" 
	authors="bradsev" 
	manager="paulettm" 
	editor="cgronlun" />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="11/04/2015" 
	ms.author="bradsev" />


# データ サイエンス用仮想マシンのプロビジョニング

## はじめに

データ サイエンス用仮想マシンは、プレインストールの Azure 仮想マシン (VM) イメージです。データ分析と機械学習用に一般的に使用されているいくつかのツールで構成されています。含まれているツールは、次のとおりです。

- Revolution R Open
- Anaconda Python ディストリビューション
- Visual Studio Community エディション
- Power BI Desktop
- SQL Server Express エディション
- Azure SDK


データ サイエンスでは、一連のタスクの反復処理を行います。たとえば、データの検索、読み込み、および前処理、モデルの構築とテスト、インテリジェント アプリケーションで使用するためのモデルのデプロイなどです。これらのタスクを実行するために、さまざまなツールを使用するデータ科学者も珍しくありません。適切なバージョンのソフトウェアを見つけ、ダウンロードしてインストールするには、非常に時間がかかる場合があります。この負担をデータ サイエンス VM で軽減することができます。

データ サイエンス VM は分析プロジェクトをすぐに開始します。これにより、R、Python、SQL、および C# を含むさまざまな言語でタスクを処理できます。Visual Studio には、コードの開発やテストを行うための使いやすい IDE が用意されています。VM に含まれている Azure SDK を使用すると、Microsoft のクラウド プラットフォームにあるさまざまなサービスを利用してアプリケーションを構築できます。


## 前提条件

Azure VM を作成するには、以下のものが必要です。

- **Azure サブスクリプション**: これを取得する場合は、[Azure 無試用価版の取得](http://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)に関するページを参照してください。

*   **Azure ストレージ アカウント**: これを作成する場合は、[Azure ストレージ アカウントの作成](storage-whatis-account.md)に関するページを参照してください。既存のアカウントを使用しない場合は、VM を作成する手順の一環としてストレージ アカウントを作成することもできます。


## データ サイエンス用 VM の作成

データ サイエンス用 VM のインスタンスを作成する手順を以下に示します。

1.	[Azure ポータル](https://ms.portal.azure.com/)にログインします。
2.	**[新規作成]**、**[Compute]**、**[Marketplace]** の順に移動し、*[データ サイエンス用仮想マシン]* を検索します。![provision-data-science-vm](./media/machine-learning-data-science-provision-vm/provision-data-science-virtual-machine.png)
3.	Microsoft 発行の "Data Science Virtual Machine" という名前の VM を選択し、Data Science Virtual Machine の説明のパネルを表示します。 	
4.	 下部にある **[作成]** ボタンをクリックして、ウィザードを起動します。![configure-data-science-vm](./media/machine-learning-data-science-provision-vm/configure-data-science-virtual-machine.png)
5.	 以下のセクションでは、データ サイエンス用 VM を作成するために使用されるウィザードの **5 つのステップ** (上の図の右に列挙してあります) の各ステップでの**入力**について説明します。以下は、これらの各ステップを構成するために必要な入力項目です。

 **1.基本**:

- **名前**: 作成するデータ サイエンス サーバーの名前です。
- **ユーザー名**: 管理者アカウントのログイン ID です。
- **パスワード**: 管理者アカウントのパスワードです。
- **サブスクリプション**: 複数のサブスクリプションがある場合は、マシンが作成されて課金されるサブスクリプションを選択します。
- **リソース グループ**: 新しいリソース グループを作成することも、既存のグループを使用することもできます。
- **場所**: 最適なデータ センターを選択します。通常は、ネットワーク アクセスを最速にするために、データの大部分があるか、物理的に最も近いデータ センターを選びます。

 **2.サイズ**:

- 機能の要件とコストの制約を満たしている、いずれかのサーバーの種類を選択します。[すべて表示] を選択すると、他の VM のサイズも表示されます。

 **3.設定**

- **ディスクの種類**: ソリッド ステート ドライブ (SSD) を使用する場合は、[Premium] を選択します。それ以外の場合は、[Standard] を選択します。
- **ストレージ アカウント**: サブスクリプションに新しい Azure ストレージ アカウントを作成することも、ウィザードの [基本] ステップで選択したのと同じ*場所*の既存のストレージ アカウントを使用することもできます。
- **他のパートナー**: ほとんどの場合、既定値を使用します。既定以外の値の使用を検討する場合は、情報リンクにポインターを合わせ、該当するフィールドのヘルプを表示できます。

 **4.まとめ**:

- 入力したすべての情報が正しいことを確認します。

 **5.購入**:

- プロビジョニングを開始するには、**[購入]** をクリックします。取引条件へのリンクが用意されています。**[サイズ]** ステップで選択したサーバー サイズのコンピューティング以外の追加料金が VM で発生することはありません。 


プロビジョニングには、10 ～ 20 分くらいかかります。プロビジョニングの状態は、Azure ポータルに表示されます。

## データ サイエンス用 VM へのアクセス方法

VM が作成されたら、リモート デスクトップを使用して、ステップ 4. の [基本] セクションで作成した管理者アカウントの資格情報でログインできます。

VM が作成され、プロビジョニングされた後は、VM にインストールされて構成されたツールを使い始めることができます。多くのツールには、デスクトップ アイコンとスタート メニュー タイルがあります。

## データ サイエンス用 VM にインストールされているツール

### R
分析に R を使用する場合のために、VM には Revolution R Open (RRO) がインストールされています。これは、R のオープン ソース ディストリビューションであり、CRAN-R との完全な互換性があります。最新のオープン ソース R エンジンと、Intel Math Kernel Library が含まれています。VM には、"RRO RGui" と呼ばれる IDE もパッケージ化されています。[RStudio](http://www.rstudio.com) のような他の IDE をダウンロードして使用することもできます。

### Python
Python を使用して開発するために、Anaconda Python ディストリビューション 2.7 がインストールされています。このディストリビューションには、基本 Python と、約 300 の最も一般的な数学、エンジニアリング、およびデータ分析パッケージが含まれています。IDLE や Spyder のような、Anaconda にバンドルされている IDE を使用できます。検索バーで検索して (**Win** + **S** キー)、いずれかの IDE を起動できまます。

### IPython Notebook
Anaconda ディストリビューションは、コードと分析を共有するための環境である IPython Notebook にも付属しています。IPython Notebook サーバーは、あらかじめ構成されています。Notebook サーバーにアクセスするためのブラウザーを起動するデスクトップ アイコンがあります。リモート デスクトップを通じて VM を利用している場合は、IPython Notebook サーバーにアクセスするために、[https://localhost:9999/](https://localhost:9999/) を使用することもできます (注: 証明書の警告が表示されても続行します)。

### Visual Studio 2015 Community エディション
Visual Studio Community エディションが VM にインストールされています。これは、Microsoft の人気のある IDE の無料版で、評価用または非常に小規模なチーム用に使用できます。ライセンス条項は、[ここ](https://www.visualstudio.com/support/legal/mt171547)で確認することができます。Visual Studio を開くには、デスクトップ アイコンをダブルクリックするか、**[スタート]** メニューを使用します。**Win** + **S** キーを押し、「Visual Studio」と入力することで、プログラムを検索することもできます。

注: 評価期間が終了したことを示すメッセージが表示される場合があります。Visual Studio Community Edition にアクセスするには、Microsoft アカウント資格情報を入力するか、Microsoft アカウント資格情報を作成して、それを入力します。C#、Python などの言語でプロジェクトを作成できます。

### SQL Server Express
SQL Server の制限付きバージョンも、Visual Studio Community エディションにパッケージ化されています。SQL Server には、**SQL Server Management Studio** を起動してアクセスできます。VM の名前は、サーバー名として設定されます。Windows の管理者としてログインする場合は、Windows 認証を使用します。SQL Server Management Studio にアクセスできたら、他のユーザーの作成、データベースの作成、データのインポート、SQL クエリの実行などを行うことができます。

### Azure 
いくつかの Azure ツールが VM にインストールされています。- Azure SDK ドキュメントにアクセスするためのデスクトップ ショートカットがあります。- **AzCopy** は、Microsoft Azure ストレージ アカウントとの間でデータを移動するために使用されます。- **Azure ストレージ エクスプローラー**は、Azure ストレージ アカウントに格納されているオブジェクトを参照するために使用されます。- **Microsoft Azure Powershell**。- スクリプト言語で Azure リソースを管理するために使用されるスクリプト言語も、VM にインストールされています。

###Power BI

ダッシュボードの構築と高度な視覚化を支援するために、**Power BI Desktop** がインストールされています。このツールを使用すると、他のソースからデータを抽出し、ダッシュボードとレポートを作成し、それらをクラウドに公開することができます。詳細については、[Power BI](http://powerbi.microsoft.com) サイトを参照してください。

注: Power BI にアクセスするには、Office 365 アカウントが必要です。

## その他の Microsoft 開発ツール
[**Microsoft Web Platform Installer**](https://www.microsoft.com/web/downloads/platform.aspx) は、他の Microsoft 開発ツールを検出し、ダウンロードするために使用できます。VM デスクトップには、ツールへのショートカットもあります。

<!---HONumber=Nov15_HO2-->