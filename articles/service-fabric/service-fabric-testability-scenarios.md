---
title: Créer des tests de chaos et de basculement pour Azure Service Fabric
description: Utilisation des scénarios de test chaos et de test de basculement Service Fabric pour induire des erreurs et vérifier la fiabilité de vos services.
author: motanv
ms.topic: conceptual
ms.date: 10/1/2019
ms.author: motanv
ms.custom: devx-track-csharp
ms.openlocfilehash: 17f0af8c041042ddceccc1f8701e44ab8522840f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89022120"
---
# <a name="testability-scenarios"></a>Scénarios de testabilité
Les grands systèmes distribués, comme les infrastructures cloud, sont par définition peu fiables. Grâce à Azure Service Fabric, les développeurs sont en mesure d’écrire des services s’exécutant sur ces infrastructures. Pour écrire des services de haute qualité, les développeurs doivent pouvoir introduire de tels défauts de fiabilité, et ainsi tester la fiabilité des solutions qu’ils conçoivent.

Le service d’analyse des erreurs permet aux développeurs de provoquer des actions erronées afin de tester les services en présence de défaillances. Toutefois, les erreurs simulées ciblées présentent une efficacité limitée. Grâce aux scénarios de test prédéfinis de Service Fabric, vous pouvez réaliser un test de chaos et un test de basculement. Au sein du cluster, ces scénarios simulent des erreurs entrelacées en continu, avec et sans perte de données, sur des périodes prolongées. Une fois qu’un test est configuré avec la fréquence et le type des erreurs, il peut être démarré à l’aide des API C# ou de PowerShell pour générer des erreurs au sein du cluster et de votre service.

> [!WARNING]
> ChaosTestScenario est remplacé par un chaos plus robuste, basé sur le service. Pour plus d’informations, consultez le nouvel article [Chaos contrôlé](service-fabric-controlled-chaos.md) .
> 
> 

## <a name="chaos-test"></a>Test chaos
Le scénario chaos génère des erreurs dans l’ensemble du cluster Service Fabric. Le scénario compresse les erreurs habituellement étalées sur plusieurs mois voire années en quelques heures. L’utilisation d’erreurs entrelacées avec un taux élevé d’erreurs permet d’identifier des dysfonctionnements qui n’auraient pu être isolés autrement. Il en résulte une amélioration significative de la qualité du code du service.

### <a name="faults-simulated-in-the-chaos-test"></a>Erreurs simulées dans le test chaos
* Redémarrer un nœud
* Redémarrer un package de code déployé
* Supprimer un réplica
* Redémarrer un réplica
* Déplacer un réplica principal (principal)
* Déplacer un réplica secondaire (facultatif)

Le test chaos exécute de multiples itérations des validations de cluster et des erreurs pendant la période considérée. Les délais de stabilisation du cluster et de validation sont également configurables. Le scénario échoue à la première défaillance d’une validation de cluster.

Par exemple, supposons qu’un test soit défini pour s’exécuter une heure, avec un maximum de trois erreurs simultanées. Le test introduira trois erreurs, puis validera l’intégrité du cluster. Le test itérera l’étape précédente jusqu’à ce que le cluster présente un défaut d’intégrité ou après une heure. Si le cluster présente un défaut d’intégrité dans une des itérations, auquel cas il n’est pas stabilisé dans le délai configuré, le test échoue avec une exception. Cette exception indique qu’une erreur est survenue et qu’un examen approfondi est nécessaire.

Dans sa forme actuelle, le moteur de génération d’erreurs du test chaos introduit uniquement des erreurs non critiques. Cela signifie qu’en l’absence d’erreurs externes, aucune perte de données ni de quorum ne survient.

### <a name="important-configuration-options"></a>Options de configuration importantes
* **TimeToRun**: durée totale du test jusqu’à sa réussite. Si la validation échoue, le test se termine plus tôt.
* **MaxClusterStabilizationTimeout**: délai maximal nécessaire à la restauration de l’intégrité du cluster, préalablement à l’échec du test. Les contrôles consistent à vérifier que l’intégrité du cluster est acceptable, que la taille cible du jeu de réplicas est atteinte pour l’ensemble des partitions et qu’aucun réplica InBuild n’existe.
* **MaxConcurrentFaults** : nombre maximal d’erreurs simultanées introduites dans chaque itération. Plus le nombre est élevé, plus le test est efficace. Vous obtiendrez des combinaisons plus complexes de basculement et de transition. Le test garantit qu’aucune perte de quorum ou de données ne sera à déplorer en l’absence d’erreurs externes, quel que soit le niveau de la configuration.
* **EnableMoveReplicaFaults**: active ou désactive les erreurs provoquant le déplacement des réplicas primaires ou secondaires. Ces erreurs sont désactivées par défaut.
* **WaitTimeBetweenIterations**: délai d’attente entre les itérations, c’est-à-dire après une séquence d’erreurs et la validation correspondante.

