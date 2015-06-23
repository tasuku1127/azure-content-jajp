<properties 
	pageTitle="PhoneGap での Mobile Services の使用 | モバイル デベロッパー センター" 
	description="このチュートリアルでは、iOS、Android、Windows Phone 用に PhoneGap の開発を行う場合に Azure Mobile Services を使用する方法について説明します。" 
	services="mobile-services" 
	documentationCenter="" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="" 
	ms.devlang="multiple" 
	ms.topic="hero-article" 
	ms.date="11/21/2014" 
	ms.author="glenga"/>

[AZURE.INCLUDE [mobile-services-selector-get-started](../../includes/mobile-services-selector-get-started.md)]

# <a name="getting-started"> </a>Mobile Services の使用

このチュートリアルでは、Azure の Mobile Services を使用してアプリケーションにクラウドベースのバックエンド サービスを追加する方法を示します。このチュートリアルでは、新しい Mobile Services と、新しい Mobile Services にアプリ データを保存する簡単な _To do list_ アプリの両方を作成します。 

完成したアプリケーションのスクリーンショットは次のようになります。

![][3]

### <a name="additional-requirements"></a>その他の要件

このチュートリアルを完了するには、PhoneGap ツール (Windows Phone 8 プロジェクトの場合は v3.2 以降が必要です) が必要です。 

PhoneGap は、複数のプラットフォーム向けの開発をサポートします。PhoneGap ツール自体に加えて、対象としている各プラットフォーム用のツールをインストールする必要があります。

