<properties linkid="notification-hubs-php-back-end-how-to" urlDisplayName="How to use Notification Hubs with PHP" pageTitle="How to use Notification Hubs with PHP" metaKeywords="" description="Learn how to use Azure Notification Hubs from a PHP back-end." metaCanonical="" services="mobile-services,notification-hubs,push,php" documentationCenter="" title="How to use Notification Hubs with PHP" authors="elioda" solutions="" manager="" editor="" />

<tags ms.service="notification-hubs" ms.workload="mobile" ms.tgt_pltfrm="mobile-multiple" ms.devlang="php" ms.topic="article" ms.date="01/01/1900" ms.author="elioda"></tags>

# Java/PHP から Notification Hubs を使用する方法

<div class="dev-center-tutorial-selector sublanding"> 
<a href="/ja-jp/documentation/articles/notification-hubs-java-backend-how-to/" title="Java">Java</a><a href="/ja-jp/documentation/articles/notification-hubs-php-backend-how-to/" title="PHP" class="current">PHP</a>
</div>

MSDN のトピック「[通知ハブの REST API][通知ハブの REST API]」の説明にあるように Notification Hub REST インターフェイスを使用して、Java/PHP/Ruby バックエンドから Notification Hubs のすべての機能にアクセスできます。

このトピックでは、次の方法について説明します。

-   PHP で Notification Hubs 機能の REST クライアントを記述します。
-   選択したモバイル プラットフォームの「[Notification Hubs の使用][Notification Hubs の使用]」に従って、PHP のバックエンド部分を実装します。

## クライアント インターフェイス

メイン クライアント インターフェイスは、[.NET 通知ハブの SDK][.NET 通知ハブの SDK] で利用可能なものと同じメソッドを提供します。これにより、このサイトで現在利用できる、およびインターネットのコミュニティによって随時追加されるすべてのチュートリアルとサンプルを直接変換できます。

「[PHP REST wrapper sample (PHP REST ラッパー サンプル)][PHP REST wrapper sample (PHP REST ラッパー サンプル)]」で利用可能なすべてのコードを検索できます。

たとえば、クライアントを作成する場合:

    $hub = new NotificationHub("connection string", "hubname"); 

iOS ネイティブ通知を送信する場合:

    $notification = new Notification("apple", '{"aps":{"alert": "Hello!"}}');
    $hub->sendNotification($notification);

## 実装

まだインストールが済んでいない場合は、「[Notification Hubs の使用][Notification Hubs の使用]」で、バックエンドを実装する最後のセクションまでの説明に従ってください。
また、「[PHP REST wrapper sample (PHP REST ラッパー サンプル)][PHP REST wrapper sample (PHP REST ラッパー サンプル)]」のコードを使用する場合は、「[チュートリアルを完了する][チュートリアルを完了する]」セクションに直接進んでください。

REST ラッパーすべての実装の詳細については、[MSDN の記事][MSDN の記事]を参照してください。このセクションでは、Notification Hubs REST エンドポイントにアクセスするために必要な主要手順の PHP 実装について説明します。

1.  接続文字列を解析する
2.  認証トークンを生成する
3.  HTTP 呼び出しを実行する

### 接続文字列を解析する

接続文字列を解析するコンストラクターを持つクライアントを実装するメイン クラスです。

    class NotificationHub {
        const API_VERSION = "?api-version=2013-10";

        private $endpoint;
        private $hubPath;
        private $sasKeyName;
        private $sasKeyValue;

        function __construct($connectionString, $hubPath) {
            $this->hubPath = $hubPath;

            $this->parseConnectionString($connectionString);
        }

        private function parseConnectionString($connectionString) {
            $parts = explode(";", $connectionString);
            if (sizeof($parts) != 3) {
                throw new Exception("Error parsing connection string: " . $connectionString);
            }

            foreach ($parts as $part) {
                if (strpos($part, "Endpoint") === 0) {
                    $this->endpoint = "https" . substr($part, 11);
                } else if (strpos($part, "SharedAccessKeyName") === 0) {
                    $this->sasKeyName = substr($part, 20);
                } else if (strpos($part, "SharedAccessKey") === 0) {
                    $this->sasKeyValue = substr($part, 16);
                }
            }
        }
    }

