<properties linkid="manage-services-hdinsight-howto-hive" urlDisplayName="Use Hive in HDInsight Hadoop for website log analysis" pageTitle="Web サイトのログ分析のための HDInsight Hadoop での Hive の使用 | Azure" metaKeywords="" description="Web サイトのログを分析するために HDInsight で Hive を使用する方法を説明します。HDInsight テーブルへの入力にログ ファイルを使用し、HiveQL を使用してデータを照会します。" metaCanonical="" services="hdinsight" documentationCenter="" title="Web サイトのログ分析のための HDInsight Hadoop での Hive の使用" authors="nitinme" solutions="" manager="paulettm" editor="cgronlun" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/14/2014" ms.author="nitinme" />

# Web サイトのログを分析するための HDInsight での Hive の使用

Web サイトのログを分析するために HDInsight で HiveQL を使用する方法を説明します。Web サイトのログ分析は、類似するアクティビティに基づく対象ユーザーの区分、人口統計によるサイト訪問者の分類、参照されたコンテンツや訪問元の Web サイトの確認などのために使用できます。

このサンプルでは、HDInsight クラスターを使用して Web サイトのログ ファイルを分析することにより、1 日の間に発生した外部 Web サイトからの Web サイトへのアクセス数を調べ、ユーザーが経験した Web サイト エラーの概要を取得します。学習内容:

-   Web サイトのログ ファイルを格納する Azure BLOB ストレージへの接続
-   これらのログを照会するための HIVE テーブルの作成
-   データを分析するための HIVE クエリの作成
-   分析したデータを取得するための、Microsoft Excel を使用した HDInsight への接続 (ODBC 接続を使用)

![HDI.Samples.Website.Log.Analysis][HDI.Samples.Website.Log.Analysis]

## 前提条件

-   **HDInsight クラスター**のプロビジョニングを終えている必要があります。手順については、「[HDInsight での Hadoop クラスターのプロビジョニング][HDInsight での Hadoop クラスターのプロビジョニング]」を参照してください。
-   Microsoft Excel 2010 または Microsoft Excel 2013 がインストールされていること。
-   Hive から Excel にデータをインポートするための [Microsoft Hive ODBC ドライバー][Microsoft Hive ODBC ドライバー]があること。

## サンプルを実行するには

1.  Azure の管理ポータルから、サンプルの実行に使用するクラスターを選択し、下部にある **[クエリ コンソール]** をクリックします。または、次に示す URL を使用してクエリ コンソールを直接開くこともできます。

        https://<clustername>.azurehdinsight.net

    入力を要求されたら、クラスターをプロビジョニングするときに使用した管理者名とパスワードを使用して認証を実行します。

2.  表示された Web ページで **[概要ギャラリー]** タブをクリックし、**[サンプル]** カテゴリにある **[Web サイト ログ分析]** サンプルを選択します。
3.  Web ページに記載されている手順に従って、サンプルを完了します。

## 次のステップ

Azure HDInsight を使用したセンサー データの分析方法をサンプルで試します。「[Hive を HDInsight と共に使用したセンサー データの分析][Hive を HDInsight と共に使用したセンサー データの分析]」を参照してください。

  [HDI.Samples.Website.Log.Analysis]: ./media/hdinsight-hive-analyze-website-log/hdinsight-weblogs-sample.png
  [HDInsight での Hadoop クラスターのプロビジョニング]: ../hdinsight-provision-clusters/
  [Microsoft Hive ODBC ドライバー]: http://www.microsoft.com/ja-jp/download/details.aspx?id=40886
  [Hive を HDInsight と共に使用したセンサー データの分析]: ../hdinsight-use-hive-sensor-data-analysis/