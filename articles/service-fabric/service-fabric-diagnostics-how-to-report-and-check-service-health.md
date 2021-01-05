---
title: Signaler et contrôler l’intégrité avec Azure Service Fabric
description: Découvrez comment envoyer des rapports d’intégrité à partir de votre code de service et comment contrôler l’intégrité de votre service avec les outils de contrôle d’intégrité fournis par Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 02/25/2019
ms.author: srrengar
ms.custom: devx-track-csharp
ms.openlocfilehash: 59c8202b03bf1be2be5a68b75a1d7c7404b2213d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998241"
---
# <a name="report-and-check-service-health"></a>Signaler et contrôler l’intégrité du service
Lorsque vos services rencontrent des problèmes, votre capacité à réagir et à résoudre les incidents et les pannes induits dépend de votre capacité à détecter les problèmes rapidement. En signalant les problèmes et les pannes au gestionnaire de contrôle d’intégrité Azure Service Fabric à partir de votre code de service, vous pouvez utiliser les outils standard de contrôle d’intégrité fournis par Service Fabric pour contrôler l’état d’intégrité.

Il existe trois méthodes pour signaler l’intégrité à partir du service :

* Utilisez les objets [Partition](/dotnet/api/system.fabric.istatefulservicepartition) ou [CodePackageActivationContext](/dotnet/api/system.fabric.codepackageactivationcontext).  
  Les objets `Partition` et `CodePackageActivationContext` peuvent vous servir à signaler l’intégrité d’éléments qui font partie du contexte actuel. Par exemple, le code s’exécutant dans le cadre d’un réplica ne peut signaler l’intégrité que sur ce réplica, la partition à laquelle il appartient et l’application dont il fait partie.
* Utilisez `FabricClient`.   
  Vous ne pouvez pas utiliser `FabricClient` pour signaler l’intégrité à partir du code de service si le cluster n’est pas [sécurisé](service-fabric-cluster-security.md) ou si le service s’exécute avec des privilèges d’administrateur. La plupart des scénarios réels n’utilisent pas de clusters non sécurisés et ne fournissent pas de privilèges Administrateur. Avec `FabricClient`, vous pouvez signaler l’intégrité de toute entité qui fait partie du cluster. Toutefois, dans l’idéal, le code de service n’est censé envoyer que des rapports liés à sa propre intégrité.
* Utilisez les API REST au niveau du cluster, de l’application, de l’application déployée, du service, du package de service, de la partition, du réplica ou du nœud. Cela peut servir pour générer des rapports sur l’état d’un conteneur.

Cet article vous présente un exemple de rapports d’intégrité du code de service. L’exemple montre également comment les outils fournis par Service Fabric peuvent être utilisés pour vérifier l’état d’intégrité. Cet article constitue une présentation rapide des fonctionnalités de contrôle d’intégrité de Service Fabric. Pour plus d’informations, vous pouvez lire la série d’articles détaillés sur l’intégrité, à commencer par le lien situé à la fin de cet article.

## <a name="prerequisites"></a>Prérequis
Les éléments suivants doivent être installés :

* Visual Studio 2015 ou Visual Studio 2019
* SDK Service Fabric

## <a name="to-create-a-local-secure-dev-cluster"></a>Pour créer un cluster local de développement sécurisé
* Ouvrez PowerShell avec des privilèges Administrateur et exécutez les commandes suivantes :

![Commandes montrant comment créer un cluster de développement sécurisé](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/create-secure-dev-cluster.png)

## <a name="to-deploy-an-application-and-check-its-health"></a>Pour déployer une application et contrôler son intégrité
1. Ouvrez Visual Studio en tant qu’administrateur.
1. Créez un projet à l’aide du modèle **Service avec état** .
   
    ![Créer une application Service Fabric avec des services avec état](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/create-stateful-service-application-dialog.png)
1. Appuyez sur **F5** pour exécuter l’application en mode débogage. L’application est déployée sur le cluster local.
1. Une fois que l’application est en cours d’exécution, cliquez avec le bouton droit sur l’icône du gestionnaire de cluster local dans la zone de notification et sélectionnez **Gérer le cluster local** dans le menu contextuel pour ouvrir Service Fabric Explorer.
   
    ![Ouvrez Service Fabric Explorer à partir de la zone de notification](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/LaunchSFX.png)
1. L’intégrité de l’application doit s’afficher comme dans cette image. À ce stade, l’application doit être saine et sans erreurs.
   
    ![Application saine dans l’Explorateur Service Fabric](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/sfx-healthy-app.png)
1. Vous pouvez également contrôler l’intégrité à l’aide de PowerShell. Vous pouvez utiliser ```Get-ServiceFabricApplicationHealth``` pour vérifier l’intégrité d’une application et ```Get-ServiceFabricServiceHealth``` pour vérifier l’intégrité d’un service. Le rapport d’intégrité pour la même application dans PowerShell figure dans cette image.
   
    ![Application saine dans PowerShell](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/ps-healthy-app-report.png)

## <a name="to-add-custom-health-events-to-your-service-code"></a>Pour ajouter des événements d’intégrité personnalisés à votre code de service
Les modèles de projet Visual Studio de Service Fabric contiennent des exemples de code. Les étapes suivantes montrent comment vous pouvez créer des rapports sur des événements d’intégrité personnalisés à partir de votre code de service. Ces rapports s’affichent automatiquement dans les outils standard de surveillance de l’intégrité fournis par Service Fabric, tels que Service Fabric Explorer, la vue d’intégrité du portail Azure et PowerShell.