### セキュリティ トークンを作成する

セキュリティ トークン作成の詳細は、[ここ][ここ]で参照できます。
現在の要求の URI および接続文字列から抽出した資格情報に基づいてトークンを作成するには、次のメソッドを **NotificationHub** クラスに追加する必要があります。

    private function generateSasToken($uri) {
        $targetUri = strtolower(rawurlencode(strtolower($uri)));

        $expires = time();
        $expiresInMins = 60;
        $expires = $expires + $expiresInMins * 60;
        $toSign = $targetUri . "\n" . $expires;

        $signature = rawurlencode(base64_encode(hash_hmac('sha256', $toSign, $this->sasKeyValue, TRUE)));

        $token = "SharedAccessSignature sr=" . $targetUri . "&sig="
                    . $signature . "&se=" . $expires . "&skn=" . $this->sasKeyName;

        return $token;
    }

### 通知を送信する

最初に、通知を表すクラスの定義を使用します。

    class Notification {
        public $format;
        public $payload;

        # array with keynames for headers
        # Note: Some headers are mandatory: Windows: X-WNS-Type, WindowsPhone: X-NotificationType
        # Note: For Apple you can set Expiry with header: ServiceBusNotification-ApnsExpiry in W3C DTF, YYYY-MM-DDThh:mmTZD (for example, 1997-07-16T19:20+01:00).
        public $headers;

        function __construct($format, $payload) {
            if (!in_array($format, ["template", "apple", "windows", "gcm", "windowsphone"])) {
                throw new Exception('Invalid format: ' . $format);
            }

            $this->format = $format;
            $this->payload = $payload;
        }
    }

このクラスは、ネイティブ通知本体のコンテナー、またはテンプレート通知の場合にはプロパティのセットと、形式 (ネイティブ プラットフォームまたはテンプレート) およびプラットフォーム固有のプロパティを含むヘッダーのセットです (Apple 有効期限プロパティや WNS ヘッダーなど)。

利用可能なすべてのオプションについては、「[REST API のメソッド][REST API のメソッド]」、および特定の通知プラットフォームの形式を参照してください。

このクラスを利用して、**NotificationHub** クラス内に送信通知メソッドを作成できます。

    public function sendNotification($notification) {
        $this->sendNotification($notification, "");
    }

    public function sendNotification($notification, $tagsOrTagExpression) {
        if (is_array($tagsOrTagExpression)) {
            $tagExpression = implode(" || ", $tagsOrTagExpression);
        } else {
            $tagExpression = $tagsOrTagExpression;
        }

        # build uri
        $uri = $this->endpoint . $this->hubPath . "/messages" . NotificationHub::API_VERSION;
        $ch = curl_init($uri);

        if (in_array($notification->format, ["template", "apple", "gcm"])) {
            $contentType = "application/json";
        } else {
            $contentType = "application/xml";
        }

        $token = $this->generateSasToken($uri);

        $headers = [
            'Authorization: '.$token,
            'Content-Type: '.$contentType,
            'ServiceBusNotification-Format: '.$notification->format
        ];

        if ("" !== $tagExpression) {
            $headers[] = 'ServiceBusNotification-Tags: '.$tagExpression;
        }

        # add headers for other platforms
        if (is_array($notification->headers)) {
            $headers = array_merge($headers, $notification->headers);
        }
        
        curl_setopt_array($ch, array(
            CURLOPT_POST => TRUE,
            CURLOPT_RETURNTRANSFER => TRUE,
            CURLOPT_SSL_VERIFYPEER => FALSE,
            CURLOPT_HTTPHEADER => $headers,
            CURLOPT_POSTFIELDS => $notification->payload
        ));

        // Send the request
        $response = curl_exec($ch);

        // Check for errors
        if($response === FALSE){
            throw new Exception(curl_error($ch));
        }

        $info = curl_getinfo($ch);

        if ($info['http_code'] <> 201) {
            throw new Exception('Error sending notificaiton: '. $info['http_code'] . ' msg: ' . $response);
        }
    } 

