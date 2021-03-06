<properties

	pageTitle="Managing security groups in Azure Active Directory | Microsoft Azure"
	description="Covers how to sign up for Azure and first steps you can try with Azure AD."
	services="active-directory"
	documentationCenter=""
	authors="femila"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory" 
	ms.workload="identity" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="10/21/2015" 
	ms.author="femila"/>


#Azure Active Directory のセキュリティ グループの管理


##セキュリティ グループを作成して管理する方法

**Microsoft Azure 管理ポータルからグループを作成するには**

1. 管理ポータルで **[Active Directory]** をクリックし、該当する組織のディレクトリ名をクリックします。
2. **[グループ]** タブをクリックします。
3. [グループ] ページの **[グループの追加]** をクリックします。
4. **[グループの追加]** ウィンドウで、グループの名前と説明を指定します。
5. この作業には、Office 365 アカウント ポータル、Windows Intune アカウント ポータル、Microsoft Azure 管理ポータルを使用することができます。自分の組織がサブスクライブしているサービスに応じたポータルを使用してください。ポータルを使用した Azure Active Directory の管理の詳細については、Azure AD ディレクトリの管理に関するページを参照してください。

## セキュリティ グループのユーザーの割り当てと削除の方法

**Microsoft Azure 管理ポータルでグループにメンバーを追加するには**

1. 管理ポータルで [Active Directory] をクリックし、該当する組織のディレクトリ名をクリックします。
2. **[グループ]** タブをクリックします。
3. **[グループ]** ページで、メンバーの追加先となるグループの名前をクリックします。選択したグループの **[メンバー]** タブが既定で表示されます。
4. そのグループのページで、**[メンバーの追加]** をクリックします。
5. このグループのメンバーとして追加するユーザーまたはグループの名前を **[メンバーの追加]** ページでクリックし、その名前が [選択済み] ウィンドウに追加されたことを確認します。


**Microsoft Azure 管理ポータルでグループからメンバーを削除するには**

1. 管理ポータルで [Active Directory] をクリックし、該当する組織のディレクトリ名をクリックします。
2. **[グループ]** タブをクリックします。
3. [グループ] ページで、メンバーを削除するグループの名前をクリックします。
4. そのグループのページで、**[メンバー]** タブをクリックします。
5. グループから削除するメンバーの名前をそのグループのページでクリックし、**[削除]** をクリックします。
6. 削除の確認メッセージが表示されるので、このメンバーをグループから削除してよい場合は、**[はい]** をクリックします。


##ルールを使用してセキュリティ グループのメンバーを動的に管理する方法
**特定のグループの動的メンバーシップ管理を有効にするには、次の手順を実行します。**

1. Microsoft Azure 管理ポータルの **[グループ]** タブで編集対象のグループを選択し、そのグループの **[構成]** タブで **[動的メンバーシップを有効にする]** スイッチを **[はい]** に設定します。
2. ここで、このグループに対する動的メンバーシップの働きを制御する単純なルールを 1 つグループに設定することができます。**[追加するユーザーの所属]** オプション ボタンがオンになっていることを確認し、ユーザーのプロパティ (department、jobTitle など) をプルダウン メニューから選択します。 
3. 次に、条件 (等しくない、等しい、指定値で始まらない、指定値で始まる、指定値を含まない、指定値を含む、一致しない、一致する) を選び、選択したユーザー プロパティの値を指定します。
4. たとえば、グループを SaaS アプリケーションに割り当てる場合 (詳細については、「Azure AD で SaaS アプリケーションにグループのアクセスを割り当てる」を参照) で、なおかつ jobTitle が Sales Rep と等しい (-eq) ときにユーザーを追加するようにルールを設定して、そのグループの動的メンバーシップを有効にした場合、Azure AD ディレクトリ内で、役職が Sales Rep に設定されているすべてのユーザーに、この SaaS アプリケーションへのアクセス権が割り当てられます。

ここでは、Azure Active Directory の追加情報を提供するいくつかのトピックを紹介します。

* [Azure Active Directory グループによるリソースのアクセス管理](active-directory-manage-groups.md)

* [Azure Active Directory とは](active-directory-whatis.md)

* [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)

<!---HONumber=Oct15_HO4-->