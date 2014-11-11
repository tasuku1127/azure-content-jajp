<properties title="Azure Notification Hubs Notify Users" pageTitle="Azure Notification Hubs Notify Users" metaKeywords="Azure push notifications, Azure notification hubs" description="Learn how to send secure push notifications in Azure. Code samples written in C# using the .NET API." documentationCenter="Mobile" metaCanonical="" disqusComments="1" umbracoNaviHide="0" authors="elioda" />

<tags ms.service="notification-hubs" ms.workload="mobile" ms.tgt_pltfrm="mobile-android" ms.devlang="java" ms.topic="article" ms.date="01/01/1900" ms.author="elioda" />

# Azure Notification Hubs によるユーザーへの通知

<div class="dev-center-tutorial-selector sublanding"> 
<a href="/ja-jp/documentation/articles/notification-hubs-windows-dotnet-notify-users/" title="Windows ユニバーサル">Windows ユニバーサル</a><a href="/ja-jp/documentation/articles/notification-hubs-aspnet-backend-ios-notify-users/" title="iOS">iOS</a>
<a href="/ja-jp/documentation/articles/notification-hubs-aspnet-backend-android-notify-users/" title="Android" class="current">Android</a>
</div>

Azure でプッシュ通知がサポートされたことで、マルチプラットフォームに対応し、簡単に使用できる、スケールアウトされたプッシュ通知インフラストラクチャを利用できるようになりました。これにより、モバイル プラットフォーム向けアプリケーション (コンシューマー用途およびエンタープライズ用途) にプッシュ通知機能を実装する作業が大幅に簡略化されます。このチュートリアルでは、Azure Notification Hubs を使用して特定のデバイスで特定のアプリケーション ユーザーにプッシュ通知を送信する方法について説明します。ガイダンス トピック「[アプリ バックエンドからの登録][アプリ バックエンドからの登録]」に示すように、ASP.NET WebAPI バックエンドを使用してクライアントを認証し、通知を生成します。このチュートリアルは、「**Notification Hubs の使用**」チュートリアルで作成した通知ハブが基になっています。

> [AZURE.NOTE] このチュートリアルでは、「[Getting Started with Notification Hubs (Android) (Notification Hubs (Android) の使用)][Getting Started with Notification Hubs (Android) (Notification Hubs (Android) の使用)]」での説明に従って通知が作成され、構成されていると想定しています。

