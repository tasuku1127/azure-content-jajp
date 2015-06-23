<properties 
	pageTitle="Azure Notification Hubs の使用" 
	description="Azure Notification Hubs を使用してプッシュ通知を行う方法について説明します。" 
	services="notification-hubs" 
	documentationCenter="android" 
	authors="RickSaling" 
	manager="dwrede" 
	editor=""/>
<tags 
	ms.service="notification-hubs" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="mobile-android" 
	ms.devlang="java" 
	ms.topic="article" 
	ms.date="03/15/2014" 
	ms.author="ricksal"/>

# Notification Hubs の使用

[AZURE.INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

このトピックでは、Azure Notification Hubs を使用して Android アプリケーションにプッシュ通知を送信する方法について説明します。 
このチュートリアルでは、Google Cloud Messaging (GCM) を使用してプッシュ通知を受信する空の Android アプリケーションを作成します。完了すると、通知ハブを使用して、アプリケーションを実行するすべてのデバイスにプッシュ通知をブロードキャストできるようになります。

このチュートリアルでは、プッシュ通知を有効にするための、次の基本的な手順について説明します。

* [Google Cloud Messaging を有効にする](#register)
* [Notification Hubs を構成する](#configure-hub)
* [Notification Hubs にアプリケーションを接続する](#connecting-app)
* [アプリケーションへの通知の送信方法](#send)
* [アプリケーションのテスト](#run-app)

このチュートリアルでは、通知ハブを使用した簡単なブロードキャスト シナリオのデモンストレーションを行います。通知ハブを使用してデバイスの特定のユーザーとグループに対応する方法を理解するために、次のチュートリアルも一緒に参照してください。 

このチュートリアルには、次のものが必要です。

+ Android SDK (前提条件として Eclipse を使用する必要があります)。<a href="http://go.microsoft.com/fwlink/?LinkId=389797">ここ</a>からダウンロードできます。
+ [モバイル サービス Android SDK]

このチュートリアルを完了することは、Android アプリケーションの他のすべての通知ハブ チュートリアルの前提条件です。 

> [AZURE.NOTE] このチュートリアルを完了するには、アクティブな Azure アカウントが必要です。アカウントがない場合は、無料の試用アカウントを数分で作成できます。詳細については、[Azure の無料評価版サイト](http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fja-jp%2Fdocumentation%2Farticles%2Fnotification-hubs-android-get-started%2F)を参照してください。

##<a id="register"></a>Google Cloud Messaging を有効にする

[AZURE.INCLUDE [mobile-services-enable-Google-cloud-messaging](../../includes/mobile-services-enable-Google-cloud-messaging.md)]


##<a id="configure-hub"></a>Notification Hub を構成する

[AZURE.INCLUDE [notification-hubs-android-configure-push](../../includes/notification-hubs-android-configure-push.md)]

##<a id="connecting-app"></a>Notification Hub にアプリを接続する

###新しい Android プロジェクトの作成

1. Eclipse ADT で新しい Android プロジェクトを作成します ([File]、[New]、[Android Application])。

   	![][13]

2.  **[最低限必要な SDK]** が  *API 8:Android 2.2 (Froyo)* に設定されていること、およびその次の 2 つの SDK 項目が利用可能な最新バージョンに設定されていることを確認します。[次へ] を選択し、ウィザードに従い、**[Create activity]** が選択されていることを確認して、空のアクティビティを作成します。次のボックスで既定のランチャー アイコンを受け入れ、最後のボックスで **[完了]** をクリックします。

   	![][14]

###プロジェクトへの Google Play Services の追加

[AZURE.INCLUDE [Add Play Services](../../includes/mobile-services-add-Google-play-services.md)]

###コードの追加

1. Notification Hubs Android SDK を<a href="https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409">ここ</a>からダウンロードします。.zip ファイルを展開し、Package Explorer で notificationhubs\notification-hubs-0.1.jar ファイルをプロジェクトの \libs ディレクトリにコピーします。

2. [モバイル サービス Android SDK] をダウンロードし、解凍します。**notifications** フォルダーを開き、**notifications-1.0.1.jar** ファイルを Eclipse プロジェクトの  *libs* フォルダーにコピーした後、 *libs* フォルダーを最新の情報に更新します。

    > [AZURE.NOTE] ファイル名の末尾にある数値は、今後の SDK リリースで変更される可能性があります。

	ここで、GCM から  *registrationId* を取得し、registrationId を使用してアプリケーション インスタンスを通知ハブに登録するようにアプリケーションを設定します。

3. AndroidManifest.xml ファイルで、<uses-sdk/> 要素のすぐ下に次の行を追加します。`<your package>` を、手順 1 でアプリケーションに選択したパッケージに置き換えます (この例では `com.yourCompany.wams_notificationhubs`)。

        <uses-permission android:name="android.permission.INTERNET"/>
		<uses-permission android:name="android.permission.GET_ACCOUNTS"/>
		<uses-permission android:name="android.permission.WAKE_LOCK"/>
		<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

		<permission android:name="<your package>.permission.C2D_MESSAGE" android:protectionLevel="signature" />
		<uses-permission android:name="<your package>.permission.C2D_MESSAGE"/>

4. **MainActivity** クラスで、次のステートメントを追加します。

		import android.os.AsyncTask;	
		import com.google.android.gms.gcm.*;
		import com.microsoft.windowsazure.messaging.*;
		import com.microsoft.windowsazure.notifications.NotificationsManager;


5. クラスの最上位に、次のプライベート メンバーを追加します。

	> [AZURE.NOTE] SENDER_ID に、先に取得したプロジェクト番号が設定されていることを確認します。

		private String SENDER_ID = "<your project number>";
		private GoogleCloudMessaging gcm;
		private NotificationHub hub;

6. **OnCreate** メソッドで次のコードを追加し、プレースホルダーを、前のセクションで取得したリッスン アクセス権を持つ接続文字列と、(完全な URL では**なく**) Azure でハブのページの最上部に表示される通知ハブの名前に置き換えます。

		NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);

		gcm = GoogleCloudMessaging.getInstance(this);
        
		String connectionString = "<your listen access connection string>";
		hub = new NotificationHub("<your notification hub name>", connectionString, this);
		
		registerWithNotificationHubs();

7. MainActivity.java で、次のメソッドを作成します。

		@SuppressWarnings("unchecked")
		private void registerWithNotificationHubs() {
		   new AsyncTask() {
		      @Override
		      protected Object doInBackground(Object... params) {
		         try {
		            String regid = gcm.register(SENDER_ID);
		            hub.register(regid);
		         } catch (Exception e) {
		            return e;
		         }
		         return null;
		     }
		   }.execute(null, null, null);
		}

8. Android には通知が表示されないため、独自の受信者を記述する必要があります。**AndroidManifest.xml** で、`<application/>` 要素内に次の要素を追加します。

	> [AZURE.NOTE] プレースホルダーは、パッケージ名に置き換えてください。

        <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
            android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="**my_app_package**" />
            </intent-filter>
        </receiver>


9. パッケージ エクスプローラーで ( `src` ノードの下にある) パッケージを右クリックし、**[New]**、**[Class]** の順にクリックします。

10. **[Name]** に  `MyHandler` と入力し、**[Superclass]** に  `com.microsoft.windowsazure.notifications.NotificationsHandler` と入力してから、**[Finish]** をクリックします。

	![][6]

	これで、新しい MyHandler クラスが作成されます。

11. 次の import ステートメントを追加します。

		import android.app.NotificationManager;
		import android.app.PendingIntent;
		import android.content.Context;
		import android.content.Intent;
		import android.os.Bundle;
		import android.support.v4.app.NotificationCompat;
		

12. 次のコードをクラスに追加します。

		public static final int NOTIFICATION_ID = 1;
		private NotificationManager mNotificationManager;
		NotificationCompat.Builder builder;
		Context ctx;
	
		
		@Override
		public void onReceive(Context context, Bundle bundle) {
		    ctx = context;
		    String nhMessage = bundle.getString("msg");
	
		    sendNotification(nhMessage);
		}
	
		private void sendNotification(String msg) {
			mNotificationManager = (NotificationManager)
		              ctx.getSystemService(Context.NOTIFICATION_SERVICE);
	
		    PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
		          new Intent(ctx, MainActivity.class), 0);
	
		    NotificationCompat.Builder mBuilder =
		          new NotificationCompat.Builder(ctx)
		          .setSmallIcon(R.drawable.ic_launcher)
		          .setContentTitle("Notification Hub Demo")
		          .setStyle(new NotificationCompat.BigTextStyle()
		                     .bigText(msg))
		          .setContentText(msg);
	
		     mBuilder.setContentIntent(contentIntent);
		     mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
		}
	

##<a name="send"></a>アプリに通知を送信する方法

>[AZURE.NOTE]Notification Hubs を使用すれば、<a href="http://msdn.microsoft.com/library/windowsazure/dn223264.aspx">REST インターフェイス</a>を使用するどのバックエンドからでも通知が送信できます。このチュートリアルでは、.NET コンソール アプリケーションを使用して通知を送信します。Notification Hubs に統合されている Azure Mobile Services バックエンドから通知を送信する方法の例については、「[Mobile Services でのプッシュ通知の使用](../mobile-services-javascript-backend-android-get-started-push.md)」を参照してください。REST API を使用して通知を送信する方法の例については、「[Java から Notification Hubs を使用する方法](notification-hubs-java-backend-how-to.md)」、 または、「[PHP から Notification Hubs を使用する方法](notification-hubs-php-backend-how-to.md)」を参照してください。.

1. Visual Studio で、**[ファイル]** メニューから、**[新規作成]**、**[プロジェクト]** をクリックし、**[Visual C#]** で **[Windows]**、**[コンソール アプリケーション]**、**[OK]** の順にクリックします。  

   	![][20]

	これで、新しいコンソール アプリケーション プロジェクトが作成されます。

2. **[ツール]** メニューで **[ライブラリ パッケージ マネージャー]**、**[パッケージ マネージャー コンソール]** の順にクリックします。 

	これで、パッケージ マネージャー コンソールが表示されます。

3. コンソール ウィンドウで、次のコマンドを実行します。

        Install-Package WindowsAzure.ServiceBus
    
	これで、<a href="http://nuget.org/packages/WindowsAzure.ServiceBus/">WindowsAzure.ServiceBus NuGet パッケージ</a>を使用して Azure Service Bus SDK への参照が追加されます。 

4. Program.cs ファイルを開き、次の  `using` ステートメントを追加します。

        using Microsoft.ServiceBus.Notifications;

5. **Program** クラスで、次のメソッドを追加します。

        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            await hub.SendGcmNativeNotificationAsync("{ "data" : {"msg":"Hello from Azure!"}}");
        }

   	"hub name" プレースホルダーは、ポータルの **[通知ハブ]** タブに表示される通知ハブの名前に置き換えてください。また、接続文字列のプレースホルダーは、「通知ハブを構成する」で取得した **DefaultFullSharedAccessSignature** という接続文字列に置き換えてください。 

	>[AZURE.NOTE]**リッスン** アクセスではなく、**フル** アクセスを持つ接続文字列を使用してください。リッスン アクセス文字列に通知を送信するアクセス許可はありません。

5. 次に、**Main** メソッド内に次の行を追加します。

         SendNotificationAsync();
		 Console.ReadLine();

##<a name="run-app"></a>アプリのテスト

エミュレーターでアプリケーションをテストする前に、こうしたエミュレーターの構成手順を完了する必要があります (物理デバイスでテストする場合はスキップ)。

1. Google API をサポートしている Android Virtual Device (AVD) を使用します。

2. **[Window]** で **[Android Virtual Device Manager]** をクリックし、デバイスを選択してから **[Edit]** をクリックします。

   	![][18]

3. **[Target]** で **[Google APIs]** を選択し、**[OK]** をクリックします。

   	![][19]

4. 	プッシュ通知を受信するには、Android Virtual Device で Google アカウントを設定する必要があります (エミュレーターで、<strong>[Settings]</strong> に移動して <strong>[Add Account]</strong> をクリックします)。さらに、エミュレーターがインターネットに接続されていることを確認します。


次の手順を使用して、アプリケーションをデバイスまたはエミュレーターで実行します。

1. 上部の Eclipse ツール バーで、**[Run]** をクリックし、アプリケーションを選択します。 
 
	これによりエミュレーターが起動し (エミュレーターを使用している場合)、アプリケーションが読み込まれ、実行されます。アプリケーションにより、GCM から  *registrationId* が取得され、通知ハブに登録されます。

3. Visual Studio で F5 キーを押して、コンソール アプリケーションを実行します。 

	通知がアプリケーションに送信されます。  
 
5. アイコンが通知領域 (左上隅) に表示されたら、通知ドロワを引き出して、通知を表示します。  

   	![][21]

## <a name="next-steps"> </a>次のステップ

この簡単な例では、すべての Android デバイスに通知をブロードキャストします。特定のユーザーをターゲットとするには、「[Notification Hubs を使用したユーザーへのプッシュ通知]」を参照してください。一方、対象グループごとにユーザーを区分する場合は、「[Notification Hubs を使用したニュース速報の送信]」を参照してください。Notification Hubs の使用方法の詳細については、「[Notification Hubs の概要]」を参照してください。


<!-- Images. -->
[1]: ./media/notification-hubs-android-get-started/mobile-services-google-new-project.png
[2]: ./media/notification-hubs-android-get-started/mobile-services-google-create-server-key.png
[3]: ./media/notification-hubs-android-get-started/mobile-services-google-create-server-key2.png
[4]: ./media/notification-hubs-android-get-started/mobile-services-google-create-server-key3.png
[5]: ./media/notification-hubs-android-get-started/mobile-services-google-enable-GCM.png
[6]: ./media/notification-hubs-android-get-started/notification-hub-android-new-class.png

[12]: ./media/notification-hubs-android-get-started/notification-hub-connection-strings.png

[13]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app.png
[14]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app2.png
[15]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png
[16]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app5.png
[17]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app6.png

[18]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app7.png
[19]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app8.png

[20]: ./media/notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/notification-hubs-android-get-started/notification-hub-android-toast.png
[22]: ./media/notification-hubs-android-get-started/notification-hub-scheduler1.png
[23]: ./media/notification-hubs-android-get-started/notification-hub-scheduler2.png
[29]: ./media/mobile-services-android-get-started-push/mobile-eclipse-import-Play-library.png

<!-- URLs. -->
[モバイル サービス Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[ライブラリ プロジェクトの参照]: http://go.microsoft.com/fwlink/?LinkId=389800
[Azure 管理ポータル]: https://manage.windowsazure.com/
[Notification Hubs の概要]: http://msdn.microsoft.com/library/jj927170.aspx

[Notification Hubs を使用したユーザーへのプッシュ通知]: notification-hubs-aspnet-backend-android-notify-users.md
[Notification Hubs を使用したニュース速報の送信]: notification-hubs-aspnet-backend-android-breaking-news.md


<!--HONumber=49--> 