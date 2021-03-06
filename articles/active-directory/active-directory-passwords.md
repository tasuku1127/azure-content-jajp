<properties 
	pageTitle="Azure AD パスワード管理とは | Microsoft Azure"
	description="パスワードのリセット、変更、パスワード管理のレポート、ローカルのオンプレミスの Active Directory へのライトバックなど、Azure AD のパスワード管理機能の説明。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory" 
	ms.workload="identity" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="10/08/2015" 
	ms.author="asteen"/>

# 任意の場所からのパスワードの管理

  >[AZURE.IMPORTANT]Azure または O365 のパスワードをリセットする場合は、 [こちらに記載されている指示に従ってください](https://support.microsoft.com/kb/2606983)。
  
セルフ サービスの実現は、コストを削減し労力を節約する手段として、世界中の IT 部門が長年目指してきた大きな目標です。実際、市場はクラウドまたはオンプレミスから、オンプレミスのグループや、パスワード、ユーザー プロファイルを管理できるようにする製品であふれています。Azure AD は、現在利用可能な機能の中でも、最も使いやすく強力なセルフ サービス機能を提供することで、これらの製品とは一線を画しています。

**Azure AD パスワード管理**は、定義するセキュリティ ポリシーに準拠したままで、ユーザーが任意のデバイスで、時間と場所に関係なくパスワードを管理できる一連の機能です。

## 概要
5 分もかからずに、Azure AD パスワード管理を試験的に使用できます。また、数時間で組織全体にデプロイできます。次に、このサービスを使用するのに役立つリソースをいくつか示します。

* [**しくみ**](active-directory-passwords-how-it-works.md) - サービスの 6 つの異なるコンポーネントとそれぞれの機能について説明します。
* [**概要**](active-directory-passwords-getting-started.md) -ユーザーによるクラウドまたはオンプレミスのパスワードのリセットと変更を許可する方法について説明します。
* [**カスタマイズ**](active-directory-passwords-customize.md) - 組織のニーズに合わせてサービスの外観と動作をカスタマイズする方法について説明します。
* [**ベスト プラクティス**](active-directory-passwords-best-practices.md) - 組織内でのパスワードの迅速なデプロイと効果的な管理方法について説明します。
* [**洞察を得る**](active-directory-passwords-get-insights.md) - 統合レポート機能について説明します。
* [**FAQ**](active-directory-passwords-faq.md) -よく寄せられる質問の回答を得ます。
* [**トラブルシューティング**](active-directory-passwords-troubleshoot.md) - サービスに関する問題を迅速にトラブルシューティングする方法について説明します。
* [**詳細情報**](active-directory-passwords-learn-more.md) - サービスの機能の技術的な詳細を掘り下げます。


## Azure AD パスワード管理の機能
ここでは、Azure AD のパスワード管理機能を使用して実行できることをいくつか示します。

- **セルフ サービスによるパスワードの変更**では、エンド ユーザーまたは管理者は、管理者またはヘルプ デスクにサポートを依頼することなく、有効期限が切れた、または有効期限が切れていないパスワードを変更できます。
- **セルフ サービスによるパスワードのリセット**では、エンド ユーザーまたは管理者は、管理者またはヘルプ デスクにサポートを依頼することなく、パスワードを自動的にリセットできます。セルフ サービスのパスワード リセットには、Azure AD Premium または Basic が必要です。詳細については、「Azure Active Directory のエディション」をご覧ください。
- **管理者によるパスワードのリセット**では、管理者は、[Microsoft Azure 管理ポータル](https://manage.windowsazure.com)内からエンド ユーザーまたは別の管理者のパスワードをリセットできます。
- **パスワード管理アクティビティ レポート**では、組織内で発生したパスワードのリセットおよび登録アクティビティの詳細が管理者に提供されます。 
- **パスワード ライトバック**では、クラウドからオンプレミスのパスワードを管理できるので、フェデレーション ユーザーまたはパスワード同期済みユーザーは、またはこれらのユーザーに代わって、上記のシナリオをすべて実行できます。パスワード ライトバックには Azure AD Premium が必要です。詳細については、「Azure AD Premium の概要」を参照してください。

## Azure AD パスワード管理を使用する理由
ここでは、Azure AD のパスワード管理機能を使用すべき理由をいくつか示します。

- **コストを削減する** -サポート部門によるパスワードのリセットは、通常、組織の IT 支出の 20% を占めます。
- **ユーザー エクスペリエンスを向上させる** -ユーザーは、パスワードを忘れるたびにヘルプデスクに電話して、時間を費やしたくはありません。
- **ヘルプデスクの負荷を低減する** -パスワード管理は、ほとんどの組織のヘルプデスクにおいて、それひとつに最も負担のかかる仕事です。
- **モビリティを高める** -ユーザーが任意の場所からパスワードをリセットできます。

## 最新のサービス更新内容

**登録ページでの使いやすさの更新** - 2015 年 10 月

- ユーザーは既に登録されたデータがある場合、電子メールを送信したり電話をかけたりしなくても、[問題なし] をクリックするだけでデータを更新できるようになりました。

**パスワード ライトバックの信頼性が向上** - 2015 年 9 月

- Azure AD Connect の 9 月リリースの時点で、パスワード ライトバック エージェントは接続と、より堅牢な追加のフェールオーバー機能とを積極的に再試行するようになります。

**パスワード リセット レポート データを取得するための API** - 2015 年 8 月

- パスワード リセット レポートの背後にあるデータを [Azure AD レポートおよびイベント API](https://msdn.microsoft.com/library/azure/mt126081.aspx#BKMK_SsprActivityEvent) で直接取得できるようになりました。

**クラウド ドメインへの参加中に Azure AD のパスワード リセットをサポート** - 2015 年 8 月

- クラウド ユーザーは、クラウド ドメインへの参加のためのオンボード作業中に Windows 10 サインイン画面から自身のパスワードをリセットできるようになりました。この機能は Windows 10 のサインイン画面ではまだ公開されていません。

**Azure およびフェデレーション アプリへのサインイン時にパスワード リセット登録を強制** - 2015 年 7 月

- myapps.microsoft.com へのサインイン時の登録の実施に加えて、Microsoft Azure 管理ポータルおよび任意のフェデレーション シングル サインオン アプリケーションへのサインイン時の登録の実施もサポートするようになりました。

**セキュリティに関する質問のローカライズ サポート** - 2015 年 5 月

- パスワード リセットに対してセキュリティの質問を構成する場合に、完全な O365 言語セットでローカライズされた定義済みのセキュリティの質問を選択できるようになりました。

**パスワード リセット中のアカウントのロック解除をサポート** - 2015 年 6 月

- パスワード ライトバックを使用している場合、アカウントがロックされている状態でパスワードをリセットすると、Active Directory アカウントが自動的にロック解除されます。

**SSPR 登録のブランド化** - 2015 年 4 月

- パスワードのリセットの登録ページは、会社のロゴでブランド化されるようになりました。

**セキュリティの質問** - 2015 年 3 月

- セキュリティの質問を、一般に公開しました。

**アカウントのロックの解除** - 2015 年 3 月

- パスワードのリセットが発生した場合に、ユーザーがロックを解除できるようになりました。

## 近日対応予定

現在作業中の優れた機能の一部を次に示します。

**パスワードをリセットすることなく Active Directory アカウントのロックを解除する機能をサポート** - 近日公開予定

- 多くのユーザーから、パスワード リセットとは切り離して AD アカウントのロックを解除する機能が要求されました。この機能については最後の仕上げを行っている最中であります。間もなく、パスワード ライトバックを使用しているすべての方にリリースされます。

**サインイン時に登録済みデータを更新するようユーザーに喚起する機能** - 開発中

- 現在、myapps.microsoft.com へのアクセス時に登録済みデータを更新するようユーザーに喚起する機能をサポートしていますが、すべてのサインインでこれを行う機能を開発中です。

**Office 365 アプリへのサインイン時にパスワード リセット登録を実施する** - 開発中

- ますます多くの会社アプリが最新かつ最高の Azure AD サインイン エクスペリエンスに参加しつつあります。その場合、会社アプリは SSPR で実施される登録に対して自動的にサポートされます。

<br/> <br/> <br/>

**その他のリソース**


* [パスワード管理のしくみ](active-directory-passwords-how-it-works.md)
* [パスワード管理の概要](active-directory-passwords-getting-started.md)
* [パスワード管理のカスタマイズ](active-directory-passwords-customize.md)
* [パスワード管理のベスト プラクティス](active-directory-passwords-best-practices.md)
* [パスワード管理レポートで運用情報を把握する方法](active-directory-passwords-get-insights.md)
* [パスワード管理に関する FAQ](active-directory-passwords-faq.md)
* [パスワード管理のトラブルシューティング](active-directory-passwords-troubleshoot.md)
* [詳細情報](active-directory-passwords-learn-more.md)
* [MSDN のパスワード管理](https://msdn.microsoft.com/library/azure/dn510386.aspx)

<!---HONumber=Nov15_HO1-->