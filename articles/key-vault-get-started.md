<properties 
	pageTitle="Azure Key Vault の概要 | 概要" 
	description="このチュートリアルを使用すると、Azure Key Vault で、強化されたコンテナーを Azure に作成し、暗号化キーやシークレットを Azure に格納して管理できるようになります。" 
	services="key-vault" 
	documentationCenter="" 
	authors="cabailey" 
	manager="mbaldwin"/>

<tags 
	ms.service="key-vault" 
	ms.workload="identity" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="05/04/2015" 
	ms.author="cabailey"/>

# Azure Key Vault の概要 #

## はじめに  
このチュートリアルを使用すると、Azure Key Vault \(現時点ではプレビュー\) で、強化されたコンテナー \(資格情報コンテナー\) を Azure に作成し、暗号化キーやシークレットを Azure に格納して管理できるようになります。ここでは、Windows PowerShell を使用して、Azure アプリケーションで使用できるキーまたはパスワードを含む資格情報コンテナーを作成するプロセスについて説明します。アプリケーションがそのキーやパスワードを使用する方法についても説明します。

**推定所要時間:** 20 分

>[AZURE.NOTE]このチュートリアルでは、いずれかの手順に含まれる Azure アプリケーションの記述方法を説明していません。Key Vault でキーやシークレットを使用するためのアプリケーションの承認方法について説明しています。
>
>プレビュー期間中は、Azure ポータルで Azure Key Vault を構成できません。代わりに、Azure PowerShell 命令を使用します。

Azure Key Vault の概要については、「[What is Azure Key Vault? \(Azure Key Vault とは\)](key-vault-whatis.md)」をご覧ください。

## 前提条件 

このチュートリアルを完了するには、以下が必要です。

