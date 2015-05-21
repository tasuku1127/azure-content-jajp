<properties 
	pageTitle="コンテンツの保護" 
	description="このトピックでは、Media Services でコンテンツを保護する方法の概要について説明します。" 
	authors="Juliako" 
	manager="dwrede" 
	editor="" 
	services="media-services" 
	documentationCenter=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/15/2015" 
	ms.author="juliako"/>


# コンテンツの保護

## 概要

Azure Media Services を使用すると、メディアがコンピューターから離れてから、保存、処理、配信されるまでのセキュリティ保護が可能になります。Media Services を操作する際の一般的なシナリオには次のようなものがあります。

### ストレージ内のコンテンツの保護、および動的に暗号化されたストリーミング メディアの配信  

動的暗号化を使用するには、暗号化されたコンテンツのストリーミング元となるストリーミング エンドポイントに、少なくとも 1 つのストリーミング占有ユニットが必要です。

1. 高品質な中間ファイルをアセットにアップロードします。ストレージ暗号化オプションをアセットに適用します。
1. アダプティブ ビットレート MP4 セットにエンコードします。ストレージ暗号化オプションを出力アセットに適用します。
1. 再生中に動的に暗号化するアセットの、暗号化コンテンツ キーを作成します。
2. コンテンツ キー承認ポリシーを構成します。
1. アセット配信ポリシーを構成します \(動的パッケージと動的暗号化で使用\)。
1. OnDemand ロケーターを作成してアセットを発行します。
1. 公開済みコンテンツをストリーミングします。  