上記のメソッドは、HTTP POST 要求、および通知を送信する正しい本体とヘッダーを通知ハブの /messages エンドポイントに送信します。

## <a name="complete-tutorial"></a>チュートリアルを完了する

ここで、PHP バックエンドから通知を送信して、使用についてのチュートリアルを完了できます。

Notification Hubs クライアントを初期化します (「[Notification Hubs の使用][Notification Hubs の使用]」の説明に従って接続文字列とハブ名を置き換えます)。
$hub = new NotificationHub("connection string", "hubname");

次に、ターゲット モバイル プラットフォームに応じて送信コードを追加します。

### Windows ストアおよび Windows Phone 8.1 (非 Silverlight)

    $toast = '<toast><visual><binding template="ToastText01"><text id="1">Hello from PHP!</text></binding></visual></toast>';
    $notification = new Notification("windows", $toast);
    $notification->headers[] = 'X-WNS-Type: wns/toast';
    $hub->sendNotification($notification);

### iOS

    $alert = '{"aps":{"alert":"Hello from PHP!"}}';
    $notification = new Notification("apple", $alert);
    $hub->sendNotification($notification);

### Android

    $message = '{"data":{"msg":"Hello from PHP!"}}';
    $notification = new Notification("gcm", $message);
    $hub->sendNotification($notification);

### Windows Phone 8.0 および 8.1 Silverlight

    $toast = '<?xml version="1.0" encoding="utf-8"?>' .
                '<wp:Notification xmlns:wp="WPNotification">' .
                   '<wp:Toast>' .
                        '<wp:Text1>Hello from PHP!</wp:Text1>' .
                   '</wp:Toast> ' .
                '</wp:Notification>';
    $notification = new Notification("mpns", $toast);
    $notification->headers[] = 'X-WindowsPhone-Target : toast';
    $notification->headers[] = 'X-NotificationClass : 2';
    $hub->sendNotification($notification);

### Kindle Fire

    $message = '{"data":{"msg":"Hello from PHP!"}}';
    $notification = new Notification("adm", $message);
    $hub->sendNotification($notification);

PHP コードを実行すると、ターゲット デバイスに表示される通知が生成されます。

## 次のステップ

このトピックでは、Notification Hubs 用の単純な Java REST クライアントの作成方法を説明しました。次は、以下を実行できます。

-   [PHP REST ラッパー サンプル][PHP REST wrapper sample (PHP REST ラッパー サンプル)]をすべてダウンロードします。サンプルには上記のコード、および登録管理のコードがすべて含まれています。
-   引き続き、「[ニュース速報チュートリアル]」で Notification Hubs のタグ付け機能について学習してください。
-   個別ユーザーへの通知のプッシュについては、「[ユーザーへの通知チュートリアル]」で学習してください。

  [Java]: /ja-jp/documentation/articles/notification-hubs-java-backend-how-to/ "Java"
  [PHP]: /ja-jp/documentation/articles/notification-hubs-php-backend-how-to/ "PHP"
  [通知ハブの REST API]: http://msdn.microsoft.com/ja-jp/library/dn223264.aspx
  [Notification Hubs の使用]: http://azure.microsoft.com/ja-jp/documentation/articles/notification-hubs-ios-get-started/
  [.NET 通知ハブの SDK]: http://msdn.microsoft.com/ja-jp/library/jj933431.aspx
  [PHP REST wrapper sample (PHP REST ラッパー サンプル)]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/notificationhubs-rest-php
  [チュートリアルを完了する]: #complete-tutorial
  [MSDN の記事]: http://msdn.microsoft.com/ja-jp/library/dn530746.aspx
  [ここ]: http://msdn.microsoft.com/ja-jp/library/dn495627.aspx
  [REST API のメソッド]: http://msdn.microsoft.com/ja-jp/library/dn495827.aspx