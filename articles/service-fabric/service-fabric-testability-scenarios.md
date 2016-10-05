<properties
   pageTitle="混乱とフェールオーバーのテスト | Microsoft Azure"
   description="Service Fabric の混乱テストとフェールオーバー テストのシナリオを利用し、障害を誘導し、サービスの信頼性を検証します。"
   services="service-fabric"
   documentationCenter=".net"
   authors="motanv"
   manager="rsinha"
   editor="toddabel"/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="07/08/2016"
   ms.author="motanv"/>

# Testability のシナリオ
クラウド インフラストラクチャのような大規模な分散システムは、本質的に信頼性の低いものです。Azure Service Fabric を使用すると、開発者は信頼性の低いインフラストラクチャ上で実行できるサービスのコードを記述できます。高品質なサービスのコードを記述するには、開発者はこのような信頼性の低いインフラストラクチャを誘発してサービスの安定性をテストできる必要があります。

Fault Analysis Service を使用すると、開発者は障害アクションを誘発して、障害のある状態でサービスをテストできます。ただし、ターゲットを特定した障害のシミュレーションには限界があります。さらにテストを実行するには、混乱のテストとフェールオーバー テストという Service Fabric のテスト シナリオを使用できます。これらのシナリオでは、長時間にわたり、クラスター全体で、グレースフルとアングレースフルの両方が交互配置された連続した障害がシミュレートされます。テストで障害の発生率と種類を構成したら、C# API または PowerShell を使用してテストを開始し、クラスターとサービスに障害を生成します。

>[AZURE.WARNING] ChaosTestScenario は、弾力性に優れたサービス ベースの混乱で置き換えられています。詳細については、[制御された混乱](service-fabric-controlled-chaos.md)に関する新しい記事をご覧ください。

## 混乱のテスト
混乱のシナリオでは、Service Fabric クラスター全体にわたる障害を生成します。このシナリオには、一般的に数か月間または数年間から数時間で発生するエラーが圧縮されています。障害率の高い交互に配置された障害の組み合わせにより、通常は見過ごされるめったに発生しないケースが検出されます。これにより、サービスのコードの品質が大幅に向上します。

### 混乱のテストでシミュレートされる障害
 - ノードの再起動
 - デプロイされたコード パッケージの再起動
 - レプリカの削除
 - レプリカの再起動
 - プライマリ レプリカの移動 (オプション)
 - セカンダリ レプリカの移動 (オプション)

混乱のテストでは、一定の期間、障害とクラスターの検証が複数回実行されます。クラスターが安定し、検証が成功するまでの時間も設定できます。クラスターの検証で 1 つの障害が検出されると、シナリオは失敗します。

たとえば、1 時間実行して、最大 3 つの障害が同時に発生するように設定したテストを考えてみます。このテストは 3 つの障害を誘発してから、クラスターの正常性を検証します。テストでは、クラスターが異常な状態になるまで、または 1 時間経過するまで、前述の手順が繰り返されます。繰り返しているうちに、クラスターの状態が異常 (つまり、設定時間内に安定した状態にならない) になると、テストは失敗して、例外が発生します。この例外は問題が発生したことを示し、さらに調査が必要になります。

現在の形式では、混乱のテストの障害生成エンジンからは、安全な障害しか誘発されません。これは、外部障害がなければ、クォーラムまたはデータの損失が起こらないことを意味します。

### 重要な構成オプション
 - **TimeToRun**: テストが正常に完了するまでの実行時間の合計。検証エラーにならないようにテストを早めに完了できます。
 - **MaxClusterStabilizationTimeout**: テストが失敗する前に、クラスターが正常になるまで待機する最大時間。実行されるチェックは、クラスターの正常性が問題ないかどうか、サービスの正常性が問題ないかどうか、サービス パーティションの対象となるレプリカ セットのサイズに達しているかどうか、InBuild レプリカが存在しないかどうかです。
 - **MaxConcurrentFaults**: 各イテレーションで誘発される同時実行のエラーの最大数。数値が大きいほど、より積極的なテストとなり、結果的に、より複雑なフェールオーバーと遷移の組み合わせになります。このテストは、外部エラーが存在しない場合、この構成がどれだけ高い値であったとしても、クォーラムまたはデータの損失が発生しないことを保証します。
 - **EnableMoveReplicaFaults**: プライマリまたはセカンダリ レプリカの移動を誘発する障害を有効または無効にします。このようなエラーは、既定で無効になっています。
 - **WaitTimeBetweenIterations**: イテレーション間 (つまり、障害とそれに対応する検証が一巡した後) の待機時間。

### 混乱のテストを実行する方法
C# のサンプル

```csharp
using System;
using System.Fabric;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        string clusterConnection = "localhost:19000";

        Console.WriteLine("Starting Chaos Test Scenario...");
        try
        {
            RunChaosTestScenarioAsync(clusterConnection).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Chaos Test Scenario did not complete: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Chaos Test Scenario completed.");
        return 0;
    }

    static async Task RunChaosTestScenarioAsync(string clusterConnection)
    {
        TimeSpan maxClusterStabilizationTimeout = TimeSpan.FromSeconds(180);
        uint maxConcurrentFaults = 3;
        bool enableMoveReplicaFaults = true;

        // Create FabricClient with connection and security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);

        // The chaos test scenario should run at least 60 minutes or until it fails.
        TimeSpan timeToRun = TimeSpan.FromMinutes(60);
        ChaosTestScenarioParameters scenarioParameters = new ChaosTestScenarioParameters(
          maxClusterStabilizationTimeout,
          maxConcurrentFaults,
          enableMoveReplicaFaults,
          timeToRun);

        // Other related parameters:
        // Pause between two iterations for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenIterations = TimeSpan.FromSeconds(30);
        // Pause between concurrent actions for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenFaults = TimeSpan.FromSeconds(10);

        // Create the scenario class and execute it asynchronously.
        ChaosTestScenario chaosScenario = new ChaosTestScenario(fabricClient, scenarioParameters);

        try
        {
            await chaosScenario.ExecuteAsync(CancellationToken.None);
        }
        catch (AggregateException ae)
        {
            throw ae.InnerException;
        }
    }
}
```