[WACOM.INCLUDE [notification-hubs-aspnet-backend-notifyusers](../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## Android プロジェクトを作成する

次の手順では、Android アプリケーションを作成します。

1.  「[Getting Started with Notification Hubs (Android) (Notification Hubs (Android) の使用)][Getting Started with Notification Hubs (Android) (Notification Hubs (Android) の使用)]」チュートリアルに従って、アプリケーションを作成、構成して GCM からプッシュ通知を受け取ります。

2.  res/layout/activity\_main.xml ファイルを開き、コンテンツを次の内容と置き換えます。

        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:tools="http://schemas.android.com/tools"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:paddingBottom="@dimen/activity_vertical_margin"
            android:paddingLeft="@dimen/activity_horizontal_margin"
            android:paddingRight="@dimen/activity_horizontal_margin"
            android:paddingTop="@dimen/activity_vertical_margin"
            tools:context="com.example.notifyusers.MainActivity"
            android:orientation="vertical">
            <EditText
                android:id="@+id/usernameText"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:ems="10"
                android:hint="@string/usernameHint" >
            </EditText>
            <EditText
                android:id="@+id/passwordText"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:ems="10"
                android:hint="@string/passwordHint"
                android:inputType="textPassword" />
            <Button
                android:id="@+id/buttonLogin"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/loginButton"
                android:onClick="login" />
            <Button
                android:id="@+id/buttonSend"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/sendButton"
                android:onClick="sendPush" />
        </LinearLayout>

3.  res/values/strings.xml ファイルを開き、次の行を追加します。

        <string name="usernameHint">Username</string>
        <string name="passwordHint">Password</string>
        <string name="loginButton">1. Log in</string>
        <string name="sendButton">2. Send push</string>

    main\_activity.xml の画像レイアウトは次のようになります。

    ![][0]

4.  ここで、**MainActivity** クラスと同じパッケージにクラス **RegisterClient** を作成します。必ず、前のセクションで取得したバックエンド エンドポイントで `{backend endpoint}` を置き換えてください。

        import java.io.IOException;
        import java.io.UnsupportedEncodingException;
        import java.util.Set;

        import org.apache.http.HttpResponse;
        import org.apache.http.HttpStatus;
        import org.apache.http.client.ClientProtocolException;
        import org.apache.http.client.HttpClient;
        import org.apache.http.client.methods.HttpPost;
        import org.apache.http.client.methods.HttpPut;
        import org.apache.http.client.methods.HttpUriRequest;
        import org.apache.http.entity.StringEntity;
        import org.apache.http.impl.client.DefaultHttpClient;
        import org.apache.http.util.EntityUtils;
        import org.json.JSONArray;
        import org.json.JSONException;
        import org.json.JSONObject;

        import android.content.Context;
        import android.content.SharedPreferences;
        import android.util.Log;

        public class RegisterClient {
            private static final String PREFS_NAME = "ANHSettings";
            private static final String REGID_SETTING_NAME = "ANHRegistrationId";
            private static final String BACKEND_ENDPOINT = "{backend endpoint}/api/register";
            SharedPreferences settings;
            protected HttpClient httpClient;
            private String authorizationHeader;

            public RegisterClient(Context context) {
                super();
                this.settings = context.getSharedPreferences(PREFS_NAME, 0);
                httpClient =  new DefaultHttpClient();
            }

            public String getAuthorizationHeader() {
                return authorizationHeader;
            }

            public void setAuthorizationHeader(String authorizationHeader) {
                this.authorizationHeader = authorizationHeader;
            }

            public void register(String handle, Set<String> tags) throws ClientProtocolException, IOException, JSONException {
                String registrationId = retrieveRegistrationIdOrRequestNewOne(handle);

                JSONObject deviceInfo = new JSONObject();
                deviceInfo.put("Platform", "gcm");
                deviceInfo.put("Handle", handle);
                deviceInfo.put("Tags", new JSONArray(tags));

                int statusCode = upsertRegistration(registrationId, deviceInfo);

                if (statusCode == HttpStatus.SC_OK) {
                    return;
                } else if (statusCode == HttpStatus.SC_GONE){
                    settings.edit().remove(REGID_SETTING_NAME).commit();
                    registrationId = retrieveRegistrationIdOrRequestNewOne(handle);
                    statusCode = upsertRegistration(registrationId, deviceInfo);
                    if (statusCode != HttpStatus.SC_OK) {
                        Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                        throw new RuntimeException("Error upserting registration");
                    }
                } else {
                    Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                    throw new RuntimeException("Error upserting registration");
                }
            }

            private int upsertRegistration(String registrationId, JSONObject deviceInfo)
                    throws UnsupportedEncodingException, IOException,
                    ClientProtocolException {
                HttpPut request = new HttpPut(BACKEND_ENDPOINT+"/"+registrationId);
                request.setEntity(new StringEntity(deviceInfo.toString()));
                request.addHeader("Authorization", "Basic "+authorizationHeader);
                request.addHeader("Content-Type", "application/json");
                HttpResponse response = httpClient.execute(request);
                int statusCode = response.getStatusLine().getStatusCode();
                return statusCode;
            }

            private String retrieveRegistrationIdOrRequestNewOne(String handle) throws ClientProtocolException, IOException {
                if (settings.contains(REGID_SETTING_NAME))
                    return settings.getString(REGID_SETTING_NAME, null);

                HttpUriRequest request = new HttpPost(BACKEND_ENDPOINT+"?handle="+handle);
                request.addHeader("Authorization", "Basic "+authorizationHeader);
                HttpResponse response = httpClient.execute(request);
                if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                    Log.e("RegisterClient", "Error creating registrationId: " + response.getStatusLine().getStatusCode());
                    throw new RuntimeException("Error creating Notification Hubs registrationId");
                }
                String registrationId = EntityUtils.toString(response.getEntity());
                registrationId = registrationId.substring(1, registrationId.length()-1);

                settings.edit().putString(REGID_SETTING_NAME, registrationId).commit();

                return registrationId;
            }
        }

    プッシュ通知を登録するために、このコンポーネントはアプリケーション バックエンドにアクセスするのに必要な REST 呼び出しを実装します。「[アプリ バックエンドからの登録][アプリ バックエンドからの登録]」で説明しているとおり、通知ハブによって作成された *registrationIds* もローカルに格納されます。**[ログインして登録]** をクリックすると、ローカル ストレージに格納した認証トークンが使用されることに注意してください。

5.  **MainActivity** クラスで、**NotificationHub** のプライベート フィールドを削除し、**RegisterClient** に対するフィールドを追加します。

        //private NotificationHub hub;
        private RegisterClient registerClient;

6.  次に、**onCreate** メソッドで、**hub** フィールドの初期化と **registerWithNotificationHubs** を削除します。次に、**RegisterClient** クラスのインスタンスを初期化する次の行を追加します。メソッドには次の行が含まれています。

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            NotificationsManager.handleNotifications(this, SENDER_ID,
                    MyHandler.class);
            gcm = GoogleCloudMessaging.getInstance(this);

            registerClient = new RegisterClient(this);

            setContentView(R.layout.activity_main);
        }

