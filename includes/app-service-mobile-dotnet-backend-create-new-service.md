

新しいモバイル アプリ バックエンドを作成するには、次の手順に従います。

1. [Azure ポータル]にログインします。

2. ウィンドウの左上で、**[+ 新規]**、**[Web + モバイル]**、**[モバイル アプリ]** の順にクリックし、モバイル アプリ バックエンドの名前を指定します。

3. **[リソース グループ]** ボックスで、アプリと同じ名前を入力します。

4. 既定の App Service プランが選択されます。App Service プランを変更するには、[App Service プラン]、**[+ 新規作成]** の順にクリックします。新しい App Service プランの名前を指定し、適切な場所を選択します。[価格レベル] をクリックし、サービスに適切な価格レベルを選択します。**[すべて表示]** を選択して、**Free** や **Shared** などの価格オプションをさらに表示します。価格レベルを選択したら、**[選択]** をクリックします。**[App Service プラン]** ブレードに戻り、**[OK]** をクリックします。

5. **[作成]** をクリックします。これにより、モバイル アプリ バックエンドが作成され、後でサーバー プロジェクトをデプロイすることができます。モバイル アプリ バックエンドのプロビジョニングには数分かかる場合があります。モバイル アプリ バックエンドのプロビジョニングが完了すると、ポータルでモバイル アプリ バックエンドの **[設定]** ブレードが開きます。次の手順で、新しい SQL データベースを作成します。

    > [AZURE.NOTE]このチュートリアルの一環として、新しい SQL Database インスタンスと SQL Database サーバーを作成します。この新しいデータベースは、再利用したり、その他の SQL Database インスタンスと同様に管理したりすることができます。新しいモバイル アプリ バックエンドと同じ場所に、既にデータベースがある場合は、**[既存のデータベースを使用する]** を選ぶと、そのデータベースを選択できます。別の場所にあるデータベースを使用することは、帯域幅コストと待機時間が増加するため、お勧めしません。

6. 新しいモバイル アプリ バックエンドで、**[設定]**、**[モバイル アプリ]**、**[データ]**、**[+ 追加]** の順にクリックします。

7. **[データ接続の追加]** ブレードで、**[SQL Database - 必要な設定の構成]**、**[新しいデータベースの作成]** の順にクリックします。**[名前]** フィールドに新しいデータベースの名前を入力します。

8. **[サーバー]** をクリックします。**[新しいサーバー]** ブレードで、**[サーバー名]** フィールドに一意のサーバー名を入力し、**[サーバー管理ログイン]** と **[パスワード]** に適切な内容を指定します。**[Azure サービスにサーバーへのアクセスを許可する]** チェック ボックスがオンになっていることを確認します。**[OK]** をクリックします。

    ![](./media/app-service-mobile-dotnet-backend-create-new-service/dotnet-backend-create-db.png)

9. **[新しいデータベース]** ブレードで、**[OK]** をクリックします。

10. **[データ接続の追加]** ブレードに戻り、**[接続文字列]** を選択して、データベースの作成時に指定したログイン名とパスワードを入力します。既存のデータベースを使用する場合は、そのデータベースのログイン資格情報を入力します。入力したら、**[OK]** をクリックします。

11. もう一度 **[データ接続の追加]** ブレードに戻り、**[OK]** をクリックしてデータベースを作成します。

データベースの作成には数分かかる場合があります。**[通知]** 領域を使用して、デプロイの進行状況を監視します。データベースのデプロイが正常に完了するまでは、先に進まないでください。

これで、モバイル クライアント アプリケーションで使用できる Azure モバイル アプリ バックエンドのプロビジョニングが完了しました。次は、簡単な "todo list" バックエンドのサーバー プロジェクトをダウンロードして、それを Azure に発行します。

<!-- URLs. -->
[Azure ポータル]: https://portal.azure.com/

<!---HONumber=Nov15_HO1-->