- Windows Phone:[Visual Studio 2012 Express for Windows Phone](https://go.microsoft.com/fwLink/p/?LinkID=268374) のインストール
- iOS:[Xcode] (v4.4 以上が必要) のインストール
- Android:[Android Developer Tools][Android SDK] のインストール
	<br/>(Android 向けのMobile Services SDK は Android 2.2 以降用のアプリケーションをサポートします。Android 4.2 以降がクイック スタート アプリケーションの実行に必要です。)

## <a name="create-new-service"> </a>新しい Mobile Services を作成する

[AZURE.INCLUDE [mobile-services-create-new-service](../../includes/mobile-services-create-new-service.md)]

<h2>新しい PhoneGap アプリを作成する</h2>

Mobile Services を作成したら、管理ポータルの簡単なクイック スタートに従って、新しいアプリケーションを作成するか、既存のアプリケーションを変更して Mobile Services に接続できます。 

ここでは、Mobile Services に接続された新しい PhoneGap アプリケーションを作成します。

1.  管理ポータルで、**[Mobile Services]** をクリックし、先ほど作成した Mobile Services をクリックします。

2. [クイック スタート] タブの **[プラットフォームの選択]** で **[PhoneGap]** を選択し、**[新しい PhoneGap アプリケーションを作成する]** を展開します。

   	![][0]

   	これにより、Mobile Services に接続された PhoneGap アプリケーションを作成するための簡単な 3 つの手順が表示されます。

  	![][1]

3. まだインストールしていない場合は、PhoneGap および、プラットフォーム開発ツールの少なくとも 1 つ (Windows Phone、iOS、または Android) をダウンロードしてインストールします。

4. **[TodoItems テーブルを作成する]** をクリックして、アプリケーション データを格納するテーブルを作成します。

5. **[アプリケーションをダウンロードして実行する]** の下の **[ダウンロード]** をクリックします。 

	これにより、Mobile Services に接続されている _To do list_ サンプル アプリケーションのプロジェクトが、Mobile Services JavaScript SDK と共にダウンロードされます。圧縮されたプロジェクト ファイルをローカル コンピューターに保存し、保存場所を書き留めておいてください。

## 新しい PhoneGap アプリケーションを実行する

このチュートリアルの最後に、新しいアプリケーションをビルドして実行します。 

1.	圧縮されたプロジェクト ファイルの保存場所を参照し、ファイルをコンピューター上に展開します。 

2.	各プラットフォームについてここに示す手順に従って、プロジェクトを開いて実行します。

	+ **Windows Phone 8**

	1. Windows Phone 8:Visual Studio 2012 Express for Windows Phone の **platforms\wp8** フォルダーにある .sln ファイルを開きます。
	
	2. **F5** キーを押してプロジェクトをリビルドし、アプリケーションを開始します。
	
	  	![][2]

	+ **iOS**

	1. Xcode の **platforms/ios** フォルダーのプロジェクトを開きます。
	
	2. **[実行]** をクリックしてプロジェクトをビルドし、このプロジェクトの既定である iPhone エミュレーターでアプリケーションを開始します。
	
	  	![][3]

	+ **Android**

		1. Eclipse で、**[File]**、**[Import]** の順にクリックし、**[Android]** を展開します。**[Existing Android Code into Workspace]** をクリックし、**[Next]** をクリックします。 
		
		2. **[Browse]** をクリックします。展開したプロジェクト ファイルの場所を参照し、**[OK]** をクリックします。TodoActivity プロジェクトのチェック ボックスがオンになっていることを確認し、**[Finish]** をクリックします。 <p>これにより、プロジェクト ファイルが現在のワークスペースにインポートされます。</p>
		
		3. **[Run]** メニューの **[Run]** をクリックして、Android エミュレーター内でプロジェクトを開始します。
		
			![][4]
	
		>[AZURE.NOTE]プロジェクトを Android エミュレーターで実行するには、Android Virtual Device (AVD) を 1 つ以上定義する必要があります。これらのデバイスを作成および管理するには、AVD Manager を使用します。
			
	
3. 上で示したモバイル エミュレーターのいずれかでアプリケーションを起動した後、テキスト ボックスにテキストを入力し、**[追加]** をクリックします。

	これで、Azure でホストされている新しい Mobile Services に POST 要求が送信されます。要求のデータは **TodoItems** テーブルに挿入されます。テーブルに格納された項目が Mobile Services によって返され、データが一覧に表示されます。

	> [AZURE.IMPORTANT] メイン プロジェクトが PhoneGap ツールで再ビルドされる場合、このプラットフォーム プロジェクトへの変更が上書きされます。以下のセクションで説明されているように、代わりの方法として、変更はプロジェクトのルートの www ディレクトリで行います。

4. 管理ポータルに戻り、<strong>[データ]</strong> タブ、<strong>TodoItems</strong> テーブルの順にクリックします。

	![](./media/mobile-services-javascript-backend-phonegap-get-started/mobile-data-tab.png)

	これで、アプリケーションによってテーブルに挿入されたデータを参照できます。

	![](./media/mobile-services-javascript-backend-phonegap-get-started/mobile-data-browse.png)
	

## アプリケーションを更新して、各プラットフォーム用にプロジェクトを再ビルドする

1. ´www´ ディレクトリ (この場合は ´todolist/www´) のコード ファイルを変更します。

2. すべてのターゲット プラットフォームのツールがシステム パスでアクセスできることを確認します。 

2. ルート プロジェクト ディレクトリでコマンド プロンプトを開き、次のプラットフォームに固有のコマンドのいずれかを実行します。 

	+ **Windows Phone**

		Visual Studio 開発者用コマンド プロンプトで次のコマンドを実行します。

    		phonegap local build wp8

	+ **iOS**
 
		ターミナルを開き、次のコマンドを実行します。

    		phonegap local build ios

	+ **Android**

		コマンド プロンプトまたはターミナル ウィンドウを開き、次のコマンドを実行します。 

		    phonegap local build android


4. 前のセクションで説明したように、適切な開発環境で各プロジェクトを開きます。

>[AZURE.NOTE]Mobile Services にアクセスして js/index.js ファイルにあるデータを照会および挿入するコードを確認できます。

## <a name="next-steps"> </a>次のステップ
クイック スタートはこれで完了です。Mobile Services で重要になるこれ以外の作業については、以下のトピックをご覧ください。 

* [データの使用]
  <br/>Mobile Services を使用してデータの格納とクエリを実行する方法について説明します。

* [認証の使用]
  <br/>ID プロバイダーを使用してアプリケーションのユーザーを認証する方法について説明します。
  
<!-- Images. -->
[0]: ./media/mobile-services-javascript-backend-phonegap-get-started/portal-screenshot1.png
[1]: ./media/mobile-services-javascript-backend-phonegap-get-started/portal-screenshot2.png
[2]: ./media/mobile-services-javascript-backend-phonegap-get-started/mobile-portal-quickstart-wp8.png
[3]: ./media/mobile-services-javascript-backend-phonegap-get-started/mobile-portal-quickstart-ios.png
[4]: ./media/mobile-services-javascript-backend-phonegap-get-started/mobile-portal-quickstart-android.png

<!-- URLs. -->
[データの使用]: mobile-services-html-get-started-data.md
[認証の使用]: mobile-services-html-get-started-users.md
[プッシュ通知の使用]: /develop/mobile/tutorials/mobile-services-html-get-started-push
[Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=280125
[管理ポータル]: https://manage.windowsazure.com/
[Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[Visual Studio 2012 Express for Windows Phone]: https://go.microsoft.com/fwLink/p/?LinkID=268374



<!--HONumber=52--> 