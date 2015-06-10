<properties
	pageTitle="Azure のバックアップ - バックアップと Windows Server または Windows クライアントからの復元"
	description="バックアップし、Windows Server または Windows クライアントから復元する方法をについて説明します。代替サーバーの復旧についても説明します。"
	services="backup"
	documentationCenter=""
	authors="prvijay"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	 ms.tgt_pltfrm="na"
	 ms.devlang="na"
	 ms.topic="article"
	 ms.date="04/02/2015"
	 ms.author="prvijay"/>

# バックアップと Windows server または Windows クライアント コンピューターからの復元
この記事にするための手順では、Windows server または Windows のクライアント コンピューターからバックアップします。また、同じコンピューターと、他のコンピューター上のバックアップ ファイルの復元に必要な手順は、上のファイルをバックアップを復元するのに必要な手順についても説明します。

## バックアップ ファイル
1. コンピューターが登録されると、Microsoft Azure Backup mmc スナップインを開きます。 <br/> ![検索結果][1]

2. クリックして **のバックアップをスケジュール** <br/> ![バックアップのスケジュール][2]

3. 選択する項目をバックアップします。Windows Server または Windows クライアント (つまり System Center Data Protection Manager なし) での azure のバックアップでは、ファイルとフォルダーを保護することができます。 <br/> ![バックアップする項目][3]

4. 次で詳しく説明するバックアップのスケジュールと保有ポリシーを指定して [記事](backup-azure-backup-cloud-as-tape.md)

5. 最初のバックアップを送信する方法を選択します。初期シードを完了するは、バックアップと、インターネットのアップロード リンク速度するデータの量によります。バックアップの GB しようとして/TB の待機時間の間でデータの低帯域幅接続場合、は、最も近い Azure データ センターへのディスクを配布して、最初のバックアップを完了することをお勧めします。これは「オフライン バックアップ」と呼ばこの詳細については [記事](https://msdn.microsoft.com/library/azure/dn894419.aspx)です。十分な帯域幅の接続がある場合は、ネットワーク経由で、最初のバックアップを完了することをお勧めします。 <br/> ![最初のバックアップ][4]

6. バックアップを mmc スナップインをクリックして、ワークフローが完了すると、参照してください **を今すぐバックアップ** をネットワーク経由での初期シードが完了する <br/> 。![今すぐバックアップ][5]

7. 初期シードが完了した後、 **ジョブ** Azure Backup のコンソールのビューには、状態を示します <br/> 。![赤外線の完了][6]

## 同じコンピューター上のデータを回復します。
ファイルを誤って削除した場合、および (、バックアップの作成に使用が) 同じマシンでのファイルまたはボリュームを復元する場合は、次の手順は、データの回復に役立ちます。

1. 開く、 **Microsoft Azure Backup** でスナップします。

2. クリックして **データの回復** ワークフローを開始する <br/> 。![データを回復します。][7]

3. 選択 **このサーバー (* yourmachinename *)** を同じコンピューター上のファイルをバックアップを復元することを計画するときのオプションします <br/> 。![同じコンピューター][8]

4. データを **ファイルの参照]** または **ファイルを検索する**です。パスを持つがわかっている 1 つまたは複数のファイルを復元する予定の場合は、既定のオプションままにします。ファイルを検索するには、フォルダー構造がわからない場合は選択、 **ファイルを検索する** オプション。ここでは、既定のオプションを続行します。 <br/> ![ファイルを参照します。][9]

5. 次の画面で、ファイルを復元するボリュームを選択します。任意の時点から復元する画面のできるようにします。表示される日付 **太字** 、予定表コントロールで、復元ポイントの可用性を示します。日付を選択すると、バックアップ スケジュールを設定 (および、バックアップ操作の成功) に基づくで選択できる、ポイントからの時間、 **時間** ボックスの一覧 <br/> 。![ボリュームと日付][10]

6. [次へ] 画面で、回復する項目を選択します。複数選択してフォルダーまたはファイルを復元することができます。 <br/> ![ファイルを選択します。][11]

7. 次の画面で、回復のパラメーターを指定します。 <br/> ![回復オプション][12]
  + (これで、ファイルまたはフォルダーが上書きされます)、元の場所に、または同じコンピューターで別の場所に復元するオプションがあるとします。

  + ターゲットの場所に復元するファイルまたはフォルダーが存在する場合はコピー (同じファイルの 2 つのバージョン) を作成や、ターゲットの場所でのファイルを上書きまたはのターゲットに存在するファイルの回復をスキップするオプションがあります。

  + 復元されるファイルの Acl を復元する既定のオプションのままにしておくことを強くお勧めします。