7.  その後、次のメソッドを追加し、前のセクションで取得したバックエンド エンドポイントで `{backend endpoint}` を置き換えてください。

        @Override
        protected void onStart() {
            super.onStart();
            Button sendPush = (Button) findViewById(R.id.buttonSend);
            sendPush.setEnabled(false);
        }

        public void login(View view) throws UnsupportedEncodingException {
            this.registerClient.setAuthorizationHeader(getAuthorizationHeader());

            final Context context = this;
            new AsyncTask<Object, Object, Object>() {
                @Override
                protected Object doInBackground(Object... params) {
                    try {
                        String regid = gcm.register(SENDER_ID);
                        registerClient.register(regid, new HashSet<String>());
                    } catch (Exception e) {
                        Log.e("MainActivity", "Failed to register - " + e.getMessage());
                        return e;
                    }
                    return null;
                }

                protected void onPostExecute(Object result) {
                    Button sendPush = (Button) findViewById(R.id.buttonSend);
                    sendPush.setEnabled(true);
                    Toast.makeText(context, "Logged in and registered.",
                            Toast.LENGTH_LONG).show();
                }
            }.execute(null, null, null);
        }

        public void sendPush(View view) throws ClientProtocolException, IOException {
            new AsyncTask<Object, Object, Object>() {
                @Override
                protected Object doInBackground(Object... params) {
                    try {
                        HttpUriRequest request = new HttpPost("{backend endpoint}/api/notifications");
                        request.addHeader("Authorization", "Basic "+getAuthorizationHeader());
                        HttpResponse response = new DefaultHttpClient().execute(request);
                        if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                            Log.e("MainActivity", "Error sending notification" + response.getStatusLine().getStatusCode());
                            throw new RuntimeException("Error sending notification");
                        }
                    } catch (Exception e) {
                        Log.e("MainActivity", "Failed to send notification - " + e.getMessage());
                        return e;
                    }
                    return null;
                }
            }.execute(null, null, null);
        }

        private String getAuthorizationHeader() throws UnsupportedEncodingException {
            EditText username = (EditText) findViewById(R.id.usernameText);
            EditText password = (EditText) findViewById(R.id.passwordText);
            String basicAuthHeader = username.getText().toString()+":"+password.getText().toString();
            basicAuthHeader = Base64.encodeToString(basicAuthHeader.getBytes("UTF-8"), Base64.NO_WRAP);
            return basicAuthHeader;
        }

    **[ログイン]** に対するコールバックは、入力したユーザー名とパスワードに基づいて基本的な認証トークンを生成し (これは認証スキームが使用する任意のトークンを表します)、`RegisterClient` を使用してバックエンドを呼び出します。**[プッシュを送信する]** に対するコールバックは、バックエンドを呼び出し、このユーザーのすべてのデバイスに対してセキュリティで保護された通知をトリガーします。

## アプリケーションの実行

アプリケーションを実行するには、以下の手順に従います。

1.  **AppBackend** が Azure にデプロイされていることを確認します。Visual Studio を使用している場合は、**AppBackend** Web API アプリケーションを実行します。ASP.NET Web ページが表示されます。

2.  Eclipse で、物理的な Android デバイスまたはエミュレーターでアプリケーションを実行します。

3.  Android アプリケーションの UI で、ユーザー名とパスワードを入力します。文字列は任意ですが、値は同じである必要があります。

4.  Android アプリケーションの UI で、**[ログイン]** をクリックします。次に、**[プッシュを送信する]** をクリックします。

  [アプリ バックエンドからの登録]: http://msdn.microsoft.com/ja-jp/library/dn743807.aspx
  [notification-hubs-aspnet-backend-notifyusers]: ../includes/notification-hubs-aspnet-backend-notifyusers.md
  [0]: ./media/notification-hubs-aspnet-backend-android-notify-users/android-notify-users1.PNG