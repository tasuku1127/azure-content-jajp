<properties
	pageTitle="OS プラットフォームとハードウェアの互換性 | Microsoft Azure"
	description="IoT デバイス SDK と OS プラットフォームおよびデバイス ハードウェアとの互換性をまとめてあります。"
	services="iot-hub"
	documentationCenter=""
	authors="hegate"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="na"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="10/09/2015"
     ms.author="hegate"/>

# OS プラットフォームとハードウェアの、デバイス SDK との互換性

このドキュメントでは、SDK を使用できるさまざまな OS プラットフォームと、[Microsoft Azure Certified for IoT](#microsoft-azure-certified-for-iot) プログラムに含まれる特定のデバイス構成について説明します。既にデバイスがある場合、プログラムに含まれるデバイスの一覧を参照し、対応状況に関するデバイス固有の情報を確認してください。使用するデバイスがわからない場合、[OS プラットフォームやライブラリ](#os-platforms)の対応状況のセクションを参照してください。


## OS プラットフォーム

Azure IoT ライブラリは、次のオペレーティング システム プラットフォームでテストされています。


|Linux/Unix OS プラットフォーム | バージョン|
|:---------------|:------------:|
|Debian Linux| 7\.5|
|Fedora Linux|20|
|Raspbian Linux| 3\.18 |
|Ubuntu Linux| 14\.04 |
|Yocto Linux|2\.1 |

|Windows OS プラットフォーム | バージョン|
|:---------------|:------------:|
|Windows デスクトップ| 7、8、10 |
|Windows IoT Core| 10 |
|Windows Server| 2012 R2|

|その他のプラットフォーム | バージョン|
|:---------------|:------------:|
|mbed | 2\.0 |
|TI-RTOS | 2\.x |



## C ライブラリ

[C 用 Microsoft Azure IoT デバイス SDK](https://github.com/Azure/azure-iot-sdks/blob/master/c/readme.md) は、次の構成でテストされています。

|OS プラットフォーム| バージョン|プロトコル|
|:---------|:----------:|:----------:|
|Debian Linux| 7\.5 | HTTPS、AMQP、MQTT |
|Fedora Linux| 20 | HTTPS、AMQP、MQTT |
|mbed OS| 2\.0 | HTTPS、AMQP |
|TI-RTOS| 2\.x | HTTPS |
|Ubuntu Linux| 14\.04 | HTTPS、AMQP、MQTT |
|Windows デスクトップ| 7、8、10 | HTTPS、AMPQ、MQTT |
|Yocto Linux|2\.1 | HTTPS、AMQP|



## Node.js ライブラリ

[Node.js 用 Microsoft Azure IoT デバイス SDK](https://github.com/Azure/azure-iot-sdks/blob/master/node/device/readme.md) は、次の構成でテストされています。


|ランタイム| バージョン|プロトコル|
|:---------|:----------:|:----:|
|Node.js| 4\.1.0 | HTTPS|



## Java ライブラリ

[Java 用 Microsoft Azure IoT デバイス SDK](https://github.com/Azure/azure-iot-sdks/blob/master/java/device/readme.md) は、次の構成でテストされています。

|ランタイム| バージョン|プロトコル|
|:---------|:----------:|----|
|Java SE (Windows)| 1\.7 | HTTPS、AMQP |
|Java SE (Linux)| 1\.8 | HTTPS、AMQP|

Java 用 Microsoft Azure IoT サービス SDK は、次の構成でテストされています。

|ランタイム| バージョン|プロトコル|
|:---------|:----------:|:-----|
|Java SE| 1\.8 | HTTPS、AMQP |


## CSharp

[.NET 用 Microsoft Azure IoT デバイス SDK](https://github.com/Azure/azure-iot-sdks/blob/master/csharp/readme.md) は、次の構成でテストされています。

|OS プラットフォーム| バージョン|プロトコル|
|:---------|:----------:|:----------:|
|Windows デスクトップ| 7、8、10 | HTTPS、AMPQ|
|Windows IoT Core|10 | HTTPS|

管理対象エージェントのコードには、Microsoft .NET Framework 4.5 が必要です。


## Microsoft Azure Certified for IoT

**Microsoft Azure Certified for IoT** は、開発者やアーキテクトが対応状況のシナリオを理解できるように、より広範な IoT エコシステムを Microsoft Azure に接続するパートナー プログラムです。具体的には、IoT プロジェクト (概念実証フェーズまたはパイロット フェーズ) をすぐに始めるために役立つ、OS とデバイスの信頼できる組み合わせの一覧を提供しています。認定された、デバイスとオペレーティング システムの組み合わせを使用することにより、デバイスが Azure IoT Suite および Azure IoT Hub に対応していることを確認するために必要な作業とカスタマイズを減らし、IoT プロジェクトをすぐに開始することができます。

## Certified for IoT デバイス

**Certified for IoT** デバイスは、Azure SDK への対応がテストされ、IoT アプリケーションで使用する準備が整っています。具体的には、オペレーティング システム プラットフォームと、コードの言語に基づいて対応状況を特定します。

#### デバイスの一覧

各デバイスは、デバイスの製造元が選択した OS と言語で、SDK が機能することが認定されています。たとえば、BeagleBone Black は、Debian で、C、JavaScript、および Java 言語を使用して動作します。これは、特定のデバイスで任意の言語と OS の組み合わせにより、開発者がアプリケーションを構築できることを意味します。

|デバイス| テスト対象の OS |言語|
|:---------|:----------|:----------|
|Raspberry Pi 2| Raspbian | C、JavaScript、Java |
|Raspberry Pi 2| Windows 10 IoT Core| C、JavaScript、C#|
|BeagleBone Black| Debian |C、JavaScript、Java|
|BeagleBone Green|Debian |C、JavaScript、Java|
|TI CC3200 | TI-RTOS 2.x|C|
|Intel Edison |Yocto |C、JavaScript|
|MinnowBoard Max |Windows 7、8、10 |C#|
|Arrow DragonBoard 410c |Windows 10 IoT Core | C#|
|Freescale FRDM K64 |mbed 2.0 | C|

## 次のステップ

**Certified for IoT** デバイスを使用したソリューションの開発に関する詳細については、[こちら](http://azure.com/iotdev)を参照してください。

<!---HONumber=Oct15_HO3-->