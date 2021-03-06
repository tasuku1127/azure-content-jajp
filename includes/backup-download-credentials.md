## 資格情報コンテナーの資格情報を使用して、Azure Backup サービスで認証する

データを Azure にバックアップする前に、オンプレミスのサーバー (Windows クライアントまたは Windows Server または Data Protection Manager サーバー) をバックアップ資格情報コンテナーで認証する必要があります。認証は "資格情報コンテナーの資格情報" を使用して実行されます。資格情報コンテナーの資格情報の概念は、Azure PowerShell で使用される "発行の設定" ファイルの概念に似ています。

### 資格情報コンテナーの資格情報ファイルとは

資格情報コンテナーの資格情報ファイルは、各バックアップ資格情報コンテナーごとにポータルによって生成される証明書です。ポータルは公開キーを Access Control Service (ACS) にアップロードします。ユーザーは、マシン登録ワークフローの入力として指定したワークフローの一部として、証明書の秘密キーを使用できます。これにより、コンピューターは Azure Backup サービスで識別された資格情報のコンテナーにバックアップ データを送信できるようになります。

資格情報コンテナーの資格情報は登録ワークフロー中しか使用されません。資格情報コンテナーの資格情報ファイルが漏えいしないようにするのはユーザーの責任です。悪意のあるユーザーが入手した場合、資格情報コンテナーの資格情報ファイルを使用し、同じ資格情報コンテナーに対してその他のコンピューターが登録されてしまう可能性があります。ただし、バックアップ データは顧客に属するパスフレーズを使用して暗号化されているため、既存のバックアップ データが漏えいすることはありません。この問題をなくすため、資格情報コンテナーの資格情報は 48 時間で期限切れになるよう設定されています。バックアップ資格情報コンテナーの資格情報は何回でもダウンロードできますが、登録ワークフロー時には最新の資格情報コンテナーの資格情報ファイルだけが適用できます。

### 資格情報コンテナーの資格情報ファイルのダウンロード

資格情報コンテナーの資格情報ファイルは、セキュリティで保護されたチャネルを介して Azure ポータルからダウンロードされます。Azure Backup サービスは証明書の秘密キーを認識せず、ポータルまたはサービスでは秘密キーは保持されません。資格情報コンテナーの資格情報ファイルをローカル コンピューターにダウンロードするには、次の手順を使用します。

1.  [管理ポータル](https://manage.windowsazure.com/)にサインインします。
2.  左側のナビゲーション ウィンドウの **[Recovery Services]** をクリックして、作成したバックアップ資格情報コンテナーを選択します。クラウド アイコンをクリックして、バックアップ資格情報コンテナーの [クイック スタート] ビューに移動します。

    ![Quick view](./media/backup-download-credentials/quickview.png)

3.  [クイック スタート] ページで **[資格情報コンテナーの資格情報のダウンロード]** をクリックします。ポータルは、ダウンロードで使用可能な資格情報コンテナーの資格情報ファイルを生成します。

    ![ダウンロード](./media/backup-download-credentials/downloadvc.png)

4.  ポータルは、資格情報コンテナーの名前と現在の日付の組み合わせを使用して、資格情報コンテナーの資格情報を生成します。**[保存]** をクリックして、資格情報コンテナーの資格情報をローカル アカウントのダウンロード フォルダーにダウンロードするか、[保存] メニューから [名前を付けて保存] を選択して、保存場所を指定します。

### 注
- 資格情報コンテナーの資格情報は、使用するコンピューターからアクセスできる場所に保存してください。ファイル共有 / SMB に格納されている場合は、アクセス許可を確認します。
- 資格情報コンテナーの資格情報ファイルは登録ワークフロー中しか使用されません。
- 資格情報コンテナーの資格情報ファイルの有効期限は 48 時間であり、ポータルからダウンロードすることができます。
- ワークフローに関する質問については、Azure Backup の [FAQ](backup-azure-backup-faq.md) を参照してください。

<!---HONumber=Oct15_HO3-->