1. Rouvrez l’application créée précédemment dans Visual Studio ou créez une application à l’aide du modèle **Service avec état** de Visual Studio.
1. Ouvrez le fichier Stateful1.cs, puis recherchez l’appel `myDictionary.TryGetValueAsync` dans la méthode `RunAsync`. La méthode renvoie un `result` contenant la valeur actuelle du compteur, car la logique principale de cette application est de tenir un décompte. S’il s’agissait d’une application réelle et que l’absence de résultat représentait un échec, il faudrait marquer cet événement.
1. Pour signaler un événement d’état quand l’absence de résultat représente un échec, ajoutez les étapes suivantes.
   
    a. Ajoutez l’espace de noms `System.Fabric.Health` au fichier Stateful1.cs.
   
    ```csharp
    using System.Fabric.Health;
    ```
   
    b. Ajoutez le code suivant après l’appel `myDictionary.TryGetValueAsync`
   
    ```csharp
    if (!result.HasValue)
    {
        HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
        this.Partition.ReportReplicaHealth(healthInformation);
    }
    ```
    Nous signalons l’intégrité du réplica, car il provient d’un service avec état. Le paramètre `HealthInformation` stocke les informations relatives au problème d’intégrité signalé.
   
    Si vous aviez créé un service sans état, utilisez le code suivant
   
    ```csharp
    if (!result.HasValue)
    {
        HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
        this.Partition.ReportInstanceHealth(healthInformation);
    }
    ```
1. Si votre service s’exécute avec des privilèges d’administrateur ou que le cluster n’est pas [sécurisé`FabricClient`, vous pouvez également utiliser ](service-fabric-cluster-security.md) pour signaler l’intégrité comme indiqué dans les étapes suivantes.  
   
    a. Créez l’instance `FabricClient` après la déclaration `var myDictionary`.
   
    ```csharp
    var fabricClient = new FabricClient(new FabricClientSettings() { HealthReportSendInterval = TimeSpan.FromSeconds(0) });
    ```
   
    b. Ajoutez le code suivant après l’appel `myDictionary.TryGetValueAsync` :
   
    ```csharp
    if (!result.HasValue)
    {
       var replicaHealthReport = new StatefulServiceReplicaHealthReport(
            this.Context.PartitionId,
            this.Context.ReplicaId,
            new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error));
        fabricClient.HealthManager.ReportHealth(replicaHealthReport);
    }
    ```
1. Simulons cette panne et voyons comment elle s’affiche dans les outils de contrôle d’intégrité. Pour simuler la panne, commentez la première ligne dans le code de rapport d’intégrité ajouté précédemment. Une fois le commentaire ajouté à la première ligne, le code se présente comme suit.
   
    ```csharp
    //if(!result.HasValue)
    {
        HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
        this.Partition.ReportReplicaHealth(healthInformation);
    }
    ```
   Ce code déclenche le rapport d’intégrité à chaque exécution de `RunAsync`. Après avoir apporté la modification, appuyez sur **F5** pour exécuter l’application.
1. Une fois que l’application est en cours d’exécution, ouvrez Service Fabric Explorer pour vérifier l’intégrité de l’application. Cette fois-ci, Service Fabric Explorer affiche un problème d’intégrité de l’application. L’application s’affiche comme non saine en raison de l’erreur signalée à partir du code que nous avons ajouté précédemment.
   
    ![Application non saine dans l’Explorateur Service Fabric](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/sfx-unhealthy-app.png)
1. Si vous sélectionnez le réplica principal dans l’arborescence de Service Fabric Explorer, vous verrez que l’ **état d’intégrité** indique également une erreur. Service Fabric Explorer affiche également les détails du rapport d’intégrité qui ont été ajoutés au paramètre `HealthInformation` dans le code. Vous pouvez voir les mêmes rapports d’intégrité dans PowerShell, ainsi que dans le portail Azure.
   
    ![Intégrité du réplica dans l’Explorateur Service Fabric](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/replica-health-error-report-sfx.png)

Ce rapport est conservé dans Health Manager tant qu’il n’est pas remplacé par un autre rapport ou que ce réplica n’est pas supprimé. Étant donné que nous n’avons pas défini `TimeToLive` pour ce rapport d’intégrité dans l’objet `HealthInformation`, le rapport n’expire jamais.

Il est recommandé que l’intégrité soit signalée au niveau le plus granulaire, qui dans ce cas est le réplica. Vous pouvez également signaler l’intégrité sur `Partition`.

```csharp
HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
this.Partition.ReportPartitionHealth(healthInformation);
```

Pour créer un rapport d’intégrité sur `Application`, `DeployedApplication` et `DeployedServicePackage`, utilisez `CodePackageActivationContext`.

```csharp
HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
var activationContext = FabricRuntime.GetActivationContext();
activationContext.ReportApplicationHealth(healthInformation);
```

## <a name="next-steps"></a>Étapes suivantes
* [Présentation approfondie de l’intégrité de Service Fabric](service-fabric-health-introduction.md)
* [API REST pour les rapports d’intégrité du service](/rest/api/servicefabric/report-the-health-of-a-service)
* [API REST pour les rapports d’intégrité de l’application](/rest/api/servicefabric/report-the-health-of-an-application)
