<properties
	pageTitle="Azure Functions F# 開発者向けリファレンス | Microsoft Azure"
	description="F# を使用して Azure Functions を開発する方法について説明します。"
	services="functions"
	documentationCenter="fsharp"
	authors="sylvanc"
	manager="jbronsk"
	editor=""
	tags=""
	keywords="Azure Functions, 機能, イベント処理, Webhook, 動的コンピューティング, サーバーなしのアーキテクチャ, F#"/>

<tags
	ms.service="functions"
	ms.devlang="fsharp"
	ms.topic="reference"
	ms.tgt_pltfrm="multiple"
	ms.workload="na"
	ms.date="09/09/2016"
	ms.author="syclebsc"/>

# Azure Functions F# 開発者向けリファレンス

> [AZURE.SELECTOR]
- [C# スクリプト](../articles/azure-functions/functions-reference-csharp.md)
- [F# スクリプト](../articles/azure-functions/functions-reference-fsharp.md)
- [Node.JS](../articles/azure-functions/functions-reference-node.md)

Azure Functions 用 F# は、小規模なコード ("関数") をクラウドで手軽に実行できるソリューションです。データは関数の引数を通じて F# 関数に渡されます。引数名は `function.json` で指定され、関数のロガーやキャンセル トークンなどにアクセスするための定義済みの名前があります。

この記事では、「[Azure Functions developer reference (Azure Functions 開発者向けリファレンス)](functions-reference.md)」を既に読んでいることを前提としています。

## .fsx のしくみ

`.fsx` ファイルは、F# スクリプトです。1 つのファイルに含まれている F# プロジェクトとして考えることができます。ファイルには、プログラムのためのコード (この場合は、Azure 関数) と、依存関係を管理するためのディレクティブの両方が含まれています。

Azure 関数に対して `.fsx` を使用すると、一般に必要とされるアセンブリが自動的に含められるため、"定型" コードではなく、関数のコードに集中できます。

## 引数へのバインド

「[Azure Functions のトリガーとバインドの開発者用リファレンス](functions-triggers-bindings.md)」で詳しく説明されているように、各バインドはいくつかのセットの引数をサポートしています。たとえば、BLOB トリガーでサポートされている引数バインドの 1 つは、POCO です。これは、F# レコードを使用して表すことができます。次に例を示します。

```fsharp
type Item = { Id: string }

let Run(blob: string, output: byref<Item>) =
    let item = { Id = "Some ID" }
    output <- item
```

F# Azure 関数は、1 つまたは複数の引数を受け取ります。Azure Functions 引数についての説明では、"_入力_" 引数と "_出力_" 引数が出てきます。入力引数は、名前が示すとおり、F# Azure 関数への入力です。"_出力_" 引数は、関数から "_出力_" データを返す方法として機能する、変更可能なデータまたは `byref<>` 引数です。

上の例では、`blob` が入力引数で、`output` が出力引数です。`output` に `byref<>` を使用したことに注意してください (`[<Out>]` という注釈を付ける必要はありません)。`byref<>` 型を使用すると、関数は引数が参照するレコードまたはオブジェクトを変更できます。

F# レコードを入力型として使用する場合は、Azure Functions フレームワークがレコードを関数に渡す前にフィールドを適切に設定できるように、レコード定義を `[<CLIMutable>]` でマークする必要があります。内部的には、`[<CLIMutable>]` によってレコードのプロパティのセッターが生成されます。次に例を示します。

```fsharp
[<CLIMutable>]
type TestObject =
    { SenderName : string
      Greeting : string }

let Run(req: TestObject, log: TraceWriter) =
    { req with Greeting = sprintf "Hello, %s" req.SenderName }
```

F# クラスも、入力と出力の両方の引数で使用できます。クラスの場合、プロパティには通常、ゲッターとセッターが必要です。次に例を示します。

```fsharp
type Item() =
    member val Id = "" with get,set
    member val Text = "" with get,set

let Run(input: string, item: byref<Item>) =
    let result = Item(Id = input, Text = "Hello from F#!")
    item <- result
```

## ログの記録

出力を F# の[ストリーミング ログ](../app-service-web/web-sites-streaming-logs-and-console.md)にログ記録するには、関数が型 `TraceWriter` の引数を受け取る必要があります。一貫性のために、この引数は `log` という名前にすることをお勧めします。次に例を示します。

```fsharp
let Run(blob: string, output: byref<string>, log: TraceWriter) =
    log.Verbose(sprintf "F# Azure Function processed a blob: %s" blob)
    output <- input
```

## 非同期

`async` ワークフローを使用できますが、結果は `Task` を返す必要があります。そのためには、たとえば `Async.StartAsTask` を使用します。

```fsharp
let Run(req: HttpRequestMessage) =
    async {
        return new HttpResponseMessage(HttpStatusCode.OK)
    } |> Async.StartAsTask
```

## キャンセル トークン

関数が正常なシャットダウンを処理する必要がある場合は、[`CancellationToken`](https://msdn.microsoft.com/library/system.threading.cancellationtoken.aspx) 引数を設定できます。これは、たとえば `async` と結合される場合があります。

```fsharp
let Run(req: HttpRequestMessage, token: CancellationToken)
    let f = async {
        do! Async.Sleep(10)
        return new HttpResponseMessage(HttpStatusCode.OK)
    }
    Async.StartAsTask(f, token)
```

## 名前空間のインポート

名前空間は、通常の方法で開くことができます。

```fsharp
open System.Net
open System.Threading.Tasks

let Run(req: HttpRequestMessage, log: TraceWriter) =
    ...
```

以下の名前空間は、自動的に開かれます。

* `System`
* `System.Collections.Generic`
* `System.IO`
* `System.Linq`
* `System.Net.Http`
* `System.Threading.Tasks`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`

## 外部アセンブリの参照

同様に、フレームワーク アセンブリ参照を、`#r "AssemblyName"` ディレクティブと共に追加できます。

```fsharp
#r "System.Web.Http"

open System.Net
open System.Net.Http
open System.Threading.Tasks

let Run(req: HttpRequestMessage, log: TraceWriter) =
    ...
```

次のアセンブリは、Azure Functions をホストしている環境によって自動的に追加されます。

* `mscorlib`,
* `System`
* `System.Core`
* `System.Xml`
* `System.Net.Http`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`
* `Microsoft.Azure.WebJobs.Extensions`
* `System.Web.Http`
* `System.Net.Http.Formatting`

さらに、次のアセンブリは特別扱いされ、simplename によって参照される場合があります (例: `#r "AssemblyName"`)。

* `Newtonsoft.Json`
* `Microsoft.WindowsAzure.Storage`
* `Microsoft.ServiceBus`
* `Microsoft.AspNet.WebHooks.Receivers`
* `Microsoft.AspNEt.WebHooks.Common`

プライベート アセンブリを参照する必要がある場合は、アセンブリ ファイルを関数に関連する `bin` フォルダーにアップロードし、ファイル名 (例: `#r "MyAssembly.dll"`) を使用して参照できます。関数フォルダーにファイルをアップロードする方法については、パッケージ管理の次のセクションを参照してください。

## エディター準備

F# Compiler Services をサポートしているエディターは、Azure Functions が自動的に含める名前空間とアセンブリを認識しません。そのため、使用されているアセンブリを検索するエディターを支援するための準備を行い、明示的に名前空間を開くと、便利な場合があります。次に例を示します。

```fsharp
#if !COMPILED
#I "../../bin/Binaries/WebJobs.Script.Host"
#r "Microsoft.Azure.WebJobs.Host.dll"
#endif

open Sytem
open Microsoft.Azure.WebJobs.Host

let Run(blob: string, output: byref<string>, log: TraceWriter) =
    ...
```

Azure Functions は、コードを実行するとき、`COMPILED` が定義されているソースを処理するため、エディター準備は無視されます。

## パッケージの管理

NuGet パッケージを F# 関数で使用するには、`project.json` ファイルを関数アプリのファイル システムにある関数のフォルダーに追加します。次に示すのは、`Microsoft.ProjectOxford.Face` Version 1.1.0 への NuGet パッケージ参照を追加する、`project.json` ファイルの例です。

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.ProjectOxford.Face": "1.1.0"
      }
    }
   }
}
```

.NET Framework 4.6 のみがサポートされているので、次に示すように `project.json` ファイルが `net46` を指定していることを確認します。

`project.json` ファイルをアップロードすると、ランタイムによってパッケージが取得され、パッケージ アセンブリへの参照が自動的に追加されます。`#r "AssemblyName"` ディレクティブを追加する必要はありません。単に、必要な `open` ステートメントを `.fsx` ファイルに追加します。

