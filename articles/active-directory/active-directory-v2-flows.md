<properties
	pageTitle="アプリ モデル v2.0 のアプリの種類 | Microsoft Azure"
	description="Azure AD アプリ モデル v2.0 パブリック プレビューでサポートされているアプリの種類とシナリオ。"
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/11/2015"
	ms.author="dastrock"/>

# アプリ モデル v2.0 のプレビュー: アプリの種類
v2.0 アプリ モデルは、さまざまなモダン アプリ アーキテクチャ向けの認証をサポートしています。そのいずれも、業界標準のプロトコルである [OAuth 2.0](active-directory-v2-protocols.md#oauth2-authorization-code-flow) と [OpenID Connect](active-directory-v2-protocols.md#openid-connect-sign-in-flow) のどちらか一方または両方に基づいています。このドキュメントでは、利用する言語やプラットフォームを問わず、作成できるアプリの種類について簡単に説明します。このドキュメントは、[コードを詳しく確認する](active-directory-appmodel-v2-overview.md#getting-started)前に、大まかなシナリオを理解するのに役立ちます。

> [AZURE.NOTE]この情報は、v2.0 アプリ モデルのパブリック プレビューに関するものです。一般公開されている Azure AD サービスとの統合手順については、「[Azure Active Directory 開発者ガイド](active-directory-developers-guide.md)」を参照してください。

## 基本操作
v2.0 アプリ モデルを使うアプリは、すべて [apps.dev.microsoft.com](https://apps.dev.microsoft.com) で登録する必要があります。アプリの登録プロセスでは、いくつかの値が収集され、対象のアプリに割り当てられます。

- アプリを一意に識別する**アプリケーション ID**。
- 応答をアプリにリダイレクトして戻すために使用できる**リダイレクト URI**。
- その他シナリオに応じた値。詳細については、[アプリの登録](active-directory-v2-app-registration.md)方法を参照してください。

登録済みのアプリは、v2.0 エンドポイントに要求を送ることによって、Azure AD と通信を行います。

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
https://login.microsoftonline.com/common/oauth2/v2.0/token
```

アプリによる v2.0 エンドポイントとのやり取りは、いずれも次のような大まかなパターンに従って行われます。

1. サインインのために、アプリがエンド ユーザーを v2.0 エンドポイントにリダイレクトします。
2. ユーザーがユーザー名とパスワードを入力するなどして、認証を行います。
3. ユーザーがアプリから要求されたアクセス許可を付与し、アプリがユーザーの代理で処理することを承認します。
4. アプリが v2.0 エンドポイントから何らかの種類のセキュリティ トークンを受け取ります。
5. アプリがセキュリティ トークンを使って、保護された情報 (リソース) にアクセスします。
6. リソース サーバーがセキュリティ トークンを検証し、アクセスを許可できることを確かめます。
7. アプリがセキュリティ トークンを定期的に更新します。

<!-- TODO: Need a page for libraries to link to -->
これら 7 つの手順は、作成しているアプリの種類に応じて若干異なりますが、細かい処理を代わりに実行してくれるオープン ソースのライブラリが用意されています。[アクセス許可、同意、スコープ](active-directory-v2-scopes.md)について調べるか、このまま読み進んで具体例を確認してください。

## Web Apps
ブラウザーからアクセスされる Web アプリ (.NET、PHP、Java、Ruby、Python、Node など) の場合、v2.0 アプリ モデルは [OpenID Connect](active-directory-v2-protocols.md#openid-connect-sign-in-flow) によりユーザーのサインインをサポートします。OpenID Connect では、Web アプリは `id_token` を受け取ります。これは、ユーザーの ID を検証し、要求の形でユーザーに関する情報を提供するセキュリティ トークンです。

```
// Partial raw id_token
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImtyaU1QZG1Cd...

// Partial content of a decoded id_token
{
	"name": "John Smith",
	"email": "john.smith@gmail.com",
	"oid": "d9674823-dffc-4e3f-a6eb-62fe4bd48a58"
	...
}
```

アプリから利用できるすべての種類のトークンと要求については、[v2.0 トークン リファレンス](active-directory-v2-tokens.md)を参照してください。

Web サーバー アプリにおけるサインイン認証フローは、主に次のステップで構成されます。

![Web App Swimlanes Image](../media/active-directory-v2-flows/convergence_scenarios_webapp.png)

v2.0 エンドポイントから受け取った公開署名キーを使用した id\_token の検証は、ユーザーの ID を確認し、後続のページ要求でユーザーを識別するために使用できるセッション Cookie を設定するうえで十分です。

このシナリオを実際に確認するには、「[使用の開始](active-directory-appmodel-v2-overview.md#getting-started)」セクションのいずれかの Web アプリ サインイン コード サンプルを試してください。

Web サーバー アプリは、単純なサインインを実行するだけでなく、REST API をはじめとする他の Web サービスにアクセスすることが必要な場合もあります。この場合、Web サーバー アプリは、OpenID Connect と OAuth 2.0 を組み合わせたフローに関与できます。その際使用されるのが[OAuth 2.0 承認コード フロー](active-directory-v2-protocols.md#oauth2-authorization-code-flow)です。このシナリオは、以下の [Web API のセクション](#web-apis)と、[WebApp-WebAPI の使用の開始に関するトピック](active-directory-v2-devquickstarts-webapp-webapi-dotnet.md)で取り上げています。

## Web API
v2.0 アプリ モデルを使用すると、アプリの RESTful Web API など、Web サービスもセキュリティで保護できます。Web API では、id\_token とセッション Cookie の代わりに OAuth 2.0 access\_token を使ってデータをセキュリティ保護し、受信要求を認証します。Web API の呼び出し元は、HTTP 要求の承認ヘッダーの末尾に access\_token を追加します。

```
GET /api/items HTTP/1.1
Host: www.mywebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6...
Accept: application/json
...
```

Web API はその access\_token を使って API の呼び出し元の ID を検証し、access\_token でエンコードされている要求から呼び出し元に関する情報を抽出します。アプリから利用できるすべての種類のトークンと要求については、[v2.0 トークン リファレンス](active-directory-v2-tokens.md)を参照してください。

Web API は、アクセス許可 ([スコープ](active-directory-v2-scopes.md)とも呼ばれる) を公開することで、特定の機能またはデータをオプトイン/オプトアウトできるようにします。呼び出し元のアプリがスコープに対するアクセス許可を取得するには、ユーザーがフローの途中でスコープに同意する必要があります。v2.0 エンドポイントはユーザーにアクセス許可を求め、Web API が受け取るすべての access\_token にこれらのアクセス許可を記録します。Web API で必要なことは、各呼び出しで受け取った access\_token を検証することと、適切な承認確認を行うことだけです。

Web API は、すべての種類のアプリ (Web サーバー アプリ、デスクトップ アプリとモバイル アプリ、シングル ページ アプリ、サーバー側のデーモン、さらにそれ以外の Web API など) から access\_token を受信できます。例として、Web API を呼び出す Web サーバー アプリのフロー全体を見てみましょう。

![Web API のスイムレーン イメージ](../media/active-directory-v2-flows/convergence_scenarios_webapp_webapi.png)

authorization\_code と refresh\_token についてのほか、access\_tokens の詳しい取得手順については、[OAuth 2.0 プロトコルの説明](active-directory-v2-protocols.md#oauth2-authorization-code-flow)を参照してください。

v2.0 アプリ モデルと OAuth 2.0 access\_token で Web API をセキュリティ保護する方法については、[使用の開始セクション](active-directory-appmodel-v2-overview.md#getting-started)で Web API コード サンプルを確認してください。


## モバイル アプリとネイティブ アプリ
多くの場合、モバイル アプリやデスクトップ アプリなど、デバイスにインストールされているアプリは、データを格納し、ユーザーの代わりにさまざまな処理を実行するバックエンド サービスや Web API にアクセスする必要があります。これらのアプリは、v2.0 モデルと [OAuth 2.0 承認コード フロー](active-directory-v2-protocols.md#oauth2-authorization-code-flow)を使ってバックエンド サービスにサインインと承認を追加します。

このフローでは、アプリはユーザーのサインインの際に v2.0 エンドポイントから authorization\_code を受け取ります。これは、現在サインインしているユーザーの代わりにバックエンド サービスを呼び出すアプリのアクセス許可を表します。次に、アプリはバックグラウンドで authoriztion\_code を OAuth 2.0 access\_token および refresh\_token と交換します。アプリはその access\_token を使って HTTP 要求で Web API に対して認証を実行します。また、access\_token の有効期限が切れた場合は、refresh\_token を使って新しい access\_token を取得します。

![ネイティブ アプリのスイムレーン イメージ](../media/active-directory-v2-flows/convergence_scenarios_native.png)

## 現在のプレビューの制限事項
これらの種類のアプリは、現時点では v2.0 アプリ モデルのプレビューでサポートされていませんが、一般公開に間に合うように準備中です。v2.0 アプリ モデル パブリック プレビューのその他の制限事項については、[v2.0 プレビューの制限事項に関する記事](active-directory-v2-limitations.md)を参照してください。

### シングル ページ アプリ (JavaScript)
最近多く見かけるようになったシングル ページ アプリのフロントエンドは、主に JavaScript で作成されています。AngularJS、Ember.js、Durandal などの SPA フレームワークが使われることも少なくありません。一般公開される Azure AD サービスでは、これらのアプリが [OAuth 2.0 Implicit Flow](active-directory-v2-protocols.md#oauth2-implicit-flow) を使ってサポートされます。しかし v2.0 アプリ モデルでは、このフローがまだ利用できません。サポートされるまでには、あと少し時間がかかります。

今すぐ v2.0 アプリ モデルで SPA を動作させる必要がある場合は、上記の[Web サーバー アプリ フロー](#web-apps)を使用して認証を実装してください。ただしこの方法は推奨されません。そのような使い方に関するドキュメントも限られています。SPA の使い方について感触をつかむことが目的であれば、[一般提供版 Azure AD の SPA コード サンプル](active-directory-devquickstarts-angular.md)を参照してください。

### デーモン/サーバー サイド アプリ
長時間実行されるプロセスを含んだアプリや、ユーザーの介入なしで動作するアプリも、セキュリティで保護されたリソース (Web API など) にアクセスする必要があります。これらのアプリは、[OAuth 2.0 クライアント資格情報フロー](active-directory-v2-protocols.md#oauth2-client-credentials-grant-flow)を使用することで、(ユーザーの委任 ID ではなく) アプリの ID を使って認証を行い、トークンを取得することができます。

v2.0 アプリ モデルでは、このフローが現在サポートされていません。つまりアプリがトークンを取得できるのは、対話的なユーザー サインイン フローの後に限られます。近い将来、クライアント資格情報フローが追加されます。一般提供版 Azure AD サービスでのクライアント資格情報フローについては、[GitHub のデーモン サンプル](https://github.com/AzureADSamples/Daemon-DotNet)を参照してください。

### Web API の連鎖 (On-Behalf-Of)
v2.0 アプリ モデルによって保護された Web API から、同じように保護された別の下流の Web API を呼び出す手法は、多くのアーキテクチャで使われています。このシナリオは、バックエンドの Web API から Microsoft Online サービス (Office 365、Graph API など) を呼び出すネイティブ クライアントでよく見られます。

このように Web API を連鎖的に呼び出すシナリオは、OAuth 2.0 Jwt Bearer Credential Grant ([On-Behalf-Of フロー](active-directory-v2-protocols.md#oauth2-on-behalf-of-flow)) を使用してサポートできます。しかし現時点では v2.0 アプリ モデル プレビューに On-Behalf-Of フローが実装されていません。一般提供版 Azure AD サービスにおけるこのフローの動作については、[GitHub の On-Behalf-Of コード サンプル](https://github.com/AzureADSamples/WebAPI-OnBehalfOf-DotNet)を参照してください。

<!---HONumber=Oct15_HO3-->