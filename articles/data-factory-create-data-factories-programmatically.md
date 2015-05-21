﻿<properties title="Create, monitor, and manage Azure data factories using Data Factory SDK" pageTitle="Data Factory SDK を使用して Azure Data Factory を作成、監視、管理する" description="プログラムを使用して、作成、監視、およびデータの工場出荷時の SDK を使用して Azure のデータのファクトリを管理する方法について説明します。" metaKeywords=""  services="data-factory" solutions=""  documentationCenter="" authors="spelluru" manager="jhubbard" editor="monicar" />

<tags ms.service="data-factory" ms.workload="data-services" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="12/08/2014" ms.author="spelluru" />

# Data Factory .NET SDK を使用して Azure Data Factory を作成、監視、管理する
Data Factory .NET SDK を使用して Azure Data Factory をプログラムによって作成、監視、管理できます。この記事には、Data Factory の作成と監視を行うサンプルの .NET コンソール アプリケーションを作成できるチュートリアルが含まれています。Data Factory .NET SDK の詳細については、「[Data Factory Class Library Reference (Data Factory クラス ライブラリ リファレンス)][adf-class-library-reference]」を参照してください。 

## チュートリアル:Data Factory .NET SDK を使用して Azure Data Factory を作成する

**前提条件:**

- Visual Studio 2012 または 2013
- [Windows Azure .NET SDK][azure-developer-center] のダウンロードとインストール
- Azure Data Factory の NuGet パッケージをダウンロードしてインストールします。手順はこのチュートリアルにあります。

### 手順 1.Data Factory .NET SDK を使用して Azure Data Factory を作成する
1. Visual Studio 2012 または 2013 を使用して、C# .NET コンソール アプリケーションを作成します。
	<ol type="a">
		<li>ここで、 <b>Visual Studio 2012</b> または <b>Visual Studio 2013</b>を起動します。</li>
		<li>次に、 <b>ファイル</b>をクリックし、 <b>新規</b>をポイントして、 <b>[プロジェクト]</b>をクリックします。</li> 
		<li>ここで、 <b>テンプレート</b>を展開し、 <b>Visual C#</b>を選択します。このチュートリアルでは C# を使用しますが、どの .NET 言語でも使用できます。</li> 
		<li>右側のプロジェクトの種類の一覧から <b>コンソール アプリケーション</b> を選択します。</li>
		<li>次に、 <b>DataFactoryAPITestApp</b> を <b>名前</b>に入力します</li> 
		<li>。 <b>C:\ADFGetStarted</b> を <b>場所</b>に選択します</li>
		<li>。 <b>OK</b> をクリックしてプロジェクトを作成します。</li>
	</ol>
2. 次に、 <b>ツール</b>をクリックし、 <b>[NuGet パッケージ マネージャー]</b>をポイントして、 <b>Package Manager Console</b>をクリックします
3.	。 <b>Package Manager Console</b>で、次のコマンドを 1 つずつ実行します</b>。 

		Install-Package Microsoft.Azure.Management.DataFactories -Pre
		Install-Package Microsoft.DataFactories.Runtime -Pre
		Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
6. 次の **appSetttings** セクションを **App.config** ファイルに追加します。これらは、ヘルパー メソッド **GetAuthorizationHeader** によって使用されます。  

	**SubscriptionId** および **ActiveDirectoryTenantId** の値を Azure サブスクリプションとテナント ID に置き換えます。これらの値は、Azure PowerShell から **Get-AzureAccount** を実行して取得できます (まず Add-AzureAccount を使用してログインする必要があります)。
 
		<appSettings>
		    <!--CSM Prod related values-->
		    <add key="ActiveDirectoryEndpoint" value="https://login.windows.net/" />
		    <add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
		    <add key="WindowsManagementUri" value="https://management.core.windows.net/" />
		    <add key="AdfClientId" value="1950a258-227b-4e31-a9cf-717495945fc2" />
		    <add key="RedirectUri" value="urn:ietf:wg:oauth:2.0:oob" />
		    <!--Make sure to write your own tenenat id and subscription ID here-->
		    <add key="SubscriptionId" value="49fb6e5f-3098-4fb2-ba2f-6d6eed843a65" />
    		<add key="ActiveDirectoryTenantId" value="37330244-7828-4a28-99b7-c8c3a437c7ac" />
		</appSettings>
