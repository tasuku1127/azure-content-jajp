<properties
	pageTitle="Azure 仮想マシンのバックアップ |Microsoft Azure"
	description="Azure 仮想マシンをバックアップする手順"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="10/29/2015"
	ms.author="trinadhk; aashishr; jimpark; markgal"/>


# Azure 仮想マシンのバックアップ
この記事では、既存の Azure 仮想マシンを会社のバックアップと保持に関するポリシーに従って保護するためにバックアップする手順について説明します。

まず、Azure 仮想マシンをバックアップする前にいくつかのことに注意する必要があります。まだ実行していない場合は、[前提条件](backup-azure-vms-prepare.md)を完了して、VM をバックアップできるように環境を準備してから、操作を続行します。

[Azure における VM のバックアップ インフラストラクチャの計画](backup-azure-vms-introduction.md)か [Azure Virtual Machines](https://azure.microsoft.com/documentation/services/virtual-machines/) に関する情報を確認する場合は、これらの情報に関するそれぞれのリンクを使用できます。

Azure 仮想マシンのバックアップには、次の 3 つの主要な手順が含まれます。

![Three steps to back up an Azure virtual machine](./media/backup-azure-vms/3-steps-for-backup.png)

>[AZURE.NOTE]仮想マシンは、ローカルにバックアップされます。あるリージョンの仮想マシンを別のリージョンのバックアップ コンテナーにバックアップすることはできません。そのため、バックアップが必要な VM がある Azure リージョンごとに、そのリージョン内に少なくとも 1 つ以上のバックアップ コンテナーを作成する必要があります。

## 手順 1. Azure 仮想マシンを検出する
サブスクリプションに追加された新しい仮想マシンが確実に識別されるように、検出プロセスは常に最初の手順として実行する必要があります。このプロセスでは、サブスクリプションに含まれる仮想マシンの一覧を、クラウド サービス名、リージョンなどの追加情報と共に Azure にクエリします。

1. Azure ポータルの **[Recovery Services]** にあるバックアップ コンテナーに移動し、**[登録済みの項目]** をクリックします。

2. ドロップダウン メニューから **[Azure 仮想マシン]** を選択します。

    ![ワークロードの選択](./media/backup-azure-vms/discovery-select-workload.png)

3. ページの下部にある **[検出]** をクリックします。![[検出] ボタン](./media/backup-azure-vms/discover-button-only.png)

    仮想マシンが集計されるまで、この検出プロセスに数分かかる場合があります。プロセスが実行中であることを知らせる通知が画面の下部に表示されます。

    ![VMS の検出](./media/backup-azure-vms/discovering-vms.png)

    プロセスが完了すると、通知が変更されます。

    ![discover-done](./media/backup-azure-vms/discovery-complete.png)

##  手順 2. Azure 仮想マシンを登録する
Azure Backup サービスに関連付けるために、Azure 仮想マシンを登録します。登録は、通常、1 回限りの操作です。

1. Azure ポータルの **[Recovery Services]** にあるバックアップ コンテナーに移動し、**[登録済みの項目]** をクリックします。

2. ドロップダウン メニューから **[Azure 仮想マシン]** を選択します。

    ![ワークロードの選択](./media/backup-azure-vms/discovery-select-workload.png)

3. ページの下部にある **[登録]** をクリックします。![[登録] ボタン](./media/backup-azure-vms/register-button-only.png)

4. **[項目の登録]** ショートカット メニューで、登録する仮想マシンを選択します。同じ名前の仮想マシンが 2 つ以上ある場合は、クラウド サービスを使用して仮想マシンを区別します。

    >[AZURE.TIP]一度に複数の仮想マシンを登録することができます。

    選択した仮想マシンごとにジョブが作成されます。

5. 通知内の **[ジョブの表示]** をクリックして **[ジョブ]** ページに移動します。

    ![登録ジョブ](./media/backup-azure-vms/register-create-job.png)

    仮想マシンが登録済みの項目の一覧にも、登録操作の状態と共に表示されます。

    ![登録状態 1](./media/backup-azure-vms/register-status01.png)

    操作が完了すると、*登録済み*の状態を反映するように状態が変更されます。

    ![登録状態 2](./media/backup-azure-vms/register-status02.png)

## 手順 3. Azure 仮想マシンを保護する
この手順で、仮想マシンのバックアップおよび保持に関するポリシーを設定でます。1 回の保護操作で複数の仮想マシンを保護できます。

2015 年 5 月以降に作成された Azure のバックアップ コンテナーには、既定のポリシーが組み込まれています。この既定のポリシーには、30 日間の既定の保持期間と 1 日 1 回のバックアップ スケジュールが含まれています。

1. Azure ポータルの **[Recovery Services]** にあるバックアップ コンテナーに移動し、**[登録済みの項目]** をクリックします。
2. ドロップダウン メニューから **[Azure 仮想マシン]** を選択します。

    ![ポータルでのワークロードの選択](./media/backup-azure-vms/select-workload.png)

3. ページの下部にある **[保護]** をクリックします。

    **項目の保護**ウィザードが表示されます。このウィザードには、保護されていない登録済みの仮想マシンのみが一覧表示されます。このウィザードで、保護する仮想マシンを選択します。

    同じ名前の仮想マシンが 2 つ以上ある場合は、クラウド サービスを使用して仮想マシンを区別します。

    >[AZURE.TIP]一度に複数の仮想マシンを保護できます。

    ![保護をスケールで構成](./media/backup-azure-vms/protect-at-scale.png)

4. 選択した仮想マシンをバックアップするための **[バックアップのスケジュール]** を選択します。既存のポリシーのセットから選択することも、新しいポリシーを定義することもできます。

    各バックアップ ポリシーには、複数の仮想マシンを関連付けることができます。ただし、仮想マシンは常に 1 つのポリシーにしか関連付けることができません。

    ![新しいポリシーで保護](./media/backup-azure-vms/policy-schedule.png)

    >[AZURE.NOTE]バックアップ ポリシーには、スケジュールされたバックアップの保持スキーマが含まれています。既存のバックアップ ポリシーを選択した場合は、次の手順で保持オプションを変更できません。

5. バックアップに関連付ける**リテンション期間**を選択します。

    ![フレキシブルな保持期間で保護](./media/backup-azure-vms/policy-retention.png)

    バックアップを保存する期間の長さは保持ポリシーで指定します。バックアップを行うタイミングに基づいて複数の保持ポリシーを指定できます。たとえば、各四半期の最後に実行されるバックアップ ポイントは、(監査のために) 長期間保持する必要がありますが、毎日実行されるバックアップ ポイントは、運用上の復旧ポイントとして機能するため、保持は 90 日間だけで済みます。

    ![復旧ポイントを作成した後の仮想マシンのバックアップ](./media/backup-azure-vms/long-term-retention.png)

    この画像の例の場合:

    - **日ごとの保持ポリシー**: バックアップは毎日作成され、30 日間保持されます。
    - **週ごとの保持ポリシー**: バックアップは毎週日曜日に作成され、104 週間保持されます。
    - **月ごとの保持ポリシー**: バックアップは各月の最後の日曜日に作成され、120 か月保持されます。
    - **年ごとの保持ポリシー**: バックアップは 1 月の最初の日曜日に作成され、99 年間保持されます。

    選択した仮想マシンごとに、保護ポリシーを構成して仮想マシンをポリシーに関連付けるためのジョブが作成されます。

6. **[ジョブ]** をクリックし、適切なフィルターを選択して、"**保護の構成**" ジョブの一覧を表示します。

    ![保護の構成ジョブ](./media/backup-azure-vms/protect-configureprotection.png)

## 初回バックアップ
ポリシーを使用して保護した仮想マシンは、**[保護された項目]** タブに表示され、保護の状態は *[保護済み (初回バックアップは完了していません)]* と表示されます。既定では、スケジュールされた最初のバックアップが*初回バックアップ*となります。

保護を構成した直後に初回バックアップをトリガーするには:

1. **[保護された項目]** ページの下部にある **[今すぐバックアップ]** をクリックします。

    Azure Backup サービスによって、初回バックアップ操作用にバックアップ ジョブが作成されます。

2. **[ジョブ]** タブをクリックしてジョブの一覧を表示します。

    ![バックアップが進行中](./media/backup-azure-vms/protect-inprogress.png)

>[AZURE.NOTE]Azure Backup サービスは、バックアップ操作の一部として、各仮想マシンのバックアップ拡張機能に対して、すべての書き込みをフラッシュし、整合性のあるスナップショットを作成するためのコマンドを発行します。

初回バックアップが完了すると、**[保護された項目]** タブの仮想マシンの状態が *[保護済み]* になります。

![復旧ポイントを作成した後の仮想マシンのバックアップ](./media/backup-azure-vms/protect-backedupvm.png)

## バックアップの状態と詳細の表示
仮想マシンが保護されると、**[ダッシュボード]** ページの概要に示される仮想マシンの数が増加します。**[ダッシュボード]** ページには、過去 24 時間の間に*成功したジョブ*、*失敗したジョブ*、*現在進行中のジョブ*の数が表示されます。いずれかのカテゴリをクリックすると、**[ジョブ]** ページにそのカテゴリの詳細が表示されます。

![[ダッシュボード] ページのバックアップの状態](./media/backup-azure-vms/dashboard-protectedvms.png)

ダッシュボード内の値は、24 時間に 1 度更新されます。

## エラーのトラブルシューティング
仮想マシンのバックアップ中に問題が発生した場合は、この[トラブルシューティング](backup-azure-vms-troubleshoot.md) ガイダンスを参照してください。

## 次のステップ

- [仮想マシンの管理と監視](backup-azure-manage-vms.md)
- [仮想マシンの復元](backup-azure-restore-vms.md)

<!---HONumber=Nov15_HO3-->