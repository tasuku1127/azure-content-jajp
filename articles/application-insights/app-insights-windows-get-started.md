<properties
	pageTitle="Windows Phone アプリとストア アプリ用の Application Insights | Microsoft Azure"
	description="Application Insights を使用して Windows デバイス アプリの使用状況とパフォーマンスを分析します。"
	services="application-insights"
    documentationCenter="windows"
	authors="alancameronwills"
	manager="douge"/>

<tags
	ms.service="application-insights"
	ms.workload="tbd"
	ms.tgt_pltfrm="ibiza"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.date="10/16/2015"
	ms.author="awills"/>

# Windows Phone アプリとストア アプリ用の Application Insights

*Application Insights はプレビュー段階です。*

[AZURE.INCLUDE [app-insights-selector-get-started](../../includes/app-insights-selector-get-started.md)]

Visual Studio Application Insights を使用すると、発行されたアプリケーションの次の内容を監視できます。

* [**使用状況**][windowsUsage] &#151; ユーザーの数やユーザーがアプリで操作している内容を確認できます。
* [**クラッシュ**][windowsCrash] &#151; クラッシュの診断レポートを取得し、ユーザーに与える影響を把握します。

![](./media/app-insights-windows-get-started/appinsights-d018-oview.png)

多くのアプリケーションの種類では、ほとんどの場合に通知されることなく、[Visual Studio によってアプリに Application Insights が追加されます](#ide)。ただし、この記事を読んで状況を把握できるように、ここでは手動による手順について説明します。

必要なものは次のとおりです。

* [Microsoft Azure][azure] のサブスクリプション
* Visual Studio 2013 以降

## 1\.Application Insights リソースの作成

[Azure ポータル][portal]で、Application Insights の新しいリソースを作成します。

![[新規]、[開発者向けサービス]、[Application Insights] の順に選択する](./media/app-insights-windows-get-started/01-new.png)

Azure の[リソース][roles]は、サービスのインスタンスです。このリソースでは、アプリのテレメトリが分析されて画面に表示されます。

#### インストルメンテーション キーのコピー

このキーでリソースが識別されます。このキーは、リソースにデータを送信するように SDK を構成するために必要です。

![[要点] ボックスのドロワを開き、インストルメンテーション キーを選択する](./media/app-insights-windows-get-started/02-props.png)


## 2\.アプリへの Application Insights SDK の追加

Visual Studio で、適切な SDK をプロジェクトに追加します。

Windows ユニバーサル アプリの場合は、Windows Phone プロジェクトと Windows プロジェクトの両方に対してこの手順を繰り返します。

1. ソリューション エクスプローラーでプロジェクトを右クリックし、**[NuGet パッケージの管理]** をクリックします。

    ![](./media/app-insights-windows-get-started/03-nuget.png)

2. 「Application Insights」を検索します。

    ![](./media/app-insights-windows-get-started/04-ai-nuget.png)

3. **Windows アプリケーション用の Application Insights** を選択する

4. プロジェクトのルートに ApplicationInsights.config ファイルを追加し、ポータルでコピーしたインストルメンテーション キーを挿入します。この構成ファイルのサンプル xml を次に示します。

	```xml

		<?xml version="1.0" encoding="utf-8" ?>
		<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings>
			<InstrumentationKey>YOUR COPIED INSTRUMENTATION KEY</InstrumentationKey>
		</ApplicationInsights>
	```

    ApplicationInsights.config ファイルのプロパティを次のように設定します。[**ビルド アクション**] == [**コンテンツ**]、[**出力ディレクトリにコピー**] == [**常にコピーする**]。

	![](./media/app-insights-windows-get-started/AIConfigFileSettings.png)

5. 次の初期化コードを追加します。このコードを `App()` コンストラクターに追加することをお勧めします。他の場所にコピーすると、最初の PageViews のコレクションが失われることがあります。

```C#

    using Microsoft.ApplicationInsights;
    ...

	public App()
	{
	   // Add this initilization line.
	   WindowsAppInitializer.InitializeAsync();

	   this.InitializeComponent();
	   this.Suspending += OnSuspending;
	}  
```

**Windows ユニバーサル アプリ**: Phone プロジェクトとストア プロジェクトの両方に対して手順を繰り返します。[Windows 8.1 ユニバーサル アプリの例](https://github.com/Microsoft/ApplicationInsights-Home/tree/master/Samples/Windows%208.1%20Universal)。

## <a name="network"></a>3.アプリのネットワーク アクセスの有効化

アプリで[インターネット アクセスを要求](https://msdn.microsoft.com/library/windows/apps/hh452752.aspx)していない場合は、[必要な機能](https://msdn.microsoft.com/library/windows/apps/br211477.aspx)としてとしてそれをマニフェストに追加する必要があります。

## <a name="run"></a>4.プロジェクトの実行

[F5 キーを押してアプリケーションを実行](http://msdn.microsoft.com/library/windows/apps/bg161304.aspx)し、これを使用してテレメトリを生成します。

Visual Studio で、受け取ったイベント数を確認できます。

![](./media/app-insights-windows-get-started/appinsights-09eventcount.png)

デバッグ モードでは、テレメトリは生成されるとすぐに送信されます。リリース モードでは、テレメトリはデバイスに格納され、アプリケーションの再開時にのみ送信されます。


## <a name="monitor"></a>5.監視データの表示

[Azure ポータル](https://portal.azure.com)で、以前に作成した Application Insights リソースを開きます。

最初、1 つまたは 2 つのポイントだけが表示されます。次に例を示します。

![クリックしてより多くのデータを表示する](./media/app-insights-windows-get-started/appinsights-26-devices-01.png)

大量のデータが予想される場合は、数秒後に **[最新の情報に更新]** をクリックします。

いずれかのグラフをクリックして、詳細を表示します。


## <a name="deploy"></a>5.ストアへのアプリケーションの発行

[アプリケーションを発行](http://dev.windows.com/publish)して、ユーザーがそのアプリケーションをダウンロードして使用すると累積されるデータを確認します。

## テレメトリのカスタマイズ

#### コレクターの選択

Application Insights SDK には、さまざまな種類のデータをアプリから自動的に収集する、複数のコレクターが用意されています。既定では、これらはすべてアクティブになります。ただし、アプリ コンストラクターでどのコレクターを初期化するかを選択することができます。

    WindowsAppInitializer.InitializeAsync( "00000000-0000-0000-0000-000000000000",
       WindowsCollectors.Metadata
       | WindowsCollectors.PageView
       | WindowsCollectors.Session
       | WindowsCollectors.UnhandledException);

#### 独自のテレメトリ データを送信する

[API][api] を使用して、イベント、メトリック、および診断データを Application Insights に送信します。概要:

```C#

 var tc = new TelemetryClient(); // Call once per thread

 // Send a user action or goal:
 tc.TrackEvent("Win Game");

 // Send a metric:
 tc.TrackMetric("Queue Length", q.Length);

 // Provide properties by which you can filter events:
 var properties = new Dictionary{"game", game.Name};

 // Provide metrics associated with an event:
 var measurements = new Dictionary{"score", game.score};

 tc.TrackEvent("Win Game", properties, measurements);

```

詳細については、[カスタムのイベントとメトリックに関する記事][api]を参照してください。

## 次の手順

* [アプリのクラッシュの検出と診断][windowsCrash]
* [メトリックの詳細][metrics]
* [診断検索の詳細][diagnostic]


## <a name="ide"></a>自動セットアップ

セットアップ手順を Visual Studio で自動実行する場合は、Windows Phone、Windows ストアなどのさまざまな種類のアプリで自動実行できます。

### <a name="new"></a>新しい Windows アプリ プロジェクトを作成する場合...

**[新しいプロジェクト]** ダイアログ ボックスで **[Application Insights]** を選択します。

サインインが要求されたら、Azure アカウントの資格情報を使用します。

![](./media/app-insights-windows-get-started/appinsights-d21-new.png)


### <a name="existing"></a>既存のプロジェクトの場合...

ソリューション エクスプローラーから Application Insights を追加します。


![](./media/app-insights-windows-get-started/appinsights-d22-add.png)

## SDK の新しいリリースにアップグレードする

[新しい SDK がリリースされた](app-insights-release-notes-windows.md)場合:

* プロジェクトを右クリックし、[NuGet パッケージの管理] を選択します。
* インストール済みの Application Insights パッケージを選択し、**[アップグレード]** アクションを選択します。


## <a name="usage"></a>次のステップ


[アプリのクラッシュの検出と診断][windowsCrash]

[診断ログのキャプチャと検索][diagnostic]


[アプリの使用状況の追跡][windowsUsage]

[API を使用したカスタム テレメトリの送信][api]

[トラブルシューティング][qna]



<!--Link references-->

[api]: app-insights-api-custom-events-metrics.md
[azure]: ../insights-perf-analytics.md
[diagnostic]: app-insights-diagnostic-search.md
[metrics]: app-insights-metrics-explorer.md
[portal]: http://portal.azure.com/
[qna]: app-insights-troubleshoot-faq.md
[roles]: app-insights-resources-roles-access-control.md
[windowsCrash]: app-insights-windows-crashes.md
[windowsUsage]: app-insights-windows-usage.md

<!---HONumber=Oct15_HO4-->