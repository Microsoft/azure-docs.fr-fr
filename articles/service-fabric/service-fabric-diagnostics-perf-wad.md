---
title: Supervision des performances avec Diagnostics Azure pour Windows
description: Utilisez Diagnostics Azure pour Windows pour collecter les compteurs de performances de vos clusters Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 11/21/2018
ms.author: srrengar
ms.openlocfilehash: c2114f5392da788bb440589e69e704a148731e02
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86258612"
---
# <a name="performance-monitoring-with-the-windows-azure-diagnostics-extension"></a>Analyse des performances avec l’extension Diagnostics Azure pour Windows

Ce document explique comment configurer la collecte des compteurs de performances via Diagnostics Azure pour Windows (WAD) pour les clusters Windows. Pour les clusters Linux, configurez l’[agent Log Analytics](service-fabric-diagnostics-oms-agent.md) afin de collecter les compteurs de performances de vos nœuds. 

 > [!NOTE]
> Pour cette procédure, l’extension Microsoft Azure Diagnostics doit être déployée sur votre cluster. S’il n’est pas configuré, consultez [Agrégation et collecte d’événements à l’aide de Diagnostics Azure pour Windows](service-fabric-diagnostics-event-aggregation-wad.md).  


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="collect-performance-counters-via-the-wadcfg"></a>Collecter des compteurs de performances via le WadCfg

Pour collecter les compteurs de performances via Microsoft Azure Diagnostics, vous devez modifier la configuration dans le modèle Resource Manager de votre cluster. Suivez ces étapes pour ajouter un compteur de performances à collecter dans votre modèle, puis exécutez une mise à niveau des ressources Resource Manager.

1. Recherchez la configuration Microsoft Azure Diagnostics dans les modèles de votre cluster (recherchez `WadCfg`). Vous allez ajouter des compteurs de performances à collecter sous `DiagnosticMonitorConfiguration`.

2. Définissez votre configuration pour collecter les compteurs de performances en ajoutant la section suivante à votre `DiagnosticMonitorConfiguration`. 

    ```json
    "PerformanceCounters": {
        "scheduledTransferPeriod": "PT1M",
        "PerformanceCounterConfiguration": []
    }
    ```

    `scheduledTransferPeriod` définit à quelle fréquence les valeurs des compteurs collectées sont transférées vers votre table de stockage Azure et tous les récepteurs configurés. 

3. Ajoutez les compteurs de performances à collecter au `PerformanceCounterConfiguration` qui a été déclaré à l’étape précédente. Chaque compteur à collecter est défini avec un `counterSpecifier`, un `sampleRate`, un `unit` et un `annotation`, ainsi qu’avec tous les `sinks` applicables.

