<properties 
	pageTitle="Java から BLOB ストレージを使用する方法 | Microsoft Azure" 
	description="Azure BLOB サービスを使用して、BLOB の内容をアップロード、ダウンロード、一覧表示、および削除する方法について説明します。コード サンプルは Java で記述されています。" 
	services="storage" 
	documentationCenter="java" 
	authors="rmcmurray" 
	manager="adinah" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.workload="storage" 
	ms.tgt_pltfrm="na" 
	ms.devlang="Java" 
	ms.topic="article" 
	ms.date="03/11/2015" 
	ms.author="robmcm"/>

# Java から BLOB ストレージを使用する方法

[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)]

## 概要

このガイドでは、Microsoft Azure BLOB ストレージ サービスを使用して一般的なシナリオを実行する方法について説明します。サンプルは Java で記述され、[Azure Storage SDK for Java][] を利用しています。紹介するシナリオは、BLOB の**アップロード**、**一覧表示**、**ダウンロード**、および**削除**です。BLOB の詳細については、[次のステップ](#NextSteps) セクションをご覧ください。

> [AZURE.NOTE] SDK は、Android デバイスで Azure Storage を使用する開発者向けに用意されています。詳細については、[Azure Storage SDK for Android に関するページ][]を参照してください。 

[AZURE.INCLUDE [storage-blob-concepts-include](../../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## Java アプリケーションの作成

このガイドで使用するストレージ機能は、Java アプリケーション内でローカルで実行することも、Azure の Web ロールまたは worker ロールで動作するコード内で実行することもできます。

そのためには、Java Development Kit (JDK) をインストールし、Azure サブスクリプションに Azure ストレージ アカウントを作成する必要があります。その後、開発システムが、GitHub の [Azure Storage SDK for Java][] リポジトリに示されている最小要件と依存関係を満たしていることを確認する必要があります。システムがそれらの要件を満たしている場合は、指示に従って、そのリポジトリからシステムに Azure Storage Libraries for Java をダウンロードしてインストールできます。それらのタスクが完了したら、この記事の例を使用した Java アプリケーションを作成できます。

## BLOB ストレージにアクセスするようにアプリケーションを構成する

Azure Storage API を使用して BLOB にアクセスする Java ファイルの先頭には、次の import ステートメントを追加します。

    // Include the following imports to use blob APIs.
    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.blob.*;

## Azure のストレージ接続文字列の設定

Azure ストレージ クライアントでは、ストレージ接続文字列を使用して、
データ管理サービスにアクセスするためのエンドポイントおよび資格情報を保存します。クライアント アプリケーションの実行時、ストレージ接続文字列を次の形式で指定する必要があります。 *AccountName* と  *AccountKey* の値には、管理ポータルに表示されるストレージ アカウントの名前とプライマリ アクセス キーを使用します。この例では、接続文字列を保持する静的フィールドを宣言する方法を示しています。

    // Define the connection-string with your values
    public static final String storageConnectionString = 
        "DefaultEndpointsProtocol=http;" + 
        "AccountName=your_storage_account;" + 
        "AccountKey=your_storage_account_key";

Microsoft Azure のロール内で実行されるアプリケーションでは、この文字列はサービス構成ファイルである *ServiceConfiguration.cscfg* に格納でき、**RoleEnvironment.getConfigurationSettings** メソッドの呼び出しを使用してアクセスできます。次の例では、サービス構成ファイル内の  *StorageConnectionString* という名前の **Setting** 要素から接続文字列を取得しています。

    // Retrieve storage account from connection-string.
    String storageConnectionString = 
        RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");

次のサンプルでは、これら 2 つのメソッドのいずれかを使用してストレージ接続文字列を取得するとします。

## 方法:コンテナーを作成する

CloudBlobClient オブジェクトを使用すると、コンテナーと BLOB の参照オブジェクトを取得できます。次のコードでは、**CloudBlobClient** オブジェクトを作成しています。(注:**CloudStorageAccount** オブジェクトを作成する方法は他にもあります。詳細については、[Azure ストレージ クライアント SDK リファレンス]の **CloudStorageAccount** を参照してください)。

BLOB はすべてコンテナー内に格納されます。**CloudBlobClient** オブジェクトにより、使用するコンテナーへの参照を取得します。コンテナーが存在しない場合は、**createIfNotExists** メソッドを使用して作成できます。存在する場合は、このメソッドによって既存のコンテナーが返されます。既定では、新しいコンテナーはプライベートであるため、このコンテナーから BLOB をダウンロードするにはストレージ アクセス キーを指定する必要があります (前と同じ方法で)。

	try
    {
        // Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    	// Create the blob client.
	   CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Get a reference to a container.
	   // The container name must be lower case
	   CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

	   // Create the container if it does not exist.
    	container.createIfNotExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

### 省略可能:パブリック アクセス用のコンテナーの構成

コンテナーのアクセス許可は、既定では、プライベート アクセス用に構成されますが、インターネット上のすべてのユーザーに対して読み取り専用のパブリック アクセスを許可するようにコンテナーのアクセス許可を簡単に構成できます。

    // Create a permissions object.
    BlobContainerPermissions containerPermissions = new BlobContainerPermissions();

    // Include public access in the permissions object.
    containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);

    // Set the permissions on the container.
    container.uploadPermissions(containerPermissions);

## 方法:コンテナーに BLOB をアップロードする

ファイルを BLOB にアップロードするには、コンテナーの参照を取得し、それを使用して BLOB の参照を取得します。BLOB の参照を取得したら、BLOB 参照時に upload を呼び出すことで、任意のストリームを BLOB にアップロードできます。この処理により、BLOB が存在しない場合は作成され、存在する場合は上書きされます。このコード サンプルはこのことを示しており、既にコンテナーが作成されていることを前提としています。

	try
    {
        // Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    	// Create the blob client.
    	CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Retrieve reference to a previously created container.
    	CloudBlobContainer container = blobClient.getContainerReference("mycontainer");
			
        // Define the path to a local file.
        final String filePath = "C:\\myimages\\myimage.jpg";

    	// Create or overwrite the "myimage.jpg" blob with contents from a local file.
    	CloudBlockBlob blob = container.getBlockBlobReference("myimage.jpg");
    	File source = new File(filePath);
    	blob.upload(new FileInputStream(source), source.length());
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 方法:コンテナー内の BLOB を一覧表示する

コンテナー内の BLOB を一覧表示するには、BLOB のアップロード時と同様に、まずコンテナーの参照を取得します。コンテナーの **listBlobs** メソッドを **for** ループと共に使用できます。次のコードでは、コンテナー内の各 BLOB の URI をコンソールに出力しています。

	try
    {
        // Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    	// Create the blob client.
    	CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

    	// Retrieve reference to a previously created container.
    	CloudBlobContainer container = blobClient.getContainerReference("mycontainer");
			
    	// Loop over blobs within the container and output the URI to each of them.
    	for (ListBlobItem blobItem : container.listBlobs()) {
	       System.out.println(blobItem.getUri());
	   }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

BLOB サービスには、コンテナー内のディレクトリの概念もあります。これは、BLOB をよりフォルダーに近い構造で整理できるようにするためです。

たとえば、"photos" という名前のコンテナーを作成して、その中に "rootphoto1"、"2010/photo1"、"2010/photo2"、および "2011/photo1" という名前の BLOB をアップロードすることができます。このようにすると、"photos" コンテナー内に "2010" および "2011" という仮想ディレクトリが作成されます。"photos" コンテナーの **listBlobs** を呼び出すと、返されるコレクションには最上位レベルにあるディレクトリおよび BLOB を表す **CloudBlobDirectory** および **CloudBlob** オブジェクトが含まれています。この例の場合は、ディレクトリ "2010" および "2011" と、写真 "rootphoto1" が返されます。**instanceof** 演算子を使用してこれらのオブジェクトを区別できます。

必要に応じて、**listBlobs** メソッドにパラメーターを渡すことができます。
そのためには、**useFlatBlobListing** パラメーターに true を設定しておく必要があります。これにより、
ディレクトリに関係なく、すべての BLOB が返されるようになります。サブスクリプションの
[Azure ストレージ クライアント SDK に関するページ]の **CloudBlobContainer.listBlobs** の項を参照してください。

## 方法:BLOB をダウンロードする

BLOB をダウンロードするには、BLOB の参照を取得するために BLOB をアップロードしたときと同じ手順に従います。アップロードの例では、BLOB オブジェクトの upload を呼び出しました。次の例では、download を呼び出して BLOB の内容を **FileOutputStream** などのストリーム オブジェクトに転送しています。このオブジェクトは BLOB をローカル ファイルに保持するために使用できます。

    try
    {
    	// Retrieve storage account from connection-string.
	   CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

	   // Create the blob client.
	   CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Retrieve reference to a previously created container.
	   CloudBlobContainer container = blobClient.getContainerReference("mycontainer");
			
	   // Loop through each blob item in the container.
	   for (ListBlobItem blobItem : container.listBlobs()) {
	       // If the item is a blob, not a virtual directory.
	       if (blobItem instanceof CloudBlob) {
	           // Download the item and save it to a file with the same name.
    	        CloudBlob blob = (CloudBlob) blobItem;
    	        blob.download(new FileOutputStream("C:\\mydownloads" + blob.getName()));
    	    }
    	}
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 方法:BLOB を削除する

BLOB を削除するには、BLOB の参照を取得し、**deleteIfExists** を呼び出します。

    try
    {
	   // Retrieve storage account from connection-string.
	   CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

	   // Create the blob client.
	   CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Retrieve reference to a previously created container.
	   CloudBlobContainer container = blobClient.getContainerReference("mycontainer");
			
	   // Retrieve reference to a blob named "myimage.jpg".
	   CloudBlockBlob blob = container.getBlockBlobReference("myimage.jpg");

	   // Delete the blob.
	   blob.deleteIfExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 方法:BLOB コンテナーを削除する

最後に、BLOB コンテナーを削除するには、BLOB コンテナーの参照を取得し、
**deleteIfExists** を呼び出します。

    try
    {
	   // Retrieve storage account from connection-string.
	   CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

	   // Create the blob client.
	   CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Retrieve reference to a previously created container.
	   CloudBlobContainer container = blobClient.getContainerReference("mycontainer");
			
	   // Delete the blob container.
	   container.deleteIfExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 次のステップ

これで、BLOB ストレージの基本を学習できました。さらに複雑なストレージ タスクを実行するには、次のリンク先を参照してください。

- [Azure Storage SDK for Java]
- [Azure ストレージ クライアント SDK リファレンス]
- [Azure Storage REST API]
- [Azure のストレージ チーム ブログ]

[Azure SDK for Java]: http://azure.microsoft.com/develop/java/
[Azure Storage SDK for Java]: https://github.com/azure/azure-storage-java
[Azure Storage SDK for Android に関するページ]: https://github.com/azure/azure-storage-android
[Azure ストレージ クライアント SDK リファレンス]: http://dl.windowsazure.com/storage/javadoc/
[Azure Storage REST API]: http://msdn.microsoft.com/library/azure/gg433040.aspx
[Azure のストレージ チーム ブログ]: http://blogs.msdn.com/b/windowsazurestorage/

<!--HONumber=49--> 