このトピックでは、主な[概念](media-services-protecting-content-overview.md#concepts)の概要と、コンテンツの配信に関連する[作業](media-services-protecting-content-overview.md#tasks)の実行方法を示すトピックへのリンクを紹介します。

## <a id="concepts"></a>概念

### 資産暗号化オプション

アップロードするコンテンツの種類に応じて、Media Services によって提供されるさまざまな暗号化オプションから選択できます。

**None** – 暗号化は使用されません。これが既定値です。このオプションを使用した場合、送信経路上とストレージ内のいずれにおいてもコンテンツが保護されないので注意してください。

プログレッシブ ダウンロードを使用して MP4 を配信する場合は、このオプションを使用してコンテンツをアップロードします。

**StorageEncrypted** – AES 256 ビット暗号化を使用してクリア コンテンツをローカルに暗号化し、それを Azure Storage にアップロードすると、コンテンツが保存時に暗号化された状態で格納されます。ストレージの暗号化で保護された資産は、エンコーディングの前に自動的に暗号化が解除され、暗号化されたファイル システムに置かれます。その後、新しい出力資産として再度アップロードする前に必要に応じて再度暗号化されます。ストレージの暗号化の主な使用事例としては、高品質の入力メディア ファイルをディスクに保存するときに強力な暗号化を使用してセキュリティを保護する場合が挙げられます。

ストレージで暗号化された資産を配信するためには、資産の配信ポリシーを構成して、コンテンツの配信方法を Media Services に指示する必要があります。資産をストリーミングするには、ストリーミング サーバーでストレージ暗号化を解除し、指定された配信ポリシー \(AES、PlayReady、暗号化なしなど\) を使用してコンテンツをストリーミングする必要があります。

**CommonEncryptionProtected** - このオプションを使用するのは、共通暗号化または PlayReady DRM を使用して既に暗号化および保護されているコンテンツ \(たとえば、PlayReady DRM で保護されているスムーズ ストリーミング\) をアップロードする場合です。

**EnvelopeEncryptedProtected** – このオプションを使用するのは、AES \(高度暗号化標準\) で暗号化された HLS \(HTTP ライブ ストリーミング\) を保護または \(既に保護されている HLS を\) アップロードする場合です。AES で既に暗号化された HLS をアップロードする場合、HLS は Transform Manager によって暗号化されている必要があることに注意してください。

### 動的な暗号化

Microsoft Azure Media Services では、高度暗号化標準 \(AES\) \(128 ビット暗号化キーを使用\) と PlayReady DRM を使用して動的に暗号化したコンテンツを配信できます。

現時点では、以下のストリーミング形式を暗号化できます。HLS、MPEG DASH、およびスムーズ ストリーミング。HDS 形式のストリーミングやプログレッシブ ダウンロードは暗号化できません。

Media Services で資産を暗号化する場合は、暗号化キー \(CommonEncryption か EnvelopeEncryption\) を資産に関連付ける必要があります。また、キーの承認ポリシーを構成する必要があります。

資産の配信ポリシーを構成する必要もあります。ストレージで暗号化された資産をストリーミングする場合は、資産の配信ポリシーを構成して、資産の配信方法を指定する必要があります。

プレーヤーがストリームを要求すると、Media Services は指定されたキーを使用して、AES か PlayReady でコンテンツを動的に暗号化します。ストリームの暗号化を解除するには、プレーヤーはキー配信サービスからキーを要求します。ユーザーのキーの取得が承認されているかどうかを判断するために、サービスはキーに指定した承認ポリシーを評価します。

>[AZURE.NOTE]動的暗号化を活用するには、暗号化されたコンテンツの配信元となるストリーミング エンドポイントのオンデマンド ストリーミング ユニットを 1 つ以上取得する必要があります。詳細については、「[Media Services の規模の設定方法](media-services-manage-origins.md#scale_streaming_endpoints)」を参照してください。

### PlayReady DRM ライセンスと AES クリア キー配信サービス

Media Services は、承認されたクライアントに PlayReady ライセンスと AES クリア キーを配信するためのサービスを提供しています。Azure 管理ポータル、REST API、または Media Services SDK for .NET を使用して、ライセンスとキーの承認および認証ポリシーを構成できます。

ポータルを使用している場合、構成できるのは 1 つの AES ポリシー \(AES で暗号化されたすべてのコンテンツに適用されます\) および 1 つの PlayReady ポリシー \(PlayReady で暗号化されたすべてのコンテンツに適用されます\) であることに注意してください。さらに詳細に構成を制御したい場合は、Media Services SDK for .NET を使用してください。

### PlayReady ライセンス テンプレート

Media Services には PlayReady ライセンスを提供するサービスが用意されています。エンド ユーザーのプレーヤー \(Silverlight など\) は、PlayReady で保護されたコンテンツを再生しようとする際に、ライセンス配信サービスにライセンス取得要求を送信します。ライセンス サービスはその要求を承認した後、ライセンスを発行します。このライセンスはクライアントに送信され、指定されたコンテンツの暗号化解除と再生に用いられます。

ライセンスには、保護されたコンテンツをユーザーが再生しようとしたときに PlayReady DRM ランタイムが適用する権限や制限が含まれます。Media Services は、PlayReady ライセンスの構成用 API を備えています。詳細については、「[Media Services PlayReady ライセンス テンプレートの概要](https://msdn.microsoft.com/library/azure/dn783459.aspx)」を参照してください。

### トークン制限

コンテンツ キー承認ポリシーには、1 つまたは複数の承認制限 \(オープン、トークン制限、IP 制限\) を指定できます。トークン制限ポリシーには、STS \(セキュリティ トークン サービス\) によって発行されたトークンを含める必要があります。Media Services では、Simple Web Tokens \(SWT\) 形式と JSON Web Token \(JWT\) 形式のトークンがサポートされます。Media Services では、Secure Token Services は提供されません。トークンを発行するには、カスタム STS を作成するか、Microsoft Azure ACS を活用できます。STS は、トークン、トークンの制限の構成で指定した、指定されたキーと問題要求で署名を作成するように構成する必要があります。トークンが有効であり、なおかつトークンに含まれる要求とキー \(またはライセンス\) に対して構成されている要求とが一致した場合、Media Services キー配信サービスは、キー \(またはライセンス\) をクライアントに返します。

トークン制限ポリシーを構成する際は、プライマリ検証キー、発行者、対象ユーザーの各パラメーターを指定する必要があります。プライマリ検証キーには、トークンの署名に使用されたキーが含まれ、発行者は、トークンを発行するセキュリティ トークン サービスです。対象ユーザー \(スコープとも呼ばれる\) には、トークンの目的、またはトークンがアクセスを承認するリソースが記述されます。Media Services キー配信サービスでは、トークン内のこれらの値がテンプレート内の値と一致することが検証されます。

## <a id="tasks"></a>関連するタスク

### ストリーミング エンドポイントの構成

ストリーミング エンドポイントの概要とその管理方法の詳細については、「[Media Services アカウントでストリーミング エンドポイントを管理する方法](media-services-manage-origins.md)」を参照してください。

### メディアのアップロード 

**Azure の管理ポータル**、**.NET**、または **REST API** を使用して、ファイルをアップロードします。

[AZURE.INCLUDE [media-services-selector-upload-files](../includes/media-services-selector-upload-files.md)]

### アセットのエンコード

**Azure の管理ポータル**、**.NET**、または **REST API** を使用して、**Azure Media Encoder** でエンコードします。
 
[AZURE.INCLUDE [media-services-selector-encode](../includes/media-services-selector-encode.md)]

### コンテンツ キーの作成

**.NET** や **REST API** を使用してアセットを暗号化するコンテンツ キーを作成します。

[AZURE.INCLUDE [media-services-selector-create-contentkey](../includes/media-services-selector-create-contentkey.md)]

### コンテンツ キー承認ポリシーの構成 

**.NET** または **REST API** を使用してコンテンツ保護やキー承認ポリシーを構成します。

[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../includes/media-services-selector-content-key-auth-policy.md)]

### アセット配信ポリシーの構成

**.NET** または **REST API** を使用してアセット配信ポリシーを構成します。

[AZURE.INCLUDE [media-services-selector-asset-delivery-policy](../includes/media-services-selector-asset-delivery-policy.md)]


<!--HONumber=52-->