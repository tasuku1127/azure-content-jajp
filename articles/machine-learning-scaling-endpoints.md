<properties 
	pageTitle="API エンドポイントのスケーリング" 
	description="Azure Machine Learning で Web サービス エンドポイントのスケーリングを行う" 
	services="machine-learning" 
	authors="hiteshmadan" 
	manager="padou" 
	editor=""/>

<tags
	ms.service="machine-learning"
	ms.devlang="multiple"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="tbd" 
	ms.date="02/19/2015"
	ms.author="himad"/>


# API エンドポイントのスケーリング

Azure Machine Learning の Web サービス エンドポイントでは、スロットル レベルが選択可能であり、エンドポイントが使用するレートに合わせることができます。

2 種類の段階によって、エンドポイントで行われる調整量を制御します \(スロットル レベル "低" と "高"\)。有料で利用しているお客様のみ、スロットル レベルを "高" に設定できます。同時呼び出しの最大数はスロットル レベル "低" の場合 4 件で、スロットル レベル "高" の場合 20 ～ 200 件です。


同期 API は通常、短い待機時間が望まれる状況で使用されます。ここでの待機時間とは、API が 1 つの要求を完了するのにかかる時間を指し、ネットワークの遅延に影響しないものです。たとえば待機時間が 50 ms の API があるとします。スロットル レベル "高" で同時呼び出しの最大数が 20 件のときに使用可能な容量をすべて使用するには、この API を 1 秒あたり 20 × 1000 / 50 = 400 回呼び出す必要があります。さらに増やして、同時呼び出しの最大数 200 件にすると、API を 1 秒あたり 4,000 回呼び出すことができ、この場合の待機時間は 50 ms と想定されます。

同時呼び出しの最大数 200 件でサポートされる内容を上回る負荷の API を呼び出す場合は、同じ Web サービスに複数のエンドポイントを作成し、それら全体で負荷を無作為に分配する必要があります。

同時呼び出しの数を非常に大きくする場合、これに相当する高いレートで API を動作させないと悪影響が出ることがありますのでご注意ください。高い負荷で構成された API に比較的低い負荷を割り当てた場合、不規則なタイムアウトや待機時間の急増が発生することがあります。

スロットルの設定調整は、同期 API の動作にのみ影響します。同期 API で "503 サービスを利用できません" の応答が頻繁に表示される場合、これらの設定を調整してください。

管理 UI でスロットル レベルを切り替えられます。カスタムの同時呼び出しの数でスロットル レベル "高" を使用するには、パッチ エンドポイント API を使用してください。

- manage.windowsazure.com を開きます。
- \[Machine Learning\] タブに移動します。
- ワークスペースをクリックします。
- エンドポイントが含まれる Web サービスに移動します。![Web サービスに移動する](./media/machine-learning-scaling-endpoints/figure-1.png)

- エンドポイントをクリックした後、\[構成\] タブをクリックします。![エンドポイントの構成に移動する](./media/machine-learning-scaling-endpoints/figure-2.png)


- スロットル レベルを \[高\] に変更し、\[保存\] をクリックします。



<!--HONumber=54-->