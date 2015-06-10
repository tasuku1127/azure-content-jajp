<properties 
	pageTitle="SQL Database V12 へのアップグレード" 
	description="Azure SQL Database の以前のバージョンから Azure SQL Database V12 にアップグレードする方法について説明します。" 
	services="sql-database" 
	documentationCenter="" 
	authors="sonalmm" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.tgt_pltfrm="na" 
	ms.workload="data-management" 
	ms.date="05/15/2015" 
	ms.author="sonalm"/>


# SQL Database V12 へのインプレース アップグレード


SQL Database 更新プログラム V12 に[サインアップ](https://portal.azure.com)すると、Microsoft Azure で次世代の SQL Database の機能を活用できるようになります。使用を開始するには、Microsoft Azure のサブスクリプションが必要です。[Azure の無料評価版](http://azure.microsoft.com/pricing/free-trial)にサインアップして[料金](http://azure.microsoft.com/pricing/details/sql-database)の情報をご確認ください。


## SQL Database V12 へのアップグレード手順


| アップグレードの手順 | スクリーン ショット |
| :--- | :--- |
| 1.[http://portal.azure.com/](http://portal.azure.com/) にサインインします。 | ![新しい Azure ポータル][1] |
| 2.**[参照]** をクリックします。 | ![サービスの参照][2] |
| 3.**[SQL Server]** をクリックします。SQL Server 名の一覧が表示されます。 | ![SQL Server サービスの選択][3] |
| 4.SQL Database 更新プログラムを有効にした新しいサーバーにコピーするサーバーを選択します。 | ![SQL Server の一覧表示][4] |
| 5.**[最新の SQL Database 更新プログラム V12]** をクリックします。 | ![最新のプレビュー機能][5] |
| 6.**[このサーバーをアップグレード]** をクリックします。 | ![SQL Server からプレビューへのアップグレード][6] |


> [AZURE.NOTE]アップグレード オプションを選択すると、サーバーとそのサーバー内のデータベースで SQL Database V12 の機能が有効になり、元に戻すことができなくなります。サーバーを SQL Database V12 にアップグレードするには、Basic、Standard、Premium のいずれかのサービス階層が必要です。サービス階層の詳細については、「[SQL Database の Web/Business データベースを新しいサービス階層にアップグレードする](sql-database-upgrade-new-service-tiers.md)」を参照してください。


> [AZURE.IMPORTANT]SQL Database V12 (プレビュー) では、geo レプリケーションはサポートされていません。詳細については、「[Azure SQL Database V12 (プレビュー) へのアップグレードの計画と準備をする](sql-database-v12-plan-prepare-upgrade.md)」を参照してください。


**[このサーバーをアップグレード]** オプションをクリックすると、開いているブレードに、検証プロセスに関するメッセージが表示されます。


- 検証プロセスにより、データベースのサービス階層がチェックされ、Geo レプリケーションが有効になっているかどうかがチェックされます。検証が完了するとブレードに結果が表示されます。 
- 検証プロセスが完了すると、SQL Database V12 へのアップグレード要件を満たすためのアクションが必要なデータベース名の一覧が表示されます。
 - **SQL Database V12 にアップグレードするには、これらの各データベースでアクションを実行する必要があります**。
- 各データベース名をクリックすると、新しいブレードに、現在の使用状況に基づいた推奨のサービス価格レベルが表示されます。また、異なる価格レベルを参照して、お使いの環境に最適な価格レベルを選択できます。Geo レプリケーションが設定されているすべてのデータベースを再構成して、レプリケーションを停止する必要があります。 
- 十分なデータが見つからない場合は、推奨の価格レベルが表示されませんのでご注意ください。 


| アクション | スクリーン ショット |
| :--- | :--- |
| 7.サーバーのアップグレード準備のためのアクションが完了したら、アップグレードするサーバー名を入力して **[OK]** をクリックします。 | ![アップグレードするサーバー名の確認][7] |
| 8.アップグレード処理が始まります。アップグレードには最大 24 時間かかることがあります。その間、このサーバー上のすべてのデータベースはオンラインのままですが、サーバーとデータベースの管理アクションは制限されます。プロセスが完了すると、**[有効]** ステータスがサーバー ブレードに表示されます。 | ![プレビュー機能が有効になっていることの確認][8] |


## PowerShell コマンドレット


PowerShell コマンドレットを使用して、V11 を含め、V12 より前のバージョンから V12 への Azure SQL Database のアップグレードを開始、停止、監視できます。


これらの PowerShell コマンドレットに関する次のリファレンス ドキュメントを参照してください。


- [Get-AzureSqlServerUpgrade](http://msdn.microsoft.com/library/mt143621.aspx)
- [Start-AzureSqlServerUpgrade](http://msdn.microsoft.com/library/mt143623.aspx)
- [Stop-AzureSqlServerUpgrade](http://msdn.microsoft.com/library/mt143622.aspx)


Stop- コマンドレットは、一時停止ではなく取り消しを意味します。アップグレードを再開する方法は、最初からやり直す方法以外にありません。Stop- コマンドレットにより、該当するすべてのリソースがクリーンアップされ、解放されます。


## 関連リンク

-  [SQL Database V12 の新機能](sql-database-v12-whats-new.md)
- [SQL Database V12 へのアップグレードの計画と準備をする](sql-database-v12-plan-prepare-upgrade.md)


<!--Image references-->
[1]: ./media/sql-database-preview-upgrade/firstscreenportal.png
[2]: ./media/sql-database-preview-upgrade/browse.png
[3]: ./media/sql-database-preview-upgrade/sqlserver.png
[4]: ./media/sql-database-preview-upgrade/sqlserverlist.png
[5]: ./media/sql-database-preview-upgrade/latestprview.png
[6]: ./media/sql-database-preview-upgrade/upgrade.png
[7]: ./media/sql-database-preview-upgrade/typeservername.png
[8]: ./media/sql-database-preview-upgrade/enabled.png

<!---HONumber=58-->