PowerShell

```powershell
$connection = "localhost:19000"
$timeToRun = 60
$maxStabilizationTimeSecs = 180
$concurrentFaults = 3
$waitTimeBetweenIterationsSec = 60

Connect-ServiceFabricCluster $connection

Invoke-ServiceFabricChaosTestScenario -TimeToRunMinute $timeToRun -MaxClusterStabilizationTimeoutSec $maxStabilizationTimeSecs -MaxConcurrentFaults $concurrentFaults -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec
```


## フェールオーバー テスト

フェールオーバー テスト シナリオは、特定のサービス パーティションを対象にした、混乱のテスト シナリオの 1 つのバージョンです。他のサービスには影響が及ばない状態で、特定のサービス パーティションに対するフェールオーバーの影響をテストします。ターゲット パーティションの情報とその他のパラメーターを使用して構成されると、クライアント側ツールとして実行します。このツールでは C# API または PowerShell を使用してサービス パーティションの障害を生成します。このシナリオでは、ワークロードが発生するビジネス ロジックを実行すると同時に、一連のシミュレートされた障害とサービス検証を反復処理します。サービス検証の失敗は、さらに調査の必要な問題があることを示します。

### フェールオーバー テストでシミュレートされる障害
- パーティションがホストされているデプロイ済みコード パッケージを再起動します。
- プライマリ/セカンダリ レプリカまたはステートレス インスタンスを削除します。
- プライマリ/セカンダリ レプリカを再起動します (永続化されたサービスの場合)。
- プライマリ レプリカを移動します。
- セカンダリ レプリカを移動します。
- パーティションを再起動します。

フェールオーバー テストでは、選択した障害を誘発し、サービスの検証を実行してサービスの安定性を確認します。混乱のテストでは複数の障害が誘発される可能性があるのに対し、フェールオーバー テストで誘発される障害は一度に 1 つだけです。各障害が発生した後、構成したタイムアウト時間内にサービス パーティションが安定しない場合、テストは失敗します。テストで誘発されるのは安全な障害のみです。つまり、外部障害がなければ、クォーラムまたはデータの損失が起こらないことを意味します。

### 重要な構成オプション
 - **PartitionSelector**: ターゲットにする必要があるパーティションを指定するセレクター オブジェクト。
 - **TimeToRun**: テストが完了するまでの実行時間の合計。
 - **MaxServiceStabilizationTimeout**: テストが失敗する前に、クラスターが正常になるまで待機する最大時間。実行されるチェックは、サービスの正常性が問題ないかどうか、すべてのパーティションの対象となるレプリカ セットのサイズに達しているかどうか、InBuild レプリカが存在しないかどうかです。
 - **WaitTimeBetweenFaults**: 障害と検証のサイクル間の待機時間。

### フェールオーバーのテストを実行する方法

**C#**

```csharp
using System;
using System.Fabric;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        string clusterConnection = "localhost:19000";
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");

        Console.WriteLine("Starting Chaos Test Scenario...");
        try
        {
            RunFailoverTestScenarioAsync(clusterConnection, serviceName).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Chaos Test Scenario did not complete: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Chaos Test Scenario completed.");
        return 0;
    }

    static async Task RunFailoverTestScenarioAsync(string clusterConnection, Uri serviceName)
    {
        TimeSpan maxServiceStabilizationTimeout = TimeSpan.FromSeconds(180);
        PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);

        // Create FabricClient with connection and security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);

        // The chaos test scenario should run at least 60 minutes or until it fails.
        TimeSpan timeToRun = TimeSpan.FromMinutes(60);
        FailoverTestScenarioParameters scenarioParameters = new FailoverTestScenarioParameters(
          randomPartitionSelector,
          timeToRun,
          maxServiceStabilizationTimeout);

        // Other related parameters:
        // Pause between two iterations for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenIterations = TimeSpan.FromSeconds(30);
        // Pause between concurrent actions for a random duration bound by this value.
        // scenarioParameters.WaitTimeBetweenFaults = TimeSpan.FromSeconds(10);

        // Create the scenario class and execute it asynchronously.
        FailoverTestScenario failoverScenario = new FailoverTestScenario(fabricClient, scenarioParameters);

        try
        {
            await failoverScenario.ExecuteAsync(CancellationToken.None);
        }
        catch (AggregateException ae)
        {
            throw ae.InnerException;
        }
    }
}
```


**PowerShell**

```powershell
$connection = "localhost:19000"
$timeToRun = 60
$maxStabilizationTimeSecs = 180
$waitTimeBetweenFaultsSec = 10
$serviceName = "fabric:/SampleApp/SampleService"

Connect-ServiceFabricCluster $connection

Invoke-ServiceFabricFailoverTestScenario -TimeToRunMinute $timeToRun -MaxServiceStabilizationTimeoutSec $maxStabilizationTimeSecs -WaitTimeBetweenFaultsSec $waitTimeBetweenFaultsSec -ServiceName $serviceName -PartitionKindSingleton
```

<!---HONumber=AcomDC_0921_2016-->