Voici un exemple de configuration avec le compteur pour le *Temps total du processeur* (la quantité de temps d’utilisation de l’UC pour les opérations de traitement) et les *appels de la Méthode Service Fabric Actor par seconde*, un des compteurs des performances personnalisés Service Fabric. Reportez-vous à l’article [Compteurs de performances de l’acteur fiable](service-fabric-reliable-actors-diagnostics.md#list-of-events-and-performance-counters) et [Compteurs de Performances du service fiable](service-fabric-reliable-serviceremoting-diagnostics.md#list-of-performance-counters) pour obtenir une liste complète des compteurs de performances personnalisés Service Fabric.

 ```json
 "WadCfg": {
         "DiagnosticMonitorConfiguration": {
           "overallQuotaInMB": "50000",
           "EtwProviders": {
             "EtwEventSourceProviderConfiguration": [
               {
                 "provider": "Microsoft-ServiceFabric-Actors",
                 "scheduledTransferKeywordFilter": "1",
                 "scheduledTransferPeriod": "PT5M",
                 "DefaultEvents": {
                   "eventDestination": "ServiceFabricReliableActorEventTable"
                 }
               },
               {
                 "provider": "Microsoft-ServiceFabric-Services",
                 "scheduledTransferPeriod": "PT5M",
                 "DefaultEvents": {
                   "eventDestination": "ServiceFabricReliableServiceEventTable"
                 }
               }
             ],
             "EtwManifestProviderConfiguration": [
               {
                 "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                 "scheduledTransferLogLevelFilter": "Information",
                 "scheduledTransferKeywordFilter": "4611686018427387904",
                 "scheduledTransferPeriod": "PT5M",
                 "DefaultEvents": {
                   "eventDestination": "ServiceFabricSystemEventTable"
                 }
               }
             ]
           },
           "PerformanceCounters": {
                 "scheduledTransferPeriod": "PT1M",
                 "PerformanceCounterConfiguration": [
                     {
                         "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                         "sampleRate": "PT1M",
                         "unit": "Percent",
                         "annotation": [
                         ],
                         "sinks": ""
                     },
                     {
                         "counterSpecifier": "\\Service Fabric Actor Method(*)\\Invocations/Sec",
                         "sampleRate": "PT1M",
                     }
                 ]
             }
         }
       },
  ```

 Vous pouvez modifier le taux d’échantillonnage du compteur selon vos besoins. Le format utilisé est le suivant : `PT<time><unit>`. Par conséquent, si vous souhaitez que le compteur soit collecté à chaque seconde, vous devez définir le taux d’échantillonnage ainsi : `"sampleRate": "PT15S"`.

 Vous pouvez également utiliser des variables dans votre modèle ARM pour collecter un tableau de compteurs de performances, lequel peut s’avérer utile lorsque vous collectez des compteurs de performances par processus. Dans l’exemple ci-dessous, nous collectons le temps processeur et le temps récupérateur de mémoire par processus, puis 2 compteurs de performance sur les nœuds eux-mêmes, uniquement à l’aide de variables. 

 ```json
"variables": {
  "copy": [
      {
        "name": "processorTimeCounters",
        "count": "[length(parameters('monitoredProcesses'))]",
        "input": {
          "counterSpecifier": "\\Process([parameters('monitoredProcesses')[copyIndex('processorTimeCounters')]])\\% Processor Time",
          "sampleRate": "PT1M",
          "unit": "Percent",
          "sinks": "applicationInsights",
          "annotation": [
            {
              "displayName": "[concat(parameters('monitoredProcesses')[copyIndex('processorTimeCounters')],' Processor Time')]",
              "locale": "en-us"
            }
          ]
        }
      },
      {
        "name": "gcTimeCounters",
        "count": "[length(parameters('monitoredProcesses'))]",
        "input": {
          "counterSpecifier": "\\.NET CLR Memory([parameters('monitoredProcesses')[copyIndex('gcTimeCounters')]])\\% Time in GC",
          "sampleRate": "PT1M",
          "unit": "Percent",
          "sinks": "applicationInsights",
          "annotation": [
            {
              "displayName": "[concat(parameters('monitoredProcesses')[copyIndex('gcTimeCounters')],' Time in GC')]",
              "locale": "en-us"
            }
          ]
        }
      }
    ],
    "machineCounters": [
      {
        "counterSpecifier": "\\Memory\\Available Bytes",
        "sampleRate": "PT1M",
        "unit": "KB",
        "sinks": "applicationInsights",
        "annotation": [
          {
            "displayName": "Memory Available Kb",
            "locale": "en-us"
          }
        ]
      },
      {
        "counterSpecifier": "\\Memory\\% Committed Bytes In Use",
        "sampleRate": "PT15S",
        "unit": "percent",
        "annotation": [
          {
            "displayName": "Memory usage",
            "locale": "en-us"
          }
        ]
      }
    ]
  }
....
"WadCfg": {
    "DiagnosticMonitorConfiguration": {
      "overallQuotaInMB": "50000",
      "Metrics": {
        "metricAggregation": [
          {
            "scheduledTransferPeriod": "PT1M"
          }
        ],
        "resourceId": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', variables('vmNodeTypeApp2Name'))]"
      },
      "PerformanceCounters": {
        "scheduledTransferPeriod": "PT1M",
        "PerformanceCounterConfiguration": "[concat(variables ('processorTimeCounters'), variables('gcTimeCounters'),  variables('machineCounters'))]"
      },
....
```

1. Une fois que vous avez ajouté les compteurs de performances à collecter appropriés, vous devez mettre à niveau votre ressource de cluster afin que ces modifications soient répercutées dans le cluster en cours d’exécution. Enregistrez le `template.json` modifié, puis ouvrez PowerShell. Vous pouvez mettre à niveau votre cluster à l’aide de `New-AzResourceGroupDeployment`. L’appel nécessite le nom du groupe de ressources, le fichier de modèle mis à jour et le fichier de paramètres. Ensuite, il demande à Resource Manager d’apporter les modifications nécessaires aux ressources que vous avez mises à jour. Une fois connecté à votre compte et au bon abonnement, utilisez la commande suivante pour exécuter la mise à niveau :

    ```sh
    New-AzResourceGroupDeployment -ResourceGroupName <ResourceGroup> -TemplateFile <PathToTemplateFile> -TemplateParameterFile <PathToParametersFile> -Verbose
    ```

1. Une fois la mise à niveau terminée (au bout de 15 à 45 minutes selon qu’il s’agisse du premier déploiement ou non, et selon la taille de votre groupe de ressources), WAD doit collecter les compteurs de performances et les envoyer à la table nommée WADPerformanceCountersTable dans le compte de stockage associé à votre cluster. Pour voir vos compteurs de performances dans Application Insights, [ajoutez le Récepteur IA au modèle Resource Manager](service-fabric-diagnostics-event-aggregation-wad.md#add-the-application-insights-sink-to-the-resource-manager-template).

## <a name="next-steps"></a>Étapes suivantes
* Collectez d’autres compteurs de performances pour votre cluster. Consultez [Mesures de performances](service-fabric-diagnostics-event-generation-perf.md) pour obtenir la liste des compteurs à collecter.
* [Utilisez les fonctionnalités de surveillance et de diagnostics avec une machine virtuelle Windows et des modèles Azure Resource Manager](../virtual-machines/extensions/diagnostics-template.md) pour apporter des modifications à votre `WadCfg` (y compris pour la configuration de comptes de stockage supplémentaires vers lesquels envoyer des données de diagnostics).
* Visitez le [générateur WadCfg](https://azure.github.io/azure-diagnostics-tools/config-builder/) pour créer un modèle à partir de zéro et assurez-vous que la syntaxe est correcte. Visitez https://azure.github.io/azure-diagnostics-tools/config-builder/) pour créer un modèle à partir de zéro et assurez-vous que la syntaxe est correcte.
