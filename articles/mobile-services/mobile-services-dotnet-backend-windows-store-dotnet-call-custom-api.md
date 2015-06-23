<properties 
	pageTitle="Windows ストア クライアントからのカスタム API の呼び出し - Mobile Services" 
	description="カスタム API を定義し、Azure Mobile Services を使用する Windows ストア アプリからそれを呼び出す方法について説明します。" 
	services="mobile-services" 
	documentationCenter="windows" 
	authors="ggailey777" 
	writer="glenga" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="" 
	ms.devlang="dotnet" 
	ms.topic="article" 
	ms.date="02/26/2015" 
	ms.author="glenga"/>

# クライアントからのカスタム API 呼び出し

[AZURE.INCLUDE [mobile-services-selector-call-custom-api](../../includes/mobile-services-selector-call-custom-api.md)]

このトピックでは、Windows ストア アプリからカスタム API を呼び出す方法について説明します。カスタム API を使用してカスタム エンドポイントを定義することにより、insert、update、delete、read のいずれの操作にも関連しないサーバー機能を公開することができます。カスタム API を使用することによって、HTTP メッセージ ヘッダーの読み取りや設定、JSON 以外のメッセージ本文形式の定義など、メッセージングをより柔軟に制御することができます。

このトピックで作成したカスタム API では、テーブル内のすべての Todo 項目の完了フラグを `true` に設定する単一の POST 要求を送信できます。このカスタム API を使用せずにこれを行う場合、クライアントは、テーブル内の各 Todo 項目について個別にフラグの更新要求を送信する必要があります。

チュートリアル「[モバイル サービスの使用]」または「[データの使用]」の最後に作成したアプリケーションが、この機能の追加対象となります。次の手順を実行します。

1. [カスタム API を定義する]
2. [カスタム API を呼び出すようにアプリケーションを更新する]
3. [アプリケーションをテストする] 

このチュートリアルは、モバイル サービスのクイック スタートに基づいています。このチュートリアルを開始する前に、「[モバイル サービスの使用]」または「[データの使用]」を完了している必要があります。

[AZURE.INCLUDE [mobile-services-dotnet-backend-create-custom-api](../../includes/mobile-services-dotnet-backend-create-custom-api.md)]

## <a name="update-app"></a>カスタム API を呼び出すようにアプリケーションを更新する

[AZURE.INCLUDE [mobile-services-windows-store-dotnet-call-custom-api](mobile-services-windows-store-dotnet-call-custom-api.md)]


## 次のステップ

以上、カスタム API を作成し、Windows ストア アプリから呼び出す方法について説明しました。モバイル サービスに関連した次のトピックも参考にしてください。

* [定期的な通知をサポートするカスタム API の定義] <br/>Windows ストア アプリでカスタム API を使用し、定期的な通知をサポートする方法について説明します。定期的な通知が有効になっている場合、Windows が定期的にカスタム API のエンドポイントにアクセスし、返された XML (タイル固有の形式) を使用して、スタート メニューのアプリケーションのタイルを更新します。

* [Mobile Services のサーバー スクリプト リファレンス] <br/>カスタム API の作成について説明します。

* [ソース管理へのサーバー スクリプトの保存] <br/> ソース管理機能を使用して、カスタム API スクリプト コードを簡単かつ安全に開発、発行する方法について説明します。

<!-- Anchors. -->
[カスタム API を定義する]: #define-custom-api
[カスタム API を呼び出すようにアプリケーションを更新する]: #update-app
[アプリケーションをテストする]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->
[Mobile Services のサーバー スクリプト リファレンス]: http://go.microsoft.com/fwlink/?LinkId=262293
[モバイル サービスの使用]: ../mobile-services-windows-store-get-started.md
[データの使用]: ../mobile-services-dotnet-backend-windows-store-dotnet-get-started-data.md
[Get started with authentication]: ../mobile-services-dotnet-backend-windows-store-dotnet-get-started-users.md
[Get started with push notifications]: ../mobile-services-dotnet-backend-windows-store-dotnet-get-started-push.md

[定期的な通知をサポートするカスタム API の定義]: mobile-services-windows-store-dotnet-create-pull-notifications.md
[ソース管理へのサーバー スクリプトの保存]: mobile-services-store-scripts-source-control.md

<!--HONumber=54--> 