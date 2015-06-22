<properties 
	pageTitle="ハイブリッド接続を使用して Azure App Service の Web アプリから内部設置型の SQL Server に接続する" 
	description="Microsoft Azure で Web アプリを作成し、それを内部設置型の SQL Server データベースに接続します。"
	services="app-service\web" 
	documentationCenter="" 
	authors="cephalin" 
	manager="wpickett" 
	editor="mollybos"/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/23/2015" 
	ms.author="cephalin"/>

# ハイブリッド接続を使用して Azure App Service の Web アプリから内部設置型の SQL Server に接続する

ハイブリッド接続により、[Azure App Service](http://go.microsoft.com/fwlink/?LinkId=529714) Web Apps を、静的 TCP ポートを使用する内部設置型リソースに接続できます。サポートされているリソースには、Microsoft SQL Server、MySQL、HTTP Web APIs、Mobile Services、およびほとんどのカスタム Web サービスが含まれます。

このチュートリアルでは、[Azure プレビュー](http://go.microsoft.com/fwlink/?LinkId=529715)での App Service Web アプリの作成方法、新しいハイブリッド接続機能を使用したローカルの内部設置型 SQL Server データベースへの Web アプリの接続方法、ハイブリッド接続を使用する単純な ASP.NET アプリケーションの作成方法、App Service Web アプリへのアプリケーションのデプロイ方法について説明します。完成した Azure の Web アプリでは、ユーザー資格情報を内部設置型のメンバーシップ データベースに保存します。このチュートリアルは、Azure または ASP.NET を使用した経験がない読者を対象に作成されています。

>[AZURE.NOTE]Azure アカウントにサインアップする前に Azure App Service の使用を開始したい場合は、[App Service の試用](http://go.microsoft.com/fwlink/?LinkId=523751)に関するページをご覧ください。このページでは、App Service で有効期間の短いスターター Web アプリをすぐに作成できます。このサービスの利用にあたり、クレジット カードは必要ありません。契約も必要ありません。

> [AZURE.NOTE]ハイブリッド接続機能の Web Apps 部分は、[Azure プレビュー ポータル](https://portal.azure.com)でのみ使用できます。BizTalk Services で接続を作成するには、「[Hybrid Connections (ハイブリッド接続)](http://go.microsoft.com/fwlink/p/?LinkID=397274)」を参照してください。

## 前提条件 ##

このチュートリアルを完了するには、以下の製品が必要です。すべて無料版を利用できるため、Azure の開発を完全に無料で始めることができます。

- **Azure サブスクリプション** - 無料サブスクリプションについては、「[1 か月間無料評価版](/pricing/free-trial/)」を参照してください。 

- **Visual Studio 2013** - Visual Studio 2013 の無料試用版のダウンロードについては、「[Visual Studio のダウンロード](http://www.visualstudio.com/downloads/download-visual-studio-vs)」を参照してください。これをインストールしてから、次に進みます。

- **Microsoft .NET Framework 3.5 Service Pack 1** - オペレーティング システムが Windows 8.1、Windows Server 2012 R2、Windows 8、Windows Server 2012、Windows 7、または Windows Server 2008 R2 の場合は、[コントロール パネル] > [プログラムと機能] > [Windows の機能の有効化または無効化] の順に選択してこれを実行できます。そうでない場合は、[Microsoft Download Center](http://www.microsoft.com/download/en/details.aspx?displaylang=en&id=22) からダウンロードできます。

- **SQL Server 2014 Express with Tools** - [Microsoft Web プラットフォーム データベースのページ](http://www.microsoft.com/web/platform/database.aspx)で Microsoft SQL Server Express の無料版をダウンロードします。**Express** (LocalDB ではない) 版を選択します。**Express with Tools** 版には、このチュートリアルで使用する SQL Server Management Studio が含まれています。

- **SQL Server Management Studio Express** - これは、上記の SQL Server 2014 Express with Tools のダウンロードに含まれていますが、個別にインストールする必要がある場合は、[SQL Server Express のダウンロードのページ](http://www.microsoft.com/web/platform/database.aspx)からダウンロードしてインストールすることができます。

このチュートリアルでは、Azure サブスクリプションを持っている、Visual Studio 2013 をインストールしている、および .NET Framework 3.5 をインストールしているか有効にしていると想定しています。このチュートリアルでは、Azure のハイブリッド接続機能 (静的 TCP ポートの既定のインスタンス) を使用する構成に SQL Server 2014 Express をインストールする方法を説明します。SQL Server をインストールしていない場合は、チュートリアルを開始する前に、上記の場所から SQL Server 2014 Express with Tools をダウンロードしてください。

### メモ ###
ハイブリッド接続で内部設置型の SQL Server または SQL Server Express のデータベースを使用するには、TCP/IP が静的ポートで有効になっている必要があります。SQL Server の既定のインスタンスは静的ポート 1433 を使用しますが、名前付きインスタンスは静的ポート 1433 を使用しません。

内部設置型の Hybrid Connection Manager のエージェントをインストールするコンピューターの条件は、次のとおりです。

- 次を使用して Azure に送信接続できること

> <table border="1">
    <tr>
       <th><strong>ポート</strong></th>
        <th>理由</th>
    </tr>
    <tr>
        <td>80</td>
        <td>証明書の検証用の HTTP ポート (<strong>必須</strong>)。必要に応じて、データ接続に使用することもできます。</td>
    </tr>
    <tr>
        <td>使用します</td>
        <td>データ接続用 (<strong>任意</strong>)。443 への送信接続ができない場合は、TCP ポート 80 を使用します。</td>
    </tr>
	<tr>
        <td>5671 と 9352</td>
        <td>データ接続用の<strong>推奨</strong>ポート。ただし、必須ではありません。このモードでは通常、スループットが高くなります。これらのポートへの送信接続ができない場合は、TCP ポート 443 を使用します。</td>
	</tr>
</table>

- 内部設置型リソースの *hostname*:*portnumber* に到達できること 

この記事の手順では、内部設置型のハイブリッド接続のエージェントをホストするコンピューターからブラウザーを使用していると想定しています。

上記の条件を満たす構成および環境に既に SQL Server をインストールしている場合は、この手順をスキップし、「[内部設置型の SQL Server を作成する](#CreateSQLDB)」から開始できます。

<a name="InstallSQL"></a>
## A.SQL Server Express をインストールし、TCP/IP を有効にして、内部設置型の SQL Server データベースを作成する ##

このセクションでは、Web アプリケーションが Azure プレビュー環境を使用するために、SQL Server Express をインストールし、TCP/IP を有効にして、データベースを作成する方法を説明します。

### SQL Server Express をインストールする ###

1. SQL Server Express をインストールするには、ダウンロードした **SQLEXPRWT_x64_ENU.exe** または **SQLEXPR_x86_ENU.exe** ファイルを実行します。SQL Server インストール センター ウィザードが表示されます。
	
	![SQL Server Install][SQLServerInstall]
	
2. **[SQL Server の新規スタンドアロン インストールを実行するか、既存のインストールに機能を追加します]** をクリックします。指示に従って、**[インスタンスの構成]** ページが表示されるまで、既定の選択と設定を使用します。
	
3. **[インスタンスの構成]** ページで、**[既定のインスタンス]** を選択します。
	
	![Choose Default Instance][ChooseDefaultInstance]
	
	既定では、SQL Server の既定のインスタンスが静的ポート 1433 の SQL Server クライアントから要求をリッスンします。ハイブリッド接続機能には静的ポート 1433 が必要です。名前付きインスタンスは動的ポートと UDP を使用します。これはハイブリッド接続ではサポートされません。
	
4. **[サーバーの構成]** ページでは既定値をそのまま使用します。
	
5. **[データベース エンジンの構成]** ページの **[認証モード]** で、**[混合モード (SQL Server 認証と Windows 認証)]** を選択して、パスワードを入力します。
	
	![Choose Mixed Mode][ChooseMixedMode]
	
	このチュートリアルでは、SQL Server 認証を使用します。後で必要になるため、入力したパスワードを忘れないでください。
	
6. ウィザードの残りの手順を実行し、インストールを完了します。

### TCP/IP を有効にする ###
TCP/IP を有効にするには、SQL Server Express をインストールした際にインストールされた SQL Server 構成マネージャーを使用します。次に進む前に、「[SQL Server の TCP/IP ネットワーク プロトコルの有効化](http://technet.microsoft.com/library/hh231672%28v=sql.110%29.aspx)」の手順に従ってください。

<a name="CreateSQLDB"></a>
### 内部設置型の SQL Server データベースを作成する ###

Visual Studio Web アプリケーションには、Azure がアクセスできるメンバーシップ データベースが必要です。これには、(既定で MVC テンプレートが使用する LocalDB データベースではなく) SQL Server または SQL Server Express データベースが必要なため、次にメンバーシップ データベースを作成します。

1. SQL Server Management Studio で、インストールした SQL Server に接続します(**[サーバーへの接続]** ダイアログが自動的に表示されない場合は、左パネルの **[オブジェクト エクスプローラー]** に移動し、**[接続]** をクリックして、**[データベース エンジン]** をクリックします)。![サーバーへの接続][SSMSConnectToServer]
	
	**[サーバーの種類]** には、**[データベース エンジン]** を選択します。**[サーバー名]** には、**[localhost]**、または使用しているコンピューターの名前を使用します。**[SQL Server 認証]** を選択し、前に作成したユーザー名とパスワードでログインします。
	
2. SQL Server Management Studio を使用して新しいデータベースを作成するには、オブジェクト エクスプローラーで **[データベース]** を右クリックしてから、**[新しいデータベース]** をクリックします。
	
	![Create new database][SSMScreateNewDB]
	
3. **[新しいデータベース]** ダイアログで、データベース名に「MembershipDB」と入力し、**[OK]** をクリックします。
	
	![Provide database name][SSMSprovideDBname]
	
	この時点では、データベースに変更を加えることはできないことに注意してください。メンバーシップ情報は、後で Web アプリケーションを実行すると、自動的に追加されます。
	
4. オブジェクト エクスプローラーで、**[データベース]** を展開すると、メンバーシップ データベースが作成されたことが示されます。
	
	![MembershipDB created][SSMSMembershipDBCreated]
	
<a name="CreateSite"></a>
## B.Azure プレビュー ポータルで Web アプリを作成する ##

> [AZURE.NOTE]このチュートリアルで使用する Web アプリを Azure プレビュー ポータルで既に作成している場合は、この手順をスキップし、「[ハイブリッド接続および BizTalk サービスを作成する](#CreateHC)」から続けてください。

1. [Azure プレビュー ポータル](https://portal.azure.com)で、**[新規]** > **[Web + モバイル]** > **[Web アプリ]** をクリックします。
	
	![New button][New]
	
2. Web アプリを構成し、**[作成]** をクリックします。
	
	![Website name][WebsiteCreationBlade]
	
3. しばらくすると、Web アプリケーションが作成され、Web アプリケーションのブレードが表示されます。ブレードは縦方向にスクロールできるダッシュボードで、Web アプリを管理できます。
	
	![Website running][WebSiteRunningBlade]
	
	Web アプリがライブかどうかを確認するには、**[参照]** アイコンをクリックして、既定のページを表示します。
	
次に、Web アプリケーションに対するハイブリッド接続と BizTalk サービスを作成します。

<a name="CreateHC"></a>
## C.ハイブリッド接続および BizTalk サービスを作成する ##

1. ポータルに戻り、Web アプリのブレードを下にスクロールして、**[ハイブリッド接続]** をクリックします。
	
	![Hybrid connections][CreateHCHCIcon]
	
2. [ハイブリッド接続] ブレードで、**[追加]** > **[新しいハイブリッド接続]** をクリックします。
	
3. **[ハイブリッド接続の作成] ブレード**で、次の手順を実行します。
	- **[名前]** に、接続の名前を入力します。
	- **[ホスト名]** には、SQL Server ホスト コンピューターのコンピューター名を入力します。
	- **[ポート]** には、1433 (SQL Server の既定のポート) を入力します。
	- **[BizTalk サービス]** をクリックし、BizTalk サービスの名前を入力します。
	
	![Create a hybrid connection][TwinCreateHCBlades]
		
5. **[OK]** を 2 回クリックします。

	処理が完了すると、**[通知]** 領域に緑色の "**SUCCESS**" という文字が点滅します。**[ハイブリッド接続]** ブレードには、状態が **[未接続]** の新しいハイブリッド接続が表示されます。
	
	![One hybrid connection created][CreateHCOneConnectionCreated]
	
これで、クラウド ハイブリッド接続のインフラストラクチャの重要な部分が完了しました。次に、対応する内部設置型の部分を作成します。

<a name="InstallHCM"></a>
## D.内部設置型の Hybrid Connection Manager をインストールして接続を完了する ##

1. **[ハイブリッド接続]** ブレードで、作成したハイブリッド接続をクリックし、**[リスナーのセットアップ]** をクリックします。
	
	![Click Listener Setup][ClickListenerSetup]
	
4. **[ハイブリッド接続のプロパティ]** ブレードが開きます。**内部設置型の Hybrid Connection Manager** で、**[インストールするにはここをクリックします]** をクリックします。
	
	![Click here to install][ClickToInstallHCM]
	
5. [アプリケーションの実行 - セキュリティの警告] ダイアログで、**[実行]** を選択します。
	
	![Choose Run to continue][ApplicationRunWarning]
	
6.	**[ユーザー アカウント制御]** ダイアログで、**[はい]** を選択します。
	
	![Choose Yes][UAC]
	
7. Hybrid Connection Manager がダウンロードされ、インストールされます。
	
	![Installing][HCMInstalling]
	
8. インストールが完了したら、**[閉じる]** をクリックします。
	
	![Click Close][HCMInstallComplete]
	
	**[ハイブリッド接続]** ブレードで、**[状態]** 列に **[接続]** と表示されています。
	
	![Connected Status][HCStatusConnected]

これで、ハイブリッド接続のインフラストラクチャが完成しました。この接続を使用する Web アプリケーションを作成できます。

<a name="CreateASPNET"></a>
## E.基本的な ASP.NET Web プロジェクトを作成し、データベース接続文字列を編集して、プロジェクトをローカルで実行する ##

### 基本的な ASP.NET プロジェクトを作成する ###
1. Visual Studio の **[ファイル]** メニューで、新しいプロジェクトを作成します。
	
	![New Visual Studio project][HCVSNewProject]
	
2. **[新しいプロジェクト]** ダイアログの **[テンプレート]** セクションで、**[Web]**、**[ASP.NET Web アプリケーション]** の順に選択して、**[OK]** をクリックします。
	
	![Choose ASP.NET Web Application][HCVSChooseASPNET]
	
3. **[新しい ASP.NET プロジェクト]** ダイアログで、**[MVC]** を選択し、**[OK]** をクリックします。
	
	![Choose MVC][HCVSChooseMVC]
	
4. プロジェクトが作成されると、アプリケーションの readme ページが表示されます。まだ Web プロジェクトを実行しないでください。
	
	![Readme page][HCVSReadmePage]

### アプリケーションのデータベース接続文字列を編集する ###

この手順では、ローカル SQL Server Express データベースの場所をアプリケーションに伝える接続文字列を編集します。接続文字列は、アプリケーションの Web.config ファイルにあり、アプリケーションの構成情報が含まれています。

> [AZURE.NOTE]アプリケーションで、Visual Studio の既定の LocalDB ではなく、SQL Server Express で作成したデータベースを使用するためには、プロジェクトを実行する前にこの手順を完了することが重要です。

1. ソリューション エクスプローラーで、Web.config ファイルをダブルクリックします。
	
	![web.config][HCVSChooseWebConfig]
	
2. **[connectionStrings]** セクションを編集して、ローカル コンピューターの SQL Server データベースを指すようにします。次の例の構文を使用します。
	
	![Connection string][HCVSConnectionString]
	
	接続文字列を作成する際には、次の事項に留意してください。
	
	- 既定のインスタンスではなく、名前付きインスタンス (YourServer\\SQLEXPRESS など) に接続している場合は、静的ポートを使用するように SQL Server を構成する必要があります。静的ポートの構成の詳細については、「[特定のポートでリッスンするように SQL Server を構成する方法](http://support.microsoft.com/kb/823938)」を参照してください。既定では、名前付きインスタンスは動的ポートと UDP を使用します。これはハイブリッド接続ではサポートされません。 
	
	- ポート (例に示すように既定では 1433) を接続文字列に指定することをお勧めします。これにより、ローカル SQL Server で TCP が有効になり、正しいポートが使用されます。
	
	- SQL Server 認証を使用して接続するのを忘れないでください。接続文字列にユーザー ID とパスワードを指定します。
	
3. Visual Studio で **[保存]** をクリックして、Web.config ファイルを保存します。

### プロジェクトをローカルで実行し、新しいユーザーを登録する ###

1. ここで、[デバッグ] の [参照] ボタンをクリックして、新しい Web プロジェクトをローカルで実行します。この例では、Internet Explorer を使用します。
	
	![Run project][HCVSRunProject]
	
2. 既定の Web ページの右上で、**[登録]** をクリックして新しいアカウントを登録します。
	
	![Register a new account][HCVSRegisterLocally]
	
3. ユーザー名とパスワードを入力します。
	
	![Enter user name and password][HCVSCreateNewAccount]
	
	これで、アプリケーションのメンバーシップ情報を保持するローカル SQL Server にデータベースが自動的に作成されます。テーブルの 1 つ (**dbo.AspNetUsers**) に、入力した Web アプリ ユーザー資格情報が保持されます。このテーブルは後でチュートリアルで使用されます。
	
4. 既定の Web ページのブラウザー ウィンドウを閉じます。これで、Visual Studio でアプリケーションが停止します。

これで、アプリケーションを Azure に発行してテストする次の手順に進む準備ができました。

<a name="PubNTest"></a>
## F.Web アプリケーションを Azure に発行してテストする ##

アプリケーションを App Service Web アプリに発行し、テストします。ローカル コンピューターのデータベースに Web アプリを接続する際に、前に構成したハイブリッド接続がどのように使用されるのかを確認します。

### Web アプリケーションを発行する ###

1. Azure プレビュー ポータルの App Service Web アプリに対する発行プロファイルをダウンロードできます。Web アプリのブレードで、**[発行プロファイルを取得]** をクリックし、コンピューターにファイルを保存します。
	
	![Download publish profile][PortalDownloadPublishProfile]
	
	次に、このファイルを Visual Studio Web アプリケーションにインポートします。
	
2. Visual Studio で、ソリューション エクスプローラーのプロジェクト名を右クリックし、**[発行]** を選択します。
	
	![Select publish][HCVSRightClickProjectSelectPublish]
	
3. **[Web の発行]** ダイアログの **[プロファイル]** タブで、**[インポート]** を選択します。
	
	![Import][HCVSPublishWebDialogImport]
	
4. ダウンロードした発行プロファイルを参照して、選択してから、**[OK]** をクリックします。
	
	![Browse to profile][HCVSBrowseToImportPubProfile]
	
5. 発行情報がインポートされ、ダイアログの **[接続]** タブに表示されます。
	
	![Click Publish][HCVSClickPublish]
	
	**[発行]** をクリックします。
	
	発行が完了すると、ブラウザーが起動し、見慣れた ASP.NET アプリケーションが表示されますが、Azure クラウドでライブになっているという点が異なります。

次に、ライブ Web アプリケーションを使用して、ハイブリッド接続の実行を確認します。

### 完成した Web アプリケーションを Azure でテストする ###

1. Azure の Web ページの右上で、**[ログイン]** を選択します。
	
	![Test log in][HCTestLogIn]
	
2. App Service Web アプリが、ローカル コンピューターの Web アプリケーションのメンバーシップ データベースに接続されています。これを確認するには、前にローカル データベースに入力した資格情報でログインします。
	
	![Hello greeting][HCTestHelloContoso]
	
3. 新しいハイブリッド接続を詳細にテストするには、Azure Web アプリケーションからログオフし、別のユーザーとして登録します。新しいユーザー名とパスワードを入力して、**[登録]** をクリックします。
	
	![Test register another user][HCTestRegisterRelecloud]
	
4. ハイブリッド接続を通じて新しいユーザーの資格情報がローカル データベースに保存されたことを確認するには、ローカル コンピューターで SQL Management Studio を開きます。オブジェクト エクスプローラーで、**[MembershipDB]** データベースを展開してから、**[テーブル]** を展開します。**[dbo.AspNetUsers]** メンバーシップ テーブルを右クリックし、**[先頭の 1000 行を選択]** を選択して、結果を確認します。
	
	![View the results][HCTestSSMSTree]
	
5. ローカル メンバーシップ テーブルには、ローカルで作成したメンバーシップと、Azure クラウドで作成したメンバーシップの両方が表示されています。クラウドで作成したメンバーシップは、Azure のハイブリッド接続機能で内部設置型のデータベースに保存されています。
	
	![Registered users in on-premises database][HCTestShowMemberDb]
	
これで、Azure クラウドの Web アプリと内部設置型の SQL Server データベース間のハイブリッド接続を使用する ASP.NET Web アプリケーションを作成し、デプロイすることができました。ご利用ありがとうございます。

## 関連項目 ##
[ハイブリッド接続の概要](http://go.microsoft.com/fwlink/p/?LinkID=397274)

[Josh Twist introduces hybrid connections (Josh Twist によるハイブリッド接続の紹介) (Channel 9 のビデオ)](http://channel9.msdn.com/Shows/Azure-Friday/Josh-Twist-introduces-hybrid-connections)

[ハイブリッド接続の概要](/services/biztalk-services/)

[BizTalk サービス: [ダッシュボード]、[監視]、[スケール]、[構成]、および [ハイブリッド接続] タブ](../biztalk-dashboard-monitor-scale-tabs/)

[Building a Real-World Hybrid Cloud with Seamless Application Portability (シームレスなアプリケーションの移植性を使用して実際のハイブリッド クラウドをビルドする) (Channel 9 のビデオ)](http://channel9.msdn.com/events/TechEd/NorthAmerica/2014/DCIM-B323#fbid=)

[ハイブリッド接続を使用して Azure Mobile Services から内部設置型の SQL Server に接続する](../mobile-services-dotnet-backend-hybrid-connections-get-started.md)

[Connect to an on-premises SQL Server from Azure Mobile Services using Hybrid Connections (ハイブリッド接続を使用して Azure Mobile Services から内部設置型の SQL Server に接続する) (Channel 9 のビデオ)](http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Connect-to-an-on-premises-SQL-Server-from-Azure-Mobile-Services-using-Hybrid-Connections)

[ASP.NET の ID の概要](http://www.asp.net/identity)

[AZURE.INCLUDE [app-service-web-whats-changed](../../includes/app-service-web-whats-changed.md)]
 

<!-- IMAGES -->
[SQLServerInstall]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A01SQLServerInstall.png
[ChooseDefaultInstance]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A02ChooseDefaultInstance.png
[ChooseMixedMode]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A03ChooseMixedMode.png
[SSMSConnectToServer]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A04SSMSConnectToServer.png
[SSMScreateNewDB]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A05SSMScreateNewDBlh.png
[SSMSprovideDBname]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A06SSMSprovideDBname.png
[SSMSMembershipDBCreated]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/A07SSMSMembershipDBCreated.png
[New]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B01New.png
[NewWebsite]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B02NewWebsite.png
[WebsiteCreationBlade]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B03WebsiteCreationBlade.png
[WebSiteRunningBlade]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B04WebSiteRunningBlade.png
[Browse]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B05Browse.png
[DefaultWebSitePage]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/B06DefaultWebSitePage.png
[CreateHCHCIcon]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C01CreateHCHCIcon.png
[CreateHCAddHC]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C02CreateHCAddHC.png
[TwinCreateHCBlades]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C03TwinCreateHCBlades.png
[CreateHCCreateBTS]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C04CreateHCCreateBTS.png
[CreateBTScomplete]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C05CreateBTScomplete.png
[CreateHCSuccessNotification]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C06CreateHCSuccessNotification.png
[CreateHCOneConnectionCreated]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/C07CreateHCOneConnectionCreated.png
[HCIcon]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D01HCIcon.png
[NotConnected]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D02NotConnected.png
[NotConnectedBlade]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D03NotConnectedBlade.png
[ClickListenerSetup]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D04ClickListenerSetup.png
[ClickToInstallHCM]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D05ClickToInstallHCM.png
[ApplicationRunWarning]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D06ApplicationRunWarning.png
[UAC]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D07UAC.png
[HCMInstalling]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D08HCMInstalling.png
[HCMInstallComplete]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D09HCMInstallComplete.png
[HCStatusConnected]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/D10HCStatusConnected.png
[HCVSNewProject]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E01HCVSNewProject.png
[HCVSChooseASPNET]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E02HCVSChooseASPNET.png
[HCVSChooseMVC]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E03HCVSChooseMVC.png
[HCVSReadmePage]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E04HCVSReadmePage.png
[HCVSChooseWebConfig]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E05HCVSChooseWebConfig.png
[HCVSConnectionString]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E06HCVSConnectionString.png
[HCVSRunProject]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E06HCVSRunProject.png
[HCVSRegisterLocally]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E07HCVSRegisterLocally.png
[HCVSCreateNewAccount]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/E08HCVSCreateNewAccount.png
[PortalDownloadPublishProfile]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F01PortalDownloadPublishProfile.png
[HCVSPublishProfileInDownloadsFolder]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F02HCVSPublishProfileInDownloadsFolder.png
[HCVSRightClickProjectSelectPublish]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F03HCVSRightClickProjectSelectPublish.png
[HCVSPublishWebDialogImport]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F04HCVSPublishWebDialogImport.png
[HCVSBrowseToImportPubProfile]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F05HCVSBrowseToImportPubProfile.png
[HCVSClickPublish]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F06HCVSClickPublish.png
[HCTestLogIn]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F07HCTestLogIn.png
[HCTestHelloContoso]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F08HCTestHelloContoso.png
[HCTestRegisterRelecloud]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F09HCTestRegisterRelecloud.png
[HCTestSSMSTree]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F10HCTestSSMSTree.png
[HCTestShowMemberDb]: ./media/web-sites-hybrid-connection-connect-on-premises-sql-server/F11HCTestShowMemberDb.png

<!--HONumber=54--> 