<properties 
   pageTitle="StorSimple Snapshot Manager を使用したボリューム グループの作成と管理 | Microsoft Azure"
   description="StorSimple Snapshot Manager MMC スナップインを使用して、ボリューム グループを作成および管理する方法について説明します。"
   services="storsimple"
   documentationCenter="NA"
   authors="SharS"
   manager="adinah"
   editor="" />
<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="07/09/2015"
   ms.author="v-sharos" />

# StorSimple Snapshot Manager を使用したボリューム グループの作成と管理

## 概要

**[スコープ]** ウィンドウの **[ボリューム グループ]** ノードを使用して、ボリューム グループへのボリュームの割り当て、ボリューム グループに関する情報の表示、バックアップのスケジュール設定、およびボリューム グループの編集を行うことができます。

ボリューム グループは関連するボリュームのプールであり、バックアップのアプリケーション整合性を確保するために使用されます。詳細については、「[ボリュームとボリューム グループ](storsimple-what-is-snapshot-manager.md#volumes-and-volume-groups)」および「[Windows ボリューム シャドウ コピー サービスとの統合](storsimple-what-is-snapshot-manager.md#integration-with-windows-volume-shadow-copy-service)」をご覧ください。

>[AZURE.IMPORTANT]ボリューム グループを構成するときは、クラスター共有ボリューム (CSV) と非 CSV を同じボリューム グループに混在させないでください。StorSimple Snapshot Manager では、同じスナップショット内に CSV と非 CSV が混在した環境はサポートされません。
 
![[ボリューム グループ] ノード](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_Volume_groups.png)

**図 1: StorSimple Snapshot Manager の [ボリューム グループ] ノード**

このチュートリアルでは、StorSimple Snapshot Manager を使用して次を行う方法を説明します。

- ボリューム グループに関する情報の表示 
- ボリューム グループの作成
- ボリューム グループのバックアップ
- ボリューム グループの編集
- ボリューム グループの削除

これらの操作はすべて、**[操作]** ウィンドウでも使用できます。
 
## ボリューム グループの表示

**[ボリューム グループ]** ノードをクリックすると、列の選択内容に従って、**[結果]** ウィンドウに各ボリューム グループに関する次の情報が表示されます (**[結果]** ウィンドウの列は構成可能です。**[ボリューム]** ノードを右クリックし、**[ビュー]** を選択して、**[列の追加と削除]** を選択します)。

結果の列 | 説明 
:--------------|:------------ 
名前 | **[名前]** 列には、ボリューム グループの名前が含まれます。
アプリケーション | **[アプリケーション]** 列は、Windows ホスト上で現在インストールされ、実行されている VSS ライターの数を示します。
オン | **[オン]** 列には、ボリューム グループに含まれているボリュームの数が表示されます。ゼロ (0) は、ボリューム グループ内のボリュームに関連付けられたアプリケーションが存在しないことを示します。
インポート済み | **[インポート済み]** 列には、インポートされたボリュームの数が表示されます。**True** に設定されている場合、この列はボリューム グループが Microsoft Azure の管理ポータルからインポートされ、StorSimple Snapshot Manager では作成されなかったことを示します。
 
>[AZURE.NOTE]StorSimple Snapshot Manager ボリューム グループも Microsoft Azure の管理ポータルの **[バックアップ ポリシー]** タブに表示されます。
 
## ボリューム グループの作成

ボリューム グループを作成するには、以下の手順を実行します。

#### ボリューム グループを作成するには

1. デスクトップ アイコンをクリックして、StorSimple Snapshot Manager を起動します。 

2. **[スコープ]** ウィンドウで、**[ボリューム グループ]** を右クリックし、**[ボリューム グループの作成]** をクリックします。

    ![ボリューム グループの作成](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_Create_volume_group.png)
 
    **[ボリューム グループの作成**] ダイアログ ボックスが表示されます。

    ![[ボリューム グループの作成] ダイアログ](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_CreateVolumeGroup_dialog.png)

3.  次の情報を入力します。

    1. **[名前]** ボックスに、新しいボリューム グループの一意の名前を入力します。 

    2. **[アプリケーション]** ボックスで、ボリューム グループに追加するボリュームに関連付けられたアプリケーションを選択します。

        **[アプリケーション]** ボックスには、Azure StorSimple ボリュームを使用し、それらに対して VSS ライターが有効になっているアプリケーションのみが一覧表示されます。VSS ライターは、ライターが認識しているすべてのボリュームが Azure StorSimple ボリュームである場合にのみ有効です。[アプリケーション] ボックスが空の場合は、Azure StorSimple ボリュームを使用し、サポートされる VSS ライターを持つアプリケーションはインストールされていません (現在、Azure StorSimple は Microsoft Exchange と SQL Server をサポートしています)。 VSS ライターの詳細については、「[Windows ボリューム シャドウ コピー サービスとの統合](storsimple-what-is-snapshot-manager.md#integration-with-windows-volume-shadow-copy-service)」をご覧ください。

        アプリケーションを選択すると、それに関連付けられているすべてのボリュームが自動的に選択されます。逆に、特定のアプリケーションに関連付けられているボリュームを選択すると、**[アプリケーション]** ボックスでそのアプリケーションが自動的に選択されます。

    3. **[ボリューム]** ボックスで、ボリューム グループに追加する Azure StorSimple ボリュームを選択します。

      - 1 つまたは複数のパーティションを持つボリュームを含めることができます (複数のパーティションを持つボリュームは、複数のパーティションを持つ動的ディスクまたは基本ディスクにすることができます)。 複数のパーティションが含まれているボリュームは、1 つの単位として扱われます。このため、ボリューム グループに 1 つのパーティションのみを追加する場合、その他のすべてのパーティションが、同時にそのボリューム グループに自動的に追加されます。複数パーティション ボリュームをボリューム グループに追加した後も、その複数パーティション ボリュームは 1 つの単位として扱われます。

      - ボリュームを割り当てないようにして、空のボリューム グループを作成できます。

      - クラスター共有ボリューム (CSV) と非 CSV を同じボリューム グループで混在させないでください。StorSimple Snapshot Manager では、同じスナップショット内に CSV ボリュームと非 CSV ボリュームが混在した環境はサポートされません。

4. **[OK]** をクリックしてボリューム グループを保存します。

## ボリューム グループのバックアップ

ボリューム グループをバックアップするには、以下の手順を実行します。

#### ボリューム グループをバックアップするには

1. デスクトップ アイコンをクリックして、StorSimple Snapshot Manager を起動します。

2. **[スコープ]** ウィンドウで、**[ボリューム グループ]** ノードを展開し、ボリューム グループ名を右クリックして、**[バックアップの作成]** をクリックします。

    ![ボリューム グループの即時バックアップ](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_Take_backup.png)

3. **[バックアップの作成]** ダイアログ ボックスで、**[ローカル スナップショット]** または **[クラウド スナップショット]** を選択し、**[作成]** をクリックします。

    ![[バックアップの作成] ダイアログ](./media/storsimple-snapshot-manager-manage-volume-groups/HCS_SSM_TakeBackup_dialog.png)

4. バックアップが実行されていることを確認するには、**[ジョブ]** ノードを展開し、**[実行中]** をクリックします。バックアップが表示されます。

5. 完了したスナップショットを表示するには、**[バックアップ カタログ]** ノードを展開し、ボリューム グループ名を展開して、**[ローカル スナップショット]** または **[クラウド スナップショット]** をクリックします。バックアップが正常に完了した場合は、バックアップが表示されます。

## ボリューム グループの編集

ボリューム グループを編集するには、以下の手順を実行します。

#### ボリューム グループを編集するには

1. デスクトップ アイコンをクリックして、StorSimple Snapshot Manager を起動します。

2. **[スコープ]** ウィンドウで、**[ボリューム グループ]** ノードを展開し、ボリューム グループ名を右クリックして、**[編集]** をクリックします。

3. **[ボリューム グループの作成]** ダイアログ ボックスが表示されます。**[名前]**、**[アプリケーション]**、および **[ボリューム]** エントリは変更できます。

4. **[OK]** をクリックして変更を保存します。

## ボリューム グループの削除

ボリューム グループを削除するには、以下の手順を実行します。

>[AZURE.WARNING]この処理により、ボリューム グループに関連付けられているすべてのバックアップも削除されます。

#### ボリューム グループを削除するには

1. デスクトップ アイコンをクリックして、StorSimple Snapshot Manager を起動します。 

2. **[スコープ]** ウィンドウで、**[ボリューム グループ]** ノードを展開し、ボリューム グループ名を右クリックして、**[削除]** をクリックします。

3. **[ボリューム グループの削除]** ダイアログ ボックスが表示されます。テキスト ボックスに「**Confirm**」と入力し、**[OK]** をクリックします。

    削除されたボリューム グループは **[結果]** ウィンドウのリストから消去され、そのボリューム グループに関連付けられているすべてのバックアップはバックアップ カタログから削除されます。

## 次のステップ

[StorSimple Snapshot Manager を使用したバックアップ ポリシーの作成と管理](storsimple-snapshot-manager-manage-backup-policies.md)。

<!---HONumber=July15_HO3-->