### <a name="how-to-run-the-chaos-test"></a>Procédure d’exécution du test chaos
Exemple de code C#

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

Le module PowerShell Service Fabric comprend deux façons de démarrer un scénario chaos. `Invoke-ServiceFabricChaosTestScenario` est basé sur le client ; si l’ordinateur client est arrêté au milieu du test, aucune autre erreur n’est introduite. En guise d’alternative, il existe un ensemble de commandes destinées à maintenir l’exécution du test en cas d’arrêt de l’ordinateur. `Start-ServiceFabricChaos` utilise un service système avec état et fiable appelé FaultAnalysisService ; ainsi, les erreurs demeurent introduites jusqu’à ce que TimeToRun soit écoulé. `Stop-ServiceFabricChaos` peut être utilisé pour arrêter manuellement le scénario, tandis que `Get-ServiceFabricChaosReport` obtient un rapport. Pour plus d’informations, consultez les [informations de référence sur Azure Service Fabric Powershell](/powershell/module/servicefabric/?view=azureservicefabricps) et [Induction d’un chaos contrôlé dans les clusters Service Fabric](service-fabric-controlled-chaos.md).

```powershell
$connection = "localhost:19000"
$timeToRun = 60
$maxStabilizationTimeSecs = 180
$concurrentFaults = 3
$waitTimeBetweenIterationsSec = 60

Connect-ServiceFabricCluster $connection

Invoke-ServiceFabricChaosTestScenario -TimeToRunMinute $timeToRun -MaxClusterStabilizationTimeoutSec $maxStabilizationTimeSecs -MaxConcurrentFaults $concurrentFaults -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec
```


## <a name="failover-test"></a>Test de basculement
Le scénario de test de basculement est une version du test chaos qui cible une partition de service spécifique. Il évalue l’effet du basculement sur une partition spécifique de service, sans affecter les autres services. Une fois configuré avec les informations de partition cible et d’autres paramètres, il s’exécute en tant qu’outil côté client à l’aide des API C# ou de PowerShell pour générer des erreurs associées à une partition de service. Le scénario effectue une itération dans une séquence d’erreurs simulées et de validation de service, tandis que votre logique métier s’exécute sur le côté pour fournir une charge de travail. Un échec de validation de service indique une erreur nécessitant un examen approfondi.

### <a name="faults-simulated-in-the-failover-test"></a>Erreurs simulées dans le test de basculement
* Redémarrez un package de code déployé à l’emplacement d’hébergement de la partition
* Supprimez une instance sans état ou un réplica principal/secondaire
* Redémarrez un réplica principal/secondaire (en cas de service persistant)
* Déplacez un réplica principal
* Déplacez un réplica secondaire
* Redémarrez la partition

Le test de basculement introduit une erreur déterminée, avant d’exécuter une validation du service afin d’évaluer sa stabilité. Le test de basculement incorpore une erreur à la fois, contrairement au test chaos, qui en introduit plusieurs simultanément. Si la partition ne se stabilise pas dans le délai configuré après chacune des erreurs, le test est considéré comme échoué. Le test introduit uniquement des erreurs non critiques. Cela signifie qu’en l’absence de défaillances externes, aucune perte de données ni de quorum ne survient.

### <a name="important-configuration-options"></a>Options de configuration importantes
* **PartitionSelector**: objet de sélecteur qui spécifie la partition à cibler.
* **TimeToRun**: durée totale d’exécution du test.
* **MaxServiceStabilizationTimeout**: délai maximal nécessaire à la restauration de l’intégrité du cluster, préalablement à l’échec du test. Les contrôles consistent à vérifier que l’intégrité du service est acceptable, que la taille cible du jeu de réplicas est atteinte pour l’ensemble des partitions et qu’aucun réplica InBuild n’existe.
* **WaitTimeBetweenFaults**: délai d’attente avant chaque erreur et cycle de validation.

### <a name="how-to-run-the-failover-test"></a>Procédure d’exécution du test de basculement
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