- Microsoft Azure サブスクリプション。サブスクリプションがない場合でも、[無料評価版](../../../pricing/free-trial)にサインアップできます。
- Azure PowerShell バージョン 0.9.1 以降。最新バージョンをインストールして、Azure サブスクリプションに関連付けるには、「[How to install and configure Azure PowerShell \(Azure PowerShell のインストールと構成の方法\)](powershell-install-configure.md)」をご覧ください。
- このチュートリアルで作成したキーやパスワードを使用して構成されるアプリケーション。サンプル アプリケーションは、[Microsoft ダウンロード センター](http://www.microsoft.com/ja-jp/download/details.aspx?id=45343)から入手できます。手順については、付属の Readme ファイルをご覧ください。


このチュートリアルは、Windows PowerShell の初心者向けに設計されていますが、モジュール、コマンドレット、セッションなどの基本概念を理解していることを前提としています。Windows PowerShell の詳細については、「[Getting Started with Windows PowerShell \(Windows PowerShell の概要\)](https://technet.microsoft.com/library/hh857337.aspx)」を参照してください。

このチュートリアルに表示されているすべてのコマンドレットの詳細なヘルプを取得するには、Get-Help コマンドレットを使用します。

	Get-Help <cmdlet-name> -Detailed

たとえば、Add-AzureAccount コマンドレットについてのヘルプを確認するには、次のように入力します。

	Get-Help Add-AzureAccount -Detailed

また、次のチュートリアルをお読みになり、Windows PowerShell での Azure リソース マネージャーについて詳しく理解してください。

- [How to install and configure Azure PowerShell \(Azure PowerShell のインストールと構成の方法\)](powershell-install-configure.md)
- [リソース マネージャーでの Windows PowerShell の使用](powershell-azure-resource-manager.md)


## <a id="connect"></a>サブスクリプションへの接続 ##

Azure PowerShell セッションを開始し、次のコマンドで Azure アカウントにサインインします。

    Add-AzureAccount

ポップアップ ブラウザー ウィンドウで、Azure アカウントのユーザー名とパスワードを入力します。Windows PowerShell は、このアカウントに関連付けられているすべてのサブスクリプションを取得し、既定で最初のサブスクリプションを使用します。

複数のサブスクリプションがあり、特定の 1 つのサブスクリプションを指定して Azure Key Vault を使用する場合は、次のように入力して自分のアカウントのサブスクリプションを表示します。

    Get-AzureSubscription

使用するサブスクリプションを指定するには、次のように入力します。

    Select-AzureSubscription -SubscriptionName <subscription name>

Azure PowerShell の詳細については、「[How to install and configure Azure PowerShell \(Azure PowerShell のインストールと構成の方法\)](powershell-install-configure.md)」をご覧ください。

## <a id="switch"></a>Azure リソース マネージャーの使用に切り替える ##

Key Vault のコマンドレットは Azure リソース マネージャーが必要なため、次のように入力して Azure リソース マネージャー モードに切り替えます。

	Switch-AzureMode AzureResourceManager

## <a id="resource"></a>新しいリソース グループを作成する ##

Azure リソース マネージャーを使用すると、すべての関連するリソースが 1 つのリソース グループ内に作成されます。このチュートリアルでは、'ContosoResourceGroup' という名前の新しいリソース グループを作成します。

	New-AzureResourceGroup –Name 'ContosoResourceGroup' –Location 'East Asia'

\-Location パラメーターでは、[Get-AzureLocation](https://msdn.microsoft.com/library/azure/dn654582.aspx) コマンドを使用して、この例の場所に別の場所を指定する方法を識別します。詳細情報が必要な場合は、`Get-Help Get-AzureLocation` と入力します。


## <a id="vault"></a>Key Vault を作成する ##

[New-AzureKeyVault](https://msdn.microsoft.com/library/azure/dn903602.aspx) コマンドレットを使用して、Key Vault を作成します。このコマンドレットには、3 つの必須パラメーター \(**リソース グループ名**、**Key Vault 名**、**地理的な場所**\) が含まれています。

たとえば、Key Vault 名に **ContosoKeyVault**、リソース グループ名に **ContosoResourceGroup**、場所に**東アジア**を使用する場合は、次のように入力します。

    New-AzureKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia' 

このコマンドレットの出力は、作成されたばかりの Key Vault のプロパティを示します。最も重要な 2 つのプロパティは、次のとおりです。

- **Name**: この例では、これは ContosoKeyVault です。この名前を他の ContosoKeyVault コマンドレットに使用できます。
- **vaultUri**: この例では、これは https://contosokeyvault.vault.azure.net/ です。その REST API から資格情報コンテナーを使用するアプリケーションは、この URI を使用する必要があります。

Azure アカウントは、この Key Vault ですべての操作の実行が許可されるようになりました。まだ、どのユーザーも許可されていません。

## <a id="add"></a>キーやシークレットを Key Vault に追加する ##

Azure Key Vault でソフトウェアで保護されたキーを作成する場合は、[Add-AzureKeyVaultKey](https://msdn.microsoft.com/library/azure/dn868048.aspx) コマンドレットを使用して次のように入力します。

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey' -Destination 'Software'


ただし、既存のソフトウェアで保護されたキーが、C:\\ ドライブの PFX ファイルに保存されていて、softkey.pfx という名前で Azure Key Vault にアップロードする場合は、次のように入力して、変数 **securepfxpwd** を .PFX ファイルのパスワード **123** に設定します。

    $securepfxpwd = ConvertTo-SecureString –String '123' –AsPlainText –Force

次のように入力して、.PFX ファイルからキーをインポートします。このキーは、Key Vault サービスのソフトウェアで保護されます。

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey' -KeyFilePath 'c:\softkey.pfx' -KeyFilePassword $securepfxpwd


作成したキーや、Azure Key Vault にアップロードしたキーは、その URI を使用すると参照できます。例: **https://ContosoKeyVault.vault.azure.net/Keys/ContosoFirstKey/a10f5336-9d93-44a3-9e26-e86e3488b768** 

このキーの URI を表示するには、次のように入力します。

	$Key.key.kid

資格情報コンテナーにシークレットを追加するには \(SQLPassword という名前のパスワードで、Azure Key Vault に Pa$$w0rd 値を設定\)、次のように入力して、まず Pa$$w0rd 値をセキュリティで保護された文字列に変換します。

	$secretvalue = ConvertTo-SecureString 'Pa$$w0rd' -AsPlainText -Force

その後、次のように入力します。

	$secret = Set-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword' -SecretValue $secretvalue

Azure Key Vault に追加したパスワードは、その URI を使用すると参照できます。例: **https://ContosoVault.vault.azure.net/Secrets/778c3e43-3fdb-4cdf-b58e-7f501eb41d68** 

このシークレットの URI を表示するには、次のように入力します。

	$secret.Id

作成したキーやシークレットを表示してみましょう。

- キーを表示するには、次のように入力します。`Get-AzureKeyVaultKey –VaultName 'ContosoKeyVault'`
- シークレットを表示するには、次のように入力します。`Get-AzureKeyVaultSecret –VaultName 'ContosoKeyVault'`

Key Vault とキーやシークレットは、アプリケーションを使用できる状態になりました。これらを使用するには、アプリケーションを承認する必要があります。

## <a id="register"></a>Azure Active Directory にアプリケーションを登録する ##

この手順は通常、開発者が別のコンピューター上で行います。これは Azure Key Vault に固有のものではありませんが、完全を期すために説明します。


>[AZURE.IMPORTANT]チュートリアルを完了するには、この手順で登録するアカウント、資格情報コンテナー、アプリケーションがすべて同じ Azure ディレクトリに格納されている必要があります。

Key Vault を使用するアプリケーションは、Azure Active Directory から取得したトークンを使用して認証する必要があります。これを行うには、アプリケーションの所有者は、まず Azure Active Directory でアプリケーションを登録する必要があります。登録の最後に、アプリケーションの所有者は次の値を取得します。


- **アプリケーション ID** \(クライアント ID とも呼ばれます\) と**認証キー** \(共有シークレットとも呼ばれます\)。アプリケーションは、トークンを取得するために、この 2 つの値を Azure Active Directory に示す必要があります。これを行うようにアプリケーションを構成する方法は、アプリケーションによって異なります。Key Vault のサンプル アプリケーション用に、アプリケーション所有者は app.config ファイルでこれらの値を設定します。



Azure Active Directory にアプリケーションを登録するには:

1. Azure ポータルにサインインします。
2. 左側で、**\[Active Directory\]** をクリックし、アプリケーションを登録するディレクトリを選択します。<br> <br> 注: Key Vault を作成した Azure サブスクリプションが含まれている、同じディレクトリを選択する必要があります。ディレクトリが不明な場合は、**\[設定\]** をクリックし、Key Vault を作成したサブスクリプションを見つけて、最後の列に表示されているディレクトリ名をご確認ください。

3. **\[アプリケーション\]** をクリックします。アプリがディレクトリに追加されていない場合は、このページには **\[アプリケーションの追加\]** リンクのみが表示されます。リンクをクリックするか、コマンド バーの **\[追加\]** をクリックします。
4.	**\[アプリケーションの追加\]** ウィザードの **\[What do you want to do?\]** ページで、**\[組織で開発中のアプリケーションを追加\]** をクリックします。
5.	**\[アプリケーション情報の指定\]** ページで、アプリケーションの名前を指定し、**\[Web アプリケーションや Web API\]** \(既定値\) を選択します。\[次へ\] アイコンをクリックします。
6.	**\[アプリのプロパティ\]** ページで、**\[サインオン URL\]** と **\[アプリケーション ID/URI\]** を Web アプリケーションに指定します。アプリケーションにこれらの値を設定していない場合は、この手順のための値を作成します \(両方のボックスに http://test1.contoso.com を指定するなど\)。これらのサイトが存在するかどうかは関係ありません。各アプリケーションのアプリケーション ID URI がディレクトリ内のすべてのアプリケーションで異なっていることが重要です。ディレクトリは、この文字列を使用してアプリを識別します。
7.	\[完了\] アイコンをクリックし、ウィザードで変更内容を保存します。
8.	\[クイック スタート\] ページで、**\[構成\]** をクリックします。 
9.	**\[キー\]** セクションまでスクロールし、期間を選択し、**\[保存\]** をクリックします。ページが更新され、キーの値が表示されます。このキーと **\[クライアント ID\]** の値を使用してアプリケーションを構成する必要があります\(この構成の手順はアプリケーション固有です\)。
10.	このページからクライアント ID 値をコピーします。この値は、資格情報コンテナーに権限を設定するために次の手順で使用します




## <a id="authorize"></a>キーまたはシークレットを使用してアプリケーションを承認する ##


資格情報コンテナーのキーやシークレットにアクセスするアプリケーションを承認するには、[Set-AzureKeyVaultAccessPolicy](https://msdn.microsoft.com/library/azure/dn903607.aspx) コマンドレットを使用します。

たとえば、資格情報コンテナー名が ContosoKeyVault で、承認するアプリケーションのクライアント ID が 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed の場合、アプリケーションの暗号化を解除し、資格情報コンテナー内のキーで署名することを承認するには、次のように実行します。

	Set-AzureKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToKeys decrypt,sign



## <a id="HSM"></a>ハードウェア セキュリティ モジュール \(HSM\) を使用する場合 ##

さらに安心感を高めたい場合には、ハードウェア セキュリティ モジュール \(HSM\) でキーのインポートや生成を行うことができ、キーは HSM の境界内から出ることはありません。HSM は FIPS 140-2 レベル 2 規格の認定を受けています。この要件が自分に当てはまらない場合は、このセクションをスキップし、[Key Vault と関連するキーとシークレットを削除する](#delete)に進んでください。

これらの HSM で保護されたキーを作成するには、[HSM で保護されたキーをサポートする資格情報コンテナーのサブスクリプション](../../../pricing/free-trial)が必要です。


資格情報コンテナーを作成するときに、'SKU' パラメーターを追加します。

	New-AzureKeyVault -VaultName 'ContosoKeyVaultHSM' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia' -SKU 'Premium'

この資格情報コンテナーには、ソフトウェアで保護されたキー \(前述のとおり\) と HSM で保護されたキーを追加できます。HSM で保護されたキーを作成するには、Destination パラメーターを 'HSM' に設定します。

	$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -Destination 'HSM'

次のコマンドを使用して、自分のコンピューターの PFX ファイルからキーをインポートできます。このコマンドでは、Key Vault サービスでキーを HSM にインポートします。

	$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -KeyFilePath 'c:\softkey.pfx' -KeyFilePassword $securepfxpwd -Destination 'HSM'

次のコマンドは、“bring your own key" \(BYOK\) パッケージをインポートします。これを使用すると、ローカルの HSM でキーを生成し、これを Key Vault サービスで HSM に転送でき、キーは HSM の境界内から出ることはありません。

	$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -KeyFilePath 'c:\ITByok.byok' -Destination 'HSM'

この BYOK パッケージを生成する方法の詳細な手順については、「[Azure Key Vault の HSM 保護キーを生成し、転送する方法](https://msdn.microsoft.com/library/azure/dn903624.aspx)」をご覧ください。

## <a id="delete"></a>Key Vault と関連付けられているキーやシークレットを削除する ##

Key Vault と、これに含まれるキーやシークレットが不要になった場合は、[Remove-AzureKeyVault](https://msdn.microsoft.com/library/azure/dn903603.aspx) コマンドレットを使用して Key Vault を削除できます。

	Remove-AzureKeyVault -VaultName 'ContosoKeyVault'

または、Key Vault やそのグループに含まれる他のすべてのリソースを含む、Azure リソース グループ全体を削除できます。

	Remove-AzureResourceGroup -ResourceGroupName 'ContosoResourceGroup'


## <a id="other"></a>その他の Azure PowerShell コマンドレット ##

Azure Key Vault の管理に役立つその他のコマンドは次のとおりです。

- `$Keys = Get-AzureKeyVaultKey -VaultName 'ContosoKeyVault'`: このコマンドは、すべてのキーと選択したプロパティを表形式で取得します。
- `$Keys[0]`: このコマンドは、指定されたキーのプロパティの完全な一覧を表示します。
- `Get-AzureKeyVaultSecret`: このコマンドは、すべてのシークレットの名前と選択したプロパティを表形式で一覧します。
- `Remove-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey'`: 特定のキーを削除する方法の例です。
- `Remove-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword'`: 特定のシークレットを削除する方法の例です。






## <a id="next"></a>次のステップ ##

Azure Key Vault の Windows PowerShell コマンドレットの一覧については、「[Azure Key Vault Cmdlets \(Azure Key Vault コマンドレット\)](https://msdn.microsoft.com/library/azure/dn868052.aspx)」をご覧ください。

プログラミング リファレンスについては、「[Key Vault の REST API](https://msdn.microsoft.com/library/azure/dn903609.aspx)」と「[Key Vault C\# クライアント API リファレンス](https://msdn.microsoft.com/library/azure/dn903628.aspx)」をご覧ください。



<!--HONumber=54-->