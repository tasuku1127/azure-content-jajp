<properties 
	pageTitle="Mac OS X 上で Tedious を含む Node.js を使用して SQL Database に接続する" 
	description="Azure SQL Database への接続に使用できる Node.js コード サンプルについて説明します。サンプルは、Tedious ドライバーを使用して接続します。"
	services="sql-database" 
	documentationCenter="" 
	authors="meet-bhagdev" 
	manager="jeffreyg" 
	editor=""/>


<tags 
	ms.service="sql-database" 
	ms.workload="data-management" 
	ms.tgt_pltfrm="na" 
	ms.devlang="nodejs" 
	ms.topic="article" 
	ms.date="10/20/2015" 
	ms.author="meetb"/>


# Mac OS X 上で Tedious を含む Node.js を使用して SQL Database に接続する


[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]


このトピックでは、Mac OS X 上で実行する Node.js のコード サンプルを示します。このサンプルは、Tedious ドライバーを使用して、Azure SQL Database に接続します。


## 必要なソフトウェア アイテム


既にコンピューターにインストールされている場合を除いて、**node** をインストールします。


OSX 10.10 Yosemite に node.js をインストールするには、簡単にインストールができる、便利な事前コンパイルされたバイナリ パッケージをダウンロードします。[nodejs.org に進み](http://nodejs.org/)、インストール ボタンをクリックして最新のパッケージをダウンロードします。

**node** と **npm** の両方のインストールを行うインストール ウィザードに従い、.dmg からパッケージをインストールします (npm は、node.js 用にその他のパッケージのインストールを容易にするノード パッケージ マネージャーです)。


**ノード** と **npm** でコンピューターを構成したら、Node.js プロジェクトの作成を予定しているディレクトリに移動して、次のコマンドを入力します。


	npm init
	npm install tedious


**npm init** はノード プロジェクトを作成します。プロジェクトの作成中に既定値を保持するには、プロジェクトが作成されるまで Enter キーを押します。プロジェクト ディレクトリに **package.json** が表示されます。


### AdventureWorks データベースの作成


このトピックのコード サンプルには、**AdventureWorks** テスト データベースが必要です。まだお持ちでない場合は、「[SQL Database の使用](sql-database-get-started.md)」をご覧ください。ガイドに従って、**AdventureWorks データベースのテンプレート**を作成することが重要です。以下に示す例は、**AdventureWorks スキーマ** とのみ動作します。


## SQL Database に接続する

[新しい Connection ](http://pekim.github.io/tedious/api-connection.html)関数は、SQL Database に接続するために使用します。

	var Connection = require('tedious').Connection;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.windows.net',
		// If you are on Microsoft Azure, you need this:
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
	// If no error, then good to proceed.
		console.log("Connected");
	});


## SQL SELECT の実行


[新しい Request()](http://pekim.github.io/tedious/api-request.html) 関数 を使用して、すべての SQL ステートメントが実行されます。ステートメントが SELECT ステートメントなどの行を返す場合は、[request.on()](http://pekim.github.io/tedious/api-request.html) 関数を使用してそれらを取得することができます。行が存在しない場合は、[request.on()](http://pekim.github.io/tedious/api-request.html) 関数は空のリストを返します。


	var Connection = require('tedious').Connection;
	var Request = require('tedious').Request;
	var TYPES = require('tedious').TYPES;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.windows.net',
		// When you connect to Azure SQL Database, you need these next options.
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
		// If no error, then good to proceed.
		console.log("Connected");
		executeStatement();
	});
	
	
	function executeStatement() {
		request = new Request("SELECT c.CustomerID, c.CompanyName,COUNT(soh.SalesOrderID) AS OrderCount FROM SalesLT.Customer AS c LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID GROUP BY c.CustomerID, c.CompanyName ORDER BY OrderCount DESC;", function(err) {
	  	if (err) {
	   		console.log(err);} 
		});
		var result = "";
		request.on('row', function(columns) {
		    columns.forEach(function(column) {
		      if (column.value === null) {
		        console.log('NULL');
		      } else {
		        result+= column.value + " ";
		      }
		    });
		    console.log(result);
		    result ="";
		});
	
		request.on('done', function(rowCount, more) {
		console.log(rowCount + ' rows returned');
		});
		connection.execSql(request);
	}


## 行を挿入し、パラメーターを適用し、生成されたプライマリ キーを取得する


SQL Database では、[IDENTITY](https://msdn.microsoft.com/library/ms186775.aspx) プロパティと [SEQUENCE](https://msdn.microsoft.com/library/ff878058.aspx) オブジェクトを使用して、[プライマリ キーの値](https://msdn.microsoft.com/library/ms179610.aspx)を自動生成できます。この例では、INSERT ステートメントを実行し、SQL インジェクションから保護されているパラメーターを安全に渡し、自動生成されたプライマリ キー値を取得する方法について説明しています。


このセクションのコード サンプルは、SQL の INSERT ステートメントにパラメーターを適用します。生成されるプライマリ キーの値は、プログラムによって取得されます。


	var Connection = require('tedious').Connection;
	var Request = require('tedious').Request
	var TYPES = require('tedious').TYPES;
	var config = {
		userName: 'yourusername',
		password: 'yourpassword',
		server: 'yourserver.database.windows.net',
		// If you are on Azure SQL Database, you need these next options.
		options: {encrypt: true, database: 'AdventureWorks'}
	};
	var connection = new Connection(config);
	connection.on('connect', function(err) {
		// If no error, then good to proceed.
		console.log("Connected");
		executeStatement1();
	});
	
	
	function executeStatement1() {
		request = new Request("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES (@Name, @Number, @Cost, @Price, CURRENT_TIMESTAMP);", function(err) {
		 if (err) {
		 	console.log(err);} 
		});
		request.addParameter('Name', TYPES.NVarChar,'SQL Server Express 2014');
		request.addParameter('Number', TYPES.NVarChar , 'SQLEXPRESS2014');
		request.addParameter('Cost', TYPES.Int, 11);
		request.addParameter('Price', TYPES.Int,11);
		request.on('row', function(columns) {
		    columns.forEach(function(column) {
		      if (column.value === null) {
		        console.log('NULL');
		      } else {
		        console.log("Product id of inserted item is " + column.value);
		      }
		    });
		});		
		connection.execSql(request);
	}

<!---HONumber=Oct15_HO4-->