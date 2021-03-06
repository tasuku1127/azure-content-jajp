<properties
	pageTitle="承認されていないクラウド アプリケーションを Cloud App Discovery で検出する | Microsoft Azure"
	description="このトピックでは、Cloud App Discvery とは何かと、これを使用する理由について説明します。"
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	manager="swadhwa"
	editor="lisatoft"/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/16/2015"
	ms.author="markusvi"/>

# 承認されていないクラウド アプリケーションを Cloud App Discovery で検出する

## 概要
現代の企業では、IT 部門が、組織のメンバーが作業のために使用しているクラウド アプリケーションを部分的にしか認識できていないことがよくあります。管理者が企業データへの不正アクセスを心配している理由は容易に理解できます。データの漏洩やその他のセキュリティ リスクが発生するおそれがあるからです。このような認識不足は、セキュリティ リスクに対処するための計画策定が困難であるように思わせる可能性があります。

Cloud App Discovery は、Azure Active Directory (AD) Premium の機能で、組織の従業員が使用しているクラウド アプリケーションを検出することができます。

**Cloud App Discovery を使用すると次のことができます。**

- 使用されているクラウド アプリケーションを検出し、ユーザー数、トラフィック量、アプリケーションに対する Web 要求数別に使用状況を測定する
- アプリケーションを使用しているユーザーを特定する
- オフライン分析用にデータをエクスポートする
- これらのアプリケーションを IT 部門の管理下に置き、ユーザー管理のためのシングル サインオンを有効にする

## 動作のしくみ
1. アプリケーションの使用状況に対するエージェントがユーザーのコンピューターにインストールされます。
2. エージェントによってキャプチャされるアプリケーションの使用状況情報は、セキュリティで保護され、暗号化されたチャネル経由で Cloud App Discovery サービスに送信されます。
3. Cloud App Discovery サービスでは、データが評価され、レポートが生成されます。

![Cloud App Discovery の図](./media/active-directory-cloudappdiscovery/cad01.png)

Cloud App Discovery の使用を開始するには、[Cloud App Discovery の概要に関するページ](http://social.technet.microsoft.com/wiki/contents/articles/30962.getting-started-with-cloud-app-discovery.aspx)をご覧ください。

## 関連記事
- [Cloud App Discovery のセキュリティとプライバシーの考慮事項](active-directory-cloudappdiscovery-security-and-privacy-considerations.md)  
- [Cloud App Discovery のグループ ポリシーのデプロイメント ガイド](http://social.technet.microsoft.com/wiki/contents/articles/30965.cloud-app-discovery-group-policy-deployment-guide.aspx)
- [Cloud App Discovery の System Center デプロイメント ガイドに](http://social.technet.microsoft.com/wiki/contents/articles/30968.cloud-app-discovery-system-center-deployment-guide.aspx)
- [Cloud App Discovery のカスタム ポートを使用するプロキシ サービス用レジストリ設定](active-directory-cloudappdiscovery-registry-settings-for-proxy-services.md)
- [Cloud App Discovery エージェントの変更ログ](http://social.technet.microsoft.com/wiki/contents/articles/24616.cloud-app-discovery-agent-changelog.aspx)
- [Cloud App Discovery に関してよく寄せられる質問](http://social.technet.microsoft.com/wiki/contents/articles/24037.cloud-app-discovery-frequently-asked-questions.aspx)

<!---HONumber=Oct15_HO4-->