<properties 
	pageTitle="エンタープライズ Web アプリの Azure App Service への移行" 
	description="Web Apps 移行アシスタントを使用して既存の IIS Web サイトを Azure App Service Web Apps に手軽に移行する方法について説明します。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="cephalin" 
	writer="cephalin" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/24/2015" 
	ms.author="cephalin"/>

# エンタープライズ Web アプリの Azure App Service への移行

インターネット インフォメーション サービス (IIS) 6 以降で実行されている既存の Web サイトを簡単に [App Service Web Apps](http://go.microsoft.com/fwlink/?LinkId=529714) に移行できます。[Web Apps Migration Assistant](https://www.movemetothecloud.net/) では、IIS サーバーのインストールを分析し、どのサイトを App Service に移行できるかを識別し、移行できない要素またはプラットフォームでサポートされていない要素を強調表示してから、Web サイトおよび関連するデータベースを Azure に移行できます。

>[AZURE.NOTE]Windows Server 2003 は、2015 年 7 月 14 日でサポートが終了します。現在 Windows Server 2003 の IIS サーバーに Web サイトが置かれている場合、Web Apps により、低リスク、低コスト、および低干渉で Web サイトがオンラインの状態に保たれます。また、Web Apps Migration Assistant により移行プロセスが自動化されます。

## 互換性の分析中に検証される要素 ##
Migration Assistant により対応状況レポートが作成されます。このレポートを使用して、考えられる問題点を識別し、内部設置型の IIS から Azure App Service Web Apps への移行を妨げる可能性のある問題点をブロックします。注意すべき主な項目のいくつかを次に示します。

-	ポートのバインド – Web Apps でサポートされるポートは、HTTP トラフィック用のポート 80、および HTTPS トラフィック用のポート 443 のみです。それ以外のポート構成は無視され、トラフィックは 80 または 443 にルーティングされます。 
-	認証 - Web Apps では、匿名認証が既定でサポートされていますが、アプリケーションによってフォーム認証が指定される場合もあります。Azure Active Directory および ADFS と統合する場合に限り、Windows 認証を使用できます。基本認証など、その他すべての認証形式は現在サポートされていません。 
-	グローバル アセンブリ キャッシュ (GAC) – GAC は Web Apps でサポートされていません。アプリケーションで、通常 GAC にデプロイするアセンブリを参照する場合は、Web Apps でアプリケーションの bin フォルダーにデプロイする必要があります。 
-	IIS5 互換性モード – Web Apps でサポートされていません。 
-	アプリケーション プール – Web Apps では、各サイトとその子アプリケーションが同じアプリケーション プールで実行されます。サイトに複数のアプリケーション プールを利用する子アプリケーションが複数ある場合は、共通の設定で 1 つのアプリケーション プールに統合するか、または各アプリケーションを個別の Web アプリに移行します。
-	COM コンポーネント – Web Apps では、プラットフォームで COM コンポーネントを登録できません。Web サイトまたはアプリケーションで COM コンポーネントを使用する場合、その COM コンポーネントをマネージ コードで書き換えて、Web サイトまたはアプリケーションにデプロイする必要があります。
-	ISAPI フィルター – Web Apps では、ISAPI フィルターの使用をサポートできます。次の手順を実行する必要があります。
	-	Web アプリで DLL をデプロイします。 
	-	[Web.config](http://www.iis.net/configreference/system.webserver/isapifilters) を使用して DLL を登録します。
	-	次の内容を含むサイト ルートに、applicationHost.xdt ファイルを配置します。

			<?xml version="1.0"?>
			<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
			<configSections>
			    <sectionGroup name="system.webServer">
			      <section name="isapiFilters" xdt:Transform="SetAttributes(overrideModeDefault)" overrideModeDefault="Allow" />
			    </sectionGroup>
			  </configSections>
			</configuration>

		Web サイトで XML ドキュメントの変換を使用する方法の例については、「[Microsoft Azure Web サイトの変換](http://blogs.msdn.com/b/waws/archive/2014/06/17/transform-your-microsoft-azure-web-site.aspx)」を参照してください。

-	SharePoint、Front Page Server Extensions (FPSE)、FTP、SSL 証明書など、その他のコンポーネントは移行されません。

## Azure Apps Migration Assistant の使用方法 ##
このセクションでは、SQL Server データベースを使用して、内部設置型の Windows Server 2003 R2 (IIS 6.0) マシンで実行されている、いくつかの Web サイトを移行する場合の例を紹介します。

1.	IIS サーバーまたはクライアント コンピューターで、[https://www.movemetothecloud.net/](https://www.movemetothecloud.net/) に移動します。 

	![](./media/web-sites-migration-from-iis-server/migration-tool-homepage.png)

2.	[**専用 IIS サーバー**] ボタンをクリックして、Web Apps Migration Assistant をインストールします。近日中にオプションが追加される予定です。
4.	コンピューターに Web Apps Migration Assistant をインストールするには、[**ツールのインストール**] をクリックします。

	![](./media/web-sites-migration-from-iis-server/install-page.png)

	>[AZURE.NOTE]また、[**オフライン インストール用のダウンロード**] をクリックして ZIP ファイルをダウンロードし、インターネットに接続していないサーバーにインストールすることもできます。または、[**既存の移行準備レポートのアップロード**] をクリックすることもできます。これは、以前に生成した既存の移行準備レポート (後述) を使用する場合の高度なオプションです。

5.	[**アプリケーションのインストール**] 画面で [**インストール**] をクリックして、アプリケーションをコンピューターにインストールします。必要に応じて、Web Deploy、DacFX、IIS などの対応する依存関係もインストールされます。

	![](./media/web-sites-migration-from-iis-server/install-progress.png)

	インストールが完了すると、Web Apps Migration Assistant が自動的に開始されます。
  
6.	[**リモート サーバーから Azure にサイトおよびデータベースを移行する**] を選択します。リモート サーバーの管理者の資格情報を入力し、[**続行**] をクリックします。

	![](./media/web-sites-migration-from-iis-server/migrate-from-remote.png)

	もちろんローカル サーバーから移行することもできます。実稼働環境の IIS サーバーから Web サイトを移行する場合は、リモート オプションが役立ちます。
 
	この時点で移行ツールにより、サイト、アプリケーション、アプリケーション プール、依存関係など、IIS サーバーの構成を確認し、移行対象となる Web サイトを識別します。

8.	次のスクリーンショットは、**既定の Web サイト**、**TimeTracker**、および **CommerceNet4** の 3 つの Web サイトを示しています。これらすべてのサイトに、移行するデータベースが関連付けられています。評価するサイトをすべて選択し、[**次へ**] をクリックします。

	![](./media/web-sites-migration-from-iis-server/select-migration-candidates.png)
 
9.	[**アップロード**] をクリックし、対応状況レポートをアップロードします。[**ファイルをローカルに保存**] をクリックすると、前述したように、移行ツールを後でもう一度実行し、保存されている対応状況レポートをアップロードできます。

	![](./media/web-sites-migration-from-iis-server/upload-readiness-report.png)
 
	対応状況レポートをアップロードした後、Azure で対応状況の分析が実行され、結果が表示されます。各 Web サイトの評価の詳細を確認し、すべての問題について理解しているか、または処理する前にすべての問題点が解決していることを確認します。
 
	![](./media/web-sites-migration-from-iis-server/readiness-assessment.png)

12.	[**移行の開始**] をクリックして移行を開始します。これで Azure にリダイレクトされ、自分のアカウントにログインできるようになります。アクティブな Azure サブスクリプションを持つアカウントでログインすることが重要です。Azure アカウントがない場合は、無料評価版でサインアップできます。

13.	移行後の Azure Web アプリとデータベースに対して使用するテナント アカウント、Azure サブスクリプション、およびリージョンを選択し、[**移行を開始する**] をクリックします。移行する Web サイトは後で選択することもできます。

	![](./media/web-sites-migration-from-iis-server/choose-tenant-account.png)

14.	次の画面では、次に示すように、既定の移行設定を変更できます。

	- 既存の Azure SQL データベースを使用するか、または新しい Azure SQL データベースを作成し、その資格情報を構成する
	- 移行する Web サイトを選択する
	- Azure Web アプリの名前と、リンクされた SQL データベースを定義する
	- グローバルな設定とサイト レベルの設定をカスタマイズする

	次のスクリーンショットは、既定の設定による移行対象として選択されたすべての Web サイトを示しています。

	![](./media/web-sites-migration-from-iis-server/migration-settings.png)

	>[AZURE.NOTE]カスタム設定で [**Azure Active Directory を有効にする**] チェック ボックスをオンにすると、Azure Web アプリが [Azure Active Directory](active-directory-whatis.md) (**既定のディレクトリ**) に統合されます。内部設置型 Active Directory と Azure Active Directory の同期の詳細については、「[ディレクトリ統合](http://msdn.microsoft.com/library/jj573653)」を参照してください。

16.	 必要な変更をすべて行ったら、[**作成**] をクリックして移行プロセスを開始します。移行ツールにより、Azure SQL データベースと Azure Web アプリが作成され、Web サイトのコンテンツとデータベースが発行されます。移行の進行状況が移行ツールで明確に表示され、最後に概要のページが表示されます。このページには、移行されたサイトの詳細情報、移行が正しく行われたかどうか、新しく作成された Azure Web アプリへのリンクなどが表示されます。

	移行中にエラーが発生した場合、移行ツールでエラーの詳細が明確に表示され、変更がロールバックされます。[**エラー報告を送信する**] をクリックすると、キャプチャされたエラーのコール スタックとメッセージ本文と共に、エンジニアリング チームに直接エラー レポートを送信できます。

	![](./media/web-sites-migration-from-iis-server/migration-error-report.png)

	移行が問題なく完了した場合は、[**フィードバックの送信**] をクリックして直接フィードバックを提供することもできます。
 
20.	Azure Web アプリへのリンクをクリックし、移行が完了したことを確認します。

21. これで、移行した Web アプリを Azure App Service で管理できるようになりました。管理を行うには、[Azure ポータル](http://go.microsoft.com/fwlink/?LinkId=529715)にログインします。

22. Azure ポータルで、[Web Apps] ブレードを開き、移行した Web サイト (Web アプリとして表示される) を確認します。いずれかをクリックして Web アプリの管理を開始します。継続的発行の構成、バックアップの作成、自動スケーリング、使用状況やパフォーマンスの監視などを行うことができます。

	![](./media/web-sites-migration-from-iis-server/TimeTrackerMigrated.png)

>[AZURE.NOTE]Azure アカウントにサインアップする前に Azure App Service の使用を開始する場合は、「[App Service の試用](http://go.microsoft.com/fwlink/?LinkId=523751)」を参照してください。そこでは、App Service で有効期間の短いスターター Web アプリをすぐに作成できます。このサービスの利用にあたり、クレジット カードは必要ありません。契約も必要ありません。

## 変更内容
* Web サイトから App Service への変更ガイドについては、「[Azure App Service および既存の Azure サービスへの影響](http://go.microsoft.com/fwlink/?LinkId=529714)」を参照してください。
* 古いポータルから新しいポータルへの変更ガイドについては、「[プレビュー ポータル内の移動に関するリファレンス](http://go.microsoft.com/fwlink/?LinkId=529715)」を参照してください。

<!--HONumber=54--> 