<properties 
	pageTitle="Azure Multi-Factor Authentication Server の概要" 
	description="Azure MFA Server を開始する方法について説明する Azure Multi-Factor Authentication のページです。" 
	services="multi-factor-authentication" 
	documentationCenter="" 
	authors="billmath" 
	manager="terrylan" 
	editor="bryanla"/>

<tags 
	ms.service="multi-factor-authentication" 
	ms.workload="identity" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/02/2015" 
	ms.author="billmath"/>

# Azure Multi-Factor Authentication Server の概要




<center>![Cloud](./media/multi-factor-authentication-get-started-server/server2.png)</center>

オンプレミスの多要素認証を使用するかどうかを判断しました。次に進みましょう。このページでは、新しいサーバーのインストールや、オンプレミスの Active Directory を使用したそのサーバーの設定について説明します。Phonefactor サーバーが既にインストールされており、Azure Multi-Factor Server へのアップグレード方法を探している場合は「Azure Multi-Factor Server へのアップグレード」、Web サービスのみのインストールに関する情報を探している場合は「Azure Multi-Factor Authentication Server モバイル アプリ Web サービスの展開」を参照してください。



## Azure Multi-Factor Authentication Server のダウンロード

Azure Multi-Factor Authentication Server は 2 つの方法でダウンロードできます。1 番目の方法は Azure ポータルにサインインする方法で、2 番目の方法は [https://pfweb.phonefactor.net](https://pfweb.phonefactor.net) から直接ダウンロードする方法です。


### Azure ポータルから Azure Multi-Factor Authentication Server をダウンロードするには
--------------------------------------------------------------------------------

1. Azure ポータルに管理者としてサインインします。
2. 左側で、[Active Directory] を選択します。
3. [Active Directory] ページの上部で **[多要素認証プロバイダー]** をクリックします。
4. 下部で **[管理]** をクリックします。
5. **[ダウンロード]** をクリックします。
6. **[アクティブ化資格情報の生成]** の上にある **[ダウンロード]** をクリックします。
7. ダウンロードしたファイルを保存します。

### Azure Multi-Factor Authentication Server から直接ダウンロードするには
--------------------------------------------------------------------------------

1. [https://pfweb.phonefactor.net](https://pfweb.phonefactor.net) にサインインします。
2. **[ダウンロード]** をクリックします。
<center>![Cloud](./media/multi-factor-authentication-get-started-server/download2.png)</center>
3. **[アクティブ化資格情報の生成]** の上にある **[ダウンロード]** をクリックします。後ほどこのページに戻ってくるため、そのままにしておきます。
4. ダウンロードしたファイルを保存します。



## Azure Multi-Factor Authentication Server のインストールと構成
サーバーをダウンロードできたので、インストールおよび構成を行うことができます。インストールしているサーバーが次の要件を満たすようにしてください。



Azure Multi-Factor Authentication Server の要件|説明|
:------------- | :------------- | 
ハードウェア|<li>ハード ディスク空き容量 200 MB</li><li>x32 または x64 対応のプロセッサ</li><li>RAM 1 GB 以上</li>
ソフトウェア|<li>ホストがサーバー OS の場合は Windows Server 2003 以降</li><li>ホストがクライアント OS の場合は Windows Vista 以降</li><li>Microsoft .NET 2.0 Framework</li><li>ユーザー ポータルまたは Web サービス SDK をインストールしている場合は IIS 6.0 以降</li>

### Azure Multi-Factor Authentication Server ファイアウォールの要件
--------------------------------------------------------------------------------
各 MFA Server は、次へ送信するポート 443 で通信できる必要があります。

- https://pfd.phonefactor.net
- https://pfd2.phonefactor.net
- https://css.phonefactor.net

送信ファイアウォールがポート 443 で制限されている場合、次の IP アドレスの範囲を開く必要があります。

IP サブネット|ネットマスク|IP 範囲
:------------- | :------------- | :------------- |
134.170.116.0/25|255.255.255.128|134.170.116.1 – 134.170.116.126
134.170.165.0/25|255.255.255.128|134.170.165.1 – 134.170.165.126
70.37.154.128/25|255.255.255.128|70.37.154.129 – 70.37.154.254

Azure Multi-Factor Authentication Event Confirmation 機能を使用しておらず、ユーザーが企業ネットワーク上にあるデバイスから Multi-Factor Auth モバイル アプリによる認証を受けていない場合は、IP の範囲が以下のとおりに狭くなる可能性があります。


IP サブネット|ネットマスク|IP 範囲
:------------- | :------------- | :------------- |
134.170.116.72/29|255.255.255.248|134.170.116.72 – 134.170.116.79
134.170.165.72/29|255.255.255.248|134.170.165.72 – 134.170.165.79
70.37.154.200/29|255.255.255.248|70.37.154.201 – 70.37.154.206


### Azure Multi-Factor Authentication Server をインストールおよび構成するには
--------------------------------------------------------------------------------


1. 実行可能ファイルをダブルクリックします。これによりインストールが開始されます。
2. [インストール フォルダーの選択] 画面で、フォルダーが正しいかどうかを確認し、[次へ] をクリックします。
3. インストールが完了したら、[終了] をクリックします。これによって、構成ウィザードが起動します。
4. 構成ウィザードのようこそ画面で、**[認証構成ウィザードの使用をスキップする]** をオンにし、**[次へ]** をクリックします。これによりウィザードが終了し、サーバーが起動します。
<center>![Cloud](./media/multi-factor-authentication-get-started-server/skip2.png)</center>

5. サーバーをダウンロードしたページに戻るには、**[アクティブ化資格情報の生成]** ボタンをクリックします。この情報を提供されたボックスの Azure MFA Server にコピーし、**[アクティブ化]** をクリックします。


上記の手順は構成ウィザードを使用した簡単セットアプリです。認証ウィザードを再実行するには、サーバーの [ツール] メニューからウィザードを選択します。



##Active Directory からのユーザーのインポート

サーバーのインストールおよび構成を行ったため、ユーザーを Azure MFA Server に簡単にインポートできます。

### Active Directory からユーザーをインポートするには
--------------------------------------------------------------------------------


1. Azure MFA Server の左側で **[ユーザー]** を選択します。
2. 下部で **[Active Directory からインポート]** を選択します。
3. 個々のユーザーを検索したり、AD ディレクトリでユーザーを含む OU を検索したりできるようになります。ここではユーザー OU を指定します。
4. 右側のすべてのユーザーを強調表示し、**[インポート]** をクリックします。成功したことを通知するポップアップが表示されます。インポート ウィンドウを閉じます。

<center>![Cloud](./media/multi-factor-authentication-get-started-server/import2.png)</center>

<!---HONumber=July15_HO2-->