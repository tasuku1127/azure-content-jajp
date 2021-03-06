<properties 
	pageTitle="Media Encoder Standard の形式とコーデック" 
	description="このトピックでは、Azure Media Encoder Standard の形式とコーデックの概要を示します。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako,anilmur" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="10/15/2015"  
	ms.author="juliako"/>

#Media Encoder Standard の形式とコーデック


このドキュメントでは、Media Encoder Standard で使用できる一般的なインポートおよびエクスポート ファイル形式の一覧を示しています。


##入力コンテナー/ファイル形式

ファイル形式 (ファイル拡張子)|サポートされています
---|---|---|---
(H.264 および AAC コーデックでの) FLV (.flv) |あり 
MXF (.mxf) |あり 
GXF (.gxf) |あり 
MPEG2-PS、MPEG2-TS、3GP (.ts、.ps、.3gp、.3gpp、.mpg) |あり 
Windows Media Video (WMV)/ASF (.wmv、.asf) |あり 
AVI (非圧縮 8-bit/10-bit) (.avi)|あり 
MP4 (.mp4、.m4a、.m4v)/ISMV (.isma、.ismv)|あり 
[Microsoft Digital Video Recording (DVR-MS)](https://msdn.microsoft.com/library/windows/desktop/dd692984) (.dvr-ms) |あり 
Matroska/WebM (.mkv) |あり 
WAVE/WAV (.wav) |あり 
QuickTime (.mov) |あり
 
###入力コンテナーのオーディオ形式 

Media Encoder Standard の入力コンテナーは次のオーディオ形式に対応しています。

- オーディオ トラックにインターリーブ ステレオまたは 5.1 サンプルが含まれる MXF、GXF、QuickTime ファイル。

または

- オーディオが個別 PCM トラックとして送信されるが、(ステレオまたは 5.1 への) チャネル マッピングをファイル メタデータから推測できる MXF、GXF、QuickTime ファイル。

明示的/ユーザー指定のチャネル マッピングが近い将来にサポートされる予定です。


##入力ビデオ コーデック

入力ビデオ コーデック|サポートされています
---|---|---|---
AVC 8-bit/10-bit、最大 4:2:2 (AVCIntra を含む) |8 ビット 4:2:0 および 4:2:2 
Avid DNxHD (MXF) |あり 
DVCPro/DVCProHD (MXF) |あり 
デジタル ビデオ (DV) (AVI ファイルで) |あり
JPEG 2000 |あり 
MPEG-2 (XDCAM、XDCAM HD、XDCAM IMX、CableLabs®、D10 など、最大 422 プロファイルおよびハイ レベル)|最大 422 プロファイル 
MPEG-1 |あり 
VC-1/WMV9 |あり 
Canopus HQ/HQX |いいえ 
MPEG-4 Part 2 |あり 
[Theora](https://en.wikipedia.org/wiki/Theora) |あり 
YUV420 非圧縮または中間 |あり
Apple ProRes 422 |あり
Apple ProRes 422 LT |あり
Apple ProRes 422 HQ |あり
Apple ProRes プロキシ|あり
Apple ProRes 4444 |あり
Apple ProRes 4444 XQ |あり



##入力オーディオ コーデック

入力オーディオ コーデック|サポートされています
---|---|---|---
AAC (AAC-LC、AAC-HE、AAC-HEv2。最大 5.1)|あり 
MPEG Layer 2|あり 
MP3 (MPEG-1 Audio Layer 3)|あり 
Windows Media オーディオ|あり 
WAV/PCM|あり 
[FLAC](https://en.wikipedia.org/wiki/FLAC)</a>|あり 
[Opus](https://en.wikipedia.org/wiki/Opus_(audio_format) |あり 
[Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a>|あり 
AMR (アダプティブ マルチ レート)|あり
AES (SMPTE 331M および 302M、AES3-2003) |いいえ 
Dolby® E |いいえ 
Dolby® Digital (AC3) |いいえ 
Dolby® Digital Plus (E-AC3) |いいえ 


##出力形式とコーデック

次の表では、エクスポートでサポートされるコーデックおよびファイル形式の一覧を示します。


ファイル形式|ビデオ コーデック|オーディオ コーデック
---|---|---
MP4 <br/><br/>(マルチビットレートの MP4 コンテナーを含む) |H.264 (High、Main、Baseline Profile)|AAC-LC、HE-AAC v1、HE-AAC v2 
MPEG2-TS |H.264 (High、Main、Baseline Profile)|AAC-LC、HE-AAC v1、HE-AAC v2 



##Media Services のラーニング パス

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##フィードバックの提供

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

##関連項目

[Azure Media Services を使用してオンデマンド コンテンツをエンコードする](media-services-encode-asset.md)

[メディア エンコーダー スタンダードを使用したエンコード方法](media-services-dotnet-encode-with-media-encoder-standard.md)

<!---HONumber=Nov15_HO3-->