8. これらの入力を指定すると、このコンピューターに、ファイルを復元する回復ワークフローが開始します。

## 代わりのコンピューターに回復します。
サーバー全体が失われた場合でも、別のコンピューターでファイルまたはボリュームを回復できます。次の手順には、ワークフローを示しています。

手順で使用される命名規則は、次のとおり: + *ソース マシン* –、元のマシンから、バックアップし、は現在使用できません。

  + *ターゲット マシン* – データが取得されているコンピューター。

  + *サンプルの資格情報コンテナー* – をバックアップ コンテナー、 *ソース マシン* と *ターゲット マシン* が登録されています。 <br/>

> [AZURE.NOTE]以前のバージョンのオペレーティング システムが実行されているコンピューターで、マシンから作成されたバックアップを復元できません。たとえば場合は、Windows 7 マシンから作成されたバックアップが、その復元できますマシン以上、Windows 8 で。ただし、その逆は true 保持しません。

1. 開く、 **Microsoft Azure Backup** でスナップ、 *ターゲット マシン*です。

2. いることを確認、 *ターゲット マシン* 、 *ソース マシン* 同じのバックアップ資格情報コンテナーに復元されます (- この記事で *サンプルの資格情報コンテナー*)。

3. クリックして **データの回復** ワークフローを開始する <br/> 。![データを回復します。][7]

4. 選択 **別のサーバー** <br/> ![別のサーバー][13]

5. 対応する資格情報コンテナー資格情報のファイルを指定して、 *サンプルの資格情報コンテナー*です。資格情報コンテナーの資格情報ファイルには、無効な (または有効期限が切れた) が、ダウンロード、新しい資格情報コンテナー資格情報ファイルから、 *サンプルの資格情報コンテナー* 、Azure ポータルでします。資格情報コンテナーの資格情報ファイルを指定すると、資格情報のコンテナー ファイルに対しては、バックアップ コンテナーが表示されます。

6. 選択、 *ソース マシン* 表示されているマシンの一覧から <br/> 。![マシンの一覧][14]

7. 前回と同様にいずれかを選択、 **ファイルを検索する** または **ファイルの参照]** オプション。使用して、このセクションの目的で、 **ファイルを検索する** オプション <br/> 。![検索][15]

8. 次の画面には、ボリュームと日付を選択します。復元するフォルダーとファイル名を検索します。 <br/> ![項目の検索][16]

9. 復元する必要がある、ファイルの場所を選択します。 <br/> ![復元の場所][17]

10. 提供時に指定された暗号化のパスフレーズ *変換元コンピュータの* への登録を *サンプルの資格情報コンテナー*. <br/> ![暗号化][18]

入力を指定すると、したら、 **回復** 提供先のファイルをバックアップの復元をトリガーするボタンをクリックします。

<!--Image references-->
[1]: ./media/backup-azure-backup-and-recover/result.png
[2]: ./media/backup-azure-backup-and-recover/schedulebackup.png
[3]: ./media/backup-azure-backup-and-recover/items.png
[4]: ./media/backup-azure-backup-and-recover/initialbackup.png
[5]: ./media/backup-azure-backup-and-recover/backupnow.png
[6]: ./media/backup-azure-backup-and-recover/ircomplete.png

[7]: ./media/backup-azure-backup-and-recover/recover.png
[8]: ./media/backup-azure-backup-and-recover/samemachine.png
[9]: ./media/backup-azure-backup-and-recover/browseandsearch.png
[10]: ./media/backup-azure-backup-and-recover/volanddate.png
[11]: ./media/backup-azure-backup-and-recover/selectfiles.png
[12]: ./media/backup-azure-backup-and-recover/recoveroptions.png

[13]: ./media/backup-azure-backup-and-recover/anotherserver.png
[14]: ./media/backup-azure-backup-and-recover/machinelist.png
[15]: ./media/backup-azure-backup-and-recover/search.png
[16]: ./media/backup-azure-backup-and-recover/searchitems.png
[17]: ./media/backup-azure-backup-and-recover/restorelocation.png
[18]: ./media/backup-azure-backup-and-recover/encryption.png

<!---HONumber=GIT-SubDir-->