<properties 
	pageTitle="ポータルを使用してコンテンツ キー承認ポリシーを構成します。" 
	description="コンテンツ キー承認ポリシーを構成する方法について説明します。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/20/2015" 
	ms.author="juliako"/>



# コンテンツ キー承認ポリシーを構成する 
[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../../includes/media-services-selector-content-key-auth-policy.md)]

この記事は、「[Media Services ビデオ オン デマンド ワークフロー](media-services-video-on-demand-workflow.md)」や「[Media Services のライブ ストリーミングのワークフロー](media-services-live-streaming-workflow.md)」シリーズの一部です。 


## 概要

Microsoft Azure Media Services では、高度暗号化標準 (AES) (128 ビット暗号化キーを使用) と PlayReady DRM を使用して動的に暗号化したコンテンツを配信できます。Media Services は、クライアントがキーまたはライセンスを取得して暗号化されたコンテンツを再生する **Key\License 配信サービス**も提供しています。 

このトピックでは、**Azure 管理ポータル**を使用して、コンテンツ キー承認ポリシーを構成する方法について説明します。キーは、動的にコンテンツを暗号化するために、後で使用できます。現時点では、ストリーミング形式、HLS、MPEG DASH、スムーズ ストリーミングを暗号化できます。HDS 形式のストリーミングやプログレッシブ ダウンロードは暗号化できません。
 
プレーヤーが動的に暗号化するように設定されたストリームを要求すると、Media Services は、AES または PlayReady 暗号化で構成済みのキーを使用してコンテンツを暗号化します。ストリームの暗号化を解除するには、プレーヤーはキー配信サービスからキーを要求します。ユーザーのキーの取得が承認されているかどうかを判断するために、サービスはキーに指定した承認ポリシーを評価します。


複数のコンテンツ キー持つ、または  Media Services のキー配信サービス以外の **Key\License 配信サービス** URL を指定する場合は、Media Services .NET SDK または REST APIs を使用します。

[Media Services .NET SDK を使用したコンテンツ キー承認ポリシーの構成](media-services-dotnet-configure-content-key-auth-policy.md)

[Media Services REST API を使用したコンテンツ キー承認ポリシーの構成](media-services-rest-configure-content-key-auth-policy.md)

### いくつかの考慮事項が適用されます。

- 動的パッケージングや動的暗号化を使用するには、少なくとも 1 つのストリーミング占有ユニットが必要です。詳細については、「[Media Services の規模の設定方法](media-services-manage-origins.md#scale_streaming_endpoints)」をご覧ください。 
- アセットには、一連のアダプティブ ビットレート MP4 または アダプティブ ビットレート スムーズ ストリーミング ファイルが含まれている必要があります。詳細については、[アセットをエンコードする方法](media-services-encode-asset.md)に関するページをご覧ください。  
- キー配信サービスでは、ContentKeyAuthorizationPolicy とそれに関連するオブジェクト (ポリシーのオプションと制限) を 15 分間キャッシュします。ContentKeyAuthorizationPolicy を作成して、"Token" 制限を使用するように指定した場合に、"Token" 制限をテストしてから、ポリシーを "Open" 制限に更新すると、ポリシーが "Open" バージョンのポリシーに切り替わるまで、約 15 分かかります。


## 方法: キーの承認ポリシーを構成する

キーの承認ポリシーを構成するには、**コンテンツ保護**ページを選択します。
	
Media Services では、キーを要求するユーザーを承認する複数の方法がサポートされています。コンテンツ キー承認ポリシーには、**開く**、**トークン**、または**IP** 承認制限を持つことができます (**IP** は REST または .NET SDK で構成できます)。 

### オープン制限

**開く**制限は、システムがキーを要求するすべてのユーザーにキーを配信します。この制限は、テストに便利です。

![OpenPolicy][open_policy]

### トークン制限

トークンの制限ポリシーを選択するには、**[トークン]** ボタンを押します。

**トークン**制限ポリシーには、**STS (セキュリティ トークン サービス)** によって発行されたトークンを含める必要があります。Media Services では、**Simple Web Tokens** ([SWT](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_2)) 形式と **JSON Web Token** (JWT) 形式のトークンがサポートされます。詳細については、「[JWT token authentication (JWT トークン認証)](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)」をご覧ください。

Media Services では、**Secure Token Services** は提供されません。トークンを発行するには、カスタム STS を作成するか、Microsoft Azure ACS を活用できます。STS は、トークン、トークンの制限の構成で指定した、指定されたキーと問題要求で署名を作成するように構成する必要があります。Media Services のキー配信サービスは、トークンが有効で、トークン内のクレームがコンテンツ キー向けに構成されたクレームと一致する場合、暗号化キーをクライアントに返します。詳細については、「[Use Azure ACS to issue tokens (Azure ACS を使用してトークンを発行する)http://mingfeiy.com/acs-with-key-services)」をご覧ください。

**トークン**制限ポリシーを構成する場合は、**検証キー**、**発行者**と**対象ユーザー**の値を設定する必要があります。プライマリ検証キーには、トークンの署名に使用されたキーが含まれ、発行者は、トークンを発行するセキュリティ トークン サービスです。対象ユーザー (スコープとも呼ばれる) には、トークンの目的、またはトークンがアクセスを承認するリソースが記述されます。Media Services  キー配信サービスでは、トークン内のこれらの値がテンプレート内の値と一致することが検証されます。  

### PlayReady

**PlayReady** を使用してコンテンツを保護するときは、PlayReady ライセンス テンプレートを定義する XML 文字列を認証ポリシーで指定する必要があります。既定では、次のポリシーが設定されます。
		
	<PlayReadyLicenseResponseTemplate xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1">
	  <LicenseTemplates>
	    <PlayReadyLicenseTemplate><AllowTestDevices>true</AllowTestDevices>
	      <ContentKey i:type="ContentEncryptionKeyFromHeader" />
	      <LicenseType>Nonpersistent</LicenseType>
	      <PlayRight>
	        <AllowPassingVideoContentToUnknownOutput>Allowed</AllowPassingVideoContentToUnknownOutput>
	      </PlayRight>
	    </PlayReadyLicenseTemplate>
	  </LicenseTemplates>
	</PlayReadyLicenseResponseTemplate>

**XML ポリシーのインポート** ボタンをクリックして、[ここ](https://msdn.microsoft.com/library/azure/dn783459.aspx)で定義された XML スキーマ に準拠した別の XML を提供します。

## 次のステップ
これで、コンテンツ キー承認ポリシーが構成されました。[方法:Azure 管理ポータルを使用して暗号化を有効にする](media-services-manage-content#encrypt.md)トピックに移動します。


[open_policy]: ./media/media-services-portal-configure-content-key-auth-policy/media-services-protect-content-with-open-restriction.png
[token_policy]: ./media/media-services-key-authorization-policy/media-services-protect-content-with-token-restriction.png



<!--HONumber=52--> 