エディターと F# Compile Services との相互作用を向上させるために、エディター準備に参照アセンブリを自動的に配置したい場合があるかもしれません。

### `project.json` ファイルを Azure 関数に追加する方法

1. Azure ポータルで関数を開き、関数アプリが実行中であることを確認して開始します。これにより、パッケージのインストール出力が表示されるストリーミング ログへのアクセス権が付与されます。

2. `project.json` ファイルをアップロードするには、「[関数アプリ ファイルを更新する方法](functions-reference.md#fileupdate)」で説明されているいずれかの方法を使用します。[Azure Functions の継続的なデプロイ](functions-continuous-deployment.md)を使用している場合は、`project.json` ファイルをステージング分岐に追加することができます。そうすることで、デプロイ分岐に追加する前にファイルを実験することができます。

3. `project.json` ファイルが追加された後、関数のストリーミング ログの出力は次の例のようになります。

```
2016-04-04T19:02:48.745 Restoring packages.
2016-04-04T19:02:48.745 Starting NuGet restore
2016-04-04T19:02:50.183 MSBuild auto-detection: using msbuild version '14.0' from 'D:\Program Files (x86)\MSBuild\14.0\bin'.
2016-04-04T19:02:50.261 Feeds used:
2016-04-04T19:02:50.261 C:\DWASFiles\Sites\facavalfunctest\LocalAppData\NuGet\Cache
2016-04-04T19:02:50.261 https://api.nuget.org/v3/index.json
2016-04-04T19:02:50.261
2016-04-04T19:02:50.511 Restoring packages for D:\home\site\wwwroot\HttpTriggerCSharp1\Project.json...
2016-04-04T19:02:52.800 Installing Newtonsoft.Json 6.0.8.
2016-04-04T19:02:52.800 Installing Microsoft.ProjectOxford.Face 1.1.0.
2016-04-04T19:02:57.095 All packages are compatible with .NETFramework,Version=v4.6.
2016-04-04T19:02:57.189
2016-04-04T19:02:57.189
2016-04-04T19:02:57.455 Packages restored.
```

## 環境変数

環境変数またはアプリ設定値を取得するには、次の例のように、`System.Environment.GetEnvironmentVariable` を使用します。

```fsharp
open System.Environment

let Run(timer: TimerInfo, log: TraceWriter) =
    log.Info("Storage = " + GetEnvironmentVariable("AzureWebJobsStorage"))
    log.Info("Site = " + GetEnvironmentVariable("WEBSITE_SITE_NAME"))
```

## .fsx コードの再利用

他の `.fsx` ファイルのコードを利用するには、`#load` ディレクティブを使用します。次に例を示します。

`run.fsx`

```fsharp
#load "logger.fsx"

let Run(timer: TimerInfo, log: TraceWriter) =
    mylog log (sprintf "Timer: %s" DateTime.Now.ToString())
```

`logger.fsx`

```fsharp
let mylog(log: TraceWriter, text: string) =
    log.Verbose(text);
```

`#load` ディレクティブに指定するパスは、`.fsx` ファイルの場所からの相対パスです。

* `#load "logger.fsx"` によって、関数フォルダーにあるファイルが読み込まれます。

* `#load "package\logger.fsx"` によって、関数フォルダー内の `package` フォルダーにあるファイルが読み込まれます。

* `#load "..\shared\mylogger.fsx"` によって、関数フォルダーと同じレベル (`wwwroot` の直下) にある `shared` フォルダーのファイルが読み込まれます。

`#load` ディレクティブは、`.fsx` (F# スクリプト) ファイルだけで動作します。`.fs` ファイルでは動作しません。

## 次のステップ

詳細については、次のリソースを参照してください。

* [F# Guide (F# ガイド)](https://docs.microsoft.com/ja-JP/dotnet/articles/fsharp/index)
* [Azure Functions developer reference (Azure Functions 開発者向けリファレンス)](functions-reference.md)
* [Azure Functions C# developer reference (Azure Functions C# 開発者向けリファレンス)](functions-reference-csharp.md)
* [Azure Functions NodeJS 開発者向けリファレンス](functions-reference-node.md)
* [Azure Functions triggers and bindings (Azure Functions のトリガーとバインド)](functions-triggers-bindings.md)
* [Azure Functions のテスト](functions-test-a-function.md)
* [Azure Functions のスケーリング方法](functions-scale.md)

<!---HONumber=AcomDC_0921_2016-->