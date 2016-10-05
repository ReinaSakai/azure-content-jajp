<properties
   pageTitle="データ ソースへの接続方法 | Microsoft Azure"
   description="Azure Data Catalog で検出されたデータ ソースへの接続方法を説明する操作方法に関する記事"
   services="data-catalog"
   documentationCenter=""
   authors="steelanddata"
   manager="NA"
   editor=""
   tags=""/>
<tags
   ms.service="data-catalog"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-catalog"
   ms.date="09/15/2016"
   ms.author="maroche"/>


# データ ソースへの接続方法

## はじめに
**Microsoft Azure Data Catalog** は、完全に管理されたクラウド サービスであり、エンタープライズ データ ソースの登録のシステムと検出のシステムとして機能します。つまり、**Azure Data Catalog** を使用すると、ユーザーはデータ ソースを検出、理解、使用でき、組織は既存のデータからより多くの価値を引き出すことができます。このシナリオの主な特徴はデータを使用することです。ユーザーがデータ ソースを検出しその目的を理解したら、次の手順としてデータ ソースに接続し、データを使用します。

## データ ソースの場所
データ ソースの登録時に、**Azure Data Catalog** は、データ ソースに関するメタデータを受信します。このメタデータには、データ ソースの場所に関する詳細が含まれています。場所の詳細はデータ ソースによって異なりますが、それには必ず接続に必要な情報が含まれています。たとえば、SQL Server テーブルの場所には、サーバー名、データベース名、スキーマ名、およびテーブル名が含まれます。一方、SQL Server Reporting Services レポートには、サーバー名とレポートへのパスが含まれます。その他のデータ ソースの種類には、ソース システムの構造と機能を反映する場所が含まれます。

## 統合クライアント ツール
データ ソースに接続する最も簡単な方法は、**Azure Data Catalog** ポータルで「開く」メニューを使用することです。このメニューには、選択したデータ資産に接続するためのオプションの一覧が表示されます。既定のタイル ビューを使用する場合、このメニューは各タイルで確認できます。

 ![データ資産タイルから Excel で SQL Server のテーブルを開く](./media/data-catalog-how-to-connect/data-catalog-how-to-connect1.png)

リスト ビューを使用すると、ポータル ウィンドウの上部にある検索バーにメニューが表示されます。

 ![検索バーからレポート マネージャーで SQL Server Reporting Services レポートを開く](./media/data-catalog-how-to-connect/data-catalog-how-to-connect2.png)

## サポートされているクライアント アプリケーション
Azure Data Catalog ポータルでデータ ソースの [開く] メニューを使用する場合は、適切なクライアント アプリケーションがクライアント コンピューターにインストールされている必要があります。

| 開くアプリケーション | ファイル拡張子/プロトコル | サポートされているアプリケーションのバージョン |
| --- | --- | --- |
| Excel | .odc | Excel 2010 以降 |
| Excel (上位 1000) | .odc | Excel 2010 以降 |
| Power Query | .xlsx | Excel 2016、Power Query for Excel アドインがインストールされている Excel 2010 または Excel 2013
| Power BI Desktop | .pbix | Power BI Desktop (2016 年 7 月以降) |
| SQL Server Data Tools | vsweb:// | SQL Server ツールがインストールされている Visual Studio 2013 Update 4 以降 |
| レポート マネージャー | http:// | [SQL Server Reporting Services のブラウザーの要件](https://technet.microsoft.com/ja-JP/library/ms156511.aspx)に関するページを参照してください。 |

## データ、ツール
メニューで使用できるオプションは、現在選択されているデータ資産の種類によって異なります。もちろん、使用可能なツールがすべて「開く」メニューに表示されるわけではありませんが、それでも任意のクライアント ツールを使用してデータ ソースに簡単に接続できます。**Azure Data Catalog** ポータルでデータ資産を選択すると、プロパティ ペインに完全な場所が表示されます。

 ![SQL Server テーブルの接続情報](./media/data-catalog-how-to-connect/data-catalog-how-to-connect3.png)

接続情報の詳細はデータ ソースの種類によって異なりますが、ポータルに含まれる情報を確認すれば、任意のクライアント ツールでデータ ソースに接続するために必要なすべての情報が得られます。ユーザーは、**Azure Data Catalog** を使用して検出したデータ ソースの接続に関する詳細をコピーすることができるので、適切なツールでデータを操作することができるようになります。

## 接続とデータ ソースのアクセス許可
**Azure Data Catalog** を使用すればデータ ソースの探索は可能ですが、データ自体へのアクセスは引き続きデータ ソースの所有者または管理者の管理下にあります。**Azure Data Catalog** でのデータ ソースの探索で、データ ソース自体へのアクセス許可がユーザーに付与されることはありません。

データ ソースを探索するがそのデータへのアクセス許可を持たないユーザーがデータにアクセスしやすくするために、ユーザーはデータ ソースに注釈を付ける際に、[アクセス権の要求] プロパティに情報を指定することができます。ここで指定した情報 (たとえば、データ ソースへのアクセス権を取得するためのプロセスまたは接続ポイントへのリンク) は、ポータルにデータ ソースの場所情報と共に表示されます。

 ![アクセスの要求に関する指示が含まれた接続情報](./media/data-catalog-how-to-connect/data-catalog-how-to-connect4.png)

##概要
**Azure Data Catalog** にデータ ソースを登録すると、構造メタデータと記述メタデータがデータ ソースから Catalog サービスにコピーされ、データの探索が可能になります。データ ソースが登録され探索されると、ユーザーは **Azure Data Catalog** ポータルの [開く] メニューから、または適切なデータ ツールを使用してデータソースに接続できるようになります。

## 関連項目
- データ ソースに接続する方法の詳細な手順については、「[Azure Data Catalog の概要](data-catalog-get-started.md)」チュートリアルをご覧ください。

<!---HONumber=AcomDC_0921_2016-->