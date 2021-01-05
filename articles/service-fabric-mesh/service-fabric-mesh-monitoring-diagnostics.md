---
title: Surveillance et diagnostics pour les applications Azure Service Fabric Mesh
description: Découvrez l’application de surveillance et de diagnostic dans Service Fabric mesh sur Azure.
author: srrengar
ms.topic: conceptual
ms.date: 03/19/2019
ms.author: srrengar
ms.custom: mvc, devcenter, devx-track-azurecli
ms.openlocfilehash: eda0b62729343b0a138d027548d8750b1e0fc74f
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844401"
---
# <a name="monitoring-and-diagnostics"></a>Surveillance et diagnostics
Azure Service Fabric mesh est un service entièrement géré qui permet aux développeurs de déployer des applications de microservices sans gestion de machines virtuelles, de stockage ou de mise en réseau. La surveillance et les diagnostics pour Service Fabric mesh portent sur trois types principaux de données de diagnostic :

- Journaux des applications : ils sont définis comme les journaux d’activité de vos applications en conteneur, selon la façon dont vous avez instrumenté votre application (par exemple, journaux d’activité Docker).
- Événements de plateforme : événements de la plateforme Mesh pertinents pour votre opération de conteneur. Ils incluent actuellement l’activation, la désactivation et l’arrêt de conteneur.
- Métriques de conteneur : métriques de l’utilisation et des performances des ressources pour vos conteneurs (statistiques Docker).

Cet article décrit les options de surveillance et de diagnostic pour la dernière préversion disponible.

## <a name="application-logs"></a>Journaux d’activité d’application

Vous pouvez afficher les journaux d’activité Docker de vos conteneurs déployés, séparément pour chaque conteneur. Dans le modèle d’application Service Fabric mesh, chaque conteneur est un package de code dans votre application. Pour afficher les journaux d’activité associés avec un package de code, utilisez la commande suivante :

```cli
az mesh code-package-log get --resource-group <nameOfRG> --app-name <nameOfApp> --service-name <nameOfService> --replica-name <nameOfReplica> --code-package-name <nameOfCodePackage>
```

> [!NOTE]
> Vous pouvez utiliser la commande « az mesh service-replica » pour obtenir le nom du réplica. Les noms de réplica sont des entiers incrémentés à partir de 0.

Voici à quoi cela ressemble pour l’affichage de journaux d’activité du conteneur VotingWeb.Code de l’application de vote :

```cli
az mesh code-package-log get --resource-group <nameOfRG> --application-name SbzVoting --service-name VotingWeb --replica-name 0 --code-package-name VotingWeb.Code
```

## <a name="container-metrics"></a>Métriques des conteneurs 

L’environnement Mesh expose un certain nombre de métriques indiquant le fonctionnement de vos conteneurs. Les métriques suivantes sont disponibles via le Portail Microsoft Azure et l’interface de ligne de commande Azure Monitor :

| Métrique | Description | Units|
|----|----|----|
| CpuUtilization | ActualCpu/AllocatedCpu sous forme de pourcentage | % |
| MemoryUtilization | ActualMem/AllocatedMem sous forme de pourcentage | % |
| AllocatedCpu | UC alloué conformément au modèle Azure Resource Manager | Millicores |
| AllocatedMemory | Mémoire allouée conformément au modèle Azure Resource Manager | Mo |
| ActualCpu | Utilisation de l’UC | Millicores |
| ActualMemory | Utilisation de la mémoire | Mo |
| ContainerStatus | 0 - Non valide : L’état du conteneur est inconnu <br> 1 - En attente : Le conteneur a planifié le démarrage <br> 2 - Démarrage en cours : Le conteneur est en cours de démarrage <br> 3 - Démarré : Le conteneur a démarré avec succès <br> 4 - Arrêt en cours : Le conteneur est en cours d’arrêt <br> 5 - Arrêté : Le conteneur s’est arrêté avec succès | N/A |
| ApplicationStatus | 0 - Inconnu : L’état n’est pas récupérable <br> 1- Prêt : L’application s’exécute avec succès <br> 2 - Mise à niveau en cours: Une mise à niveau est en cours <br> 3 - Création en cours : L’application est en cours de création <br> 4 - Suppression en cours : L’application est en cours de suppression <br> 5 - Échec : Le déploiement de l’application a échoué | N/A |
| ServiceStatus | 0 - Non valide : Le service ne dispose pas actuellement d’un état d’intégrité <br> 1 - OK : Le service est sain  <br> 2 - Avertissement : Il peut y avoir un problème nécessitant une investigation <br> 3 - Erreur : Il existe un problème qui nécessite une investigation <br> 4 - Inconnu : L’état n’est pas récupérable | N/A |
| ServiceReplicaStatus | 0 - Non valide : Le réplica ne dispose pas actuellement d’un état d’intégrité <br> 1 - OK : Le service est sain  <br> 2 - Avertissement : Il peut y avoir un problème nécessitant une investigation <br> 3 - Erreur : Il existe un problème qui nécessite une investigation <br> 4 - Inconnu : L’état n’est pas récupérable | N/A | 
| RestartCount | Nombre de redémarrages du conteneur | N/A |

