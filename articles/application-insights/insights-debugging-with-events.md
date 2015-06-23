<properties 
	pageTitle="イベントに関連したデバッグの方法" 
	description="Azure でイベントを表示する方法について説明します。" 
	authors="HaniKN-MSFT" 
	manager="kamrani" 
	editor="" 
	services="application-insights" 
	documentationCenter=""/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="11/24/2014" 
	ms.author="hanikn"/>

# Azure のリソースまたはリソース グループに影響を与えるイベントの監視

1. [Azure プレビュー ポータル](https://portal.azure.com/)にサインインします。
2. 左側のナビゲーション バー (**ジャンプ バー**とも呼びます) にある **[参照]**をクリックします。  
    ![Browse Hub](./media/insights-debugging-with-events/Insights_Browse.png)
3. (関心のあるイベントに応じて) リソースを選択します。説明の目的で、このドキュメントのスクリーン ショットでは、リソース グループに対応するイベントを使用しています。
4. **[リソース グループ]** ブレードで、リソース グループの名前をクリックします。この結果、そのリソース グループに対応するブレードに移動します。  
    ![Resource groups](./media/insights-debugging-with-events/Insights_SelectRG.png)
5. リソース グループのブレードには、**[過去 1 週間のイベント]** というパーツが含まれています。このパーツ内にある各バーは、過去 1 週間の各日に発生したイベントの数を表します。各バーに、青およびピンクという 2 つの異なる色が表示されていることがあります。ピンクはその日の**失敗**イベントを表し、青はそれ以外のすべてのイベントを表します。  
    ![Resource groups](./media/insights-debugging-with-events/Insights_RGBlade.png)
6. ここで、**[過去 1 週間のイベント]** パーツをクリックします。**[イベント]** ブレードという新しいブレードが表示されます。この中には、リソース グループに影響を及ぼした、過去 1 週間のすべてのイベントが含まれています。
    ![Resource groups](./media/insights-debugging-with-events/Insights_AllEvents.png)
7. イベントのいずれかをクリックします。  
    ![Resource groups](./media/insights-debugging-with-events/Insights_EventDetails.png)  
    イベントに関して多くの詳細を示す新しいブレードが表示されます。**[失敗]** イベントに関しては、このページで通常、デバッグの目的で役立つ詳細を示す **[サブステータス]** および **[プロパティ]** というセクションが表示されます。


<!--HONumber=46--> 
 