6. 次の **using** ステートメントをプロジェクト内のソース ファイル (Program.cs) に追加します。

		using System.Threading;
		using System.Configuration;
		using System.Collections.ObjectModel;
				
		using Microsoft.Azure.Management.DataFactories;
		using Microsoft.Azure.Management.DataFactories.Models;
		using Microsoft.IdentityModel.Clients.ActiveDirectory;
		using Microsoft.WindowsAzure; 
6. **DataPipelineManagementClient** クラスのインスタンスを作成する次のコードを **Main** メソッドに追加します。このオブジェクトを使用して、Data Factory、リンクされたサービス、入力テーブルと出力テーブル、パイプラインを作成します。また、実行時にテーブルのスライスも監視します。    

        // create data pipeline management client
        string resourceGroupName = "ADF";
        string dataFactoryName = "APITutorialFactory";

        TokenCloudCredentials aadTokenCredentials =
            new TokenCloudCredentials(
                ConfigurationManager.AppSettings["SubscriptionId"],
                GetAuthorizationHeader());

        Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

        DataPipelineManagementClient client = new DataPipelineManagementClient(aadTokenCredentials, resourceManagerUri);
7. Add the following code that creates a **data factory** to the **Main** method.

        // create a data factory
        Console.WriteLine("Creating a data factory");
        client.DataFactories.CreateOrUpdate(resourceGroupName,
            new DataFactoryCreateOrUpdateParameters()
            {
                DataFactory = new DataFactory()
                {
                    Name = dataFactoryName,
                    Location = "westus",
                    Properties = new DataFactoryProperties() { }
                }
            }
        );
8. **linked service** を作成する次のコードを **Main** メソッドに追加します。 
	> [WACOM.NOTE] **ConnectionString** には、Azure ストレージ アカウントの**アカウント名**と**アカウント キー**を使用します。 

		// create a linked service
        Console.WriteLine("Creating a linked service");
        client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
            new LinkedServiceCreateOrUpdateParameters()
            {
                LinkedService = new LinkedService()
                {
                    Name = "LinkedService-AzureStorage",
                    Properties = new AzureStorageLinkedService()
                    {
                        ConnectionString = "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=account key",
                    }
                }
            }
        );