> [!NOTE]
> Les valeurs ServiceStatus et ServiceReplicaStatus sont les mêmes que [HealthState](/dotnet/api/system.fabric.health.healthstate?view=azure-dotnet) dans Service Fabric. 

Chaque métrique est disponible sur les différentes dimensions afin que vous puissiez voir les agrégats à différents niveaux. Voici la liste actuelle des dimensions :

* ApplicationName
* NomService
* ServiceReplicaName
* CodePackageName

> [!NOTE]
> La dimension CodePackageName n’est pas disponible pour les applications Linux. 

Chaque dimension correspond à différents composants du [modèle d’application Service Fabric](service-fabric-mesh-service-fabric-resources.md#applications-and-services)

### <a name="azure-monitor-cli"></a>Interface de ligne de commande Azure Monitor

Une liste complète des commandes est disponible dans les [documents de l’interface de ligne de commande Azure Monitor](/cli/azure/monitor/metrics?view=azure-cli-latest#az-monitor-metrics-list), mais nous avons inclus quelques exemples utiles ci-dessous 

Dans chaque exemple, l’ID de ressource suit ce modèle

`"/subscriptions/<your sub ID>/resourcegroups/<your RG>/providers/Microsoft.ServiceFabricMesh/applications/<your App name>"`


* Utilisation de l’UC des conteneurs dans une application

```cli
    az monitor metrics list --resource <resourceId> --metric "CpuUtilization"
```
* Utilisation de la mémoire pour chaque réplica de service
```cli
    az monitor metrics list --resource <resourceId> --metric "MemoryUtilization" --dimension "ServiceReplicaName"
``` 

* Redémarre pour chaque conteneur dans une fenêtre d’une heure 
```cli
    az monitor metrics list --resource <resourceId> --metric "RestartCount" --start-time 2019-02-01T00:00:00Z --end-time 2019-02-01T01:00:00Z
``` 

* Utilisation moyenne de l’UC entre les services nommés « VotingWeb » dans une fenêtre d’une heure
```cli
    az monitor metrics list --resource <resourceId> --metric "CpuUtilization" --start-time 2019-02-01T00:00:00Z --end-time 2019-02-01T01:00:00Z --aggregation "Average" --filter "ServiceName eq 'VotingWeb'"
``` 

### <a name="metrics-explorer"></a>Metrics Explorer

Metrics Explorer est un panneau dans le portail dans lequel vous pouvez visualiser toutes les métriques de votre application Mesh. Ce panneau est accessible dans la page de l’application du portail et dans le panneau Azure Monitor, ce dernier pouvant être utilisé pour afficher les métriques de toutes vos ressources Azure qui prennent en charge Azure Monitor. 

![Metrics Explorer](./media/service-fabric-mesh-monitoring-diagnostics/metricsexplorer.png)


<!--
### Container Insights

In addition to the metrics explorer, we also have a dashboard available out of the box that shows sample metrics over time under the Insights blade in the application's page in the portal. 

![Container Insights](./media/service-fabric-mesh-monitoring-diagnostics/containerinsights.png)
-->

## <a name="next-steps"></a>Étapes suivantes
* Pour en savoir plus sur Service Fabric mesh, voir la [vue d’ensemble de Service Fabric mesh](service-fabric-mesh-overview.md).
* Pour en savoir plus sur les commandes des métriques Azure Monitor, consultez les [documents de l’interface de ligne de commande Azure Monitor](/cli/azure/monitor/metrics?view=azure-cli-latest#az-monitor-metrics-list).