9. **入力テーブルと出力テーブル**を作成する次のコードを **Main** メソッドに追加します。 

	入力 BLOB の **FolderPath** を **adftutorial/** に設定します。この **adftutorial** は BLOB ストレージ内のコンテナーの名前です。Azure BLOB ストレージにこのコンテナーが存在しない場合は、**adftutorial** という名前のコンテナーを作成し、このコンテナーにテキスト ファイルをアップロードします。
	
	出力 BLOB の FolderPath を **adftutorial/apifactoryoutput/{Slice}** に設定します。この **Slice** は **SliceStart** (各スライスの開始日時) の値に基づいて動的に計算されます。  

 
        // create input and output tables
        Console.WriteLine("Creating input and output tables");
        string Table_Source = "TableBlobSource";
        string Table_Destination = "TableBlobDestination";

        client.Tables.CreateOrUpdate(resourceGroupName, dataFactoryName,
            new TableCreateOrUpdateParameters()
            {
                Table = new Table()
                {
                    Name = Table_Source,
                    Properties = new TableProperties()
                    {
                        Location = new AzureBlobLocation()
                        {
                            FolderPath = "adftutorial/",
                            LinkedServiceName = "LinkedService-AzureStorage",
                        },

                        Availability = new Availability()
                        {
                            Frequency = SchedulePeriod.Hour,
                            Interval = 1,
                            WaitOnExternal = new WaitOnExternal()
                        },

                        Policy = new Policy()
                        {
                            Validation = new ValidationPolicy()
                            {
                                MinimumRows = 1
                            }
                        }
                    }
                }
            });

        client.Tables.CreateOrUpdate(resourceGroupName, dataFactoryName,
            new TableCreateOrUpdateParameters()
            {
                Table = new Table()
                {
                    Name = Table_Destination,
                    Properties = new TableProperties()
                    {
                        Location = new AzureBlobLocation()
                        {
                            FolderPath = "adftutorial/apifactoryoutput/{Slice}",
                            PartitionedBy = new Collection<Partition>()
                            {
                                new Partition()
                                {
                                    Name = "Slice",
                                    Value = new DateTimePartitionValue()
                                    {
                                        Date = "SliceStart",
                                        Format = "yyyyMMdd-HH"
                                    }
                                }
                            },
                            LinkedServiceName = "LinkedService-AzureStorage",
                        },

                        Availability = new Availability()
                        {
                            Frequency = SchedulePeriod.Hour,
                            Interval = 1,
                        },
                    }
                }
            });
10. **パイプラインを作成してアクティブにする**次のコードを **Main** メソッドに追加します。このパイプラインには、ソースとして **BlobSource**、シンクとして **BlobSink** を使用する **CopyActivity** があります。 

        // create a pipeline
        Console.WriteLine("Creating a pipeline");
        DateTime PipelineActivePeriodStartTime = new DateTime(2014, 8, 9, 0, 0, 0, 0, DateTimeKind.Utc);
        DateTime PipelineActivePeriodEndTime = PipelineActivePeriodStartTime.AddMinutes(60);
        string PipelineName = "PipelineBlobSample";

        client.Pipelines.CreateOrUpdate(resourceGroupName, dataFactoryName,
            new PipelineCreateOrUpdateParameters()
            {
                Pipeline = new Pipeline()
                {
                    Name = PipelineName,
                    Properties = new PipelineProperties()
                    {
                        Description = "Demo Pipeline for data transfer between blobs",

                        // Initial value for pipeline's active period. With this, you won't need to set slice status
                        Start = PipelineActivePeriodStartTime,
                        End = PipelineActivePeriodEndTime,

                        Activities = new List<BaseActivity>()
                        {
                            new CopyActivity()
                            {
                                Name = "BlobToBlob",
                                Inputs = new List<ActivityInput>()
                                {
                                    new ActivityInput()
                                    {
                                        Name = Table_Source,
                                    }
                                },
                        
                                Outputs = new List<ActivityOutput>()
                                {
                                    new ActivityOutput()
                                    {
                                        Name = Table_Destination, 
                                    }
                                },
                     
                                Transformation = new CopyActivityProperties()
                                {
                                    Source = new BlobSource()
                                    {
                                        BlobColumnSeparators = ",",
                                    },
                            
                                    Sink = new BlobSink()
                                    {
                                        WriteBatchSize = 10000,
                                        WriteBatchTimeout = TimeSpan.FromMinutes(10)
                                    },

                                },

                                Policy = new ActivityPolicy()
                                {
                                    ExecutionPriorityOrder = ExecutionPriorityOrder.NewestFirst,
                                    Concurrency = 1,
                                    Retry = 2,
                                    Timeout = TimeSpan.FromMinutes(10),
                                }
                            }

                        },
                    }
                }
            });

11. **Main** メソッドで使用される 次のヘルパー メソッドを **Program** クラスに追加します。このメソッドは、Azure ポータルへのログインに使用する**ユーザー名**と**パスワード**の入力が可能なダイアログ ボックスを表示します。 
 
		public static string GetAuthorizationHeader()
        {
            AuthenticationResult result = null;
            var thread = new Thread(() =>
            {
                try
                {
                    var context = new AuthenticationContext(ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] + ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);

                    result = context.AcquireToken(
                        resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
                        clientId: ConfigurationManager.AppSettings["AdfClientId"],
                        redirectUri: new Uri(ConfigurationManager.AppSettings["RedirectUri"]),
                        promptBehavior: PromptBehavior.Always);
                }
                catch (Exception threadEx)
                {
                    Console.WriteLine(threadEx.Message);
                }
            });

            thread.SetApartmentState(ApartmentState.STA);
            thread.Name = "AcquireTokenThread";
            thread.Start();
            thread.Join();

            if (result != null)
            {
                return result.AccessToken;
            }

            throw new InvalidOperationException("Failed to acquire token");
        }  
 
13. 次のコードを **Main** メソッドに追加して、出力テーブルのデータ スライスのステータスを取得します。この例で予想されるスライスのみが存在します。   
 
        // Pulling status within a timeout threshold
        DateTime start = DateTime.Now;
        bool done = false;

        while (DateTime.Now - start < TimeSpan.FromMinutes(5) && !done)
        {
            Console.WriteLine("Pulling the slice status");
            // wait before the next status check
            Thread.Sleep(1000 * 12);

            var datalistResponse = client.DataSlices.List(resourceGroupName, dataFactoryName, Table_Destination,
                PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString(),
                PipelineActivePeriodEndTime.ConvertToISO8601DateTimeString());

            foreach (DataSlice slice in datalistResponse.DataSlices)
            {
                if (slice.Status == DataSliceStatus.Failed || slice.Status == DataSliceStatus.Ready)
                {
                    Console.WriteLine("Slice execution is done with status: {0}", slice.Status);
                    done = true;
                    break;
                }
                else
                {
                    Console.WriteLine("Slice status is: {0}", slice.Status);
                }
            }
        }

14. データ スライスの実行の詳細を取得するために次のコードを **Main** メソッドに追加します。

        Console.WriteLine("Getting run details of a data slice");

        var datasliceRunListResponse = client.DataSliceRuns.List(resourceGroupName, dataFactoryName, Table_Destination,
                PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString());

        foreach (DataSliceRun run in datasliceRunListResponse.DataSliceRuns)
        {
            Console.WriteLine("Status: \t\t{0}", run.Status);
            Console.WriteLine("DataSliceStart: \t{0}", run.DataSliceStart);
            Console.WriteLine("DataSliceEnd: \t\t{0}", run.DataSliceEnd);
            Console.WriteLine("ActivityId: \t\t{0}", run.ActivityName);
            Console.WriteLine("ProcessingStartTime: \t{0}", run.ProcessingStartTime);
            Console.WriteLine("ProcessingEndTime: \t{0}", run.ProcessingEndTime);
            Console.WriteLine("ErrorMessage: \t{0}", run.ErrorMessage);
        }

        Console.WriteLine("\nPress any key to exit.");
        Console.ReadKey();
    }

15. コンソール アプリケーションをビルドします。[メニュー] から **[ビルド]** をクリックし、**[ソリューションのビルド]** をクリックします。
16. Azure BLOB ストレージ内の adftutorial コンテナーに少なくとも 1 つのファイルが存在することを確認します。存在しない場合は、以下の内容を記述した Emp.txt ファイルをメモ帳で作成し、これを adftutorial コンテナーにアップロードします。

        John, Doe
		Jane, Doe
	 
17. メニューの **[デバッグ]**、**[デバッグの開始]** の順にクリックして、サンプルを実行します。
18. Azure プレビュー ポータルを使用して、Data Factory の **APITutorialFactory** が次のアーティファクトで作成されることを確認します。 
	- リンクされたサービス:**LinkedService_AzureStorage** 
	- Tables:**TableBlobSource** および **TableBlobDestination**.
	- パイプライン:**PipelineBlobSample** 
18. **adftutorial** コンテナーの **apifactoryoutput** フォルダーに出力ファイルが作成されることを確認します。


## 関連項目

記事 | 説明
------ | ---------------
[Azure Data Factory の概要][data-factory-introduction] | この記事では、Azure Data Factory のサービス、コンセプト、利用価値、およびサポートするシナリオを紹介します。
[Azure Data Factory を使ってみる][adf-getstarted] | この記事には、Azure BLOB から Azure SQL データベースにデータをコピーする Azure Data Factory サンプルの作成方法についての詳細なチュートリアルが記載されています。
[パイプラインが内部設置型のデータを扱えるようにする][use-onpremises-datasources] | この記事には、内部設置型の SQL Server データベースから Azure BLOB にデータをコピーする方法を説明したチュートリアルが記載されています。
[チュートリアル:Data Factory を使用してログ ファイルの移動と処理を行う][adf-tutorial] | この記事には、Azure Data Factory を使用してログ ファイルのデータを洞察へと変換する現実に近いシナリオの実行方法について、詳細なチュートリアルが記載されています。
[Azure Data Factory パイプラインでカスタム アクティビティを使用する][use-custom-activities] | この記事には、カスタム アクティビティを作成してパイプラインで使用する詳細な手順のチュートリアルが記載されています。
[Azure Data Factory Developer Reference (Azure Data Factory 開発者向けリファレンス)][developer-reference] | この開発者向けリファレンスには、コマンドレット、JSON スクリプト、関数などを対象とした包括的なリファレンスが記載されています。 


[data-factory-introduction]: ../data-factory-introduction
[adf-getstarted]: ../data-factory-get-started
[use-onpremises-datasources]: ../data-factory-use-onpremises-datasources
[adf-tutorial]: ../data-factory-tutorial
[use-custom-activities]: ../data-factory-use-custom-activities
[developer-reference]: http://go.microsoft.com/fwlink/?LinkId=516908
 
[adf-class-library-reference]: http://go.microsoft.com/fwlink/?LinkID=521877
[azure-developer-center]: http://azure.microsoft.com/ja-jp/downloads/

<!--HONumber=35.2-->